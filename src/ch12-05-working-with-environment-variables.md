## 使用环境变量（Working with Environment Variables）

我们将通过添加一个额外功能来改进 `minigrep` 二进制文件：一个不区分大小写的搜索选项，用户可以通过环境变量打开它。我们可以将此功能作为一个命令行选项，并要求用户每次想要使用时都输入它，但通过将其设置为环境变量，我们允许用户设置一次环境变量，然后在该终端会话中的所有搜索都不区分大小写。

<!-- Old headings. Do not remove or links may break. -->

<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### 为不区分大小写的搜索编写一个会失败的测试

我们首先在 `minigrep` 库中添加一个新的 `search_case_insensitive` 函数，当环境变量有值时将调用该函数。我们将继续遵循 TDD 过程，因此第一步同样是编写一个会失败的测试。我们将为新的 `search_case_insensitive` 函数添加一个新测试，并将旧测试从 `one_result` 重命名为 `case_sensitive`，以澄清两个测试之间的区别，如示例 12-20 所示。

<Listing number="12-20" file-name="src/lib.rs" caption="为我们即将添加的不区分大小写函数添加一个新的会失败的测试">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

注意，我们也编辑了旧测试的 `contents`。我们添加了一行包含 `"Duct tape."` 的文本，其中使用了大写 _D_，在以区分大小写方式搜索时不应与查询 `"duct"` 匹配。以这种方式更改旧测试有助于确保我们不会意外破坏已经实现的区分大小写搜索功能。此测试现在应该通过，并且在我们进行不区分大小写搜索的工作时应继续通过。

不*区分大小写的*搜索的新测试使用 `"rUsT"` 作为其查询。在我们即将添加的 `search_case_insensitive` 函数中，查询 `"rUsT"` 应匹配包含 `"Rust:"`（大写 _R_）的行，并匹配 `"Trust me."` 这一行，尽管它们的大小写与查询不同。这是我们会失败的测试，并且由于我们尚未定义 `search_case_insensitive` 函数，它将无法编译。请随意添加一个始终返回空向量的骨架实现，类似于我们在示例 12-16 中对 `search` 函数的做法，以观察测试编译和失败。

### 实现 `search_case_insensitive` 函数

`search_case_insensitive` 函数（如示例 12-21 所示）将与 `search` 函数几乎相同。唯一的区别是，我们将把 `query` 和每个 `line` 转换为小写，这样无论输入参数的大小写如何，当检查该行是否包含查询时，它们将具有相同的大小写。

<Listing number="12-21" file-name="src/lib.rs" caption="定义 `search_case_insensitive` 函数，在比较之前将查询和行都转换为小写">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

首先，我们将 `query` 字符串转换为小写，并将其存储在一个同名的新变量中，遮盖了原始的 `query`。对查询调用 `to_lowercase` 是必要的，这样无论用户的查询是 `"rust"`、`"RUST"`、`"Rust"` 还是 `"rUsT"`，我们都会将查询视为 `"rust"` 并且不区分大小写。虽然 `to_lowercase` 会处理基本的 Unicode，但它不会是 100% 准确的。如果我们正在编写一个真正的应用程序，我们想在这里做更多的工作，但本节是关于环境变量的，而不是 Unicode，所以我们在此就此打住。

注意，`query` 现在是一个 `String` 而不是一个字符串切片，因为调用 `to_lowercase` 创建了新数据，而不是引用现有数据。例如，假设查询是 `"rUsT"`：该字符串切片不包含小写的 `u` 或 `t` 供我们使用，因此我们必须分配一个新的包含 `"rust"` 的 `String`。现在我们向 `contains` 方法传递 `query` 作为参数时，需要添加一个 &，因为 `contains` 的签名被定义为接受一个字符串切片。

接下来，我们在每个 `line` 上添加对 `to_lowercase` 的调用，将所有字符转换为小写。现在我们已经将 `line` 和 `query` 都转换为小写，无论查询的大小写如何，我们都会找到匹配项。

让我们看看这个实现是否通过了测试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

太好了！它们通过了。现在让我们从 `run` 函数调用新的 `search_case_insensitive` 函数。首先，我们向 `Config` 结构体添加一个配置选项，以便在区分大小写和不区分大小写搜索之间切换。添加此字段会导致编译器错误，因为我们还没有在任何地方初始化此字段：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

我们添加了 `ignore_case` 字段，它持有一个布尔值。接下来，`run` 函数需要检查 `ignore_case` 字段的值，并用它来决定是调用 `search` 函数还是 `search_case_insensitive` 函数，如示例 12-22 所示。这仍然不能编译。

<Listing number="12-22" file-name="src/main.rs" caption="根据 `config.ignore_case` 的值调用 `search` 或 `search_case_insensitive`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

最后，我们需要检查环境变量。用于处理环境变量的函数位于标准库的 `env` 模块中，该模块已在 _src/main.rs_ 顶部的作用域中。我们将使用 `env` 模块的 `var` 函数来检查是否已为名为 `IGNORE_CASE` 的环境变量设置了任何值，如示例 12-23 所示。

<Listing number="12-23" file-name="src/main.rs" caption="检查名为 `IGNORE_CASE` 的环境变量的任何值">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

在这里，我们创建一个新变量 `ignore_case`。为了设置其值，我们调用 `env::var` 函数，并传递环境变量的名称 `IGNORE_CASE`。`env::var` 函数返回一个 `Result`：如果环境变量设置为任何值，它将成功返回包含该环境变量值的 `Ok` 变体；如果环境变量未设置，它将返回 `Err` 变体。

我们使用 `Result` 上的 `is_ok` 方法检查环境变量是否已设置，这意味着程序应进行不区分大小写的搜索。如果 `IGNORE_CASE` 环境变量未设置任何值，`is_ok` 将返回 `false`，程序将执行区分大小写的搜索。我们不关心环境变量的*值*，只关心它是已设置还是未设置，因此我们检查 `is_ok`，而不是使用 `unwrap`、`expect` 或我们在 `Result` 上见过的其他方法。

我们将 `ignore_case` 变量中的值传递给 `Config` 实例，以便 `run` 函数可以读取该值并决定是调用 `search_case_insensitive` 还是 `search`，正如我们在示例 12-22 中实现的那样。

让我们试试看！首先，我们将在没有设置环境变量且查询 `to` 的情况下运行程序，它应该匹配任何包含全小写单词 _to_ 的行：

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

看起来仍然有效！现在让我们在设置 `IGNORE_CASE` 为 `1` 的情况下运行程序，但使用相同的查询 `to`：

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

如果你使用的是 PowerShell，则需要设置环境变量并作为单独的命令运行程序：

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

这将使 `IGNORE_CASE` 在你的 shell 会话的剩余时间内持续存在。可以使用 `Remove-Item` cmdlet 取消设置：

```console
PS> Remove-Item Env:IGNORE_CASE
```

我们应得到包含 _to_（可能有大写字母）的行：

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

太棒了，我们还得到了包含 _To_ 的行！我们的 `minigrep` 程序现在可以进行由环境变量控制的不区分大小写的搜索。现在你知道了如何管理使用命令行参数或环境变量设置的选项。

有些程序允许同一配置使用参数*和*环境变量。在这些情况下，程序决定其中一个具有优先权。作为你自己的另一个练习，尝试通过命令行参数或环境变量来控制大小写敏感性。如果程序在运行时一个设置为区分大小写、一个设置为忽略大小写，请决定命令行参数或环境变量哪个应优先。

`std::env` 模块包含更多用于处理环境变量的有用功能：请查看其文档以了解哪些功能可用。
