<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## 深入探究 Async 的 Trait

在本章中，我们以各种方式使用了 `Future`、`Stream` 和 `StreamExt` trait。不过，到目前为止，我们一直避免过于深入地探讨它们的工作原理或它们如何组合在一起，对于日常 Rust 工作来说，这在大多数情况下是没问题的。但有时，你会遇到需要更多了解这些 trait 细节的情况，以及 `Pin` 类型和 `Unpin` trait。在本节中，我们将深入探究到足以帮助应对这些场景的程度，但仍将*真正*深入的探讨留给其他文档。

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### `Future` Trait

让我们首先仔细看看 `Future` trait 是如何工作的。以下是 Rust 对其的定义：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

这个 trait 定义包含了一堆新类型以及一些我们之前未见过的语法，所以让我们逐个部分地分析这个定义。

首先，`Future` 的关联类型 `Output` 表示该 future 解析成的结果。这类似于 `Iterator` trait 的 `Item` 关联类型。其次，`Future` 有一个 `poll` 方法，该方法为 `self` 参数接受一个特殊的 `Pin` 引用，以及一个对 `Context` 类型的可变引用，并返回 `Poll<Self::Output>`。我们稍后会更多地讨论 `Pin` 和 `Context`。现在，让我们关注该方法返回的内容，即 `Poll` 类型：

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

这个 `Poll` 类型类似于 `Option`。它有一个带值的变体 `Ready(T)`，和一个不带值的变体 `Pending`。然而，`Poll` 的含义与 `Option` 相当不同！`Pending` 变体表示 future 仍有工作要做，因此调用者稍后需要再次检查。`Ready` 变体表示 `Future` 已完成其工作，`T` 值现在可用。

> 注意：很少需要直接调用 `poll`，但如果确实需要，请记住，对于大多数 future 来说，调用者在 future 返回 `Ready` 之后不应再次调用 `poll`。许多 future 在变为就绪后如果再次被轮询会 panic。可以安全地再次轮询的 future 会在其文档中明确说明。这与 `Iterator::next` 的行为类似。

