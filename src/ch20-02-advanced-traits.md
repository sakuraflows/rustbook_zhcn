## 高级 Trait

我们第一次介绍 trait 是在第 10 章的["使用 Trait 定义共享行为"][traits]<!-- ignore -->部分，但当时我们没有讨论更高级的细节。既然你对 Rust 有了更多了解，我们可以深入探讨其细节了。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### 使用关联类型定义 Trait

*关联类型（Associated types）*将一个类型占位符与 trait 连接起来，使得 trait 方法定义可以在其签名中使用这些占位符类型。trait 的实现者将为特定实现指定要使用的具体类型，以替代占位符类型。这样，我们可以定义一个使用某些类型的 trait，而无需在 trait 被实现之前确切知道这些类型是什么。

我们在本章中将大多数高级特性描述为很少需要用到。关联类型处于中间位置：它们的使用比本书其余部分解释的特性更少，但比本章讨论的许多其他特性更常见。

一个具有关联类型的 trait 的例子是标准库提供的 `Iterator` trait。关联类型名为 `Item`，代表实现 `Iterator` trait 的类型正在迭代的值的类型。`Iterator` trait 的定义如清单 20-13 所示。

<Listing number="20-13" caption="具有关联类型 `Item` 的 `Iterator` trait 的定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

类型 `Item` 是一个占位符，`next` 方法的定义表明它将返回 `Option<Self::Item>` 类型的值。`Iterator` trait 的实现者将为 `Item` 指定具体类型，并且 `next` 方法将返回一个包含该具体类型值的 `Option`。

关联类型可能看起来与泛型概念相似，因为后者允许我们定义函数而不指定它可以处理什么类型。为了检查这两个概念之间的区别，我们来看一个在名为 `Counter` 的类型上实现 `Iterator` trait 的例子，该类型指定 `Item` 类型为 `u32`：

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

这种语法看起来与泛型语法类似。那么，为什么不像清单 20-14 那样直接用泛型定义 `Iterator` trait 呢？

<Listing number="20-14" caption="使用泛型的假设性 `Iterator` trait 定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

区别在于，当使用泛型时（如清单 20-14），我们必须在每个实现中标注类型；因为我们也可以为 `Counter` 实现 `Iterator<String>` 或任何其他类型，我们可能为 `Counter` 拥有多个 `Iterator` 的实现。换句话说，当 trait 有一个泛型参数时，它可以为一个类型实现多次，每次改变泛型类型参数的具体类型。当我们在 `Counter` 上使用 `next` 方法时，我们必须提供类型标注，以指示我们想要使用哪个 `Iterator` 实现。

使用关联类型，我们不需要标注类型，因为我们不能为一个类型多次实现一个 trait。在清单 20-13 使用关联类型的定义中，我们只能选择一次 `Item` 的类型，因为只能有一个 `impl Iterator for Counter`。我们不必在每次对 `Counter` 调用 `next` 时都指定我们想要一个 `u32` 值的迭代器。

关联类型也成为了 trait 契约的一部分：trait 的实现者必须提供一个类型来替代关联类型占位符。关联类型通常有一个描述该类型将如何被使用的名称，并且在 API 文档中记录关联类型是一个好的做法。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### 使用默认泛型参数和运算符重载

当我们使用泛型类型参数时，我们可以为泛型类型指定一个默认的具体类型。如果默认类型可行，这就消除了 trait 的实现者指定具体类型的需要。你可以使用 `<PlaceholderType=ConcreteType>` 语法在声明泛型类型时指定一个默认类型。

一个这种技术有用的情况很好的例子是*运算符重载（operator overloading）*，其中你在特定情况下自定义运算符（例如 `+`）的行为。

Rust 不允许你创建自己的运算符或重载任意运算符。但你可以通过实现与运算符关联的 trait 来重载 `std::ops` 中列出的操作和相应的 trait。例如，在清单 20-15 中，我们重载了 `+` 运算符来将两个 `Point` 实例相加。我们通过在 `Point` 结构体上实现 `Add` trait 来实现这一点。

