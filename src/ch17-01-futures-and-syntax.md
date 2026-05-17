## Future 与 Async 语法

Rust 中异步编程的关键元素是 *future* 以及 Rust 的 `async` 和 `await` 关键字。

一个 *future* 是一个现在可能尚未就绪，但在将来某个时刻会变为就绪的值。（同样的概念出现在许多语言中，有时以其他名称出现，如 *task* 或 *promise*。）Rust 提供了 `Future` trait 作为构建块，以便不同的异步操作可以用不同的数据结构实现，但共享一个公共接口。在 Rust 中，future 是实现 `Future` trait 的类型。每个 future 都持有自己的关于已取得进展以及"就绪"意味着什么的信息。

你可以将 `async` 关键字应用于代码块和函数，以指定它们可以被中断和恢复。在一个 async 代码块或 async 函数内部，你可以使用 `await` 关键字来*等待（await）一个 future*（即，等待它变为就绪）。你在 async 代码块或函数中等待 future 的任何位置，都是该代码块或函数可能暂停和恢复的潜在位置。检查一个 future 以查看其值是否可用的过程称为*轮询（polling）*。

其他一些语言（如 C# 和 JavaScript）也使用 `async` 和 `await` 关键字进行异步编程。如果你熟悉这些语言，你可能会注意到 Rust 在处理这些语法方面有一些显著的不同。这是有充分理由的，我们稍后就会看到！

在编写异步 Rust 代码时，我们大多数时候都使用 `async` 和 `await` 关键字。Rust 将它们编译为使用 `Future` trait 的等效代码，就像它将 `for` 循环编译为使用 `Iterator` trait 的等效代码一样。不过，由于 Rust 提供了 `Future` trait，你也可以在需要时为自定义数据类型实现它。我们在本章中会看到的许多函数，返回的类型都带有它们自己对 `Future` 的实现。我们将在本章末尾回到该 trait 的定义，深入探讨其工作原理，但这些细节已经足够我们继续前进了。

这一切可能感觉有点抽象，所以让我们来编写第一个异步程序：一个小型网页抓取器。我们将从命令行传入两个 URL，并发地获取它们，并返回最先完成那个的结果。这个例子会有不少新语法，但不用担心——我们会一路解释你所需了解的一切。

## 我们的第一个异步程序

为了让本章聚焦于学习 async 而非在生态系统的各个部分之间周旋，我们创建了 `trpl` crate（`trpl` 是"The Rust Programming Language"的缩写）。它重新导出了你需要的所有类型、trait 和函数，主要来自 [`futures`][futures-crate]<!-- ignore --> 和 [`tokio`][tokio]<!-- ignore --> crate。`futures` crate 是 Rust 异步代码实验的官方家园，实际上 `Future` trait 最初就是在此处设计的。Tokio 是当今 Rust 中使用最广泛的异步运行时，尤其适用于 Web 应用。还有其他优秀的运行时存在，它们可能更适合你的目的。我们在 `trpl` 底层使用了 `tokio` crate，因为它经过了充分测试且广泛使用。

在某些情况下，`trpl` 还会重命名或包装原始 API，以让你专注于与本章相关的细节。如果你想了解该 crate 的实际工作，我们鼓励你查看[其源代码][crate-source]<!-- ignore -->。你将能够看到它调用了哪个 crate 以及重新导出了什么。

创建一个小型命令行工具，读取两个 URL，并发获取两者，并返回最先完成那个的名称，向我们展示了很多关键部分。在之前的章节中，我们采用自底向上的方式，先讲授细节再将其组合成一个综合示例，但在这里，我们将以相反的方式进行。我们将先编写一个函数，然后逐步处理过程中遇到的编译器错误，直到一切就绪。我们从示例 17-1 中显示的函数开始。

<Listing number="17-1" file-name="src/main.rs" caption="定义一个 async 函数，从一个 URL 获取页面标题">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

在示例 17-1 中，我们定义了一个名为 `page_title` 的函数，并将其标记为 `async`。然后我们使用 `trpl::get` 来获取传入的任何 URL，并使用 `await` 关键字来等待（await）响应。然后我们调用 `text` 获取响应的文本内容，并再次使用 `await` 关键字来等待它。这两步都是异步的。对于 `get`，我们需要等待服务器发送其响应的第一部分，其中包括标头（headers）、连接信息等，并且在完整响应通过线路传输时，主体数据的一部分可能已经到达。即使整个响应已经到达，之后的 `text` 也需要等待整个响应主体作为一个 `String` 返回。我们必须显式等待这两个 future，因为 Rust 中的 future 是*惰性（lazy）*的：在你等待它们之前，它们不会做任何事情。（实际上，如果你使用一个 future 而不等待它，Rust 会发出一个编译器警告。）

