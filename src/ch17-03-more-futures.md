
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### 向运行时让出控制权

回想一下[“我们的第一个异步程序”][async-program]<!-- ignore --> 一节，在每个等待点，如果被等待的 future 尚未就绪，Rust 会给运行时一个机会来暂停任务并切换到另一个任务。反之亦然：Rust *仅*在等待点暂停 async 代码块并将控制权交还给运行时。等待点之间的所有内容都是同步的。

这意味着如果你在一个 async 代码块中做了大量工作而没有等待点，该 future 将阻止任何其他 future 取得进展。你有时可能会听到这被称为一个 future *饿死（starving）*其他 future。在某些情况下，这可能没什么大不了的。然而，如果你在进行某种昂贵的初始化或长时间运行的工作，或者你有一个 future 将无限期地持续完成某个特定任务，你就需要考虑在何时何地将控制权交还给运行时。

让我们模拟一个长时间运行的操作来说明饿死问题，然后探索如何解决它。示例 17-14 引入了一个 `slow` 函数。

<Listing number="17-14" caption="使用 `thread::sleep` 模拟慢速操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

这段代码使用 `std::thread::sleep` 而不是 `trpl::sleep`，因此调用 `slow` 将阻塞当前线程一段毫秒数。我们可以用 `slow` 来代表现实世界中既长时间运行又阻塞的操作。

在示例 17-15 中，我们使用 `slow` 来模拟在一对 future 中进行此类 CPU 密集型工作。

<Listing number="17-15" caption="调用 `slow` 函数来模拟慢速操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

每个 future 只有在执行了一堆慢速操作*之后*才将控制权交还给运行时。如果你运行这段代码，你将看到如下输出：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

与示例 17-5 中我们使用 `trpl::select` 让获取两个 URL 的 future 竞速类似，`select` 仍然在 `a` 完成后立即结束。不过，两个 future 中对 `slow` 的调用之间没有交错。`a` future 完成其所有工作，直到 `trpl::sleep` 调用被等待，然后 `b` future 完成其所有工作，直到它自己的 `trpl::sleep` 调用被等待，最后 `a` future 完成。为了让两个 future 都能在它们的慢速任务之间取得进展，我们需要等待点，这样我们才能将控制权交还给运行时。这意味着我们需要一个可以等待的东西！

我们已经在示例 17-15 中看到了这种交接发生：如果我们在 `a` future 的末尾移除了 `trpl::sleep`，它将完成而 `b` future *完全*不运行。让我们尝试使用 `trpl::sleep` 函数作为让操作轮流取得进展的起点，如示例 17-16 所示。

<Listing number="17-16" caption="使用 `trpl::sleep` 让操作轮流取得进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

我们在每次 `slow` 调用之间添加了带等待点的 `trpl::sleep` 调用。现在两个 future 的工作交错进行了：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future 在将控制权交给 `b` 之前仍然运行了一小段，因为它在调用 `trpl::sleep` 之前调用了 `slow`，但在此之后，每当其中一个 future 到达等待点时，它们就会来回切换。在这个例子中，我们在每次 `slow` 调用之后都这样做了，但我们可以以任何对我们最有意义的方式分解工作。

不过，我们并不真正想在这里*休眠*：我们想尽快取得进展。我们只需要将控制权交还给运行时。我们可以使用 `trpl::yield_now` 函数直接做到这一点。在示例 17-17 中，我们将所有 `trpl::sleep` 调用替换为 `trpl::yield_now`。

<Listing number="17-17" caption="使用 `yield_now` 让操作轮流取得进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

这段代码既更清晰地表达了实际意图，又可以比使用 `sleep` 快得多，因为像 `sleep` 使用的定时器通常对粒度的精细程度有上限。例如，我们使用的 `sleep` 版本始终会休眠至少一毫秒，即使我们传入一纳秒的 `Duration` 也是如此。再说一遍，现代计算机*非常快*：它们可以在一毫秒内完成很多工作！