当你看到使用 `await` 的代码时，Rust 在底层将其编译为调用 `poll` 的代码。如果你回顾示例 17-4，我们在其中等待单个 URL 解析后打印页面标题，Rust 将其编译为类似（虽然不完全相同）以下这样的代码：

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // 但这里应该放什么？
    }
}
```

当 future 仍然是 `Pending` 时我们应该做什么？我们需要某种方式一次又一次地重试，直到 future 最终就绪。换句话说，我们需要一个循环：

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

然而，如果 Rust 将其编译为完全那样的代码，那么每个 `await` 都会是阻塞的——与我们想要达到的目标恰恰相反！相反，Rust 确保该循环可以将控制权交给某个可以暂停此 future 工作、处理其他 future 然后稍后再次检查这个 future 的东西。正如我们所看到的，那个东西就是异步运行时，而这种调度和协调工作是它的主要职责之一。

在[“使用消息传递在两个任务之间发送数据”][message-passing]<!-- ignore -->部分中，我们描述了等待 `rx.recv`。`recv` 调用返回一个 future，而等待该 future 会对其进行轮询。我们注意到，运行时将暂停该 future，直到它就绪，此时要么是 `Some(message)` 要么是通道关闭时的 `None`。凭借我们对 `Future` trait 更深入的理解，特别是 `Future::poll`，我们可以看到其工作原理。当运行时返回 `Poll::Pending` 时，它知道 future 尚未就绪。相反，当 `poll` 返回 `Poll::Ready(Some(message))` 或 `Poll::Ready(None)` 时，运行时知道 future *已*就绪并推进它。

运行时如何做到这一点的确切细节超出了本书的范围，但关键是理解 future 的基本机制：运行时*轮询*它负责的每个 future，当 future 尚未就绪时将其重新置于休眠状态。

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### `Pin` 类型与 `Unpin` Trait

回到示例 17-13，我们使用了 `trpl::join!` 宏来等待三个 future。然而，常见的情况是拥有一个包含若干数量 future 的集合（如向量），而这些数量在运行时之前是未知的。让我们将示例 17-13 更改为示例 17-23 中的代码，将三个 future 放入一个向量中，并改用 `trpl::join_all` 函数，这段代码还无法编译。

<Listing number="17-23" caption="等待集合中的 future"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

我们将每个 future 放入 `Box` 中以将其变为*trait 对象（trait object）*，就像我们在第 12 章“从 `run` 返回错误”部分中所做的那样。（我们将在第 18 章中详细介绍 trait 对象。）使用 trait 对象让我们可以将这些类型产生的每个匿名 future 视为同一类型，因为它们都实现了 `Future` trait。

这可能令人惊讶。毕竟，没有一个 async 代码块返回任何值，所以每个都产生 `Future<Output = ()>`。然而请记住，`Future` 是一个 trait，编译器为每个 async 代码块创建一个唯一的枚举，即使它们具有相同的输出类型也是如此。就像你不能将两个不同的手写结构体放入 `Vec` 中一样，你也不能混合编译器生成的枚举。

然后我们将 future 的集合传递给 `trpl::join_all` 函数并等待结果。然而，这无法编译；以下是错误消息的相关部分。

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

此错误消息中的注释告诉我们，应该使用 `pin!` 宏来*固定（pin）*这些值，这意味着将它们放入 `Pin` 类型中，该类型保证值不会在内存中被移动。错误消息说需要固定（pinning）是因为 `dyn Future<Output = ()>` 需要实现 `Unpin` trait，而它目前并没有。

`trpl::join_all` 函数返回一个名为 `JoinAll` 的结构体。该结构体在类型 `F` 上是泛型的，该类型被约束为实现 `Future` trait。直接用 `await` 等待一个 future 会隐式地固定该 future。这就是为什么我们不需要在想要等待 future 的每个地方都使用 `pin!`。

然而，我们在这里并不是直接等待一个 future。相反，我们通过将 future 的集合传递给 `join_all` 函数构造了一个新的 future，即 `JoinAll`。`join_all` 的签名要求集合中各项的类型都实现 `Future` trait，而 `Box<T>` 实现 `Future` 的条件是它所包裹的 `T` 是一个实现了 `Unpin` trait 的 future。

这需要消化很多！为了真正理解它，让我们稍微深入探究 `Future` trait 实际上是如何工作的，特别是围绕固定（pinning）。再次查看 `Future` trait 的定义：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`cx` 参数及其 `Context` 类型是运行时如何知道何时检查任何给定 future 同时仍然保持惰性的关键。同样，其工作细节超出了本章的范围，你通常只需在编写自定义 `Future` 实现时才需要考虑这个。我们将转而关注 `self` 的类型，因为这是我们第一次看到 `self` 带有类型注解的方法。`self` 的类型注解与其他函数参数的类型注解类似，但有两个关键区别：

- 它告诉 Rust `self` 必须是什么类型才能调用该方法。
- 它不能是任意类型。它被限制为方法实现所在的类型、该类型的引用或智能指针，或者包裹对该类型的引用的 `Pin`。

我们将在[第 18 章][ch-18]<!-- ignore -->中看到更多关于此语法的内容。目前，知道如果我们想要轮询一个 future 以检查它是 `Pending` 还是 `Ready(Output)`，我们需要一个对该类型的 `Pin` 包裹的可变引用就足够了。

`Pin` 是一个用于类似指针的类型（如 `&`、`&mut`、`Box` 和 `Rc`）的包装器。（从技术上讲，`Pin` 适用于实现了 `Deref` 或 `DerefMut` trait 的类型，但这实际上等效于仅适用于引用和智能指针。）`Pin` 本身不是一个指针，也没有像 `Rc` 和 `Arc` 那样的引用计数行为；它纯粹是编译器可以用来对指针使用强制约束的工具。

回想一下 `await` 是基于对 `poll` 的调用来实现的，这开始解释我们之前看到的错误消息，但那条消息是关于 `Unpin` 而不是 `Pin` 的。那么 `Pin` 究竟如何与 `Unpin` 相关联，为什么 `Future` 需要 `self` 在 `Pin` 类型中才能调用 `poll`？

回想本章前面部分，一个 future 中的一系列等待点会被编译成一个状态机，编译器确保该状态机遵循 Rust 关于安全性的所有常规规则，包括借用和所有权。为了实现这一点，Rust 会查看从一个等待点到下一个等待点或到 async 代码块末尾之间需要哪些数据，然后在编译后的状态机中创建相应的变体。每个变体获得了对源代码该部分中将使用的数据所需的访问权，无论是通过获取该数据的所有权，还是通过获取其可变或不可变引用。

到目前为止都很好：如果我们在给定 async 代码块中的所有权或引用方面出了任何错误，借用检查器会告诉我们。当我们想要移动该 async 代码块对应的 future 时——例如将其移入一个 `Vec` 以供传递给 `join_all`——事情就变得更加棘手。

当一个 future 被移动时——无论是通过将其推入数据结构用作迭代器，还是通过从函数返回——这实际上意味着 Rust 没有足够的信息来保证借用检查器所依赖的安全性。这让我们回到了前面做出的关于编译后的 async 代码块生成匿名枚举以保存其状态的观察。现在假设我们有一个这样的 async 代码块：

```rust
async {
    let mut x = [];
    let y = &x;
    let z = &y;
    *z = 3_i32;
    x
}
```

为了管理状态，Rust 创建一个匿名枚举，其第一个变体具有 `x`、`y` 和 `z`，第二个变体具有 `y` 和 `z`，最后一个变体只有 `z`。这是一个简化版，但它举例说明了基本思想。

现在想象如果我们将该 async 代码块产生的 future 移动到不同的内存位置会发生什么。移动意味着实际位的复制，就像任何其他情况一样。因为编译器将引用替换为 offset，移动后该数据的位置发生了偏移，尽管该数据中的值被精确复制。结果是，该 future 内的所有自引用都指向内存中的旧位置，而不是新位置。它们现在是悬垂的（dangling）。为了防止这种情况，在 `poll` 调用中，`self` 必须是 `Pin<&mut Self>` 而不是普通的 `&mut self`。

这引出了一个问题：为什么这个提议中 `poll` 定义的第一个版本（完全正常的 `&mut self`）就足够呢？答案是：最初确实足够——直到 Rust 开发者试图真正*使用*它为止。现在类型系统允许你在调用 `poll` 之间移动 futures，但所有这些自引用都导致了问题。这就是为什么我们现在必须使用 `Pin` 和 `Unpin`。

现在考虑如果我们有一个像 `let cx = ...` 这样的引用，指向我们刚才看到的 future *内部*的 `x`，情况会怎样。当 future 被移动后，`cx` 现在将指向一个完全错误的位置。这是个坏消息。然而，`Pin` 让我们可以向编译器保证我们不打算将值移动到不同的内存位置。`Pin` 做了它名字所暗示的事：它将一个值固定（pin）在内存中的原位，使其无法移动。这正是用来确保在 async 代码块中创建的自引用保持有效所需的条件。

但是等等：编译器已经为我们*自动*创建了所有这些状态机枚举。它为什么不能为我们自动处理 `Pin` 呢？实际上，它确实可以……在有限的情况下。当使用 `await` 关键字时——编译器确实通过代码生成来处理 `Pin`。然而，大多数类型自动实现了 `Unpin`（我们马上会更详细地讨论这一点）。编译器不能自动在任何地方都为你使用 `Pin`，因为在某些情况下这样做不安全，因此编写代码的人需要选择加入。但是，有些类型除了固定之外永远不能安全使用，在这些情况下，作者必须确保固定发生。不过，要遇到这些极端情况，你通常必须深入底层工作。

大多数时候，你不需要担心 `Pin` 和 `Unpin` 的具体细节。然而，在数据库、Web 服务器和其他在运行时处理大量 future 的工具的库代码中，你通常会使用像 `Box::pin` 这样的组合，其中 `Box<T>` 使你能够将其放在堆上，而 `Pin` 固定其位置。然后，你可以使用 `Pin<Box<T>>` 类型来固定类型 `T` 的 future。

还有另一种使用 `Pin` 的方法，我们在前面的 `join_all` 错误消息中也看到过：`pin!` 宏。你可以调用 `pin!`，传入要固定的值，它会返回一个固定到栈上的值。你现在会得到一个 `Pin<&mut T>` 类型的值。`Pin<&mut T>` 类型之所以有意义，是因为你无法移动 `&mut`，但可以移动引用指向的 `T`。`Pin` 保证 `T` 本身的内存位置是固定的，因此你的代码是安全的。

回到未来：现在我们对 `Pin` 有了一定的了解，让我们更仔细地看看 `Unpin`。`Pin` 防止类型被移动。同时，`Unpin` 的作用正如其名称所暗示的那样：它表示一个类型*可以*安全移动，即使它已被固定。正如你可能期望的那样，`Unpin` 和 `Pin` 可以结合使用。

所以，你在本章示例的编译器错误消息中看到了 `Unpin`，因为它以一种间接的方式出现在背景中。请记住，`Future` 的错误消息（即 "the trait `Unpin` is not implemented for `dyn Future<Output = ()>`"）指的是我们最初尝试在 `Vec<Box<dyn Future<Output = ()>>>` 中收集由 async 代码块产生的 future。编译器在这里看到的需要 `Unpin` 的原因，是因为这些 future 的内部引用。编译器*正在保护我们免受这个问题的影响*。我们需要告诉编译器，我们不会在将它们放入 `Vec` 后移动它们，这样它就可以放心这些引用不会失效。我们通过固定（pinning）它们来做到这一点，这正是 `pin!` 宏所做的。当我们固定它们之后，我们得到的是固定类型 `Pin<Box<dyn Future<Output = ()>>>`。这给了我们一个指向实现了 `Future` 的类型 `dyn Future<Output = ()>` 的 `Pin<Box<T>>`。当这样的类型实现 `Unpin` 时，编译器知道可以移动值而不会有任何风险。当我们固定一个指针后，如果该指针指向的类型实现了 `!Unpin`（即不实现 `Unpin`），编译器知道如果移动它将会出错，因此*不允许*移动它。这正是我们想要的行为。

因此，`Unpin` 是一个标记 trait（marker trait），类似于我们在第 16 章中看到的 `Send` 和 `Sync` trait，因此本身没有任何功能。标记 trait 的存在仅仅是为了告诉编译器在特定上下文中使用实现给定 trait 的类型是安全的。`Unpin` 通知编译器，给定类型*不*需要维护关于所涉值是否可以安全移动的任何保证。

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

与 `Send` 和 `Sync` 一样，编译器会自动为所有它可以证明是安全的类型实现 `Unpin`。一种特殊情况，同样类似于 `Send` 和 `Sync`，是 `Unpin` 对某个类型*没有*被实现的情况。其表示法是 <code>impl !Unpin for <em>SomeType</em></code>，其中 <code><em>SomeType</em></code> 是一种*确实*需要维护这些保证以确保安全的类型的名称，此类类型在 `Pin` 中使用的指针指向它时尤甚。

换句话说，关于 `Pin` 和 `Unpin` 之间的关系，需要记住两件事。首先，`Unpin` 是"正常"情况，而 `!Unpin` 是特殊情况。其次，类型是否实现 `Unpin` 或 `!Unpin`，*仅*在你使用指向该类型的固定指针（如 <code>Pin<&mut <em>SomeType</em>></code>）时才重要。

为了具体说明，考虑一个 `String`：它有长度和组成它的 Unicode 字符。我们可以将 `String` 包裹在 `Pin` 中，如图 17-8 所示。然而，`String` 自动实现 `Unpin`，就像 Rust 中大多数其他类型一样。

<figure>

<img alt="左侧有一个标注为“Pin”的框，箭头从它指向右侧一个标注为“String”的框。“String”框包含数据 5usize（表示字符串长度）以及字母“h”、“e”、“l”、“l”、“o”（表示存储在该 String 实例中的字符串“hello”的字符）。一个虚线矩形包围着“String”框及其标签，但不包围“Pin”框。" src="img/trpl17-08.svg" class="center" />

<figcaption>图 17-8：固定一个 `String`；虚线表示 `String` 实现了 `Unpin` trait，因此未被固定</figcaption>

</figure>

因此，我们可以做一些在 `String` 实现 `!Unpin` 的情况下会非法的操作，例如在同一内存位置将一个字符串替换为另一个字符串，如图 17-9 所示。这不会违反 `Pin` 的约定，因为 `String` 没有内部引用使其移动不安全。这正是它实现 `Unpin` 而非 `!Unpin` 的原因。

<figure>

<img alt="来自前一个示例的相同“hello”字符串数据，现在标注为“s1”并灰显。前一个示例中的“Pin”框现在指向一个不同的 String 实例，标注为“s2”，该实例有效，长度为 7usize，包含字符串“goodbye”的字符。s2 被虚线矩形包围，因为它也实现了 Unpin trait。" src="img/trpl17-09.svg" class="center" />

<figcaption>图 17-9：在内存中将 `String` 替换为完全不同的 `String`</figcaption>

</figure>

现在我们已经掌握了足够的知识来理解示例 17-23 中那个 `join_all` 调用报告的错误。我们最初尝试将由 async 代码块产生的 future 移入一个 `Vec<Box<dyn Future<Output = ()>>>` 中，但正如我们看到的，这些 future 可能具有内部引用，因此它们不会自动实现 `Unpin`。一旦我们将它们固定，就可以将得到的 `Pin` 类型传递给 `Vec`，确信 future 中的底层数据*不会*被移动。示例 17-24 展示了如何通过定义三个 future 的位置调用 `pin!` 宏并调整 trait 对象类型来修复代码。

<Listing number="17-24" caption="固定 future 以便能够将它们移入向量中">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

此示例现在可以编译和运行了，我们可以在运行时向向量中添加或移除 future，并将它们全部连接起来。

`Pin` 和 `Unpin` 主要用于构建底层库，或者当你构建运行时本身时，而非用于日常 Rust 代码。不过，当你在错误消息中看到这些 trait 时，现在你将更好地了解如何修复你的代码！

> 注意：`Pin` 和 `Unpin` 的组合使得在 Rust 中安全地实现一整类复杂的类型成为可能，否则这些类型由于其自引用（self-referential）性质而难以实现。需要 `Pin` 的类型在当今的异步 Rust 中最常见，但偶尔你也会在其他上下文中看到它们。
>
> `Pin` 和 `Unpin` 如何工作的具体细节以及它们需要遵守的规则在 `std::pin` 的 API 文档中有广泛涵盖，因此如果你有兴趣了解更多，这是一个很好的起点。
>
> 如果你想更详细地了解事物在底层是如何工作的，请参阅 [_Asynchronous Programming in Rust_][async-book] 的[第 2 章][under-the-hood]<!-- ignore -->和[第 4 章][pinning]<!-- ignore -->。

### `Stream` Trait

现在你对 `Future`、`Pin` 和 `Unpin` trait 有了更深入的掌握，我们可以将注意力转向 `Stream` trait。正如你在本章前面所学到的，流类似于异步迭代器。然而，与 `Iterator` 和 `Future` 不同，截至本文撰写之时，`Stream` 在标准库中没有定义，但是*确实*有一个来自 `futures` crate 的非常通用的定义，在整个生态系统中广泛使用。

让我们在查看 `Stream` trait 如何将它们合并之前，先回顾一下 `Iterator` 和 `Future` trait 的定义。从 `Iterator` 中，我们得到了序列的概念：其 `next` 方法提供 `Option<Self::Item>`。从 `Future` 中，我们得到了随时间推移变成就绪的概念：其 `poll` 方法提供 `Poll<Self::Output>`。为了表示随时间推移变成就绪的项目序列，我们定义了一个将这两个特性结合在一起的 `Stream` trait：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` trait 定义了一个名为 `Item` 的关联类型，表示流产生的项目的类型。这类似于 `Iterator`，其中可能有零到多个项目，而不像 `Future`，其中总是有一个单一的 `Output`（即使它是单元类型 `()`）。

