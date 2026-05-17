## 将 Crate 发布到 Crates.io

我们使用过 [crates.io](https://crates.io/)<!-- ignore --> 上的包作为项目的依赖，但你也可以通过发布自己的包来与他人共享代码。[crates.io](https://crates.io/)<!-- ignore --> 上的 crate 注册表分发你包的源代码，因此它主要托管开源代码。

Rust 和 Cargo 具有使你的已发布包更容易被他人发现和使用的功能。我们接下来将讨论其中的一些功能，然后解释如何发布一个包。

### 编写有用的文档注释

准确记录你的包将帮助其他用户知道如何以及何时使用它们，因此投入时间编写文档是值得的。在第 3 章中，我们讨论了如何使用两个斜杠 `//` 来注释 Rust 代码。Rust 还有一种特殊的文档注释，恰当地称为*文档注释（documentation comment）*，它将生成 HTML 文档。HTML 显示文档注释的内容，用于公共 API 项，适用于有兴趣了解如何*使用*你的 crate 的程序员，而不是你的 crate 是如何*实现的*。

文档注释使用三个斜杠 `///` 而不是两个，并支持 Markdown 表示法来格式化文本。将文档注释放在它们所记录的项之前。示例 14-1 显示了一个名为 `my_crate` 的 crate 中 `add_one` 函数的文档注释。

<Listing number="14-1" file-name="src/lib.rs" caption="函数的文档注释">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

在这里，我们描述了 `add_one` 函数的功能，以标题 `Examples` 开始一个部分，然后提供了演示如何使用 `add_one` 函数的代码。我们可以通过运行 `cargo doc` 从此文档注释生成 HTML 文档。此命令运行与 Rust 一起分发的 `rustdoc` 工具，并将生成的 HTML 文档放在 _target/doc_ 目录中。

为方便起见，运行 `cargo doc --open` 将为你当前 crate 的文档（以及你 crate 的所有依赖的文档）构建 HTML，并在 Web 浏览器中打开结果。导航到 `add_one` 函数，你将看到文档注释中的文本是如何呈现的，如图 14-1 所示。

<img alt="`my_crate` 的 `add_one` 函数的渲染 HTML 文档" src="img/trpl14-01.png" class="center" />

<span class="caption">图 14-1：`add_one` 函数的 HTML 文档</span>

#### 常用部分

我们在示例 14-1 中使用了 `# Examples` Markdown 标题在 HTML 中创建了一个标题为"Examples"的部分。以下是 crate 作者在他们的文档中经常使用的一些其他部分：

- **Panics**：被记录的函数可能 panic 的场景。不希望程序 panic 的函数调用者应确保他们不会在这些情况下调用该函数。
- **Errors**：如果函数返回一个 `Result`，描述可能发生的错误类型以及什么条件可能导致这些错误被返回，对调用者很有帮助，以便他们可以编写代码以不同方式处理不同类型的错误。
- **Safety**：如果调用该函数是 `unsafe` 的（我们将在第 20 章讨论不安全性），应该有一个部分解释为什么该函数是不安全的，并涵盖该函数期望调用者遵守的不变式。

大多数文档注释不需要所有这些部分，但这是一个很好的清单，可以提醒你用户有兴趣了解的代码方面。

#### 文档注释作为测试

在文档注释中添加示例代码块可以帮助演示如何使用你的库，并且还有一个额外的好处：运行 `cargo test` 会将文档中的代码示例作为测试运行！没有什么比带有示例的文档更好的了。但没有什么比因为文档编写后代码已更改而导致示例无法运行更糟糕的了。如果我们使用示例 14-1 中 `add_one` 函数的文档运行 `cargo test`，我们将在测试结果中看到一个部分，如下所示：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

现在，如果我们更改函数或示例，使示例中的 `assert_eq!` panic，然后再次运行 `cargo test`，我们将看到文档测试检测到示例和代码不同步！

<!-- Old headings. Do not remove or links may break. -->

<a id="commenting-contained-items"></a>

#### 包含项的注释

`//!` 风格的文档注释为*包含*注释的项（而不是注释*后面*的项）添加文档。我们通常将这些文档注释放在 crate 根文件（按惯例是 _src/lib.rs_）内部或模块内部，以记录整个 crate 或模块。

例如，为了添加描述包含 `add_one` 函数的 `my_crate` crate 的目的的文档，我们在 _src/lib.rs_ 文件的开头添加以 `//!` 开头的文档注释，如示例 14-2 所示。

<Listing number="14-2" file-name="src/lib.rs" caption="整个 `my_crate` crate 的文档">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

注意，在以 `//!` 开头的最后一行之后没有任何代码。因为我们以 `//!` 而不是 `///` 开头注释，所以我们正在记录包含此注释的项，而不是此注释后面的项。在这种情况下，该项是 _src/lib.rs_ 文件，它是 crate 根。这些注释描述整个 crate。

当我们运行 `cargo doc --open` 时，这些注释将显示在 `my_crate` 文档的首页上，位于 crate 中公共项列表上方，如图 14-2 所示。

项内的文档注释对于描述 crate 和模块尤其有用。使用它们来解释容器的整体目的，以帮助你的用户理解 crate 的组织结构。

<img alt="带有整个 crate 注释的渲染 HTML 文档" src="img/trpl14-02.png" class="center" />

<span class="caption">图 14-2：`my_crate` 的渲染文档，包括描述整个 crate 的注释</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### 导出方便的公共 API

发布 crate 时，公共 API 的结构是一个主要的考虑因素。使用你的 crate 的人不像你那样熟悉结构，如果你的 crate 有大型模块层次结构，他们可能难以找到他们想要使用的部分。

在第 7 章中，我们介绍了如何使用 `pub` 关键字使项公开，以及如何使用 `use` 关键字将项引入作用域。但是，在开发 crate 时对你来说有意义的结构可能对你的用户并不方便。你可能希望将结构体组织成包含多个级别的层次结构，但是想要使用你在层次结构深处定义的类型的人可能很难发现该类型的存在。他们也可能对必须输入 `use my_crate::some_module::another_module::UsefulType;` 而不是 `use my_crate::UsefulType;` 感到恼火。

好消息是，如果该结构*不*方便其他人从另一个库使用，你不必重新安排你的内部组织：相反，你可以使用 `pub use` 重新导出项，以创建与你的私有结构不同的公共结构。*重新导出（Re-exporting）*将一个位置的公共项在另一个位置也变为公共项，就好像它是在另一个位置定义的一样。

例如，假设我们制作了一个名为 `art` 的库来建模艺术概念。在这个库中，有两个模块：一个 `kinds` 模块包含两个枚举 `PrimaryColor` 和 `SecondaryColor`，以及一个 `utils` 模块包含一个名为 `mix` 的函数，如示例 14-3 所示。

<Listing number="14-3" file-name="src/lib.rs" caption="一个 `art` 库，其项组织在 `kinds` 和 `utils` 模块中">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

图 14-3 显示了 `cargo doc` 为此 crate 生成的文档首页的外观。

<img alt="`art` crate 的渲染文档，列出了 `kinds` 和 `utils` 模块" src="img/trpl14-03.png" class="center" />

<span class="caption">图 14-3：`art` 的文档首页，列出了 `kinds` 和 `utils` 模块</span>

注意，`PrimaryColor` 和 `SecondaryColor` 类型没有列在首页上，`mix` 函数也没有。我们必须点击 `kinds` 和 `utils` 才能看到它们。

另一个依赖此库的 crate 将需要使用 `use` 语句将 `art` 中的项引入作用域，指定当前定义的模块结构。示例 14-4 显示了一个使用 `art` crate 中的 `PrimaryColor` 和 `mix` 项的 crate 示例。

<Listing number="14-4" file-name="src/main.rs" caption="一个使用 `art` crate 的项（使用其内部结构导出）的 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

示例 14-4 中使用 `art` crate 的代码的作者必须弄清楚 `PrimaryColor` 在 `kinds` 模块中，`mix` 在 `utils` 模块中。`art` crate 的模块结构对从事 `art` crate 的开发人员比对使用它的人更相关。内部结构不包含任何对试图理解如何使用 `art` crate 的人有用的信息，反而会引起混淆，因为使用它的开发人员必须弄清楚在哪里查找，并且必须在 `use` 语句中指定模块名称。

要从公共 API 中移除内部组织，我们可以修改示例 14-3 中的 `art` crate 代码，添加 `pub use` 语句在顶层重新导出项，如示例 14-5 所示。

<Listing number="14-5" file-name="src/lib.rs" caption="添加 `pub use` 语句以重新导出项">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

`cargo doc` 为此 crate 生成的 API 文档现在将在首页列出和链接重新导出的项，如图 14-4 所示，使 `PrimaryColor` 和 `SecondaryColor` 类型以及 `mix` 函数更容易找到。

<img alt="`art` crate 的渲染文档，在首页上显示了重新导出的项" src="img/trpl14-04.png" class="center" />

<span class="caption">图 14-4：`art` 的文档首页，列出了重新导出的项</span>

`art` crate 的用户仍然可以看到并使用示例 14-3 中的内部结构，如示例 14-4 所示，或者他们可以使用示例 14-5 中更方便的结构，如示例 14-6 所示。

<Listing number="14-6" file-name="src/main.rs" caption="一个使用 `art` crate 中重新导出项的程序">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

在存在许多嵌套模块的情况下，使用 `pub use` 在顶层重新导出类型可以对使用 crate 的人的体验产生重大影响。`pub use` 的另一个常见用途是在当前 crate 中重新导出依赖项的定义，以使该 crate 的定义成为你的 crate 公共 API 的一部分。

创建有用的公共 API 结构与其说是科学，不如说是一门艺术，你可以迭代以找到最适合你的用户的 API。选择 `pub use` 使你在如何内部组织 crate 方面具有灵活性，并将该内部结构与向用户呈现的内容解耦。看看你已经安装的一些 crate 的代码，看看它们的内部结构是否与其公共 API 不同。

### 设置 Crates.io 账户

在你可以发布任何 crate 之前，你需要在 [crates.io](https://crates.io/)<!-- ignore --> 上创建一个帐户并获取一个 API token。为此，请访问 [crates.io](https://crates.io/)<!-- ignore --> 的主页并通过 GitHub 帐户登录。（GitHub 帐户目前是必需的，但该网站将来可能支持创建帐户的其他方式。）登录后，请访问 [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> 的帐户设置并检索你的 API key。然后，运行 `cargo login` 命令并在提示时粘贴你的 API key，如下所示：

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

此命令将告知 Cargo 你的 API token，并将其本地存储在 _~/.cargo/credentials.toml_ 中。注意，此 token 是保密的：不要与其他人共享。如果由于任何原因确实与他人共享，你应在 [crates.io](https://crates.io/)<!-- ignore --> 上撤销它并生成一个新 token。

### 向新 Crate 添加元数据

假设你有一个想要发布的 crate。在发布之前，你需要在 crate 的 _Cargo.toml_ 文件的 `[package]` 部分添加一些元数据。

你的 crate 需要一个唯一的名称。在本地开发 crate 时，你可以随心所欲地命名 crate。然而，[crates.io](https://crates.io/)<!-- ignore --> 上的 crate 名称是先到先得的。一旦一个 crate 名称被占用，其他任何人都不能使用该名称发布 crate。在尝试发布 crate 之前，搜索你想要使用的名称。如果该名称已被使用，你将需要找到另一个名称，并编辑 _Cargo.toml_ 文件中 `[package]` 部分下的 `name` 字段以使用新名称进行发布，如下所示：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

即使你选择了唯一的名称，此时运行 `cargo publish` 发布 crate 时，你会收到一个警告，然后是一个错误：

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

这会导致错误，因为你缺少一些关键信息：description 和 license 是必需的，以便人们知道你的 crate 是做什么的以及他们可以在什么条件下使用它。在 _Cargo.toml_ 中，添加一个描述，只需一两句话，因为它将与你的 crate 一起出现在搜索结果中。对于 `license` 字段，你需要提供一个*许可证标识符值（license identifier value）*。[Linux 基金会的软件包数据交换（SPDX）][spdx] 列出了你可以使用该值的标识符。例如，要指定你已使用 MIT 许可证许可你的 crate，请添加 `MIT` 标识符：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

如果你想使用 SPDX 中没有的许可证，你需要将该许可证的文本放在一个文件中，将该文件包含在你的项目中，然后使用 `license-file` 指定该文件的名称，而不是使用 `license` 键。

关于哪种许可证适合你的项目的指南超出了本书的范围。Rust 社区中的许多人以与 Rust 相同的方式许可他们的项目，使用 `MIT OR Apache-2.0` 的双重许可。这种做法表明你还可以通过 `OR` 分隔多个许可证标识符来为你的项目拥有多个许可证。

有了唯一的名称、版本、描述和许可证，一个准备发布的项目 _Cargo.toml_ 文件可能如下所示：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo 的文档](https://doc.rust-lang.org/cargo/)描述了你可以指定的其他元数据，以确保其他人可以更容易地发现和使用你的 crate。

### 发布到 Crates.io

既然你已经创建了帐户、保存了 API token、为你的 crate 选择了名称并指定了所需的元数据，你就可以发布了！发布 crate 会将特定版本上传到 [crates.io](https://crates.io/)<!-- ignore --> 供其他人使用。

请小心，因为发布是*永久性的*。该版本永远不能被覆盖，除非在特定情况下，代码不能被删除。Crates.io 的一个主要目标是作为代码的永久存档，以便依赖于 [crates.io](https://crates.io/)<!-- ignore --> 的 crate 的所有项目的构建都能继续工作。允许删除版本将使实现该目标变得不可能。但是，你可以发布的 crate 版本数量没有限制。

再次运行 `cargo publish` 命令。现在应该成功了：

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
     Packaged 6 files, 1.2KiB (895.0B compressed)
    Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
    Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
    Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
     Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

恭喜！现在你已经与 Rust 社区共享了你的代码，任何人都可以轻松地将你的 crate 添加为他们项目的依赖。

### 发布现有 Crate 的新版本

当你对 crate 进行了更改并准备发布新版本时，请更改 _Cargo.toml_ 文件中指定的 `version` 值并重新发布。使用[语义化版本控制规则][semver]根据你所做的更改类型确定适当的下一版本号。然后，运行 `cargo publish` 上传新版本。

<!-- Old headings. Do not remove or links may break. -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>
<a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### 从 Crates.io 弃用版本

虽然你不能删除 crate 的旧版本，但你可以阻止任何未来的项目将其添加为新依赖。当 crate 版本因某种原因损坏时，这很有用。在这种情况下，Cargo 支持 yanking（废弃）一个 crate 版本。

*Yanking* 一个版本可防止新项目依赖于该版本，同时允许所有依赖于它的现有项目继续。本质上，yank 意味着所有具有 _Cargo.lock_ 的项目都不会中断，而任何将来生成的 _Cargo.lock_ 文件都不会使用被 yank 的版本。

要 yank crate 的一个版本，在你之前发布的 crate 的目录中，运行 `cargo yank` 并指定你想要 yank 的版本。例如，如果我们发布了一个名为 `guessing_game` 的 1.0.1 版本的 crate，并且我们想要 yank 它，那么我们在 `guessing_game` 的项目目录中运行以下命令：

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

通过向命令添加 `--undo`，你也可以撤销 yank 并允许项目再次开始依赖该版本：

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Yank *不会*删除任何代码。例如，它无法删除意外上传的机密信息。如果发生这种情况，你必须立即重置这些机密。

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/
