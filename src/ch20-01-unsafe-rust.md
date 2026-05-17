## Unsafe Rust

到目前为止，我们讨论的所有代码在编译时都强制实施了 Rust 的内存安全保证。然而，Rust 内部隐藏着第二种不强制实施这些内存安全保证的语言：它被称为 *unsafe Rust（不安全的 Rust）*，其工作方式与常规 Rust 一样，但给了我们额外的超能力。

Unsafe Rust 之所以存在，是因为静态分析本质上是保守的。当编译器试图确定代码是否遵守保证时，它宁可拒绝一些有效程序，也比接受一些无效程序要好。尽管代码*可能*是没问题的，但如果 Rust 编译器没有足够的信息来确信，它会拒绝这段代码。在这些情况下，你可以使用 unsafe 代码告诉编译器："相信我，我知道我在做什么。" 然而，请注意，你使用 unsafe Rust 需要自担风险：如果你错误地使用了 unsafe 代码，可能会因为内存不安全导致问题，例如空指针解引用。

Rust 有 unsafe 另一面的另一个原因是，底层的计算机硬件本质上是不安全的。如果 Rust 不让你执行不安全的操作，你就无法完成某些任务。Rust 需要允许你进行底层系统编程，例如直接与操作系统交互，甚至编写你自己的操作系统。与底层系统编程打交道是这门语言的目标之一。让我们探讨一下我们可以用 unsafe Rust 做什么以及如何做。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="unsafe-superpowers"></a>

### 执行不安全的超能力

要切换到 unsafe Rust，请使用 `unsafe` 关键字，然后开始一个包含不安全代码的新块。在 unsafe Rust 中，你可以执行在安全 Rust 中无法执行的五种操作，我们称之为*不安全超能力（unsafe superpowers）*。这些超能力包括：

1. 解引用裸指针（Dereference a raw pointer）
2. 调用不安全的函数或方法（Call an unsafe function or method）
3. 访问或修改可变静态变量（Access or modify a mutable static variable）
4. 实现不安全 trait（Implement an unsafe trait）
5. 访问 `union` 的字段（Access fields of `union`s）

理解 `unsafe` 不会关闭借用检查器或禁用 Rust 的任何其他安全检查是很重要的：如果你在不安全代码中使用了引用，它仍然会被检查。`unsafe` 关键字只让你能够访问这五个特性，编译器不会对这些特性进行内存安全检查。你仍然会在 unsafe 块内获得一定程度的安全性。

此外，`unsafe` 并不意味着块内的代码一定是危险的，或者它肯定会有内存安全问题：其意图是，作为程序员，你将确保 `unsafe` 块内的代码以有效的方式访问内存。

人都会犯错，错误总会发生，但通过要求这五种不安全操作位于用 `unsafe` 标注的块内，你将知道任何与内存安全相关的错误一定在 `unsafe` 块内。保持 `unsafe` 块小巧；当你以后调查内存错误时，你会感谢自己这样做的。

为了尽可能隔离不安全代码，最好将此类代码封装在一个安全抽象（safe abstraction）中并提供安全的 API，我们将在本章后面讨论不安全的函数和方法时介绍这一点。标准库的某些部分就是作为经过审计的不安全代码之上的安全抽象来实现的。将不安全代码包装在安全抽象中可以防止 `unsafe` 的使用泄露到你或你的用户可能想要使用用 `unsafe` 代码实现的功能的所有地方，因为使用安全抽象是安全的。

让我们逐一看看这五种不安全超能力。我们还将介绍一些为不安全代码提供安全接口的抽象。

### 解引用裸指针

在第 4 章的["悬垂引用"][dangling-references]<!-- ignore -->部分，我们提到编译器确保引用始终有效。Unsafe Rust 有两种新的类型，称为*裸指针（raw pointers）*，与引用类似。与引用一样，裸指针可以是不可变的或可变的，分别写为 `*const T` 和 `*mut T`。星号不是解引用运算符；它是类型名称的一部分。在裸指针的上下文中，*不可变（immutable）*意味着指针在解引用后不能直接被赋值。