<Listing number="20-15" file-name="src/main.rs" caption="实现 `Add` trait 以重载 `Point` 实例的 `+` 运算符">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` 方法将两个 `Point` 实例的 `x` 值和两个 `Point` 实例的 `y` 值相加，创建一个新的 `Point`。`Add` trait 有一个名为 `Output` 的关联类型，它决定了从 `add` 方法返回的类型。

这段代码中的默认泛型类型在 `Add` trait 内部。以下是它的定义：

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

这段代码看起来应该大致熟悉：一个带有一个方法和一个关联类型的 trait。新的部分是 `Rhs=Self`：这种语法被称为*默认类型参数（default type parameters）*。`Rhs` 泛型类型参数（"右操作数（right-hand side）"的缩写）定义了 `add` 方法中 `rhs` 参数的类型。如果我们在实现 `Add` trait 时没有为 `Rhs` 指定具体类型，`Rhs` 的类型将默认为 `Self`，也就是我们正在实现 `Add` 的类型。

当我们为 `Point` 实现 `Add` 时，我们使用了 `Rhs` 的默认值，因为我们想要添加两个 `Point` 实例。让我们看一个想要自定义 `Rhs` 类型而不是使用默认值的 `Add` trait 实现示例。

我们有两个结构体 `Millimeters` 和 `Meters`，它们持有不同单位的值。这种将现有类型薄薄地包装在另一个结构体中的方式被称为*新类型模式（newtype pattern）*，我们将在["使用新类型模式实现外部 Trait"][newtype]<!-- ignore -->部分更详细地描述。我们想将毫米（Millimeters）值加到米（Meters）值上，并且让 `Add` 的实现正确地进行转换。我们可以用 `Meters` 作为 `Rhs` 为 `Millimeters` 实现 `Add`，如清单 20-16 所示。

<Listing number="20-16" file-name="src/lib.rs" caption="在 `Millimeters` 上实现 `Add` trait 以将 `Millimeters` 和 `Meters` 相加">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

要相加 `Millimeters` 和 `Meters`，我们指定 `impl Add<Meters>` 来设置 `Rhs` 类型参数的值，而不是使用默认的 `Self`。

你将在两种主要情况下使用默认类型参数：

1. 在不破坏现有代码的情况下扩展类型
2. 在大多数用户不需要的特定情况下允许自定义

标准库的 `Add` trait 是第二个目的的示例：通常，你会将两个同类类型相加，但 `Add` trait 提供了超越这一点的自定义能力。在 `Add` trait 定义中使用默认类型参数意味着大多数情况下你不需要指定额外的参数。换句话说，不需要一些实现样板，使得使用 trait 更容易。

第一个目的与第二个类似，但是反向的：如果你想向一个现有 trait 添加类型参数，你可以给它一个默认值，从而允许扩展 trait 的功能而不破坏现有的实现代码。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### 消除同名方法之间的歧义

在 Rust 中，没有什么能阻止一个 trait 拥有与另一个 trait 方法同名的方法，Rust 也不阻止你在同一类型上实现这两个 trait。在同一类型上直接实现一个与 trait 方法同名的方法也是可能的。

当调用同名的方法时，你需要告诉 Rust 你想要使用哪一个。考虑清单 20-17 中的代码，我们定义了两个 trait `Pilot` 和 `Wizard`，它们都有一个名为 `fly` 的方法。然后，我们在一个已经直接实现了一个名为 `fly` 的方法的 `Human` 类型上实现这两个 trait。每个 `fly` 方法做的事情都不同。

<Listing number="20-17" file-name="src/main.rs" caption="定义了两个具有 `fly` 方法的 trait，在 `Human` 类型上实现了它们，并且在 `Human` 上直接实现了一个 `fly` 方法。">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

当我们在 `Human` 实例上调用 `fly` 时，编译器默认会调用直接实现在该类型上的方法，如清单 20-18 所示。

<Listing number="20-18" file-name="src/main.rs" caption="在 `Human` 实例上调用 `fly`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

运行这段代码将打印 `*waving arms furiously*`，表明 Rust 直接调用了实现在 `Human` 上的 `fly` 方法。

要调用 `Pilot` trait 或 `Wizard` trait 中的 `fly` 方法，我们需要使用更明确的语法来指定我们指的是哪个 `fly` 方法。清单 20-19 演示了这种语法。

<Listing number="20-19" file-name="src/main.rs" caption="指定我们想要调用哪个 trait 的 `fly` 方法">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

在方法名称之前指定 trait 名称，向 Rust 阐明了我们想要调用哪个 `fly` 实现。我们也可以写成 `Human::fly(&person)`，这与我们在清单 20-19 中使用的 `person.fly()` 等效，但如果我们不需要消除歧义，这写起来会长一些。

运行这段代码会打印以下内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

因为 `fly` 方法接受一个 `self` 参数，如果我们有两个实现了同一个 trait 的*类型*，Rust 可以根据 `self` 的类型确定要使用哪个 trait 实现。

然而，不是方法的关联函数没有 `self` 参数。当有多个类型或 trait 定义具有相同函数名的非方法函数时，Rust 并不总是知道你的意思，除非你使用完全限定语法。例如，在清单 20-20 中，我们为一家动物收容所创建了一个 trait，该收容所想给所有幼犬取名为 Spot。我们创建了一个 `Animal` trait，其中包含一个关联的非方法函数 `baby_name`。`Animal` trait 为 `Dog` 结构体实现，同时我们也在 `Dog` 上直接提供了一个同名的关联非方法函数 `baby_name`。

<Listing number="20-20" file-name="src/main.rs" caption="一个带有关联函数的 trait 和一个带有同名关联函数且也实现了该 trait 的类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

我们在定义在 `Dog` 上的 `baby_name` 关联函数中实现了给所有小狗命名为 Spot 的代码。`Dog` 类型也实现了 `Animal` trait，它描述了所有动物都具有的特征。幼犬被称为 puppy，这是在 `Dog` 上实现的 `Animal` trait 中、与 `Animal` trait 关联的 `baby_name` 函数中表达的。

在 `main` 中，我们调用了 `Dog::baby_name` 函数，它直接调用了定义在 `Dog` 上的关联函数。这段代码打印如下：

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

这个输出不是我们想要的。我们想要调用作为 `Animal` trait 一部分的 `baby_name` 函数，该 trait 我们在 `Dog` 上实现了，这样代码会打印 `A baby dog is called a puppy`。我们在清单 20-19 中使用的指定 trait 名称的技术在这里没有帮助；如果我们把 `main` 改成清单 20-21 中的代码，我们会得到一个编译错误。

<Listing number="20-21" file-name="src/main.rs" caption="尝试从 `Animal` trait 调用 `baby_name` 函数，但 Rust 不知道要使用哪个实现">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

因为 `Animal::baby_name` 没有 `self` 参数，并且可能还有其他类型实现了 `Animal` trait，Rust 无法确定我们想要哪个 `Animal::baby_name` 实现。我们将会得到这个编译器错误：

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

为了消除歧义并告诉 Rust 我们想要使用 `Animal` 对 `Dog` 的实现，而不是 `Animal` 对其他类型的实现，我们需要使用完全限定语法。清单 20-22 演示了如何使用完全限定语法。

<Listing number="20-22" file-name="src/main.rs" caption="使用完全限定语法指定我们要调用 `Animal` trait 的 `baby_name` 函数在 `Dog` 上的实现">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

我们在尖括号内为 Rust 提供了一个类型标注，通过说我们想将 `Dog` 类型在此函数调用中视为 `Animal`，来指示我们想要调用 `Animal` trait 在 `Dog` 上实现的 `baby_name` 方法。这段代码现在将打印我们想要的内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

通常来说，完全限定语法定义如下：

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

对于不是方法的关联函数，不会有 `receiver`：只有其他参数的列表。你可以在任何调用函数或方法的地方使用完全限定语法。然而，对于 Rust 可以从程序中的其他信息确定的部分，你可以省略此语法。你只需要在存在多个使用相同名称的实现，并且 Rust 需要帮助才能识别你想要的实现的情况下使用这种更冗长的语法。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### 使用超 Trait

有时你可能会编写一个依赖于另一个 trait 的 trait 定义：要一个类型实现第一个 trait，你想要求该类型也必须实现第二个 trait。你这样做是为了让你的 trait 定义能够使用第二个 trait 的关联项。你的 trait 定义所依赖的 trait 被称为你的 trait 的*超 trait（supertrait）*。

例如，假设我们想要创建一个 `OutlinePrint` trait，它有一个 `outline_print` 方法，将给定值格式化为带星号边框的形式打印出来。也就是说，给定一个实现了标准库 `Display` trait 的 `Point` 结构体，其结果格式为 `(x, y)`，当我们在一个 `x` 为 `1`、`y` 为 `3` 的 `Point` 实例上调用 `outline_print` 时，它应该打印如下：

```text
**********
*        *
* (1, 3) *
*        *
**********
```

在 `outline_print` 方法的实现中，我们想要使用 `Display` trait 的功能。因此，我们需要指定 `OutlinePrint` trait 只对也实现了 `Display` 并提供 `OutlinePrint` 所需功能的类型起作用。我们可以在 trait 定义中通过指定 `OutlinePrint: Display` 来实现这一点。这种技术类似于向 trait 添加一个 trait 约束。清单 20-23 展示了 `OutlinePrint` trait 的实现。

<Listing number="20-23" file-name="src/main.rs" caption="实现需要 `Display` 功能的 `OutlinePrint` trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

因为我们指定了 `OutlinePrint` 需要 `Display` trait，我们可以使用 `to_string` 函数，该函数会自动为任何实现了 `Display` 的类型实现。如果我们试图使用 `to_string` 而没有在 trait 名称后面添加冒号和指定 `Display` trait，我们会得到一个错误，指出在当前作用域中未找到类型 `&Self` 的名为 `to_string` 的方法。

让我们看看当我们尝试在不实现 `Display` 的类型（例如 `Point` 结构体）上实现 `OutlinePrint` 时会发生什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

我们得到一个错误，说需要 `Display` 但没有实现：

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

要修复这个问题，我们在 `Point` 上实现 `Display` 并满足 `OutlinePrint` 所需的条件，如下所示：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

然后，在 `Point` 上实现 `OutlinePrint` trait 将成功编译，我们可以在 `Point` 实例上调用 `outline_print` 来在星号边框内显示它。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### 使用新类型模式实现外部 Trait

在第 10 章的["在类型上实现 Trait"][implementing-a-trait-on-a-type]<!-- ignore -->部分，我们提到了孤儿规则（orphan rule），该规则规定我们只有在 trait 或类型（或两者）位于我们自己的 crate 中时，才能在该类型上实现该 trait。可以使用新类型模式绕过这一限制，这涉及在元组结构体中创建一个新类型。（我们在第 5 章的["使用元组结构体创建不同类型"][tuple-structs]<!-- ignore -->部分介绍了元组结构体。）元组结构体将有一个字段，并成为我们希望在其上实现 trait 的类型的薄包装。然后，包装类型位于我们的 crate 中，我们可以在包装器上实现 trait。*新类型（Newtype）*是一个源自 Haskell 编程语言的术语。使用这种模式没有运行时性能损失，并且包装类型在编译时会被优化掉。

例如，假设我们想在 `Vec<T>` 上实现 `Display`，孤儿规则阻止我们直接这样做，因为 `Display` trait 和 `Vec<T>` 类型都定义在我们的 crate 之外。我们可以创建一个持有 `Vec<T>` 实例的 `Wrapper` 结构体；然后，我们可以在 `Wrapper` 上实现 `Display` 并使用 `Vec<T>` 值，如清单 20-24 所示。

<Listing number="20-24" file-name="src/main.rs" caption="创建一个包装 `Vec<String>` 的 `Wrapper` 类型以实现 `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display` 的实现使用 `self.0` 来访问内部的 `Vec<T>`，因为 `Wrapper` 是一个元组结构体，而 `Vec<T>` 是元组中索引为 0 的元素。然后，我们可以在 `Wrapper` 上使用 `Display` trait 的功能。

使用这种技术的缺点是 `Wrapper` 是一个新类型，因此它没有所持有的值的方法。我们必须直接在 `Wrapper` 上实现 `Vec<T>` 的所有方法，以便这些方法委托给 `self.0`，这将允许我们将 `Wrapper` 完全当作 `Vec<T>` 来对待。如果我们希望新类型具有内部类型的每一个方法，在 `Wrapper` 上实现 `Deref` trait 以返回内部类型是一个解决方案（我们在第 15 章的["将智能指针视为常规引用"][smart-pointer-deref]<!-- ignore -->部分讨论了实现 `Deref` trait）。如果我们不希望 `Wrapper` 类型拥有内部类型的所有方法——例如，为了限制 `Wrapper` 类型的行为——我们就必须手动实现我们想要的那些方法。

这种新类型模式即使在涉及 trait 时也很有用。让我们切换焦点，来看看与 Rust 的类型系统交互的一些高级方式。

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
