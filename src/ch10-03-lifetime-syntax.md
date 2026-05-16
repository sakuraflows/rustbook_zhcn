## 使用生命周期（Lifetime）验证引用

生命周期是另一种我们已经在使用中的泛型。不同于确保类型具有我们想要的行为，生命周期确保引用在我们需要它们的时候一直有效。

我们在第 4 章的[“引用与借用”][references-and-borrowing]<!-- ignore -->部分没有讨论的一个细节是，Rust 中的每个引用都有一个生命周期（lifetime），也就是该引用有效的范围。大多数情况下，生命周期是隐式且可推断的，就像大多数情况下类型可以被推断一样。我们只在可能存在多种类型时才需要标注类型。类似地，当引用的生命周期可能以几种不同方式相互关联时，我们必须标注生命周期。Rust 要求我们使用泛型生命周期参数来标注这些关系，以确保运行时使用的实际引用绝对是有效的。

标注生命周期甚至不是大多数其他编程语言都有的概念，所以这可能会让人感到陌生。尽管我们不会在本章中完整地介绍生命周期，但我们将讨论你可能遇到的生命周期语法的常见方式，以便你能熟悉这个概念。

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### 悬垂引用（Dangling References）

生命周期的主要目的是防止悬垂引用（dangling references），如果允许它们存在，程序会引用非其意图引用的数据。考虑示例 10-16 中的程序，它有一个外部作用域和一个内部作用域。

<Listing number="10-16" caption="尝试使用值已经离开作用域的引用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> 注意：示例 10-16、10-17 和 10-23 中的示例声明了变量但未赋予初始值，因此变量名存在于外部作用域。乍一看，这似乎与 Rust 没有空值（null）相冲突。然而，如果我们尝试在给变量赋值之前使用它，我们会得到一个编译时错误，这表明 Rust 确实不允许空值。

外部作用域声明了一个名为 `r` 的变量，没有初始值，内部作用域声明了一个名为 `x` 的变量，初始值为 `5`。在内部作用域中，我们尝试将 `r` 的值设置为对 `x` 的引用。然后，内部作用域结束，我们尝试打印 `r` 中的值。这段代码无法编译，因为 `r` 所引用的值在我们尝试使用它之前已经离开了作用域。以下是错误消息：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

错误消息说变量 `x` "does not live long enough"。原因是当内部作用域在第 7 行结束时，`x` 将超出作用域。但 `r` 在外部作用域中仍然有效；因为它的作用域更大，我们说它"活得更长"。如果 Rust 允许这段代码工作，`r` 将引用当 `x` 超出作用域时已释放的内存，而我们尝试对 `r` 做的任何事情都不会正确工作。那么，Rust 如何确定这段代码是无效的呢？它使用了一个借用检查器。

### 借用检查器（Borrow Checker）

Rust 编译器有一个*借用检查器（borrow checker）*，它比较作用域以确定所有借用是否有效。示例 10-17 显示了与示例 10-16 相同的代码，但带有注释显示变量的生命周期。

<Listing number="10-17" caption="`r` 和 `x` 的生命周期注释，分别命名为 `'a` 和 `'b`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

在这里，我们用 `'a` 注释了 `r` 的生命周期，用 `'b` 注释了 `x` 的生命周期。如你所见，内部的 `'b` 块比外部的 `'a` 生命周期块小得多。在编译时，Rust 比较两个生命周期的大小，并看到 `r` 的生命周期是 `'a`，但它引用的内存的生命周期是 `'b`。程序被拒绝，因为 `'b` 比 `'a` 短：引用的主题（subject）没有引用本身活得长。

示例 10-18 修复了代码，使其没有悬垂引用，并且可以编译而没有任何错误。

<Listing number="10-18" caption="一个有效的引用，因为数据的生命周期比引用的生命周期长">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

在这里，`x` 的生命周期是 `'b`，在这种情况下它比 `'a` 大。这意味着 `r` 可以引用 `x`，因为 Rust 知道只要 `x` 有效，`r` 中的引用就始终有效。

现在你已经知道引用的生命周期在哪里以及 Rust 如何分析生命周期以确保引用始终有效，让我们探讨函数参数和返回值中的泛型生命周期。

### 函数中的泛型生命周期

我们将编写一个返回两个字符串切片中较长者的函数。这个函数将接受两个字符串切片并返回一个字符串切片。在我们实现了 `longest` 函数之后，示例 10-19 中的代码应该打印 `The longest string is abcd`。

<Listing number="10-19" file-name="src/main.rs" caption="调用 `longest` 函数以查找两个字符串切片中较长者的 `main` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

