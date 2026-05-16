## 重构以改善模块化和错误处理（Refactoring to Improve Modularity and Error Handling）

为了改进我们的程序，我们将解决与程序结构以及处理潜在错误方式相关的四个问题。首先，我们的 `main` 函数现在执行两个任务：解析参数和读取文件。随着程序增长，`main` 函数处理的不同任务数量将会增加。随着一个函数承担越来越多的职责，它变得更加难以推理、更难以测试、更难以在不破坏其某个部分的情况下进行更改。最好将功能分开，使每个函数只负责一项任务。

此问题也与第二个问题相关：虽然 `query` 和 `file_path` 是我们程序的配置变量，但像 `contents` 这样的变量用于执行程序的逻辑。`main` 越长，我们需要引入作用域的变量就越多；作用域中的变量越多，就越难跟踪每个变量的用途。最好将配置变量分组到一个结构中，以明确它们的用途。

第三个问题是，我们在读取文件失败时使用了 `expect` 来打印错误消息，但错误消息只打印了 `Should have been able to read the file`。读取文件可能以多种方式失败：例如，文件可能缺失，或者我们可能没有权限打开文件。目前，无论情况如何，我们都会为所有情况打印相同的错误消息，这不会给用户任何信息！

第四，我们使用 `expect` 来处理错误，如果用户运行我们的程序时未指定足够的参数，他们将得到一个来自 Rust 的 `index out of bounds` 错误，这并没有清楚地解释问题。最好将所有错误处理代码放在一个地方，这样如果错误处理逻辑需要更改，未来的维护者只需查阅代码的一个地方。将所有错误处理代码放在一个地方也将确保我们打印的消息对最终用户有意义。

让我们通过重构项目来解决这四个问题。

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### 分离二进制项目的关注点

将多个任务的职责分配给 `main` 函数的组织问题在许多二进制项目中都很常见。因此，许多 Rust 程序员发现，当 `main` 函数开始变得庞大时，将二进制程序的不同关注点分开是很有用的。此过程包含以下步骤：

- 将程序拆分为 _main.rs_ 文件和 _lib.rs_ 文件，并将程序的逻辑移到 _lib.rs_ 中。
- 只要你的命令行解析逻辑很小，它可以保留在 `main` 函数中。
- 当命令行解析逻辑开始变得复杂时，将其从 `main` 函数提取到其他函数或类型中。

在此过程之后，`main` 函数中保留的职责应限于以下内容：

- 使用参数值调用命令行解析逻辑
- 设置任何其他配置
- 调用 _lib.rs_ 中的 `run` 函数
- 如果 `run` 返回错误，则处理该错误

这种模式是关于分离关注点：_main.rs_ 处理运行程序，_lib.rs_ 处理手头任务的所有逻辑。因为你无法直接测试 `main` 函数，这种结构通过将程序的所有逻辑移出 `main` 函数，使你可以测试所有逻辑。`main` 函数中保留的代码将足够小，可以通过阅读来验证其正确性。让我们按照此过程重新组织我们的程序。

#### 提取参数解析器

我们将提取解析参数的功能到一个 `main` 将调用的函数中。示例 12-5 显示了 `main` 函数的新开头，它调用了一个新的 `parse_config` 函数，我们将在 _src/main.rs_ 中定义。

<Listing number="12-5" file-name="src/main.rs" caption="从 `main` 提取一个 `parse_config` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

我们仍然将命令行参数收集到一个向量中，但不再在 `main` 函数中将索引 1 处的参数值赋给变量 `query`、将索引 2 处的参数值赋给变量 `file_path`，而是将整个向量传递给 `parse_config` 函数。`parse_config` 函数随后包含确定哪个参数进入哪个变量的逻辑，并将值传递回 `main`。我们仍然在 `main` 中创建 `query` 和 `file_path` 变量，但 `main` 不再负责确定命令行参数和变量之间的对应关系。

这种重写对于我们的小程序来说可能看起来有点小题大做，但我们正在以小的、递增的步骤进行重构。在进行此更改后，再次运行程序以验证参数解析仍然有效。经常检查你的进度是很好的做法，以帮助在出现问题时识别原因。

