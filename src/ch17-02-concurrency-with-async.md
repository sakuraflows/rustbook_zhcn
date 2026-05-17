<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## 使用 Async 实现并发

在本节中，我们将把 async 应用于第 16 章中用线程处理过的一些相同的并发挑战。由于我们已经在那里讨论了很多关键思想，本节将重点放在线程与 future 之间的不同之处上。

在许多情况下，使用 async 处理并发的 API 与使用线程的 API 非常相似。而在其他情况下，它们最终会相当不同。即使线程和 async 的 API *看起来*相似，它们的行为往往也不同——而且它们的性能特性几乎总是不一样。

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### 使用 `spawn_task` 创建新任务

我们在第 16 章[“使用 `spawn` 创建新线程”][thread-spawn]<!-- ignore --> 一节中处理的第一个操作是在两个独立的线程上计数。让我们用 async 来做同样的事情。`trpl` crate 提供了一个 `spawn_task` 函数，看起来与 `thread::spawn` API 非常相似，还提供了一个 `sleep` 函数，它是 `thread::sleep` API 的异步版本。我们可以将它们组合起来实现计数示例，如示例 17-6 所示。

<Listing number="17-6" caption="创建一个新任务，在主任务打印其他内容的同时打印一些内容" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

作为起点，我们使用 `trpl::block_on` 设置 `main` 函数，以便顶层函数可以是异步的。

> 注意：从本章此处开始，每个示例都将在 `main` 中包含这份完全相同的 `trpl::block_on` 包裹代码，因此我们通常会像省略 `main` 一样省略它。请记住在你的代码中包含它！

然后我们在该代码块中编写两个循环，每个循环都包含一个 `trpl::sleep` 调用，该调用等待半秒（500 毫秒）后再发送下一条消息。我们将一个循环放在 `trpl::spawn_task` 的函数体中，另一个放在顶层的 `for` 循环中。我们还在 `sleep` 调用之后添加了 `await`。

这段代码的行为类似于基于线程的实现——包括你在运行它时可能在终端中看到消息以不同顺序出现的事实：

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

这个版本会在主 async 代码块中的 `for` 循环完成时立即停止，因为当 `main` 函数结束时，由 `spawn_task` 生成的任务会被关闭。如果你想让它一直运行到任务完成，你需要使用 join 句柄（join handle）来等待第一个任务完成。在线程中，我们使用 `join` 方法来"阻塞"，直到线程完成运行。在示例 17-7 中，我们可以使用 `await` 来做同样的事情，因为任务句柄本身就是一个 future。其 `Output` 类型是 `Result`，所以我们在等待它之后还要对其解包。

<Listing number="17-7" caption="使用 `await` 配合 join 句柄来运行任务到完成" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

这个更新后的版本会运行到*两个*循环都完成：

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

到目前为止，async 和线程看起来给出了相似的结果，只是语法不同：使用 `await` 而不是在 join 句柄上调用 `join`，以及等待 `sleep` 调用。

更大的区别在于，我们不需要为此生成另一个操作系统线程。事实上，我们甚至不需要在这里生成一个任务。因为 async 代码块编译为匿名 future，我们可以将每个循环放入一个 async 代码块中，并让运行时使用 `trpl::join` 函数将两者运行到完成。

在第 16 章的[“等待所有线程完成”][join-handles]<!-- ignore --> 一节中，我们展示了如何在调用 `std::thread::spawn` 时返回的 `JoinHandle` 类型上使用 `join` 方法。`trpl::join` 函数与之类似，但是针对 future 的。当你给它两个 future 时，它产生一个新的 future，其输出是一个元组，包含你传入的每个 future 在*两者都*完成后的输出。因此，在示例 17-8 中，我们使用 `trpl::join` 等待 `fut1` 和 `fut2` 都完成。我们*不*等待 `fut1` 和 `fut2`，而是等待由 `trpl::join` 产生的新 future。我们忽略了输出，因为它只是一个包含两个单元值的元组。

<Listing number="17-8" caption="使用 `trpl::join` 等待两个匿名 future" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

当我们运行它时，我们看到两个 future 都运行到完成：

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

现在，你每次都会看到完全相同的顺序，这与我们在示例 17-7 中看到的线程和 `trpl::spawn_task` 非常不同。这是因为 `trpl::join` 函数是*公平（fair）*的，意味着它以相同的频率检查每个 future，在它们之间交替，并且永远不会让其中一个在另一个就绪时遥遥领先。对于线程，操作系统决定检查哪个线程以及让它运行多长时间。对于异步 Rust，运行时决定检查哪个任务。（在实践中，细节变得复杂，因为异步运行时在底层可能会使用操作系统线程作为管理并发方式的一部分，因此保证公平性对运行时来说可能需要更多工作——但这仍然是可能的！）运行时不必为任何给定操作保证公平性，它们通常提供不同的 API 让你选择是否想要公平性。