当我们等待完对 `text` 的调用后，我们有了一个 `String`。我们用 `Html::parse` 将其包装为一个 `Html` 类型。我们没有定义原始字符串作为 `Html` 的解析，而是使用 `trpl` crate 将 `scraper` crate 的 `Html` 类型重新导出为 `trpl::Html`。然后我们使用 `select_first` 方法查找第一个匹配指定 CSS 选择器的元素。我们传入字符串 `"title"`，并得到一个 `Option`，包含一个表示匹配到元素的项（如果有匹配的话）。然后我们调用 `inner_html` 方法获取该元素的内容，它是一个 `String`。最终，我们得到一个 `Option<String>`。

请注意，Rust 的 `await` 关键字位于你要等待的表达式*之后*，而不是之前。也就是说，它是一个*后置（postfix）*关键字。这可能与你在其他语言中使用 `async` 的习惯不同，但在 Rust 中，这使得方法链的编写更加顺畅。因此，我们可以将 `page_title` 的函数体修改为将 `trpl::get` 和 `text` 函数调用串联在一起，并在它们之间使用 `await`，如示例 17-2 所示。

<Listing number="17-2" file-name="src/main.rs" caption="使用 `await` 关键字进行链式调用">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

至此，我们已成功编写了第一个异步函数！在 `main` 中添加调用它的代码之前，让我们再多谈一些关于我们所写内容及其含义。

当 Rust 看到一个标记有 `async` 关键字的*代码块*时，它将其编译为一个实现了 `Future` trait 的唯一的匿名数据类型。当 Rust 看到一个标记有 `async` 的*函数*时，它将编译为一个非异步函数，其函数体是一个 async 代码块。异步函数的返回类型是编译器为该 async 代码块创建的匿名数据类型的类型。

因此，编写 `async fn` 等同于编写一个返回该返回类型的 *future* 的函数。对编译器来说，像示例 17-1 中的 `async fn page_title` 这样的函数定义大致等同于如下定义的非异步函数：

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

让我们逐一分析转换后版本的各个部分：

- 它使用了我们在第 10 章"[将 Trait 作为参数][impl-trait]<!-- ignore -->"一节中讨论过的 `impl Trait` 语法。
- 返回值实现了 `Future` trait，其关联类型为 `Output`。请注意，`Output` 类型是 `Option<String>`，这与 `async fn` 版本中 `page_title` 的原始返回类型相同。
- 原始函数体中调用的所有代码都被包裹在一个 `async move` 代码块中。请记住，代码块也是表达式。整个代码块就是从函数返回的表达式。
- 这个 async 代码块产生一个类型为 `Option<String>` 的值，如上所述。该值与返回类型中的 `Output` 类型匹配。这与你之前见过的其他代码块一样。
- 新的函数体是一个 `async move` 代码块，这是因为它使用了 `url` 参数。（我们将在本章后面更多地讨论 `async` 与 `async move` 的区别。）

现在我们可以从 `main` 中调用 `page_title` 了。

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### 使用运行时执行异步函数

首先，我们将获取单个页面的标题，如示例 17-3 所示。不幸的是，这段代码目前还无法编译。

<Listing number="17-3" file-name="src/main.rs" caption="从 `main` 中使用用户提供的参数调用 `page_title` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

我们遵循与第 12 章中[接受命令行参数][cli-args]<!-- ignore -->小节相同的模式来获取命令行参数。然后我们将 URL 参数传递给 `page_title`，并等待结果。因为 future 产生的值是一个 `Option<String>`，所以我们使用 `match` 表达式来打印不同的消息，以反映页面是否包含 `<title>`。

