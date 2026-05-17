## 通过发布配置（Release Profile）自定义构建

在 Rust 中，*发布配置（release profiles）*是预定义的、可自定义的配置，它们具有不同的设置，允许程序员对编译代码的各种选项有更多控制。每个配置都独立于其他配置进行配置。

Cargo 有两个主要配置：当你运行 `cargo build` 时 Cargo 使用的 `dev` 配置，以及当你运行 `cargo build --release` 时 Cargo 使用的 `release` 配置。`dev` 配置为开发定义了良好的默认值，`release` 配置为发布构建定义了良好的默认值。

这些配置名称可能从构建输出中就很熟悉：

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` 和 `release` 是编译器使用的这些不同的配置。

Cargo 对每个配置都有默认设置，当你没有在项目的 _Cargo.toml_ 文件中显式添加任何 `[profile.*]` 节时，这些设置会生效。通过为你想要自定义的任何配置添加 `[profile.*]` 节，你可以覆盖默认设置的任何子集。例如，以下是 `dev` 和 `release` 配置的 `opt-level` 设置的默认值：

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 设置控制 Rust 将应用于代码的优化数量，范围为 0 到 3。应用更多优化会延长编译时间，因此如果你在开发过程中经常编译代码，您希望使用较少的优化以加快编译速度，即使生成的代码运行速度较慢。因此 `dev` 的默认 `opt-level` 是 `0`。当你准备发布代码时，最好花更多时间编译。你只会以发布模式编译一次，但你会多次运行编译后的程序，因此发布模式用更长的编译时间换取运行更快的代码。这就是为什么 `release` 配置的默认 `opt-level` 是 `3`。

你可以通过在 _Cargo.toml_ 中为其添加不同的值来覆盖默认设置。例如，如果我们想在开发配置中使用优化级别 1，我们可以将这两行添加到项目的 _Cargo.toml_ 文件中：

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

此代码覆盖了默认设置 `0`。现在当我们运行 `cargo build` 时，Cargo 将使用 `dev` 配置的默认值以及我们对 `opt-level` 的自定义。因为我们将 `opt-level` 设置为 `1`，Cargo 将应用比默认更多的优化，但不如发布构建那么多。

有关每个配置的配置选项和默认值的完整列表，请参阅 [Cargo 的文档](https://doc.rust-lang.org/cargo/reference/profiles.html)。
