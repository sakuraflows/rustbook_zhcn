## 接收命令行参数（Accepting Command Line Arguments）

让我们像往常一样使用 `cargo new` 创建一个新项目。我们将项目命名为 `minigrep`，以区别于你可能已在系统上安装的 `grep` 工具：

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

第一个任务是让 `minigrep` 接受两个命令行参数：文件路径和要搜索的字符串。也就是说，我们希望能够使用 `cargo run`、两个连字符（表示后面的参数是给我们的程序的，而不是给 `cargo` 的）、要搜索的字符串和要搜索的文件路径来运行我们的程序，如下所示：

```console
$ cargo run -- searchstring example-filename.txt
```

目前，`cargo new` 生成的程序无法处理我们给它的参数。[crates.io](https://crates.io/) 上的一些现有库可以帮助编写接受命令行参数的程序，但由于你刚刚学习这个概念，让我们自己实现这个能力。

### 读取参数值

为了使 `minigrep` 能够读取传递给它的命令行参数的值，我们需要使用 Rust 标准库中的 `std::env::args` 函数。此函数返回传递给 `minigrep` 的命令行参数的迭代器（iterator）。我们将在[第 13 章][ch13]<!-- ignore -->全面介绍迭代器。现在，你只需要了解关于迭代器的两个细节：迭代器产生一系列值，我们可以在迭代器上调用 `collect` 方法将其转换为集合（例如向量），其中包含迭代器产生的所有元素。

示例 12-1 中的代码允许你的 `minigrep` 程序读取传递给它的任何命令行参数，然后将这些值收集到一个向量中。

<Listing number="12-1" file-name="src/main.rs" caption="将命令行参数收集到向量中并打印它们">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

首先，我们使用 `use` 语句将 `std::env` 模块引入作用域，以便可以使用其 `args` 函数。注意，`std::env::args` 函数嵌套在两层模块中。正如我们在[第 7 章][ch7-idiomatic-use]<!-- ignore -->中讨论的，当所需的函数嵌套在多个模块中时，我们选择将父模块引入作用域，而不是函数本身。这样做，我们可以轻松地使用 `std::env` 中的其他函数。这也比添加 `use std::env::args` 然后仅用 `args` 调用函数更不容易产生歧义，因为 `args` 可能很容易被误认为是当前模块中定义的函数。

> ### `args` 函数与无效 Unicode
>
> 注意，如果任何参数包含无效的 Unicode，`std::env::args` 会 panic。如果你的程序需要接受包含无效 Unicode 的参数，请改用 `std::env::args_os`。该函数返回一个产生 `OsString` 值（而不是 `String` 值）的迭代器。我们为了简单起见在此选择使用 `std::env::args`，因为 `OsString` 值因平台而异，并且比 `String` 值更难以处理。

在 `main` 的第一行，我们调用 `env::args`，并立即使用 `collect` 将迭代器转换为包含迭代器产生的所有值的向量。我们可以使用 `collect` 函数创建多种类型的集合，因此我们显式标注了 `args` 的类型，以指定我们想要一个字符串向量。尽管你在 Rust 中很少需要标注类型，但 `collect` 是你经常需要标注的一个函数，因为 Rust 无法推断出你想要哪种集合。

最后，我们使用调试宏打印向量。让我们先尝试不带参数、再带两个参数运行代码：

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

注意，向量中的第一个值是 `"target/debug/minigrep"`，这是我们的二进制文件名称。这与 C 中参数列表的行为一致，允许程序使用其在执行时被调用的名称。能够访问程序名称通常很方便，以防你想在消息中打印它，或根据调用程序的命令行列名改变程序的行为。但出于本章的目的，我们将忽略它，只保存我们需要的两个参数。

### 将参数值保存在变量中

程序目前能够访问指定为命令行参数的值。现在我们需要将这两个参数的值保存在变量中，以便在程序的其余部分使用这些值。我们在示例 12-2 中执行此操作。

<Listing number="12-2" file-name="src/main.rs" caption="创建变量来保存查询参数和文件路径参数">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

正如我们在打印向量时看到的，程序名称占用了 `args[0]` 中的第一个值，因此我们从索引 1 开始处理参数。`minigrep` 接受的第一个参数是我们要搜索的字符串，因此我们将对第一个参数的引用放入变量 `query` 中。第二个参数将是文件路径，因此我们将对第二个参数的引用放入变量 `file_path` 中。

我们临时打印这些变量的值，以证明代码按我们预期的方式工作。让我们使用参数 `test` 和 `sample.txt` 再次运行此程序：

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

很好，程序在正常工作！我们需要的参数值已保存到正确的变量中。稍后我们将添加一些错误处理，以应对某些潜在的错误情况，例如用户不提供参数时；目前，我们将忽略这种情况，而是着手添加文件读取功能。

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
