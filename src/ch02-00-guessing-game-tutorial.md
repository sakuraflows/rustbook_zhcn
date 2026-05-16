# 编写猜数字游戏

让我们通过一个动手项目一起深入 Rust！本章通过向你展示如何在实际程序中使用它们，介绍一些常见的 Rust 概念。你将学习 `let`、`match`、方法（methods）、关联函数（associated functions）、外部 crate 以及更多！在接下来的章节中，我们将更详细地探讨这些概念。在本章中，你只需练习基础知识。

我们将实现一个经典的初学者编程问题：猜数字游戏（guessing game）。它的工作原理如下：程序将生成一个 1 到 100 之间的随机整数。然后提示玩家输入一个猜测。输入猜测后，程序会提示猜测是太小还是太大。如果猜对了，游戏会打印祝贺信息并退出。

## 设置新项目

要设置新项目，请进入你在第 1 章中创建的 _projects_ 目录，并使用 Cargo 创建一个新项目，如下所示：

```console
$ cargo new guessing_game
$ cd guessing_game
```

第一个命令 `cargo new` 将项目名称（`guessing_game`）作为第一个参数。第二个命令切换到新项目的目录。

查看生成的 _Cargo.toml_ 文件：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

正如你在第 1 章中看到的，`cargo new` 会为你生成一个 “Hello, world!” 程序。查看 _src/main.rs_ 文件：

<span class="filename">文件名: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

现在让我们编译这个 “Hello, world!” 程序并使用 `cargo run` 命令一步运行它：

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

当你需要快速迭代项目时，`run` 命令非常方便，正如我们在这个游戏中所做的，在进入下一个迭代之前快速测试每个迭代。

重新打开 _src/main.rs_ 文件。你将在此文件中编写所有代码。

## 处理猜测

猜数字游戏程序的第一部分将请求用户输入，处理该输入，并检查输入是否符合预期格式。首先，我们将允许玩家输入一个猜测。将清单 2-1 中的代码输入到 _src/main.rs_ 中。

<Listing number="2-1" file-name="src/main.rs" caption="获取用户猜测并打印它的代码">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

这段代码包含很多信息，让我们逐行来看。为了获取用户输入并将结果作为输出打印，我们需要将 `io` 输入/输出库引入作用域。`io` 库来自标准库（standard library），即 `std`：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

默认情况下，Rust 在标准库中定义了一组项，它会将这些项引入每个程序的作用域。这组项称为 _prelude_（预导入模块），你可以在[标准库文档][prelude]中查看其所有内容。

如果你要使用的类型不在 prelude 中，则必须使用 `use` 语句显式地将该类型引入作用域。使用 `std::io` 库可以为你提供许多有用的功能，包括接受用户输入的能力。

正如你在第 1 章中看到的，`main` 函数是程序的入口点：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 语法声明了一个新函数；圆括号 `()` 表示没有参数；花括号 `{` 开始函数体。

正如你在第 1 章中也学到的，`println!` 是一个宏（macro），用于将字符串打印到屏幕：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

这段代码打印了一条提示信息，说明游戏的名称并请求用户输入。

### 使用变量存储值

接下来，我们将创建一个 _变量_（variable）来存储用户输入，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

现在程序变得有趣了！这一小行代码中包含了很多内容。我们使用 `let` 语句来创建变量。下面是另一个例子：

```rust,ignore
let apples = 5;
```

这一行创建了一个名为 `apples` 的新变量，并将其绑定到值 `5`。在 Rust 中，变量默认是不可变的（immutable），这意味着一旦我们给变量赋值，该值就不会改变。我们将在第 3 章的[\u201c变量与可变性\u201d][variables-and-mutability]<!-- ignore --> 部分详细讨论这个概念。要使变量可变，我们在变量名之前添加 `mut`：

```rust,ignore
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> 注意：`//` 语法开始一个注释，该注释持续到行尾。Rust 会忽略注释中的所有内容。我们将在[第 3 章][comments]<!-- ignore --> 中更详细地讨论注释。

