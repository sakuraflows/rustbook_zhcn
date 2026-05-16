## 如何编写测试（How to Write Tests）

*测试（Tests）*是验证非测试代码是否按预期方式运行的 Rust 函数。测试函数的主体通常执行以下三个操作：

- 设置所需的任何数据或状态。
- 运行你想要测试的代码。
- 断言结果是你所期望的。

让我们看看 Rust 专门为编写执行这些操作的测试而提供的功能，其中包括 `test` 属性、几个宏以及 `should_panic` 属性。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### 测试函数的结构

最简单地说，Rust 中的一个测试是标注了 `test` 属性的函数。属性是关于 Rust 代码片段的元数据；一个例子是我们在第 5 章中与结构体一起使用的 `derive` 属性。要将函数更改为测试函数，请在 `fn` 之前的一行添加 `#[test]`。当你使用 `cargo test` 命令运行测试时，Rust 会构建一个测试运行器（test runner）二进制文件，该文件运行标注过的函数，并报告每个测试函数是通过还是失败。

每当我们使用 Cargo 创建新的库项目时，都会自动为我们生成一个包含测试函数的测试模块。这个模块为你提供了编写测试的模板，这样你就不必在每次启动新项目时都查找确切的结构和语法。你可以添加任意数量的额外测试函数和测试模块！

在实际测试任何代码之前，我们将通过尝试模板测试来探讨测试工作的一些方面。然后，我们将编写一些真实的测试，调用我们编写的一些代码，并断言其行为是正确的。

让我们创建一个名为 `adder` 的新库项目，它将两个数相加：

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

`adder` 库中 _src/lib.rs_ 文件的内容应该如示例 11-1 所示。

<Listing number="11-1" file-name="src/lib.rs" caption="`cargo new` 自动生成的代码">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

该文件以一个示例 `add` 函数开头，以便我们有东西可以测试。

现在，让我们只关注 `it_works` 函数。注意 `#[test]` 注解：此属性表明这是一个测试函数，因此测试运行器知道将此函数视为测试。我们可能还在 `tests` 模块中有非测试函数，用于帮助设置常见场景或执行常见操作，因此我们总是需要指出哪些函数是测试。

示例函数体使用 `assert_eq!` 宏来断言 `result`（包含调用参数 2 和 2 的 `add` 的结果）等于 4。这个断言作为典型测试格式的示例。让我们运行它，看看这个测试是否通过。

`cargo test` 命令运行我们项目中的所有测试，如示例 11-2 所示。

<Listing number="11-2" caption="运行自动生成的测试的输出">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo 编译并运行了测试。我们看到 `running 1 test` 这一行。下一行显示了生成的测试函数的名称 `tests::it_works`，并且运行该测试的结果是 `ok`。总体摘要 `test result: ok.` 意味着所有测试都通过了，`1 passed; 0 failed` 部分汇总了通过或失败的测试数量。

可以将测试标记为忽略，使其在特定实例中不运行；我们将在本章后面的[“除非特别请求，否则忽略测试”][ignoring]<!-- ignore -->部分介绍这一点。因为我们这里还没有这样做，所以摘要显示 `0 ignored`。我们也可以向 `cargo test` 命令传递参数，以仅运行名称与字符串匹配的测试；这称为*过滤（filtering）*，我们将在[“按名称运行测试子集”][subset]<!-- ignore -->部分介绍。在这里，我们没有过滤正在运行的测试，因此摘要末尾显示 `0 filtered out`。

`0 measured` 统计信息适用于衡量性能的基准测试（benchmark tests）。截至撰写本文时，基准测试仅在 nightly Rust 中可用。请参阅[关于基准测试的文档][bench]以了解更多。

测试输出中从 `Doc-tests adder` 开始的下一部分是关于任何文档测试的结果。我们还没有文档测试，但 Rust 可以编译出现在我们 API 文档中的任何代码示例。这个功能有助于保持文档和代码的同步！我们将在第 14 章的[“文档注释作为测试”][doc-comments]<!-- ignore -->部分讨论如何编写文档测试。现在，我们将忽略 `Doc-tests` 输出。

让我们开始根据我们自己的需要定制测试。首先，将 `it_works` 函数的名称更改为其他名称，例如 `exploration`，如下所示：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

然后，再次运行 `cargo test`。现在的输出显示 `exploration` 而不是 `it_works`：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