与引用和智能指针不同，裸指针：

- 允许忽略借用规则，可以同时拥有不可变和可变的指针，或指向同一位置的多个可变指针
- 不保证指向有效的内存
- 允许为 null
- 不实现任何自动清理

通过选择退出 Rust 强制执行这些保证，你可以放弃有保证的安全性，以换取更高的性能或与其他语言或硬件交互的能力（Rust 的保证在那里不适用）。

清单 20-1 展示了如何创建不可变和可变的裸指针。

<Listing number="20-1" caption="使用原始借用运算符创建裸指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

请注意，这段代码中没有包含 `unsafe` 关键字。我们可以在安全代码中创建裸指针；只是不能在 unsafe 块之外解引用裸指针，你稍后会看到。

我们通过使用原始借用运算符创建裸指针：`&raw const num` 创建一个 `*const i32` 不可变裸指针，而 `&raw mut num` 创建一个 `*mut i32` 可变裸指针。因为我们是直接从局部变量创建的，我们知道这些特定的裸指针是有效的，但我们不能对任意裸指针做出这样的假设。

为了演示这一点，接下来我们将创建一个对其有效性不那么确定的裸指针，使用关键字 `as` 来转换一个值，而不是使用原始借用运算符。清单 20-2 展示了如何创建一个指向内存中任意地址的裸指针。尝试使用任意内存是未定义行为：该地址可能有数据也可能没有，编译器可能优化代码以致没有内存访问，或者程序可能因段错误而终止。通常，没有充分的理由编写这样的代码，尤其是在你可以使用原始借用运算符的情况下，但这是可能的。

<Listing number="20-2" caption="创建指向任意内存地址的裸指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

回想一下，我们可以在安全代码中创建裸指针，但不能解引用裸指针并读取所指向的数据。在清单 20-3 中，我们在需要 `unsafe` 块的裸指针上使用了解引用运算符 `*`。

<Listing number="20-3" caption="在 `unsafe` 块内解引用裸指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

创建指针本身没有害处；只有当我们试图访问它所指向的值时，才可能最终处理无效值。

还要注意，在清单 20-1 和 20-3 中，我们创建了指向同一内存位置（存储 `num` 的地方）的 `*const i32` 和 `*mut i32` 裸指针。如果我们尝试改为创建对 `num` 的不可变和可变引用，代码将无法编译，因为 Rust 的所有权规则不允许同时存在可变引用和任何不可变引用。使用裸指针，我们可以创建指向同一位置的可变指针和不可变指针，并通过可变指针更改数据，这可能导致数据竞争（data race）。请小心！

尽管有所有这些危险，你为什么还要使用裸指针呢？一个主要用例是与 C 代码交互时，你将在下一节中看到。另一个用例是构建借用检查器无法理解的安全抽象。我们将介绍不安全函数，然后看一个使用不安全代码的安全抽象示例。

### 调用不安全函数或方法

你可以在 unsafe 块中执行的第二种操作是调用不安全的函数。不安全的函数和方法看起来与常规函数和方法完全一样，但在定义的其余部分之前多了一个 `unsafe`。此上下文中的 `unsafe` 关键字表示该函数有一些我们在调用该函数时需要保证满足的要求，因为 Rust 无法保证我们已经满足了这些要求。通过在 `unsafe` 块中调用不安全函数，我们是在说我们已经阅读了该函数的文档，并且我们承担了履行该函数契约的责任。

这是一个名为 `dangerous` 的不安全函数，其函数体内不执行任何操作：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

我们必须在一个单独的 `unsafe` 块中调用 `dangerous` 函数。如果我们尝试在没有 `unsafe` 块的情况下调用 `dangerous`，将会收到一个错误：

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

使用 `unsafe` 块，我们向 Rust 断言我们已经阅读了该函数的文档，我们理解如何正确使用它，并且我们已经验证了我们正在履行该函数的契约。