回到猜数字游戏程序，你现在知道 `let mut guess` 将引入一个名为 `guess` 的可变变量。等号（`=`）告诉 Rust 我们现在想将某些东西绑定到该变量。等号右侧是 `guess` 所绑定的值，即调用 `String::new` 的结果，该函数返回一个新的 `String` 实例。[`String`][string]<!-- ignore --> 是标准库提供的一种字符串类型，是一种可增长的、UTF-8 编码的文本。

`::new` 行中的 `::` 语法表示 `new` 是 `String` 类型的关联函数（associated function）。_关联函数_ 是在类型上实现的函数，在本例中是 `String` 类型。这个 `new` 函数创建了一个新的空字符串。你会在许多类型上找到 `new` 函数，因为它是一种常见命名约定，用于创建某种类型的新值。

总的来说，`let mut guess = String::new();` 这一行创建了一个可变变量，该变量当前绑定到一个新的空 `String` 实例。呼！

### 接收用户输入

回想一下，我们在程序的第一行使用 `use std::io;` 引入了标准库的输入/输出功能。现在我们将调用 `io` 模块中的 `stdin` 函数，这将允许我们处理用户输入：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

如果我们没有在程序开头使用 `use std::io;` 导入 `io` 模块，我们仍然可以通过将此函数调用写为 `std::io::stdin` 来使用该函数。`stdin` 函数返回一个 [`std::io::Stdin`][iostdin]<!-- ignore --> 实例，这是一种表示终端标准输入句柄的类型。

接下来，`.read_line(&mut guess)` 这一行调用了标准输入句柄上的 [`read_line`][read_line]<!-- ignore --> 方法来获取用户输入。我们还将 `&mut guess` 作为参数传递给 `read_line`，告诉它将用户输入存储到哪个字符串中。`read_line` 的全部工作是将用户输入到标准输入中的任何内容追加到该字符串中（而不覆盖其内容），因此我们将该字符串作为参数传递。字符串参数需要是可变的，以便方法可以更改字符串的内容。

`&` 表示此参数是一个 _引用_（reference），它让你能够使代码的多个部分访问同一份数据，而无需多次将该数据复制到内存中。引用是一个复杂的功能，而 Rust 的主要优势之一就是使用引用既安全又容易。你不需要了解很多细节就能完成这个程序。现在，你只需要知道，像变量一样，引用默认也是不可变的。因此，你需要写 `&mut guess` 而不是 `&guess` 来使其可变。（第 4 章会更详细地解释引用。）

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### 使用 `Result` 处理潜在的错误

我们还在处理这一行代码。我们现在正在讨论第三行文本，但请注意它仍然是单个逻辑代码行的一部分。下一部分是这个方法：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

我们本可以将这段代码写成：

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

但是，一个长行难以阅读，所以最好将其拆分。当你使用 `.method_name()` 语法调用方法时，通常明智的做法是引入换行和其他空白来帮助分隔长行。现在让我们讨论这一行的作用。

如前所述，`read_line` 将用户输入的任何内容放入我们传递的字符串中，但它也会返回一个 `Result` 值。[`Result`][result]<!-- ignore --> 是一个[_枚举_][enums]<!-- ignore -->（enumeration，通常称为 enum），它是一种可以有多种可能状态的类型。我们将每种可能的状态称为一个 _变体_（variant）。

[第 6 章][enums]<!-- ignore --> 将更详细地介绍枚举。这些 `Result` 类型的目的是编码错误处理信息。

`Result` 的变体是 `Ok` 和 `Err`。`Ok` 变体表示操作成功，并包含成功生成的值。`Err` 变体表示操作失败，并包含有关操作失败方式或原因的信息。

`Result` 类型的值，就像任何类型的值一样，都定义了方法。`Result` 实例有一个你可以调用的 [`expect` 方法][expect]<!-- ignore -->。如果这个 `Result` 实例是 `Err` 值，`expect` 将导致程序崩溃并显示你传递给 `expect` 作为参数的消息。如果 `read_line` 方法返回 `Err`，这很可能是底层操作系统出现错误的结果。如果这个 `Result` 实例是 `Ok` 值，`expect` 将获取 `Ok` 持有的返回值并仅将该值返回给你，以便你可以使用它。在这种情况下，该值是用户输入的字节数。

