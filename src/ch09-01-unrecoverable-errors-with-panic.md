## 使用 `panic!` 的不可恢复错误

有时候你的代码中会发生糟糕的事情，而你对此无能为力。在这些情况下，Rust 提供了 `panic!` 宏。实际上有两种方式导致 panic：执行一个会导致我们的代码 panic 的操作（例如访问数组末尾之后的位置），或者显式调用 `panic!` 宏。在这两种情况下，我们都会导致程序中出现 panic。默认情况下，这些 panic 会打印一条失败消息，展开（unwind）、清理栈并退出。通过环境变量，你也可以让 Rust 在发生 panic 时显示调用栈，以便更容易地追踪 panic 的来源。

> ### 关于 panic 时的栈展开与终止
>
> 默认情况下，当 panic 发生时，程序开始*展开（unwinding）*，这意味着 Rust 会回退栈并清理它遇到的每个函数中的数据。然而，回退和清理是大量的工作。因此，Rust 允许你选择另一种方式：立即*终止（aborting）*，即不进行清理就结束程序。
>
> 程序使用的内存随后需要由操作系统来清理。如果在你的项目中你希望使生成的二进制文件尽可能小，你可以通过在 _Cargo.toml_ 文件的适当 `[profile]` 部分添加 `panic = 'abort'`，来让 panic 时从展开切换为终止。例如，如果你希望在发布模式下 panic 时终止，请添加以下内容：
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

让我们在一个简单的程序中尝试调用 `panic!`：

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

当你运行这个程序时，你会看到类似这样的输出：

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

对 `panic!` 的调用导致了最后两行中包含的错误消息。第一行显示了我们的 panic 消息以及 panic 发生位置在源代码中的位置：_src/main.rs:2:5_ 表示它是 _src/main.rs_ 文件的第二行、第五个字符。

在这种情况下，所指出的行是我们代码的一部分，如果去查看那一行，我们会看到 `panic!` 宏调用。在其他情况下，`panic!` 调用可能在我们代码所调用的代码中，错误消息报告的文件名和行号将是调用 `panic!` 宏的其他人的代码，而不是最终导致 `panic!` 调用的我们自己的代码行。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

我们可以使用 `panic!` 调用来源函数的回溯（backtrace）来找出导致问题的代码部分。为了理解如何使用 `panic!` 回溯，让我们来看另一个例子，看看当 `panic!` 调用来自库（由于我们代码中的 bug）而不是我们的代码直接调用宏时是什么样子。示例 9-1 中的代码尝试访问向量中超出有效索引范围的索引。

<Listing number="9-1" file-name="src/main.rs" caption="尝试访问向量末尾之后的元素，这将导致对 `panic!` 的调用">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

在这里，我们试图访问向量的第 100 个元素（因为索引从零开始，实际上是索引 99），但向量只有三个元素。在这种情况下，Rust 会 panic。使用 `[]` 应该返回一个元素，但如果你传递了一个无效的索引，Rust 无法返回任何正确的元素。

在 C 中，尝试读取数据结构末尾之后的内存是未定义行为（undefined behavior）。你可能会得到内存中对应于该数据结构中该元素位置的值，即使该内存不属于该数据结构。这被称为*缓冲区过度读取（buffer overread）*，如果攻击者能够操纵索引以读取存储在该数据结构之后的不应被允许读取的数据，则可能导致安全漏洞。

为了保护你的程序免受此类漏洞的侵害，如果你尝试读取不存在的索引处的元素，Rust 将停止执行并拒绝继续。让我们试一下看看：

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

这个错误指向了 _main.rs_ 的第 4 行，我们在那里尝试访问 `v` 中索引为 99 的元素。

`note:` 这一行告诉我们，我们可以设置 `RUST_BACKTRACE` 环境变量来获取一个回溯，精确地追踪到导致错误的原因。*回溯（backtrace）* 是一个列表，列出了到达此点所调用的所有函数。Rust 中的回溯与其他语言中的工作方式相同：阅读回溯的关键是从顶部开始，一直读到你看自己所写的文件。那就是问题起源的地方。该位置之上的行是你的代码所调用的代码；之下的行是调用你代码的代码。这些前后的行可能包括 Rust 核心代码、标准库代码或你正在使用的 crate。让我们尝试通过将 `RUST_BACKTRACE` 环境变量设置为除 `0` 之外的任何值来获取回溯。示例 9-2 显示了你将看到的类似输出。

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="由对 `panic!` 的调用生成的回溯，在设置了环境变量 `RUST_BACKTRACE` 时显示">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

输出真多！你看到的确切输出可能因你的操作系统和 Rust 版本而异。为了获取具有此信息的回溯，必须启用调试符号（debug symbols）。在使用 `cargo build` 或 `cargo run` 而不带 `--release` 标志时，默认情况下启用调试符号，正如我们这里所做的那样。

在示例 9-2 的输出中，回溯的第 6 行指向了我们项目中导致问题的行：_src/main.rs_ 的第 4 行。如果我们不希望程序 panic，我们应该从第一个提到我们写的文件的位置开始调查。在示例 9-1 中，我们故意编写了会导致 panic 的代码，修复 panic 的方法是不要请求超出向量索引范围的元素。将来当你的代码 panic 时，你需要找出代码使用了什么值执行了什么操作导致了 panic，以及代码应该怎么做。

我们将在本章后面的[“是 `panic!` 还是不 `panic!`”][to-panic-or-not-to-panic]<!-- ignore -->部分回到 `panic!` 以及我们在处理错误情况时应该和不应该使用 `panic!` 的讨论。接下来，我们将看看如何使用 `Result` 从错误中恢复。

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