`Stream` 还定义了一个方法来获取这些项目。我们称其为 `poll_next`，以明确它以与 `Future::poll` 相同的方式进行轮询，并以与 `Iterator::next` 相同的方式产生项目序列。其返回类型将 `Poll` 与 `Option` 结合在一起。外层类型是 `Poll`，因为它必须像 future 一样检查就绪状态。内层类型是 `Option`，因为它需要像迭代器一样指示是否还有更多消息。

类似于此定义的版本很可能最终会成为 Rust 标准库的一部分。与此同时，它是大多数运行时工具包的一部分，因此你可以依赖它，我们接下来讨论的所有内容通常都适用！

然而，在我们在[“流：顺序处理的 Future”][streams]<!-- ignore --> 部分中看到的示例中，我们并没有使用 `poll_next` *或* `Stream`，而是使用了 `next` 和 `StreamExt`。当然，我们*可以*直接以 `poll_next` API 的方式来工作，手动编写我们自己的 `Stream` 状态机，就像我们*可以*直接通过 future 的 `poll` 方法来使用 future 一样。不过，使用 `await` 要好得多，而 `StreamExt` trait 提供了 `next` 方法，因此我们可以这样做：

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> 注意：我们在本章前面使用的实际定义看起来与此略有不同，因为它支持的 Rust 版本尚未支持在 trait 中使用异步函数。因此，它看起来像这样：
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> 那个 `Next` 类型是一个实现了 `Future` 的 `struct`，并允许我们使用 `Next<'_, Self>` 来命名对 `self` 引用的生命周期，以便 `await` 可以与这个方法一起使用。

`StreamExt` trait 也是所有可用的有趣方法的归属地。`StreamExt` 会为每个实现 `Stream` 的类型自动实现，但这些 trait 是分开定义的，以使社区能够在便利性 API 上进行迭代而不影响基础 trait。

在 `trpl` crate 使用的 `StreamExt` 版本中，该 trait 不仅定义了 `next` 方法，还提供了一个正确调用 `Stream::poll_next` 细节的 `next` 默认实现。这意味着即使你需要编写自己的流数据类型，你*只需*实现 `Stream`，然后任何使用你数据类型的人都可以自动使用 `StreamExt` 及其方法。

这就是我们要介绍的关于这些 trait 的底层细节的全部内容。总结一下，让我们考虑 future（包括 stream）、任务和线程是如何协同工作的！

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html