如果你不调用 `expect`，程序可以编译，但你会收到一个警告：

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust 警告你没有使用 `read_line` 返回的 `Result` 值，表明程序没有处理可能的错误。

消除警告的正确方法是实际编写错误处理代码，但在我们的案例中，我们只希望在出现问题时使程序崩溃，因此我们可以使用 `expect`。你将在[第 9 章][recover]<!-- ignore --> 中学习如何从错误中恢复。

### 使用 `println!` 占位符打印值

除了右花括号之外，到目前为止的代码中还有一行需要讨论：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

这一行打印了现在包含用户输入的字符串。`{}` 花括号是一个占位符：可以将 `{}` 想象成小小的螃蟹钳子，它们将值固定在适当位置。打印变量的值时，变量名可以放在花括号内。打印表达式求值结果时，在格式字符串中放置空花括号，然后格式字符串后跟一个逗号分隔的表达式列表，按照相同的顺序打印到每个空花括号占位符中。在一次 `println!` 调用中打印变量和表达式的结果如下所示：

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

这段代码将打印 `x = 5 and y + 2 = 12`。

### 测试第一部分

让我们测试猜数字游戏的第一部分。使用 `cargo run` 运行它：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

至此，游戏的第一部分完成了：我们从键盘获取输入然后打印出来。

## 生成一个秘密数字

接下来，我们需要生成一个秘密数字，玩家将尝试猜测它。秘密数字每次都应该不同，这样游戏才能多次玩耍仍然有趣。我们将使用 1 到 100 之间的随机数，这样游戏不会太难。Rust 的标准库中尚未包含随机数功能。但是，Rust 团队提供了一个包含此功能的 [`rand` crate][randcrate]。

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### 使用 crate 增加功能

记住，crate（包）是一个 Rust 源代码文件的集合。我们一直在构建的项目是一个二进制 crate（binary crate），它是一个可执行文件。`rand` crate 是一个库 crate（library crate），其中包含旨在用于其他程序的代码，不能独立执行。

Cargo 对外部 crate 的协调才是 Cargo 真正闪耀的地方。在我们可以编写使用 `rand` 的代码之前，需要修改 _Cargo.toml_ 文件以将 `rand` crate 作为依赖包含进来。现在打开该文件，并在 Cargo 为你创建的 `[dependencies]` 部分标题下方添加以下行。请务必像我们这里一样精确指定 `rand` 及其版本号，否则本教程中的代码示例可能无法正常工作：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">文件名: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

在 _Cargo.toml_ 文件中，标题后的所有内容都是该部分的一部分，直到另一个部分开始。在 `[dependencies]` 中，你告诉 Cargo 你的项目依赖哪些外部 crate 以及你需要这些 crate 的哪些版本。在这种情况下，我们使用语义版本说明符 `0.8.5` 指定了 `rand` crate。Cargo 理解[语义化版本（Semantic Versioning）][semver]<!-- ignore -->（有时称为 _SemVer_），这是一种编写版本号的标准。说明符 `0.8.5` 实际上是 `^0.8.5` 的简写，这意味着任何至少为 0.8.5 但低于 0.9.0 的版本。

Cargo 认为这些版本具有与 0.8.5 兼容的公共 API，并且此规范确保你将获得最新的补丁版本，该版本仍然可以与本章中的代码一起编译。任何 0.9.0 或更高版本都不能保证具有与以下示例相同的 API。

现在，在不更改任何代码的情况下，让我们构建项目，如清单 2-2 所示。

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="将 `rand` crate 添加为依赖后运行 `cargo build` 的输出">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

你可能会看到不同的版本号（但由于 SemVer，它们都将与代码兼容！）和不同的行（取决于操作系统），并且行的顺序可能不同。

当我们包含一个外部依赖时，Cargo 会从 _registry_（注册表）获取该依赖所需的所有内容的最新版本，该注册表是来自 [Crates.io][cratesio] 的数据副本。Crates.io 是 Rust 生态系统中的人们发布其开源 Rust 项目供他人使用的地方。

