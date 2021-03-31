---
layout: post
title: Clang AST Matcher 源码阅读与实现（一）
categories: [C++, LLVM]
description: 阅读 Clang AST Matcher 的源码，并在自己的语言上实现一个 Matcher。
keywords: C++, LLVM
---

本文并不会详细的讲解 Clang AST Matcher 的具体使用方法，以及如何通过 clang-query 来尝试它们，相关的内容已有较多博客介绍。本文将着重于其具体实现，以及如何在你的 AST 或 IR 中实现 AST Matcher。
{:.success}

本文将尝试实现一个 AST Matcher，主要依照 Clang AST Matcher 的结构。因此可以将本文看作是研读 Clang 源代码*（版本：llvm-project-11.0.0）*，并探讨其具体的实现方法和设计理由。由于这部分代码与设计过于复杂，~~并且有了之前给别人完全没讲明白的经验，~~本文将采用自上而下的方法进行讨论。即从目标入手，在一点点地细化。

首先，AST Matcher 是用来干嘛的？首先是分析代码，查找代码中的特定结构（比如做个查重之类的工作？）。其次是优化代码，对匹配特定结构的代码进行简单的重构。

## AST Matcher 的简单介绍

我们从一个没有什么用的需求开始：将一个 if 的条件取否定，并将 then 和 else 的语句块进行调换。例如原始代码为

``` cpp
if (10 > 1.0) log("oh no!");
else log("ok");
```

则要将其转换为

``` cpp
if (!(10 > 1.0)) log("ok");
else log("oh no!");
```

（注意虽然这里写成代码，但实际上我们是对 AST 进行操作）首先我们要做的是对 if 进行匹配，记住 then block 和 else block。我们给出这部分的写法，然后再进行详细介绍：

``` cpp
ifStmt(hasCondition(expr().bind("C")), 
       hasThen(stmt().bind("T")),
       hasElse(stmt().bind("E")))
```

首先先明确他们是什么！这非常的关键，例如，这里的 `ifStmt` 并非 IR 节点，而是一个匹配器实例！

- `ifStmt`：用来判断匹配当前节点是否为 if 节点，并且满足括号内的其他匹配器要求。他是一个包含 `operator()` 方法的**类的实例**，但是你也可以将其简单地看作是一个**函数**，原因我们将会在后文继续进行解释。
- `expr` 和 `stmt`：与 `ifStmt` 类似，也是匹配器的示例，用于匹配 Expression 和 Statement。
- `.bind(str)`：*一些* 匹配器拥有的方法，它可以将被匹配的这个子表达式树绑定到 `str` 中。例如上文中，匹配成功后会将 `(10 > 0.1)` 绑定到 `C` 上，我们后续可以通过 `C` 找到这个表达式。
- `hasConditon`、`hasThen`、`hasElse`：它是一个作用于特定节点类型的匹配器，它尝试去获得节点的子树。

也许你注意到了，上面的匹配器看起来有两种不同的种类，前几个是用来匹配特定类型节点的，而后几个则是用来进行深层次遍历的。在 Clang 中，AST Matcher 被分为了三类：

1. 节点匹配器（Node Matchers）：用于匹配特定类型的节点，是匹配器的核心。同时，**Node Matchers 是唯一具有 bind 方法的一类匹配器**。例如 `ifStmt`、`expr`、`classTemplateDecl` 等。
2. 窄化匹配器（Narrowing Matchers）：对当前节点上的某些属性进行匹配，从而缩小当前类型的节点集的匹配范围。例如 `isPrivate`、`equals`、`isConst` 等。此外，有一些逻辑匹配器（`allOf`、`anyOf`、`anything`、`unless`），他们能构成更强大的匹配器。
3. 遍历匹配器（Traversal Matchers）：遍历匹配器指定从当前节点可到达的其他节点的关系。例如 `hasCondition`、`hasParent`、`hasLHS` 等。此外，有一些强大的匹配器 （`has`、`hasDecendant`、`forEach`）可以构成更强大的匹配器。

