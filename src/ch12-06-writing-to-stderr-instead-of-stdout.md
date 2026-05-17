<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## 将错误重定向到标准错误（Redirecting Errors to Standard Error）

目前，我们使用 `println!` 宏将所有输出写入终端。在大多数终端中，有两种输出：用于一般信息的*标准输出（standard output，stdout）*和用于错误消息的*标准错误（standard error，stderr）*。这种区别使用户能够选择将程序的成功输出定向到文件，但仍然在屏幕上打印错误消息。

`println!` 宏只能打印到标准输出，因此我们必须使用其他方法来打印到标准错误。

### 检查错误写入位置

首先，让我们观察一下 `minigrep` 当前打印的内容是如何写入标准输出的，包括任何我们希望改为写入标准错误的错误消息。我们将通过将标准输出流重定向到文件，同时故意引发错误来做到这一点。我们不会重定向标准错误流，因此任何发送到标准错误的内容都会继续显示在屏幕上。

命令行程序应该将错误消息发送到标准错误流，这样即使我们将标准输出流重定向到文件，我们仍然可以在屏幕上看到错误消息。我们的程序目前行为不佳：我们即将看到它将错误消息输出保存到了文件中！

为了演示这种行为，我们将使用 `>` 和要重定向标准输出流的文件路径 _output.txt_ 来运行程序。我们不传递任何参数，这应该会引发错误：

```console
$ cargo run > output.txt
```

`>` 语法告诉 shell 将标准输出的内容写入 _output.txt_ 而不是屏幕。我们没有看到期望的错误消息打印到屏幕上，所以这意味着它一定进入了文件中。这就是 _output.txt_ 包含的内容：

```text
Problem parsing arguments: not enough arguments
```

是的，我们的错误消息被打印到了标准输出。对于这样的错误消息来说，将其打印到标准错误要有用得多，这样只有成功运行的数据才会进入文件。我们将改变这一点。

### 将错误打印到标准错误

我们将使用示例 12-24 中的代码来更改错误消息的打印方式。由于我们在本章前面进行了重构，所有打印错误消息的代码都在一个函数 `main` 中。标准库提供了 `eprintln!` 宏，它打印到标准错误流，因此让我们将调用 `println!` 打印错误的两个地方改为使用 `eprintln!`。

<Listing number="12-24" file-name="src/main.rs" caption="使用 `eprintln!` 将错误消息写入标准错误而不是标准输出">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

现在让我们以相同的方式再次运行程序，不带任何参数并使用 `>` 重定向标准输出：

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

现在我们看到错误在屏幕上，而 _output.txt_ 为空，这是我们对命令行程序的期望行为。

让我们再次运行程序，使用不会引发错误的参数，但仍然将标准输出重定向到一个文件，如下所示：

```console
$ cargo run -- to poem.txt > output.txt
```

我们不会在终端中看到任何输出，而 _output.txt_ 将包含我们的结果：

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

这表明我们现在适当地使用标准输出用于成功输出，使用标准错误用于错误输出。

## 总结

本章回顾了你到目前为止学到的一些主要概念，并介绍了如何在 Rust 中执行常见的 I/O 操作。通过使用命令行参数、文件、环境变量和用于打印错误的 `eprintln!` 宏，你现在已经准备好编写命令行应用程序。结合之前章节中的概念，你的代码将组织良好，有效地在适当的数据结构中存储数据，妥善处理错误，并且得到良好的测试。

接下来，我们将探索一些受函数式语言启发的 Rust 特性：闭包和迭代器。
