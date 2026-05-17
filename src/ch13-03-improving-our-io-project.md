## 改进我们的 I/O 项目

有了这些关于迭代器的新知识，我们可以使用迭代器来改进第 12 章的 I/O 项目，使代码中的某些部分更加清晰和简洁。让我们看看迭代器如何改进我们的 `Config::build` 函数和 `search` 函数的实现。

### 使用迭代器移除一个 `clone`

在示例 12-6 中，我们添加了代码，获取了一个 `String` 值的切片，通过索引切片并克隆值来创建 `Config` 结构体的实例，从而使 `Config` 结构体拥有这些值。在示例 13-17 中，我们重现了示例 12-23 中 `Config::build` 函数的实现。

<Listing number="13-17" file-name="src/main.rs" caption="来自示例 12-23 的 `Config::build` 函数的重现">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

当时，我们说不必担心低效的 `clone` 调用，因为我们将来会移除它们。好吧，现在就是时候了！

我们在这里需要 `clone`，因为我们在参数 `args` 中有一个包含 `String` 元素的切片，但 `build` 函数并不拥有 `args`。为了返回 `Config` 实例的所有权，我们不得不从 `query` 和 `file_path` 字段克隆值，以便 `Config` 实例可以拥有其值。

有了关于迭代器的新知识，我们可以更改 `build` 函数，使其获取迭代器的所有权作为参数，而不是借用切片。我们将使用迭代器功能，而不是检查切片长度和索引特定位置的代码。这将澄清 `Config::build` 函数正在做什么，因为迭代器将访问这些值。

一旦 `Config::build` 获取了迭代器的所有权并停止使用借用的索引操作，我们可以将 `String` 值从迭代器移入 `Config`，而不是调用 `clone` 并进行新的分配。

#### 直接使用返回的迭代器

打开你的 I/O 项目的 _src/main.rs_ 文件，它应该如下所示：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

我们首先将示例 12-24 中的 `main` 函数开头改为示例 13-18 中的代码，这次使用了迭代器。在更新 `Config::build` 之前，这还不能编译。

<Listing number="13-18" file-name="src/main.rs" caption="将 `env::args` 的返回值传递给 `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` 函数返回一个迭代器！现在，我们不将迭代器值收集到向量中然后将切片传递给 `Config::build`，而是直接将 `env::args` 返回的迭代器的所有权传递给 `Config::build`。

接下来，我们需要更新 `Config::build` 的定义。让我们将 `Config::build` 的签名改为示例 13-19 所示。这仍然不能编译，因为我们需要更新函数体。

<Listing number="13-19" file-name="src/main.rs" caption="更新 `Config::build` 的签名以期望迭代器">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

`env::args` 函数的标准库文档显示，它返回的迭代器的类型是 `std::env::Args`，该类型实现了 `Iterator` trait 并返回 `String` 值。

我们更新了 `Config::build` 函数的签名，使参数 `args` 具有泛型类型，并带有 trait 约束 `impl Iterator<Item = String>` 而不是 `&[String]`。我们在第 10 章的[“使用 Trait 作为参数”][impl-trait]<!-- ignore -->部分讨论的这种 `impl Trait` 语法意味着 `args` 可以是任何实现了 `Iterator` trait 并返回 `String` 项的类型。

因为我们正在获取 `args` 的所有权，并且将通过遍历它来修改 `args`，我们可以在 `args` 参数的规范中添加 `mut` 关键字使其可变。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### 使用 `Iterator` Trait 方法

接下来，我们将修复 `Config::build` 的函数体。因为 `args` 实现了 `Iterator` trait，我们知道可以在它上面调用 `next` 方法！示例 13-20 更新了示例 12-23 中的代码以使用 `next` 方法。

<Listing number="13-20" file-name="src/main.rs" caption="更改 `Config::build` 的函数体以使用迭代器方法">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

记住，`env::args` 返回值中的第一个值是程序名称。我们想忽略它并获取下一个值，因此首先我们调用 `next` 并且不对返回值做任何操作。然后，我们调用 `next` 来获取我们想要放入 `Config` 的 `query` 字段的值。如果 `next` 返回 `Some`，我们使用 `match` 提取该值。如果它返回 `None`，意味着没有提供足够的参数，我们提前返回一个 `Err` 值。我们对 `file_path` 值做同样的事情。

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### 使用迭代器适配器使代码更清晰

我们还可以利用 I/O 项目中 `search` 函数的迭代器，该函数在示例 13-21 中重现，与示例 12-19 中的相同。

<Listing number="13-21" file-name="src/lib.rs" caption="来自示例 12-19 的 `search` 函数的实现">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

我们可以使用迭代器适配器方法以更简洁的方式编写此代码。这样做还可以避免拥有一个可变的中间 `results` 向量。函数式编程风格倾向于最小化可变状态的数量以使代码更清晰。移除可变状态可能使未来的增强（如并行搜索）成为可能，因为我们不必管理对 `results` 向量的并发访问。示例 13-22 显示了此更改。

<Listing number="13-22" file-name="src/lib.rs" caption="在 `search` 函数的实现中使用迭代器适配器方法">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```
</Listing>

回想一下，`search` 函数的目的是返回 `contents` 中包含 `query` 的所有行。与示例 13-16 中的 `filter` 示例类似，此代码使用 `filter` 适配器仅保留 `line.contains(query)` 返回 `true` 的行。然后，我们使用 `collect` 将匹配的行收集到另一个向量中。简单多了！你也可以自由地对 `search_case_insensitive` 函数进行相同的更改以使用迭代器方法。

为了进一步改进，可以通过移除对 `collect` 的调用并将返回类型更改为 `impl Iterator<Item = &'a str>` 来从 `search` 函数返回一个迭代器，这样函数就变成了一个迭代器适配器。注意，你还需要更新测试！在进行此更改前后，使用你的 `minigrep` 工具搜索大文件以观察行为差异。在此更改之前，程序在收集所有结果之前不会打印任何结果，但在更改之后，一旦找到每个匹配行，结果就会被打印出来，因为 `run` 函数中的 `for` 循环能够利用迭代器的惰性。

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### 在循环和迭代器之间选择

下一个合理的问题是在你自己的代码中应该选择哪种风格以及为什么：示例 13-21 中的原始实现还是示例 13-22 中使用迭代器的版本（假设我们在返回结果之前收集所有结果，而不是返回迭代器）。大多数 Rust 程序员更喜欢使用迭代器风格。一开始要掌握它有点困难，但是一旦你熟悉了各种迭代器适配器及其作用，迭代器就会更容易理解。代码不是摆弄循环和构建新向量的各个部分，而是专注于循环的高级目标。这抽象掉了一些常见代码，使得更容易看到此代码特有的概念，例如迭代器中每个元素必须通过的过滤条件。

但是这两个实现真正等价吗？直观的假设可能是较低级别的循环会更快。让我们谈谈性能。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
