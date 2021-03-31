---
layout: post
title: Clang AST Matcher 源码阅读与实现（二）
categories: [C++, LLVM]
description: 阅读 Clang AST Matcher 的源码，并在自己的语言上实现一个 Matcher。
keywords: C++, LLVM
---

我们来继续吧。（说实话我有点煎熬……）

## AST Matcher 的实现（二）

这一部分的内容构造很复杂，但是有了上一篇的基础后，其实不难理解。不过这一章会出现大量的代码！不过不用害怕，大部分都是针对一个枚举类型的不同实例而已。

### AllOf 与变长：Node Matcher 的完整实现

以 `ifStmt` 为例，作为一个 Node Matcher，他本应该可以接受若干个匹配器，当且仅当这些匹配器全都正确匹配之后，该匹配器才算上是匹配成功了。这其中包含了一个 `AllOf` 的语义，同时还有一个**变长参数**的问题。这两个问题将是这一个小节的目标。

我们先实现一个支持变长参数的函数对象（其实这是个支线），这个东西挺直观的，所以我就不再解释了吧~

``` cpp
template <typename ResultT, typename ArgT, ResultT (*Func)(std::vector<ArgT>)>
struct VariadicFunction {
  ResultT operator()() const { return Func(std::vector<ArgT>()); }

  template <typename... ArgsT>
  ResultT operator()(ArgT arg1, ArgsT... args) const {
    return execute(arg1, static_cast<ArgT>(args)...);
  }

  ResultT operator()(std::vector<ArgT> args) const { return Func(args); }

 private:
  template <typename... ArgsT>
  ResultT execute(ArgsT... args) const {
    std::vector<ArgT> args_vector = std::vector<ArgT>({args...});
    return Func(args_vector);
  }
};
```

为什么不用 C++ 11 的 Variadic Function？Most of the functions below that use VariadicFunction could be implemented using plain C++11 variadic functions, but the function object allows us to capture it on the dynamic matcher registry.
{:.warning}

下面我们来完成另一个目标：构建一个 `AllOf` Matcher，他接受一个 IRNodeType，和一组 DynTypedMatcher。类似地，我们需要一个 `AllOfMatcherInterface`，对每个子匹配器进行匹配。

``` cpp
class AllOfMatcherInterface : public DynMatcherInterface {
  public:
    AllOfMatcherInterface(std::vector<DynTypedMatcher> innerMatchers)
        : innerMatchers(std::move(innerMatchers)) {}
    
    bool dynMatches(Node* node, BoundNodesTreeBuilder* builder) const override {
        return all_of(innerMatchers, [&](const DynTypedMatcher& innerMatcher) {
           	return innerMatcher.matches(node, builder); 
        });
    }
  private:
    std::vector<DynTypedMatcher> innerMatchers;
};
```

OK！现在变长和 AllOf 的语义都有了，我们就该构建我们的 `ifStmt` 了。注意：`ifStmt` 是一个 接受任意个匹配器作为参数的、包含 AllOf 语义的、检查节点是 ifStmt 的、可绑定的匹配器。

注意：变长参数函数对象应该是最外层的，因为只有他拥有 `operator()` 方法。这个函数对象的返回值应当是 Bindable 的，因为需要做 `.bind` 绑定。
{:.info}

我们先写一个辅助的函数用来塞到 VariadicFunction 里面作为其函数体，它返回一个可绑定的匹配器：

```cpp
template <typename T>
BindableMatcher makeAllOfComposite(
    std::vector<DynTypedMatcher> innerMatchers) {
  if (inner_matchers.empty()) {
    return BindableMatcher(::ir::type_traits::TypeByTemplate<T>::value,
                           new TrueMatcherInterface());
  }

  if (inner_matchers.size() == 1) {
    return BindableMatcher(innerMatchers[0]);
  }

  return BindableMatcher(
      DynTypedMatcher(
          ::ir::type_traits::TypeByTemplate<T>::value,
          AllOfMatcherInterface(innerMatchers)));
}
```

这里有一个 `TrueMatcherInterface`，我们将放在这一章的最后讨论。
{:.info}

然后，我们把这些东西塞进 VariadicFunction 里面，称为 VariadicAllOfMatcher：

``` cpp
template <typename T>
class VariadicAllOfMatcher
    : public VariadicFunction<BindableMatcher, DynTypedMatcher,
                              makeAllOfComposite<T>> {
 public:
  VariadicAllOfMatcher() = default;
};
```

最终！我们可以定义出我们的 `ifStmt`：

``` cpp
const VariadicAllOfMatcher<ir::IfStmt> ifStmt;
```

*真的是太曲折了，写到这里我都要落泪了！！* 类似地，我们可以定义出所有我们想要的 Node Matcher 了，他们都有相同的实现方式。

``` cpp
VariadicAllOfMatcher<ir::Stmt> stmt;
VariadicAllOfMatcher<ir::IfStmt> ifStmt;
VariadicAllOfMatcher<ir::Expr> expr;
...
```

### 更多 Variadic 匹配器

正如我们在上一篇文章中提到的，除了 AllOf，Clang AST Matcher 还提供其他的几种语义，他们在实现上有着类似的方式。因此接下来，我们可以将整合实现其他的一些 Variadic Operator。Clang 提供了如下几种 Variadic Operator：