注意，我们希望函数接受字符串切片，即引用，而不是字符串，因为我们不希望 `longest` 函数获取其参数的所有权。请参阅第 4 章的[“字符串切片作为参数”][string-slices-as-parameters]<!-- ignore -->部分，了解更多关于为什么我们在示例 10-19 中使用的参数是我们想要的参数。

如果我们尝试按照示例 10-20 所示实现 `longest` 函数，它将无法编译。

<Listing number="10-20" file-name="src/main.rs" caption="一个返回两个字符串切片中较长者的 `longest` 函数实现，但尚不能编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

相反，我们得到以下关于生命周期的错误：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

帮助文本显示返回类型需要一个泛型生命周期参数，因为 Rust 无法判断返回的引用是指向 `x` 还是 `y`。实际上，我们也不知道，因为这个函数体中的 `if` 块返回对 `x` 的引用，而 `else` 块返回对 `y` 的引用！

当我们定义这个函数时，我们不知道将传入此函数的具体值，因此我们不知道 `if` 情况还是 `else` 情况会执行。我们也不知道传入的引用的具体生命周期，因此我们无法像在示例 10-17 和 10-18 中那样查看作用域来确定我们返回的引用是否始终有效。借用检查器也无法确定这一点，因为它不知道 `x` 和 `y` 的生命周期如何与返回值的生命周期相关联。要修复这个错误，我们将添加泛型生命周期参数，用于定义引用之间的关系，以便借用检查器可以执行其分析。

### 生命周期注解语法

生命周期注解不会改变任何引用的存活时间。相反，它们描述多个引用的生命周期之间的相互关系，而不影响生命周期。就像函数在签名指定泛型类型参数时可以接受任何类型一样，函数可以通过指定泛型生命周期参数来接受具有任何生命周期的引用。

生命周期注解的语法有点不寻常：生命周期参数的名称必须以撇号（`'`）开头，通常全是小写且非常短，就像泛型类型一样。大多数人使用名称 `'a` 作为第一个生命周期注解。我们将生命周期参数注解放在引用的 `&` 之后，使用空格将注解与引用的类型分隔开。

以下是一些示例——没有生命周期参数的 `i32` 引用、具有名为 `'a` 的生命周期参数的 `i32` 引用，以及同样具有生命周期 `'a` 的可变 `i32` 引用：

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

一个生命周期注解本身没有多大意义，因为注解的目的是告诉 Rust 多个引用的泛型生命周期参数如何相互关联。让我们看看在 `longest` 函数的上下文中，生命周期注解如何相互关联。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### 在函数签名中

要在函数签名中使用生命周期注解，我们需要在函数名称和参数列表之间的尖括号内声明泛型生命周期参数，就像我们对泛型类型参数所做的那样。

我们希望签名表达以下约束：只要两个参数都有效，返回的引用就有效。这是参数生命周期和返回值生命周期之间的关系。我们将生命周期命名为 `'a`，然后将其添加到每个引用中，如示例 10-21 所示。

<Listing number="10-21" file-name="src/main.rs" caption="`longest` 函数定义，指定签名中的所有引用必须具有相同的生命周期 `'a`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

当我们将此代码与示例 10-19 中的 `main` 函数一起使用时，它应该能够编译并产生我们想要的结果。

函数签名现在告诉 Rust，对于某个生命周期 `'a`，该函数接受两个参数，这两个参数都是字符串切片，其存活时间至少与生命周期 `'a` 一样长。函数签名还告诉 Rust，从函数返回的字符串切片将至少存活与生命周期 `'a` 一样长。实际上，这意味着 `longest` 函数返回的引用的生命周期等于函数参数引用的值的生命周期中较小的那个。这些关系正是我们希望 Rust 在分析此代码时使用的。

记住，当我们在函数签名中指定生命周期参数时，我们并没有改变传入或返回的任何值的生命周期。相反，我们指定借用检查器应拒绝任何不遵守这些约束的值。注意，`longest` 函数不需要确切知道 `x` 和 `y` 将存活多久，只需要知道某个可以替代 `'a` 的作用域将满足此签名。

在函数中注解生命周期时，注解放在函数签名中，而不是函数体中。生命周期注解成为函数契约的一部分，就像签名中的类型一样。函数签名包含生命周期契约意味着 Rust 编译器所做的分析可以更简单。如果函数的注解方式或调用方式有问题，编译器错误可以更精确地指向我们代码和约束的部分。相反，如果 Rust 编译器对我们意图的生命周期关系做了更多推断，编译器可能只能在远离问题根源的多个步骤之后指向我们代码的使用处。

