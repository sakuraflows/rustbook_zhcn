## 宏

在本书中，我们一直在使用像 `println!` 这样的宏，但我们还没有完全探讨什么是宏以及它是如何工作的。术语*宏（macro）*指的是 Rust 中的一系列特性——使用 `macro_rules!` 的声明式宏（declarative macros）和三种过程宏（procedural macros）：

- 自定义 `#[derive]` 宏，指定用 `derive` 属性添加的代码，用于结构体和枚举
- 类属性宏（Attribute-like macros），定义可用于任何项的自定义属性
- 类函数宏（Function-like macros），看起来像函数调用但操作指定为参数的标记（tokens）

我们将依次讨论每种宏，但首先，让我们看看为什么在已经有函数的情况下还需要宏。

### 宏与函数的区别

从根本上说，宏是一种编写能生成其他代码的代码的方式，这被称为*元编程（metaprogramming）*。在附录 C 中，我们讨论了 `derive` 属性，它会为你生成各种 trait 的实现。我们在本书中也使用了 `println!` 和 `vec!` 宏。所有这些宏都会*展开（expand）*，生成比你手动编写的更多的代码。

元编程对于减少你必须编写和维护的代码量非常有用，这也是函数的作用之一。然而，宏具有一些函数没有的额外能力。

函数签名必须声明函数的参数数量和类型。而宏则可以接受可变数量的参数：我们可以用一个参数调用 `println!("hello")`，或者用两个参数调用 `println!("hello {}", name)`。此外，宏在编译器解释代码含义之前展开，因此宏可以例如在给定类型上实现 trait。函数不能这样做，因为它在运行时被调用，而 trait 需要在编译时实现。

实现宏而非函数的缺点是，宏定义比函数定义更复杂，因为你编写的是生成 Rust 代码的 Rust 代码。由于这种间接性，宏定义通常比函数定义更难阅读、理解和维护。

宏和函数之间的另一个重要区别是，你必须在文件中*之前*定义宏或将其引入作用域，然后才能调用它，而函数则可以在任何位置定义并在任何位置调用。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### 用于通用元编程的声明式宏

Rust 中应用最广泛的宏形式是*声明式宏（declarative macro）*。这些有时也被称为"示例宏（macros by example）"、"`macro_rules!` 宏"或简称为"宏"。其核心是，声明式宏允许你编写类似于 Rust `match` 表达式的东西。如第 6 章所述，`match` 表达式是一种控制结构，它接受一个表达式，将表达式的结果值与模式进行比较，然后运行与匹配模式关联的代码。宏也将一个值与与特定代码关联的模式进行比较：在这种情况下，该值是传递给宏的字面 Rust 源代码；模式与这些源代码的结构进行比较；当匹配时，与每个模式关联的代码会替换传递给宏的代码。这一切都发生在编译期间。

要定义一个宏，你可以使用 `macro_rules!` 结构。让我们通过查看 `vec!` 宏的定义来探索如何使用 `macro_rules!`。第 8 章介绍了如何使用 `vec!` 宏创建一个包含特定值的新向量。例如，以下宏创建一个包含三个整数的新向量：

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

我们也可以使用 `vec!` 宏创建包含两个整数或五个字符串切片的向量。我们不能使用函数来做同样的事情，因为我们无法预先知道值的数量或类型。

清单 20-35 显示了 `vec!` 宏的一个略微简化的定义。

<Listing number="20-35" file-name="src/lib.rs" caption="简化版的 `vec!` 宏定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> 注意：标准库中 `vec!` 宏的实际定义包括预先分配正确内存量的代码。我们在这里不包含这些代码，它是一个优化，以使示例更简单。

`#[macro_export]` 标注表明，只要定义了该宏的 crate 被引入作用域，该宏就应该可用。如果没有这个标注，宏就不能被引入作用域。

然后，我们以 `macro_rules!` 和我们正在定义的宏名称（*不带*感叹号）开始宏定义。名称（本例中为 `vec`）后面跟着花括号，表示宏定义的主体。

