<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## 闭包（Closures）

Rust 的闭包（closures）是可以保存在变量中或作为参数传递给其他函数的匿名函数。你可以在一个地方创建闭包，然后在不同的上下文中调用它以求值。与函数不同，闭包可以从其定义的作用域中捕获值。我们将演示这些闭包特性如何实现代码重用和行为定制。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### 捕获环境（Capturing the Environment）

我们首先考察如何使用闭包从其定义的环境中捕获值以供以后使用。场景如下：我们的 T 恤公司时不时将一件独家限量版衬衫赠送给邮件列表中的某人作为促销活动。邮件列表中的人可以选择将他们的最喜欢的颜色添加到个人资料中。如果被选中获得免费衬衫的人设置了最喜欢的颜色，他们将获得该颜色的衬衫。如果该人没有指定最喜欢的颜色，他们将获得公司目前库存最多的颜色。

有很多方法可以实现这一点。对于此示例，我们将使用一个名为 `ShirtColor` 的枚举，它有 `Red` 和 `Blue` 两个变体（为简单起见限制了可用颜色数量）。我们使用一个 `Inventory` 结构体来表示公司的库存，该结构体有一个名为 `shirts` 的字段，包含一个 `Vec<ShirtColor>`，表示当前库存的衬衫颜色。在 `Inventory` 上定义的 `giveaway` 方法获取免费衬衫赢家的可选衬衫颜色偏好，并返回此人将获得的衬衫颜色。此设置在示例 13-1 中展示。

<Listing number="13-1" file-name="src/main.rs" caption="衬衫公司赠送场景">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`main` 中定义的 `store` 还剩下两件蓝衬衫和一件红衬衫可供此限量版促销分发。我们为一位偏好红衬衫的用户和一位没有偏好的用户调用 `giveaway` 方法。

同样，这段代码可以用多种方式实现，此处为了专注于闭包，我们坚持使用你已经学过的概念，除了 `giveaway` 方法体中使用了一个闭包。在 `giveaway` 方法中，我们将用户偏好作为类型 `Option<ShirtColor>` 的参数获取，并在 `user_preference` 上调用 `unwrap_or_else` 方法。`Option<T>` 上的 [`unwrap_or_else` 方法][unwrap-or-else]<!-- ignore -->由标准库定义。它接受一个参数：一个不带任何参数并返回 `T` 类型值的闭包（与存储在 `Option<T>` 的 `Some` 变体中的类型相同，在本例中是 `ShirtColor`）。如果 `Option<T>` 是 `Some` 变体，`unwrap_or_else` 返回 `Some` 内部的值。如果 `Option<T>` 是 `None` 变体，`unwrap_or_else` 调用闭包并返回闭包返回的值。

我们将闭包表达式 `|| self.most_stocked()` 指定为 `unwrap_or_else` 的参数。这是一个自身不带参数的闭包（如果闭包有参数，它们会出现在两个竖线之间）。闭包体调用 `self.most_stocked()`。我们在此定义闭包，而 `unwrap_or_else` 的实现将在需要结果时稍后求值闭包。

运行此代码会打印以下内容：

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

这里一个有趣的方面是我们传递了一个闭包，它在当前的 `Inventory` 实例上调用 `self.most_stocked()`。标准库不需要知道我们定义的 `Inventory` 或 `ShirtColor` 类型，也不需要知道我们在此场景中想要使用的逻辑。闭包捕获了对 `self` `Inventory` 实例的不可变引用，并将其与我们指定的代码一起传递给 `unwrap_or_else` 方法。而函数则无法以这种方式捕获它们的环境。

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### 推断和标注闭包类型

函数和闭包之间还有更多区别。闭包通常不需要你像 `fn` 函数那样标注参数类型或返回值类型。类型注解在函数上是必需的，因为类型是暴露给用户的显式接口的一部分。严格定义这个接口对于确保每个人都同意函数使用和返回什么类型的值很重要。而闭包则不会在这种暴露的接口中使用：它们存储在变量中，在不命名也不暴露给库用户的情况下使用。

闭包通常很短，并且仅与狭窄的上下文相关，而不是任意场景。在这些有限的上下文中，编译器可以推断参数类型和返回类型，类似于它推断大多数变量类型的方式（也有极少数情况下编译器也需要闭包类型注解）。

与变量一样，如果我们想增加明确性和清晰度，可以添加类型注解，代价是比严格必要的更冗长。为闭包标注类型看起来像示例 13-2 中所示的定义。在此示例中，我们定义了一个闭包并将其存储在变量中，而不是像在示例 13-1 中那样将闭包定义在将其作为参数传递的位置。

<Listing number="13-2" file-name="src/main.rs" caption="在闭包中添加参数和返回值类型的可选类型注解">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

添加类型注解后，闭包的语法看起来更类似于函数的语法。为了比较，我们定义了一个将其参数加 1 的函数和一个具有相同行为的闭包。我们添加了一些空格来对齐相关部分。这说明闭包语法与函数语法相似，除了使用竖线和可选的一些语法：

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