``` cpp
  enum VariadicOperator {
    VO_AllOf,
    VO_AnyOf,
    VO_EachOf,
    VO_Optionally,
    VO_UnaryNot
  };
```

我们给出共同的一套 MatcherInterface 来代替上文的 AllOfMatcherInterface。他们的都是对一组内部匹配器进行某种目标的匹配。

``` cpp
template <VariadicOperatorFunction Func>
class VariadicMatcherInterface : public DynMatcherInterface {
public:
  VariadicMatcher(std::vector<DynTypedMatcher> innerMatchers)
      : innerMatchers(std::move(InnerMatchers)) {}

  bool dynMatches(Node* node, BoundNodesTreeBuilder* builder) const override {
    return Func(node, builder, innerMatchers);
  }

private:
  std::vector<DynTypedMatcher> innerMatchers;
};
```

然后我们要为每种 Variadic Operator 提供它的 Function（可以看一个 AllOf 其他的直接跳过哟，不影响后面的内容哒）

``` cpp
bool AllOfVariadicOperator(Node* node,
                           BoundNodesTreeBuilder* builder,
                           std::vector<DynTypedMatcher> inner_matchers) {
  return all_of(inner_matchers, [&](const DynTypedMatcher& matcher) {
    return matcher.matches(node, builder);
  });
}


bool EachOfVariadicOperator(Node* node,
                            BoundNodesTreeBuilder* builder,
                            std::vector<DynTypedMatcher> inner_matchers) {
  BoundNodesTreeBuilder result;
  bool matched = false;
  for (const DynTypedMatcher& inner_matcher : inner_matchers) {
    BoundNodesTreeBuilder builder_inner(*builder);
    if (inner_matcher.matches(node, &builder_inner)) {
      matched = true;
      result.addMatch(builder_inner);
    }
  }
  *builder = std::move(result);
  return matched;
}

bool AnyOfVariadicOperator(Node* node,
                           BoundNodesTreeBuilder* builder,
                           std::vector<DynTypedMatcher> inner_matchers) {
  for (const DynTypedMatcher& inner_matcher : inner_matchers) {
    BoundNodesTreeBuilder result = *builder;
    if (inner_matcher.matches(node, &result)) {
      *builder = std::move(result);
      return true;
    }
  }
  return false;
}

bool OptionallyVariadicOperator(Node* node
                                BoundNodesTreeBuilder* builder,
                                std::vector<DynTypedMatcher> inner_matchers) {
  if (inner_matchers.size() != 1) return false;

  BoundNodesTreeBuilder result(*builder);
  if (inner_matchers[0].matches(node, &result))
    *builder = std::move(result);

  return true;
}

bool NotUnaryVariadicOperator(Node* node
                              BoundNodesTreeBuilder* builder,
                              std::vector<DynTypedMatcher> inner_matchers) {
  if (inner_matchers.size() != 1) return false;
  BoundNodesTreeBuilder discard(*builder);
  return !inner_matchers[0].matches(node, &discard);
}
```

为了使用时的方便，我们可以创建一个从 VariadicOperator 到这些 Matcher 的映射。我们可以将其放在 DynTypedMatcher 中作为其一个**静态**方法使用。

``` cpp

DynTypedMatcher DynTypedMatcher::ConstructVariadic(
    VariadicOperator op, IRNodeType nodeType, std::vector<DynTypedMatcher> innerMatchers) {
  ASSERT(!innerMatchers.empty(), "Inner matchers must not be empty.");

  IRNodeType t = supported_node_type;

  switch (op) {
    case VariadicOperator::VO_AllOf:
      return DynTypedMatcher(
          t, new VariadicMatcherInterface<AllOfVariadicOperator>(
                 std::move(innerMatchers)));

    case VariadicOperator::VO_AnyOf:
      return DynTypedMatcher(
          t, new VariadicMatcherInterface<AnyOfVariadicOperator>(
                 std::move(innerMatchers)));

    case VariadicOperator::VO_EachOf:
      return DynTypedMatcher(
          t, new VariadicMatcherInterface<EachOfVariadicOperator>(
                 std::move(inner_matchers)));

    case VariadicOperator::VO_Optionally:
      return DynTypedMatcher(
          t, new VariadicMatcherInterface<OptionallyVariadicOperator>(
                 std::move(inner_matchers)));

    case VariadicOperator::VO_UnaryNot:
      return DynTypedMatcher(
          t, new VariadicMatcherInterface<NotUnaryVariadicOperator>(
                 std::move(inner_matchers)));
  }
}
```

### `TrueMatcherInterface`

TrueMatcher 是一个永远返回 True 的匹配器。它的实现非常简单，对于任意的输入都只要简单的返回 True 就可以了。

``` cpp
class TrueMatcherInterface : public DynMatcherInterface {
public:
  bool dynMatches(Node*, BoundNodesTreeBuilder*) const override {
    return true;
  }
};
```

因为任何时候，我们仅需要一个 TrueMatcher 的实例。所以在 Clang 中，TrueMatcher 被进行了严格的内存管理，这可能使得一些部分看起来不是非常的自然。不过没关系，任何时候看到 TrueMatcher，就看成是上面的 Interface 或是由它构建的 DynTypedMatcher 就好了。
{:.warning}

