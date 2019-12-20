---
layout: post
title: 小谈 λ 演算（二）邱奇数
categories: [Theory, Functional]
description: 介绍 λ 演算
keywords: LC, Theory of computation, Functional programming
---
<font color="red">本系列尚未更新完毕！</font>

或许我们经常使用一个相同的 lambda 函数，为此，我们不妨给这些相同的函数命一个名，就像变量赋值一样。

``` haskell
let square = lambda x . x ^ 2
```

事实上，`let` 的定义只是一层语法糖。

在这个例子中，我们使用了数字 2，但 lambda 演算中并不存在所谓的“数字”，但是这样貌似也就无法解决任何数学问题了？事实上，邱奇找到了在 lambda 演算中表示数字的方法，这种表示方法被称为**邱奇数**。

## 邱奇数

我们不妨回忆一下我们在 lambda 演算中有什么。我们有一堆阴阳怪气的表达式，他们是由标识符和其他表达式组成的。换句话说，我们有一个函数 `f` 和参数 `x`，而且我们只有他们。我们什么都做不了啊，我们只能将 `f` 应用到 `x` 上，最多再把结果传给 `f` 依此类推罢了。

事实上，邱奇数就是使用的这种方式。所有的邱奇数都是带有两个参数的函数：

``` haskell
let 0 = lambda f x . x
let 1 = lambda f x . f x
let 2 = lambda f x . f (f x)
...
```

换言之，数字 `n` 就是将函数 `f` 作用在 `x` 上 `n` 次（表示为 $f^n(x)$ ）。这就是自然数的表示方法，接下来我们要使得这种“数字”可以进行相应的运算。

## 邱奇数的运算

### 加法

不妨直接先给出加法的表示 $f^{m+n}(x)=f^m(f^n(x))$ 和定义：

``` haskell
let add = lambda m n f x . m f (n f x)
```

看起来一片混乱……我们需要将其柯里化看看到底发生了什么。不难将其转化为如下的样子：

``` haskell
let add = lambda m n . (lambda f x . (m f (n f x)))
```

现在我们大概看出来了我们试图计算 `m+n` ，`f` 和` x` 只是例行需要表示邱奇数的工具。`(n f x)` 是将 `f` 作用于 `x` 上 `n` 次，表示为 $f^n(x)$，接下来将这个结果作为参数传递给 `f` 并作用 `m` 次。因此，相当于对 `f` 对 `x` 作用了 `m + n` 次，即 $f^{m+n}(x)$。

不妨通过一个例子进行通俗的理解：2 + 3

``` haskell
add (lambda f x . f (f x)) (lambda f x . f (f (f x)))
-- 为了清晰我们进行一次 alpha 变换
add (lambda f2 x2 . f2 (f2 x2)) (lambda f3 x3 . f3 (f3 (f3 x3)))
-- add 定义替换
(lambda f x . ((lambda f2 x2 . f2 (f2 x2)) f ((lambda f3 x3 . f3 (f3 (f3 x3))) f x)))
-- beta 规约
(lambda f x . ((lambda f2 x2 . f2 (f2 x2)) f (f (f (f x))) ))
-- beta 规约             m                  f      f^n(x)
(lambda f x . f (f (f (f (f x)))))
```

也就是5。大功告成！

### 乘法

有了加法的铺垫，乘法看起来就容易很多了 $f^{m*n}(x)=(f^n)^m(x)$

``` haskell
let mul = lambda m n f x . m (n f) x
```

即“将 `f` 作用 `n `次”这个效果整体作用于 `x` 上 `m` 次。详细细节在此不再赘述。

### 乘方

乘方即 $f^{m^n}(x)$，表示为

``` haskell
let exp = lambda m n f x . (n m) f x
```

<font color="blue">以下内容留给有兴趣的读者，不影响后文的阅读。如果理解起来较为困难，不妨读完后续内容后再回来研究。</font>

在讨论减法之前，我们重新回顾一下加法（现在看起来可能非常简单），但是我们尝试使用“后继”的方式表示加法。即将“后继”的效果作用 n 次，就是 +n。

### 后继

显然，就是将 `f` 作用在 $f^n(x)$ 上一次，即

``` haskell
let succ = lambda n f x . f (n f x)
```

由此我们可以重新定义加法，即

``` haskell
let add = lambda m n . (n succ) m
```

### 前驱

接下来我们讨论一下“前驱”。注意：邱奇数只能表示自然数，并且 0 的前驱定义为 0。前驱被定义为 `if (n==0) 0  else (n-1)`。这件事情变得困难很多，直观来看 $f^{n-1}(x) = f^{-1}(f^n(x))$。可是我们应该怎样获得所谓 `f` 的逆呢？

显然我们无法控制 `f` 的样子，更无从有办法获取他的逆。那便换一种思路罢。即然 `n` 是将 `f` 作用在` x` 上 `n` 次，那么我们试图获取 `n - 1`，就是试图在构建这一串的调用时*剥离*其中的一层。

同样先给出 lambda 表达式的写法

``` haskell
let pred = lambda n f x . n (lambda g h . h (g f)) (lambda u . x) (lambda u . u)
```

举例永远是最好的理解方式，我们看看 `(pred 2)` 会得到怎样的结果，注意函数是左结合的：

``` haskell
pred (lambda f2 x2 . f2 (f2 x2))
-- beta (n)
lambda f x . (lambda f2 x2 . f2 (f2 x2)) (lambda g h . h (g f)) (lambda u . x) (lambda u . u) (lambda u . u)
-- beta (f2)
lambda f x . (lambda x2 . (lambda g h . h (g f)) ((lambda g h . h (g f)) x2)) (lambda u . x) (lambda u . u)
-- beta (x2)
lambda f x . (lambda g h . h (g f)) ((lambda g h . h (g f)) (lambda u . x)) (lambda u . u)
-- beta (g) **
lambda f x . (lambda g h . h (g f)) ((lambda h . h ((lambda u . x) f))) (lambda u . u)
-- beta (u) **
lambda f x . (lambda g h . h (g f)) (lambda h . h x) (lambda u . u)
-- alpha 
lambda f x . (lambda g h . h (g f)) (lambda h2 . h2 x) (lambda u . u)
-- beta *
lambda f x . (lambda h . h ((lambda h2 . h2 x) f)) (lambda u . u)
-- beta *
lambda f x . (lambda h . h (f x)) (lambda u . u)
-- beta
lambda f x . ((lambda u . u) (f x))
-- beta
lambda f x . f x
```

注意其中星号的位置，也就是其中“剥离”的操作。注意表达式的最后两个部分：`(lambda u . x)`无论你给我什么我都给你 `x`，`(lambda u . u)` 你给我什么我就给你什么。

### 减法

``` haskell
let sub = lambda m n . (n pred) m
```



## 参考文献

1. https://en.wikipedia.org/wiki/λ_calculus
2. https://cgnail.github.io/academic/λ-1/
3. https://en.wikipedia.org/wiki/β_normal_form
4. *LEARN YOU A HASKELL FOR GREAT GOOD*