`vec!` 主体中的结构类似于 `match` 表达式的结构。这里我们有一个分支，模式为 `( $( $x:expr ),* )`，后跟 `=>` 和与此模式关联的代码块。如果模式匹配，将发出关联的代码块。由于这是此宏中唯一的模式，因此只有一种有效的匹配方式；任何其他模式都会导致错误。更复杂的宏会有多个分支。

宏定义中有效的模式语法与第 19 章中介绍的模式语法不同，因为宏模式是与 Rust 代码结构而不是值进行匹配。让我们逐步分析清单 20-29 中的模式各部分含义；完整的宏模式语法请参阅 [Rust 参考文档][ref]。

首先，我们使用一组圆括号来包含整个模式。我们使用美元符号（`$`）在宏系统中声明一个变量，该变量将包含与模式匹配的 Rust 代码。美元符号使这一点很明确：这是宏变量而非常规 Rust 变量。接下来是一组圆括号，用于捕获与括号内模式匹配的值，以便在替换代码中使用。在 `$()` 内部是 `$x:expr`，它匹配任何 Rust 表达式，并将该表达式命名为 `$x`。

`$()` 后面的逗号表示，在匹配 `$()` 内代码的每个实例之间必须出现一个字面逗号分隔符字符。`*` 指定该模式匹配零个或多个 `*` 之前的任何内容。

当我们使用 `vec![1, 2, 3];` 调用此宏时，`$x` 模式会与三个表达式 `1`、`2` 和 `3` 匹配三次。

现在让我们看看与此分支关联的代码体内的模式：`$()*` 内的 `temp_vec.push()` 会为每个匹配 `$()` 的部分生成，次数取决于模式匹配的次数（零次或多次）。`$x` 被替换为每个匹配的表达式。当我们使用 `vec![1, 2, 3];` 调用此宏时，替换此宏调用生成的代码如下：

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

我们定义了一个可以接受任意数量任意类型参数的宏，并且可以生成创建包含指定元素的向量的代码。

要了解更多关于如何编写宏的信息，请查阅在线文档或其他资源，例如 Daniel Keep 始创、Lukas Wirth 继续维护的["The Little Book of Rust Macros"][tlborm]。

### 用于从属性生成代码的过程宏

第二种宏形式是过程宏（procedural macro），它更像一个函数（并且是一种过程）。*过程宏*接受一些代码作为输入，对这些代码进行操作，并产生一些代码作为输出，而不是像声明式宏那样匹配模式并用其他代码替换代码。三种过程宏是自定义 `derive` 宏、类属性宏和类函数宏，它们都以类似的方式工作。

创建过程宏时，定义必须位于具有特殊 crate 类型的自己的 crate 中。这是因为复杂的技术原因，我们希望在将来消除这些原因。在清单 20-36 中，我们展示了如何定义一个过程宏，其中 `some_attribute` 是使用特定宏变种的占位符。

<Listing number="20-36" file-name="src/lib.rs" caption="定义过程宏的示例">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

定义过程宏的函数接受一个 `TokenStream` 作为输入，并产生一个 `TokenStream` 作为输出。`TokenStream` 类型由 Rust 自带的 `proc_macro` crate 定义，表示一系列标记（tokens）。这是宏的核心：宏操作的源代码构成输入 `TokenStream`，而宏产生的代码是输出 `TokenStream`。该函数还附加了一个属性，指定我们正在创建哪种过程宏。我们可以在同一个 crate 中拥有多种过程宏。

让我们看看不同类型的过程宏。我们将从自定义 `derive` 宏开始，然后解释使其他形式不同的细微差异。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="how-to-write-a-custom-derive-macro"></a>

### 自定义 `derive` 宏

让我们创建一个名为 `hello_macro` 的 crate，它定义了一个名为 `HelloMacro` 的 trait，其中带有一个名为 `hello_macro` 的关联函数。我们不会让用户为他们的每种类型都实现 `HelloMacro` trait，而是提供一个过程宏，以便用户可以用 `#[derive(HelloMacro)]` 标注其类型，从而获得 `hello_macro` 函数的默认实现。默认实现将打印 `Hello, Macro! My name is TypeName!`，其中 `TypeName` 是定义了该 trait 的类型名称。换句话说，我们将编写一个 crate，使另一个程序员能够使用我们的 crate 编写类似清单 20-37 的代码。