当我们向 `longest` 传递具体引用时，替代 `'a` 的具体生命周期是 `x` 的作用域与 `y` 的作用域重叠的部分。换句话说，泛型生命周期 `'a` 将获得等于 `x` 和 `y` 中较小的那个生命周期的具体生命周期。因为我们用相同的生命周期参数 `'a` 注解了返回的引用，所以返回的引用也将对 `x` 和 `y` 中较小的生命周期长度有效。

让我们通过传入具有不同具体生命周期的引用来看看生命周期注解如何限制 `longest` 函数。示例 10-22 是一个直接了当的例子。

<Listing number="10-22" file-name="src/main.rs" caption="使用具有不同具体生命周期的 `String` 值引用调用 `longest` 函数">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

在这个例子中，`string1` 在外部作用域结束之前有效，`string2` 在内部作用域结束之前有效，而 `result` 引用的对象在内部作用域结束之前有效。运行这段代码，你会看到借用检查器批准了它；它将编译并打印 `The longest string is long string is long`。

接下来，让我们尝试一个例子，展示 `result` 中引用的生命周期必须是两个参数中较小的那个生命周期。我们将 `result` 变量的声明移到内部作用域之外，但将 `result` 变量的赋值留在 `string2` 的内部作用域内。然后，我们将使用 `result` 的 `println!` 移到内部作用域之外，在内部作用域结束之后。示例 10-23 中的代码将无法编译。

<Listing number="10-23" file-name="src/main.rs" caption="在 `string2` 离开作用域后尝试使用 `result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

当我们尝试编译这段代码时，会得到这个错误：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

错误显示，要使 `result` 对 `println!` 语句有效，`string2` 需要直到外部作用域结束都有效。Rust 知道这一点，因为我们使用相同的生命周期参数 `'a` 标注了函数参数和返回值的生命周期。

作为人类，我们可以查看这段代码，看到 `string1` 比 `string2` 长，因此 `result` 将包含对 `string1` 的引用。因为 `string1` 尚未离开作用域，对 `string1` 的引用对于 `println!` 语句仍然有效。然而，编译器无法在这种情况下看到引用是有效的。我们已经告诉 Rust，`longest` 函数返回的引用的生命周期与传入的引用中较小的那个生命周期相同。因此，借用检查器禁止示例 10-23 中的代码，认为它可能具有无效的引用。

尝试设计更多实验，改变传入 `longest` 函数的引用的值和生命周期以及返回引用的使用方式。在编译之前，对你的实验是否能够通过借用检查器做出假设；然后，检查你是否正确！

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### 关系

你需要指定生命周期参数的方式取决于你的函数正在做什么。例如，如果我们将 `longest` 函数的实现改为始终返回第一个参数而不是最长的字符串切片，我们就不需要在 `y` 参数上指定生命周期。以下代码将编译：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

我们为参数 `x` 和返回类型指定了生命周期参数 `'a`，但没有为参数 `y` 指定，因为 `y` 的生命周期与 `x` 的生命周期或返回值没有任何关系。

从函数返回引用时，返回类型的生命周期参数需要与其中一个参数的生命周期参数匹配。如果返回的引用*不*指向其中一个参数，它必须指向在此函数内部创建的值。然而，这将是一个悬垂引用，因为该值将在函数结束时离开作用域。考虑这个无法编译的 `longest` 函数实现尝试：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

在这里，即使我们已经为返回类型指定了生命周期参数 `'a`，这个实现也无法编译，因为返回值的生命周期与参数的生命周期完全无关。这是我们得到的错误消息：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

问题在于 `result` 在 `longest` 函数结束时离开作用域并被清理掉。我们还试图从函数返回对 `result` 的引用。我们无法指定会改变悬垂引用的生命周期参数，Rust 也不允许我们创建悬垂引用。在这种情况下，最好的修复方法是返回拥有所有权的数据类型而不是引用，这样调用函数就负责清理该值。

最终，生命周期语法是关于连接函数各种参数和返回值的生命周期的。一旦它们被连接起来，Rust 就有足够的信息来允许内存安全的操作，并禁止可能导致悬垂指针或以其他方式违反内存安全的操作。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### 在结构体定义中

到目前为止，我们定义的结构体都持有拥有所有权的类型。我们可以定义持有引用的结构体，但在此情况下，我们需要在结构体定义的每个引用上添加生命周期注解。示例 10-24 有一个名为 `ImportantExcerpt` 的结构体，它持有一个字符串切片。

<Listing number="10-24" file-name="src/main.rs" caption="一个持有引用的结构体，需要生命周期注解">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

