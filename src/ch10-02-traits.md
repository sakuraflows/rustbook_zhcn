<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## 使用 Trait 定义共享行为（Shared Behavior）

*trait* 定义了一个特定类型所具有的、并可以与其它类型共享的功能。我们可以使用 trait 以抽象的方式定义共享行为。我们可以使用 *trait 约束（trait bounds）* 来指定泛型类型可以是任何具有特定行为的类型。

> 注意：Trait 类似于其他语言中通常称为 *接口（interfaces）* 的功能，尽管有一些差异。

### 定义 Trait

一个类型的行为包括我们可以对该类型调用的方法。如果我们能在所有这些类型上调用相同的方法，则不同类型共享相同的行为。Trait 定义是一种将方法签名组合在一起的方法，用于定义实现某个目的所需的一组行为。

例如，假设我们有多个结构体，它们持有各种类型和数量的文本：一个 `NewsArticle` 结构体，持有特定地点存档的新闻报道；一个 `SocialPost`，最多可以有 280 个字符，并带有元数据，指示它是新帖子、转发还是对另一帖子的回复。

我们想要创建一个名为 `aggregator` 的媒体聚合器库 crate，它可以显示可能存储在 `NewsArticle` 或 `SocialPost` 实例中的数据的摘要。为此，我们需要每个类型的摘要，我们将通过在实例上调用 `summarize` 方法来请求该摘要。示例 10-12 定义了一个公共的 `Summary` trait，它表达了这种行为。

<Listing number="10-12" file-name="src/lib.rs" caption="一个 `Summary` trait，由 `summarize` 方法提供的行为组成">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

在这里，我们使用 `trait` 关键字声明了一个 trait，然后是 trait 的名称，在本例中是 `Summary`。我们还声明了这个 trait 为 `pub`，以便依赖此 crate 的其他 crate 也可以使用这个 trait，我们将在几个示例中看到这一点。在花括号内，我们声明了方法签名，这些签名描述了实现此 trait 的类型的行为，在本例中是 `fn summarize(&self) -> String`。

在方法签名之后，我们不提供花括号内的实现，而是使用分号。实现此 trait 的每个类型必须为方法体提供自己的自定义行为。编译器将强制执行：任何具有 `Summary` trait 的类型都必须完全按照此签名定义 `summarize` 方法。

一个 trait 在其主体中可以拥有多个方法：方法签名每行列出一个，每行以分号结束。

### 在类型上实现 Trait

现在我们已经定义了 `Summary` trait 方法的期望签名，我们可以在媒体聚合器中的类型上实现它。示例 10-13 展示了在 `NewsArticle` 结构体上 `Summary` trait 的实现，它使用 headline、author 和 location 来创建 `summarize` 的返回值。对于 `SocialPost` 结构体，我们将 `summarize` 定义为用户名后接帖子的全文，假设帖子内容已经限制在 280 个字符以内。

<Listing number="10-13" file-name="src/lib.rs" caption="在 `NewsArticle` 和 `SocialPost` 类型上实现 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

在类型上实现 trait 类似于实现常规方法。区别在于，在 `impl` 之后，我们放入要实现的 trait 名称，然后使用 `for` 关键字，然后指定要为其实现 trait 的类型名称。在 `impl` 块内，我们放入 trait 定义已经定义的方法签名。不在每个签名后加分号，而是使用花括号并填充方法体，其中包含我们希望 trait 的方法为该特定类型具有的特定行为。

现在库已经在 `NewsArticle` 和 `SocialPost` 上实现了 `Summary` trait，crate 的用户可以像调用常规方法一样在 `NewsArticle` 和 `SocialPost` 实例上调用 trait 方法。唯一的区别是用户必须将 trait 与类型一起引入作用域。以下是一个二进制 crate 如何使用我们的 `aggregator` 库 crate 的示例：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

这段代码打印 `1 new post: horse_ebooks: of course, as you probably already know, people`。

依赖于 `aggregator` crate 的其他 crate 也可以将 `Summary` trait 引入作用域，以在其自己的类型上实现 `Summary`。需要注意的一个限制是，只有当 trait 或类型（或两者）对于我们的 crate 是本地（local）的时候，我们才能在类型上实现 trait。例如，我们可以在自定义类型 `SocialPost` 上实现标准库 trait（如 `Display`），作为 `aggregator` crate 功能的一部分，因为类型 `SocialPost` 对于我们的 `aggregator` crate 是本地的。我们也可以在 `aggregator` crate 中的 `Vec<T>` 上实现 `Summary`，因为 trait `Summary` 对于我们的 `aggregator` crate 是本地的。

