## 附录 C：可派生 trait（Derivable Traits）

在本书的多个地方，我们讨论了 `derive` 属性（attribute），你可以将其应用于结构体（struct）或枚举（enum）定义。`derive` 属性会生成代码，在你使用 `derive` 语法标注的类型上，以 trait 自身的默认实现来实现该 trait。

在本附录中，我们提供了标准库中所有可与 `derive` 配合使用的 trait 的参考。每个章节涵盖以下内容：

- 派生（derive）该 trait 将启用的运算符和方法
- `derive` 提供的 trait 实现做了什么
- 实现该 trait 对类型意味着什么
- 允许或不允许实现该 trait 的条件
- 需要该 trait 的操作示例

如果你想要与 `derive` 属性提供的不同的行为，请查阅每个 trait 的[标准库文档](../std/index.html)<!-- ignore -->，了解如何手动实现它们。

这里列出的 trait 是标准库中唯一可以使用 `derive` 在你的类型上实现的 trait。标准库中定义的其他 trait 没有合理的默认行为，因此需要你来以最适合你目标的方式实现它们。

一个无法派生（derive）的 trait 示例是 `Display`，它负责为最终用户处理格式化。你应该始终考虑向最终用户展示类型的适当方式。最终用户应该能够看到类型的哪些部分？哪些部分对他们来说是相关的？哪种数据格式对他们最有用？Rust 编译器没有这种洞察力，因此无法为你提供合适的默认行为。

本附录中提供的可派生 trait 列表并不全面：库可以为自己的 trait 实现 `derive`，这使得你可以使用 `derive` 的 trait 列表实际上是无限开放的。实现 `derive` 涉及使用过程宏（procedural macro），这在第 20 章的[“自定义 `derive` 宏”][custom-derive-macros]<!-- ignore -->章节中有介绍。

### `Debug` — 为程序员提供输出（for Programmer Output）

`Debug` trait 支持在格式字符串（format strings）中使用调试格式化，通过向 `{}` 占位符中添加 `:?` 来指示。

`Debug` trait 允许你出于调试目的打印类型的实例，因此你和使用你类型的其他程序员可以在程序执行的特定时刻检查实例。

例如，在使用 `assert_eq!` 宏时需要 `Debug` trait。如果相等性断言失败，该宏会打印作为参数给出的实例的值，以便程序员查看两个实例为什么不相等。

### `PartialEq` 和 `Eq` — 相等性比较（for Equality Comparisons）

`PartialEq` trait 允许你比较类型的实例以检查是否相等，并支持使用 `==` 和 `!=` 运算符。

派生 `PartialEq` 会实现 `eq` 方法。当在结构体上派生 `PartialEq` 时，只有在 **所有** 字段都相等时两个实例才相等，如果 **任一** 字段不相等则两个实例不相等。当在枚举上派生时，每个变体（variant）与自身相等，与其他变体不相等。

例如，在使用 `assert_eq!` 宏时需要 `PartialEq` trait，该宏需要能够比较两个类型实例是否相等。

`Eq` trait 没有方法。其目的是表明对于标注类型的每个值，该值都等于自身。`Eq` trait 只能应用于同时也实现了 `PartialEq` 的类型，尽管并非所有实现了 `PartialEq` 的类型都能实现 `Eq`。一个例子是浮点数类型：浮点数的实现规定非数字（`NaN`）值的两个实例彼此不相等。

需要 `Eq` 的一个例子是 `HashMap<K, V>` 中的键，这样 `HashMap<K, V>` 才能判断两个键是否相同。

### `PartialOrd` 和 `Ord` — 顺序比较（for Ordering Comparisons）

`PartialOrd` trait 允许你比较类型的实例以进行排序。实现了 `PartialOrd` 的类型可以用于 `<`、`>`、`<=` 和 `>=` 运算符。你只能将 `PartialOrd` trait 应用于同时实现了 `PartialEq` 的类型。

派生 `PartialOrd` 会实现 `partial_cmp` 方法，该方法返回一个 `Option<Ordering>`，当给定的值无法产生顺序时返回 `None`。一个无法产生顺序的值示例是 `NaN` 浮点值，尽管该类型的大多数值都可以进行比较。使用任何浮点数和 `NaN` 浮点值调用 `partial_cmp` 将返回 `None`。

当在结构体上派生时，`PartialOrd` 按照字段在结构体定义中出现的顺序比较每个字段的值来比较两个实例。当在枚举上派生时，枚举定义中较早声明的变体被认为小于较晚列出的变体。

