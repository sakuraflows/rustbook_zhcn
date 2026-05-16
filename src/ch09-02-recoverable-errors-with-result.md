## 使用 `Result` 的可恢复错误

大多数错误并不严重到需要程序完全停止。有时当函数失败时，是由于一个你可以轻松解释并做出反应的原因。例如，如果你尝试打开一个文件，但由于文件不存在而失败，你可能想要创建文件而不是终止进程。

回想一下第 2 章中[“使用 `Result` 处理潜在失败”][handle_failure]<!-- ignore -->部分，`Result` 枚举被定义为有两个变体（variant），`Ok` 和 `Err`，如下所示：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 和 `E` 是泛型类型参数（generic type parameters）：我们将在第 10 章更详细地讨论泛型。你现在需要知道的是，`T` 表示在成功情况下 `Ok` 变体中将返回的值的类型，而 `E` 表示在失败情况下 `Err` 变体中将返回的错误类型。由于 `Result` 具有这些泛型类型参数，我们可以在许多不同的情况下使用 `Result` 类型及其上定义的函数，这些情况下我们想要返回的成功值和错误值可能不同。

让我们调用一个返回 `Result` 值的函数，因为该函数可能会失败。在示例 9-3 中，我们尝试打开一个文件。

<Listing number="9-3" file-name="src/main.rs" caption="打开一个文件">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

`File::open` 的返回类型是 `Result<T, E>`。泛型参数 `T` 已被 `File::open` 的实现填充为成功值的类型，即 `std::fs::File`，这是一个文件句柄（file handle）。错误值中使用的 `E` 的类型是 `std::io::Error`。这个返回类型意味着对 `File::open` 的调用可能成功并返回一个我们可以读取或写入的文件句柄。该函数调用也可能失败：例如，文件可能不存在，或者我们可能没有权限访问该文件。`File::open` 函数需要有一种方式来告诉我们它是否成功或失败，同时给我们文件句柄或错误信息。这正是 `Result` 枚举所传达的信息。

在 `File::open` 成功的情况下，变量 `greeting_file_result` 中的值将是一个包含文件句柄的 `Ok` 实例。在失败的情况下，`greeting_file_result` 中的值将是一个包含更多错误信息的 `Err` 实例。

我们需要向示例 9-3 中的代码添加一些内容，以根据 `File::open` 返回的值执行不同的操作。示例 9-4 展示了一种使用基本工具——我们在第 6 章讨论过的 `match` 表达式——来处理 `Result` 的方法。

<Listing number="9-4" file-name="src/main.rs" caption="使用 `match` 表达式处理可能返回的 `Result` 变体">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

请注意，与 `Option` 枚举一样，`Result` 枚举及其变体已由 prelude（预导入）引入作用域，因此我们不需要在 `match` 分支中的 `Ok` 和 `Err` 变体前指定 `Result::`。

当结果是 `Ok` 时，这段代码将返回 `Ok` 变体中的内部 `file` 值，然后我们将该文件句柄值赋给变量 `greeting_file`。在 `match` 之后，我们可以使用文件句柄进行读取或写入。

`match` 的另一个分支处理我们从 `File::open` 得到 `Err` 值的情况。在这个例子中，我们选择了调用 `panic!` 宏。如果当前目录中没有名为 _hello.txt_ 的文件，并且我们运行这段代码，我们将看到来自 `panic!` 宏的以下输出：

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

和往常一样，这个输出准确地告诉我们出了什么问题。

### 对不同错误进行匹配

示例 9-4 中的代码无论 `File::open` 失败的原因是什么，都会 `panic!`。然而，我们想针对不同的失败原因采取不同的操作。如果 `File::open` 因为文件不存在而失败，我们希望创建该文件并返回新文件的句柄。如果 `File::open` 因任何其他原因失败——例如，因为我们没有打开文件的权限——我们仍然希望代码像示例 9-4 中那样 `panic!`。为此，我们添加一个内部的 `match` 表达式，如示例 9-5 所示。

<Listing number="9-5" file-name="src/main.rs" caption="以不同方式处理不同类型的错误">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

