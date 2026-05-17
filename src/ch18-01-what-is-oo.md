## 面向对象语言的特点

在编程社区中，对于一门语言必须具备哪些特性才能被视为面向对象，并没有共识。Rust 受到了许多编程范式的影响，包括 OOP；例如，我们在第 13 章中探讨了来自函数式编程的特性。可以说，OOP 语言共享了一些共同的特性——即对象（objects）、封装（encapsulation）和继承（inheritance）。让我们来看看这些特性各自意味着什么，以及 Rust 是否支持它们。

### 对象包含数据和行为

由 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 合著的《设计模式：可复用面向对象软件的基础》（*Design Patterns: Elements of Reusable Object-Oriented Software*，Addison-Wesley，1994 年），俗称"四人组"（*The Gang of Four*）一书，是一本面向对象设计模式的目录。它这样定义 OOP：

> 面向对象的程序由对象组成。一个**对象（object）**封装了数据以及操作这些数据的流程（procedures）。这些流程通常被称为**方法（methods）**或**操作（operations）**。

根据这个定义，Rust 是面向对象的：结构体（struct）和枚举（enum）包含数据，而 `impl` 块为结构体和枚举提供方法。尽管带有方法的结构体和枚举并不*叫做*对象，但根据四人组对对象的定义，它们提供了相同的功能。

### 封装隐藏了实现细节

另一个与 OOP 相关的常见方面是*封装（encapsulation）*的概念，这意味着对象的实现细节对于使用该对象的代码来说是不可访问的。因此，与对象交互的唯一方式是通过其公有 API（public API）；使用该对象的代码不应该能够直接访问对象的内部并更改数据或行为。这使得程序员可以更改和重构对象的内部实现，而无需修改使用该对象的代码。

我们在第 7 章中讨论了如何控制封装：我们可以使用 `pub` 关键字来决定代码中的哪些模块、类型、函数和方法应该是公有的，而其他所有内容默认都是私有的。例如，我们可以定义一个 `AveragedCollection` 结构体，它有一个包含 `i32` 值向量的字段。该结构体还可以有一个字段用于保存这些值的平均值，这意味着平均值不需要在每次有人需要时都重新计算。换句话说，`AveragedCollection` 会为我们缓存计算好的平均值。清单 18-1 展示了 `AveragedCollection` 结构体的定义。

<Listing number="18-1" file-name="src/lib.rs" caption="一个 `AveragedCollection` 结构体，维护一个整数列表以及集合中元素的平均值">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

该结构体被标记为 `pub`，以便其他代码可以使用它，但结构体内部的字段保持私有。在这种情况下，这一点很重要，因为我们希望确保每当从列表中添加或删除值时，平均值也会被更新。我们通过在结构体上实现 `add`、`remove` 和 `average` 方法来实现这一点，如清单 18-2 所示。

<Listing number="18-2" file-name="src/lib.rs" caption="在 `AveragedCollection` 上实现公有方法 `add`、`remove` 和 `average`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

公有方法 `add`、`remove` 和 `average` 是访问或修改 `AveragedCollection` 实例中数据的唯一方式。当使用 `add` 方法向 `list` 添加一个元素，或使用 `remove` 方法移除一个元素时，这两个方法的实现都会调用私有的 `update_average` 方法来处理 `average` 字段的更新。

我们将 `list` 和 `average` 字段设为私有，这样外部代码就无法直接向 `list` 字段添加或移除元素；否则，当 `list` 发生变化时，`average` 字段可能会不同步。`average` 方法返回 `average` 字段的值，允许外部代码读取平均值但不能修改它。

由于我们封装了 `AveragedCollection` 结构体的实现细节，我们可以在未来轻松地更改某些方面，例如数据结构。举例来说，我们可以使用 `HashSet<i32>` 而不是 `Vec<i32>` 作为 `list` 字段。只要 `add`、`remove` 和 `average` 这些公有方法的签名保持不变，使用 `AveragedCollection` 的代码就不需要更改。如果我们把 `list` 改为公有，情况就不一定了：`HashSet<i32>` 和 `Vec<i32>` 有不同的添加和移除元素的方法，因此如果外部代码直接修改 `list`，很可能也需要更改。

如果封装被认为是面向对象语言的必要条件，那么 Rust 满足这个要求。在代码的不同部分选择是否使用 `pub`，使得实现细节的封装成为可能。

### 继承作为类型系统和代码共享

*继承（Inheritance）*是一种机制，通过这种机制，一个对象可以继承另一个对象定义中的元素，从而无需重新定义即可获得父对象的数据和行为。

如果一门语言必须支持继承才能被认为是面向对象的，那么 Rust 不是这样的语言。没有一种方式可以在不使用宏的情况下定义继承父结构体字段和方法实现的结构体。

然而，如果你习惯于在编程工具箱中使用继承，你可以根据当初使用继承的原因，在 Rust 中找到其他解决方案。

选择继承主要有两个原因。一是代码复用：你可以为一种类型实现特定的行为，而继承使你能够为另一种类型复用该实现。在 Rust 中，你可以通过默认 trait 方法实现（default trait method implementations）以有限的方式做到这一点，正如你在清单 10-14 中看到的那样，我们为 `Summary` trait 上的 `summarize` 方法添加了默认实现。任何实现了 `Summary` trait 的类型都会自动拥有 `summarize` 方法，无需再编写任何额外代码。这类似于父类拥有一个方法的实现，而继承的子类也拥有该方法的实现。我们也可以在实现 `Summary` trait 时覆盖 `summarize` 方法的默认实现，这类似于子类覆盖从父类继承的方法的实现。

使用继承的另一个原因与类型系统有关：使子类型能够在与父类型相同的地方使用。这也被称为*多态（polymorphism）*，这意味着如果多个对象在运行时共享某些特征，它们可以相互替换。

> ### 多态
>
> 对于许多人来说，多态是继承的同义词。但实际上它是一个更通用的概念，指的是可以处理多种类型数据的代码。对于继承而言，这些类型通常是子类（subclasses）。
>
> Rust 则使用泛型（generics）来抽象不同的可能类型，并使用 trait 约束（trait bounds）来限制这些类型必须提供什么。这有时被称为*有界参数化多态（bounded parametric polymorphism）*。

Rust 通过不提供继承选择了另一套权衡方案。继承常常存在共享比所需更多代码的风险。子类不一定应该共享其父类的所有特征，但在继承中却会如此。这可能会使程序的设计灵活性降低。它还可能导致在子类上调用没有意义或引起错误的方法，因为这些方法不适用于子类。此外，有些语言只允许*单一继承（single inheritance）*（即子类只能从一个类继承），进一步限制了程序设计的灵活性。

由于这些原因，Rust 采用了不同的方法，使用 trait 对象（trait objects）而不是继承来实现运行时的多态。让我们来看看 trait 对象是如何工作的。