更新注册表后，Cargo 会检查 `[dependencies]` 部分，并下载任何尚未下载的已列出的 crate。在这种情况下，虽然我们只列出了 `rand` 作为依赖，但 Cargo 也获取了 `rand` 依赖的其他 crate 才能工作。下载完 crate 后，Rust 编译它们，然后编译可用的依赖项目。

如果你立即再次运行 `cargo build` 而不做任何更改，你将看不到除 `Finished` 行之外的任何输出。Cargo 知道它已经下载并编译了依赖项，并且你没有在 _Cargo.toml_ 文件中更改它们。Cargo 也知道你没有更改任何代码，因此它也不会重新编译。无事可做，它就简单退出了。

如果你打开 _src/main.rs_ 文件，做一个微不足道的更改，然后保存并再次构建，你将只看到两行输出：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

这些行显示 Cargo 仅更新了构建，因为你对你对 _src/main.rs_ 文件的小改动。你的依赖项没有改变，因此 Cargo 知道它可以重复使用已经下载和编译的内容。

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### 确保可重现的构建

Cargo 有一个机制，可确保你或其他任何人每次构建代码时都能重建相同的工件：Cargo 将只使用你指定的依赖版本，直到你另有指示。例如，假设下周 `rand` crate 的 0.8.6 版本发布了，该版本包含一个重要错误修复，但也包含一个会破坏你的代码的回归。为了处理这种情况，Rust 会在你第一次运行 `cargo build` 时创建 _Cargo.lock_ 文件，因此我们现在在 _guessing_game_ 目录中有这个文件。

当你第一次构建项目时，Cargo 会找出符合条件的所有依赖版本，然后将它们写入 _Cargo.lock_ 文件。当你将来构建项目时，Cargo 会看到 _Cargo.lock_ 文件存在，并使用其中指定的版本，而不是重新做所有找出版本的工作。这让你自动获得可重现的构建。换句话说，多亏了 _Cargo.lock_ 文件，你的项目将保持在 0.8.5 版本，直到你显式升级。由于 _Cargo.lock_ 文件对于可重现的构建非常重要，它通常会与项目中的其余代码一起检入源代码管理。

#### 更新 crate 以获取新版本

