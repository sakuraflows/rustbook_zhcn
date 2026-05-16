## 使用向量（Vector）存储值列表

我们要介绍的第一个集合类型是 `Vec<T>`，也称为向量（vector）。向量允许你在一个数据结构中存储多个值，这些值在内存中连续排列。向量只能存储相同类型的值。当你有一列项目（例如文件中的文本行或购物车中商品的价格）时，它们非常有用。

### 创建新向量

要创建一个新的空向量，我们调用 `Vec::new` 函数，如示例 8-1 所示。

<Listing number="8-1" caption="创建一个新的空向量，用于存储 `i32` 类型的值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

注意，我们在这里添加了类型注解。因为我们没有向这个向量插入任何值，Rust 不知道我们打算存储什么类型的元素。这是一个重要的点。向量是使用泛型（generics）实现的；我们将在第 10 章介绍如何在自定义类型中使用泛型。现在，只需要知道标准库提供的 `Vec<T>` 类型可以持有任何类型。当我们创建一个持有特定类型的向量时，可以在尖括号中指定该类型。在示例 8-1 中，我们告诉 Rust `v` 中的 `Vec<T>` 将持有 `i32` 类型的元素。

更常见的情况是，你会使用初始值创建一个 `Vec<T>`，Rust 会推断出你想要存储的值类型，因此你很少需要做这种类型注解。Rust 方便地提供了 `vec!` 宏，它会创建一个包含你给定值的新向量。示例 8-2 创建了一个新的 `Vec<i32>`，其中包含值 `1`、`2` 和 `3`。整数类型是 `i32`，因为它是默认的整数类型，正如我们在第 3 章的[“数据类型”][data-types]<!-- ignore -->部分所讨论的那样。

<Listing number="8-2" caption="创建一个包含值的新向量">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

因为我们给出了初始的 `i32` 值，Rust 可以推断出 `v` 的类型是 `Vec<i32>`，因此类型注解不是必需的。接下来，我们将看看如何修改一个向量。

### 更新向量

要创建一个向量然后向其中添加元素，我们可以使用 `push` 方法，如示例 8-3 所示。

<Listing number="8-3" caption="使用 `push` 方法向向量添加值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

与任何变量一样，如果我们想要更改它的值，需要使用 `mut` 关键字使其可变，如第 3 章所述。我们放入其中的数字都是 `i32` 类型，Rust 从数据中推断出这一点，因此我们不需要 `Vec<i32>` 注解。

### 读取向量的元素

有两种方法可以引用向量中存储的值：通过索引（indexing）或使用 `get` 方法。在以下示例中，我们注释了这些函数返回的值的类型，以便更清晰。

示例 8-4 展示了访问向量中值的两种方法：索引语法和 `get` 方法。

<Listing number="8-4" caption="使用索引语法和 `get` 方法访问向量中的元素">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

这里有几个细节需要注意。我们使用索引值 `2` 来获取第三个元素，因为向量是按数字索引的，从零开始。使用 `&` 和 `[]` 会给我们一个指向索引处元素的引用。当我们使用 `get` 方法并将索引作为参数传入时，会得到一个 `Option<&T>`，我们可以与 `match` 配合使用。

Rust 提供了这两种引用元素的方式，以便你可以选择程序在尝试使用超出现有元素范围的索引值时的行为。举个例子，让我们看看当有一个包含五个元素的向量，然后尝试使用每种技术访问索引 100 处的元素时会发生什么，如示例 8-5 所示。

<Listing number="8-5" caption="尝试访问包含五个元素的向量中索引 100 处的元素">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

当我们运行这段代码时，第一个 `[]` 方法会导致程序 panic，因为它引用了一个不存在的元素。当你希望在尝试访问向量末尾之后的元素时程序崩溃，这种方法最合适。

当向 `get` 方法传递一个超出向量范围的索引时，它返回 `None` 而不会 panic。如果在正常情况下的某些操作偶尔会访问超出向量范围的元素，你会使用这种方法。然后你的代码将包含处理 `Some(&element)` 或 `None` 的逻辑，如第 6 章所述。例如，索引可能来自用户输入的数字。如果他们不小心输入了一个太大的数字，程序得到了 `None` 值，你可以告诉用户当前向量中有多少项，并给他们另一次输入有效值的机会。这比因为一个输入错误就让程序崩溃要更加用户友好！

