---
layout: post
title: 配置 Haskell 开发环境（2023）
title-cn: 【计算概论】配置 Haskell 开发环境（2023）
title-en: "Tutorial: Install Haskell (2023, in Chinese)"
categories: [cn]
description: 计算概论扩展文档
keywords: Haskell
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

本文章面向北京大学《计算概论A（实验班）》课程，也适用于其他想要学习 Haskell 的朋友们。

> 说明：这并不是一篇纯粹的安装指南，其中会包含大量的废话以及对大一的朋友们的忠告。
> **配置环境**将是一直伴随你们的梦魇，本文的目的除了告诉你们怎么做以外，还会讨论**为什么**要这么做。
> 更纯净的安装指南可以参考去年助教的文章[配置Haskell开发环境（2022）](https://zhuanlan.zhihu.com/p/559892399)，
> 请注意时效性问题。

我们将要逐一完成下面的任务：
- 安装 GHCup（和 MSys2）
- 使用 GHCup 安装 GHC 和其它工具链
- 安装 VSCode 并配置 Haskell 开发环境

需要注意的是，如果你涉及如下情况，你的安装将可能是失败的：
- 用手机安装 Haskell
- 所在地没有网络
- **你的用户名是中文**

<details>
  <summary>为什么用户名不能是中文？</summary>
  <p>
  这与字符编码等问题相关。很多专业软件对路径的读取是不支持中文的，这就好比我们写一个软件也不会优先考虑阿拉伯语的用户一样。
  例如，在Windows中，如果你使用一个中文的用户名如“王富贵”，那么你的个人用户的路径将是“C:\Users\王富贵\”，
  那么软件在读取你的个人目录（以保存应用配置等文件）时就会出错。
  </p>
  <p>
  除此之外，安装软件时也尽可能选择英文路径。推荐的安装路径如“D:\Software”，不推荐的安装路径如“D:\软件”（在安装Python时可能会遇到这个问题）
  一些炫酷的用户名如“꧁༺ᵒᯅᵒ༻꧂”也不建议使用，出了问题只能折磨自己。
  </p>
</details>

## 安装 GHCup

[GHCup](https://www.haskell.org/ghcup/) 是一款 [Haskell](https://www.haskell.org/) 安装工具，
可以安装特定版本的 GHC、stack、cabal、HLS，并可以帮助管理多个 Haskell 版本共存的环境。
我们按照官网给出的方法安装 GHCup，并使用 GHCup 完成后续的安装。

如果你是 Linux、macOS、FreeBSD 或 WSL，执行如下命令：
```
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
```
如果你是 Windows，执行如下命令：
```
Set-ExecutionPolicy Bypass -Scope Process -Force;[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; try { Invoke-Command -ScriptBlock ([ScriptBlock]::Create((Invoke-WebRequest https://www.haskell.org/ghcup/sh/bootstrap-haskell.ps1 -UseBasicParsing))) -ArgumentList $true } catch { Write-Error $_ }
```
建议打开[这个链接](https://www.haskell.org/ghcup/)点击页面右侧的按钮直接复制命令。请**不要**以管理员身份执行上述命令。

<details>
  <summary>我应该在哪里执行上面的命令？</summary>
  <p>
  如果你是 macOS，通过“Command + 空格”聚焦搜索，输入“终端”或“Terminal”，然后打开；
  如果你是 Windows，点击开始旁的搜索，输入“powershell”，注意<b>不要</b>以管理员的方式运行。
  你将得到一个朴素的经常出现在电影里的窗口，复制上面的命令并回车执行。
  如果你是 macOS，通过“Command + V”进行粘贴，如果你是 Windows，在黑色窗口任何位置右键即可粘贴。 
  </p>
  <p>
  此外，macOS 用户可以考虑安装 iTerm，Windows 用户建议安装 PowerShell 7，并可以考虑安装 Windows Terminal。它们的安装不在本教程中涉及。
  </p>
</details>

<details>
  <summary>我看到其他教程没有使用 GHCup？</summary>
  <p>
  在阅读网上的教程时，<b>一定要注意文章的时效性</b>，包括你正在阅读的这篇文章！
  如果你搜到了2017年左右的文章，它会推荐你使用 Haskell platform；
  如果你搜到了2020年左右的文章，<a href="https://krantz-xrf.github.io/2020/09/25/windows-install-stack-ghc.html">它</a>可能推荐你使用 stack 安装 Haskell；
  而2023年，使用 GHCup 则是最好的方法。
  </p>
  <p>
  当然 GHCup 不是唯一安装 Haskell 的方法，在此不再赘述使用 GHCup 的好处。
  </p>
</details>

在你执行命令后，你应当看到如下的内容：

![](/assets/images/InstallHaskell/1.png)

接下来理想情况下，我们应该跟随 GHCup 的脚步完成 Haskell 的整个安装：
1. 安装 GHCup 自身；
2. 安装 msys2（仅 Windows 用户）；
3. 安装 Haskell 相关工具链。

当前命令行等待你提供一个 GHCup 的安装路径，默认为`C:\`，输入回车即可。
**建议使用默认的路径**。如果一定要自选路径，那么我希望你知道自己在做什么以及出了问题后应该怎么做，且路径不要有中文。

之后会让你提供 cabal 的安装路径，我们依旧选取默认路径。
而后安装程序会询问你是否要安装 HLS（默认为不安装），我们输入`Y`并回车表示安装；
stack 和 MSys2 同理选择安装。

### 安装 GHCup 和 MSys2

而后我们可以看到下载进度，首先会下载并安装 GHCup 和 MSys2：

![](/assets/images/InstallHaskell/2.png)

安装完成后，GHCup 会试图进行后续的安装，即安装上文所说的 cabal 等等。
（如果你是 Windows，那么会弹出一个新的窗口。）
考虑到国内的网络情况，我们在这一步的安装过程中大概率发生如下错误。
其大致含义就是在下载 `https://raw.githubusercontent.com/haskell/...` 这个文件时失败了。
下面我们要进行**换源**。

![](/assets/images/InstallHaskell/4.png)

在换源之前我们需要首先保证我们的 GHCup 已经安装成功了。
**重新打开**一个终端，输入 `ghcup --version` 并回车，如下给出 `ghcup` 的版本号则是成功了。（`$`符号表示提示符，后面的文本是需要输入的命令，不以`$`开头的行是程序显示的输出）

```
$ ghcup --version
The GHCup Haskell installer, version 0.1.19.4
```

<details class="alert">
  <summary>在这一步下载失败了应该怎么办？</summary>
  <p>
  输入上面的命令重试 ^_^（你在期待什么）。在命令行中，你可以按“上方向键”来快速地获取你的上一条命令。
  </p>
</details>

<details>
  <summary>这句命令是什么意思？</summary>
  <p>
  就像我们在图形界面里双击运行程序一样，在命令行运行程序就要输入它的名字（PATH 相关问题不在此展开）。
  <code>--version</code> 是程序运行的参数，由程序自身定义并处理。
  这里的参数告诉 ghcup 我要查看你的版本，ghcup 返回输出结果：0.1.19.4。
  </p><p>
  你可以通过输入 <code>ghcup --help</code> 来查看 GHCup 的使用方式，以及各个参数的含义。
  注意并不是所有参数都以 <code>--</code> 开头。
  例如在后文中我们会使用到 <code>ghcup install ...</code> 这样的命令。
  <b>一般而言</b>，不以 <code>--</code> 开头的代表程序的主要功能；反之可以看作是程序运行的选项。
  </p><p>
  如果你输入了一些程序无法处理的参数，例如运行 <code>ghcup pku</code>，
  它会输出 <code>Invalid argument `pku'</code>。
  最后再次注意，如何处理参数完全是由程序自己决定的，
  例如你也可以写个程序要求参数必须是 <code>pku>thu</code>，这里说的只是一般的惯例。
  </p>
</details>


### 给 GHCup 换源

<details>
  <summary>什么是换源？</summary>
  <p>
  简单来说，<b>源</b>是在你想要找什么东西的时候<b>去哪找</b>。
  但是这个地方你往返可能很不方便，速度很慢，下载东西非常折磨。
  <b>镜像站</b>就是其他人把源的东西拷贝了一份，你本来想找的东西在镜像站也能找到了，
  东西是一样的，你选择一个你最方便的镜像站找就可以了，也就是<b>换源</b>。
  </p>
</details>

我们这里推荐使用[中科大的源](https://mirrors.ustc.edu.cn/help/ghcup.html)。
Windows 用户编辑 `C:\ghcup\config.yaml` 文件（用记事本打开即可）；
macOS 用户编辑 `~/.ghcup/config.yaml`（前往个人文件夹，如果其中没有 `.ghcup`，需要首先在文件夹中点击 `Command+Shift+。` 以显示隐藏文件和文件夹）。
内容如图所示：

![](/assets/images/InstallHaskell/5.png)

在最后添加如下内容并保存：

``` yaml
url-source:
  OwnSource: https://mirrors.ustc.edu.cn/ghcup/ghcup-metadata/ghcup-0.0.7.yaml
```
![](/assets/images/InstallHaskell/7.png)

## 安装 Haskell 相关工具链

重新在终端中执行命令 `ghcup list`，可以看到如下输出（部分内容被截去）：

![](/assets/images/InstallHaskell/8.png)

这里列出了所有可用的版本，左侧为“X”代表未安装，“I”代表已安装，“IS”代表已安装并作为默认工具链版本。
右侧为 recommended 代表推荐安装的版本，lastest 代表最后一个版本。
我们在这里安装推荐版本。**请所有同学在此安装推荐版本。**
逐一执行下面的四条命令（其他读者请注意版本号的时效性），同时我会简单介绍我们安装的是什么。

1. 安装 GHC。GHC（Glasgow Haskell Compiler）是 Haskell 最主流的编译器。
```
ghcup install ghc 9.2.8
```
1. 安装 cabal。cabal 是 Haskell 的一款包管理器。
```
ghcup install cabal 3.6.2.0
```
1. 安装 stack。stack 也是 Haskell 的一款包管理器。
```
ghcup install stack 2.9.3
```
1. 安装 HLS。HLS（Haskell Language Server）提供了 IDE 支持，方便程序员的开发。
```
ghcup install hls 2.2.0.0
```

![](/assets/images/InstallHaskell/10.png)

<details>
  <summary>什么是编译器？Haskell 只有这一种编译器吗？</summary>
  <p>
  简单来说，编译器就是将你写的程序变为计算机能够识别的二进制格式。
  </p><p>
  你可能知道 C++ 常用的编译器有很多，如 G++、Clang、MSVC 等。
  Haskell 当然也有其它的编译器，如 JHC、UHC。然而 GHC 是最广泛使用的。
  我们建议所有人使用 GHC。
  </p>
</details>

<details>
  <summary>什么是包管理器？</summary>
  <p>
  包管理器是一种工具，允许你安装和管理其他库和依赖。
  包/库是其他人写好的实现了某种功能的代码，你可以直接复用它而不需要自己从头实现。
  然而一个包可能依赖其他特定版本的包。包管理器就可以帮助我们处理好这些版本和依赖关系。
  </p>
</details>

<details>
  <summary>cabal 和 stack 的区别是什么？我都需要安装吗？</summary>
  <p>
  cabal 和 stack 都是 Haskell 常用的包管理工具，这也就意味着必然有一些关于 cabal 和 stack 哪个更好的争论。
  </p><p>
  首先 cabal 可以帮助你配置和发布 Haskell 项目，这就像 Python 中的 pip、JavaScript 中的 npm、Rust 中的 cargo。cabal 可以提供依赖解析的功能，但是如果出现复杂依赖情况（菱形依赖）就会导致错误，这种情况一般被称为“cabal 地狱”。
  </p><p>
  stack 实际上是依赖于 cabal 的，stack 会在 build 过程中生成依赖项的固定版本，并避免了菱形依赖的问题，
  这意味着你总是可以对旧的项目进行复现。此外，stack 可以管理多个 GHC 版本并总是可以使用正确的版本。
  </p><p>
  注意看，stack的一个重要的好处是<b>可以管理多个 GHC 版本</b>。
  然后你就会惊奇的发现这件事情 GHCup 也可以做啊！
  是这样的，如果是在两年前，我会毫不犹豫地推荐 stack。
  如今，cabal 修改了解析依赖的方法，cabal + GHCup 的管理方法在社区中反馈也较为积极。
  因此我无法给出哪种更好的回答。
  </p><p>
  当然，完成大型 Haskell 项目对于还在安装 Haskell 的朋友们来说还是太早了。
  由于在课上我们会使用到 stack，因此笔者建议安装 GHCup、cabal、stack 全家桶。
  </p>
</details>

<details class="alert">
  <summary>在其中的一步下载失败了应该怎么办？</summary>
  <p>
  重试这一步的命令 ^_^（你在期待什么）。在命令行中，你可以按“上方向键”来快速地获取你的上一条命令。
  </p>
</details>

在安装好上面的工具链之后，我们可以重新使用 `ghcup list` 来查看安装情况，在我这里的情况如下所示：
![](/assets/images/InstallHaskell/11.png)

**设置默认工具链版本**。我们将上面安装的版本都设为默认，在上图中 cabal 3.6.2.0 左侧为“IS”，代表为默认版本；
而 ghc 9.2.8 左侧为“I”，代表仅安装，而未被设置为默认版本。我们通过以下命令完成：
```
ghcup set ghc 9.2.8
```
cabal、stack、HLS 类似。

### 测试上述工具安装情况

如同我们在上文中检查 GHCup，我们**重新打开**一个终端，并逐一下面的四条命令。
再次注意，`$` 后为输入的命令。

```
$ ghc --version
The Glorious Glasgow Haskell Compilation System, version 9.2.8
$ cabal --version
cabal-install version 3.6.2.0
compiled using version 3.6.2.0 of the Cabal library
$ stack --version
Version 2.9.3, Git revision ...
```

我们也可以输入 `ghci` 命令来进行交互式运行 Haskell 程序。
如下图所示，红框圈起来的是我们的输入。GHCi 的使用我们将在后续介绍。

![](/assets/images/InstallHaskell/9.png)

## 安装并配置编辑器

安装 [VSCode](https://code.visualstudio.com/) 
以及名为 *Haskell* 的官方扩展（这个扩展会使用前面安装的HLS）。不推荐安装其它 Haskell 和 GHC 相关的扩展
（*Haskell Syntax Highlighting* 除外，它会在你安装 *Haskell* 的时候自动安装）。
如有需求，可以安装*Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code*扩展。

**不建议使用 code-runner 扩展来运行 Haskell 程序**。其对 Haskell 的支持并不好，作为给 C/C++/Python 玩家的忠告。

## 写在最后

到此为止，Haskell 及其开发环境到此就全部完成了。希望读者不要机械地完成上面的指令，而是尝试理解一下每一步是在做什么。
本教程面向无计算机基础以及只接触过 OI 的朋友们，因此讨论了很多基础概念。如果有什么依旧难以理解或有错误的地方，欢迎提出指正。如果在安装过程中遇到任何问题，可以随时在课程群或私聊中反馈。希望你这一次的环境配置一切顺利。
