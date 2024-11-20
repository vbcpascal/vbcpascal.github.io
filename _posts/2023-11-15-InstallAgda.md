---
layout: post
title: 配置 Agda 开发环境（2023）
title-cn: 【计算概论】配置 Agda 开发环境（2023）
title-en: "Tutorial: Install Agda (2023, in Chinese)"
categories: [cn]
description: 计算概论扩展文档
keywords: Agda
language: zh
---

<style>
.link-block {
  font-style: normal;
  overflow: auto;
  white-space: nowrap;
  font-family: SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  margin-bottom: 15px;
  padding-left: 10px;
  color: #666;
}
</style>

本文章面向北京大学《计算概论A（实验班）》课程，也适用于其他想要学习 Agda 的朋友们。

## 安装 Agda

我们通过 Cabal 安装 Agda，请确保您已经为 Cabal 换源，由于 Agda 的体量，否则可能产生安装过慢或失败的情况。（注意大写）
```
cabal install Agda
```
这一步会消耗很长时间和很大的内存，请静静等待。安装的最后一步会是将`agda.exe`和`agda-mode.exe`复制到`C:\cabal\bin`文件夹，意味着你可以在命令行运行这两个程序了。

请重新打开一个命令行并输入
```
agda --version
```
没什么神奇的，和以前一样，确保安装成功。这一次，请记住 agda 的版本号，例如在我这里是 2.6.4。

<details>
  <summary>为什么命令行可以找到这两个文件？</summary>
  <p>
  前提是 `C:\cabal\bin` 在你的环境变量 PATH 中，除非你自己操作过，否则安装 Haskell 的过程中这一步是自动完成的。
  </p>
  <p>
  大体而言，环境变量中的 PATH 就是一个导航。在你试图运行一个程序时，系统会去环境变量中的每一个路径里面找一找，看看你试图运行的应用程序是否在这个文件夹中。在 Windows 中，你可以在开始菜单的搜索中输入“PATH”或“环境变量”来打开“系统属性”的窗口，点击“环境变量”就可以看到所有环境变量，PATH 是其中之一。
  </p>
</details>

正确的步骤是现在安装标准库，但是在这里我们稍微调整一下正常的安装顺序，以确保 Agda 本体安装完成。

## 配置 Agda 环境

下面我们将配置 Agda 的编辑器，由于 Agda 中有很多非 ASCII 字符，所以一个辅助输入的工具是必要的。幸运的是，Agda 确实给我们提供了。如果你注意了的话你会发现，安装过程中我们同时安装了 `agda-mode`，这个就是 Agda 自带的 emacs 支持。

我们也可以在 VSCode 中使用它。注意，在学习 Haskell 的时候，我知道很多同学是使用了其它的编辑器。在这里，我推荐大家使用 emacs 或 VSCode，其它编辑器的环境怎么配我也不知道。

在 VSCode 的插件中搜索 Agda，安装`agda-mode`和`language-agda`（安装后可能需要重新启动 VSCode），然后创建`hello.agda`，并输入如下内容：
```
data Greeting : Set where
  hello : Greeting

greet : Greeting
greet = hello
```
然后输入`C-c C-l`，其中`C-c`代表`Ctrl+C`，`C-l`同理。如果下方弹出窗口显示`*All Done*`说明环境配置成功。

## 安装 Agda 标准库

不幸的是，Agda的标准库并不是会在安装Agda中预先安装完成的，我们需要到 https://github.com/agda/agda-stdlib 下载标准库。

不要着急点开上面的链接！我们最好还是到 https://wiki.portal.chalmers.se/agda/Libraries/StandardLibrary 看看和 Agda 版本匹配的标准库版本。
例如我的版本是 Agda 2.6.4，因此我选择 1.7.3 的标准库版本，然后点击 1.7.3（这一步实际上是在 github 上下载的，因此需要一些上网技巧）。

将对应下载后的文件解压缩（mac用户可以搜一下如何解压缩.tar.gz，windows 用户直接解压两次就可以了），把这个文件夹，例如我这里是 `agda-stdlib-1.7.3` 放在任何你喜欢的地方（最好系统也喜欢，所以不要有英文）。记住你当前的路径，例如我这里是 `C:\Users\xxx\Utils\Agda\agda-stdlib-1.7.3`。

下一步，我们要让 Agda 知道标准库在哪里。我们根据系统来找到对应位置：
- UNIX 和 macOS，请打开 `~/.agda`
- Windows，请打开 `%AppData%\agda`（`%AppData%`一般是指`C:\Users\用户名\AppData\Roaming`），你只需要在资源管理器（也就是你随便点开一个文件夹之后的窗口），在地址栏输入这个路径就可以。
如果这个文件夹不存在，请自己创建它。例如 windows 用户请打开路径 `%AppData%`，然后创建 agda 文件夹。

在这个文件夹下创建两个文件分别为`defaults`和`libraries`，注意没有文件扩展名！！注意没有文件扩展名！！注意没有文件扩展名！！不是`defaults.txt`！！如果你看到了一个小记事本的图标，说明是不对的。

说得再清楚一点的话，你可以用VSCode打开这个**文件夹**，然后创建或编辑这两个文件。

`defaults` 文件填入以下内容：
```
standard-library
```

`libraries`文件填入你刚才记住的路径并在末尾加上`\standard-library.agda-lib`，（macOS用户的路径应该都是`/`而非`\`）例如在我这里是
```
C:\Users\xxx\Utils\Agda\agda-stdlib-1.7.3\standard-library.agda-lib
```
保存后，我们的标准库就安装完成了。

下面我们测试一下标准库安装是否成功。创建`bool.agda`，并输入如下内容：
```
module bool where

import Relation.Binary.PropositionalEquality as Eq
open Eq using (_≡_; refl)

data 𝔹 : Set where
  tt : 𝔹
  ff : 𝔹

~_ : 𝔹 → 𝔹
~ tt = ff
~ ff = tt

~~tt : ~ ~ tt ≡ tt
~~tt = refl
```

通过输入`C-c C-l`来保证安装成功。

## Agda-mode 指令

在后续我们会用到各种指令，现在你只需要记住 `C-c C-l` 加载（Load）。

此外，在你输入`\`后，下面会给出一个关于特殊符号的提示框，你可以在其中选择。当然你也可以只通过键盘输入，请试着在编辑器中打出`\->`，`\to`, `\bB`, `\bN`, `\==`, `\>=`。

## 常见问题

1. 在 VSCode 中加载 Agda 代码后出现乱码怎么办？
   
   在 `agda-mode` 的插件设置中关闭 `Highlighting: Get Highlight With Theme Colors`

2. 在 VSCode 中部分字符出现方框怎么办？

   在 VSCode 的设置中关闭 `Unicode Highlight: Non Basic ASCII`