要在 `unsafe` 函数的函数体中执行不安全操作，你仍然需要使用 `unsafe` 块，就像在常规函数中一样，如果忘记使用，编译器会警告你。这有助于我们保持 `unsafe` 块尽可能小，因为不安全操作可能不需要跨越整个函数体。

#### 在不安全代码之上创建安全抽象

仅仅因为一个函数包含不安全代码，并不意味着我们需要将整个函数标记为 unsafe。事实上，将不安全代码包装在安全函数中是一种常见的抽象。例如，让我们研究一下来自标准库的 `split_at_mut` 函数，它需要一些不安全代码。我们将探讨如何实现它。这个安全方法定义在可变切片上：它接受一个切片，并通过在给定索引处分割切片将其一分为二。清单 20-4 展示了如何使用 `split_at_mut`。

<Listing number="20-4" caption="使用安全的 `split_at_mut` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

我们不能仅使用安全 Rust 来实现这个函数。一次尝试可能看起来像清单 20-5，它无法编译。为简单起见，我们将 `split_at_mut` 实现为一个函数而不是方法，并且仅针对 `i32` 值的切片，而不是针对泛型类型 `T`。

<Listing number="20-5" caption="仅使用安全 Rust 尝试实现 `split_at_mut`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

这个函数首先获取切片的长度。然后，它通过检查索引是否小于等于长度来断言作为参数给定的索引在切片内。断言意味着，如果我们传递一个大于长度的索引来分割切片，函数会在尝试使用该索引之前 panic。

然后，我们返回一个包含两个可变切片的元组：一个从原切片开头到 `mid` 索引，另一个从 `mid` 到切片末尾。

当我们尝试编译清单 20-5 中的代码时，会得到一个错误：

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust 的借用检查器无法理解我们在借用切片的不同部分；它只知道我们在两次借用同一个切片。借用切片的不同部分从根本上说是没问题的，因为这两个切片不重叠，但 Rust 不够聪明，无法知道这一点。当我们知道代码没问题，但 Rust 不知道时，就该使用不安全代码了。

清单 20-6 展示了如何使用 `unsafe` 块、裸指针和一些对不安全函数的调用来使 `split_at_mut` 的实现工作。

<Listing number="20-6" caption="在 `split_at_mut` 函数的实现中使用不安全代码">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

回顾第 4 章中的["切片类型"][the-slice-type]<!-- ignore -->部分，切片是一个指向某些数据的指针以及切片的长度。我们使用 `len` 方法获取切片的长度，使用 `as_mut_ptr` 方法访问切片的裸指针。在这种情况下，因为我们有一个指向 `i32` 值的可变切片，`as_mut_ptr` 返回一个类型为 `*mut i32` 的裸指针，我们将其存储在变量 `ptr` 中。

我们保留了 `mid` 索引在切片内的断言。然后，我们进入不安全代码：`slice::from_raw_parts_mut` 函数接受一个裸指针和一个长度，并创建一个切片。我们使用这个函数创建一个从 `ptr` 开始、长度为 `mid` 个元素的切片。然后，我们在 `ptr` 上调用 `add` 方法，以 `mid` 为参数，获得一个从 `mid` 开始的裸指针，并使用该指针和 `mid` 之后的剩余元素数量作为长度来创建另一个切片。

函数 `slice::from_raw_parts_mut` 是不安全的，因为它接受一个裸指针，并且必须相信这个指针是有效的。裸指针上的 `add` 方法也是不安全的，因为它必须相信偏移量位置也是一个有效的指针。因此，我们必须在调用 `slice::from_raw_parts_mut` 和 `add` 的周围放一个 `unsafe` 块，才能调用它们。通过查看代码并添加 `mid` 必须小于等于 `len` 的断言，我们可以判断 `unsafe` 块中使用的所有裸指针都是指向切片内数据的有效指针。这是对 `unsafe` 的可接受且适当的使用。