现在我们将添加另一个测试，但这次我们将使其失败！当测试函数中的某些内容 panic 时，测试就会失败。每个测试都在一个新的线程中运行，当主线程看到测试线程已死亡时，该测试被标记为失败。在第 9 章中，我们讨论了最简单的 panic 方式是调用 `panic!` 宏。将新的测试输入为名为 `another` 的函数，这样你的 _src/lib.rs_ 文件看起来像示例 11-3。

<Listing number="11-3" file-name="src/lib.rs" caption="添加第二个测试，该测试将失败，因为我们调用了 `panic!` 宏">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

再次使用 `cargo test` 运行测试。输出应该如示例 11-4 所示，显示我们的 `exploration` 测试通过了，而 `another` 失败了。

<Listing number="11-4" caption="一个测试通过、一个测试失败时的测试结果">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

代替 `ok`，`test tests::another` 这一行显示 `FAILED`。在单个结果和摘要之间出现了两个新部分：第一部分显示了每个测试失败的详细原因。在这种情况下，我们得到详细信息：`tests::another` 失败，因为它在 _src/lib.rs_ 文件的第 17 行 panic，消息为 `Make this test fail`。下一部分仅列出了所有失败测试的名称，当测试很多且失败的测试输出很详细时，这很有用。我们可以使用失败测试的名称仅运行该测试以更轻松地进行调试；我们将在[“控制测试的运行方式”][controlling-how-tests-are-run]<!-- ignore -->部分中更多地讨论运行测试的方法。

最后显示摘要行：总体而言，我们的测试结果是 `FAILED`。我们有一个测试通过、一个测试失败。

现在你已经看到了在不同场景下测试结果的样子，让我们看看除 `panic!` 之外的一些在测试中有用的宏。

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### 使用 `assert!` 宏检查结果

标准库提供的 `assert!` 宏在你想要确保测试中的某个条件计算结果为 `true` 时非常有用。我们向 `assert!` 宏传递一个计算结果为布尔值的参数。如果值为 `true`，则不会发生任何事情，测试通过。如果值为 `false`，`assert!` 宏会调用 `panic!` 以使测试失败。使用 `assert!` 宏有助于检查我们的代码是否按照我们期望的方式运行。

在第 5 章示例 5-15 中，我们使用了 `Rectangle` 结构体和 `can_hold` 方法，这些内容在示例 11-5 中重复出现。让我们将此代码放入 _src/lib.rs_ 文件中，然后使用 `assert!` 宏为其编写一些测试。

<Listing number="11-5" file-name="src/lib.rs" caption="第 5 章中的 `Rectangle` 结构体及其 `can_hold` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` 方法返回一个布尔值，这意味着它是 `assert!` 宏的完美用例。在示例 11-6 中，我们编写了一个测试，通过创建一个宽度为 8、高度为 7 的 `Rectangle` 实例，并断言它可以容纳另一个宽度为 5、高度为 1 的 `Rectangle` 实例，来检验 `can_hold` 方法。

<Listing number="11-6" file-name="src/lib.rs" caption="测试 `can_hold`，检查较大的矩形是否确实可以容纳较小的矩形">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

