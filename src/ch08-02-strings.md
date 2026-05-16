## 使用字符串（String）存储 UTF-8 编码的文本

我们在第 4 章讨论过字符串，但现在我们将更深入地研究它们。新 Rustaceans（Rust 初学者）经常在字符串问题上卡住，这是由于三个原因共同造成的：Rust 倾向于暴露可能的错误、字符串是一种比许多程序员所认为的更复杂的数据结构，以及 UTF-8。这些因素结合在一起，使得来自其他编程语言的人会感到困难。

我们在集合的语境下讨论字符串，因为字符串是作为字节集合实现的，再加上一些方法，用于在这些字节被解释为文本时提供有用的功能。在本节中，我们将讨论 `String` 上所有集合类型都具有的操作，例如创建、更新和读取。我们还将讨论 `String` 与其他集合的不同之处，即：由于人和计算机对 `String` 数据的解释方式不同，对 `String` 进行索引变得复杂。

<!-- Old headings. Do not remove or links may break. -->

<a id="what-is-a-string"></a>

### 定义字符串

我们首先定义一下术语*字符串（string）*的含义。Rust 核心语言中只有一种字符串类型，那就是字符串切片（string slice）`str`，通常以其借用形式 `&str` 出现。在第 4 章中，我们讨论过字符串切片，它是对存储在其他位置的某些 UTF-8 编码字符串数据的引用。例如，字符串字面量（string literal）存储在程序的二进制文件中，因此是字符串切片。

`String` 类型由 Rust 的标准库提供，而非核心语言内置，它是一种可增长的、可变的、拥有的、UTF-8 编码的字符串类型。当 Rustaceans 在 Rust 中提到"字符串"时，他们可能指的是 `String` 或字符串切片 `&str` 类型，而不仅仅是其中一种。尽管本节主要讨论 `String`，但两种类型在 Rust 标准库中都大量使用，并且 `String` 和字符串切片都是 UTF-8 编码的。

### 创建新字符串

`Vec<T>` 可用的许多操作同样适用于 `String`，因为 `String` 实际上是围绕一个字节向量（vector of bytes）实现的包装器，并带有一些额外的保证、限制和功能。一个对 `Vec<T>` 和 `String` 工作方式相同的函数的例子是用于创建实例的 `new` 函数，如示例 8-11 所示。

<Listing number="8-11" caption="创建一个新的空 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

这一行创建了一个名为 `s` 的新空字符串，然后我们可以向其中加载数据。通常，我们会有一些初始数据，希望用这些数据来初始化字符串。为此，我们使用 `to_string` 方法，该方法适用于任何实现了 `Display` trait 的类型，就像字符串字面量那样。示例 8-12 展示了两个例子。

<Listing number="8-12" caption="使用 `to_string` 方法从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

这段代码创建了一个包含 `initial contents` 的字符串。

我们也可以使用 `String::from` 函数从字符串字面量创建 `String`。示例 8-13 中的代码与使用 `to_string` 的示例 8-12 中的代码是等价的。

<Listing number="8-13" caption="使用 `String::from` 函数从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

因为字符串用途广泛，所以我们可以使用许多不同的通用 API 来处理字符串，这为我们提供了大量选择。其中一些可能看起来是多余的，但它们都有各自的用武之地！在这种情况下，`String::from` 和 `to_string` 做的是同一件事，所以选择哪一个取决于风格和可读性。

请记住，字符串是 UTF-8 编码的，因此我们可以在其中包含任何正确编码的数据，如示例 8-14 所示。

<Listing number="8-14" caption="在字符串中存储不同语言的问候语">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

所有这些都是有效的 `String` 值。

### 更新字符串

`String` 可以增大其大小，其内容也可以改变，就像 `Vec<T>` 一样，如果你向其中推入更多数据的话。此外，你可以方便地使用 `+` 运算符或 `format!` 宏来拼接 `String` 值。

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### 使用 `push_str` 或 `push` 追加

我们可以使用 `push_str` 方法来追加一个字符串切片，从而使 `String` 增长，如示例 8-15 所示。

<Listing number="8-15" caption="使用 `push_str` 方法向 `String` 追加字符串切片">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

经过这两行之后，`s` 将包含 `foobar`。`push_str` 方法接受一个字符串切片，因为我们不一定需要获取参数的所有权。例如，在示例 8-16 的代码中，我们希望将 `s2` 的内容追加到 `s1` 之后，仍然能够使用 `s2`。

<Listing number="8-16" caption="在将字符串切片的内容追加到 `String` 后继续使用它">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

如果 `push_str` 方法获取了 `s2` 的所有权，我们就无法在最后一行打印它的值了。然而，这段代码正如我们所期望的那样工作！

`push` 方法接受单个字符作为参数，并将其添加到 `String` 中。示例 8-17 使用 `push` 方法向 `String` 添加字母 _l_。