请注意，我们不需要将最终的 `split_at_mut` 函数标记为 `unsafe`，我们可以从安全 Rust 中调用这个函数。我们为不安全代码创建了一个安全抽象，通过以安全的方式使用 `unsafe` 代码的函数实现，因为它只从该函数有权访问的数据中创建有效的指针。

相比之下，清单 20-7 中对 `slice::from_raw_parts_mut` 的使用在切片被使用时很可能会导致崩溃。这段代码接受一个任意的内存位置，并创建一个 10,000 个元素的切片。

<Listing number="20-7" caption="从任意内存位置创建切片">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

我们并不拥有这个任意位置的内存，并且无法保证这段代码创建的切片包含有效的 `i32` 值。尝试将 `values` 当作有效切片来使用会导致未定义行为。

#### 使用 `extern` 函数调用外部代码

有时你的 Rust 代码可能需要与用其他语言编写的代码交互。为此，Rust 有关键字 `extern`，它有助于创建和使用*外部函数接口（Foreign Function Interface，FFI）*，这是一种编程语言定义函数并使另一种（外部的）编程语言能够调用这些函数的方式。

清单 20-8 演示了如何设置与 C 标准库中的 `abs` 函数的集成。在 `extern` 块中声明的函数通常从 Rust 代码调用是不安全的，因此 `extern` 块也必须标记为 `unsafe`。原因是其他语言不强制执行 Rust 的规则和保证，而 Rust 无法检查它们，因此确保安全性的责任落在了程序员身上。

<Listing number="20-8" file-name="src/main.rs" caption="声明并调用在另一种语言中定义的 `extern` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

在 `unsafe extern "C"` 块内，我们列出了想从另一种语言调用的外部函数的名称和签名。`"C"` 部分定义了外部函数使用的*应用程序二进制接口（application binary interface，ABI）*：ABI 定义了如何在汇编层面调用该函数。`"C"` ABI 是最常见的，遵循 C 编程语言的 ABI。关于 Rust 支持的所有 ABI 的信息，请参阅[Rust 参考文档][ABI]。

在 `unsafe extern` 块中声明的每个项都隐式地是不安全的。然而，某些 FFI 函数*是*可以安全调用的。例如，C 标准库中的 `abs` 函数没有任何内存安全方面的考虑，并且我们知道可以用任何 `i32` 来调用它。在这种情况下，我们可以使用 `safe` 关键字来说明这个特定函数即使在 `unsafe extern` 块中也是可以安全调用的。一旦我们做出这个更改，调用它就不再需要 `unsafe` 块，如清单 20-9 所示。

<Listing number="20-9" file-name="src/main.rs" caption="在 `unsafe extern` 块中明确将一个函数标记为 `safe` 并安全调用它">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

将一个函数标记为 `safe` 并不会从本质上使其变得安全！相反，它就像是你在向 Rust 承诺它是安全的。你仍然有责任确保这个承诺得到履行！

#### 从其他语言调用 Rust 函数

我们也可以使用 `extern` 来创建允许其他语言调用 Rust 函数的接口。我们不必创建整个 `extern` 块，而是在相关函数的 `fn` 关键字之前添加 `extern` 关键字并指定要使用的 ABI。我们还需要添加 `#[unsafe(no_mangle)]` 标注来告诉 Rust 编译器不要混淆（mangle）这个函数的名称。*混淆（Mangling）*是指编译器将我们给定的函数名称更改为另一个名称，该名称包含供编译过程的其他部分使用的更多信息，但可读性较差。每种编程语言编译器混淆名称的方式略有不同，因此要使 Rust 函数能被其他语言通过名称调用，我们必须禁用 Rust 编译器的名称混淆。这是不安全的，因为如果没有内置的混淆机制，库之间可能会存在名称冲突，因此我们有责任确保我们选择的名称在没有混淆的情况下安全导出。

在以下示例中，我们将 `call_from_c` 函数编译为共享库并从 C 链接后，使其可以从 C 代码访问：

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