这个结构体有单个字段 `part`，它持有一个字符串切片，即一个引用。与泛型数据类型一样，我们在结构体名称后面的尖括号内声明泛型生命周期参数的名称，这样我们就可以在结构体定义的主体中使用该生命周期参数。此注解意味着 `ImportantExcerpt` 的实例不能比它在 `part` 字段中持有的引用存活得更久。

这里的 `main` 函数创建了一个 `ImportantExcerpt` 结构体的实例，该实例持有对变量 `novel` 拥有的 `String` 的第一个句子的引用。`novel` 中的数据在 `ImportantExcerpt` 实例创建之前就已存在。此外，`novel` 在 `ImportantExcerpt` 离开作用域之后才离开作用域，因此 `ImportantExcerpt` 实例中的引用是有效的。

### 生命周期省略（Lifetime Elision）

你已经了解到每个引用都有一个生命周期，并且你需要为使用引用的函数或结构体指定生命周期参数。然而，我们在示例 4-9 中有一个函数（在示例 10-25 中再次展示），它在没有生命周期注解的情况下编译通过了。

<Listing number="10-25" file-name="src/lib.rs" caption="我们在示例 4-9 中定义的函数，它在没有生命周期注解的情况下编译通过，尽管参数和返回类型都是引用">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

这个函数在没有生命周期注解的情况下编译通过的原因是有历史原因的：在 Rust 的早期版本（pre-1.0）中，这段代码无法编译，因为每个引用都需要显式的生命周期。那时，函数签名将写成这样：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

在编写了大量 Rust 代码之后，Rust 团队发现 Rust 程序员在特定情况下反复输入相同的生命周期注解。这些情况是可预测的，并遵循一些确定性的模式。开发者将这些模式编程到编译器的代码中，以便借用检查器在这些情况下可以推断出生命周期，而不需要显式注解。

这段 Rust 历史是相关的，因为将来可能会出现更多确定性的模式并添加到编译器中。在未来，可能需要的生命周期注解会更少。

编程到 Rust 的引用分析中的模式被称为*生命周期省略规则（lifetime elision rules）*。这些不是程序员要遵循的规则；它们是编译器会考虑的一组特定情况，如果你的代码符合这些情况，你就不需要显式地编写生命周期。

省略规则并不提供完整的推断。如果应用这些规则后，引用的生命周期仍然存在歧义，编译器不会猜测剩余引用的生命周期应该是什么。编译器不会猜测，而是会给你一个错误，你可以通过添加生命周期注解来解决。

函数或方法参数上的生命周期被称为*输入生命周期（input lifetimes）*，而返回值上的生命周期被称为*输出生命周期（output lifetimes）*。

编译器使用三条规则来确定引用在没有显式注解时的生命周期。第一条规则适用于输入生命周期，第二和第三条规则适用于输出生命周期。如果编译器执行完三条规则后，仍然有它无法确定生命周期的引用，编译器将报错停止。这些规则适用于 `fn` 定义以及 `impl` 块。

第一条规则是编译器为每个引用参数分配一个生命周期参数。换句话说，有一个参数的函数得到一个生命周期参数：`fn foo<'a>(x: &'a i32)`；有两个参数的函数得到两个独立的生命周期参数：`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`；以此类推。

第二条规则是，如果只有一个输入生命周期参数，该生命周期被分配给所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。

第三条规则是，如果有多个输入生命周期参数，但其中一个是 `&self` 或 `&mut self`（因为这是一个方法），则 `self` 的生命周期被分配给所有输出生命周期参数。第三条规则使得方法更易于读写，因为需要的符号更少。

让我们假装我们是编译器。我们将应用这些规则来确定示例 10-25 中 `first_word` 函数签名中引用的生命周期。签名开始时没有任何与引用关联的生命周期：

```rust,ignore
fn first_word(s: &str) -> &str {
```

然后，编译器应用第一条规则，该规则指定每个参数获得自己的生命周期。我们照常称之为 `'a`，所以现在的签名是这样的：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

第二条规则适用，因为只有一个输入生命周期。第二条规则指定一个输入参数的生命周期被分配给输出生命周期，所以现在的签名是这样的：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

现在这个函数签名中的所有引用都有了生命周期，编译器可以继续其分析，而无需程序员在此函数签名中标注生命周期。

让我们再看一个例子，这次是我们在示例 10-20 中开始使用的没有生命周期参数的 `longest` 函数：

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

让我们应用第一条规则：每个参数获得自己的生命周期。这次我们有两个参数而不是一个，所以我们有两个生命周期：

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

