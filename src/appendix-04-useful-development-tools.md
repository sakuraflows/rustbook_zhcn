## 附录 D：有用的开发工具（Useful Development Tools）

在本附录中，我们将讨论 Rust 项目提供的一些有用的开发工具。我们将了解自动格式化、快速应用警告修复、一个 linter（代码检查工具）以及与 IDE 的集成。

### 使用 `rustfmt` 进行自动格式化（Automatic Formatting）

`rustfmt` 工具会根据社区代码风格重新格式化你的代码。许多协作项目使用 `rustfmt` 来避免在编写 Rust 时关于使用哪种风格的争论：每个人都使用该工具格式化他们的代码。

Rust 安装默认包含 `rustfmt`，因此你的系统上应该已经有了 `rustfmt` 和 `cargo-fmt` 程序。这两个命令类似于 `rustc` 和 `cargo` 的关系：`rustfmt` 提供更精细的控制，而 `cargo-fmt` 则理解使用 Cargo 的项目的惯例。要格式化任何 Cargo 项目，请输入以下命令：

```console
$ cargo fmt
```

运行此命令会重新格式化当前 crate 中的所有 Rust 代码。这只会改变代码风格，不会改变代码语义。有关 `rustfmt` 的更多信息，请参见[其文档][rustfmt]。

### 使用 `rustfix` 修复代码（Fix Your Code）

`rustfix` 工具包含在 Rust 安装中，可以自动修复那些有明确修正方案的编译器警告（通常正是你想要的）。你可能之前见过编译器警告。例如，考虑以下代码：

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

这里，我们将变量 `x` 定义为可变的（mutable），但从未真正修改它。Rust 对此发出警告：

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

警告建议我们移除 `mut` 关键字。我们可以使用 `rustfix` 工具通过运行 `cargo fix` 命令来自动应用该建议：

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

当我们再次查看 _src/main.rs_ 时，会发现 `cargo fix` 已经修改了代码：

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

现在变量 `x` 是不可变的（immutable），警告也不再出现。

你还可以使用 `cargo fix` 命令在不同 Rust 版本（editions）之间迁移代码。版本相关内容在[附录 E][editions]<!-- ignore --> 中介绍。

### 使用 Clippy 获得更多 Lint（More Lints with Clippy）

Clippy 工具是一组 lint（代码检查规则）的集合，用于分析你的代码，帮助你发现常见错误并改进 Rust 代码。Clippy 包含在标准 Rust 安装中。

要在任何 Cargo 项目上运行 Clippy 的 lint，请输入以下命令：

```console
$ cargo clippy
```

例如，假设你编写了一个程序，使用了数学常量的近似值，比如圆周率 pi：

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

在此项目上运行 `cargo clippy` 会导致以下错误：

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

这个错误让你知道 Rust 已经定义了一个更精确的 `PI` 常量，如果你的程序使用该常量将会更准确。然后你应该修改代码来使用 `PI` 常量。

以下代码不会导致 Clippy 产生任何错误或警告：

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

有关 Clippy 的更多信息，请参见[其文档][clippy]。

### 使用 `rust-analyzer` 进行 IDE 集成（IDE Integration）

为了帮助实现 IDE 集成，Rust 社区推荐使用 [`rust-analyzer`][rust-analyzer]<!-- ignore -->。这个工具是一组以编译器为中心的实用工具，实现了[语言服务器协议（Language Server Protocol，LSP）][lsp]<!-- ignore -->——这是一个用于 IDE 和编程语言之间相互通信的规范。不同的客户端可以使用 `rust-analyzer`，例如 [Visual Studio Code 的 Rust analyzer 插件][vscode]。

请访问 `rust-analyzer` 项目的[主页][rust-analyzer]<!-- ignore -->获取安装说明，然后在你的特定 IDE 中安装语言服务器支持。你的 IDE 将获得诸如自动补全（autocompletion）、跳转到定义（jump to definition）和内联错误提示（inline errors）等功能。

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
