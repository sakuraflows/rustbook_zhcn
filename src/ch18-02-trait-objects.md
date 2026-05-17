<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## 使用 Trait 对象对共享行为进行抽象

在第 8 章中，我们提到向量（vector）的一个局限是它们只能存储一种类型的元素。我们在清单 8-9 中创建了一个变通方案，我们定义了一个 `SpreadsheetCell` 枚举，其变体（variant）可以持有整数、浮点数和文本。这意味着我们可以在每个单元格中存储不同类型的数据，同时仍然拥有一个表示一行单元格的向量。当我们的可互换元素是在代码编译时已知的一组固定类型时，这是一个非常好的解决方案。

然而，有时我们希望库的用户能够扩展在特定情况下有效的类型集合。为了展示如何实现这一点，我们将创建一个示例性的图形用户界面（GUI）工具，它遍历一个项目列表，对每个项目调用 `draw` 方法将其绘制到屏幕上——这是 GUI 工具常用的一种技术。我们将创建一个名为 `gui` 的库 crate，其中包含一个 GUI 库的结构。这个 crate 可能包含一些供人们使用的类型，例如 `Button` 或 `TextField`。此外，`gui` 的用户将希望创建他们自己的可绘制类型：例如，一个程序员可能会添加一个 `Image`，而另一个程序员可能会添加一个 `SelectBox`。

在编写这个库的时候，我们无法知道并定义其他程序员可能想要创建的所有类型。但我们确实知道 `gui` 需要跟踪许多不同类型的不同值，并且它需要对这些不同类型的每个值调用 `draw` 方法。它不需要确切知道调用 `draw` 方法时会发生什么，只需要知道该值会有可供调用的 `draw` 方法。

在具有继承的语言中，要完成这个功能，我们可能会定义一个名为 `Component` 的类，其上有一个名为 `draw` 的方法。其他类，如 `Button`、`Image` 和 `SelectBox`，将从 `Component` 继承，从而继承 `draw` 方法。它们每个都可以覆盖 `draw` 方法来定义自己的自定义行为，而框架可以将所有类型视为 `Component` 的实例并对它们调用 `draw`。但由于 Rust 没有继承，我们需要另一种方式来构建 `gui` 库，以允许用户创建与库兼容的新类型。

### 定义公共行为的 Trait

为了实现我们希望 `gui` 拥有的行为，我们将定义一个名为 `Draw` 的 trait，它有一个名为 `draw` 的方法。然后，我们可以定义一个接受 trait 对象的向量。一个*trait 对象（trait object）*既指向一个实现了我们指定 trait 的类型的实例，也指向一个用于在运行时查找该类型上的 trait 方法的表。我们通过指定某种指针（如引用或 `Box<T>` 智能指针），然后跟 `dyn` 关键字，再指定相关的 trait 来创建 trait 对象。（关于 trait 对象为什么必须使用指针的原因，我们将在第 20 章的[“动态大小类型和 `Sized` Trait”][dynamically-sized]<!-- ignore -->中讨论。）我们可以在泛型或具体类型的位置使用 trait 对象。无论我们在何处使用 trait 对象，Rust 的类型系统都将在编译时确保在该上下文中使用的任何值都实现了该 trait 对象的 trait。因此，我们不需要在编译时知道所有可能的类型。

我们在第 8 章中提到，向量只能存储一种类型的一个限制。我们可以通过使用 trait 对象绕过这个限制：我们在向量中可以存储实现了给定 trait 的不同类型的具体类型。例如，在清单 18-3 中，我们定义了一个名为 `Screen` 的结构体，它有一个名为 `components` 的字段，该字段包含一个 `Box<dyn Draw>` 的向量。这个以 `Box<dyn Draw>` 作为元素的向量将容纳任何实现了 `Draw` trait 的类型，并且我们可以在其上调用 `draw` 方法。

<Listing number="18-3" file-name="src/lib.rs" caption="定义 `Screen` 结构体，其 `components` 字段包含实现了 `Draw` trait 的类型的 trait 对象">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

在 `Screen` 结构体上，我们将定义一个名为 `run` 的方法，该方法将对其 `components` 中的每个元素调用 `draw` 方法，如清单 18-4 所示。

<Listing number="18-4" file-name="src/lib.rs" caption="`Screen` 上的 `run` 方法，它在每个组件上调用 `draw` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

这与定义一个使用泛型类型参数（generic type parameter）加上 trait 约束（trait bound）的结构体不同。泛型类型参数一次只能被一个具体类型替代，而 trait 对象则允许在运行时容纳多个具体类型。例如，我们可以使用泛型类型参数来定义 `Screen` 结构体，如清单 18-5 所示。