#### 分组配置值

我们可以再采取一小步来进一步改进 `parse_config` 函数。目前，我们返回一个元组，但随后我们立即将该元组再次分解为各个部分。这是一个迹象，表明我们可能还没有正确的抽象。

另一个表明有改进空间的指标是 `parse_config` 名称中的 `config` 部分，它暗示我们返回的两个值是相关的，并且都是一个配置值的两个部分。我们目前没有在数据结构中传达这种含义，除了将两个值分组到元组中；相反，我们将把这两个值放入一个结构体中，并给每个结构体字段一个有意义的名称。这样做将使未来代码的维护者更容易理解不同值之间的关系以及它们的用途。

示例 12-6 显示了 `parse_config` 函数的改进。

<Listing number="12-6" file-name="src/main.rs" caption="重构 `parse_config` 以返回 `Config` 结构体的实例">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

我们添加了一个名为 `Config` 的结构体，其字段名为 `query` 和 `file_path`。`parse_config` 的签名现在表明它返回一个 `Config` 值。在 `parse_config` 的函数体中，我们之前返回引用 `args` 中 `String` 值的字符串切片，现在我们定义 `Config` 包含拥有所有权的 `String` 值。`main` 中的 `args` 变量是参数值的所有者，并且只让 `parse_config` 函数借用它们，这意味着如果 `Config` 试图获取 `args` 中值的所有权，我们将违反 Rust 的借用规则。

有几种方法可以管理 `String` 数据；最简单（虽然效率略低）的方法是在值上调用 `clone` 方法。这将为 `Config` 实例制作数据的完整副本以供其拥有，这比存储对字符串数据的引用需要更多时间和内存。然而，克隆数据也使我们的代码非常直接，因为我们不必管理引用的生命周期；在这种情况下，牺牲一点性能来换取简单性是一个值得的权衡。

> ### 使用 `clone` 的权衡
>
> 许多 Rustaceans 倾向于避免使用 `clone` 来解决所有权问题，因为它有运行时成本。在[第 13 章][ch13]<!-- ignore -->中，你将学习如何在类似情况下使用更高效的方法。但现在，复制几个字符串以继续前进是可以的，因为你只会复制一次，而且你的文件路径和查询字符串非常小。拥有一个虽然效率略低但可以工作的程序，比在第一次尝试时就过度优化代码要好。随着你对 Rust 越来越有经验，更容易从最有效的解决方案开始，但现在，调用 `clone` 是完全可接受的。

我们已经更新了 `main`，使其将通过 `parse_config` 返回的 `Config` 实例放入名为 `config` 的变量中，并更新了之前使用单独的 `query` 和 `file_path` 变量的代码，使其现在改为使用 `Config` 结构体的字段。

现在我们的代码更清晰地传达了 `query` 和 `file_path` 是相关的，并且它们的目的是配置程序将如何工作。任何使用这些值的代码都知道在 `config` 实例的字段中找到它们，这些字段以它们的用途命名。

#### 为 `Config` 创建构造函数

到目前为止，我们已经将从 `main` 解析命令行参数的逻辑提取出来，并放入 `parse_config` 函数中。这样做帮助我们看到了 `query` 和 `file_path` 值是相关的，并且这种关系应该在代码中传达。然后，我们添加了一个 `Config` 结构体来命名 `query` 和 `file_path` 的相关用途，并能够从 `parse_config` 函数中将值的名称作为结构体字段名返回。

所以，既然 `parse_config` 函数的目的是创建一个 `Config` 实例，我们可以将 `parse_config` 从一个普通函数更改为一个与 `Config` 结构体关联的名为 `new` 的函数。进行此更改将使代码更符合惯例（idiomatic）。我们可以通过调用 `String::new` 来创建标准库中类型（如 `String`）的实例。类似地，通过将 `parse_config` 更改为与 `Config` 关联的 `new` 函数，我们将能够通过调用 `Config::new` 来创建 `Config` 实例。示例 12-7 显示了我们需要做的更改。

