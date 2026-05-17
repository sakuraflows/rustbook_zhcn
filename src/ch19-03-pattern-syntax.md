## 模式语法

在本节中，我们汇总了所有在模式中有效的语法，并讨论为什么以及何时你可能想要使用每一种。

### 匹配字面量

正如你在第 6 章中看到的，你可以直接将模式与字面量进行匹配。以下代码给出了一些示例：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

这段代码打印 `one`，因为 `x` 中的值是 `1`。当你希望代码在接收到某个特定的具体值时采取行动，这种语法非常有用。

### 匹配命名变量

命名变量是不可反驳的模式，可以匹配任何值，我们在本书中已经多次使用过它们。然而，当你在 `match`、`if let` 或 `while let` 表达式中使用命名变量时，会遇到一个复杂情况。因为这些表达式都会开启一个新的作用域，在这些表达式内部作为模式一部分声明的变量，会像所有变量一样遮蔽外部的同名变量。在清单 19-11 中，我们声明了一个值为 `Some(5)` 的变量 `x`，以及一个值为 `10` 的变量 `y`。然后，我们在值 `x` 上创建了一个 `match` 表达式。请查看匹配分支中的模式和末尾的 `println!`，并在运行此代码或继续阅读之前，试着弄清楚代码会打印什么。

<Listing number="19-11" file-name="src/main.rs" caption="一个 `match` 表达式，其中一个分支引入了一个遮蔽现有变量 `y` 的新变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

让我们来逐步分析 `match` 表达式运行时发生了什么。第一个匹配分支的模式不匹配 `x` 的定义值，因此代码继续执行。

第二个匹配分支的模式引入了一个名为 `y` 的新变量，它将匹配 `Some` 值中的任何值。因为我们处于 `match` 表达式内部的新作用域中，这是一个新的 `y` 变量，不是我们在开头声明的值为 `10` 的那个 `y`。这个新的 `y` 绑定会匹配 `Some` 中的任何值，而这正是 `x` 中的情况。因此，这个新的 `y` 绑定到了 `x` 中 `Some` 的内部值。该值为 `5`，因此该分支的表达式执行并打印 `Matched, y = 5`。

如果 `x` 是 `None` 值而不是 `Some(5)`，前两个分支的模式都不会匹配，因此值将匹配下划线。我们没有在下划线分支的模式中引入 `x` 变量，因此表达式中的 `x` 仍然是未被遮蔽的外部 `x`。在这种假设情况下，`match` 将打印 `Default case, x = None`。

当 `match` 表达式结束时，它的作用域也随之结束，内部 `y` 的作用域也是如此。最后的 `println!` 输出 `at the end: x = Some(5), y = 10`。

