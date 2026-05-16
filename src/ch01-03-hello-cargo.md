## Hello, Cargo!

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustaceans（Rust 开发者）使用这个工具来管理他们的 Rust 项目，因为 Cargo 为你处理了许多任务，例如构建代码、下载代码所依赖的库以及构建这些库。（我们把代码所需要的库称为_依赖项（dependencies）_。）

最简单的 Rust 程序，比如我们目前编写的这个，没有任何依赖项。如果我们用 Cargo 构建"Hello, world!"项目，它只会用到 Cargo 中处理代码构建的部分。随着你编写更复杂的 Rust 程序，你会添加依赖项，而如果你从一开始就使用 Cargo 创建项目，添加依赖项就会变得容易得多。

因为绝大多数 Rust 项目都使用 Cargo，本书的其余部分也假设你使用 Cargo。如果你使用["安装"][installation]<!-- ignore -->章节中讨论的官方安装程序安装了 Rust，那么 Cargo 会随 Rust 一同安装。如果你通过其他方式安装了 Rust，请在终端中输入以下命令来检查 Cargo 是否已安装：

```console
$ cargo --version
```

如果你看到了版本号，说明已经安装成功！如果你看到类似 `command not found` 的错误信息，请查阅你的安装方式对应的文档，了解如何单独安装 Cargo。

### 使用 Cargo 创建项目

让我们使用 Cargo 创建一个新项目，并看看它与我们之前的"Hello, world!"项目有何不同。首先回到你的 _projects_ 目录（或者你决定存放代码的任何位置）。然后，在任何操作系统上运行以下命令：

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

第一个命令创建了一个名为 _hello_cargo_ 的新目录和项目。我们将项目命名为 _hello_cargo_，Cargo 会在同名目录中创建它的文件。

进入 _hello_cargo_ 目录并列出文件。你会看到 Cargo 为我们生成了两个文件和一个目录：一个 _Cargo.toml_ 文件和一个 _src_ 目录，其中包含一个 _main.rs_ 文件。

它同时还初始化了一个新的 Git 仓库以及一个 _.gitignore_ 文件。如果你在已有的 Git 仓库中运行 `cargo new`，则不会生成 Git 文件；你可以通过使用 `cargo new --vcs=git` 来覆盖此行为。

> 注意：Git 是一种常见的版本控制系统。你可以通过 `--vcs` 标志改变 `cargo new` 使用的版本控制系统，或不使用任何版本控制系统。运行 `cargo new --help` 可查看可用选项。

用你选择的文本编辑器打开 _Cargo.toml_。它应该类似于示例 1-2 中的代码。

<Listing number="1-2" file-name="Cargo.toml" caption="`cargo new` 生成的 *Cargo.toml* 内容">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

该文件采用 [_TOML_][toml]<!-- ignore -->（_Tom's Obvious, Minimal Language_）格式，这是 Cargo 的配置格式。

第一行 `[package]` 是一个章节标题，表示接下来的语句是在配置一个包。随着我们向此文件添加更多信息，我们将添加其他章节。

接下来的三行设置了 Cargo 编译程序所需的配置信息：名称（name）、版本（version）和要使用的 Rust 版本（edition）。我们将在[附录 E][appendix-e]<!-- ignore -->中讨论 `edition` 键。

最后一行 `[dependencies]` 是一个章节的开始，用于列出项目的所有依赖项。在 Rust 中，代码包被称为_crate_。这个项目不需要任何其他 crate，但在第 2 章的第一个项目中我们将会用到，到时我们会使用这个 dependencies 章节。

现在打开 _src/main.rs_ 看一看：

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 已经为你生成了一个"Hello, world!"程序，就像我们在示例 1-1 中编写的一样！到目前为止，我们的项目与 Cargo 生成的项目的区别在于：Cargo 将代码放在了 _src_ 目录中，并且我们在顶层目录中有一个 _Cargo.toml_ 配置文件。

Cargo 期望你的源文件存放在 _src_ 目录内。项目顶层目录只存放 README 文件、许可信息、配置文件以及与代码无关的其他内容。使用 Cargo 有助于你组织项目。每样东西都有其位置，所有东西都各就各位。

如果你启动的项目没有使用 Cargo，就像我们之前的"Hello, world!"项目一样，你可以将其转换为使用 Cargo 的项目。将项目代码移到 _src_ 目录中，并创建一个合适的 _Cargo.toml_ 文件。获取 _Cargo.toml_ 文件的一个简单方法是运行 `cargo init`，它会自动为你创建该文件。

### 构建并运行 Cargo 项目