<Listing number="12-7" file-name="src/main.rs" caption="将 `parse_config` 改为 `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

我们已经更新了 `main`，将调用 `parse_config` 的地方改为调用 `Config::new`。我们将 `parse_config` 的名称改为 `new`，并将其移到 `impl` 块中，这将 `new` 函数与 `Config` 关联起来。再次编译此代码以确保其正常工作。

### 修复错误处理

现在我们来修复错误处理。回想一下，如果 `args` 向量包含少于三个项，尝试访问索引 1 或索引 2 处的值会导致程序 panic。尝试不带任何参数运行程序；它将如下所示：

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1` 这一行是面向程序员的错误消息。它不会帮助我们的最终用户理解他们应该怎么做。让我们现在修复它。

#### 改进错误消息

在示例 12-8 中，我们在 `new` 函数中添加了一个检查，用于在访问索引 1 和索引 2 之前验证切片是否足够长。如果切片不够长，程序会 panic 并显示更好的错误消息。

<Listing number="12-8" file-name="src/main.rs" caption="添加参数数量的检查">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

这段代码类似于[我们在示例 9-13 中编写的 `Guess::new` 函数][ch9-custom-types]<!-- ignore -->，当 `value` 参数超出有效值范围时，我们调用了 `panic!`。这里我们不是检查值的范围，而是检查 `args` 的长度是否至少为 `3`，函数的其余部分可以在假定此条件已满足的情况下运行。如果 `args` 少于三个项，此条件将为 `true`，我们调用 `panic!` 宏立即结束程序。

在 `new` 中添加这几行代码后，让我们再次不带任何参数运行程序，看看现在的错误是什么样子：

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

这个输出更好：现在我们有了一个合理的错误消息。但是，我们还有一些不想给用户的无关信息。也许我们在示例 9-13 中使用的技术不是这里的最佳选择：调用 `panic!` 更适合编程问题而不是使用问题，[正如第 9 章中讨论的那样][ch9-error-guidelines]<!-- ignore -->。相反，我们将使用你在第 9 章中学到的另一种技术——[返回一个 `Result`][ch9-result]<!-- ignore -->，它表示成功或错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### 返回 `Result` 而不是调用 `panic!`

我们可以改为返回一个 `Result` 值，在成功情况下包含一个 `Config` 实例，在错误情况下描述问题。我们还打算将函数名称从 `new` 改为 `build`，因为许多程序员期望 `new` 函数永远不会失败。当 `Config::build` 与 `main` 通信时，我们可以使用 `Result` 类型来指示存在问题。然后，我们可以更改 `main`，将 `Err` 变体转换为对用户更实用的错误，而不包含 `panic!` 调用带来的 `thread 'main'` 和 `RUST_BACKTRACE` 等周围文本。

示例 12-9 显示了我们现在称为 `Config::build` 的函数的返回值以及函数体返回 `Result` 所需的更改。请注意，在更新 `main` 之前，这将无法编译，我们将在下一个示例中执行此操作。

<Listing number="12-9" file-name="src/main.rs" caption="从 `Config::build` 返回 `Result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

我们的 `build` 函数返回一个 `Result`，在成功情况下包含 `Config` 实例，在错误情况下包含字符串字面量。我们的错误值将始终是具有 `'static` 生命周期的字符串字面量。

我们在函数体中做了两处更改：当用户没有传递足够的参数时，我们现在返回一个 `Err` 值，而不是调用 `panic!`，并且我们将 `Config` 返回值包裹在 `Ok` 中。这些更改使函数符合其新的类型签名。

从 `Config::build` 返回 `Err` 值允许 `main` 函数处理从 `build` 函数返回的 `Result` 值，并在错误情况下更干净地退出进程。

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### 调用 `Config::build` 并处理错误

为了处理错误情况并打印用户友好的消息，我们需要更新 `main` 以处理 `Config::build` 返回的 `Result`，如示例 12-10 所示。我们还将从 `panic!` 中移除以非零错误代码退出命令行工具的责任，并改为手动实现。非零退出状态是一种约定，通知调用我们程序的进程程序以错误状态退出。

