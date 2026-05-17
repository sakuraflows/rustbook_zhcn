## 高级函数与闭包

本节探讨与函数和闭包相关的一些高级特性，包括函数指针和返回闭包。

### 函数指针

我们已经讨论过如何将闭包传递给函数；你也可以将常规函数传递给函数！当你想要传递一个已经定义好的函数而不是定义一个新的闭包时，这种技术很有用。函数会强制转换为 `fn` 类型（小写 _f_），不要与 `Fn` 闭包 trait 混淆。`fn` 类型被称为*函数指针（function pointer）*。使用函数指针传递函数可以让你将函数作为其他函数的参数使用。

指定参数是函数指针的语法类似于闭包，如清单 20-28 所示，我们定义了一个函数 `add_one`，它将参数加 1。函数 `do_twice` 接受两个参数：一个函数指针，指向任何接受 `i32` 参数并返回 `i32` 的函数；以及一个 `i32` 值。`do_twice` 函数调用函数 `f` 两次，每次传递 `arg` 值，然后将两次函数调用的结果相加。`main` 函数使用参数 `add_one` 和 `5` 调用 `do_twice`。

<Listing number="20-28" file-name="src/main.rs" caption="使用 `fn` 类型接受函数指针作为参数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

此代码打印 `The answer is: 12`。我们指定 `do_twice` 中的参数 `f` 是一个 `fn`，它接受一个 `i32` 类型的参数并返回一个 `i32`。然后我们可以在 `do_twice` 的函数体中调用 `f`。在 `main` 中，我们可以将函数名 `add_one` 作为第一个参数传递给 `do_twice`。

与闭包不同，`fn` 是一种类型而不是 trait，因此我们直接指定 `fn` 作为参数类型，而不是用某个 `Fn` trait 作为 trait 约束来声明泛型类型参数。

函数指针实现了所有三种闭包 trait（`Fn`、`FnMut` 和 `FnOnce`），这意味着你始终可以将函数指针作为参数传递给期望闭包的函数。最好使用泛型类型和其中一个闭包 trait 来编写函数，这样你的函数就可以接受函数或闭包。

话虽如此，你只想接受 `fn` 而不接受闭包的一个例子是与没有闭包的外部代码交互时：C 函数可以接受函数作为参数，但 C 没有闭包。

作为一个可以使用内联定义的闭包或命名函数的例子，我们来看一下标准库中 `Iterator` trait 提供的 `map` 方法的一个用法。要使用 `map` 方法将数字向量转换为字符串向量，我们可以使用闭包，如清单 20-29 所示。

<Listing number="20-29" caption="将闭包与 `map` 方法一起使用，将数字转换为字符串">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

或者，我们可以将一个命名函数作为 `map` 的参数，而不是闭包。清单 20-30 展示了这种形式。

<Listing number="20-30" caption="使用 `String::to_string` 函数结合 `map` 方法将数字转换为字符串">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

请注意，我们必须使用在["高级 Trait"][advanced-traits]<!-- ignore -->部分中讨论的完全限定语法，因为有多个名为 `to_string` 的函数可用。

在这里，我们使用了定义在 `ToString` trait 中的 `to_string` 函数，标准库为任何实现了 `Display` 的类型实现了该 trait。

回顾第 6 章中的["枚举值"][enum-values]<!-- ignore -->部分，我们定义的每个枚举变体的名称也是一个初始化函数。我们可以将这些初始化函数用作实现闭包 trait 的函数指针，这意味着我们可以将初始化函数指定为接受闭包的方法的参数，如清单 20-31 所示。

<Listing number="20-31" caption="使用枚举初始化函数与 `map` 方法从数字创建 `Status` 实例">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

在这里，我们通过使用 `Status::Value` 的初始化函数，使用 `map` 调用的范围内每个 `u32` 值创建了 `Status::Value` 实例。有些人喜欢这种风格，有些人则喜欢使用闭包。它们编译成相同的代码，因此使用你觉得更清晰的风格即可。

### 返回闭包

闭包由 trait 表示，这意味着你不能直接返回闭包。在大多数情况下，你可能想要返回一个 trait，你可以使用实现了该 trait 的具体类型作为函数的返回值。但通常你不能对闭包这样做，因为它们没有可以返回的具体类型；例如，如果闭包从其作用域捕获了任何值，则不允许使用函数指针 `fn` 作为返回类型。

相反，你通常会使用我们在第 10 章学到的 `impl Trait` 语法。你可以使用 `Fn`、`FnOnce` 和 `FnMut` 返回任何函数类型。例如，清单 20-32 中的代码将编译通过。

<Listing number="20-32" caption="使用 `impl Trait` 语法从函数返回闭包">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

然而，正如我们在第 13 章的["推断和标注闭包类型"][closure-types]<!-- ignore -->部分中指出的，每个闭包也是其自身的独特类型。如果你需要处理多个具有相同签名但不同实现的函数，则需要对它们使用 trait 对象。考虑如果你编写类似清单 20-33 所示的代码会发生什么。

<Listing file-name="src/main.rs" number="20-33" caption="创建由返回 `impl Fn` 类型的函数定义的闭包的 `Vec<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

这里我们有两个函数，`returns_closure` 和 `returns_initialized_closure`，它们都返回 `impl Fn(i32) -> i32`。请注意，它们返回的闭包是不同的，即使它们实现了相同的类型。如果我们尝试编译，Rust 会告诉我们它无法工作：

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

错误消息告诉我们，每当我们返回 `impl Trait` 时，Rust 会创建一个唯一的*不透明类型（opaque type）*，这是一种我们无法看到 Rust 为我们构造的细节的类型，也无法猜测 Rust 会生成什么类型来自己编写。因此，即使这些函数返回实现相同 trait `Fn(i32) -> i32` 的闭包，Rust 为每个函数生成的不透明类型也是不同的。（这类似于 Rust 如何为不同的异步块生成不同的具体类型，即使它们具有相同的输出类型，正如我们在第 17 章的["`Pin` 类型和 `Unpin` Trait"][future-types]<!-- ignore -->中看到的。）我们已经见过这个问题的解决方案几次了：我们可以使用 trait 对象，如清单 20-34 所示。

<Listing number="20-34" caption="创建由返回 `Box<dyn Fn>` 的函数定义的闭包的 `Vec<T>`，使它们具有相同的类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

这段代码将编译通过。有关 trait 对象的更多信息，请参阅第 18 章中的["使用 Trait 对象对共享行为进行抽象"][trait-objects]<!-- ignore -->部分。

接下来，让我们看看宏！

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html