我们只能在使用 `await` 关键字的 async 函数或代码块中使用它，而 Rust 不允许我们将特殊的 `main` 函数标记为 `async`。

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` 不能标记为 `async` 的原因是，异步代码需要一个*运行时（runtime）*：一个管理所有异步代码执行细节的 Rust crate。程序的 `main` 函数可以*初始化*一个运行时，但它本身*不是*一个运行时。（稍后我们将更详细地介绍这是为什么。）每个执行异步代码的 Rust 程序至少有一个位置设置了执行 future 的运行时。

大多数支持 async 的语言都捆绑了一个运行时，但 Rust 没有。相反，有许多不同的异步运行时可用，每个都在其目标用例适用的权衡之间做出不同的选择。例如，一个具有多个 CPU 核心和大量 RAM 的高吞吐量 Web 服务器，与一个单核、少量 RAM 且没有堆分配能力的微控制器，有非常不同的需求。提供这些运行时的 crate 也常常提供常用功能（如文件或网络 I/O）的异步版本。

在这里以及本章的其余部分，我们将使用 `trpl` crate 中的 `block_on` 函数，它接受一个 future 作为参数，并阻塞当前线程直到该 future 运行完毕。在幕后，调用 `block_on` 会使用 `tokio` crate 设置一个运行时，用于运行传入的 future（`trpl` crate 的 `block_on` 行为类似于其他运行时 crate 的 `block_on` 函数）。一旦 future 完成，`block_on` 返回该 future 产生的任何值。

我们可以将 `page_title` 返回的 future 直接传递给 `block_on`，一旦完成，我们可以像在示例 17-3 中尝试做的那样，对结果 `Option<String>` 进行 `match`。然而，对于本章的大多数示例（以及现实世界中的大多数异步代码），我们要做的不仅仅是调用一个异步函数，所以我们将传入一个 `async` 代码块，并显式地等待 `page_title` 调用的结果，如示例 17-4 所示。

<Listing number="17-4" caption="使用 `trpl::block_on` 等待一个 async 代码块" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

当我们运行这段代码时，得到了最初预期的行为：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

呼——我们终于有了一些可工作的异步代码！但在添加让两个站点相互竞速的代码之前，让我们先简要地将注意力转回到 future 的工作原理上。

每个*等待点（await point）*——即代码使用 `await` 关键字的每个位置——都代表控制权被交还给运行时的位置。为了实现这一点，Rust 需要跟踪 async 代码块中涉及的状态，以便运行时可以启动其他工作，然后在准备好再次尝试推进第一个工作时回来。这是一个不可见的状态机，就好像你编写了一个类似如下的枚举来保存每个等待点的当前状态：

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

然而，手动编写在每个状态之间转换的代码将是繁琐且容易出错的，尤其是当你以后需要向代码中添加更多功能和更多状态时。幸运的是，Rust 编译器会自动创建和管理异步代码的状态机数据结构。通常的借用和所有权规则仍然适用于数据结构，值得高兴的是，编译器也会处理这些检查，并提供有用的错误信息。我们将在本章后面处理其中一些。

最终，必须由某个东西来执行这个状态机，而那个东西就是运行时。（这就是为什么你在查阅运行时相关资料时可能会遇到*执行器（executor）*这个词：执行器是运行时负责执行异步代码的那部分。）

现在你可以看到为什么编译器在示例 17-3 中阻止我们将 `main` 本身设为异步函数了。如果 `main` 是一个异步函数，那么就需要其他东西来管理 `main` 返回的任何 future 的状态机，但 `main` 是程序的起点！相反，我们在 `main` 中调用了 `trpl::block_on` 函数，以设置一个运行时，并运行 `async` 代码块返回的 future，直到它完成。

> 注意：一些运行时提供了宏，因此你*可以*编写一个异步 `main` 函数。这些宏将 `async fn main() { ... }` 重写为一个普通的 `fn main`，其所做的与我们手动在示例 17-4 中所做的相同：调用一个函数来运行 future 直到完成，就像 `trpl::block_on` 所做的那样。

现在，让我们把这些部分组合起来，看看如何编写并发代码。

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### 并发地让两个 URL 相互竞速

在示例 17-5 中，我们使用从命令行传入的两个不同 URL 调用 `page_title`，并通过选择先完成哪个 future 让它们竞速。

<Listing number="17-5" caption="为两个 URL 调用 `page_title` 以查看哪个先返回" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

我们首先为用户提供的每个 URL 调用 `page_title`。我们将生成的 future 保存为 `title_fut_1` 和 `title_fut_2`。请记住，这些尚未做任何事情，因为 future 是惰性的，而且我们尚未等待它们。然后我们将这些 future 传递给 `trpl::select`，它返回一个值来指示传入的哪个 future 最先完成。

> 注意：在底层，`trpl::select` 是基于 `futures` crate 中定义的一个更通用的 `select` 函数构建的。`futures` crate 的 `select` 函数可以做很多 `trpl::select` 函数做不到的事情，但它也有一些额外的复杂性，我们现在可以跳过不管。

任意一个 future 都可以合法地"胜出"，因此返回 `Result` 没有意义。相反，`trpl::select` 返回一个我们之前没见过的类型——`trpl::Either`。`Either` 类型在某种程度上类似于 `Result`，它也有两种情况。但与 `Result` 不同的是，`Either` 中没有内置成功或失败的概念。相反，它使用 `Left` 和 `Right` 来表示"非此即彼"：

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

`select` 函数在第一个参数胜出时返回包含该 future 输出的 `Left`，在第二个 future 参数胜出时返回包含*该* future 输出的 `Right`。这与参数在调用函数时的顺序一致：第一个参数位于第二个参数的左侧。

我们还更新了 `page_title`，使其返回传入的相同 URL。这样，如果首先返回的页面没有可解析的 `<title>`，我们仍然可以打印一条有意义的消息。有了这些可用信息，我们更新 `println!` 输出，以指示哪个 URL 最先完成，以及该 URL 对应网页的 `<title>` 是什么（如果有的话）。

现在你已经构建了一个可以工作的小型网页抓取器！选几个 URL 并运行命令行工具。你可能会发现某些站点始终比其他站点更快，而在其他情况下，更快的站点因运行而异。更重要的是，你已经学习了使用 future 的基础知识，现在我们可以更深入地探讨 async 能做什么。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
