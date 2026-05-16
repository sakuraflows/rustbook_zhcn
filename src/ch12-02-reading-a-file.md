## 读取文件（Reading a File）

现在我们将添加功能来读取 `file_path` 参数中指定的文件。首先，我们需要一个示例文件来进行测试：我们将使用一个包含少量文本、多行且有一些重复单词的文件。示例 12-3 中艾米莉·狄金森的一首诗就很好用！在项目的根目录下创建一个名为 _poem.txt_ 的文件，并输入诗歌"我是无名之辈！你是谁？"

<Listing number="12-3" file-name="poem.txt" caption="艾米莉·狄金森的一首诗，是个很好的测试案例">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

文本就位后，编辑 _src/main.rs_ 并添加读取文件的代码，如示例 12-4 所示。

<Listing number="12-4" file-name="src/main.rs" caption="读取由第二个参数指定的文件的内容">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

首先，我们使用 `use` 语句引入标准库的一个相关部分：我们需要 `std::fs` 来处理文件。

在 `main` 中，新的语句 `fs::read_to_string` 接受 `file_path`，打开该文件，并返回一个包含文件内容的 `std::io::Result<String>` 类型的值。

之后，我们再次添加一个临时的 `println!` 语句，在读取文件后打印 `contents` 的值，以便我们可以检查程序到目前为止是否正常工作。

让我们使用任意字符串作为第一个命令行参数（因为我们尚未实现搜索部分），并将 _poem.txt_ 文件作为第二个参数来运行此代码：

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

很好！代码读取并打印了文件的内容。但代码有一些缺陷。目前，`main` 函数有多个职责：通常，如果每个函数只负责一个概念，函数会更清晰且更易于维护。另一个问题是我们没有尽可能好地处理错误。程序仍然很小，所以这些缺陷不是大问题，但随着程序增长，将更难干净地修复它们。在开发程序时尽早开始重构是一个好习惯，因为重构少量代码要容易得多。我们接下来将这样做。