<Listing number="12-10" file-name="src/main.rs" caption="如果在构建 `Config` 时失败，则以错误代码退出">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

在此示例中，我们使用了一个尚未详细介绍的方法：`unwrap_or_else`，它由标准库在 `Result<T, E>` 上定义。使用 `unwrap_or_else` 允许我们定义一些自定义的、非 `panic!` 的错误处理。如果 `Result` 是 `Ok` 值，此方法的行为类似于 `unwrap`：它返回 `Ok` 包装的内部值。然而，如果值是 `Err` 值，此方法会调用闭包中的代码——闭包是我们定义并作为参数传递给 `unwrap_or_else` 的匿名函数。我们将在[第 13 章][ch13]<!-- ignore -->更详细地介绍闭包。现在，你只需要知道 `unwrap_or_else` 会将 `Err` 的内部值（在本例中是我们添加在示例 12-9 中的静态字符串 `"not enough arguments"`）传递给我们闭包中竖线之间的参数 `err`。然后闭包中的代码可以在运行时使用 `err` 值。

我们添加了一个新的 `use` 行，将标准库中的 `process` 引入作用域。在错误情况下运行的闭包代码只有两行：我们打印 `err` 值，然后调用 `process::exit`。`process::exit` 函数将立即停止程序并返回作为退出状态码传入的数字。这与我们在示例 12-8 中使用的基于 `panic!` 的处理类似，但我们不再获得所有额外的输出。让我们尝试一下：

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

很好！这个输出对我们的用户友好得多。

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### 从 `main` 提取逻辑