<Listing number="20-37" file-name="src/main.rs" caption="我们 crate 的用户在使用我们的过程宏时能够编写的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

当我们完成时，这段代码将打印 `Hello, Macro! My name is Pancakes!`。第一步是创建一个新的库 crate，如下所示：

```console
$ cargo new hello_macro --lib
```

接下来，在清单 20-38 中，我们将定义 `HelloMacro` trait 及其关联函数。

<Listing file-name="src/lib.rs" number="20-38" caption="一个将与 `derive` 宏一起使用的简单 trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

我们有一个 trait 及其函数。此时，我们的 crate 用户可以实现该 trait 以获得所需功能，如清单 20-39 所示。

<Listing number="20-39" file-name="src/main.rs" caption="如果用户手动实现 `HelloMacro` trait 会是什么样子">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

然而，他们需要为每个想要使用 `hello_macro` 的类型编写实现块；我们希望免去他们做这项工作。

此外，我们还不能提供能够打印实现该 trait 的类型名称的 `hello_macro` 函数默认实现：Rust 没有反射（reflection）能力，因此它无法在运行时查找类型名称。我们需要一个宏在编译时生成代码。

下一步是定义过程宏。在撰写本文时，过程宏需要位于它们自己的 crate 中。最终，这个限制可能会被解除。组织 crate 和宏 crate 的约定如下：对于一个名为 `foo` 的 crate，自定义 `derive` 过程宏 crate 称为 `foo_derive`。让我们在 `hello_macro` 项目中启动一个名为 `hello_macro_derive` 的新 crate：

```console
$ cargo new hello_macro_derive --lib
```

我们的两个 crate 紧密相关，因此我们在 `hello_macro` crate 的目录中创建过程宏 crate。如果我们更改 `hello_macro` 中的 trait 定义，我们将不得不同时更改 `hello_macro_derive` 中过程宏的实现。这两个 crate 需要分别发布，使用这些 crate 的程序员需要将两者都添加为依赖并将它们都引入作用域。我们也可以让 `hello_macro` crate 将 `hello_macro_derive` 作为依赖并使用并重新导出过程宏代码。然而，我们这样组织项目的方式使得即使程序员不想使用 `derive` 功能，也可以使用 `hello_macro`。

我们需要将 `hello_macro_derive` crate 声明为过程宏 crate。我们还需要来自 `syn` 和 `quote` crate 的功能，你稍后会看到，因此我们需要将它们添加为依赖。将以下内容添加到 `hello_macro_derive` 的 _Cargo.toml_ 文件中：

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

要开始定义过程宏，请将清单 20-40 中的代码放入 `hello_macro_derive` crate 的 _src/lib.rs_ 文件中。请注意，在添加 `impl_hello_macro` 函数的定义之前，此代码将无法编译。

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="大多数过程宏 crate 处理 Rust 代码所需的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

注意，我们将代码拆分为 `hello_macro_derive` 函数（负责解析 `TokenStream`）和 `impl_hello_macro` 函数（负责转换语法树）：这使得编写过程宏更加方便。外部函数（这里指 `hello_macro_derive`）中的代码对于你看到或创建的大多数过程宏 crate 来说都是相同的。你在内部函数（这里指 `impl_hello_macro`）主体中指定的代码将根据你的过程宏目的而有所不同。

我们引入了三个新的 crate：`proc_macro`、[`syn`][syn]<!-- ignore --> 和 [`quote`][quote]<!-- ignore -->。`proc_macro` crate 随 Rust 自带，因此我们不需要将其添加到 _Cargo.toml_ 的依赖中。`proc_macro` crate 是编译器的 API，允许我们从我们的代码中读取和操作 Rust 代码。