第一行显示了一个函数定义，第二行显示了一个完整标注的闭包定义。在第三行中，我们移除了闭包定义中的类型注解。在第四行中，我们移除了花括号，这是可选的，因为闭包体只有一个表达式。这些都是有效的定义，在调用时会产生相同的行为。`add_one_v3` 和 `add_one_v4` 行要求闭包被求值才能编译，因为类型将从它们的使用中推断出来。这类似于 `let v = Vec::new();` 需要类型注解或某些类型的值插入到 `Vec` 中，Rust 才能推断出类型。

对于闭包定义，编译器将为每个参数和返回值推断一个具体类型。例如，示例 13-3 显示了一个简短的闭包定义，它只是返回它收到的参数作为值。除了此示例的目的之外，这个闭包不是很有用。注意，我们没有在定义中添加任何类型注解。因为没有类型注解，我们可以用任何类型调用这个闭包，我们第一次用 `String` 这样做。如果我们随后尝试用整数调用 `example_closure`，我们将得到一个错误。

<Listing number="13-3" file-name="src/main.rs" caption="尝试用两种不同的类型调用其类型被推断的闭包">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

编译器给出这个错误：

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

第一次用 `String` 值调用 `example_closure` 时，编译器推断 `x` 的类型和闭包的返回类型是 `String`。这些类型随后被锁定在 `example_closure` 的闭包中，当我们下次尝试用不同的类型使用同一个闭包时，就会得到类型错误。

### 捕获引用或移动所有权

闭包可以通过三种方式从其环境中捕获值，这直接映射到函数接受参数的三种方式：不可变借用、可变借用和获取所有权。闭包将根据函数体对捕获值执行的操作来决定使用哪种方式。

在示例 13-4 中，我们定义了一个闭包，它捕获对名为 `list` 的向量的不可变引用，因为它只需要一个不可变引用来打印值。

<Listing number="13-4" file-name="src/main.rs" caption="定义和调用捕获不可变引用的闭包">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

这个例子还说明变量可以绑定到闭包定义，我们以后可以通过使用变量名和括号来调用闭包，就像变量名是函数名一样。

因为我们可以同时对 `list` 有多个不可变引用，所以在闭包定义之前、闭包定义之后但闭包被调用之前以及闭包被调用之后，`list` 仍然可从代码中访问。这段代码编译、运行并打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

接下来，在示例 13-5 中，我们更改了闭包体，使其向 `list` 向量添加一个元素。闭包现在捕获了一个可变引用。

<Listing number="13-5" file-name="src/main.rs" caption="定义和调用捕获可变引用的闭包">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

这段代码编译、运行并打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

注意，在 `borrows_mutably` 闭包的定义和调用之间不再有 `println!`：当 `borrows_mutably` 被定义时，它捕获了对 `list` 的可变引用。我们在闭包被调用后不再使用该闭包，因此可变借用结束。在闭包定义和闭包调用之间，不允许进行不可变借用打印，因为在存在可变借用时不允许其他借用。尝试在那里添加一个 `println!`，看看你会得到什么错误消息！

如果你想强制闭包获取它在其环境中使用的值的所有权，即使闭包体并非严格需要所有权，你也可以在参数列表前使用 `move` 关键字。

这种技术主要在将闭包传递给新线程以移动数据使其由新线程所有时有用。我们将在第 16 章讨论并发时详细讨论线程以及为什么要使用它们，但现在，让我们简要地探索使用需要 `move` 关键字的闭包来生成新线程。示例 13-6 显示了示例 13-4 的修改版本，在新线程中而不是在主线程中打印向量。

<Listing number="13-6" file-name="src/main.rs" caption="使用 `move` 强制线程的闭包获取 `list` 的所有权">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

我们生成了一个新线程，将闭包作为参数传递给线程运行。闭包体打印出列表。在示例 13-4 中，闭包仅使用不可变引用捕获了 `list`，因为这是打印它所需的最少访问权限。在此示例中，尽管闭包体仍然只需要不可变引用，但我们需要通过在闭包定义的开头放置 `move` 关键字来指定 `list` 应被移入闭包。如果主线程在调用新线程上的 `join` 之前执行了更多操作，新线程可能在其他主线程部分完成之前完成，或者主线程可能先完成。如果主线程保持 `list` 的所有权但在新线程结束前结束并释放了 `list`，则线程中的不可变引用将无效。因此，编译器要求将 `list` 移入给新线程的闭包中，以便引用有效。尝试移除 `move` 关键字，或者在定义闭包后在主线程中使用 `list`，以查看你会得到什么编译器错误！

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### 将被捕获的值移出闭包

一旦闭包捕获了一个引用或从定义闭包的环境中获取了一个值的所有权（从而影响了什么被移*入*闭包），闭包体中的代码就定义了当闭包稍后被求值时引用或值会发生什么（从而影响了什么被移*出*闭包）。

闭包体可以执行以下任何操作：将捕获的值移出闭包、改变捕获的值、既不移动也不改变该值，或者一开始就不从环境中捕获任何东西。

闭包从环境捕获和处理值的方式影响闭包实现了哪些 trait，而 trait 是函数和结构体指定它们可以使用哪些闭包的方式。闭包将根据闭包体如何处理值，以累加的方式自动实现一个、两个或全部三个 `Fn` trait：