既然我们已经完成了配置解析的重构，让我们转向程序的逻辑。正如我们在[“分离二进制项目的关注点”](#separation-of-concerns-for-binary-projects)<!-- ignore -->中所说的，我们将提取一个名为 `run` 的函数，该函数将包含 `main` 函数中当前所有不涉及设置配置或处理错误的逻辑。完成后，`main` 函数将简洁且易于通过检查验证，我们将能够为所有其他逻辑编写测试。

示例 12-11 显示了提取 `run` 函数的微小增量改进。

<Listing number="12-11" file-name="src/main.rs" caption="提取包含其余程序逻辑的 `run` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

`run` 函数现在包含 `main` 中的剩余逻辑，从读取文件开始。`run` 函数将 `Config` 实例作为参数。

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### 从 `run` 返回错误

将剩余的程序逻辑分离到 `run` 函数后，我们可以改进错误处理，就像我们在示例 12-9 中对 `Config::build` 所做的那样。`run` 函数不会通过调用 `expect` 来允许程序 panic，而是在出现问题时返回一个 `Result<T, E>`。这将使我们能够进一步将处理错误的逻辑以用户友好的方式整合到 `main` 中。示例 12-12 显示了我们需要对 `run` 的签名和主体进行的更改。

<Listing number="12-12" file-name="src/main.rs" caption="将 `run` 函数改为返回 `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

我们在这里做了三个重要更改。首先，我们将 `run` 函数的返回类型更改为 `Result<(), Box<dyn Error>>`。此函数之前返回单元类型 `()`，我们保持该值作为 `Ok` 情况下的返回值。

对于错误类型，我们使用了 trait 对象 `Box<dyn Error>`（并且我们在顶部使用 `use` 语句将 `std::error::Error` 引入作用域）。我们将在[第 18 章][ch18]<!-- ignore -->介绍 trait 对象。现在，只需知道 `Box<dyn Error>` 意味着该函数将返回一个实现了 `Error` trait 的类型，但我们不必指定返回值将是哪种特定类型。这为我们提供了灵活性，可以在不同的错误情况下返回可能不同类型的错误值。`dyn` 关键字是 _dynamic_ 的缩写。

其次，我们移除了对 `expect` 的调用，转而使用 `?` 运算符，正如我们在[第 9 章][ch9-question-mark]<!-- ignore -->中讨论的那样。`?` 不会在错误时 `panic!`，而是从当前函数返回错误值供调用者处理。

第三，`run` 函数现在在成功情况下返回一个 `Ok` 值。我们已经在签名中声明了 `run` 函数的成功类型为 `()`，这意味着我们需要将单元类型值包裹在 `Ok` 值中。这种 `Ok(())` 语法一开始可能看起来有点奇怪。但像这样使用 `()` 是一种符合惯例的方式，表明我们调用 `run` 只是为了它的副作用；它不返回我们需要的结果值。

当你运行此代码时，它将编译但会显示一个警告：

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust 告诉我们，我们的代码忽略了 `Result` 值，而 `Result` 值可能表示发生了错误。但我们没有检查是否发生了错误，编译器提醒我们这里可能应该有一些错误处理代码！让我们现在纠正这个问题。

#### 在 `main` 中处理从 `run` 返回的错误

我们将检查错误并使用与我们在示例 12-10 中对 `Config::build` 使用的类似技术来处理它们，但略有不同：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

我们使用 `if let` 而不是 `unwrap_or_else` 来检查 `run` 是否返回 `Err` 值，如果返回则调用 `process::exit(1)`。`run` 函数不像 `Config::build` 返回 `Config` 实例那样返回我们想要 `unwrap` 的值。因为 `run` 在成功情况下返回 `()`，我们只关心检测错误，所以我们不需要 `unwrap_or_else` 来返回解包的值（那将只是 `()`）。

`if let` 和 `unwrap_or_else` 的函数体在两种情况下是相同的：我们打印错误并退出。

### 将代码拆分为库 Crate

我们的 `minigrep` 项目目前看起来不错！现在我们将拆分 _src/main.rs_ 文件，并将一些代码放入 _src/lib.rs_ 文件中。这样，我们可以测试代码，并且 _src/main.rs_ 文件的职责更少。

让我们在 _src/lib.rs_ 中定义负责搜索文本的代码，而不是在 _src/main.rs_ 中，这将使我们（或任何其他使用我们 `minigrep` 库的人）能够从比我们的 `minigrep` 二进制文件更多的上下文中调用搜索函数。

首先，在 _src/lib.rs_ 中定义 `search` 函数的签名，如示例 12-13 所示，其函数体调用 `unimplemented!` 宏。我们将在填写实现时更详细地解释签名。

<Listing number="12-13" file-name="src/lib.rs" caption="在 *src/lib.rs* 中定义 `search` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

我们在函数定义上使用了 `pub` 关键字，将 `search` 指定为库 crate 公共 API 的一部分。现在我们有了一个库 crate，可以从二进制 crate 中使用它，并且可以测试它！

现在，我们需要将在 _src/lib.rs_ 中定义的代码引入二进制 crate _src/main.rs_ 的作用域并调用它，如示例 12-14 所示。

<Listing number="12-14" file-name="src/main.rs" caption="在 *src/main.rs* 中使用 `minigrep` 库 crate 的 `search` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

我们添加了一行 `use minigrep::search;`，将 `search` 函数从库 crate 引入二进制 crate 的作用域。然后，在 `run` 函数中，我们不再打印文件的内容，而是调用 `search` 函数并传递 `config.query` 值和 `contents` 作为参数。然后，`run` 将使用 `for` 循环打印从 `search` 返回的匹配查询的每一行。这也是一个很好的时机来移除 `main` 函数中显示查询和文件路径的 `println!` 调用，这样我们的程序只打印搜索结果（如果没有发生错误）。

注意，搜索函数将在任何打印发生之前将所有结果收集到一个向量中返回。这种实现在搜索大文件时显示结果可能会很慢，因为结果不会在找到时立即打印；我们将在第 13 章讨论使用迭代器解决此问题的可能方法。

唷！这是大量的工作，但我们为未来的成功奠定了基础。现在处理错误容易得多，而且我们已经使代码更加模块化。从现在开始，我们几乎所有的工作将在 _src/lib.rs_ 中完成。

让我们利用这种新获得的模块化性来做一些旧代码难以做到但新代码容易做到的事情：我们将编写一些测试！

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