这种 `extern` 的使用仅在属性中需要 `unsafe`，而不在 `extern` 块上。

### 访问或修改可变静态变量

在本书中，我们还没有讨论过全局变量，Rust 确实支持全局变量，但它们可能给 Rust 的所有权规则带来问题。如果两个线程正在访问同一个可变全局变量，可能导致数据竞争。

在 Rust 中，全局变量被称为*静态（static）*变量。清单 20-10 展示了以字符串切片为值的静态变量的声明和使用示例。

<Listing number="20-10" file-name="src/main.rs" caption="定义和使用不可变静态变量">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

静态变量类似于常量（constants），我们在第 3 章的["声明常量"][constants]<!-- ignore -->部分讨论过。按照惯例，静态变量的名称使用 `SCREAMING_SNAKE_CASE`。静态变量只能存储具有 `'static` 生命周期的引用，这意味着 Rust 编译器可以推断出生命周期，我们不需要显式标注。访问不可变静态变量是安全的。

常量和不可变静态变量之间的一个细微区别是，静态变量中的值在内存中具有固定的地址。使用该值将始终访问相同的数据。另一方面，常量在每次使用时可以复制它们的数据。另一个区别是，静态变量可以是可变的。访问和修改可变静态变量是*不安全的*。清单 20-11 展示了如何声明、访问和修改一个名为 `COUNTER` 的可变静态变量。

<Listing number="20-11" file-name="src/main.rs" caption="读取或写入可变静态变量是不安全的。">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

与常规变量一样，我们使用 `mut` 关键字指定可变性。任何读取或写入 `COUNTER` 的代码都必须在 `unsafe` 块内。清单 20-11 中的代码可以编译并按照我们预期打印 `COUNTER: 3`，因为它是单线程的。让多个线程访问 `COUNTER` 很可能导致数据竞争，因此这是未定义行为。因此，我们需要将整个函数标记为 `unsafe` 并记录安全限制，以便任何调用该函数的人都知道他们可以安全地做什么和不能做什么。

每当我们编写不安全函数时，习惯上要编写一个以 `SAFETY` 开头的注释，解释调用者需要做什么来安全地调用该函数。同样，每当我们执行不安全操作时，习惯上要编写一个以 `SAFETY` 开头的注释，解释安全规则是如何得到维护的。

此外，编译器默认会通过编译器 lint（compiler lint）拒绝任何尝试创建对可变静态变量的引用的操作。你必须通过添加 `#[allow(static_mut_refs)]` 标注来显式退出该 lint 的保护，或者通过使用原始借用运算符之一创建的裸指针来访问可变静态变量。这包括引用被不可见地创建的情况，例如在本代码清单中的 `println!` 中使用它时。要求通过裸指针创建对静态可变变量的引用，有助于使使用它们的安全要求更加明显。

对于全局可访问的可变数据，很难确保没有数据竞争，这就是为什么 Rust 认为可变静态变量是不安全的。在可能的情况下，最好使用我们在第 16 章讨论的并发技术和线程安全智能指针，以便编译器检查来自不同线程的数据访问是否安全进行。

### 实现不安全 Trait

我们可以使用 `unsafe` 来实现一个不安全的 trait。当某个 trait 的至少一个方法具有编译器无法验证的某种不变性（invariant）时，该 trait 就是不安全的。我们通过在 `trait` 之前添加 `unsafe` 关键字来声明 trait 是 `unsafe` 的，并将 trait 的实现也标记为 `unsafe`，如清单 20-12 所示。

<Listing number="20-12" caption="定义和实现不安全 trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

通过使用 `unsafe impl`，我们承诺我们将会维护编译器无法验证的那些不变性。

