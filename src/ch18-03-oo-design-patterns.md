## 实现面向对象设计模式

*状态模式（state pattern）*是一种面向对象的设计模式。该模式的核心是，我们定义一组值在内部可以具有的状态。这些状态由一组*状态对象（state objects）*表示，值的行为会根据其状态发生变化。我们将通过一个博客文章结构体的示例来实践一下，该结构体有一个用于保存其状态的字段，该状态将是"草稿（draft）"、"审核（review）"或"已发布（published）"集合中的一个状态对象。

状态对象共享功能：在 Rust 中，当然，我们使用结构体和 trait，而不是对象和继承。每个状态对象负责自己的行为，并负责控制何时应该转换为另一种状态。持有状态对象的值对状态的不同行为以及状态之间的转换时机一无所知。

使用状态模式的好处是，当程序的业务需求发生变化时，我们不需要更改持有状态的值的代码或使用该值的代码。我们只需要更新某个状态对象内部的代码来改变其规则，或者可能添加更多的状态对象。

首先，我们将以更传统的面向对象方式实现状态模式。然后，我们将使用一种在 Rust 中更为自然的方法。让我们深入了解一下，使用状态模式逐步实现一个博客文章工作流。

最终的功能将如下所示：

1. 博客文章以空白草稿开始。
2. 草稿完成后，请求对文章进行审核。
3. 文章批准后，它就会被发布。
4. 只有已发布的博客文章才会返回要打印的内容，因此未经批准的文章不会意外发布。

任何其他对文章的更改尝试都无效。例如，如果我们试图在请求审核之前就批准一篇草稿博客文章，该文章应保持为未发布的草稿。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="a-traditional-object-oriented-attempt"></a>

### 尝试传统面向对象风格

解决同一个问题有无数种代码组织方式，每种方式都有不同的权衡。本章节的实现更接近传统的面向对象风格，这在 Rust 中是可以实现的，但并没有充分利用 Rust 的一些优势。稍后，我们将展示另一种解决方案，它仍然使用面向对象的设计模式，但组织方式对于有面向对象经验的程序员来说可能看起来不那么熟悉。我们将比较这两种方案，以体验在 Rust 中设计代码与在其他语言中设计代码的权衡。

清单 18-11 以代码形式展示了这个工作流：这是一个在名为 `blog` 的库 crate 中实现 API 的使用示例。由于我们还没有实现 `blog` crate，这段代码还不能编译。

<Listing number="18-11" file-name="src/main.rs" caption="展示我们希望 `blog` crate 具有的所需行为的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

我们希望允许用户使用 `Post::new` 创建一个新的草稿博客文章。我们希望允许向博客文章添加文本。如果我们尝试在审批之前立即获取文章的内容，我们不应该得到任何文本，因为该文章仍处于草稿状态。我们在代码中添加了 `assert_eq!` 用于演示目的。一个优秀的单元测试应该是断言草稿博客文章的 `content` 方法返回空字符串，但本示例中我们不打算编写测试。

接下来，我们希望能够为文章请求审核，并且在等待审核期间，`content` 方法应返回空字符串。当文章获得批准后，它应该被发布，这意味着当调用 `content` 时，将返回文章的文本。

请注意，我们从 crate 中交互的唯一类型是 `Post` 类型。该类型将使用状态模式，并持有一个值，该值将是三个状态对象中的一个，表示文章可能处于的各种状态——草稿（draft）、审核（review）或已发布（published）。从一个状态到另一个状态的更改将在 `Post` 类型内部进行管理。状态的改变是响应库用户对 `Post` 实例调用的方法，但他们不必直接管理状态转换。此外，用户不会在状态上犯错，例如在审核之前发布文章。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### 定义 `Post` 并创建新实例

让我们开始实现这个库！我们知道需要一个持有一些内容的公有 `Post` 结构体，因此我们将从结构体的定义和一个关联的公有 `new` 函数开始，用于创建 `Post` 的实例，如清单 18-12 所示。我们还将创建一个私有的 `State` trait，它将定义所有 `Post` 的状态对象必须具有的行为。