`File::open` 在 `Err` 变体内部返回的值的类型是 `io::Error`，这是标准库提供的一个结构体。这个结构体有一个方法 `kind`，我们可以调用它来获取一个 `io::ErrorKind` 值。`io::ErrorKind` 枚举由标准库提供，其变体表示可能由 `io` 操作导致的各种错误类型。我们想要使用的变体是 `ErrorKind::NotFound`，它表示我们试图打开的文件尚不存在。因此，我们在 `greeting_file_result` 上进行匹配，但我们还在 `error.kind()` 上进行内部匹配。

我们在内部匹配中想要检查的条件是 `error.kind()` 返回的值是否是 `ErrorKind` 枚举的 `NotFound` 变体。如果是，我们尝试用 `File::create` 创建文件。然而，因为 `File::create` 也可能失败，所以我们需要内部 `match` 表达式中的第二个分支。当文件无法创建时，会打印不同的错误消息。外部 `match` 的第二个分支保持不变，因此除了文件缺失错误之外，任何其他错误都会导致程序 panic。

> #### 使用 `match` 处理 `Result<T, E>` 的替代方案
>
> 那是很多的 `match`！`match` 表达式非常有用，但也非常原始。在第 13 章中，你将学习闭包（closures），它们与 `Result<T, E>` 上定义的许多方法一起使用。在处理代码中的 `Result<T, E>` 值时，这些方法可能比使用 `match` 更简洁。
>
> 例如，以下是另一种编写与示例 9-5 相同逻辑的方式，这次使用闭包和 `unwrap_or_else` 方法：
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> 尽管这段代码与示例 9-5 具有相同的行为，但它不包含任何 `match` 表达式，并且阅读起来更清晰。在你阅读完第 13 章后，请回到这个例子，并在标准库文档中查阅 `unwrap_or_else` 方法。在处理错误时，还有更多这样的方法可以清理庞大、嵌套的 `match` 表达式。

<!-- Old headings. Do not remove or links may break. -->

<a id="shortcuts-for-panic-on-error-unwrap-and-expect"></a>

#### 错误时 panic 的快捷方式

使用 `match` 效果很好，但它可能有点冗长，并且不一定能很好地传达意图。`Result<T, E>` 类型上定义了许多辅助方法，用于执行各种更具体的任务。`unwrap` 方法是一个快捷方式方法，其实现方式与我们写在示例 9-4 中的 `match` 表达式类似。如果 `Result` 值是 `Ok` 变体，`unwrap` 将返回 `Ok` 内部的值。如果 `Result` 是 `Err` 变体，`unwrap` 将为我们调用 `panic!` 宏。以下是 `unwrap` 在行动中的示例：

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

如果我们没有 _hello.txt_ 文件就运行这段代码，我们将看到来自 `unwrap` 方法调用的 `panic!` 的错误消息：

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

类似地，`expect` 方法让我们也可以选择 `panic!` 的错误消息。使用 `expect` 而不是 `unwrap` 并提供良好的错误消息可以传达你的意图，并使追踪 panic 的来源更容易。`expect` 的语法如下所示：

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

我们使用 `expect` 的方式与 `unwrap` 相同：返回文件句柄或调用 `panic!` 宏。`expect` 在其调用 `panic!` 时使用的错误消息将是我们传递给 `expect` 的参数，而不是 `unwrap` 使用的默认 `panic!` 消息。以下是它的样子：

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

在生产质量的代码中，大多数 Rustaceans 选择 `expect` 而不是 `unwrap`，并提供更多关于为什么该操作预期总是成功的上下文。这样，如果你的假设被证明是错误的，你在调试时就有更多信息可以使用。

### 传播错误

当函数的实现调用可能失败的内容时，你可以不在此函数内部处理错误，而是将错误返回给调用代码，以便它可以决定该怎么做。这被称为*传播（propagating）*错误，并赋予调用代码更多的控制权，因为调用代码可能拥有更多信息或逻辑来决定应如何处理错误，而不是在你的代码上下文中可用。

例如，示例 9-6 显示了一个从文件读取用户名的函数。如果文件不存在或无法读取，此函数将把这些错误返回给调用它的代码。

<Listing number="9-6" file-name="src/main.rs" caption="使用 `match` 将错误返回给调用代码的函数">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