<Listing number="18-5" file-name="src/lib.rs" caption="一种使用泛型和 trait 约束的 `Screen` 结构体替代实现">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs}}
```

</Listing>

这个限制在大多数情况下是合适的，因为使用泛型的定义已经覆盖了我们所遇到的大多数情况。然而，对于清单 18-4 中的实现，`Screen` 实例可以容纳实现 `Draw` 的多种不同类型的 `Vec<T>`，而在清单 18-5 中，`Screen` 实例只能容纳一种具体类型的 `Vec<T>`。也就是说，清单 18-4 中的 `Screen`（没有泛型）适用于需要容纳不同类型的情况；而清单 18-5 中的 `Screen`（使用泛型）则适用于 `components` 集合都是同一类型的情况。

### 实现 Trait

现在我们添加了一些实现了 `Draw` trait 的类型。我们将提供 `Button` 类型。再次强调，实际上编写 GUI 库超出了本书的范围，所以 `draw` 方法体内不会有任何有用的实现。为了想象这个实现可能是什么样子，`Button` 结构体可能包含 `width`、`height` 和 `label` 字段，如清单 18-7 所示。

<Listing number="18-7" file-name="src/lib.rs" caption="一个实现了 `Draw` trait 的 `Button` 结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button` 上的 `width`、`height` 和 `label` 字段将与其他组件上的字段不同；例如，一个 `TextField` 类型可能具有相同的字段，外加一个 `placeholder` 字段。我们想要在屏幕上绘制的每种类型都会实现 `Draw` trait，但在 `draw` 方法中使用不同的代码来定义如何绘制该特定类型，就像这里的 `Button` 一样（如前所述，没有实际的 GUI 代码）。例如，`Button` 类型可能还有一个额外的 `impl` 块，其中包含与用户点击按钮时发生的事件相关的方法。这类方法不适用于 `TextField` 等类型。

如果有人使用我们的库决定实现一个具有 `width`、`height` 和 `options` 字段的 `SelectBox` 结构体，他们也将在 `SelectBox` 类型上实现 `Draw` trait，如清单 18-8 所示。

<Listing number="18-8" file-name="src/main.rs" caption="另一个使用 `gui` 并且在 `SelectBox` 结构体上实现 `Draw` trait 的 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

库的用户现在可以编写他们的 `main` 函数来创建一个 `Screen` 实例。对于 `Screen` 实例，他们可以通过将 `SelectBox` 和 `Button` 分别放入 `Box<T>` 中使其成为 trait 对象，从而将它们添加进去。然后他们可以调用 `Screen` 实例上的 `run` 方法，该方法将对每个组件调用 `draw` 方法。清单 18-9 展示了这个实现。

<Listing number="18-9" file-name="src/main.rs" caption="使用 trait 对象存储实现同一 trait 的不同类型的值">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

在我们编写这个库时，我们并不知道有人可能会添加 `SelectBox` 类型，但是我们的 `Screen` 实现仍然能够操作这个新类型并绘制它，因为 `SelectBox` 实现了 `Draw` trait，这意味着它实现了 `draw` 方法。

这种概念——只关心一个值响应哪些消息（messages），而不关心该值的具体类型——类似于动态类型语言中的*鸭子类型（duck typing）*：如果它走路像鸭子、叫起来像鸭子，那么它就是鸭子！在清单 18-5 中 `Screen` 上的 `run` 实现中，`run` 不需要知道每个组件的具体类型是什么。它不检查某个组件是 `Button` 的实例还是 `SelectBox` 的实例，它只是在组件上调用 `draw` 方法。通过将 `Box<dyn Draw>` 指定为 `components` 向量中值的类型，我们将 `Screen` 定义为需要那些我们可以在其上调用 `draw` 方法的值。

使用 trait 对象和 Rust 的类型系统来编写类似于使用鸭子类型的代码的好处在于，我们永远不必在运行时检查某个值是否实现了特定的方法，也不必担心如果某个值没有实现某个方法但我们调用了它会出现错误。如果这些值没有实现 trait 对象所需的 trait，Rust 将不会编译我们的代码。

例如，清单 18-10 展示了如果我们尝试使用 `String` 作为组件来创建 `Screen` 会发生什么。

<Listing number="18-10" file-name="src/main.rs" caption="尝试使用未实现 trait 对象所需 trait 的类型">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

我们会得到这个错误，因为 `String` 没有实现 `Draw` trait：

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

这个错误告诉我们，要么我们传递了不想要的东西给 `Screen`，所以应该传递一个不同的类型；要么我们应该在 `String` 上实现 `Draw`，这样 `Screen` 才能在其上调用 `draw`。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### 执行动态派发

回想一下我们在第 10 章中的[“使用泛型代码的性能”][performance-of-code-using-generics]<!-- ignore -->部分讨论的编译器对泛型执行的单态化（monomorphization）过程：编译器为我们在泛型类型参数位置上使用的每种具体类型生成非泛型的函数和方法实现。单态化产生的代码执行的是*静态派发（static dispatch）*，即编译器在编译时就知道你在调用哪个方法。与此相对的是*动态派发（dynamic dispatch）*，即编译器在编译时无法判断你在调用哪个方法。在动态派发的情况下，编译器生成的代码会在运行时知道该调用哪个方法。

当我们使用 trait 对象时，Rust 必须使用动态派发。编译器不知道使用 trait 对象的代码中可能会出现哪些类型，因此它不知道调用哪个类型上实现的哪个方法。相反，在运行时，Rust 使用 trait 对象内部的指针来知道该调用哪个方法。这种查找会产生静态派发所没有的运行时开销。动态派发还会阻止编译器选择内联（inline）方法的代码，从而进一步阻止某些优化，并且 Rust 有一些关于可以在何处以及不能使用动态派发的规则，称为*dyn 兼容性（dyn compatibility）*。这些规则超出了本次讨论的范围，但你可以在[参考文档][dyn-compatibility]<!-- ignore -->中阅读更多相关信息。然而，我们在清单 18-5 中编写的代码确实获得了额外的灵活性，并能够支持清单 18-9 中的功能，所以这是一个需要权衡的取舍。

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