但我们不能对外部类型实现外部 trait。例如，我们不能在 `aggregator` crate 中的 `Vec<T>` 上实现 `Display` trait，因为 `Display` 和 `Vec<T>` 都在标准库中定义，而不是 `aggregator` crate 本地的。此限制是属于称为 *连贯性（coherence）* 的属性，更具体地说是 *孤儿规则（orphan rule）*，之所以如此命名是因为父类型不存在。此规则确保其他人的代码不会破坏你的代码，反之亦然。如果没有这条规则，两个 crate 可以为同一类型实现相同的 trait，而 Rust 将不知道使用哪个实现。

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### 使用默认实现

有时，为 trait 中的部分或全部方法提供默认行为是很有用的，而不是要求在每个类型上实现所有方法。然后，当我们在特定类型上实现 trait 时，我们可以保留或覆盖每个方法的默认行为。

在示例 10-14 中，我们为 `Summary` trait 的 `summarize` 方法指定了默认字符串，而不是像在示例 10-12 中那样仅定义方法签名。

<Listing number="10-14" file-name="src/lib.rs" caption="定义一个具有 `summarize` 方法默认实现的 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

要使用默认实现来摘要 `NewsArticle` 实例，我们指定一个空的 `impl` 块，其中包含 `impl Summary for NewsArticle {}`。

尽管我们不再直接在 `NewsArticle` 上定义 `summarize` 方法，但我们提供了一个默认实现，并指定 `NewsArticle` 实现了 `Summary` trait。因此，我们仍然可以在 `NewsArticle` 实例上调用 `summarize` 方法，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

这段代码打印 `New article available! (Read more...)`。

创建默认实现不要求我们对示例 10-13 中 `SocialPost` 上的 `Summary` 实现进行任何更改。原因是覆盖默认实现的语法与实现没有默认实现的 trait 方法的语法相同。

默认实现可以调用同一 trait 中的其他方法，即使这些其他方法没有默认实现。这样，trait 可以提供大量有用的功能，并且只要求实现者指定其中的一小部分。例如，我们可以定义 `Summary` trait 有一个需要实现的 `summarize_author` 方法，然后定义一个 `summarize` 方法，该方法具有调用 `summarize_author` 方法的默认实现：

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

要使用此版本的 `Summary`，我们只需要在类型上实现 trait 时定义 `summarize_author`：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

在定义了 `summarize_author` 之后，我们可以对 `SocialPost` 结构体的实例调用 `summarize`，并且 `summarize` 的默认实现将调用我们提供的 `summarize_author` 的定义。因为我们实现了 `summarize_author`，`Summary` trait 给了我们 `summarize` 方法的行为，而不需要我们再写任何代码。这就是它的样子：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

这段代码打印 `1 new post: (Read more from @horse_ebooks...)`。

请注意，无法从同一方法的覆盖实现中调用默认实现。

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### 将 Trait 用作参数

现在你已经知道如何定义和实现 trait，我们可以探讨如何使用 trait 来定义接受许多不同类型参数的函数。我们将使用在示例 10-13 中 `NewsArticle` 和 `SocialPost` 类型上实现的 `Summary` trait，来定义一个 `notify` 函数，该函数调用其 `item` 参数上的 `summarize` 方法，该参数是某个实现了 `Summary` trait 的类型。为此，我们使用 `impl Trait` 语法，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

对于 `item` 参数，我们不指定具体类型，而是指定 `impl` 关键字和 trait 名称。此参数接受任何实现了指定 trait 的类型。在 `notify` 的函数体中，我们可以调用 `item` 上来自 `Summary` trait 的任何方法，例如 `summarize`。我们可以调用 `notify` 并传入任何 `NewsArticle` 或 `SocialPost` 实例。使用任何其他类型（例如 `String` 或 `i32`）调用该函数的代码将无法编译，因为这些类型没有实现 `Summary`。

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Trait 约束语法

`impl Trait` 语法适用于简单情况，但它实际上是较长形式的语法糖，称为 *trait 约束（trait bound）*；它看起来像这样：

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这种较长形式等同于上一节中的示例，但更冗长。我们将 trait 约束放在泛型类型参数的声明之后，在冒号和尖括号内。

`impl Trait` 语法在简单情况下方便且使代码更简洁，而更全面的 trait 约束语法可以在其他情况下表达更复杂的含义。例如，我们可以有两个实现 `Summary` 的参数。使用 `impl Trait` 语法这样做看起来像这样：

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

如果我们希望此函数允许 `item1` 和 `item2` 具有不同的类型（只要两种类型都实现 `Summary`），使用 `impl Trait` 是合适的。然而，如果我们希望强制两个参数具有相同的类型，我们必须使用 trait 约束，如下所示：

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

指定为 `item1` 和 `item2` 参数类型的泛型类型 `T` 约束了该函数，使得作为 `item1` 和 `item2` 参数传入的值必须是相同的具体类型。

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### 通过 `+` 语法指定多个 Trait 约束