每一个匹配表达式都以 Node Matchers 开始，然后使用 Narrowing 或 Traversal Matcher 进一步完善。所有遍历匹配器都将 Node Matcher 作为其参数。此外，每个 Node Matcher 都隐含了一个 allOf Matcher。
{:error}

Clang 中所有可使用的 Matcher 可以在 [AST Matcher Reference](https://clang.llvm.org/docs/LibASTMatchersReference.html) 中找到。

接下来我们将对我们的语言实现一个类似的功能！首先我们要给我们的语言 AST 做一些假设。

## AST Matcher 的实现（一）

### 本博客中使用的 AST 结构

我们给出一个简单的节点关系：

``` 
Node
|--- Stmt
|    |--- IfStmt
|    |--- ExprStmt
|    |--- BlockStmt
|    |--- AssignmentStmt
|
|--- Expr
|    |--- Var
|    |--- BoolConst
|    |--- IntConst
|    |--- Unary   // + - !
|    |--- Binary  // + - * / && || ... 
|    |--- Logical // > < == != ...
```

其中 Node 是 Stmt 和 Expr 的父类，依次类推。每个节点拥有一个 `getType` 的方法，返回一个节点类型（这一定是最右侧的一个节点类型！）。

### AST Matcher 的逐步实现

以下的结构均为 首先我们设定一个目标，我们尝试去实现它，最后再去阅读 Clang 中相关的代码。在读源码的过程中，与本节无关的内容请读者暂且忽略。
{:.success}

#### Matcher 的简单实现：节点类型检查

我们先来尝试实现一个简单的 ifStmt。或者说，我们实现一类节点匹配器。他很菜，不能添加子匹配器做 allof，甚至尚还不能绑定，但是他能匹配节点检查类型。我们把这个匹配器类叫做 `DynTypedMatcher`，他在构建的过程中记录这个匹配器将要匹配的节点类型，并且让他具有一个 `matches` 方法，检查节点类型。

``` cpp
class DynTypedMatcher {
  public:
    DynTypedMatcher(IRNodeType supportedNodeType)
        : supportedNodeType(supportedNodeType) {}
    
    bool canMatchNodesOfKind(IRNodeType nodeType) const {
      return ::ir::isBaseOf(supportedNodeType, nodeType);    
    }
    
    bool matches(Node* node) const {
        return canMatchNodesOfKind(node->getType());
    }
    
  private:
    IRNodeType supportedNodeType;
};
```

不要太过放松，难的东西马上就要来了。在这之前，我们先定位一下 Clang 源代码的位置：`clang/include/clang/ASTMatchers/ASTMatcherInternal.h:349`。记得忽略掉还没有讨论到的东西！

1. 你可能注意到了后面还有一个 `Matcher<T>` 的匹配器，用来匹配 `T` 类型节点的匹配器。这类匹配器可以在编译时期检查到节点类型不符合，本文为了简洁，忽略掉这部分内容。
2. Clang 中的 private 部分记录了两种类型，分别是 `SupportedKind` 和 `RestrictKind`，但是被初始化成了相同的值。说实话笔者这里也没太看懂……希望读懂的朋友留言。
{:.error}

#### MatcherInterface

以上的匹配器看起来比较简单，事实也是如此。但是考虑到匹配器不仅有 Node Matcher，还需要 Narrowing Matcher 和 Traversal Matcher 提供更详细的功能。我们将这些“功能”看作是 Matcher 的接口，将 Matcher 看作这些功能的具体实现。

这一步看起来比较抽象，可能完全不理解为什么不实现更多的 DynTypedMatcher。在下一步我们将会为 DynTypedMatcher 实现更多的功能，使用 Interface 的好处便会体现出来了。

Interface 分为 `DynMatcherInterface` 和 `MatcherInterface<T>`，他们都是抽象类。

``` cpp
class DynMatcherInterface {
public:
  virtual ~DynMatcherInterface() = default;

  virtual bool dynMatches(const Node* node) const = 0;
};

template <typename T> 
class MatcherInterface : public DynMatcherInterface {
public:
  virtual bool matches(const T* Node) const = 0;

  bool dynMatches(const Node* node) const override {
    return matches(::ir::ptr_cast<T>(node));
  }
};
```

同时，我们在 DynTypedMatcher 中记录这些 Interface。现在我们不仅要求节点类型匹配，也需要 implementation 满足匹配要求。

``` cpp
class DynTypedMatcher {
  public:
    // changed
    DynTypedMatcher(IRNodeType supportedNodeType, DynMatcherInterface* implementation)
        : supportedNodeType(supportedNodeType), 
          implementation(implementation) {}
    
    bool canMatchNodesOfKind(IRNodeType nodeType) const {
      return ::ir::isBaseOf(supportedNodeType, nodeType);    
    }
    
    // changed
    bool matches(Node* node) const {
        if (canMatchNodesOfKind(node->getType()) && 
            implementation->dynMatches(node)) {
            return true;
        };
        return false;
    }
    
  private:
    IRNodeType supportedNodeType;
    DynMatcherInterface* implementation; // changed
};
```

这里我们实现一个 hasCond 来举例说明 Interface 的作用。我们先确认一下 `hasCond` 匹配器的性质：

1. hasCond 匹配器仅作用于 IfStmt 节点 → 实现为 `MatcherInterface<IfStmt>`；
2. 取一个 Node Matcher 作为内部匹配器，例如 `expr` → 构造时取一个 `DynTypedMatcher` 为参数；
3. 匹配成功的条件是：节点是 IfStmt 类型、有 condition 成员变量、内部匹配器匹配成功。

这些性质在它的 Interface 中体现。

``` cpp
class hasCond_MatcherInterface: public MatcherInterface<IfStmt> {
  public:
    explicit hasCond_MatcherInterface(DynTypedMatcher& innerMatcher)
        : innerMatcher(innerMatcher) {}
    
    bool matches(IfStmt* node) const override {
    	Expr* condition = node->condition;
        return (conditon && innerMatcher.matches(condition));
    }
    
  private:
    DynTypedMatcher innerMatcher;
};
```

`hasCond` 的定义仅仅是返回一个以 `hasCond_MatcherInterface` 为 implement 的 `DynTypedMatcher`。

``` cpp
DynTypedMatcher hasCond(DynTypedMatcher& innerMatcher) {
    return DynTypedMatcher(IRNodeType::IfStmt, 
                           new hasCond_MatcherInterface(innerMatcher));
}
```

在 Clang 中，`hasCond` 等一类匹配器是通过统一的宏实现的，这是因为大部分的 Traversal Matcher 的实现方法与上例使用了相同的 Pattern，利于对匹配器的扩展。
{:.error}

下面我们给 Matcher 加一个新功能：绑定！

#### BindableMatcher：可绑定的 Matcher

还是明确一下实现这个功能的思路……

首先我们可以假设我们的绑定是通过 `.bind(const std::string&)` 方法，那么这个方法应该返回一个什么？还是一个 Matcher！准确的说，是添加了这个变量绑定的相同的 Matcher。

其次，我们绑定的结果应该保存在哪里？需要通过参数指针传递！也就是说我们的 `matches` 函数要变样子了（包含以上所有的 match 的函数）：

``` cpp
bool dynMatches(Node* node, BoundNodesTreeBuilder* builder)
```

其中第二个参数便是用于完成将字符串和节点匹配的数据结构，我们可以暂时忽略其实现细节，假设其含有两个方法：

```cpp
void setBinding(const std::string&, Node*);

template <typename ExcludePredicate>
void removeBindings(const ExcludePredicate &predicate);
```

分别用于绑定变量和移除变量绑定。其中移除变量绑定的函数接受一个 Lambda 表达式，移除这些绑定（马上就会有例子！）。

另一种想法是可以作为成员变量随着匹配过程传递，但是这会让我们的代码变得复杂，我们期望这个过程是自动完成的。只在匹配的最后去查看相应的结果。
{:.info}

最后，正如前文所说，仅有 Node Matcher 是可绑定的，我们需要区分出这一类 Matcher，编译时期检查是否正确的使用了 `bind` 方法。

思路清晰了，现在我们开始实现。我们将这类 Matcher 称为 `BindableMatcher`，它继承自 `DynTypedMatcher`（不是 Interface！）。

我们的实现方法是这样的，我们将对 `DynTypedMatcher` 的定义做出修正，在其中添加一个 `allowBind` 的属性用于描述其是否是可以用于绑定的（默认是 false），BindableMatcher 只是简单的将其改成 true。DynTypedMatcher 中将提供一个 `tryBind` 方法，返回一个 `optional<DynTypedMatcher>` 用于检查 `allowBind` 属性是否为 true。BindableMatcher 中提供 `bind` 方法，调用基类的 `tryBind` 并返回一个 DynTypedMatcher。这样我们就实现了目标 3。

``` cpp
class DynTypedMatcher {
  public:
    DynTypedMatcher(IRNodeType supportedNodeType): allowBind(false), supportedNodeType(supportedNodeType) {}
    
    void setAllowBind(bool ab) { allowBind = ab; }
    
    bool canMatchNodesOfKind(IRNodeType nodeType) const {
      return ::ir::isBaseOf(supportedNodeType, nodeType);    
    }
    
    bool matches(Node* node, BoundNodesTreeBuilder* builder) const {
        return canMatchNodesOfKind(node->getType());
    }
    
    optional<DynTypedMatcher> tryBind(const string& id) const {
        ...
    }
    
  private:
    IRNodeType supportedNodeType;
    bool allowBind;
};

class BindableMatcher : public DynTypedMatcher {
  public:
    explicit BindableMatcher(const DynTypedMatcher& m)
        : DynTypedMatcher(m) {}
    explicit BindableMatcher(IRNodeType nodeType,
                             DynMatcherInterface* implementation)
      	: DynTypedMatcher(nodeType, implementation) {}
    
   	DynTypedMatcher bind(const std::string& id) {
    	auto result = DynTypedMatcher(*this);
    	result.setAllowBind(true);
    	auto res = result.tryBind(id);
    	ASSERT(res.has_value(), "Matcher is not bindable.")
    	return res.value();
  	}
}
```

1. Clang 中将 BindableMatcher 同样实现成了一个模板类，继承自 `Matcher<T>`，我们这里为了方便，同样忽略这一部分。
2. Clang 中使用的是 LLVM 中自己实现的 Optional，我们可以使用 C++ 标准库中的版本（注意：这是 C++17！）
{:.error}

下面就要实现目标1了。Clang 中的实现是实现了一个 IdMatcherInterface。它的功能是接受一个 MatcherInterface，匹配时如果成功，则在 Binding 中加入新的变量绑定。

``` cpp
class IdMatcherInterface : public DynMatcherInterface {
  public:
    IdMatcherInterface(const std::string& id,
                       DynMatcherInterface* innerMatcher)
    	: id(id), innerMatcher(innerMatcher) {}
    bool dynMatches(Node* node,
                    BoundNodesTreeBuilder* builder) const override {
        bool result = innerMatcher->dynMatches(node, builder);
        if (result) builder->setbinding(id, node);
        return result;
    }
    
  private:
    std::string id;
    DynMatcherInterface* innerMatcher;
}
```

1. Clang 中使用的名字是 IdDynMatcher，这里为了防止概念混淆，改名为 IdMatcherInterface。
2. Clang 中的成员变量写作 `IntrusiveRefCntPtr<DynMatcherInterface>` ，是带引用计数的，或许可以通过 `std::shared_ptr` 实现，但是我这里偷懒了。
{:.error}

这里的 Id 是带了个“标签”的匹配器，不是函数式里的 `id :: a -> a`。
{:.info}

通过 Id，我们可以实现 tryBind 的功能了：

``` cpp
optional<DynTypedMatcher> tryBind(const string& id) const {
    if (!allowBind) return nullopt;
    DynTypedMatcher result = *this;
    result.implementation = 
        new IdDynMatcherInterface(id, result.implementation);
    return make_optional(result);
}
```

到现在位置，我们的一部分 Matcher 已经可以提供绑定功能了。现在，我们可以休息一下！下一次我们将面临更加复杂的局面。