这个函数可以用更简短的方式编写，但我们先手动做很多工作来探索错误处理；最后，我们将展示更简短的方式。首先看看函数的返回类型：`Result<String, io::Error>`。这意味着该函数返回一个 `Result<T, E>` 类型的值，其中泛型参数 `T` 已被具体类型 `String` 填充，泛型类型 `E` 已被具体类型 `io::Error` 填充。

如果此函数成功且没有任何问题，调用此函数的代码将收到一个包含 `String` 的 `Ok` 值——即此函数从文件中读取的 `username`。如果此函数遇到任何问题，调用代码将收到一个包含 `io::Error` 实例的 `Err` 值，其中包含有关问题所在的更多信息。我们选择 `io::Error` 作为此函数的返回类型，因为碰巧我们在此函数体中调用的两个可能失败的操作返回的错误值的类型都是它：`File::open` 函数和 `read_to_string` 方法。

函数体首先调用 `File::open` 函数。然后，我们使用 `match` 处理 `Result` 值，类似于示例 9-4 中的 `match`。如果 `File::open` 成功，模式变量 `file` 中的文件句柄成为可变变量 `username_file` 中的值，函数继续执行。在 `Err` 情况下，我们不调用 `panic!`，而是使用 `return` 关键字从函数中提前返回，并将 `File::open` 中的错误值（现在在模式变量 `e` 中）作为此函数的错误值传回给调用代码。

因此，如果在 `username_file` 中有一个文件句柄，则该函数在变量 `username` 中创建一个新的 `String`，并调用 `username_file` 中文件句柄上的 `read_to_string` 方法，将文件内容读入 `username`。`read_to_string` 方法也返回一个 `Result`，因为它可能失败，即使 `File::open` 成功了。所以，我们需要另一个 `match` 来处理那个 `Result`：如果 `read_to_string` 成功，那么我们的函数成功了，我们返回文件中的用户名（现在在 `username` 中），包裹在 `Ok` 中。如果 `read_to_string` 失败，我们以与处理 `File::open` 返回值的 `match` 中相同的错误值返回方式返回错误值。然而，我们不需要显式写出 `return`，因为这是函数中的最后一个表达式。

调用此代码的代码将随后处理得到包含用户名的 `Ok` 值或包含 `io::Error` 的 `Err` 值。由调用代码决定如何处理这些值。如果调用代码得到 `Err` 值，它可以调用 `panic!` 并使程序崩溃，使用默认用户名，或者从文件以外的其他地方查找用户名，等等。我们没有足够的信息知道调用代码实际想做什么，因此我们将所有成功或错误信息向上传播以供其适当处理。

这种错误传播模式在 Rust 中非常常见，以至于 Rust 提供了问号运算符 `?` 来使其更容易。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-shortcut-for-propagating-errors-the--operator"></a>

#### `?` 运算符快捷方式

示例 9-7 展示了 `read_username_from_file` 的一个实现，它具有与示例 9-6 相同的功能，但此实现使用了 `?` 运算符。

<Listing number="9-7" file-name="src/main.rs" caption="使用 `?` 运算符将错误返回给调用代码的函数">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

放在 `Result` 值后面的 `?` 被定义为与我们在示例 9-6 中定义的用于处理 `Result` 值的 `match` 表达式几乎完全相同的方式工作。如果 `Result` 的值是 `Ok`，`Ok` 内部的值将从此表达式返回，程序继续执行。如果值是 `Err`，`Err` 将像我们使用了 `return` 关键字一样从整个函数返回，从而将错误值传播给调用代码。

示例 9-6 中的 `match` 表达式与 `?` 运算符之间有一个区别：对其调用 `?` 运算符的错误值会通过标准库中 `From` trait 定义的 `from` 函数，该函数用于将值从一种类型转换为另一种类型。当 `?` 运算符调用 `from` 函数时，接收到的错误类型会被转换为当前函数返回类型中定义的错误类型。当一个函数返回一种错误类型来表示函数可能失败的所有方式时，即使某些部分可能因多种不同原因而失败，这非常有用。

