## 模式可以使用的所有位置

模式在 Rust 中的许多地方都会出现，你一直在使用它们却没有意识到！本节讨论模式有效的所有位置。

### `match` 分支

如第 6 章所述，我们在 `match` 表达式的分支中使用模式。形式上，`match` 表达式定义为关键字 `match`、要匹配的值，以及一个或多个匹配分支，每个分支由一个模式和一个在该值匹配该分支模式时运行的表达式组成，如下所示：

<!--
  手动格式化而非使用 Markdown，因为 Markdown 不支持在这种块体中使代码变为斜体！
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

例如，下面是来自清单 6-5 的 `match` 表达式，它匹配变量 `x` 中的 `Option<i32>` 值：

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

这个 `match` 表达式中的模式是每个箭头左侧的 `None` 和 `Some(i)`。

`match` 表达式的一个要求是它们必须是穷尽的（exhaustive），即必须考虑到 `match` 表达式中值的所有可能性。确保覆盖所有可能性的一种方法是在最后一个分支中使用一个万能模式（catch-all pattern）：例如，一个匹配任何值的变量名永远不会失败，因此覆盖了所有剩余情况。

特定的模式 `_` 将匹配任何内容，但从不绑定到变量，因此它经常用于最后一个匹配分支。当你想要忽略任何未指定的值时，`_` 模式非常有用。我们将在本章后面的[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->中更详细地介绍 `_` 模式。

### `let` 语句

在本章之前，我们只明确讨论了在 `match` 和 `if let` 中使用模式，但事实上，我们也在其他地方使用了模式，包括在 `let` 语句中。例如，考虑这个使用 `let` 的简单变量赋值：

```rust
let x = 5;
```

每次你使用像这样的 `let` 语句时，你都在使用模式，尽管你可能没有意识到！更正式地说，`let` 语句看起来像这样：

<!--
  手动格式化而非使用 Markdown，因为 Markdown 不支持在这种块体中使代码变为斜体！
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

在像 `let x = 5;` 这样的语句中，变量名在 PATTERN（模式）位置上只是一个特别简单的模式形式。Rust 将表达式与模式进行比较，并分配它找到的任何名称。因此，在 `let x = 5;` 示例中，`x` 是一个模式，意思是"将这里匹配的内容绑定到变量 `x`。"由于名称 `x` 就是整个模式，这个模式实际上意思是"将所有内容绑定到变量 `x`，无论值是什么。"

为了更清楚地看到 `let` 的模式匹配方面，考虑清单 19-1，它使用带有模式的 `let` 来解构一个元组。

<Listing number="19-1" caption="使用模式解构元组并同时创建三个变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

这里，我们将一个元组与一个模式进行匹配。Rust 将值 `(1, 2, 3)` 与模式 `(x, y, z)` 进行比较，发现该值匹配该模式——也就是说，它看到两者的元素数量相同——因此 Rust 将 `1` 绑定到 `x`，`2` 绑定到 `y`，`3` 绑定到 `z`。你可以将这个元组模式看作在其内部嵌套了三个单独的变量模式。

如果模式中的元素数量与元组中的元素数量不匹配，则整体类型将不匹配，我们将得到一个编译器错误。例如，清单 19-2 显示了一次尝试将具有三个元素的元组解构为两个变量，这是行不通的。

<Listing number="19-2" caption="错误地构建了一个变量数量与元组元素数量不匹配的模式">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

尝试编译此代码会导致以下类型错误：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

要修复这个错误，我们可以使用 `_` 或 `..` 忽略元组中的一个或多个值，如[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->部分所述。如果问题是模式中的变量太多，解决方案是通过删除变量使类型匹配，使变量数量等于元组中的元素数量。

### 条件 `if let` 表达式

在第 6 章中，我们讨论过如何使用 `if let` 表达式，主要是作为编写只匹配一种情况的 `match` 的更简短方式。可选地，`if let` 可以有一个对应的 `else`，其中包含在 `if let` 中的模式不匹配时要运行的代码。

清单 19-3 展示了也可以混合和匹配 `if let`、`else if` 和 `else if let` 表达式。这样做比 `match` 表达式给了我们更多的灵活性，因为 `match` 中我们只能表达一个值与模式进行比较。此外，Rust 不要求一系列 `if let`、`else if` 和 `else if let` 分支中的条件相互关联。

清单 19-3 中的代码基于一系列针对多个条件的检查来决定背景颜色。在此示例中，我们创建了具有硬编码值的变量，而在实际程序中可能从用户输入接收这些值。

<Listing number="19-3" file-name="src/main.rs" caption="混合使用 `if let`、`else if`、`else if let` 和 `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

如果用户指定了喜欢的颜色，则使用该颜色作为背景。如果没有指定喜欢的颜色，并且今天是星期二，则背景颜色是绿色。否则，如果用户将年龄指定为字符串，并且我们可以成功地将其解析为数字，则颜色根据该数字的值是紫色还是橙色。如果这些条件都不适用，背景颜色是蓝色。

这种条件结构让我们能够支持复杂的需求。使用这里的硬编码值，此示例将打印 `Using purple as the background color`。

你可以看到 `if let` 也可以像 `match` 分支那样引入新的变量来遮蔽现有变量：`if let Ok(age) = age` 这一行引入了一个新的 `age` 变量，其中包含 `Ok` 变体内部的值，遮蔽了现有的 `age` 变量。这意味着我们需要将 `if age > 30` 条件放在该块内：我们不能将这两个条件组合成 `if let Ok(age) = age && age > 30`。我们想要与 30 比较的新 `age` 变量，在新的作用域以花括号开始时才是有效的。

使用 `if let` 表达式的缺点是编译器不会检查穷尽性（exhaustiveness），而 `match` 表达式会检查。如果我们省略了最后一个 `else` 块，因此错过了处理某些情况，编译器不会提醒我们可能的逻辑错误。

### `while let` 条件循环

与 `if let` 结构类似，`while let` 条件循环允许 `while` 循环在模式持续匹配的情况下持续运行。在清单 19-4 中，我们展示了一个 `while let` 循环，它等待线程之间发送的消息，但这里检查的是 `Result` 而不是 `Option`。

<Listing number="19-4" caption="使用 `while let` 循环在 `rx.recv()` 返回 `Ok` 时持续打印值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

此示例打印 `1`、`2`，然后打印 `3`。`recv` 方法从通道的接收端取出第一条消息并返回 `Ok(value)`。当我们第一次在第 16 章中看到 `recv` 时，我们直接展开了错误，或者使用 `for` 循环将其作为迭代器与之交互。然而，如清单 19-4 所示，我们也可以使用 `while let`，因为只要发送端存在，`recv` 方法每次有消息到达时都会返回 `Ok`，然后在发送端断开连接后产生一个 `Err`。

### `for` 循环

在 `for` 循环中，紧跟在关键字 `for` 之后的值是一个模式。例如，在 `for x in y` 中，`x` 就是模式。清单 19-5 展示了如何在 `for` 循环中使用模式来解构或分解元组。

<Listing number="19-5" caption="在 `for` 循环中使用模式解构元组">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

清单 19-5 中的代码将打印以下内容：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

我们使用 `enumerate` 方法来适配迭代器，使其产生一个值和该值的索引，并放入一个元组中。第一个产生的值是元组 `(0, 'a')`。当该值与模式 `(index, value)` 匹配时，`index` 将是 `0`，`value` 将是 `'a'`，打印输出的第一行。

### 函数参数

函数参数也可以是模式。清单 19-6 中的代码声明了一个名为 `foo` 的函数，它接受一个名为 `x` 的 `i32` 类型参数，到现在你应该已经熟悉了。

<Listing number="19-6" caption="一个在参数中使用模式的函数签名">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` 部分就是一个模式！就像我们使用 `let` 一样，我们可以将函数参数中的元组与模式进行匹配。清单 19-7 在我们向函数传递元组时将其中的值拆分出来。

<Listing number="19-7" file-name="src/main.rs" caption="一个参数解构元组的函数">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

此代码打印 `Current location: (3, 5)`。值 `&(3, 5)` 匹配模式 `&(x, y)`，因此 `x` 是值 `3`，`y` 是值 `5`。

我们也可以以与函数参数列表相同的方式在闭包参数列表中使用模式，因为闭包类似于函数，如第 13 章所述。

到目前为止，你已经看到了几种使用模式的方式，但模式并非在我们能使用它们的每个地方都以相同的方式工作。在某些地方，模式必须是不可反驳的（irrefutable）；在其他情况下，它们可以是可反驳的（refutable）。我们接下来将讨论这两个概念。

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