这意味着 async 甚至对于计算密集型任务也有用，这取决于你程序还在做哪些其他事情，因为它提供了一种有用的工具来构建程序不同部分之间的关系（代价是异步状态机的开销）。这是一种*协作式多任务处理（cooperative multitasking）*，其中每个 future 都有能力通过等待点来决定何时交还控制权。因此，每个 future 也负有责任避免阻塞过长时间。在一些基于 Rust 的嵌入式操作系统中，这是*唯一*一种多任务处理方式！

当然，在实际代码中，你通常不会在每一行都交替使用函数调用和等待点。虽然以这种方式让出控制权相对廉价，但它并非没有成本。在许多情况下，试图分解计算密集型任务可能会使其显著变慢，因此有时为了*整体*性能，让操作短暂阻塞反而更好。始终进行测量，以确定代码的实际性能瓶颈是什么。不过，如果你*确实*看到大量本应并发发生的工作却以串行方式进行了，那么底层动态是很重要的，需要牢记在心！

### 构建我们自己的异步抽象

我们还可以将 future 组合在一起创建新的模式。例如，我们可以用已有的异步构建块来构建一个 `timeout` 函数。完成后，结果将是另一个构建块，我们可以用它来创建更多的异步抽象。

示例 17-18 展示了我们期望这个 `timeout` 如何与一个慢速 future 一起工作。

<Listing number="17-18" caption="使用我们想象中的 `timeout` 在时间限制下运行一个慢速操作" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

让我们来实现这个！首先，思考一下 `timeout` 的 API：

- 它本身需要是一个异步函数，这样我们才能等待它。
- 它的第一个参数应该是一个待运行的 future。我们可以将其设为泛型，以便它适用于任何 future。
- 它的第二个参数将是等待的最长时间。如果我们使用 `Duration`，这将便于传递给 `trpl::sleep`。
- 它应该返回一个 `Result`。如果 future 成功完成，`Result` 将是包含该 future 产生的值的 `Ok`。如果超时先到，`Result` 将是包含超时等待时长的 `Err`。

示例 17-19 展示了这个声明。

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="定义 `timeout` 的函数签名" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

这满足了我们对类型的目标。现在让我们思考所需的*行为*：我们想要让传入的 future 与时长进行竞速。我们可以使用 `trpl::sleep` 从时长创建一个定时器 future，并使用 `trpl::select` 将该定时器与调用者传入的 future 一起运行。

在示例 17-20 中，我们通过对 `trpl::select` 的结果进行匹配来实现 `timeout`。

<Listing number="17-20" caption="使用 `select` 和 `sleep` 定义 `timeout`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

`trpl::select` 的实现不是公平的：它始终按参数传入的顺序进行轮询（其他 `select` 实现会随机选择先轮询哪个参数）。因此，我们首先将 `future_to_try` 传递给 `select`，以便即使 `max_time` 是非常短的时长，它也有机会完成。如果 `future_to_try` 先完成，`select` 将返回包含 `future_to_try` 输出的 `Left`。如果 `timer` 先完成，`select` 将返回包含定时器输出 `()` 的 `Right`。

如果 `future_to_try` 成功并且我们得到 `Left(output)`，则返回 `Ok(output)`。如果休眠定时器反而先到期并且我们得到 `Right(())`，我们用 `_` 忽略 `()`，转而返回 `Err(max_time)`。

至此，我们有了一个由另外两个异步辅助工具构建而成的可工作的 `timeout`。如果我们运行代码，它将在超时后打印失败模式：

```text
Failed after 2 seconds
```

由于 future 可以与其他 future 组合，你可以使用较小的异步构建块构建非常强大的工具。例如，你可以使用相同的方法将超时与重试结合起来，进而将它们与诸如网络调用（如示例 17-5 中的那些）之类的操作结合使用。

在实践中，你通常直接使用 `async` 和 `await`，其次使用像 `select` 这样的函数和像 `join!` 这样的宏来控制最外层 future 的执行方式。

我们现在已经看到了一些同时处理多个 future 的方法。接下来，我们将看看如何使用*流（streams）*在一段时间内按顺序处理多个 future。

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