然后，`Post` 将在名为 `state` 的私有字段中持有一个 `Box<dyn State>` 的 trait 对象，并将其包装在 `Option<T>` 中，用于保存状态对象。稍后你会明白为什么 `Option<T>` 是必需的。

<Listing number="18-12" file-name="src/lib.rs" caption="`Post` 结构体的定义、创建新 `Post` 实例的 `new` 函数、`State` trait 以及 `Draft` 结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` trait 定义了不同文章状态共享的行为。状态对象是 `Draft`、`PendingReview` 和 `Published`，它们都将实现 `State` trait。目前，该 trait 没有任何方法，我们将从仅定义 `Draft` 状态开始，因为这是我们希望文章开始所处的状态。

当我们创建一个新的 `Post` 时，将其 `state` 字段设置为一个包含 `Box` 的 `Some` 值。这个 `Box` 指向一个新的 `Draft` 结构体实例。这确保了每当我们创建新的 `Post` 实例时，它都会以草稿状态开始。由于 `Post` 的 `state` 字段是私有的，因此无法以任何其他状态创建 `Post`！在 `Post::new` 函数中，我们将 `content` 字段设置为一个新的空 `String`。

#### 存储文章内容的文本

我们在清单 18-11 中看到，我们希望能够调用一个名为 `add_text` 的方法，并向其传递一个 `&str`，然后将其添加为博客文章的文本内容。我们将其实现为一个方法，而不是将 `content` 字段暴露为 `pub`，以便稍后可以实现一个控制 `content` 字段数据读取方式的方法。`add_text` 方法非常简单明了，因此让我们在清单 18-13 中将该实现添加到 `impl Post` 块中。