要创建一个 `match` 表达式来比较外部 `x` 和 `y` 的值，而不是引入一个遮蔽现有 `y` 变量的新变量，我们需要改用匹配守卫（match guard）条件。我们将在后面的[“使用匹配守卫添加条件”](#adding-conditionals-with-match-guards)<!-- ignore -->部分讨论匹配守卫。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->
<a id="multiple-patterns"></a>

### 匹配多个模式

在 `match` 表达式中，你可以使用 `|` 语法匹配多个模式，这是模式的*或（or）*运算符。例如，在以下代码中，我们将 `x` 的值与匹配分支进行匹配，第一个分支有一个*或*选项，意味着如果 `x` 的值匹配该分支中的任何一个值，该分支的代码就会运行：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

这段代码打印 `one or two`。

### 使用 `..=` 匹配值的范围

`..=` 语法允许我们匹配一个包含的范围（inclusive range）内的值。在以下代码中，当模式匹配给定范围内的任何一个值时，该分支将执行：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

如果 `x` 是 `1`、`2`、`3`、`4` 或 `5`，第一个分支将匹配。这种语法对于多个匹配值来说，比使用 `|` 运算符表达相同的意思更方便；如果我们使用 `|`，将不得不指定 `1 | 2 | 3 | 4 | 5`。指定范围要简短得多，特别是如果我们想要匹配，比如，1 到 1,000 之间的任何数字！

编译器在编译时会检查范围是否为空，由于 Rust 唯一能判断范围是否为空或非空的类型是 `char` 和数值类型，因此范围只允许用于数值或 `char` 值。

以下是一个使用 `char` 值范围的示例：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust 可以判断出 `'c'` 位于第一个模式的范围内，并打印 `early ASCII letter`。

### 解构以分解值

我们还可以使用模式来解构结构体、枚举和元组，以便使用这些值的不同部分。让我们逐一介绍每种值。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="destructuring-structs"></a>

#### 结构体

清单 19-12 展示了一个具有两个字段 `x` 和 `y` 的 `Point` 结构体，我们可以使用带有模式的 `let` 语句将其分解。

<Listing number="19-12" file-name="src/main.rs" caption="将结构体的字段解构为单独的变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

这段代码创建了变量 `a` 和 `b`，它们匹配 `p` 结构体的 `x` 和 `y` 字段的值。这个示例表明，模式中变量的名称不必与结构体的字段名称相同。然而，通常的做法是让变量名称与字段名称匹配，以便更容易记住哪些变量来自哪些字段。由于这种常见用法，并且编写 `let Point { x: x, y: y } = p;` 包含大量重复，Rust 为匹配结构体字段的模式提供了一种简写形式：你只需列出结构体字段的名称，从模式中创建的变量将具有相同的名称。清单 19-13 的行为与清单 19-12 中的代码相同，但 `let` 模式中创建的变量是 `x` 和 `y`，而不是 `a` 和 `b`。

<Listing number="19-13" file-name="src/main.rs" caption="使用结构体字段简写解构结构体字段">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

此代码创建了变量 `x` 和 `y`，它们匹配 `p` 变量的 `x` 和 `y` 字段。结果是变量 `x` 和 `y` 包含了来自 `p` 结构体的值。

我们还可以在结构体模式中使用字面量值进行解构，而不是为所有字段创建变量。这样做允许我们测试某些字段是否为特定值，同时为其他字段创建变量以进行解构。

在清单 19-14 中，我们有一个 `match` 表达式，它将 `Point` 值分为三种情况：位于 `x` 轴上的点（当 `y = 0` 时为真）、位于 `y` 轴上的点（`x = 0`），以及不在任一轴上的点。

<Listing number="19-14" file-name="src/main.rs" caption="在一个模式中解构和匹配字面量值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

第一个分支通过指定当 `y` 字段的值匹配字面量 `0` 时匹配，从而匹配任何位于 `x` 轴上的点。该模式仍然创建了一个 `x` 变量，我们可以在该分支的代码中使用它。

类似地，第二个分支通过指定当 `x` 字段的值为 `0` 时匹配，从而匹配任何位于 `y` 轴上的点，并为 `y` 字段的值创建一个变量 `y`。第三个分支没有指定任何字面量，因此它匹配任何其他 `Point`，并为 `x` 和 `y` 字段都创建变量。

在这个示例中，值 `p` 因为 `x` 包含 `0` 而匹配了第二个分支，因此这段代码将打印 `On the y axis at 7`。

请记住，`match` 表达式一旦找到第一个匹配的模式就会停止检查分支，因此即使 `Point { x: 0, y: 0 }` 同时位于 `x` 轴和 `y` 轴上，这段代码也只会打印 `On the x axis at 0`。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="destructuring-enums"></a>

#### 枚举

我们在本书中已经解构过枚举（例如，第 6 章的清单 6-5），但尚未明确讨论的是，用于解构枚举的模式对应于枚举内部存储数据的定义方式。例如，在清单 19-15 中，我们使用了清单 6-2 中的 `Message` 枚举，并编写了一个 `match`，其模式将解构每个内部值。

<Listing number="19-15" file-name="src/main.rs" caption="解构持有不同类型值的枚举变体">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

这段代码将打印 `Change color to red 0, green 160, and blue 255`。尝试更改 `msg` 的值，以查看其他分支的代码运行情况。

对于没有数据的枚举变体，例如 `Message::Quit`，我们无法进一步解构该值。我们只能匹配字面量 `Message::Quit` 值，并且该模式中没有变量。

对于类似结构体的枚举变体，例如 `Message::Move`，我们可以使用类似于匹配结构体时指定的模式。在变体名称之后，我们放置花括号，然后列出带有变量的字段，以便分解出各个部分用于该分支的代码中。这里我们使用了与清单 19-13 中相同的简写形式。

对于类似元组的枚举变体，例如持有一个包含一个元素的元组的 `Message::Write` 和持有一个包含三个元素的元组的 `Message::ChangeColor`，其模式类似于我们指定用于匹配元组的模式。模式中的变量数量必须匹配我们要匹配的变体中的元素数量。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="destructuring-nested-structs-and-enums"></a>

#### 嵌套的结构体和枚举

到目前为止，我们的示例都只匹配一层深度的结构体或枚举，但匹配也可以作用于嵌套项！例如，我们可以重构清单 19-15 中的代码，以支持 `ChangeColor` 消息中的 RGB 和 HSV 颜色，如清单 19-16 所示。

<Listing number="19-16" caption="匹配嵌套枚举">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` 表达式中第一个分支的模式匹配一个包含 `Color::Rgb` 变体的 `Message::ChangeColor` 枚举变体；然后，该模式绑定到三个内部的 `i32` 值。第二个分支的模式也匹配一个 `Message::ChangeColor` 枚举变体，但内部的枚举匹配的是 `Color::Hsv`。我们可以在一个 `match` 表达式中指定这些复杂的条件，即使涉及两个枚举。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="destructuring-structs-and-tuples"></a>

#### 结构体和元组

我们可以以更复杂的方式混合、匹配和嵌套解构模式。以下示例展示了一个复杂的解构，我们在元组内嵌套了结构体和元组，并将所有基本类型的值解构出来：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

这段代码让我们将复杂类型分解为其组成部分，以便我们可以分别使用我们感兴趣的值。

使用模式解构是一种方便的方式，可以将值的各个部分（例如结构体中每个字段的值）彼此分开使用。

### 忽略模式中的值

你已经看到，有时忽略模式中的值是很有用的，例如在 `match` 的最后一个分支中，使用一个实际上不执行任何操作但又占用了所有剩余可能值的万能分支。有几种方法可以忽略模式中的整个值或部分值：使用 `_` 模式（你已见过）、在另一个模式中使用 `_` 模式、使用以下划线开头的名称，或者使用 `..` 来忽略值的剩余部分。让我们探讨如何使用以及为什么使用这些模式中的每一种。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="ignoring-an-entire-value-with-_"></a>

#### 使用 `_` 忽略整个值

我们已经使用下划线作为通配符模式，它将匹配任何值但不会绑定到该值。这在 `match` 表达式的最后一个分支中特别有用，但我们也可以在任何模式中使用它，包括函数参数，如清单 19-17 所示。

<Listing number="19-17" file-name="src/main.rs" caption="在函数签名中使用 `_`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

这段代码将完全忽略作为第一个参数传递的值 `3`，并打印 `This code only uses the y parameter: 4`。

在大多数情况下，当你不再需要某个特定的函数参数时，你应该更改签名，使其不包含未使用的参数。但在某些情况下，忽略函数参数特别有用，例如当你正在实现一个 trait，需要某个特定的类型签名，但你的实现中的函数体并不需要其中一个参数时。这样你就可以避免得到关于未使用函数参数的编译器警告，而如果你使用一个名称，就会得到警告。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### 使用嵌套的 `_` 忽略值的部分

我们还可以在另一个模式内部使用 `_` 来忽略值的某一部分，例如当我们只想测试值的某一部分，而对其他部分在相应要运行的代码中没有使用时。清单 19-18 展示了负责管理设置值的代码。业务要求是，用户不应允许覆盖设置的现有自定义值，但如果设置当前未设置，则可以取消设置并为其赋予一个值。

<Listing number="19-18" caption="当不需要使用 `Some` 内部的值时，在匹配 `Some` 变体的模式中使用下划线">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

这段代码将打印 `Can't overwrite an existing customized value`，然后打印 `setting is Some(5)`。在第一个匹配分支中，我们不需要匹配或使用任一 `Some` 变体内部的值，但我们需要测试 `setting_value` 和 `new_setting_value` 都是 `Some` 变体的情况。在这种情况下，我们打印不更改 `setting_value` 的原因，并且它不会被更改。

在所有其他情况下（如果 `setting_value` 或 `new_setting_value` 中有一个是 `None`），由第二个分支中的 `_` 模式表示，我们希望允许 `new_setting_value` 成为 `setting_value`。

我们还可以在一个模式中的多个位置使用下划线来忽略特定的值。清单 19-19 展示了一个忽略五个元素元组中第二个和第四个值的示例。

<Listing number="19-19" caption="忽略元组中的多个部分">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

这段代码将打印 `Some numbers: 2, 8, 32`，而值 `4` 和 `16` 将被忽略。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### 通过以下划线开头名称来忽略未使用的变量

如果你创建了一个变量但从未使用过它，Rust 通常会发出警告，因为未使用的变量可能是一个 bug。然而，有时能够创建一个你暂时还不会使用的变量是有用的，例如当你正在做原型设计或刚开始一个项目时。在这种情况下，你可以通过以_下划线开头_的变量名来告诉 Rust 不要警告你未使用的变量。在清单 19-20 中，我们创建了两个未使用的变量，但当我们编译这段代码时，我们只应收到关于其中一个变量的警告。

<Listing number="19-20" file-name="src/main.rs" caption="以下划线开头的变量名以避免未使用变量的警告">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

在这里，我们收到了一个关于未使用变量 `y` 的警告，但没有收到关于未使用 `_x` 的警告。

请注意，仅使用 `_` 和使用以下划线开头的名称之间存在细微差别。语法 `_x` 仍然将值绑定到变量，而 `_` 根本不会绑定。为了说明这种区别重要的情况，清单 19-21 将给我们一个错误。

<Listing number="19-21" caption="以下划线开头的未使用变量仍然会绑定值，这可能会取得该值的所有权。">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

我们会收到一个错误，因为 `s` 值仍然会被移动到 `_s` 中，这阻止了我们再次使用 `s`。然而，仅使用下划线本身永远不会绑定到值。清单 19-22 将编译通过且没有任何错误，因为 `s` 没有被移动到 `_` 中。

<Listing number="19-22" caption="使用下划线不会绑定该值。">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

这段代码运行得很好，因为我们从未将 `s` 绑定到任何东西；它没有被移动。

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### 使用 `..` 忽略值的剩余部分

对于具有许多部分的值，我们可以使用 `..` 语法来使用特定部分并忽略其余部分，从而避免为每个忽略的值列出下划线。`..` 模式会忽略模式中未显式匹配的值的任何部分。在清单 19-23 中，我们有一个 `Point` 结构体，它保存了三维空间中的坐标。在 `match` 表达式中，我们只想操作 `x` 坐标，并忽略 `y` 和 `z` 字段的值。

<Listing number="19-23" caption="使用 `..` 忽略 `Point` 中除 `x` 之外的所有字段">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

我们列出了 `x` 的值，然后仅包含 `..` 模式。这比必须列出 `y: _` 和 `z: _` 更快捷，特别是当我们处理具有大量字段的结构体，而只有一两个字段相关时。

`..` 语法会展开为它所需要匹配的任意数量的值。清单 19-24 展示了如何将 `..` 与元组一起使用。

<Listing number="19-24" file-name="src/main.rs" caption="仅匹配元组中的第一个和最后一个值，并忽略所有其他值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

在这段代码中，第一个和最后一个值被匹配为 `first` 和 `last`。`..` 将匹配并忽略中间的所有内容。

然而，使用 `..` 必须是明确的。如果不清楚哪些值用于匹配、哪些应该被忽略，Rust 会给我们一个错误。清单 19-25 展示了一个模糊使用 `..` 的示例，因此它不会编译。

<Listing number="19-25" file-name="src/main.rs" caption="尝试以模糊的方式使用 `..`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

当我们编译这个示例时，会得到以下错误：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust 无法确定在匹配 `second` 值之前要忽略元组中的多少个值，然后再忽略之后还有多少个值。这段代码可能意味着我们希望忽略 `2`，将 `second` 绑定到 `4`，然后忽略 `8`、`16` 和 `32`；或者我们希望忽略 `2` 和 `4`，将 `second` 绑定到 `8`，然后忽略 `16` 和 `32`；等等。变量名 `second` 对 Rust 没有任何特殊含义，因此我们得到一个编译器错误，因为在两个位置像这样使用 `..` 是模糊的。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="extra-conditionals-with-match-guards"></a>

### 使用匹配守卫添加条件

*匹配守卫（match guard）*是 `match` 分支中模式之后指定的额外 `if` 条件，也必须满足该条件才能选择该分支。匹配守卫对于表达比单独模式更复杂的想法非常有用。但请注意，它们仅在 `match` 表达式中可用，不能在 `if let` 或 `while let` 表达式中使用。

该条件可以使用模式中创建的变量。清单 19-26 展示了一个 `match`，其中第一个分支的模式是 `Some(x)`，并且还有一个匹配守卫 `if x % 2 == 0`（如果该数字是偶数，则为 `true`）。

<Listing number="19-26" caption="为模式添加匹配守卫">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

此示例将打印 `The number 4 is even`。当 `num` 与第一个分支中的模式进行比较时，它匹配是因为 `Some(4)` 匹配 `Some(x)`。然后，匹配守卫检查 `x` 除以 2 的余数是否等于 0，由于的确如此，因此选择了第一个分支。

如果 `num` 是 `Some(5)`，第一个分支中的匹配守卫将是 `false`，因为 5 除以 2 的余数是 1，不等于 0。然后 Rust 将转到第二个分支，它会匹配，因为第二个分支没有匹配守卫，因此匹配任何 `Some` 变体。

无法在模式内部表达 `if x % 2 == 0` 条件，因此匹配守卫赋予了我们表达这种逻辑的能力。这种额外表达能力的缺点是，当涉及匹配守卫表达式时，编译器不会尝试检查穷尽性。

在讨论清单 19-11 时，我们提到可以使用匹配守卫来解决模式遮蔽问题。回想一下，我们在 `match` 表达式的模式中创建了一个新变量，而不是使用 `match` 外部的变量。那个新变量意味着我们无法测试外部变量的值。清单 19-27 展示了如何使用匹配守卫来解决这个问题。

<Listing number="19-27" file-name="src/main.rs" caption="使用匹配守卫测试与外部变量的相等性">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

这段代码现在将打印 `Default case, x = Some(5)`。第二个匹配分支中的模式没有引入新的变量 `y` 来遮蔽外部的 `y`，这意味着我们可以在匹配守卫中使用外部的 `y`。我们不将模式指定为 `Some(y)`（这会遮蔽外部的 `y`），而是指定为 `Some(n)`。这创建了一个新的变量 `n`，它不会遮蔽任何东西，因为在 `match` 外部没有 `n` 变量。

匹配守卫 `if n == y` 不是一个模式，因此不会引入新变量。这个 `y` 就是外部的 `y`，而不是一个遮蔽它的新 `y`，我们可以通过比较 `n` 和 `y` 来查找与外部 `y` 具有相同值的值。

你也可以在匹配守卫中使用*或*运算符 `|` 来指定多个模式；匹配守卫条件将应用于所有模式。清单 19-28 展示了将使用 `|` 的模式与匹配守卫结合时的优先级。此示例的重要部分是 `if y` 匹配守卫应用于 `4`、`5` *和* `6`，即使可能看起来 `if y` 只应用于 `6`。

<Listing number="19-28" caption="将多个模式与匹配守卫结合">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

匹配条件表明，只有在 `x` 的值等于 `4`、`5` 或 `6` *并且* `y` 为 `true` 时，该分支才匹配。当这段代码运行时，第一个分支的模式匹配，因为 `x` 是 `4`，但匹配守卫 `if y` 是 `false`，因此没有选择第一个分支。代码继续到第二个分支，它确实匹配，此程序打印 `no`。原因是 `if` 条件应用于整个模式 `4 | 5 | 6`，而不仅仅是最后一个值 `6`。换句话说，匹配守卫相对于模式的优先级行为如下：

```text
(4 | 5 | 6) if y => ...
```

而不是这样：

```text
4 | 5 | (6 if y) => ...
```

运行代码后，优先级行为就很明显了：如果匹配守卫仅应用于使用 `|` 运算符指定的值列表中的最后一个值，那么该分支将匹配，程序将打印 `yes`。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="-bindings"></a>

### 使用 `@` 绑定

*at* 运算符 `@` 让我们在测试值是否匹配模式的同时，创建一个持有该值的变量。在清单 19-29 中，我们想测试 `Message::Hello` 的 `id` 字段是否在范围 `3..=7` 内。我们还希望将该值绑定到变量 `id`，以便在关联分支的代码中使用它。

<Listing number="19-29" caption="使用 `@` 在测试值的同时将其绑定到模式中的变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

此示例将打印 `Found an id in range: 5`。通过在范围 `3..=7` 之前指定 `id @`，我们在测试该值是否匹配范围模式的同时，将匹配到范围的值捕获到一个名为 `id` 的变量中。

在第二个分支中，我们只在模式中指定了一个范围，该分支关联的代码没有一个包含 `id` 字段实际值的变量。`id` 字段的值可能是 10、11 或 12，但该模式关联的代码并不知道它是哪一个。模式代码无法使用 `id` 字段的值，因为我们没有将 `id` 值保存到变量中。

在最后一个分支中，我们指定了一个没有范围的变量，我们确实有一个可在分支代码中使用的变量 `id` 中的值。原因是我们使用了结构体字段简写语法。但是，我们在这个分支中没有像前两个分支那样对 `id` 字段中的值进行任何测试：任何值都会匹配这个模式。

使用 `@` 让我们可以在一个模式中测试值并将其保存到变量中。

## 总结

Rust 的模式在区分不同类型的数据时非常有用。当在 `match` 表达式中使用时，Rust 确保你的模式覆盖了所有可能的值，否则你的程序将无法编译。`let` 语句和函数参数中的模式使这些结构更加有用，支持将值解构为更小的部分并将这些部分赋值给变量。我们可以创建简单或复杂的模式来满足我们的需求。

接下来，作为本书倒数第二章，我们将探讨 Rust 各种特性的一些高级方面。