当程序拥有有效引用时，借用检查器（borrow checker）会强制执行所有权和借用规则（涵盖在第 4 章），以确保该引用以及对向量内容的任何其他引用保持有效。回忆一下那条规则——你不能在同一个作用域中同时拥有可变引用和不可变引用。这条规则适用于示例 8-6，其中我们持有一个指向向量第一个元素的不可变引用，并尝试在末尾添加一个元素。如果我们后来还尝试在该函数中引用那个元素，这个程序将无法运行。

<Listing number="8-6" caption="尝试在持有元素引用时向向量添加元素">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

编译这段代码将导致以下错误：

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

示例 8-6 中的代码看起来似乎是可行的：为什么对第一个元素的引用会关心向量末尾的变化？这个错误是由于向量的工作方式造成的：因为向量将值在内存中连续排列，所以如果在向量当前存储的位置没有足够的空间把所有元素连续放置，那么向末尾添加一个新元素可能需要分配新的内存并将旧元素复制到新空间。在这种情况下，对第一个元素的引用就会指向已释放的内存。借用规则防止程序陷入这种境地。

> 注意：有关 `Vec<T>` 类型实现细节的更多信息，请参阅[“Rustonomicon”][nomicon]。

### 遍历向量中的值

要依次访问向量中的每个元素，我们可以遍历所有元素，而不是逐次使用索引访问。示例 8-7 展示了如何使用 `for` 循环获取 `i32` 值向量中每个元素的不可变引用并打印它们。

<Listing number="8-7" caption="通过 `for` 循环遍历向量中的元素来打印每个元素">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

我们还可以遍历可变向量中每个元素的可变引用，以便对所有元素进行更改。示例 8-8 中的 `for` 循环将为每个元素加上 `50`。

<Listing number="8-8" caption="遍历向量中元素的可变引用">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

要更改可变引用所指向的值，我们必须使用 `*` 解引用运算符（dereference operator）来获取 `i` 中的值，然后才能使用 `+=` 运算符。我们将在第 15 章的[“跟随引用到值”][deref]<!-- ignore -->部分进一步讨论解引用运算符。

由于借用检查器的规则，遍历向量（无论是不可变地还是可变地）是安全的。如果我们在示例 8-7 和示例 8-8 的 `for` 循环体中尝试插入或删除元素，我们会得到与示例 8-6 类似的编译器错误。 `for` 循环持有的对向量的引用会阻止对整个向量进行同时修改。

### 使用枚举存储多种类型

向量只能存储相同类型的值。这可能不太方便；确实有些用例需要存储不同类型值的列表。幸运的是，枚举的变体（variant）是在同一枚举类型下定义的，所以当我们需要一种类型来表示不同类型的元素时，我们可以定义并使用一个枚举！

例如，假设我们想要从电子表格中的一行获取值，该行中有些列包含整数，有些包含浮点数，还有些包含字符串。我们可以定义一个枚举，其变体将持有不同的值类型，并且所有枚举变体都将被视为相同类型：即该枚举的类型。然后，我们可以创建一个持有该枚举的向量，从而最终持有不同的类型。我们在示例 8-9 中演示了这一点。

<Listing number="8-9" caption="定义一个枚举以在一个向量中存储不同类型的值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust 需要在编译时知道向量中将包含哪些类型，以便它确切地知道需要在堆上分配多少内存来存储每个元素。我们还必须明确说明该向量中允许哪些类型。如果 Rust 允许一个向量持有任意类型，那么其中一种或多种类型可能会在对向量元素执行的操作中引发错误。使用枚举加 `match` 表达式意味着 Rust 会在编译时确保每个可能的情况都得到处理，如第 6 章所述。

如果你在编译时不知道程序运行时将存储在向量中的类型的完整集合，枚举技术就不起作用。相反，你可以使用 trait 对象（trait object），我们将在第 18 章中介绍。

现在我们已经讨论了一些使用向量的最常用方法，请务必查阅[API 文档][vec-api]<!-- ignore -->以了解标准库在 `Vec<T>` 上定义的所有有用的方法。例如，除了 `push` 之外，还有一个 `pop` 方法用于移除并返回最后一个元素。

### 释放向量即释放其元素

与任何其他 `struct` 一样，向量在其超出作用域时被释放，如示例 8-10 所示。

<Listing number="8-10" caption="展示向量及其元素在哪里被释放">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

当向量被释放时，其所有内容也被释放，这意味着它持有的整数将被清理掉。借用检查器确保对向量内容的任何引用只在向量本身有效时使用。

接下来让我们讨论下一种集合类型：`String`！

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