<Listing number="8-17" caption="使用 `push` 向 `String` 值添加一个字符">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

结果，`s` 将包含 `lol`。

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

#### 使用 `+` 或 `format!` 进行拼接

通常，你会想要合并两个现有的字符串。一种方法是使用 `+` 运算符，如示例 8-18 所示。

<Listing number="8-18" caption="使用 `+` 运算符将两个 `String` 值合并为一个新的 `String` 值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

字符串 `s3` 将包含 `Hello, world!`。`s1` 在加法之后不再有效的原因，以及我们使用 `s2` 的引用的原因，与使用 `+` 运算符时调用的方法的签名有关。`+` 运算符使用了 `add` 方法，其签名大致如下：

```rust,ignore
fn add(self, s: &str) -> String {
```

在标准库中，你会看到 `add` 是通过泛型和关联类型（associated types）定义的。这里我们替换成了具体类型，也就是当我们将 `String` 值传入该方法时实际发生的情况。我们将在第 10 章讨论泛型。这个签名为我们提供了理解 `+` 运算符棘手之处所需的线索。

首先，`s2` 带有 `&`，意味着我们是将第二个字符串的引用添加到第一个字符串。这是因为 `add` 函数中的 `s` 参数：我们只能将字符串切片添加到 `String`；不能将两个 `String` 值相加。但是等等——`&s2` 的类型是 `&String`，而不是 `add` 第二个参数所指定的 `&str`。那么，为什么示例 8-18 能编译通过呢？

我们能够在调用 `add` 时使用 `&s2` 的原因是编译器可以将 `&String` 参数强制转换为 `&str`。当我们调用 `add` 方法时，Rust 使用了解引用强制多态（deref coercion），在这里它将 `&s2` 转换为 `&s2[..]`。我们将在第 15 章更深入地讨论解引用强制多态。因为 `add` 不获取 `s` 参数的所有权，所以 `s2` 在此操作之后仍然是有效的 `String`。

其次，我们在签名中可以看到 `add` 获取了 `self` 的所有权，因为 `self` _没有_ `&`。这意味着示例 8-18 中的 `s1` 将被移入 `add` 调用中，并在那之后不再有效。因此，尽管 `let s3 = s1 + &s2;` 看起来像是会复制两个字符串并创建一个新的字符串，但实际上这个语句获取了 `s1` 的所有权，追加了 `s2` 内容的副本，然后返回结果的所有权。换句话说，它看起来像是在做大量的拷贝，但实际上并非如此；实现比复制更加高效。

如果我们需要拼接多个字符串，`+` 运算符的行为将变得笨拙：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

此时，`s` 将是 `tic-tac-toe`。有了所有这些 `+` 和 `"` 字符，很难看清楚发生了什么。对于更复杂的字符串组合，我们可以改用 `format!` 宏：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

这段代码也将 `s` 设置为 `tic-tac-toe`。`format!` 宏的工作方式类似于 `println!`，但它不是将输出打印到屏幕，而是返回一个包含内容的 `String`。使用 `format!` 的代码版本更易于阅读，并且 `format!` 宏生成的代码使用的是引用，因此该调用不会获取任何参数的所有权。

### 索引字符串

在许多其他编程语言中，通过索引引用字符串中的单个字符是一种有效且常见的操作。然而，如果你尝试在 Rust 中使用索引语法访问 `String` 的部分内容，你将得到一个错误。考虑示例 8-19 中无效的代码。

<Listing number="8-19" caption="尝试对 `String` 使用索引语法">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

这段代码将导致以下错误：

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

错误说明了一切：Rust 字符串不支持索引。但为什么不支持呢？要回答这个问题，我们需要讨论 Rust 如何在内存中存储字符串。

#### 内部表示

`String` 是对 `Vec<u8>` 的包装。让我们看看示例 8-14 中一些正确编码的 UTF-8 示例字符串。首先是这个：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

在这种情况下，`len` 将是 `4`，这意味着存储字符串 `"Hola"` 的向量长度为 4 字节。这些字母每个在用 UTF-8 编码时占用 1 字节。然而，下面这一行可能会让你感到惊讶（注意，这个字符串以大写西里尔字母 _Ze_ 开头，而不是数字 3）：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

如果有人问你该字符串有多长，你可能会说 12。实际上，Rust 的答案是 24：这是用 UTF-8 编码"Здравствуйте"所需的字节数，因为该字符串中的每个 Unicode 标量值（Unicode scalar value）占用 2 字节的存储空间。因此，对字符串字节的索引并不总是与有效的 Unicode 标量值对应。为了说明这一点，请考虑以下无效的 Rust 代码：

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