<Listing number="18-13" file-name="src/lib.rs" caption="实现 `add_text` 方法以向文章的内容添加文本">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` 方法获取 `self` 的可变引用，因为我们正在更改调用 `add_text` 的 `Post` 实例。然后我们在 `content` 中的 `String` 上调用 `push_str`，并将 `text` 参数传递进去以添加到保存的 `content` 中。此行为不依赖于文章所处的状态，因此它不是状态模式的一部分。`add_text` 方法完全不与 `state` 字段交互，但它确实是我们希望支持的行为的一部分。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### 确保草稿文章的内容为空

即使在我们调用了 `add_text` 并向文章添加了一些内容之后，我们仍然希望 `content` 方法返回一个空字符串切片，因为该文章仍处于草稿状态，如清单 18-11 中的第一个 `assert_eq!` 所示。现在，让我们以最简单的方式来满足这个要求：始终返回一个空字符串切片。稍后，一旦我们实现了更改文章状态以便其可以发布的功能时，再对此进行修改。到目前为止，文章只能处于草稿状态，因此文章内容应始终为空。清单 18-14 显示了这个占位实现。

<Listing number="18-14" file-name="src/lib.rs" caption="为 `Post` 上的 `content` 方法添加一个始终返回空字符串切片的占位实现">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

通过添加这个 `content` 方法，清单 18-11 中直到第一个 `assert_eq!` 的所有内容都能如预期工作了。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### 请求审核，改变文章状态

接下来，我们需要添加请求审核文章的功能，这应将其状态从 `Draft` 改为 `PendingReview`。清单 18-15 展示了这段代码。

<Listing number="18-15" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `request_review` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

我们为 `Post` 赋予一个名为 `request_review` 的公有方法，该方法接受 `self` 的可变引用。然后，我们在 `Post` 的当前状态上调用一个内部的 `request_review` 方法，而这个第二个 `request_review` 方法会消费当前状态并返回一个新状态。

我们将 `request_review` 方法添加到 `State` trait；所有实现该 trait 的类型现在都需要实现 `request_review` 方法。请注意，该方法的第一个参数不是 `self`、`&self` 或 `&mut self`，而是 `self: Box<Self>`。这种语法意味着该方法仅在持有该类型的 `Box` 上调用时才有效。这种语法获取 `Box<Self>` 的所有权，使旧状态失效，从而 `Post` 的状态值可以转换为新状态。

为了消费旧状态，`request_review` 方法需要获取状态值的所有权。这就是 `Post` 的 `state` 字段中 `Option` 的用武之地：我们调用 `take` 方法将 `Some` 值从 `state` 字段中取出，并在其位置留下一个 `None`，因为 Rust 不允许我们在结构体中存在未填充的字段。这样我们就可以将 `state` 值从 `Post` 中移出，而不是借用它。然后，我们将 `Post` 的 `state` 值设置为此操作的结果。

我们需要将 `state` 临时设置为 `None`，而不是使用像 `self.state = self.state.request_review();` 这样的代码来直接获取 `state` 值的所有权。这确保了在我们将其转换为新状态之后，`Post` 不能再使用旧的 `state` 值。

`Draft` 上的 `request_review` 方法返回一个新的、装箱的 `PendingReview` 结构体实例，该结构体表示文章正在等待审核的状态。`PendingReview` 结构体也实现了 `request_review` 方法，但不执行任何转换。相反，它返回自身，因为当我们对已经处于 `PendingReview` 状态的文章请求审核时，它应保持在 `PendingReview` 状态。

现在我们可以开始看到状态模式的优势：无论 `state` 的值是什么，`Post` 上的 `request_review` 方法都是相同的。每个状态都负责自己的规则。

我们将保持 `Post` 上的 `content` 方法不变，返回一个空字符串切片。现在，`Post` 既可以处于 `PendingReview` 状态，也可以处于 `Draft` 状态，但我们希望在 `PendingReview` 状态下也具有相同的行为。清单 18-11 现在可以工作到第二个 `assert_eq!` 调用了！

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### 添加 `approve` 以改变 `content` 的行为

`approve` 方法将与 `request_review` 方法类似：它将 `state` 设置为当前状态在该状态被批准时应该具有的值，如清单 18-16 所示。

<Listing number="18-16" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

我们将 `approve` 方法添加到 `State` trait，并添加一个新的实现了 `State` 的结构体——`Published` 状态。

类似于 `PendingReview` 上的 `request_review` 的工作方式，如果我们在 `Draft` 上调用 `approve` 方法，它将没有效果，因为 `approve` 将返回 `self`。当我们在 `PendingReview` 上调用 `approve` 时，它返回一个新的、装箱的 `Published` 结构体实例。`Published` 结构体实现了 `State` trait，并且对于 `request_review` 方法和 `approve` 方法，它都返回自身，因为在这两种情况下，文章应保持为已发布状态。

现在我们需要更新 `Post` 上的 `content` 方法：如果状态是 `Published`，我们想要返回 `content` 字段中的值；否则，我们想要返回一个空字符串切片。清单 18-17 展示了这一点。

<Listing number="18-17" file-name="src/lib.rs" caption="更新 `Post` 上的 `content` 方法以委托给 `State` 的 `content` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

因为目标是将所有这些行为委托给状态，所以我们在 `State` trait 上定义了一个名为 `content` 的方法，它接受一个博客文章的结构体引用作为参数并返回一个可选的字符串切片值。为 `content` 方法创建默认实现，返回 `None`，这表示我们不需要在每个状态对象上都实现 `content`。`Published` 结构体将覆盖 `content` 方法来返回 `post.content` 中的值。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="trade-offs-of-the-state-pattern"></a>

#### 评估状态模式

我们已经展示了 Rust 能够实现面向对象的状态模式，以封装文章在每个状态下应具有的不同类型的行为。`Post` 上的方法对各种行为一无所知。由于我们组织代码的方式，我们只需要在一个地方就知道已发布文章可以表现出的不同方式：在 `Published` 结构体上实现的 `State` trait。

如果我们创建一个不使用状态模式的替代实现，我们可能会在 `Post` 的方法中使用 `match` 表达式，甚至会在 `main` 代码中检查文章的状态并更改那些位置的行为。这将意味着我们需要查看多个地方才能理解文章处于已发布状态的所有含义。

使用状态模式，`Post` 的方法以及我们使用 `Post` 的地方都不需要 `match` 表达式，而要添加一个新状态，我们只需要添加一个新结构体并在一个位置为该结构体实现 trait 方法。

使用状态模式的实现很容易扩展以添加更多功能。为了了解使用状态模式维护代码的简洁性，试试以下几个建议：

- 添加一个 `reject` 方法，将文章的状态从 `PendingReview` 改回 `Draft`。
- 在状态可以更改为 `Published` 之前，需要两次调用 `approve`。
- 仅当文章处于 `Draft` 状态时，才允许用户添加文本内容。提示：让状态对象负责可能更改的内容，但不负责修改 `Post`。

状态模式的一个缺点是，由于状态实现了状态之间的转换，某些状态会相互耦合。如果我们在 `PendingReview` 和 `Published` 之间添加另一个状态，例如 `Scheduled`，我们将不得不更改 `PendingReview` 中的代码以转换到 `Scheduled` 而不是 `Published`。如果 `PendingReview` 不需要随新状态的添加而更改，那工作量会少一些，但这意味着要切换到另一种设计模式。

另一个缺点是，我们重复了一些逻辑。为了消除部分重复，我们可以尝试在 `State` trait 上为 `request_review` 和 `approve` 方法提供返回 `self` 的默认实现。然而，这种方法行不通：当使用 `State` 作为 trait 对象时，trait 并不知道具体的 `self` 究竟是什么，因此返回类型在编译时是未知的。（这是前面提到的 dyn 兼容性规则之一。）

其他的重复包括 `Post` 上 `request_review` 和 `approve` 方法的类似实现。两个方法都在 `Post` 的 `state` 字段上使用 `Option::take`，并且如果 `state` 是 `Some`，它们将委托给包装值的相同方法的实现，并将 `state` 字段的新值设置为结果。如果 `Post` 上有许多方法遵循这种模式，我们可能会考虑定义一个宏来消除重复（请参阅第 20 章的["宏"][macros]<!-- ignore -->部分）。

按照面向对象语言中定义的状态模式精确实现，我们并没有尽可能充分利用 Rust 的优势。让我们来看看我们可以对 `blog` crate 做出的一些更改，这些更改可以将无效状态和转换变成编译时错误。

### 将状态和行为编码为类型

我们将向你展示如何重新思考状态模式以获得另一套权衡方案。与其完全封装状态和转换以使外部代码对它们一无所知，不如将状态编码到不同的类型中。因此，Rust 的类型检查系统将阻止尝试使用草稿文章（而只允许已发布的文章）的行为，并发出编译器错误。

让我们考虑清单 18-11 中 `main` 的第一部分：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

我们仍然允许使用 `Post::new` 创建草稿状态的新文章，并允许向文章内容添加文本。但是，我们不希望草稿文章有一个返回空字符串的 `content` 方法，而是让它根本没有 `content` 方法。这样，如果我们尝试获取草稿文章的内容，将会得到一个编译器错误，告诉我们该方法不存在。因此，我们不可能在生产中意外显示草稿文章的内容，因为那段代码甚至无法编译。清单 18-19 展示了 `Post` 结构体和 `DraftPost` 结构体的定义，以及各自的方法。

<Listing number="18-19" file-name="src/lib.rs" caption="一个带有 `content` 方法的 `Post` 和一个没有 `content` 方法的 `DraftPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

