## 可反驳性：模式是否可能匹配失败

模式有两种形式：可反驳的（refutable）和不可反驳的（irrefutable）。对于任何可能传递的值都会匹配的模式称为*不可反驳的（irrefutable）*。一个例子是 `let x = 5;` 语句中的 `x`，因为 `x` 匹配任何东西，因此不可能匹配失败。对于某些可能的值可能会匹配失败的模式称为*可反驳的（refutable）*。一个例子是 `if let Some(x) = a_value` 表达式中的 `Some(x)`，因为如果 `a_value` 变量中的值是 `None` 而不是 `Some`，那么 `Some(x)` 模式将不会匹配。

函数参数、`let` 语句和 `for` 循环只能接受不可反驳的模式，因为当值不匹配时，程序无法做任何有意义的事情。`if let` 和 `while let` 表达式以及 `let...else` 语句可以接受可反驳和不可反驳的模式，但编译器会对不可反驳的模式发出警告，因为根据定义，它们本意是处理可能的失败：条件语句的功能在于它能够根据成功或失败执行不同的操作。

一般来说，你不需要担心可反驳和不可反驳模式之间的区别；然而，你需要熟悉可反驳性这个概念，以便在看到错误信息时能够应对。在这些情况下，你需要根据代码的预期行为，更改模式或使用该模式的结构。

让我们看一个例子，了解当我们尝试在 Rust 要求不可反驳模式的地方使用可反驳模式，以及反之会发生什么。清单 19-8 展示了一个 `let` 语句，但对于模式，我们指定了 `Some(x)`，这是一个可反驳模式。正如你可能预料的，这段代码将无法编译。

<Listing number="19-8" caption="尝试在 `let` 中使用可反驳模式">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

如果 `some_option_value` 是 `None` 值，它将无法匹配模式 `Some(x)`，这意味着该模式是可反驳的。然而，`let` 语句只能接受不可反驳的模式，因为代码无法对 `None` 值做任何有效的事情。在编译时，Rust 会抱怨我们在需要不可反驳模式的地方使用了可反驳模式：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

因为我们没有（也无法！）用模式 `Some(x)` 覆盖每一个有效值，Rust 理所应当地产生了一个编译器错误。

如果我们在需要不可反驳模式的地方有一个可反驳模式，我们可以通过更改使用该模式的代码来修复它：不使用 `let`，而使用 `let...else`。这样，如果模式不匹配，花括号中的代码将处理该值。清单 19-9 展示了如何修复清单 19-8 中的代码。

<Listing number="19-9" caption="使用 `let...else` 和一个包含可反驳模式的代码块替代 `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

我们给了代码一个出路！这段代码是完全有效的，尽管这意味着我们不能在不收到警告的情况下使用不可反驳的模式。如果我们给 `let...else` 一个始终会匹配的模式，例如 `x`，如清单 19-10 所示，编译器会给出一个警告。

<Listing number="19-10" caption="尝试在 `let...else` 中使用不可反驳模式">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust 会抱怨使用 `let...else` 配合不可反驳模式没有意义：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

由于这个原因，`match` 分支必须使用可反驳模式，除了最后一个分支，它应该使用不可反驳的模式来匹配所有剩余的值。Rust 允许我们在只有一个分支的 `match` 中使用不可反驳模式，但这种语法并不是特别有用，可以用更简单的 `let` 语句替代。

现在你已经知道在哪里使用模式，以及可反驳和不可反驳模式之间的区别，让我们来介绍所有可以用来创建模式的语法。
