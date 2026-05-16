## 安装

第一步是安装 Rust。我们将通过 `rustup`（一个用于管理 Rust 版本及相关工具的命令行工具）来下载 Rust。你需要联网才能下载。

> 注意：如果你出于某些原因不想使用 `rustup`，请参阅
> [其他 Rust 安装方法页面][otherinstall] 了解更多选项。

以下步骤将安装最新稳定版的 Rust 编译器。
Rust 的稳定性保证确保本书中所有能编译的示例在更新的 Rust 版本中也能继续编译。不同版本之间的输出可能略有差异，因为 Rust 经常改进错误信息和警告。换句话说，你通过这些步骤安装的任何较新的稳定版 Rust 都应该能正常使用本书的内容。

> ### 命令行符号说明
>
> 在本章及全书范围内，我们会展示一些在终端中使用的命令。你需要在终端中输入的行都以 `$` 开头。你不需要输入 `$` 字符；它只是命令行提示符，用于指示每一条命令的开始。不以 `$` 开头的行通常显示上一条命令的输出。此外，特定于 PowerShell 的示例将使用 `>` 而不是 `$`。

### 在 Linux 或 macOS 上安装 `rustup`

如果你使用的是 Linux 或 macOS，打开终端并输入以下命令：

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

该命令会下载一个脚本并开始安装 `rustup` 工具，该工具会安装最新稳定版的 Rust。系统可能会提示你输入密码。如果安装成功，将显示以下内容：

```text
Rust is installed now. Great!
```

你还需要一个 _链接器（linker）_，它是 Rust 用来将其编译输出合并成一个文件的程序。你可能已经拥有一个。如果遇到链接器错误，你应该安装一个 C 编译器，它通常会包含链接器。C 编译器也很有用，因为一些常见的 Rust 包依赖 C 代码，因此需要 C 编译器。

在 macOS 上，你可以通过运行以下命令来获取 C 编译器：

```console
$ xcode-select --install
```

Linux 用户通常应根据其发行版的文档安装 GCC 或 Clang。例如，如果你使用 Ubuntu，可以安装 `build-essential` 包。

### 在 Windows 上安装 `rustup`

在 Windows 上，请访问 [https://www.rust-lang.org/tools/install][install]<!-- ignore
--> 并按照说明安装 Rust。在安装过程中，系统会提示你安装 Visual Studio。这提供了编译程序所需的链接器和本地库。如果你在此步骤中需要更多帮助，请参阅
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore -->。

本书的其余部分使用的命令在 _cmd.exe_ 和 PowerShell 中都可运行。如果有特定差异，我们会说明应该使用哪一种。

### 故障排除

要检查 Rust 是否正确安装，打开终端并输入以下命令：

```console
$ rustc --version
```

你应该会看到最新稳定版发布的版本号、提交哈希值（commit hash）和提交日期（commit date），格式如下：

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

如果你看到了这些信息，说明 Rust 已成功安装！如果没有看到这些信息，请按照以下步骤检查 Rust 是否在你的 `%PATH%` 系统变量中。

在 Windows CMD 中，使用：

```console
> echo %PATH%
```

在 PowerShell 中，使用：

```powershell
> echo $env:Path
```

在 Linux 和 macOS 中，使用：

```console
$ echo $PATH
```

如果这些都没问题但 Rust 仍然无法工作，你可以在多个地方获得帮助。请访问[社区页面][community]了解如何与其他 Rustaceans（我们对自已的一个昵称）取得联系。

### 更新与卸载

一旦通过 `rustup` 安装了 Rust，更新到新发布的版本就很容易了。在终端中运行以下更新脚本：

```console
$ rustup update
```

要卸载 Rust 和 `rustup`，请在终端中运行以下卸载脚本：

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### 阅读本地文档

安装 Rust 时也会附带一份本地文档副本，方便你离线阅读。运行 `rustup doc` 即可在浏览器中打开本地文档。

每当标准库提供了某个类型或函数，而你不确定它的用途或使用方法时，可以查阅应用程序编程接口（API）文档来了解！

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### 使用文本编辑器和 IDE

本书对你使用什么工具来编写 Rust 代码没有任何预设。几乎任何文本编辑器都能胜任！不过，许多文本编辑器和集成开发环境（IDE）都内置了对 Rust 的支持。你可以在 Rust 官网的[工具页面][tools]上找到一份相当最新的编辑器及 IDE 列表。

### 离线使用本书

在几个示例中，我们会用到标准库之外的 Rust 包。要完成这些示例，你需要联网或提前下载这些依赖项。要提前下载依赖项，可以运行以下命令（我们稍后会详细解释 `cargo` 是什么以及这些命令各自的作用）：

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

这将缓存这些包的下载内容，这样你就不需要之后再下载了。运行此命令后，你不需要保留 `get-dependencies` 文件夹。如果你已经运行过此命令，可以在本书后续的所有 `cargo` 命令中使用 `--offline` 标志，以使用这些缓存版本，而无需尝试联网。

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