你已经知道 `answer` 不会是 `З`，即第一个字母。在用 UTF-8 编码时，`З` 的第一个字节是 `208`，第二个字节是 `151`，所以看起来 `answer` 实际上应该是 `208`，但 `208` 本身并不是一个有效的字符。返回 `208` 很可能不是用户想要的结果，当他们要求该字符串的第一个字母时；然而，这是 Rust 在字节索引 0 处所拥有的唯一数据。用户通常不希望返回字节值，即使字符串只包含拉丁字母也是如此：如果 `&"hi"[0]` 是返回字节值的有效代码，它将返回 `104`，而不是 `h`。

因此，答案是：为了避免返回意外值并可能导致无法立即发现的 bug，Rust 根本不会编译这段代码，并在开发过程的早期就防止误解。

<!-- Old headings. Do not remove or links may break. -->

<a id="bytes-and-scalar-values-and-grapheme-clusters-oh-my"></a>

#### 字节、标量值和字素簇（Grapheme Cluster）

关于 UTF-8 的另一点是，从 Rust 的角度来看，实际上有三种相关的方式来看待字符串：作为字节（bytes）、作为标量值（scalar values）和作为字素簇（grapheme clusters，最接近我们所说的*字母*的概念）。

如果我们看看用天城文书写的印地语单词"नमस्ते"，它存储为如下所示的 `u8` 值向量：

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

这是 18 个字节，也是计算机最终存储这些数据的方式。如果我们将其视为 Unicode 标量值——也就是 Rust 的 `char` 类型——这些字节看起来像这样：

```text
['न', 'म', 'स', '्', 'त', 'े']
```

这里有六个 `char` 值，但第四个和第六个不是字母：它们是变音符号（diacritics），单独存在没有意义。最后，如果我们将其视为字素簇，就会得到人们所说的组成这个印地语单词的四个字母：

```text
["न", "म", "स्", "ते"]
```

Rust 提供了不同的方式来解释计算机存储的原始字符串数据，以便每个程序可以选择它需要的解释方式，无论数据使用的是哪种人类语言。

Rust 不允许我们对 `String` 进行索引以获取字符的最后一个原因是，索引操作应该始终是常数时间（O(1)）。但是，对于 `String` 来说，无法保证这种性能，因为 Rust 必须从头遍历内容直到索引位置，以确定有多少有效字符。

### 切片字符串

对字符串进行索引通常不是一个好主意，因为不清楚字符串索引操作的返回类型应该是什么：一个字节值、一个字符、一个字素簇还是一个字符串切片。因此，如果你确实需要使用索引来创建字符串切片，Rust 要求你更具体一些。

与其使用带有单个数字的 `[]` 进行索引，不如使用带有范围的 `[]` 来创建包含特定字节的字符串切片：

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

在这里，`s` 将是一个 `&str`，包含字符串的前 4 个字节。之前，我们提到每个字符是 2 个字节，这意味着 `s` 将是 `Зд`。

如果我们尝试切割一个字符的部分字节，比如 `&hello[0..1]`，Rust 会在运行时 panic，就像在向量中访问了无效索引一样：

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

在创建带有范围的字符串切片时应谨慎，因为这可能会导致程序崩溃。

<!-- Old headings. Do not remove or links may break. -->

<a id="methods-for-iterating-over-strings"></a>

### 遍历字符串

操作字符串片段的最佳方式是明确你想要的是字符还是字节。对于单个 Unicode 标量值，使用 `chars` 方法。对"Зд"调用 `chars` 会将它们分离并返回两个 `char` 类型的值，你可以遍历结果来访问每个元素：

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

这段代码将打印以下内容：

```text
З
д
```

另外，`bytes` 方法返回每个原始字节，这可能适用于你的特定领域：

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

这段代码将打印组成该字符串的 4 个字节：

```text
208
151
208
180
```

但请务必记住，有效的 Unicode 标量值可能由超过 1 个字节组成。

从字符串中获取字素簇（如天城文脚本那样）是复杂的，因此标准库不提供此功能。如果这是你需要的功能，可以在 [crates.io](https://crates.io/)<!-- ignore --> 上找到相关的 crate。

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### 处理字符串的复杂性

总而言之，字符串是复杂的。不同的编程语言对于如何向程序员呈现这种复杂性做出了不同的选择。Rust 选择了让所有 Rust 程序的默认行为是正确处理 `String` 数据，这意味着程序员需要提前更多地思考如何处理 UTF-8 数据。这种权衡暴露了比其他编程语言更多的字符串复杂性，但它防止了你在开发生命周期的后期处理涉及非 ASCII 字符的错误。

好消息是，标准库提供了大量基于 `String` 和 `&str` 类型构建的功能，以帮助正确处理这些复杂情况。请务必查阅文档，了解有用的方法，例如用于在字符串中搜索的 `contains` 和用于将字符串的一部分替换为另一个字符串的 `replace`。

接下来让我们转向稍微不那么复杂的东西：哈希映射（hash maps）！