例如，`rand` crate 中的 `gen_range` 方法需要 `PartialOrd` trait，该方法在范围表达式指定的范围内生成一个随机值。

`Ord` trait 让你知道对于标注类型的任意两个值，都会存在一个有效的排序。`Ord` trait 实现了 `cmp` 方法，该方法返回一个 `Ordering` 而非 `Option<Ordering>`，因为始终会存在一个有效的排序。你只能将 `Ord` trait 应用于同时实现了 `PartialOrd` 和 `Eq` 的类型（而 `Eq` 又需要 `PartialEq`）。当在结构体和枚举上派生时，`cmp` 的行为与 `PartialOrd` 的 `partial_cmp` 派生实现相同。

需要 `Ord` 的一个例子是将值存储到 `BTreeSet<T>` 中时，这是一种基于值的排序顺序来存储数据的数据结构。

### `Clone` 和 `Copy` — 复制值（for Duplicating Values）

`Clone` trait 允许你显式创建值的深拷贝（deep copy），复制过程可能涉及运行任意代码和拷贝堆（heap）数据。更多关于 `Clone` 的信息请参见第 4 章的[“变量与数据的交互：Clone”][variables-and-data-interacting-with-clone]<!-- ignore -->章节。

派生 `Clone` 会实现 `clone` 方法，当为整个类型实现时，会在该类型的各个部分上调用 `clone`。这意味着类型中的所有字段或值也必须实现 `Clone` 才能派生 `Clone`。

需要 `Clone` 的一个例子是在切片（slice）上调用 `to_vec` 方法时。切片并不拥有其包含的类型实例，但从 `to_vec` 返回的向量（vector）需要拥有其实例，因此 `to_vec` 在每个项上调用 `clone`。因此，存储在切片中的类型必须实现 `Clone`。

`Copy` trait 允许你仅通过复制存储在栈（stack）上的位来复制值；无需执行任意代码。更多关于 `Copy` 的信息请参见第 4 章的[“仅栈数据：Copy”][stack-only-data-copy]<!-- ignore -->章节。

`Copy` trait 没有定义任何方法，以防止程序员重载这些方法并违反"没有执行任意代码"的假设。这样，所有程序员都可以假设复制一个值会非常快。

你可以对任何其所有组成部分都实现了 `Copy` 的类型派生 `Copy`。实现了 `Copy` 的类型还必须实现 `Clone`，因为实现了 `Copy` 的类型有一个简单的 `Clone` 实现，其执行与 `Copy` 相同的任务。

`Copy` trait 很少被要求；实现了 `Copy` 的类型可以使用优化，这意味着你不必调用 `clone`，从而使代码更简洁。

使用 `Copy` 能完成的所有事情，使用 `Clone` 也能完成，但代码可能更慢或需要在某些地方使用 `clone`。

### `Hash` — 将值映射为固定大小的值（for Mapping a Value to a Value of Fixed Size）

`Hash` trait 允许你获取一个任意大小的类型实例，并使用哈希函数（hash function）将该实例映射为一个固定大小的值。派生 `Hash` 会实现 `hash` 方法。`hash` 方法的派生实现会将对该类型各个部分调用 `hash` 的结果组合起来，这意味着所有字段或值也必须实现 `Hash` 才能派生 `Hash`。

需要 `Hash` 的一个示例是在 `HashMap<K, V>` 中存储键，以便高效地存储数据。

### `Default` — 默认值（for Default Values）

`Default` trait 允许你为类型创建默认值。派生 `Default` 会实现 `default` 函数。`default` 函数的派生实现会在该类型的每个部分上调用 `default` 函数，这意味着该类型中的所有字段或值也必须实现 `Default` 才能派生 `Default`。

`Default::default` 函数通常与第 5 章中讨论的[“使用结构体更新语法从其他实例创建实例”][creating-instances-from-other-instances-with-struct-update-syntax]<!-- ignore -->章节中的结构体更新语法（struct update syntax）结合使用。你可以自定义结构体的几个字段，然后使用 `..Default::default()` 为其余字段设置并使用默认值。

例如，在 `Option<T>` 实例上使用 `unwrap_or_default` 方法时需要 `Default` trait。如果 `Option<T>` 是 `None`，`unwrap_or_default` 方法将返回 `Option<T>` 中存储的类型 `T` 的 `Default::default` 结果。

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[custom-derive-macros]: ch20-05-macros.html#custom-derive-macros