你可以看到第二条规则不适用，因为有不止一个输入生命周期。第三条规则也不适用，因为 `longest` 是一个函数而不是方法，所以没有一个参数是 `self`。在应用了所有三条规则之后，我们仍然没有弄清楚返回类型的生命周期是什么。这就是为什么我们在尝试编译示例 10-20 中的代码时得到一个错误：编译器按照生命周期省略规则进行了处理，但仍然无法确定签名中所有引用的生命周期。

因为第三条规则实际上只适用于方法签名，所以我们接下来将看看方法中的生命周期上下文，以了解为什么第三条规则意味着我们不必经常在方法签名中标注生命周期。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### 在方法定义中

当我们在具有生命周期的结构体上实现方法时，我们使用与泛型类型参数相同的语法，如示例 10-11 所示。我们在哪里声明和使用生命周期参数取决于它们是与结构体字段相关，还是与方法参数和返回值相关。

结构体字段的生命周期名称始终需要在 `impl` 关键字之后声明，然后在结构体名称之后使用，因为这些生命周期是结构体类型的一部分。

在 `impl` 块内部的方法签名中，引用可能被绑定到结构体字段中的引用的生命周期，或者它们可能是独立的。此外，生命周期省略规则通常使得在方法签名中不需要生命周期注解。让我们看一些使用我们在示例 10-24 中定义的 `ImportantExcerpt` 结构体的例子。

首先，我们将使用一个名为 `level` 的方法，其唯一参数是对 `self` 的引用，其返回值是 `i32`，不是对任何东西的引用：

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

`impl` 之后的生命周期参数声明及其在类型名称后的使用是必需的，但由于第一条省略规则，我们不需要标注对 `self` 的引用的生命周期。

这是一个第三条生命周期省略规则适用的例子：

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

有两个输入生命周期，因此 Rust 应用第一条生命周期省略规则，并为 `&self` 和 `announcement` 分别赋予自己的生命周期。然后，因为其中一个参数是 `&self`，返回类型获得 `&self` 的生命周期，现在所有生命周期都已确定。

### 静态生命周期（Static Lifetime）

我们需要讨论的一个特殊生命周期是 `'static`，它表示受影响的引用*可以*在整个程序期间存活。所有字符串字面量（string literals）都具有 `'static` 生命周期，我们可以如下标注：

```rust
let s: &'static str = "I have a static lifetime.";
```

该字符串的文本直接存储在程序的二进制文件中，始终可用。因此，所有字符串字面量的生命周期都是 `'static`。

你可能会在错误消息中看到建议使用 `'static` 生命周期。但在将 `'static` 指定为引用的生命周期之前，请考虑该引用是否真的在整个程序的生命周期内都存在，以及你是否希望如此。大多数情况下，建议使用 `'static` 生命周期的错误消息是由尝试创建悬垂引用或可用生命周期不匹配导致的。在这种情况下，解决方案是修复这些问题，而不是指定 `'static` 生命周期。

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## 泛型类型参数、Trait 约束与生命周期

让我们简要地看一下在一个函数中指定泛型类型参数、trait 约束和生命周期的语法！

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

这是示例 10-21 中的 `longest` 函数，它返回两个字符串切片中较长的那个。但现在它有一个额外的参数 `ann`，类型为泛型 `T`，可以由任何实现了 `Display` trait 的类型填充（由 `where` 子句指定）。这个额外的参数将使用 `{}` 打印，这就是为什么 `Display` trait 约束是必需的。因为生命周期是一种泛型，所以生命周期参数 `'a` 和泛型类型参数 `T` 的声明放在函数名称后面尖括号内的同一个列表中。

## 总结

我们在本章中涵盖了很多内容！现在你已经了解了泛型类型参数、trait 和 trait 约束以及泛型生命周期参数，你已经准备好编写没有重复且在许多不同情况下工作的代码了。泛型类型参数让你可以将代码应用于不同的类型。Trait 和 trait 约束确保即使类型是泛型的，它们也具有代码所需的行为。你学习了如何使用生命周期注解来确保灵活的代码不会出现悬垂引用。而所有这些分析都在编译时进行，不会影响运行时性能！

信不信由你，关于我们在本章中讨论的主题，还有很多需要学习：第 18 章讨论了 trait 对象，这是使用 trait 的另一种方式。还有一些涉及生命周期注解的更复杂场景，你只在非常高级的场景中才需要；对于这些，你应该阅读 [Rust 参考手册][reference]。但接下来，你将学习如何在 Rust 中编写测试，以便确保你的代码按预期工作。

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/trait-bounds.html
