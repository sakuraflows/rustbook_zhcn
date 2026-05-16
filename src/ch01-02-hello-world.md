## Hello, World!

现在你已经安装了 Rust，是时候编写你的第一个 Rust 程序了。学习一门新语言时，传统上都会编写一个在屏幕上打印 `Hello, world!` 的小程序，我们这里也一样！

> 注意：本书假定你对命令行有基本的了解。Rust 对你的编辑工具、开发工具或代码存放位置没有特别要求，因此如果你更喜欢使用 IDE 而不是命令行，请随意使用你喜欢的 IDE。许多 IDE 现在都已不同程度地支持 Rust；请查看相应 IDE 的文档以了解详情。Rust 团队一直致力于通过 `rust-analyzer` 提供出色的 IDE 支持。更多细节请参见[附录 D][devtools]<!-- ignore -->。

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### 创建项目目录

首先，创建一个目录来存放你的 Rust 代码。Rust 并不关心你的代码存放在哪里，但对于本书中的练习和项目，我们建议在你的 home 目录下创建一个 _projects_ 目录，并将你所有的项目都保存在那里。

打开终端，输入以下命令来创建一个 _projects_ 目录，并在 _projects_ 目录下为"Hello, world!"项目创建一个子目录。

对于 Linux、macOS 和 Windows 上的 PowerShell，请输入：

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

对于 Windows CMD，请输入：

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### Rust 程序基础

接下来，创建一个新的源文件并命名为 _main.rs_。Rust 文件始终以 _.rs_ 扩展名结尾。如果你的文件名包含多个单词，约定使用下划线来分隔它们。例如，使用 _hello_world.rs_ 而不是 _helloworld.rs_。

现在打开你刚刚创建的 _main.rs_ 文件，并输入示例 1-1 中的代码。

<Listing number="1-1" file-name="main.rs" caption="一个打印 `Hello, world!` 的程序">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

保存文件，然后返回到 _~/projects/hello_world_ 目录下的终端窗口。在 Linux 或 macOS 上，输入以下命令来编译并运行该文件：

```console
$ rustc main.rs
$ ./main
Hello, world!
```

在 Windows 上，使用 `.\main` 代替 `./main`：

```powershell
> rustc main.rs
> .\main
Hello, world!
```

无论你使用什么操作系统，字符串 `Hello, world!` 都应该打印到终端上。如果你没有看到这个输出，请返回安装章节的[“故障排除”][troubleshooting]<!-- ignore -->部分，了解获取帮助的方法。

如果 `Hello, world!` 成功打印出来了，恭喜你！你已经正式编写了一个 Rust 程序。这意味着你已经是一名 Rust 程序员了——欢迎你！

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### Rust 程序剖析

让我们仔细回顾一下这个"Hello, world!"程序。以下是代码的第一部分：

```rust
fn main() {

}
```

这些代码定义了一个名为 `main` 的函数。`main` 函数很特殊：它始终是每个可执行 Rust 程序中第一个运行的代码。这里，第一行声明了一个名为 `main` 的函数，它没有参数，也没有返回值。如果有参数，它们会放在括号 `()` 内。

函数体被包裹在 `{}` 中。Rust 要求所有函数体都要用花括号括起来。好的代码风格是将开头的花括号放在函数声明的同一行，并在中间加一个空格。

> 注意：如果你想在 Rust 项目中保持统一的代码风格，可以使用名为 `rustfmt` 的自动格式化工具来按特定风格格式化代码（更多关于 `rustfmt` 的内容请参见[附录 D][devtools]<!-- ignore -->）。Rust 团队已将该工具随标准 Rust 发行版一同提供，就像 `rustc` 一样，因此它应该已经安装在了你的电脑上！

`main` 函数体包含以下代码：

```rust
println!("Hello, world!");
```

这一行代码完成了这个小程序中的所有工作：它将文本打印到屏幕上。这里有三个重要的细节需要注意。

第一，`println!` 调用了一个 Rust 宏（macro）。如果它调用的是一个普通函数，那么应该写成 `println`（不带 `!`）。Rust 宏是一种编写生成代码以扩展 Rust 语法的方式，我们将在[第 20 章][ch20-macros]<!-- ignore -->中更详细地讨论它们。目前，你只需要知道使用 `!` 意味着你在调用一个宏而不是普通函数，并且宏并不总是遵循与函数相同的规则。

第二，你看到了 `"Hello, world!"` 字符串。我们将这个字符串作为参数传递给 `println!`，然后该字符串被打印到屏幕上。

第三，我们用分号（`;`）结束这一行，这表明这个表达式已经结束，下一个表达式可以开始了。Rust 代码的大多数行都以分号结尾。

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### 编译与执行

你刚刚运行了一个新创建的程序，现在让我们回顾一下这个过程的具体步骤。

在运行 Rust 程序之前，你必须使用 Rust 编译器进行编译，输入 `rustc` 命令并将源文件名作为参数传入，如下所示：

```console
$ rustc main.rs
```

如果你有 C 或 C++ 背景，你会注意到这与 `gcc` 或 `clang` 类似。编译成功后，Rust 会输出一个二进制可执行文件。

在 Linux、macOS 和 Windows 的 PowerShell 上，可以通过在终端中输入 `ls` 命令来查看可执行文件：

```console
$ ls
main  main.rs
```

在 Linux 和 macOS 上，你会看到两个文件。在 Windows 的 PowerShell 上，你会看到与使用 CMD 时相同的三个文件。在 Windows 的 CMD 上，需要输入以下命令：

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

这里显示了扩展名为 _.rs_ 的源代码文件、可执行文件（在 Windows 上是 _main.exe_，在其他平台上则是 _main_），以及在 Windows 上还有一个包含调试信息的 _.pdb_ 文件。然后，你可以运行 _main_ 或 _main.exe_ 文件，如下所示：

```console
$ ./main # or .\main on Windows
```

如果你的 _main.rs_ 是"Hello, world!"程序，这行命令会将 `Hello, world!` 打印到终端上。

如果你更熟悉动态语言，比如 Ruby、Python 或 JavaScript，你可能不习惯将编译和运行分成两个步骤。Rust 是一种_预编译（ahead-of-time compiled）_语言，这意味着你可以编译程序并将可执行文件交给别人，即使他们没有安装 Rust 也能运行。如果你给别人一个 _.rb_、_.py_ 或 _.js_ 文件，他们需要分别安装 Ruby、Python 或 JavaScript 的运行时环境。不过在这些语言中，你只需要一条命令就能编译并运行程序。语言设计中处处存在权衡。

对于简单的程序，仅用 `rustc` 编译就足够了，但随着项目的增长，你会希望管理所有选项并方便地共享代码。接下来，我们将向你介绍 Cargo 工具，它将帮助你编写真实的 Rust 程序。

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