例如，我们可以将示例 9-7 中的 `read_username_from_file` 函数改为返回我们定义的自定义错误类型 `OurError`。如果我们还定义了 `impl From<io::Error> for OurError` 来从 `io::Error` 构造一个 `OurError` 实例，那么 `read_username_from_file` 函数体中的 `?` 运算符调用将调用 `from` 并转换错误类型，而无需向函数添加任何额外的代码。

在示例 9-7 的上下文中，`File::open` 调用末尾的 `?` 将返回 `Ok` 内部的值给变量 `username_file`。如果发生错误，`?` 运算符将从整个函数提前返回，并将任何 `Err` 值交给调用代码。同样的情况也适用于 `read_to_string` 调用末尾的 `?`。

`?` 运算符消除了大量样板代码，并使此函数的实现更简单。我们甚至可以通过在 `?` 之后立即链式调用方法进一步缩短这段代码，如示例 9-8 所示。

<Listing number="9-8" file-name="src/main.rs" caption="在 `?` 运算符之后链式调用方法">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

我们将 `username` 中新 `String` 的创建移到了函数的开头；这部分没有改变。我们没有创建变量 `username_file`，而是将对 `read_to_string` 的调用直接链式接到 `File::open("hello.txt")?` 的结果上。我们仍然在 `read_to_string` 调用的末尾有一个 `?`，并且当 `File::open` 和 `read_to_string` 都成功而不是返回错误时，我们仍然返回一个包含 `username` 的 `Ok` 值。功能再次与示例 9-6 和示例 9-7 相同；这只是另一种更符合人体工程学（ergonomic）的编写方式。

示例 9-9 展示了使用 `fs::read_to_string` 使这段代码更简短的一种方式。

<Listing number="9-9" file-name="src/main.rs" caption="使用 `fs::read_to_string` 代替打开然后读取文件">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

将文件读入字符串是一个相当常见的操作，因此标准库提供了方便的 `fs::read_to_string` 函数，它打开文件、创建新的 `String`、读取文件内容、将内容放入该 `String` 并返回它。当然，使用 `fs::read_to_string` 没有给我们解释所有错误处理的机会，所以我们先用了较长的写法。

<!-- Old headings. Do not remove or links may break. -->

<a id="where-the--operator-can-be-used"></a>

#### 在哪里可以使用 `?` 运算符

`?` 运算符只能用于返回类型与 `?` 所使用的值兼容的函数中。这是因为 `?` 运算符被定义为从函数中提前返回一个值，方式与我们在示例 9-6 中定义的 `match` 表达式相同。在示例 9-6 中，`match` 使用了一个 `Result` 值，并且提前返回的分支返回了一个 `Err(e)` 值。函数的返回类型必须是一个 `Result`，以便与这个 `return` 兼容。

在示例 9-10 中，让我们看看如果在返回类型与我们使用 `?` 的值的类型不兼容的 `main` 函数中使用 `?` 运算符，会得到什么错误。

<Listing number="9-10" file-name="src/main.rs" caption="尝试在返回 `()` 的 `main` 函数中使用 `?` 将无法编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

这段代码打开一个文件，这可能会失败。`?` 运算符跟在 `File::open` 返回的 `Result` 值后面，但这个 `main` 函数的返回类型是 `()`，而不是 `Result`。当我们编译这段代码时，会得到以下错误消息：

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

这个错误指出，我们只能在返回 `Result`、`Option` 或另一个实现了 `FromResidual` 的类型的函数中使用 `?` 运算符。

要修复这个错误，你有两个选择。一种选择是更改函数的返回类型，使其与你使用 `?` 运算符的值兼容，只要没有限制阻止你这样做。另一种选择是使用 `match` 或 `Result<T, E>` 方法之一以适当的任何方式处理 `Result<T, E>`。

错误消息还提到 `?` 也可以与 `Option<T>` 值一起使用。与在 `Result` 上使用 `?` 一样，你只能在返回 `Option` 的函数中对 `Option` 使用 `?`。对 `Option<T>` 调用 `?` 运算符时的行为与对 `Result<T, E>` 调用时的行为类似：如果值是 `None`，则 `None` 会在该点从函数中提前返回。如果值是 `Some`，`Some` 内部的值就是表达式的结果值，函数继续执行。示例 9-11 有一个函数的示例，该函数找到给定文本中第一行的最后一个字符。