`Post` 和 `DraftPost` 结构体都有一个私有的 `content` 字段，用于存储博客文章的文本。结构体不再有 `state` 字段，因为我们正在将状态的编码转移到结构体的类型上。`Post` 结构体表示已发布的文章，并且它有一个返回 `content` 的 `content` 方法。

我们仍然有一个 `Post::new` 函数，但它不再返回 `Post` 的实例，而是返回 `DraftPost` 的实例。由于 `content` 是私有的，并且没有返回 `Post` 的函数，因此目前无法创建 `Post` 的实例。

`DraftPost` 结构体有一个 `add_text` 方法，因此我们可以像以前一样向 `content` 添加文本，但是请注意，`DraftPost` 没有定义 `content` 方法！因此，现在程序确保所有文章都以草稿文章开始，而草稿文章的内容不可用于显示。任何绕过这些约束的尝试都会导致编译器错误。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

那么，我们如何获得已发布的文章呢？我们希望强制执行规则：草稿文章必须先经过审核和批准才能发布。处于待审核状态的文章仍然不应显示任何内容。让我们通过添加另一个结构体 `PendingReviewPost` 来实现这些约束，在 `DraftPost` 上定义 `request_review` 方法以返回 `PendingReviewPost`，并在 `PendingReviewPost` 上定义 `approve` 方法以返回 `Post`，如清单 18-20 所示。

