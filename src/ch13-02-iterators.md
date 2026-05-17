## 使用迭代器（Iterator）处理一系列项

迭代器模式（iterator pattern）允许你对一系列项依次执行某些任务。迭代器负责遍历每个项的逻辑以及确定序列何时结束的逻辑。当你使用迭代器时，你不需要自己重新实现该逻辑。

在 Rust 中，迭代器是*惰性的（lazy）*，这意味着在你调用使用迭代器的方法之前，它们不会产生效果。例如，示例 13-10 中的代码通过调用 `Vec<T>` 上定义的 `iter` 方法，在向量 `v1` 中的项上创建了一个迭代器。这段代码本身并不做任何有用的事情。

<Listing number="13-10" file-name="src/main.rs" caption="创建一个迭代器">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

该迭代器存储在 `v1_iter` 变量中。一旦我们创建了一个迭代器，我们可以以多种方式使用它。在示例 3-5 中，我们使用 `for` 循环遍历了一个数组，对其每个项执行了一些代码。在底层，这隐式地创建并消耗了一个迭代器，但直到现在我们才详细解释它是如何工作的。

在示例 13-11 的例子中，我们将迭代器的创建与在 `for` 循环中的使用分离开来。当使用 `v1_iter` 中的迭代器调用 `for` 循环时，迭代器中的每个元素都在循环的一次迭代中被使用，循环会打印出每个值。

<Listing number="13-11" file-name="src/main.rs" caption="在 `for` 循环中使用迭代器">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

在那些标准库没有提供迭代器的语言中，你可能会通过从索引 0 开始一个变量、使用该变量索引向量以获取值、并在循环中递增变量值直到达到向量中项的总数来编写相同的功能。

迭代器为你处理所有逻辑，减少了你可能搞砸的重复代码。迭代器让你更灵活地将相同的逻辑用于许多不同类型的序列，而不仅仅是像向量那样可以索引的数据结构。让我们来看看迭代器是如何做到这一点的。

### `Iterator` Trait 和 `next` 方法

