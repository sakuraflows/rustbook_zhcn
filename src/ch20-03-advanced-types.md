## 高级类型

Rust 的类型系统有一些我们到目前为止已经提到但尚未讨论的特性。我们将首先讨论新类型（newtypes）的一般概念，探讨它们作为类型的用处。然后，我们将转向类型别名（type aliases），这是一个类似于新类型但语义略有不同的特性。我们还将讨论 `!` 类型和动态大小的类型（dynamically sized types）。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### 使用新类型模式实现类型安全和抽象

本节假定你已经阅读了前面的["使用新类型模式实现外部 Trait"][newtype]<!-- ignore -->部分。新类型模式对于除了我们目前讨论之外的任务也很有用，包括在静态层面上确保值永远不会混淆，以及指示值的单位。你在清单 20-16 中看到了使用新类型指示单位的例子：回想一下，`Millimeters` 和 `Meters` 结构体将 `u32` 值包装在了新类型中。如果我们编写一个接受 `Millimeters` 类型参数的函数，我们将无法编译一个错误地尝试使用 `Meters` 类型的值或普通的 `u32` 值来调用该函数的程序。

我们还可以使用新类型模式来抽象掉某个类型的一些实现细节：新类型可以公开一个与私有内部类型的 API 不同的公有 API。

新类型还可以隐藏内部实现。例如，我们可以提供一个 `People` 类型来包装一个 `HashMap<i32, String>`，该哈希映射存储人的 ID 与其姓名的关联。使用 `People` 的代码只与我们提供的公有 API 交互，例如向 `People` 集合添加姓名字符串的方法；这些代码不需要知道我们在内部将 `i32` ID 分配给姓名。新类型模式是一种轻量级的实现封装以隐藏实现细节的方式，我们在第 18 章的["封装隐藏了实现细节"][encapsulation-that-hides-implementation-details]<!-- ignore -->部分讨论过。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### 类型同义词与类型别名

Rust 提供了声明*类型别名（type alias）*的功能，为现有类型赋予另一个名称。为此我们使用 `type` 关键字。例如，我们可以像这样创建 `Kilometers` 作为 `i32` 的别名：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

现在别名 `Kilometers` 是 `i32` 的*同义词（synonym）*；与我们之前在清单 20-16 中创建的 `Millimeters` 和 `Meters` 类型不同，`Kilometers` 不是一个独立的、新的类型。类型为 `Kilometers` 的值将与类型为 `i32` 的值一视同仁：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

因为 `Kilometers` 和 `i32` 是同一类型，我们可以将这两种类型的值相加，并且可以将 `Kilometers` 值传递给接受 `i32` 参数的函数。然而，使用这种方法，我们不会获得之前讨论的新类型模式带来的类型检查好处。换句话说，如果我们把 `Kilometers` 和 `i32` 值混用了，编译器不会给我们报错。

类型同义词的主要用途是减少重复。例如，我们可能会有像这样冗长的类型：

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

在函数签名和类型标注中到处编写这个冗长的类型可能既繁琐又容易出错。想象一下一个充满类似清单 20-25 中的代码的项目。

<Listing number="20-25" caption="在多个位置使用长类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

类型别名通过减少重复来使代码更易于管理。在清单 20-26 中，我们为这个冗长的类型引入了一个名为 `Thunk` 的别名，并且可以用更短的别名 `Thunk` 替换所有该类型的使用。

<Listing number="20-26" caption="引入类型别名 `Thunk` 以减少重复">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

这段代码更易于读写！为类型别名选择一个有意义的名称也有助于传达你的意图（*thunk* 是一个表示要在以后求值的代码的词，因此对于要存储的闭包来说是一个合适的名称）。