尝试以下这些等待 future 的变体，看看它们的效果：

- 从任意一个或两个循环周围移除 async 代码块。
- 在定义每个 async 代码块后立即等待它。
- 仅将第一个循环包裹在 async 代码块中，并在第二个循环体之后等待生成的 future。

作为额外挑战，看看你是否能在运行代码*之前*弄清楚每种情况下的输出会是什么！

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### 使用消息传递在两个任务之间发送数据

在 future 之间共享数据也会是熟悉的：我们将再次使用消息传递（message passing），但这次使用的是类型和函数的异步版本。我们将采取与第 16 章[“使用消息传递在线程之间传输数据”][message-passing-threads]<!-- ignore --> 一节中稍有不同的路径，以说明基于线程的并发和基于 future 的并发之间的一些关键差异。在示例 17-9 中，我们将从单个 async 代码块开始——*不*像生成一个单独线程那样生成一个单独任务。

<Listing number="17-9" caption="创建一个异步 channel 并将两半分别赋值给 `tx` 和 `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

在这里，我们使用 `trpl::channel`，这是我们在第 16 章中与线程一起使用的多生产者、单消费者通道（multiple-producer, single-consumer channel）API 的异步版本。该 API 的异步版本与基于线程的版本只有一点不同：它使用可变的接收者 `rx`，而不是不可变的，并且其 `recv` 方法产生一个我们需要等待的 future，而不是直接产生值。现在我们可以从发送者向接收者发送消息。注意，我们不需要生成单独的线程甚至任务；我们只需要等待 `rx.recv` 调用。

`std::mpsc::channel` 中的同步 `Receiver::recv` 方法会阻塞，直到收到消息。`trpl::Receiver::recv` 方法不会阻塞，因为它是异步的。它不会阻塞，而是将控制权交还给运行时，直到收到消息或通道的发送端关闭。相比之下，我们不等待 `send` 调用，因为它不会阻塞。它不需要阻塞，因为我们发送到的通道是无界的（unbounded）。

> 注意：因为所有这些异步代码都运行在 `trpl::block_on` 调用的 async 代码块中，所以其中的所有内容都可以避免阻塞。然而，*外部*的代码会在 `block_on` 函数返回时被阻塞。这正是 `trpl::block_on` 函数的要点：它让你*选择*在何处阻塞某些异步代码集，从而在何处进行同步和异步代码之间的转换。

注意这个示例的两个方面。首先，消息将立即到达。其次，虽然我们在这里使用了一个 future，但还没有并发。示例中的所有内容都是顺序发生的，就像没有涉及 future 一样。

让我们通过发送一系列消息并在它们之间休眠来解决第一部分，如示例 17-10 所示。

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="在异步 channel 上发送和接收多条消息，并在每条消息之间使用 `await` 进行休眠" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

除了发送消息之外，我们还需要接收它们。在这种情况下，因为我们知道有多少消息传入，我们可以通过调用四次 `rx.recv().await` 来手动处理。然而在现实世界中，我们通常会等待一些*未知*数量的消息，因此我们需要一直等待，直到确定没有更多消息。

在示例 16-10 中，我们使用 `for` 循环来处理从同步通道接收的所有项目。然而，Rust 目前还没有办法对*异步生成的*系列项目使用 `for` 循环，因此我们需要使用一种我们之前未见过的循环：`while let` 条件循环。这是我们在第 6 章[“使用 `if let` 和 `let...else` 进行简洁的控制流”][if-let]<!-- ignore --> 一节中看到的 `if let` 结构的循环版本。只要它指定的模式继续与该值匹配，该循环就会继续执行。

`rx.recv` 调用产生一个 future，我们等待它。运行时将暂停该 future 直到它就绪。一旦消息到达，future 将解析为 `Some(message)`，每次消息到达时都是如此。当通道关闭时，无论是因为*任何*发送者被丢弃还是因为接收者已耗尽所有值，future 将解析为 `None`，表明再也没有值了，因此我们应该停止轮询——即停止循环。

```console
$ cargo run
   Compiling async_await v0.1.0 (/Users/chris/dev/rust-lang/book/listings/ch17-async-await/listing-17-10)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.63s
     Running `target/debug/async_await`
received 'hi'
received 'from'
received 'the'
received 'future'
```

请注意，这段代码不会关闭！这是因为我们在这里对代码的构造方式。部分原因是有意为之，部分原因则是由于 async 代码块与线程的不同行为。