<Listing number="9-11" caption="在 `Option<T>` 值上使用 `?` 运算符">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

此函数返回 `Option<char>`，因为可能存在字符，但也可能没有。这段代码接受 `text` 字符串切片参数，并在其上调用 `lines` 方法，该方法返回字符串中各行的一个迭代器（iterator）。因为这个函数想要检查第一行，它调用迭代器上的 `next` 来获取迭代器中的第一个值。如果 `text` 是空字符串，对 `next` 的调用将返回 `None`，在这种情况下我们使用 `?` 停止并从 `last_char_of_first_line` 返回 `None`。如果 `text` 不是空字符串，`next` 将返回一个包含 `text` 中第一行字符串切片的 `Some` 值。

`?` 提取出字符串切片，我们可以在该字符串切片上调用 `chars` 来获取其字符的迭代器。我们对第一行中的最后一个字符感兴趣，所以我们调用 `last` 来返回迭代器中的最后一个元素。这是一个 `Option`，因为第一行可能是空字符串；例如，如果 `text` 以空行开头但其他行有字符，比如 `"\nhi"`。然而，如果第一行有最后一个字符，它将在 `Some` 变体中返回。中间的 `?` 运算符给了我们一种简洁的方式来表达这个逻辑，允许我们在一行中实现该函数。如果我们不能在 `Option` 上使用 `?` 运算符，我们就必须使用更多的方法调用或一个 `match` 表达式来实现这个逻辑。

请注意，你可以在返回 `Result` 的函数中对 `Result` 使用 `?` 运算符，也可以在返回 `Option` 的函数中对 `Option` 使用 `?` 运算符，但不能混合使用。`?` 运算符不会自动将 `Result` 转换为 `Option`，反之亦然；在这些情况下，你可以使用诸如 `Result` 上的 `ok` 方法或 `Option` 上的 `ok_or` 方法来显式地进行转换。

到目前为止，我们使用的所有 `main` 函数都返回 `()`。`main` 函数是特殊的，因为它是可执行程序的入口点和出口点，并且对于程序按预期行为运行，其返回类型有哪些限制。

幸运的是，`main` 也可以返回 `Result<(), E>`。示例 9-12 有来自示例 9-10 的代码，但我们将 `main` 的返回类型更改为 `Result<(), Box<dyn Error>>`，并在末尾添加了一个返回值 `Ok(())`。这段代码现在可以编译了。

<Listing number="9-12" file-name="src/main.rs" caption="将 `main` 改为返回 `Result<(), E>` 允许在 `Result` 值上使用 `?` 运算符">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

`Box<dyn Error>` 类型是一个 trait 对象（trait object），我们将在第 18 章的[“使用 Trait 对象来抽象共享行为”][trait-objects]<!-- ignore -->中讨论。现在，你可以将 `Box<dyn Error>` 理解为"任何类型的错误"。在错误类型为 `Box<dyn Error>` 的 `main` 函数中对 `Result` 值使用 `?` 是允许的，因为它允许任何 `Err` 值被提前返回。尽管这个 `main` 函数体只会返回 `std::io::Error` 类型的错误，但通过指定 `Box<dyn Error>`，即使向 `main` 函数体添加了返回其他错误的更多代码，此签名也仍然正确。

当 `main` 函数返回 `Result<(), E>` 时，如果 `main` 返回 `Ok(())`，可执行文件将以值 `0` 退出；如果 `main` 返回 `Err` 值，则以非零值退出。用 C 编写的可执行文件在退出时返回整数：成功退出的程序返回整数 `0`，出现错误的程序返回某个非 `0` 的整数。Rust 也从可执行文件返回整数以与此约定兼容。

`main` 函数可以返回任何实现了 [`std::process::Termination` trait][termination]<!-- ignore --> 的类型，该 trait 包含一个返回 `ExitCode` 的函数 `report`。请查阅标准库文档以获取有关为你自己的类型实现 `Termination` trait 的更多信息。

既然我们已经讨论了调用 `panic!` 或返回 `Result` 的细节，让我们回到在哪些情况下应该和不应该使用哪种方法的话题。

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[termination]: ../std/process/trait.Termination.html