注意 `tests` 模块中的 `use super::*;` 这一行。`tests` 模块是一个常规模块，遵循我们在第 7 章[“路径用于引用模块树中的项”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->部分中介绍的常规可见性规则。因为 `tests` 模块是一个内部模块，我们需要将外部模块中要测试的代码引入内部模块的作用域。我们在这里使用了全局导入（glob），因此我们在外部模块中定义的任何内容都可以在此 `tests` 模块中使用。

我们将测试命名为 `larger_can_hold_smaller`，并创建了我们需要的两个 `Rectangle` 实例。然后，我们调用了 `assert!` 宏，并将调用 `larger.can_hold(&smaller)` 的结果传递给它。此表达式应该返回 `true`，因此我们的测试应该通过。让我们来验证！

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

确实通过了！让我们添加另一个测试，这次断言较小的矩形不能容纳较大的矩形：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

因为在这种情况下 `can_hold` 函数的正确结果是 `false`，我们需要在将其传递给 `assert!` 宏之前对结果进行取反。因此，如果 `can_hold` 返回 `false`，我们的测试将通过：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

两个测试都通过了！现在让我们看看在代码中引入 bug 时测试结果会发生什么。我们将更改 `can_hold` 方法的实现，在比较宽度时将大于号（`>`）替换为小于号（`<`）：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

现在运行测试会产生以下结果：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

我们的测试捕捉到了 bug！因为 `larger.width` 是 `8`，`smaller.width` 是 `5`，`can_hold` 中的宽度比较现在返回 `false`：8 不小于 5。

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### 使用 `assert_eq!` 和 `assert_ne!` 宏测试相等性

验证功能的一种常见方法是测试代码的结果与你期望代码返回的值是否相等。你可以通过使用 `assert!` 宏并向其传递使用 `==` 运算符的表达式来做到这一点。然而，这是一个非常常见的测试，标准库提供了一对宏——`assert_eq!` 和 `assert_ne!`——来更方便地执行此测试。这两个宏分别比较两个参数是否相等或不相等。如果断言失败，它们还会打印这两个值，这使得更容易看到测试*为什么*失败；相反，`assert!` 宏只表明它为 `==` 表达式得到了一个 `false` 值，而不打印导致该 `false` 值的值。

在示例 11-7 中，我们编写了一个名为 `add_two` 的函数，它将 `2` 加到其参数上，然后使用 `assert_eq!` 宏测试此函数。

<Listing number="11-7" file-name="src/lib.rs" caption="使用 `assert_eq!` 宏测试函数 `add_two`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

让我们检查它是否通过！

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

我们创建了一个名为 `result` 的变量，它持有调用 `add_two(2)` 的结果。然后，我们将 `result` 和 `4` 作为参数传递给 `assert_eq!` 宏。此测试的输出行是 `test tests::it_adds_two ... ok`，而 `ok` 文本表明我们的测试通过了！

让我们在代码中引入一个 bug，看看 `assert_eq!` 在失败时是什么样子。将 `add_two` 函数的实现改为加 `3`：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

再次运行测试：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

我们的测试捕捉到了 bug！`tests::it_adds_two` 测试失败了，消息告诉我们失败的断言是 `left == right`，并且 `left` 和 `right` 的值是什么。这条消息帮助我们开始调试：`left` 参数（我们持有调用 `add_two(2)` 的结果）是 `5`，但 `right` 参数是 `4`。你可以想象，当我们有大量测试时，这将特别有帮助。

请注意，在某些语言和测试框架中，相等性断言函数的参数被称为 `expected` 和 `actual`，并且指定参数的顺序很重要。然而，在 Rust 中，它们被称为 `left` 和 `right`，并且我们指定期望值和代码产生的值的顺序并不重要。我们可以在此测试中将断言写为 `assert_eq!(4, result)`，这将产生相同的失败消息，显示 `` assertion `left == right` failed ``。

`assert_ne!` 宏在我们给定的两个值不相等时通过，在它们相等时失败。当我们不确定某个值*会*是什么，但我们知道该值肯定*不应该*是什么时，此宏最有用。例如，如果我们正在测试一个保证会以某种方式改变其输入的函数，但输入改变的方式取决于运行测试的星期几，那么最好的断言可能是函数的输出不等于输入。

在底层，`assert_eq!` 和 `assert_ne!` 宏分别使用 `==` 和 `!=` 运算符。当断言失败时，这些宏使用调试格式打印它们的参数，这意味着被比较的值必须实现 `PartialEq` 和 `Debug` trait。所有基本类型和大多数标准库类型都实现了这些 trait。对于你自己定义的结构体和枚举，你需要实现 `PartialEq` 来断言这些类型的相等性。你还需要实现 `Debug` 以在断言失败时打印这些值。由于这两个都是可派生 trait，如第 5 章示例 5-12 所述，这通常只需在结构体或枚举定义上添加 `#[derive(PartialEq, Debug)]` 注解即可。有关这些和其他可派生 trait 的更多详细信息，请参阅附录 C [“可派生 Trait”][derivable-traits]<!-- ignore -->。

### 添加自定义失败消息

你还可以添加自定义消息，作为可选参数与 `assert!`、`assert_eq!` 和 `assert_ne!` 宏一起打印在失败消息中。在必需参数之后指定的任何参数都会传递给 `format!` 宏（在第 8 章的[“使用 `+` 或 `format!` 进行拼接”][concatenating]<!-- ignore -->中讨论过），因此你可以传递一个包含 `{}` 占位符的格式字符串以及要放入这些占位符的值。自定义消息对于记录断言的含义很有用；当测试失败时，你将更好地了解代码的问题所在。

例如，假设我们有一个按名称问候人们的函数，我们希望测试传递给该函数的名称是否出现在输出中：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

该程序的要求尚未达成一致，并且我们相当确定问候语开头的 `Hello` 文本会发生变化。我们决定不想在需求发生变化时更新测试，因此我们不检查与 `greeting` 函数返回值的完全相等性，而只是断言输出包含输入参数的文本。

现在让我们通过更改 `greeting` 以排除 `name` 来在代码中引入一个 bug，看看默认的测试失败是什么样子：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

运行此测试会产生以下结果：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

此结果仅表明断言失败以及断言所在的行。更有用的失败消息会打印来自 `greeting` 函数的值。让我们添加一个自定义失败消息，由格式字符串组成，其中占位符填充了从 `greeting` 函数获得的实际值：

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

现在当我们运行测试时，我们会得到更详细的错误消息：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

我们可以在测试输出中看到实际获得的值，这将帮助我们调试发生了什么，而不是我们期望发生什么。

### 使用 `should_panic` 检查 panic

除了检查返回值之外，检查我们的代码是否按我们期望的方式处理错误条件也很重要。例如，考虑我们在第 9 章示例 9-13 中创建的 `Guess` 类型。使用 `Guess` 的其他代码依赖于这样的保证：`Guess` 实例将仅包含 1 到 100 之间的值。我们可以编写一个测试，确保尝试使用超出该范围的值创建 `Guess` 实例会导致 panic。

我们通过将 `should_panic` 属性添加到测试函数来实现这一点。如果函数内部的代码 panic，则测试通过；如果函数内部的代码没有 panic，则测试失败。

示例 11-8 显示了一个测试，它检查 `Guess::new` 的错误条件是否在我们期望的时候发生。

<Listing number="11-8" file-name="src/lib.rs" caption="测试某个条件会导致 `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

我们将 `#[should_panic]` 属性放在 `#[test]` 属性之后、它所应用的测试函数之前。当此测试通过时，我们来看看结果：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

看起来不错！现在让我们通过在代码中移除 `new` 函数在值大于 100 时会 panic 的条件来引入一个 bug：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

当我们运行示例 11-8 中的测试时，它会失败：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

在这种情况下，我们没有得到非常有用的消息，但是当我们查看测试函数时，我们看到它被注解为 `#[should_panic]`。我们得到的失败意味着测试函数中的代码没有导致 panic。

使用 `should_panic` 的测试可能不精确。即使测试由于不同于我们预期的原因 panic，`should_panic` 测试也会通过。为了使 `should_panic` 测试更精确，我们可以向 `should_panic` 属性添加一个可选的 `expected` 参数。测试框架将确保失败消息包含提供的文本。例如，考虑示例 11-9 中修改后的 `Guess` 代码，其中 `new` 函数根据值是太小还是太大而使用不同的消息 panic。

<Listing number="11-9" file-name="src/lib.rs" caption="测试包含指定子串的 panic 消息的 `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

此测试将通过，因为我们放在 `should_panic` 属性的 `expected` 参数中的值是 `Guess::new` 函数 panic 时消息的子串。我们可以指定预期的整个 panic 消息，在本例中为 `Guess value must be less than or equal to 100, got 200`。你选择指定的内容取决于 panic 消息中有多少是唯一或动态的，以及你希望测试有多精确。在这种情况下，panic 消息的子串足以确保测试函数中的代码执行了 `else if value > 100` 分支。

要查看带有 `expected` 消息的 `should_panic` 测试失败时会发生什么，让我们再次通过在代码中交换 `if value < 1` 和 `else if value > 100` 块的主体来引入一个 bug：

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

这次当我们运行 `should_panic` 测试时，它会失败：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

失败消息表明该测试确实如我们预期的那样 panic 了，但 panic 消息不包含预期的字符串 `less than or equal to 100`。在这种情况下，我们得到的 panic 消息是 `Guess value must be greater than or equal to 1, got 200`。现在我们可以开始找出 bug 在哪里了！

### 在测试中使用 `Result<T, E>`

到目前为止，我们所有的测试在失败时都会 panic。我们也可以编写使用 `Result<T, E>` 的测试！以下是示例 11-1 中的测试，重写为使用 `Result<T, E>` 并在失败时返回 `Err` 而不是 panic：

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

`it_works` 函数现在的返回类型是 `Result<(), String>`。在函数体中，我们不调用 `assert_eq!` 宏，而是在测试通过时返回 `Ok(())`，在测试失败时返回包含 `String` 的 `Err`。

编写返回 `Result<T, E>` 的测试使您能够在测试主体中使用问号运算符，这可以方便地编写如果其中的任何操作返回 `Err` 变体就应失败的测试。

你不能在使用 `Result<T, E>` 的测试上使用 `#[should_panic]` 注解。要断言操作返回 `Err` 变体，*不要*在 `Result<T, E>` 值上使用问号运算符。相反，使用 `assert!(value.is_err())`。

现在你已经了解了编写测试的几种方法，让我们看看运行测试时会发生什么，并探索我们可以与 `cargo test` 一起使用的不同选项。

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