<Listing number="18-20" file-name="src/lib.rs" caption="通过在 `DraftPost` 上调用 `request_review` 创建的 `PendingReviewPost`，以及将 `PendingReviewPost` 转换为已发布的 `Post` 的 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review` 和 `approve` 方法获取 `self` 的所有权，从而消费 `DraftPost` 和 `PendingReviewPost` 实例，并分别将它们转换为 `PendingReviewPost` 和已发布的 `Post`。这样，在我们对其调用 `request_review` 之后，就不会再有任何 `DraftPost` 实例残留，依此类推。`PendingReviewPost` 结构体上没有定义 `content` 方法，因此尝试读取其内容会导致编译器错误，就像 `DraftPost` 一样。由于获得已发布 `Post` 实例（其上定义了 `content` 方法）的唯一方法是调用 `PendingReviewPost` 上的 `approve` 方法，而获得 `PendingReviewPost` 的唯一方法是调用 `DraftPost` 上的 `request_review` 方法，我们现在已将博客文章工作流编码到类型系统中了。

但我们也需要对 `main` 做一些小的更改。`request_review` 和 `approve` 方法返回新实例，而不是修改它们所调用的结构体，因此我们需要添加更多 `let post =` 的变量遮蔽（shadowing）赋值来保存返回的实例。我们也不需要关于草稿和待审核文章内容为空字符串的断言了：我们也不再需要它们；我们无法再编译试图使用那些状态下文章内容的代码。更新后的 `main` 代码如清单 18-21 所示。

<Listing number="18-21" file-name="src/main.rs" caption="修改 `main` 以使用博客文章工作流的新实现">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

我们需要对 `main` 做出更改来重新赋值 `post`，这意味着此实现不再完全遵循面向对象的状态模式：状态之间的转换不再完全封装在 `Post` 实现内部。然而，我们的收获是，由于类型系统以及编译时发生的类型检查，无效状态现在是不可能的了！这确保了某些 bug，例如显示未发布文章的内容，将在它们进入生产环境之前被发现。

尝试在本节开始时对 `blog` crate 建议的任务，就像它在清单 18-21 之后的状态一样，看看你对这个版本代码的设计看法。请注意，在此设计中，某些任务可能已经完成了。

我们已经看到，尽管 Rust 能够实现面向对象的设计模式，但其他模式（例如将状态编码到类型系统中）在 Rust 中也是可用的。这些模式有不同的权衡。虽然你可能对面向对象模式非常熟悉，但重新思考问题以利用 Rust 的特性可以带来好处，例如在编译时防止某些 bug。由于 Rust 拥有像所有权这样面向对象语言不具备的某些特性，面向对象的模式并不总是 Rust 中的最佳解决方案。

## 总结

不管你读完本章后认为 Rust 是不是面向对象的语言，你现在知道你可以使用 trait 对象在 Rust 中获得一些面向对象的特性。动态派发可以为你的代码提供一些灵活性，但代价是少量的运行时性能。你可以利用这种灵活性来实现面向对象的模式，从而有助于代码的可维护性。Rust 还具有其他面向对象语言所没有的特性，例如所有权。面向对象模式并不总是利用 Rust 优势的最佳方式，但它是一个可用的选项。

接下来，我们将探讨模式（patterns），这是 Rust 的另一个特性，它提供了很大的灵活性。我们在本书中已经简短地看过它们，但还没有看到它们的全部能力。让我们开始吧！

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