* `FnOnce` 适用于可以被调用一次的闭包。所有闭包都至少实现这个 trait，因为所有闭包都可以被调用。将捕获的值移出其函数体的闭包只会实现 `FnOnce` 而不会实现其他 `Fn` trait，因为它只能被调用一次。
* `FnMut` 适用于不将捕获的值移出其函数体但可能改变捕获的值的闭包。这些闭包可以被调用多次。
* `Fn` 适用于不将捕获的值移出其函数体且不改变捕获的值的闭包，以及从其环境中什么都不捕获的闭包。这些闭包可以在不改变其环境的情况下被多次调用，这在此类情况下很重要，例如并发多次调用一个闭包。

让我们看看在示例 13-1 中使用的 `Option<T>` 上 `unwrap_or_else` 方法的定义：

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

回想一下，`T` 是泛型类型，代表 `Option` 的 `Some` 变体中的值类型。那个类型 `T` 也是 `unwrap_or_else` 函数的返回类型：例如，在 `Option<String>` 上调用 `unwrap_or_else` 的代码将获得一个 `String`。

接下来，注意 `unwrap_or_else` 函数有一个额外的泛型类型参数 `F`。`F` 类型是名为 `f` 的参数的类型，`f` 是我们调用 `unwrap_or_else` 时提供的闭包。

泛型类型 `F` 上指定的 trait 约束是 `FnOnce() -> T`，这意味着 `F` 必须能够被调用一次，不带参数，并返回一个 `T`。在 trait 约束中使用 `FnOnce` 表达了约束：`unwrap_or_else` 不会调用 `f` 超过一次。在 `unwrap_or_else` 的主体中，我们可以看到如果 `Option` 是 `Some`，`f` 不会被调用。如果 `Option` 是 `None`，`f` 将被调用一次。因为所有闭包都实现了 `FnOnce`，所以 `unwrap_or_else` 接受所有三种闭包，并且尽可能灵活。

> 注意：如果我们想做的事情不需要从环境中捕获值，我们可以在需要实现某个 `Fn` trait 的地方使用函数名而不是闭包。例如，在 `Option<Vec<T>>` 值上，我们可以调用 `unwrap_or_else(Vec::new)` 来在值为 `None` 时获取一个新的空向量。编译器会自动为函数定义实现适用的 `Fn` trait。

现在让我们看看标准库中在切片上定义的 `sort_by_key` 方法，以了解它与 `unwrap_or_else` 有何不同，以及为什么 `sort_by_key` 在 trait 约束中使用 `FnMut` 而不是 `FnOnce`。闭包以对切片中当前项的引用的形式接收一个参数，并返回一个可排序的 `K` 类型的值。当你想要按每个项的特定属性对切片进行排序时，此函数很有用。在示例 13-7 中，我们有一个 `Rectangle` 实例列表，并使用 `sort_by_key` 按它们的 `width` 属性从低到高排序。

<Listing number="13-7" file-name="src/main.rs" caption="使用 `sort_by_key` 按宽度对矩形排序">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

这段代码打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` 被定义为接受 `FnMut` 闭包的原因是它多次调用闭包：对切片中的每个项调用一次。闭包 `|r| r.width` 不从其环境中捕获、改变或移动任何内容，因此它满足 trait 约束要求。

相比之下，示例 13-8 显示了一个仅实现 `FnOnce` trait 的闭包示例，因为它将值移出了环境。编译器不允许我们将此闭包与 `sort_by_key` 一起使用。

<Listing number="13-8" file-name="src/main.rs" caption="尝试将 `FnOnce` 闭包与 `sort_by_key` 一起使用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

这是一种人为的、复杂的方式（不起作用），试图计算在对 `list` 排序时 `sort_by_key` 调用闭包的次数。此代码尝试通过将 `value`（来自闭包环境的 `String`）推入 `sort_operations` 向量来进行计数。闭包捕获了 `value`，然后通过将 `value` 的所有权转移到 `sort_operations` 向量将其移出闭包。这个闭包只能被调用一次；尝试第二次调用它将不起作用，因为 `value` 将不再在环境中被推入 `sort_operations`！因此，此闭包只实现了 `FnOnce`。当我们尝试编译此代码时，我们会得到这个错误，指出 `value` 不能被移出闭包，因为闭包必须实现 `FnMut`：

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

错误指向闭包体中将 `value` 移出环境的行。要修复此问题，我们需要更改闭包体，使其不将值移出环境。在环境中保持一个计数器并在闭包体中递增其值是计算闭包被调用次数的更直接方式。示例 13-9 中的闭包可以与 `sort_by_key` 一起使用，因为它只捕获了 `num_sort_operations` 计数器的可变引用，因此可以被调用多次。

<Listing number="13-9" file-name="src/main.rs" caption="允许将 `FnMut` 闭包与 `sort_by_key` 一起使用">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

在定义或使用使用闭包的函数或类型时，`Fn` trait 很重要。在下一节中，我们将讨论迭代器。许多迭代器方法接受闭包参数，因此在我们继续时，请记住这些闭包细节！

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