`syn` crate 将 Rust 代码从字符串解析为我们可以执行操作的数据结构。`quote` crate 将 `syn` 的数据结构转换回 Rust 代码。这些 crate 使得解析我们可能想要处理的任何类型的 Rust 代码变得更加简单：为 Rust 代码编写完整的解析器不是一项简单的任务。

当我们的库用户在一个类型上指定 `#[derive(HelloMacro)]` 时，`hello_macro_derive` 函数将被调用。这是可能的，因为我们在这里用 `proc_macro_derive` 标注了 `hello_macro_derive` 函数，并指定了名称 `HelloMacro`，它与我们的 trait 名称匹配；这是大多数过程宏遵循的约定。

`hello_macro_derive` 函数首先将 `input` 从 `TokenStream` 转换为我们能够解释并执行操作的数据结构。这就是 `syn` 发挥作用的地方。`syn` 中的 `parse` 函数接受一个 `TokenStream` 并返回一个 `DeriveInput` 结构体，表示解析后的 Rust 代码。清单 20-41 显示了从解析 `struct Pancakes;` 字符串得到的 `DeriveInput` 结构体的相关部分。

<Listing number="20-41" caption="解析具有宏属性的清单 20-37 中的代码时得到的 `DeriveInput` 实例">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

该结构体的字段显示我们解析的 Rust 代码是一个标识符（`ident`，即名称）为 `Pancakes` 的单元结构体（unit struct）。该结构体上还有更多用于描述各种 Rust 代码的字段；请查看 [`syn` 文档中的 `DeriveInput`][syn-docs] 以获取更多信息。

稍后我们将定义 `impl_hello_macro` 函数，这是我们构建想要包含的新 Rust 代码的地方。但在此之前，请注意，我们的 `derive` 宏的输出也是一个 `TokenStream`。返回的 `TokenStream` 被添加到我们的 crate 用户编写的代码中，因此当他们编译自己的 crate 时，他们将获得我们在修改后的 `TokenStream` 中提供的额外功能。

你可能已经注意到，我们在这里调用 `unwrap` 来使 `hello_macro_derive` 函数在调用 `syn::parse` 函数失败时 panic。过程宏在错误时 panic 是必要的，因为 `proc_macro_derive` 函数必须返回 `TokenStream` 而不是 `Result`，以符合过程宏 API。我们通过使用 `unwrap` 简化了此示例；在生产代码中，你应该使用 `panic!` 或 `expect` 提供关于出错的更具体错误消息。

现在我们有了将标注的 Rust 代码从 `TokenStream` 转换为 `DeriveInput` 实例的代码，让我们生成在被标注类型上实现 `HelloMacro` trait 的代码，如清单 20-42 所示。

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="使用解析的 Rust 代码实现 `HelloMacro` trait">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

我们使用 `ast.ident` 获取一个包含被标注类型名称（标识符）的 `Ident` 结构体实例。清单 20-41 中的结构体显示，当我们在清单 20-37 的代码上运行 `impl_hello_macro` 函数时，我们得到的 `ident` 将具有值为 `"Pancakes"` 的 `ident` 字段。因此，清单 20-42 中的 `name` 变量将包含一个 `Ident` 结构体实例，打印时将是字符串 `"Pancakes"`，即清单 20-37 中结构体的名称。

`quote!` 宏让我们定义想要返回的 Rust 代码。编译器期望的结果与 `quote!` 宏直接执行的结果不同，因此我们需要将其转换为 `TokenStream`。我们通过调用 `into` 方法来实现，该方法消费这个中间表示并返回所需的 `TokenStream` 类型值。

`quote!` 宏还提供了一些非常酷的模板机制：我们可以输入 `#name`，`quote!` 将用变量 `name` 中的值替换它。你甚至可以做一些类似于常规宏工作的重复。请查看 [`quote` crate 的文档][quote-docs]以获取全面的介绍。

我们希望我们的过程宏为用户标注的类型生成 `HelloMacro` trait 的实现，我们可以通过使用 `#name` 来获得。trait 实现有一个函数 `hello_macro`，其函数体包含我们想要提供的功能：打印 `Hello, Macro! My name is`，然后是被标注类型的名称。