现在让我们看看使用 Cargo 构建和运行"Hello, world!"程序有什么不同！在 _hello_cargo_ 目录下，输入以下命令来构建你的项目：

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

该命令会在 _target/debug/hello_cargo_（Windows 上是 _target\debug\hello_cargo.exe_）中创建一个可执行文件，而不是在当前目录中。因为默认构建是调试构建（debug build），Cargo 将二进制文件放在名为 _debug_ 的目录中。你可以使用以下命令运行该可执行文件：

```console
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

如果一切顺利，`Hello, world!` 将打印到终端。首次运行 `cargo build` 还会使 Cargo 在顶层创建一个新文件：_Cargo.lock_。该文件用于跟踪项目中依赖项的确切版本。这个项目没有依赖项，所以该文件内容比较简单。你永远不需要手动修改这个文件；Cargo 会为你管理其内容。

我们刚刚用 `cargo build` 构建了项目，并用 `./target/debug/hello_cargo` 运行了它，但我们也可以使用 `cargo run` 一次性编译代码并运行生成的可执行文件：

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

使用 `cargo run` 比记住先运行 `cargo build` 再输入完整的二进制文件路径要方便得多，因此大多数开发者使用 `cargo run`。

注意这次我们没有看到 Cargo 正在编译 `hello_cargo` 的输出。Cargo 发现文件没有改变，因此它没有重新构建，而是直接运行了二进制文件。如果你修改了源代码，Cargo 会在运行之前重新构建项目，你会看到如下输出：

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo 还提供了一个名为 `cargo check` 的命令。该命令会快速检查你的代码，确保它能编译，但不会生成可执行文件：

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

为什么你不需要可执行文件呢？通常，`cargo check` 比 `cargo build` 快得多，因为它跳过了生成可执行文件的步骤。如果你在编写代码时持续检查工作，使用 `cargo check` 可以加快你了解项目是否仍能编译的过程！因此，许多 Rustaceans 在编写程序时会定期运行 `cargo check` 以确保代码能编译。然后，当他们准备使用可执行文件时，再运行 `cargo build`。

让我们总结一下目前学到的关于 Cargo 的知识：

- 我们可以使用 `cargo new` 创建项目。
- 我们可以使用 `cargo build` 构建项目。
- 我们可以使用 `cargo run` 一步完成构建和运行项目。
- 我们可以使用 `cargo check` 在不生成二进制文件的情况下检查错误。
- Cargo 不会将构建结果保存在代码所在的目录中，而是存储在 _target/debug_ 目录中。

使用 Cargo 的另一个优点是，无论你使用哪个操作系统，命令都是相同的。因此，从现在开始，我们将不再分别提供 Linux/macOS 与 Windows 的特定说明。

### 构建发布版本

当你的项目最终准备好发布时，你可以使用 `cargo build --release` 在优化模式下编译它。该命令会在 _target/release_ 而不是 _target/debug_ 中创建可执行文件。优化使你的 Rust 代码运行得更快，但启用优化会延长程序编译时间。这就是为什么有两种不同的配置：一种用于开发，当你需要频繁快速重建时使用；另一种用于构建最终提供给用户的程序，该程序不会被反复重建，并且会尽可能快地运行。如果你在基准测试（benchmarking）代码的运行时间，请务必运行 `cargo build --release` 并使用 _target/release_ 中的可执行文件进行测试。

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### 充分利用 Cargo 的约定

对于简单的项目，Cargo 相比直接使用 `rustc` 并没有提供太多优势，但随着程序变得越来越复杂，它会证明自己的价值。一旦程序增长到多个文件或需要依赖项时，让 Cargo 来协调构建会容易得多。

尽管 `hello_cargo` 项目很简单，但它现在使用了你将在 Rust 编程生涯中用到的大部分真实工具。实际上，要处理任何现有项目，你可以使用以下命令通过 Git 检出代码，切换到该项目的目录，然后进行构建：

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

有关 Cargo 的更多信息，请查看[其文档][cargo]。

## 总结

你的 Rust 之旅已经有了一个良好的开端！在本章中，你学会了如何：

- 使用 `rustup` 安装最新稳定版 Rust。
- 更新到更新的 Rust 版本。
- 打开本地安装的文档。
- 直接使用 `rustc` 编写并运行一个"Hello, world!"程序。
- 使用 Cargo 的约定创建并运行一个新项目。

现在是时候构建一个更充实的程序来熟悉阅读和编写 Rust 代码了。因此，在第 2 章中，我们将构建一个猜谜游戏程序。如果你更愿意先学习常见编程概念在 Rust 中的用法，可以阅读第 3 章，然后再回到第 2 章。

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
