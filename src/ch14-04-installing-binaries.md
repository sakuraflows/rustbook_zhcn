<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## 使用 `cargo install` 安装二进制文件

`cargo install` 命令允许你在本地安装和使用二进制 crate。这并非旨在替代系统包管理器；它旨在成为 Rust 开发人员安装其他人已在 [crates.io](https://crates.io/)<!-- ignore --> 上共享的工具的便捷方式。注意，你只能安装具有二进制目标（binary target）的包。*二进制目标*是指如果 crate 有 _src/main.rs_ 文件或其他指定为二进制的文件时所创建的可运行程序，与此相对的是库目标——它本身不可运行，但适合包含在其他程序中。通常，crate 在 README 文件中会说明是库、具有二进制目标，还是两者兼有。

使用 `cargo install` 安装的所有二进制文件都存储在安装根目录的 _bin_ 文件夹中。如果你使用 _rustup.rs_ 安装 Rust 并且没有自定义配置，此目录将是 *$HOME/.cargo/bin*。确保此目录在你的 `$PATH` 中，以便能够运行你使用 `cargo install` 安装的程序。

例如，在第 12 章中我们提到有一个 Rust 实现的 `grep` 工具叫做 `ripgrep`，用于搜索文件。要安装 `ripgrep`，我们可以运行以下命令：

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

输出的倒数第二行显示了已安装二进制文件的位置和名称，对于 `ripgrep` 来说是 `rg`。只要安装目录在你的 `$PATH` 中，如前所述，你就可以运行 `rg --help` 并开始使用一个更快、更 Rust 化的文件搜索工具！