类型别名也常与 `Result<T, E>` 类型一起使用以减少重复。考虑标准库中的 `std::io` 模块。I/O 操作经常返回 `Result<T, E>` 来处理操作失败的情况。这个库有一个 `std::io::Error` 结构体，表示所有可能的 I/O 错误。`std::io` 中的许多函数会返回 `Result<T, E>`，其中 `E` 就是 `std::io::Error`，例如 `Write` trait 中的这些函数：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` 被大量重复。因此，`std::io` 有这个类型别名声明：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

因为此声明位于 `std::io` 模块中，我们可以使用完全限定别名 `std::io::Result<T>`；也就是说，一个 `E` 被填充为 `std::io::Error` 的 `Result<T, E>`。`Write` trait 的函数签名最终看起来像这样：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

类型别名在两个方面有帮助：它使代码更易于编写，*并且*它给了我们在整个 `std::io` 中一致的接口。因为它是一个别名，所以它只是另一个 `Result<T, E>`，这意味着我们可以对它使用任何适用于 `Result<T, E>` 的方法，以及像 `?` 运算符这样的特殊语法。

### 永不返回的 Never 类型

Rust 有一个名为 `!` 的特殊类型，在类型理论术语中被称为*空类型（empty type）*，因为它没有值。我们更喜欢称它为*永不返回类型（never type）*，因为当函数永远不会返回时，它站在返回类型的位置上。这里有一个例子：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

这段代码被解读为"函数 `bar` 返回 never"。返回 never 的函数被称为*发散函数（diverging functions）*。我们不能创建 `!` 类型的值，因此 `bar` 永远不可能返回。

但是你永远不能创建值的类型有什么用呢？回想一下清单 2-5 中的代码，这是猜数字游戏的一部分；我们在清单 20-27 中重现了其中的一部分。

<Listing number="20-27" caption="一个以 `continue` 结尾的分支的 `match`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

当时，我们跳过了这段代码中的一些细节。在第 6 章中的["`match` 控制流结构"][the-match-control-flow-construct]<!-- ignore -->部分，我们讨论了 `match` 分支必须都返回相同的类型。因此，例如，以下代码无法工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

这段代码中 `guess` 的类型必须既是整数*又是*字符串，而 Rust 要求 `guess` 只能有一个类型。那么，`continue` 返回了什么？我们怎么能在清单 20-27 中从一个分支返回 `u32`，而另一个分支以 `continue` 结尾呢？

正如你可能已经猜到的，`continue` 具有 `!` 值。也就是说，当 Rust 计算 `guess` 的类型时，它会查看两个匹配分支，前者具有 `u32` 值，后者具有 `!` 值。因为 `!` 永远不可能有值，Rust 判定 `guess` 的类型是 `u32`。

描述这种行为的正式方式是，`!` 类型的表达式可以被强制转换为任何其他类型。我们允许用 `continue` 结束这个 `match` 分支，因为 `continue` 不返回值；相反，它将控制权移回到循环的顶部，因此在 `Err` 的情况下，我们从未给 `guess` 赋值。

never 类型在 `panic!` 宏中也很有用。回想一下我们在 `Option<T>` 值上调用的 `unwrap` 函数，它要么产生一个值，要么 panic，其定义如下：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

在这段代码中，发生与清单 20-27 的 `match` 相同的事情：Rust 看到 `val` 的类型是 `T`，`panic!` 的类型是 `!`，因此整个 `match` 表达式的结果类型是 `T`。这段代码之所以工作，是因为 `panic!` 不产生值；它结束程序。在 `None` 的情况下，我们不会从 `unwrap` 返回值，因此这段代码是有效的。

最后一个具有 `!` 类型的表达式是循环：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

这里，循环永远不会结束，因此 `!` 是该表达式的值。然而，如果我们包含了一个 `break`，情况就不是这样了，因为循环在到达 `break` 时会终止。

### 动态大小类型和 `Sized` Trait

Rust 需要知道其类型的某些细节，例如为特定类型的值分配多少空间。这使得其类型系统的一个角落一开始有些令人困惑：*动态大小类型（dynamically sized types）*的概念。有时被称为 *DST* 或*不定大小类型（unsized types）*，这些类型让我们可以编写使用那些我们只能在运行时才知道其大小的值的代码。

让我们深入了解一下名为 `str` 的动态大小类型的细节，我们在本书中一直在使用它。没错，不是 `&str`，而是单独的 `str`，就是一种 DST。在许多情况下，例如存储用户输入的文本时，我们直到运行时才知道字符串有多长。这意味着我们不能创建 `str` 类型的变量，也不能接受 `str` 类型的参数。考虑以下无法工作的代码：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust 需要知道如何为特定类型的任何值分配内存，并且一个类型的所有值必须使用相同大小的内存。如果 Rust 允许我们编写这段代码，这两个 `str` 值将需要占用相同的空间。但它们的长度不同：`s1` 需要 12 字节的存储空间，而 `s2` 需要 15 字节。这就是为什么不可能创建一个持有动态大小类型的变量。

那么，我们该怎么办？在这种情况下，你已经知道答案了：我们将 `s1` 和 `s2` 的类型设为字符串切片（`&str`）而不是 `str`。回顾第 4 章中的["字符串切片"][string-slices]<!-- ignore -->部分，切片数据结构只存储起始位置和切片的长度。因此，虽然 `&T` 是一个存储 `T` 所在内存地址的单一值，但一个字符串切片是*两个*值：`str` 的地址和它的长度。因此，我们可以在编译时知道一个字符串切片值的大小：它是 `usize` 长度的两倍。也就是说，无论它所引用的字符串有多长，我们始终知道字符串切片的大小。通常，这就是在 Rust 中使用动态大小类型的方式：它们有一些额外的元数据来存储动态信息的大小。动态大小类型的黄金法则是，我们总是必须将动态大小类型的值放在某种指针的后面。

我们可以将 `str` 与各种指针组合：例如 `Box<str>` 或 `Rc<str>`。实际上，你以前见过这个，但用的是不同的动态大小类型：trait。每个 trait 都是一个动态大小类型，我们可以通过使用 trait 的名称来引用它。在第 18 章的["使用 Trait 对象对共享行为进行抽象"][using-trait-objects-to-abstract-over-shared-behavior]<!-- ignore -->部分，我们提到要将 trait 用作 trait 对象，必须将它们放在指针后面，例如 `&dyn Trait` 或 `Box<dyn Trait>`（`Rc<dyn Trait>` 也可以）。

为了处理 DST，Rust 提供了 `Sized` trait 来确定一个类型的大小是否在编译时已知。对于所有大小在编译时已知的类型，该 trait 会自动实现。此外，Rust 隐式地为每个泛型函数添加一个 `Sized` 约束。也就是说，一个像这样的泛型函数定义：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

实际上被视为我们编写了这样：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

默认情况下，泛型函数只能用于那些在编译时具有已知大小的类型。然而，你可以使用以下特殊语法来放松这一限制：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized` 上的 trait 约束意味着"`T` 可能是 `Sized`，也可能不是"，这种表示法覆盖了泛型类型必须在编译时具有已知大小的默认值。具有此含义的 `?Trait` 语法仅适用于 `Sized`，不适用于任何其他 trait。

还要注意，我们将 `t` 参数的类型从 `T` 改为了 `&T`。因为类型可能不是 `Sized`，我们需要将其放在某种指针的后面。在这种情况下，我们选择了引用。

接下来，我们将讨论函数和闭包！

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