例如，回想一下我们在第 16 章中的["使用 `Send` 和 `Sync` 实现可扩展的并发"][send-and-sync]<!-- ignore -->部分讨论的 `Send` 和 `Sync` 标记 trait：如果我们的类型完全由其他实现 `Send` 和 `Sync` 的类型组成，编译器会自动实现这些 trait。如果我们实现了一个包含未实现 `Send` 或 `Sync` 的类型（例如裸指针）的类型，并且我们希望将该类型标记为 `Send` 或 `Sync`，我们必须使用 `unsafe`。Rust 无法验证我们的类型是否维护了它可以安全地在线程间发送或从多个线程访问的保证；因此，我们需要手动进行这些检查，并用 `unsafe` 来表明这一点。

### 访问 Union 的字段

仅在使用 `unsafe` 时才能执行的最后一个操作是访问 union（联合体）的字段。*union* 类似于 `struct`，但在特定实例中一次只使用一个声明的字段。Union 主要用于与 C 代码中的 union 交互。访问 union 字段是不安全的，因为 Rust 无法保证当前存储在 union 实例中的数据的类型。你可以在 [Rust 参考文档][unions]中了解更多关于 union 的信息。

### 使用 Miri 检查不安全代码

在编写不安全代码时，你可能想要检查你所写的内容是否确实安全且正确。最好的方法之一是使用 Miri，这是一个用于检测未定义行为的官方 Rust 工具。借用检查器是一种在编译时工作的*静态（static）*工具，而 Miri 是一种在运行时工作的*动态（dynamic）*工具。它通过运行你的程序或其测试套件来检查你的代码，并在你违反它理解的关于 Rust 应该如何工作的规则时进行检测。

使用 Miri 需要 Rust 的 nightly 构建版本（我们在[附录 G：Rust 的开发和"Nightly Rust"][nightly]<!-- ignore -->中会详细讨论）。你可以通过输入 `rustup +nightly component add miri` 来同时安装 nightly 版本的 Rust 和 Miri 工具。这不会更改你的项目使用的 Rust 版本；它只是将工具添加到你的系统中，以便你想用时可以使用它。你可以通过输入 `cargo +nightly miri run` 或 `cargo +nightly miri test` 在项目上运行 Miri。

为了举例说明这有多有用，考虑一下我们在清单 20-7 上运行 Miri 时会发生什么。

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri 正确地警告我们正在将一个整数转换为指针，这可能是个问题，但 Miri 无法确定是否存在问题，因为它不知道指针的来源。然后，Miri 返回一个错误，因为清单 20-7 存在未定义行为——我们有一个悬垂指针。多亏了 Miri，我们现在知道存在未定义行为的风险，并且我们可以思考如何使代码安全。在某些情况下，Miri 甚至可以就如何修复错误提出建议。

Miri 并不能捕捉到你在编写不安全代码时可能犯的所有错误。Miri 是一个动态分析工具，因此它只能捕捉到实际运行的代码的问题。这意味着你需要将其与良好的测试技术结合使用，以增强你对所编写不安全代码的信心。Miri 也没有涵盖你的代码可能不健全的每一种方式。

换句话说：如果 Miri *确实*发现了问题，你就知道存在 bug，但仅仅因为 Miri *没有*发现 bug 并不意味着就没有问题。不过，它可以捕捉到很多问题。尝试在本章的其他不安全代码示例上运行它，看看它会说什么！

你可以在[其 GitHub 仓库][miri]中了解更多关于 Miri 的信息。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="when-to-use-unsafe-code"></a>

### 正确使用不安全代码

使用 `unsafe` 来使用刚才讨论的五种超能力之一并没有错，甚至也不会被反对，但要正确编写 `unsafe` 代码更加棘手，因为编译器无法帮助维护内存安全。当你有理由使用 `unsafe` 代码时，你可以这样做，并且拥有显式的 `unsafe` 标注使得在出现问题时分清问题根源更加容易。每当你编写不安全代码时，你可以使用 Miri 来帮助你更确信所编写的代码遵守了 Rust 的规则。

要更深入地探索如何有效地使用 unsafe Rust，请阅读 Rust 官方的 `unsafe` 指南：[The Rustonomicon][nomicon]。

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
