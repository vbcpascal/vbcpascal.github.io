---
layout: page
title: 关于
description: vbcpascal的个人博客
keywords: vbcpascal, 码后炮
comments: true
menu: 关于
permalink: /about/
---

你好，这里其实是一个不太正式的个人主页~~，主要原因是我并没有什么 Paper~~。为了这里看起来不是那么的空旷，我将安排一些比较随便的内容……
{:.mycomment}

E-mail: [vbcpascal@outlook.com](#)
### 码后炮——打码后的一场狂欢

古语有云：“博学而不自反，必有邪。”。“码后炮”看似是写码前的考虑不周，但更是对一个项目的反思。*写码是在写诗*，无论是目标、方法、语言的选择、库的使用，甚至是具体的实现方法，每个环节的都可能让这首诗变成拗口的小学生作文。反思才能让“码”更加丰满。这是打码后的一场狂欢。~~更何况“打码后”和“大马猴”同音。~~

### 看一些我的开源项目？

**[DepKit](https://github.com/vbcpascal/depkit)**

一门简单但是强大的包管理语言！给出环境中的包版本号以及相互依赖关系，自动求解出可行的安装解决方案。同时支持特定的 feature 等高级的语义（类似于 Gentoo 的 USEFLAG）。你可以在[这里](https://github.com/vbcpascal/depkit/blob/master/examples/example1.dk)看到一个示例的程序。Definitions 定义了环境中的所有包（LLVM 和 z3），Dependencies 定义了包之间的依赖关系，Requirements 指定了系统中需要什么包。你将会得到：

```
LLVM 10.0
  features: z3 [ gold ]
  backends: [ AArch-64 RISC-V ]
z3 4.8
  features: [ ]
  backends: [ AArch-64 RISC-V ]
```

**[SOFIR](https://github.com/vbcpascal/SOFIR)**

想实现一门编译型语言，但是又觉得 LLVM 的语义过于低级？SOFIR（Simple Object-Oriented and Functional IR）为你提供了一些中间层次的 IR，你可以直接使用其中的面向对象，或是函数式的特性。无论你想实现一个简单的 Java，或者一个阉割的 Haskell，这都是一个好的选择。

**[Libel](https://github.com/vbcpascal/Libel)**

蜻蜓协议栈——一个由 C++ 编写的用户态的网络协议栈。基于 PCAP，根据 RFC-793 标准实现了 TCP-IP 的基础功能，能够与内核进行通信。其中包含了一个简易的路由算法 [SDP](https://github.com/vbcpascal/Libel/blob/master/document/writing-task.pdf)。协议栈之间的通信测试如下：

![](/images/about/Libel.png)

**[探索更多](https://github.com/vbcpascal)**


去找一些更有趣的开源项目吧！

### 关键词
<div class="btn-inline">
{% for keyword in site.data.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