这里使用的 `stringify!` 宏是 Rust 内置的。它获取一个 Rust 表达式，例如 `1 + 2`，并在编译时将该表达式转换为字符串字面量，例如 `"1 + 2"`。这与 `format!` 或 `println!` 不同，后两者是求值表达式然后将结果转换为 `String` 的宏。`#name` 输入有可能是一个要逐字打印的表达式，因此我们使用 `stringify!`。使用 `stringify!` 还通过在编译时将 `#name` 转换为字符串字面量来节省一次分配。

此时，`cargo build` 应该在 `hello_macro` 和 `hello_macro_derive` 中都成功完成。让我们将这些 crate 连接到清单 20-37 中的代码，看看过程宏的运行效果！使用 `cargo new pancakes` 在你的 _projects_ 目录中创建一个新的二进制项目。我们需要在 `pancakes` crate 的 _Cargo.toml_ 中将 `hello_macro` 和 `hello_macro_derive` 添加为依赖。如果你将你的 `hello_macro` 和 `hello_macro_derive` 版本发布到 [crates.io](https://crates.io/)<!-- ignore -->，它们将是常规依赖；如果没有，你可以将它们指定为 `path` 依赖，如下所示：

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

将清单 20-37 中的代码放入 _src/main.rs_ 中，然后运行 `cargo run`：它应该打印 `Hello, Macro! My name is Pancakes!`。来自过程宏的 `HelloMacro` trait 的实现已被包含，而无需 `pancakes` crate 自行实现；`#[derive(HelloMacro)]` 添加了 trait 的实现。

接下来，让我们探讨其他类型的过程宏与自定义 `derive` 宏的不同之处。

### 类属性宏

类属性宏（Attribute-like macros）类似于自定义 `derive` 宏，但它们允许你创建新的属性，而不是为 `derive` 属性生成代码。它们也更加灵活：`derive` 仅适用于结构体和枚举；属性也可以应用于其他项，例如函数。以下是一个使用类属性宏的示例。假设你有一个名为 `route` 的属性，在使用 Web 应用程序框架时标注函数：

```rust,ignore
#[route(GET, "/")]
fn index() {
```

这个 `#[route]` 属性将由框架定义为过程宏。宏定义函数的签名如下所示：

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

这里，我们有两个 `TokenStream` 类型的参数。第一个用于属性的内容：`GET, "/"` 部分。第二个是属性所附加的项的主体：这里指 `fn index() {}` 以及函数体的其余部分。

除此之外，类属性宏的工作方式与自定义 `derive` 宏相同：你需要创建一个具有 `proc-macro` crate 类型的 crate，并实现一个生成你所需代码的函数！

### 类函数宏

类函数宏（Function-like macros）定义了看起来像函数调用的宏。与 `macro_rules!` 宏类似，它们比函数更灵活；例如，它们可以接受未知数量的参数。然而，`macro_rules!` 宏只能使用我们之前在["用于通用元编程的声明式宏"][decl]<!-- ignore -->部分讨论的类似匹配的语法来定义。类函数宏接受一个 `TokenStream` 参数，并且它们的定义使用 Rust 代码操作该 `TokenStream`，正如其他两种过程宏所做的那样。类函数宏的一个例子是 `sql!` 宏，它可能像这样被调用：

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

这个宏将解析其中的 SQL 语句并检查其语法是否正确，这比 `macro_rules!` 宏能做的处理要复杂得多。`sql!` 宏的定义如下：

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

这个定义类似于自定义 `derive` 宏的签名：我们接收括号内的标记（tokens），并返回我们想要生成的代码。

## 总结

呼！现在你的工具箱中有了一些你可能不常用的 Rust 特性，但你会知道它们在非常特定的情况下是可用的。我们介绍了几个复杂的主题，这样当你在错误消息建议中或他人的代码中遇到它们时，你将能够识别这些概念和语法。将本章作为指导你找到解决方案的参考。

接下来，我们将把本书中讨论的所有内容付诸实践，再做最后一个项目！

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
