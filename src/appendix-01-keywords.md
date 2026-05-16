## 附录 A：关键字（Keywords）

以下列表包含了 Rust 语言当前或将来保留使用的关键字。因此，它们不能被用作标识符（identifier）（原始标识符（raw identifier）除外，我们将在[“原始标识符”][raw-identifiers]<!-- ignore -->章节讨论）。**标识符（Identifiers）** 是函数、变量、参数、结构体字段、模块、crate、常量、宏、静态值、属性、类型、trait 或生命周期（lifetime）的名称。

[raw-identifiers]: #raw-identifiers

### 当前正在使用的关键字（Keywords Currently in Use）

以下是当前正在使用的关键字列表及其功能说明。

- **`as`**：执行基本类型转换（primitive casting），消除包含某一项的特质的歧义，或在 `use` 语句中重命名项。
- **`async`**：返回一个 `Future`（未来值）而非阻塞当前线程。
- **`await`**：暂停执行，直到 `Future` 的结果就绪。
- **`break`**：立即退出循环。
- **`const`**：定义常量项或常量裸指针（constant raw pointers）。
- **`continue`**：继续下一次循环迭代。
- **`crate`**：在模块路径中，指代 crate 根（crate root）。
- **`dyn`**：对 trait 对象进行动态分发（dynamic dispatch）。
- **`else`**：`if` 和 `if let` 控制流结构的后备分支。
- **`enum`**：定义枚举（enumeration）。
- **`extern`**：链接外部函数或变量。
- **`false`**：布尔（boolean）假字面量。
- **`fn`**：定义函数或函数指针类型。
- **`for`**：遍历迭代器中的项，实现 trait，或指定更高阶生命周期（higher ranked lifetime）。
- **`if`**：根据条件表达式的结果进行分支。
- **`impl`**：实现固有（inherent）或 trait 的功能。
- **`in`**：`for` 循环语法的一部分。
- **`let`**：绑定变量。
- **`loop`**：无条件循环。
- **`match`**：将值与模式（patterns）进行匹配。
- **`mod`**：定义模块（module）。
- **`move`**：使闭包（closure）获取其所有捕获（captures）的所有权。
- **`mut`**：表示引用、裸指针或模式绑定中的可变性（mutability）。
- **`pub`**：表示结构体字段、`impl` 块或模块中的公开可见性（public visibility）。
- **`ref`**：通过引用（reference）进行绑定。
- **`return`**：从函数返回。
- **`Self`**：当前正在定义或实现的类型的类型别名（type alias）。
- **`self`**：方法主体或当前模块。
- **`static`**：全局变量或持续整个程序执行的生命周期（lifetime）。
- **`struct`**：定义结构体（structure）。
- **`super`**：当前模块的父模块。
- **`trait`**：定义 trait。
- **`true`**：布尔（boolean）真字面量。
- **`type`**：定义类型别名（type alias）或关联类型（associated type）。
- **`union`**：定义[联合体（union）][union]<!-- ignore -->；仅在 union 声明中使用时为关键字。
- **`unsafe`**：表示不安全（unsafe）代码、函数、trait 或实现。
- **`use`**：将符号（symbols）引入作用域。
- **`where`**：表示约束类型的子句（clauses）。
- **`while`**：根据表达式的结果条件循环。

[union]: ../reference/items/unions.html

### 保留供将来使用的关键字（Keywords Reserved for Future Use）

以下关键字目前尚不具备任何功能，但 Rust 保留以备将来可能使用：

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### 原始标识符（Raw Identifiers）

**原始标识符（Raw identifiers）** 是一种允许你在通常不允许的地方使用关键字的语法。通过在关键字前加上 `r#` 前缀来使用原始标识符。

例如，`match` 是一个关键字。如果你尝试编译以下使用 `match` 作为函数名的函数：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

你会得到以下错误：

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

该错误表明你不能使用关键字 `match` 作为函数标识符。要将 `match` 用作函数名，你需要使用原始标识符语法，如下所示：

<span class="filename">Filename: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

这段代码可以无错误地编译。请注意函数定义中以及 `main` 中调用函数时函数名上的 `r#` 前缀。

原始标识符允许你将任何你选择的词语用作标识符，即使该词语恰好是保留关键字。这让我们在选择标识符名称时有更多的自由度，也使我们能够与那些将这些词语作为非关键字的语言编写的程序进行集成。此外，原始标识符还允许你使用与你当前 crate 不同 Rust 版本（edition）编写的库。例如，`try` 在 2015 版本中不是关键字，但在 2018、2021 和 2024 版本中则是关键字。如果你依赖一个使用 2015 版本编写的库，并且其中包含一个 `try` 函数，那么你需要使用原始标识符语法（在本例中为 `r#try`），才能在你的较新版本代码中调用该函数。更多关于版本的信息，请参见[附录 E][appendix-e]<!-- ignore -->。

[appendix-e]: appendix-05-editions.html
