<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## 使用测试驱动开发（Test-Driven Development，TDD）添加功能

既然我们已经将搜索逻辑从 `main` 函数分离到 _src/lib.rs_ 中，编写针对代码核心功能的测试就容易多了。我们可以直接使用各种参数调用函数并检查返回值，而无需从命令行调用二进制文件。

在本节中，我们将使用测试驱动开发（TDD）过程将搜索逻辑添加到 `minigrep` 程序，遵循以下步骤：

1. 编写一个会失败的测试，并运行它以确保它按照你期望的原因失败。
2. 编写或修改刚好足够的代码以使新测试通过。
3. 重构你刚刚添加或更改的代码，并确保测试继续通过。
4. 从第 1 步开始重复！

虽然这只是编写软件的众多方法之一，但 TDD 有助于驱动代码设计。在编写使测试通过的代码之前编写测试，有助于在整个过程中保持较高的测试覆盖率。

我们将测试驱动实现将在文件内容中搜索查询字符串并生成匹配查询的行列表的功能。我们将在一个名为 `search` 的函数中添加此功能。

### 编写一个会失败的测试

在 _src/lib.rs_ 中，我们将添加一个带有测试函数的 `tests` 模块，就像我们在[第 11 章][ch11-anatomy]<!-- ignore -->中所做的那样。测试函数指定了我们希望 `search` 函数具有的行为：它将接受一个查询和要搜索的文本，并且只返回文本中包含查询的那些行。示例 12-15 显示了此测试。

<Listing number="12-15" file-name="src/lib.rs" caption="为我们希望拥有的 `search` 功能创建一个会失败的测试">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

此测试搜索字符串 `"duct"`。我们要搜索的文本有三行，其中只有一行包含 `"duct"`（注意开头的双引号后的反斜杠告诉 Rust 不要在此字符串字面量内容开头放置换行符）。我们断言从 `search` 函数返回的值只包含我们期望的行。

如果我们运行此测试，它目前会失败，因为 `unimplemented!` 宏会 panic 并显示消息"not implemented"。根据 TDD 原则，我们将采取一小步，通过将 `search` 函数定义为始终返回空向量来添加刚好足够的代码使测试在调用该函数时不会 panic，如示例 12-16 所示。然后，测试应该能编译但会失败，因为空向量与包含行 `"safe, fast, productive."` 的向量不匹配。

<Listing number="12-16" file-name="src/lib.rs" caption="定义刚好足够的 `search` 函数，以便调用它时不会 panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

现在让我们讨论为什么我们需要在 `search` 的签名中定义一个显式的生命周期 `'a`，并将该生命周期与 `contents` 参数和返回值一起使用。回想一下[第 10 章][ch10-lifetimes]<!-- ignore -->中，生命周期参数指定哪个参数的生命周期与返回值的生命周期相关联。在这种情况下，我们指示返回的向量应包含引用 `contents` 参数切片的字符串切片（而不是引用 `query` 参数）。

换句话说，我们告诉 Rust，`search` 函数返回的数据将与通过 `contents` 参数传递到 `search` 函数的数据存活时间一样长。这很重要！切片*所引用*的数据必须有效，引用才能有效；如果编译器假设我们是在制作 `query` 的字符串切片而不是 `contents`，它将错误地进行安全检查。

如果我们忘记生命周期注解并尝试编译此函数，将得到以下错误：

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust 无法知道输出需要两个参数中的哪一个，因此我们需要明确告诉它。注意，帮助文本建议为所有参数和输出类型指定相同的生命周期参数，这是不正确的！因为 `contents` 是包含我们所有文本的参数，并且我们想要返回该文本中匹配的部分，我们知道 `contents` 是唯一应使用生命周期语法连接到返回值的参数。

其他编程语言不要求你在签名中连接参数和返回值，但这种做法会随着时间的推移变得更容易。你可能想将此示例与第 10 章中[“使用生命周期验证引用”][validating-references-with-lifetimes]<!-- ignore -->部分的示例进行比较。

### 编写使测试通过的代码

目前，我们的测试失败了，因为我们总是返回一个空向量。为了修复此问题并实现 `search`，我们的程序需要遵循以下步骤：

1. 遍历内容的每一行。
2. 检查该行是否包含我们的查询字符串。
3. 如果包含，则将其添加到我们要返回的值列表中。
4. 如果不包含，则不执行任何操作。
5. 返回匹配的结果列表。

让我们逐步进行，从遍历行开始。

#### 使用 `lines` 方法遍历行

Rust 有一个有用的方法来逐行遍历字符串，恰当地命名为 `lines`，它的工作方式如示例 12-17 所示。注意，这还不能编译。

<Listing number="12-17" file-name="src/lib.rs" caption="遍历 `contents` 中的每一行">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` 方法返回一个迭代器（iterator）。我们将在[第 13 章][ch13-iterators]<!-- ignore -->深入讨论迭代器。但回想一下你在[示例 3-5][ch3-iter]<!-- ignore -->中看到过使用迭代器的方式，我们在那里使用了带有迭代器的 `for` 循环来对集合中的每个项运行一些代码。

#### 搜索每行中的查询

接下来，我们将检查当前行是否包含我们的查询字符串。幸运的是，字符串有一个名为 `contains` 的有用方法可以为我们做到这点！在对 `search` 函数的调用中添加 `contains` 方法，如示例 12-18 所示。注意，这仍然不能编译。

<Listing number="12-18" file-name="src/lib.rs" caption="添加功能以查看该行是否包含 `query` 中的字符串">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

目前，我们正在逐步构建功能。为了让代码编译，我们需要像在函数签名中表示的那样从函数体返回一个值。

#### 存储匹配的行

为了完成这个函数，我们需要一种方法来存储我们要返回的匹配行。为此，我们可以在 `for` 循环之前创建一个可变向量，并调用 `push` 方法将 `line` 存储在向量中。在 `for` 循环之后，我们返回该向量，如示例 12-19 所示。

<Listing number="12-19" file-name="src/lib.rs" caption="存储匹配的行以便我们可以返回它们">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

现在，`search` 函数应该只返回包含 `query` 的行，并且我们的测试应该通过。让我们运行测试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

我们的测试通过了，所以我们知道它是有效的！

此时，我们可以考虑重构搜索函数实现的机会，同时保持测试通过以维持相同的功能。搜索函数中的代码还不错，但它没有利用迭代器的一些有用特性。我们将在[第 13 章][ch13-iterators]<!-- ignore -->回到这个例子，届时我们将详细探讨迭代器，并看看如何改进它。

现在整个程序应该可以工作了！让我们试试看，首先用一个应该从艾米莉·狄金森的诗中准确地返回一行的词：_frog_。

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

酷！现在让我们尝试一个将匹配多行的词，比如 _body_：

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

最后，让我们确保当我们搜索一个在诗中任何地方都不存在的词时，比如 _monomorphization_，不会得到任何行：

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

太棒了！我们已经构建了一个经典工具的自己版本，并学到了很多关于如何构建应用程序的知识。我们还学到了一些关于文件输入输出、生命周期、测试和命令行解析的知识。

为了完成这个项目，我们将简要地演示如何使用环境变量以及如何打印到标准错误，这两者在编写命令行程序时都很有用。

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
