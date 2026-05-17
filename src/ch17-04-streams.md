<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## 流（Streams）：顺序处理的 Future

回想一下我们在本章前面的[“消息传递”][17-02-messages]<!-- ignore --> 部分中是如何使用异步通道的接收者的。异步 `recv` 方法随时间推移产生一系列项目。这是一种更通用模式的实例，称为*流（stream）*。许多概念自然地表示为流：队列中可用的项目、当完整数据集过大无法容纳计算机内存时从文件系统增量拉取的数据块，或者随时间推移从网络到达的数据。由于流是 future，我们可以将它们与任何其他类型的 future 一起使用，并以有趣的方式组合它们。例如，我们可以将事件批量处理以避免触发太多网络调用，为一连串长时间运行的操作设置超时，或者对用户界面事件进行节流以避免做不必要的工作。

我们在第 13 章的[“Iterator Trait 和 `next` 方法”][iterator-trait]<!-- ignore --> 部分中看到过一系列项目，当时我们研究了 Iterator trait，但迭代器与异步通道接收者之间有两个区别。第一个区别是时间：迭代器是同步的，而通道接收者是异步的。第二个区别是 API。当直接使用 `Iterator` 时，我们调用其同步的 `next` 方法。而特别是对于 `trpl::Receiver` 流，我们调用了异步的 `recv` 方法。除此之外，这些 API 感觉非常相似，这种相似性并非巧合。流就像是异步形式的迭代。然而，`trpl::Receiver` 专门等待接收消息，而通用流 API 则广泛得多：它以 `Iterator` 的方式提供下一个项目，但是以异步的方式。

迭代器和流在 Rust 中的相似性意味着我们实际上可以从任何迭代器创建一个流。与迭代器一样，我们可以通过调用其 `next` 方法并等待输出来处理流，如示例 17-21 所示，但这段代码还无法编译。

<Listing number="17-21" caption="从迭代器创建流并打印其值" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

我们从一个数字数组开始，将其转换为迭代器，然后调用 `map` 将所有值加倍。然后我们使用 `trpl::stream_from_iter` 函数将迭代器转换为流。接下来，当项目到达时，我们使用 `while let` 循环遍历流中的项目。

不幸的是，当我们尝试运行代码时，它无法编译，而是报告没有可用的 `next` 方法：

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

如这个输出所解释的，编译器错误的原因是我们需要将正确的 trait 引入作用域才能使用 `next` 方法。根据我们目前的讨论，你可能合理地预期那个 trait 是 `Stream`，但它实际上是 `StreamExt`。`Ext` 是*扩展（extension）*的缩写，这是 Rust 社区中用另一个 trait 扩展一个 trait 的常见模式。

`Stream` trait 定义了一个底层接口，有效地结合了 `Iterator` 和 `Future` trait。`StreamExt` 在 `Stream` 之上提供了一组更高层的 API，包括 `next` 方法以及其他类似于 `Iterator` trait 提供的实用方法。`Stream` 和 `StreamExt` 尚未成为 Rust 标准库的一部分，但大多数生态 crate 使用类似的定义。

编译器错误的修复方法是添加一个针对 `trpl::StreamExt` 的 `use` 语句，如示例 17-22 所示。

<Listing number="17-22" caption="成功使用迭代器作为流的基础" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

将所有部分组合在一起后，这段代码按照我们期望的方式工作了！而且，现在我们将 `StreamExt` 引入作用域，我们可以使用它的所有实用方法，就像对迭代器一样。

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
