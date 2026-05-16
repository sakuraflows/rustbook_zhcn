## 在哈希映射（Hash Map）中存储键与值的关联

我们最后一个常见的集合是哈希映射（hash map）。`HashMap<K, V>` 类型使用*哈希函数（hashing function）*存储类型 `K` 的键到类型 `V` 的值的映射，该函数决定了它如何将这些键和值放入内存。许多编程语言都支持这种数据结构，但它们通常使用不同的名称，例如 *hash*、*map*、*object*、*hash table*、*dictionary* 或 *associative array*，仅举几例。

当你想要查找数据时，不是像向量那样使用索引，而是使用可以是任何类型的键，这时哈希映射就很有用。例如，在一个游戏中，你可以使用哈希映射来跟踪每个队伍的得分，其中每个键是队伍名称，值是每个队伍的得分。给定队伍名称，你可以检索其得分。

我们将在本节中介绍哈希映射的基本 API，但标准库在 `HashMap<K, V>` 上定义的函数中隐藏着更多好东西。和往常一样，请查阅标准库文档了解更多信息。

### 创建新哈希映射

创建空哈希映射的一种方法是使用 `new`，并使用 `insert` 添加元素。在示例 8-20 中，我们跟踪两个队伍（名为 _Blue_ 和 _Yellow_）的得分。Blue 队从 10 分开始，Yellow 队从 50 分开始。

<Listing number="8-20" caption="创建一个新的哈希映射并插入一些键和值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

注意，我们首先需要 `use` 标准库 collections 部分中的 `HashMap`。在我们的三种常见集合中，这是最不常用的，因此它没有被自动包含在 prelude（预导入）中。哈希映射从标准库获得的支持也较少；例如，没有内置的宏来构造它们。

与向量一样，哈希映射将其数据存储在堆上。这个 `HashMap` 的键是 `String` 类型，值是 `i32` 类型。与向量一样，哈希映射是同质的：所有键必须具有相同的类型，所有值必须具有相同的类型。

### 访问哈希映射中的值

我们可以通过将键提供给 `get` 方法来从哈希映射中获取值，如示例 8-21 所示。

<Listing number="8-21" caption="访问存储在哈希映射中的 Blue 队伍的得分">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

在这里，`score` 将具有与 Blue 队关联的值，结果将是 `10`。`get` 方法返回一个 `Option<&V>`；如果哈希映射中没有该键对应的值，`get` 将返回 `None`。该程序通过调用 `copied` 来获得 `Option<i32>`（而不是 `Option<&i32>`），然后调用 `unwrap_or` 在 `scores` 中没有该键条目时将 `score` 设置为 0，以此方式处理 `Option`。

我们可以像处理向量一样，使用 `for` 循环以类似的方式遍历哈希映射中的每个键值对：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

这段代码将以任意顺序打印每个键值对：

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### 哈希映射与所有权管理

对于实现了 `Copy` trait 的类型（如 `i32`），值会被复制到哈希映射中。对于拥有所有权的值（如 `String`），值将被移动，哈希映射将成为这些值的所有者，如示例 8-22 所示。

<Listing number="8-22" caption="展示键和值在插入后由哈希映射拥有">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

在变量 `field_name` 和 `field_value` 通过调用 `insert` 被移动到哈希映射中之后，我们无法再使用它们。

如果我们向哈希映射插入值的引用，这些值不会被移动到哈希映射中。引用所指向的值必须至少在哈希映射有效的整个期间内都有效。我们将在第 10 章的[“使用生命周期（Lifetime）验证引用”][validating-references-with-lifetimes]<!-- ignore -->中进一步讨论这些问题。

### 更新哈希映射

尽管键值对的数量是可以增长的，但每个唯一的键一次只能关联一个值（但反之则不然：例如，Blue 队和 Yellow 队都可以在 `scores` 哈希映射中存储值 `10`）。

当你想要更改哈希映射中的数据时，你必须决定如何处理键已有关联值的情况。你可以用新值替换旧值，完全忽略旧值。你可以保留旧值而忽略新值，仅在键*尚未*有关联值时添加新值。或者你可以合并旧值和新值。让我们看看如何实现每种情况！

#### 覆盖一个值

如果我们向哈希映射插入一个键和一个值，然后又用不同的值插入同一个键，则该键关联的值将被替换。尽管示例 8-23 中的代码调用了两次 `insert`，但哈希映射只会包含一个键值对，因为我们两次插入的都是 Blue 队键的值。

<Listing number="8-23" caption="替换特定键存储的值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

这段代码将打印 `{"Blue": 25}`。原来的值 `10` 已被覆盖。

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### 仅在键不存在时添加键和值