当你 _确实_ 想要更新一个 crate 时，Cargo 提供了 `update` 命令，该命令将忽略 _Cargo.lock_ 文件，并找出符合 _Cargo.toml_ 中规范的所有最新版本。然后 Cargo 会将这些版本写入 _Cargo.lock_ 文件。否则，默认情况下，Cargo 只会查找大于 0.8.5 且小于 0.9.0 的版本。如果 `rand` crate 发布了两个新版本 0.8.6 和 0.999.0，那么如果你运行 `cargo update`，你会看到以下内容：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.999.0)
```

Cargo 会忽略 0.999.0 版本。此时，你还会注意到 _Cargo.lock_ 文件中的更改，表明你现在使用的 `rand` crate 版本是 0.8.6。要使用 `rand` 版本 0.999.0 或 0.999._x_ 系列中的任何版本，你需要将 _Cargo.toml_ 文件更新为如下所示（不要实际进行此更改，因为以下示例假定你使用的是 `rand` 0.8）：

```toml
[dependencies]
rand = "0.999.0"
```

下次运行 `cargo build` 时，Cargo 将更新可用 crate 的注册表，并根据你指定的新版本重新评估你的 `rand` 需求。

关于 [Cargo][doccargo]<!-- ignore --> 及其[生态系统][doccratesio]<!-- ignore -->，还有很多要说的，我们将在第 14 章讨论，但现在，这就是你需要知道的一切。Cargo 使得重用库变得非常容易，因此 Rustaceans 能够编写由多个包组装而成的更小项目。

### 生成一个随机数

让我们开始使用 `rand` 来生成一个要猜测的数字。下一步是更新 _src/main.rs_，如清单 2-3 所示。

<Listing number="2-3" file-name="src/main.rs" caption="添加生成随机数的代码">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

首先，我们添加一行 `use rand::Rng;`。`Rng` trait（特征）定义了随机数生成器实现的方法，并且这个 trait 必须在作用域内，我们才能使用这些方法。第 10 章将详细介绍 trait。

接下来，我们在中间添加了两行。在第一行中，我们调用 `rand::thread_rng` 函数，该函数提供我们将要使用的特定随机数生成器：一个当前执行线程本地且由操作系统提供种子的生成器。然后，我们在随机数生成器上调用 `gen_range` 方法。这个方法由 `Rng` trait 定义，我们通过 `use rand::Rng;` 语句将其引入作用域。`gen_range` 方法接受一个范围表达式作为参数，并生成该范围内的一个随机数。我们在这里使用的范围表达式形式为 `start..=end`，包括上下限，因此我们需要指定 `1..=100` 来请求一个 1 到 100 之间的数字。

> 注意：你不会仅仅知道要使用哪些 trait 以及要从 crate 调用哪些方法和函数，因此每个 crate 都有带有使用说明的文档。Cargo 的另一个巧妙功能是运行 `cargo doc --open` 命令将在本地构建所有依赖项提供的文档，并在你的浏览器中打开它。例如，如果你对 `rand` crate 中的其他功能感兴趣，请运行 `cargo doc --open` 并单击左侧边栏中的 `rand`。

第二行新代码打印了秘密数字。这在开发程序时用于测试非常有用，但我们将在最终版本中删除它。如果程序一开始就打印答案，那就不太像游戏了！

尝试运行该程序几次：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

你应该得到不同的随机数，而且它们都应该是 1 到 100 之间的数字。干得漂亮！

## 比较猜测与秘密数字

现在我们已经有了用户输入和一个随机数，可以比较它们了。这一步如清单 2-4 所示。请注意，这段代码暂时还不能编译，我们接下来会解释。

<Listing number="2-4" file-name="src/main.rs" caption="处理比较两个数字的可能返回值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

首先，我们添加另一个 `use` 语句，将标准库中一个名为 `std::cmp::Ordering` 的类型引入作用域。`Ordering` 类型是另一个枚举（enum），具有 `Less`、`Greater` 和 `Equal` 三个变体（variants）。这是比较两个值时可能出现的三种结果。

然后，我们在底部添加了五行新代码，使用了 `Ordering` 类型。`cmp` 方法比较两个值，可以在任何可比较的内容上调用。它接受一个对你想要比较的值的引用：在这里，它比较 `guess` 和 `secret_number`。然后，它返回我们通过 `use` 语句引入作用域的 `Ordering` 枚举的一个变体。我们使用一个 [`match`][match]<!-- ignore --> 表达式来决定接下来做什么，基于调用 `cmp` 比较 `guess` 和 `secret_number` 的值后返回的 `Ordering` 变体。

一个 `match` 表达式由 _分支_（arms）组成。每个分支包含一个要匹配的 _模式_（pattern），以及当提供给 `match` 的值匹配该分支的模式时应运行的代码。Rust 获取提供给 `match` 的值，然后依次检查每个分支的模式。模式和 `match` 结构是强大的 Rust 特性：它们让你能够表达代码可能遇到的各种情况，并确保你处理了所有情况。这些特性将分别在第 6 章和第 19 章详细介绍。

让我们通过这里使用的 `match` 表达式来举例说明。假设用户猜了 50，而此次随机生成的秘密数字是 38。

当代码比较 50 和 38 时，`cmp` 方法将返回 `Ordering::Greater`，因为 50 大于 38。`match` 表达式获取 `Ordering::Greater` 值并开始检查每个分支的模式。它查看第一个分支的模式 `Ordering::Less`，发现值 `Ordering::Greater` 不匹配 `Ordering::Less`，因此忽略该分支中的代码并移至下一个分支。下一个分支的模式是 `Ordering::Greater`，它 _确实_ 匹配了 `Ordering::Greater`！该分支中的关联代码将执行并在屏幕上打印 `Too big!`。`match` 表达式在第一个成功匹配后结束，因此在这种场景下它不会查看最后一个分支。

然而，清单 2-4 中的代码还不能编译。让我们试试：

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

错误的核心表明存在 _类型不匹配_（mismatched types）。Rust 拥有强大的静态类型系统。然而，它也有类型推断（type inference）。当我们写 `let mut guess = String::new()` 时，Rust 能够推断出 `guess` 应该是一个 `String`，而不需要我们写出类型。而 `secret_number` 则是一个数字类型。Rust 的几种数字类型可以具有 1 到 100 之间的值：`i32`（32 位数字）、`u32`（无符号 32 位数字）、`i64`（64 位数字）等等。除非另有指定，Rust 默认使用 `i32`，这就是 `secret_number` 的类型，除非你在其他地方添加了类型信息导致 Rust 推断出不同的数字类型。错误的原因是 Rust 无法比较字符串和数字类型。

最终，我们想要将程序读取为输入的 `String` 转换为数字类型，以便我们可以将其与秘密数字进行数值比较。我们通过在 `main` 函数体中添加这一行来实现：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

这一行是：

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

我们创建了一个名为 `guess` 的变量。但是等等，程序中不是已经有一个名为 `guess` 的变量了吗？是的，但 Rust 允许我们用新值覆盖 `guess` 的旧值，这很有用。_遮蔽_（Shadowing）让我们可以重用 `guess` 变量名，而不必强迫我们创建两个不同的变量，例如 `guess_str` 和 `guess`。我们将在[第 3 章][shadowing]<!-- ignore --> 中更详细地介绍这一点，但现在，要知道这个特性在你想要将值从一种类型转换为另一种类型时经常使用。

我们将这个新变量绑定到表达式 `guess.trim().parse()`。表达式中的 `guess` 指的是包含输入字符串的原始 `guess` 变量。`String` 实例上的 `trim` 方法将消除开头和结尾的任何空白，在将字符串转换为只能包含数字数据的 `u32` 之前，我们必须这样做。用户必须按下 <kbd>enter</kbd> 才能满足 `read_line` 并输入他们的猜测，这会在字符串中添加一个换行符。例如，如果用户输入 <kbd>5</kbd> 并按下 <kbd>enter</kbd>，`guess` 看起来像这样：`5\n`。`\n` 表示换行符（newline）。（在 Windows 上，按下 <kbd>enter</kbd> 会产生一个回车符和一个换行符，`\r\n`。）`trim` 方法会消除 `\n` 或 `\r\n`，结果只剩下 `5`。

[字符串上的 `parse` 方法][parse]<!-- ignore --> 将字符串转换为另一种类型。在这里，我们用它从字符串转换为数字。我们需要通过 `let guess: u32` 告诉 Rust 我们想要的精确数字类型。`guess` 后面的冒号（`:`）告诉 Rust 我们要标注变量的类型。Rust 有几种内置的数字类型；这里看到的 `u32` 是一个无符号 32 位整数。对于小的正数来说，这是一个很好的默认选择。你将在[第 3 章][integers]<!-- ignore --> 中了解其他数字类型。

此外，这个示例程序中的 `u32` 标注以及与 `secret_number` 的比较意味着 Rust 将推断 `secret_number` 也应该是 `u32`。所以，现在比较将在两个相同类型的值之间进行！

`parse` 方法只对可以逻辑转换为数字的字符有效，因此很容易导致错误。例如，如果字符串包含 `A👍%`，则无法将其转换为数字。因为它可能会失败，`parse` 方法返回一个 `Result` 类型，很像 `read_line` 方法（前面在[“使用 `Result` 处理潜在的错误”](#handling-potential-failure-with-result)<!-- ignore --> 中讨论过）。我们将以同样的方式通过再次使用 `expect` 方法来处理这个 `Result`。如果 `parse` 因为无法从字符串创建数字而返回 `Err` `Result` 变体，`expect` 调用将使游戏崩溃并打印我们给它的消息。如果 `parse` 成功将字符串转换为数字，它将返回 `Result` 的 `Ok` 变体，而 `expect` 将返回我们想要从 `Ok` 值中获取的数字。

让我们现在运行程序：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

太棒了！即使在猜测前添加了空格，程序仍然能判断出用户猜了 76。多次运行程序以验证不同类型输入的不同行为：猜对数字、猜一个太大的数字、猜一个太小的数字。

现在游戏大部分已经可以运行了，但用户只能猜一次。让我们通过添加循环来改变这一点！

## 使用循环允许多次猜测

`loop` 关键字创建了一个无限循环。我们将添加一个循环，给用户更多猜测数字的机会：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

如你所见，我们将从猜测输入提示开始的所有内容移到了一个循环中。确保将循环内的行再缩进四个空格，然后再次运行程序。程序现在会永远要求再猜一次，这实际上引入了一个新问题。用户似乎无法退出！

用户总是可以通过键盘快捷键 <kbd>ctrl</kbd>-<kbd>C</kbd> 中断程序。但还有另一种方法可以逃离这个永不满足的怪物，正如在 `parse` 讨论中提到的[“比较猜测与秘密数字”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->：如果用户输入一个非数字的回答，程序将崩溃。我们可以利用这一点来允许用户退出，如下所示：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

输入 `quit` 将退出游戏，但你会注意到，输入任何其他非数字输入也会退出。至少可以说这不太理想；我们希望在猜对数字时游戏也能停止。

### 猜对后退出

让我们通过在用户获胜时添加 `break` 语句来编程让游戏退出：

<span class="filename">文件名: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

在 `You win!` 之后添加 `break` 行使得程序在用户猜对秘密数字时退出循环。退出循环也意味着退出程序，因为循环是 `main` 的最后一部分。

### 处理无效输入

为了进一步优化游戏的行为，我们让游戏忽略非数字输入，而不是在用户输入非数字时使程序崩溃，这样用户可以继续猜测。我们可以通过修改将 `guess` 从 `String` 转换为 `u32` 的那一行来实现，如清单 2-5 所示。

<Listing number="2-5" file-name="src/main.rs" caption="忽略非数字猜测并要求重新猜测，而不是使程序崩溃">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

我们将 `expect` 调用切换为 `match` 表达式，从出错时崩溃转变为处理错误。回想一下，`parse` 返回一个 `Result` 类型，而 `Result` 是一个枚举，具有 `Ok` 和 `Err` 变体。我们在这里使用了 `match` 表达式，就像我们对 `cmp` 方法的 `Ordering` 结果所做的那样。

如果 `parse` 能够成功地将字符串转换为数字，它将返回一个包含结果数字的 `Ok` 值。该 `Ok` 值将匹配第一个分支的模式，`match` 表达式将直接返回 `parse` 产生并放入 `Ok` 值中的 `num` 值。该数字将正好位于我们在新创建的 `guess` 变量中想要的位置。

如果 `parse` _不能_ 将字符串转换为数字，它将返回一个包含有关错误的更多信息的 `Err` 值。`Err` 值不匹配第一个 `match` 分支中的 `Ok(num)` 模式，但它确实匹配第二个分支中的 `Err(_)` 模式。下划线 `_` 是一个通配符（catch-all value）；在这个例子中，我们说我们想要匹配所有 `Err` 值，无论它们内部包含什么信息。因此，程序将执行第二个分支的代码 `continue`，这告诉程序进入 `loop` 的下一次迭代并请求另一个猜测。所以，实际上，程序忽略了 `parse` 可能遇到的所有错误！

现在程序中的所有内容都应该按预期工作了。让我们试试：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

太棒了！再做一个小小的最后调整，我们就完成了猜数字游戏。回想一下，程序仍然在打印秘密数字。这在测试时效果很好，但会破坏游戏。让我们删除输出秘密数字的 `println!`。清单 2-6 显示了最终代码。

<Listing number="2-6" file-name="src/main.rs" caption="完整的猜数字游戏代码">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

至此，你已经成功构建了猜数字游戏。恭喜！


## 总结

这个项目是一个动手实践的方式，向你介绍了许多新的 Rust 概念：`let`、`match`、函数、使用外部 crate 等等。在接下来的几章中，你将更详细地学习这些概念。第 3 章涵盖大多数编程语言都有的概念，例如变量、数据类型和函数，并展示如何在 Rust 中使用它们。第 4 章探讨所有权（ownership），这是使 Rust 与其他语言不同的一个特性。第 5 章讨论结构体（structs）和方法语法，第 6 章解释枚举（enums）如何工作。

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