我们还可以指定多个 trait 约束。假设我们希望 `notify` 在 `item` 上同时使用 display 格式化以及 `summarize`：我们在 `notify` 的定义中指定 `item` 必须同时实现 `Display` 和 `Summary`。我们可以使用 `+` 语法来实现：

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` 语法也适用于泛型类型上的 trait 约束：

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

通过指定这两个 trait 约束，`notify` 的函数体可以调用 `summarize` 并使用 `{}` 来格式化 `item`。

#### 通过 `where` 子句使 Trait 约束更清晰

使用太多的 trait 约束有其缺点。每个泛型都有自己的 trait 约束，因此具有多个泛型类型参数的函数可能会在函数名称和参数列表之间包含大量 trait 约束信息，使函数签名难以阅读。因此，Rust 提供了另一种在函数签名后的 `where` 子句中指定 trait 约束的语法。所以，不必写成这样：

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

我们可以使用 `where` 子句，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

这个函数的签名不那么杂乱：函数名称、参数列表和返回类型靠在一起，类似于没有大量 trait 约束的函数。

### 返回实现了 Trait 的类型

我们也可以在返回位置使用 `impl Trait` 语法，返回某种实现了 trait 的类型的值，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

通过使用 `impl Summary` 作为返回类型，我们指定 `returns_summarizable` 函数返回某种实现了 `Summary` trait 的类型，而不命名具体类型。在这种情况下，`returns_summarizable` 返回一个 `SocialPost`，但调用此函数的代码不需要知道这一点。

仅通过其实现的 trait 来指定返回类型的能力在闭包（closure）和迭代器（iterator）的上下文中特别有用，我们将在第 13 章中介绍。闭包和迭代器创建了只有编译器知道或非常长的类型。`impl Trait` 语法让你可以简洁地指定函数返回某种实现了 `Iterator` trait 的类型，而无需写出一个非常长的类型。

然而，只有当你返回单一类型时，才能使用 `impl Trait`。例如，返回一个 `NewsArticle` 或一个 `SocialPost` 并将返回类型指定为 `impl Summary` 的这段代码将无法工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

由于 `impl Trait` 语法在编译器中的实现方式有限制，不允许返回 `NewsArticle` 或 `SocialPost` 两者之一。我们将在第 18 章的[“使用 Trait 对象来抽象共享行为”][trait-objects]<!-- ignore -->部分介绍如何编写具有此行为的函数。

### 使用 Trait 约束有条件地实现方法

通过对使用泛型类型参数的 `impl` 块使用 trait 约束，我们可以有条件地为实现了指定 trait 的类型实现方法。例如，示例 10-15 中的 `Pair<T>` 类型始终实现了 `new` 函数以返回 `Pair<T>` 的新实例（回想一下第 5 章的[“方法语法”][methods]<!-- ignore -->部分，`Self` 是 `impl` 块类型的别名，在这种情况下是 `Pair<T>`）。但在下一个 `impl` 块中，`Pair<T>` 仅在其内部类型 `T` 实现了 `PartialOrd` trait（允许比较）*和* `Display` trait（允许打印）时，才实现 `cmp_display` 方法。

<Listing number="10-15" file-name="src/lib.rs" caption="根据 trait 约束在泛型类型上有条件地实现方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

我们也可以有条件地为实现了另一个 trait 的任何类型实现一个 trait。对满足 trait 约束的任何类型的 trait 实现被称为*覆盖实现（blanket implementations）*，并在 Rust 标准库中广泛使用。例如，标准库对任何实现了 `Display` trait 的类型实现 `ToString` trait。标准库中的 `impl` 块看起来类似于以下代码：

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

因为标准库有这个覆盖实现，我们可以在任何实现了 `Display` trait 的类型上调用由 `ToString` trait 定义的 `to_string` 方法。例如，我们可以将整数转换为其对应的 `String` 值，因为整数实现了 `Display`：

```rust
let s = 3.to_string();
```

覆盖实现出现在 trait 文档中的"Implementors"部分。

Trait 和 trait 约束使我们能够编写使用泛型类型参数来减少重复的代码，同时向编译器指定我们希望泛型类型具有特定的行为。然后编译器可以使用 trait 约束信息来检查与我们的代码一起使用的所有具体类型是否提供了正确的行为。在动态类型语言中，如果我们对未定义该方法的类型调用了该方法，我们会在运行时得到一个错误。但 Rust 将这些错误移到编译时，这样我们在代码甚至能够运行之前就被迫修复问题。此外，我们不必编写在运行时检查行为的代码，因为我们已经在编译时检查过了。这样做提高了性能，而不必放弃泛型的灵活性。

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