回到示例 17-10，我们已将异步代码块全部放入 `trpl::block_on` 调用内。这意味着其中的所有内容都是线性运行的——这里没有并发，因此没有发生任何其他事情。这也是发送调用的原因：它们是整个（单个）future 中唯一的代码，因此在发送的最后一条消息之后，没有第二对最后一条消息的接收调用。以下是对应的同步版本会做的：

```rust,ignore
fn main() {
    let (tx, mut rx) = trpl::channel();

    let val = String::from("hi");
    tx.send(val).unwrap();
    let received = rx.recv().await.unwrap();
    println!("received '{received}'");
}
```

那么是什么导致它永远不关闭呢？我们缺少的是一个循环来接收每条消息的代码。让我们在示例 17-11 中添加它。

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="使用 `while let` 循环持续接收消息" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:while-let}}
```

</Listing>

这段代码仍然不会关闭，但它产生了一个错误，这有助于我们理解问题的根源：

```console
$ cargo run
   Compiling async_await v0.1.0 (/Users/chris/dev/rust-lang/book/listings/ch17-async-await/listing-17-11)
error: the async block may outlive the current function, but it borrows `rx`, which is owned by the current function
  --> src/main.rs:41:76
   |
40 |         let fut = async {
   |                   ----- this async block may hold a reference to the current function
41 |             while let Some(message) = rx.recv().await {
   |                                        ^^ `rx` is borrowed here. The async block may outlive the current function;
note: it may outlive the current function because nothing here keeps the async block alive
  --> src/main.rs:52:27
   |
52 |         });
   |                           - the async block could outlive this function call
help: to force the async block to take ownership of `rx` (and any other referenced variables), use the `move` keyword
   |
40 |         let fut = async move {
   |                        ++++

error: could not compile `async_await` (bin "async_await") due to 1 previous error
```

当我们在将 channel 接收代码放入 `while let` 循环后尝试编译时，编译器给我们指出了一个问题。一个 async 代码块可以比当前函数存在得更久，因此编译器无法确定变量 `rx` 在你的 async 代码块使用它时是否仍然有效。在 async 代码块中借用 `rx` 意味着借用必须至少与 async 代码块一样长。像这里的代码有可能工作，但回想一下，Rust 编译器检查内存安全性意味着它不能允许这种代码。为了解决这个问题，我们将告诉编译器我们选择将 `rx` *移动*到 async 代码块中，使用我们在线程场景中使用的相同 `move` 关键字——这是我们在第 13 章中[闭包的"捕获引用还是移动所有权"][capture-or-move]<!-- ignore --> 部分首次看到的。正如我们在第 16 章[“在线程中使用 `move` 闭包”][move-threads]<!-- ignore --> 一节中看到的，在使用线程时，我们经常需要将数据移入闭包。同样的基本动态也适用于 async 代码块，因此 `move` 关键字的工作方式与闭包中的一样。

在示例 17-12 中，我们将用于发送消息的代码块从 `async` 更改为 `async move`。

<Listing number="17-12" caption="对示例 17-11 中代码的修订，使其在完成时正确关闭" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

当我们运行*这个*版本的代码时，它在最后一条消息发送和接收后优雅地关闭。接下来，让我们看看要从多个 future 发送数据需要做哪些改变。

#### 使用 `join!` 宏连接多个 Future

这个异步通道也是一个多生产者通道，所以如果我们想从多个 future 发送消息，可以在 `tx` 上调用 `clone`，如示例 17-13 所示。

<Listing number="17-13" caption="在异步代码块中使用多个生产者" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

首先，我们克隆 `tx`，在第一个 async 代码块外部创建 `tx1`。我们将 `tx1` 移入该代码块，就像我们之前对 `tx` 所做的那样。然后，稍后我们将原始的 `tx` 移入一个*新的* async 代码块，在那里我们以稍慢的延迟发送更多消息。我们恰好将这个新的 async 代码块放在用于接收消息的 async 代码块之后，但它放在之前也一样。关键是 future 被等待的顺序，而不是它们被创建的顺序。

两个用于发送消息的 async 代码块都需要是 `async move` 代码块，这样当这些代码块完成时，`tx` 和 `tx1` 才会被丢弃。否则，我们将再次陷入最初的那个无限循环。

最后，我们从 `trpl::join` 切换到 `trpl::join!` 来处理额外的 future：`join!` 宏可以等待任意数量的 future，其中 future 的数量在编译时已知。我们将在本章后面讨论如何等待未知数量的 future 集合。

现在我们看到了来自两个发送 future 的所有消息，并且由于发送 future 在发送后使用稍有不同的延迟，消息也以这些不同的间隔被接收：

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

我们已经探索了如何使用消息传递在 future 之间发送数据，async 代码块内的代码如何顺序运行，如何将所有权移入 async 代码块，以及如何连接多个 future。接下来，让我们讨论如何以及为什么要告诉运行时它可以切换到另一个任务。

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