通常需要检查哈希映射中是否已存在某个特定键及其值，然后执行以下操作：如果该键存在于哈希映射中，则保留现有值不变；如果该键不存在，则插入它及其值。

哈希映射有一个特殊的 API 用于此，称为 `entry`，它将要检查的键作为参数。`entry` 方法的返回值是一个名为 `Entry` 的枚举，它表示一个可能存在也可能不存在的值。假设我们想检查 Yellow 队键是否已有关联值。如果没有，我们想插入值 `50`；对于 Blue 队也是如此。使用 `entry` API，代码如示例 8-24 所示。

<Listing number="8-24" caption="使用 `entry` 方法仅在键尚未关联值时插入">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

`Entry` 上的 `or_insert` 方法被定义为：如果键存在，则返回对应 `Entry` 键的值的可变引用；如果不存在，则将参数作为该键的新值插入，并返回新值的可变引用。这种技术比我们自己编写逻辑要简洁得多，此外，它与借用检查器的配合也更好。

运行示例 8-24 中的代码将打印 `{"Yellow": 50, "Blue": 10}`。第一次调用 `entry` 将为 Yellow 队插入键与值 `50`，因为 Yellow 队还没有值。第二次调用 `entry` 不会更改哈希映射，因为 Blue 队已经有值 `10`。

#### 基于旧值更新值

哈希映射的另一个常见用例是查找键的值，然后根据旧值更新它。例如，示例 8-25 显示了统计每个单词在文本中出现次数的代码。我们使用一个以单词为键的哈希映射，并递增值以跟踪我们看到该单词的次数。如果我们是第一次看到某个单词，我们会先插入值 `0`。

<Listing number="8-25" caption="使用存储单词和计数的哈希映射统计单词出现次数">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

这段代码将打印 `{"world": 2, "hello": 1, "wonderful": 1}`。你可能会看到相同的键值对以不同的顺序打印：回想一下[“访问哈希映射中的值”][access]<!-- ignore -->，遍历哈希映射是以任意顺序进行的。

`split_whitespace` 方法返回一个迭代器，遍历 `text` 中以空白分隔的子切片。`or_insert` 方法返回指定键的值的可变引用（`&mut V`）。在这里，我们将该可变引用存储在 `count` 变量中，因此为了给该值赋值，我们必须首先使用星号（`*`）解引用 `count`。可变引用在 `for` 循环结束时超出作用域，因此所有这些更改都是安全的，并且为借用规则所允许。

### 哈希函数

默认情况下，`HashMap` 使用一种名为 *SipHash* 的哈希函数，它可以抵抗涉及哈希表的拒绝服务（DoS）攻击[^siphash]<!-- ignore -->。这不是可用的最快的哈希算法，但为了更好的安全性而降低性能，这种权衡是值得的。如果你对你的代码进行了性能分析，发现默认哈希函数对你的目的来说太慢，你可以通过指定一个不同的 hasher（哈希器）来切换到另一个函数。*hasher* 是一个实现了 `BuildHasher` trait 的类型。我们将在[第 10 章][traits]<!-- ignore -->讨论 trait 以及如何实现它们。你不一定需要从头开始实现自己的 hasher；[crates.io](https://crates.io/)<!-- ignore --> 上有其他 Rust 用户共享的库，它们提供了实现许多常见哈希算法的 hasher。

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## 总结

当你需要存储、访问和修改数据时，向量（vector）、字符串（string）和哈希映射（hash map）将在程序中提供所需的大量功能。以下是一些你现在应该能够解决的练习：

1.  给定一个整数列表，使用向量并返回该列表的中位数（排序后中间位置的值）和众数（出现频率最高的值；哈希映射在这里很有帮助）。
1.  将字符串转换为 Pig Latin（儿童黑话）。每个单词的第一个辅音（consonant）移到单词末尾，并加上 _ay_，因此 _first_ 变成 _irst-fay_。以元音（vowel）开头的单词在末尾加上 _hay_（_apple_ 变成 _apple-hay_）。请记住 UTF-8 编码的细节！
1.  使用哈希映射和向量，创建一个文本界面，允许用户向公司的某个部门添加员工姓名；例如，"Add Sally to Engineering"或"Add Amir to Sales"。然后，让用户检索某个部门的所有人员列表，或按部门检索公司的所有人员列表，并按键排序。

标准库 API 文档描述了向量、字符串和哈希映射具有的方法，这些方法对这些练习会有帮助！

我们正在进入更复杂的程序，其中操作可能会失败，因此现在是讨论错误处理的好时机。我们接下来将进行讨论！

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