所有迭代器都实现了 `Iterator` trait，该 trait 在标准库中定义。该 trait 的定义如下：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // 带有默认实现的方法已省略
}
```

注意，此定义使用了一些新语法：`type Item` 和 `Self::Item`，它们正在定义与此 trait 的关联类型（associated type）。我们将在第 20 章深入讨论关联类型。现在，你只需要知道这段代码表示实现 `Iterator` trait 要求你还定义一个 `Item` 类型，并且此 `Item` 类型用于 `next` 方法的返回类型。换句话说，`Item` 类型将是迭代器返回的类型。

`Iterator` trait 只要求实现者定义一个方法：`next` 方法，该方法一次返回迭代器的一个项，包裹在 `Some` 中，当迭代结束时返回 `None`。

我们可以直接在迭代器上调用 `next` 方法；示例 13-12 演示了从向量创建的迭代器上重复调用 `next` 会返回什么值。

<Listing number="13-12" file-name="src/lib.rs" caption="在迭代器上调用 `next` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

注意，我们需要将 `v1_iter` 声明为可变的：在迭代器上调用 `next` 方法会改变迭代器用于跟踪其在序列中位置的内部状态。换句话说，这段代码*消耗（consumes）*或使用了迭代器。每次对 `next` 的调用都会从迭代器中消耗一个项。当我们使用 `for` 循环时，我们不需要使 `v1_iter` 可变，因为循环获取了 `v1_iter` 的所有权并在幕后使其可变。

还要注意，我们从调用 `next` 获得的值是对向量中值的不可变引用。`iter` 方法生成一个不可变引用的迭代器。如果我们想创建一个获取 `v1` 所有权并返回拥有的值的迭代器，我们可以调用 `into_iter` 而不是 `iter`。类似地，如果我们想遍历可变引用，我们可以调用 `iter_mut` 而不是 `iter`。

### 消耗迭代器的方法

`Iterator` trait 有许多不同的方法，标准库提供了默认实现；你可以通过查看标准库 API 文档中 `Iterator` trait 来了解这些方法。其中一些方法在其定义中调用了 `next` 方法，这就是为什么在实现 `Iterator` trait 时要求你实现 `next` 方法。

调用 `next` 的方法被称为*消耗适配器（consuming adapters）*，因为调用它们会消耗掉迭代器。一个例子是 `sum` 方法，它获取迭代器的所有权，并通过重复调用 `next` 来迭代各项，从而消耗迭代器。在迭代过程中，它将每个项添加到一个运行总数中，并在迭代完成时返回总数。示例 13-13 有一个测试，说明了 `sum` 方法的用法。

<Listing number="13-13" file-name="src/lib.rs" caption="调用 `sum` 方法以获取迭代器中所有项的总和">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

在调用 `sum` 之后，我们不允许再使用 `v1_iter`，因为 `sum` 获取了我们调用它的迭代器的所有权。

### 产生其他迭代器的方法

*迭代器适配器（Iterator adapters）*是在 `Iterator` trait 上定义的方法，它们不消耗迭代器。相反，它们通过改变原始迭代器的某些方面来产生不同的迭代器。

示例 13-14 显示了一个调用迭代器适配器方法 `map` 的例子，`map` 接受一个闭包，该闭包在遍历每个项时被调用。`map` 方法返回一个新的迭代器，它产生修改后的项。这里的闭包创建了一个新迭代器，其中向量中的每个项都将递增 1。

<Listing number="13-14" file-name="src/main.rs" caption="调用迭代器适配器 `map` 以创建新迭代器">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

然而，这段代码会产生一条警告：

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

示例 13-14 中的代码没有做任何事情；我们指定的闭包从未被调用。警告提醒我们原因：迭代器适配器是惰性的，我们需要在这里消耗迭代器。

要修复此警告并消耗迭代器，我们将使用 `collect` 方法，我们在示例 12-1 中与 `env::args` 一起使用过。此方法消耗迭代器并将结果值收集到集合数据类型中。

在示例 13-15 中，我们将迭代从调用 `map` 返回的迭代器的结果收集到一个向量中。这个向量最终将包含原始向量中的每个项，每个项递增 1。

<Listing number="13-15" file-name="src/main.rs" caption="调用 `map` 方法创建新迭代器，然后调用 `collect` 方法消耗新迭代器并创建向量">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

因为 `map` 接受一个闭包，我们可以指定我们想对每个项执行的任何操作。这是一个很好的例子，说明了闭包如何让你定制某些行为，同时重用 `Iterator` trait 提供的迭代行为。

你可以链式调用多个迭代器适配器，以可读的方式执行复杂操作。但由于所有迭代器都是惰性的，你必须调用其中一个消耗适配器方法才能从迭代器适配器的调用中获取结果。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### 捕获其环境的闭包

许多迭代器适配器接受闭包作为参数，通常我们指定为迭代器适配器参数的闭包将是捕获其环境的闭包。

在此示例中，我们将使用接受闭包的 `filter` 方法。闭包从迭代器中获取一个项并返回一个 `bool`。如果闭包返回 `true`，该值将包含在 `filter` 产生的迭代中。如果闭包返回 `false`，该值将不会包含在内。

在示例 13-16 中，我们使用 `filter` 和一个捕获其环境中 `shoe_size` 变量的闭包来遍历 `Shoe` 结构体实例的集合。它将只返回指定尺寸的鞋子。

<Listing number="13-16" file-name="src/lib.rs" caption="使用 `filter` 方法以及捕获 `shoe_size` 的闭包">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` 函数获取鞋子向量的所有权和一个鞋码作为参数。它返回一个仅包含指定尺寸鞋子的向量。

在 `shoes_in_size` 的函数体中，我们调用 `into_iter` 来创建一个获取向量所有权的迭代器。然后，我们调用 `filter` 将该迭代器适配为仅包含闭包返回 `true` 的元素的新迭代器。

闭包从环境中捕获 `shoe_size` 参数，并将该值与每只鞋的尺寸进行比较，只保留指定尺寸的鞋子。最后，调用 `collect` 将由适配后的迭代器返回的值收集到一个向量中，该向量由函数返回。

测试表明，当我们调用 `shoes_in_size` 时，我们只会得到与我们指定的值相同尺寸的鞋子。
