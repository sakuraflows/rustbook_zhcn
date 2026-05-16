## 附录 B：运算符与符号（Operators and Symbols）

本附录包含 Rust 语法的词汇表，包括运算符以及其他在路径（paths）、泛型（generics）、
trait 约束（trait bounds）、宏（macros）、属性（attributes）、注释（comments）、元组（tuples）
和括号（brackets）等上下文中出现的符号。

### 运算符（Operators）

表 B-1 列出了 Rust 中的运算符，包括运算符在上下文中使用的示例、简要说明，以及该运算符
是否可重载（overloadable）。如果运算符可重载，则同时列出了用于重载该运算符的相应 trait。

<span class="caption">表 B-1：运算符（Operators）</span>

| 运算符                    | 示例                                                    | 说明                                                               | 可重载？      |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------ | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | 宏展开（Macro expansion）                                          |                |
| `!`                       | `!expr`                                                 | 按位取反或逻辑非（Bitwise or logical complement）                  | `Not`          |
| `!=`                      | `expr != expr`                                          | 不相等比较（Nonequality comparison）                               | `PartialEq`    |
| `%`                       | `expr % expr`                                           | 算术取余（Arithmetic remainder）                                   | `Rem`          |
| `%=`                      | `var %= expr`                                           | 算术取余并赋值（Arithmetic remainder and assignment）              | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | 借用（Borrow）                                                     |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | 借用指针类型（Borrowed pointer type）                              |                |
| `&`                       | `expr & expr`                                           | 按位与（Bitwise AND）                                              | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | 按位与并赋值（Bitwise AND and assignment）                         | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | 短路逻辑与（Short-circuiting logical AND）                         |                |
| `*`                       | `expr * expr`                                           | 算术乘法（Arithmetic multiplication）                              | `Mul`          |
| `*=`                      | `var *= expr`                                           | 算术乘法并赋值（Arithmetic multiplication and assignment）         | `MulAssign`    |
| `*`                       | `*expr`                                                 | 解引用（Dereference）                                              | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | 裸指针（Raw pointer）                                              |                |
| `+`                       | `trait + trait`, `'a + trait`                           | 复合类型约束（Compound type constraint）                           |                |
| `+`                       | `expr + expr`                                           | 算术加法（Arithmetic addition）                                    | `Add`          |
| `+=`                      | `var += expr`                                           | 算术加法并赋值（Arithmetic addition and assignment）               | `AddAssign`    |
| `,`                       | `expr, expr`                                            | 参数与元素分隔符（Argument and element separator）                 |                |
| `-`                       | `- expr`                                                | 算术取负（Arithmetic negation）                                    | `Neg`          |
| `-`                       | `expr - expr`                                           | 算术减法（Arithmetic subtraction）                                 | `Sub`          |
| `-=`                      | `var -= expr`                                           | 算术减法并赋值（Arithmetic subtraction and assignment）            | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | 函数与闭包返回类型（Function and closure return type）             |                |
| `.`                       | `expr.ident`                                            | 字段访问（Field access）                                           |                |
| `.`                       | `expr.ident(expr, ...)`                                 | 方法调用（Method call）                                            |                |
| `.`                       | `expr.0`, `expr.1`, 等等                                | 元组索引（Tuple indexing）                                         |                |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | 右排除范围字面量（Right-exclusive range literal）                  | `PartialOrd`   |
| `..=`                     | `..=expr`, `expr..=expr`                                | 右包含范围字面量（Right-inclusive range literal）                  | `PartialOrd`   |
| `..`                      | `..expr`                                                | 结构体字面量更新语法（Struct literal update syntax）               |                |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | "其余部分"模式绑定（"And the rest" pattern binding）               |                |
| `...`                     | `expr...expr`                                           | （已废弃，请使用 `..=`）在模式中：包含范围模式（inclusive range pattern） |                |
| `/`                       | `expr / expr`                                           | 算术除法（Arithmetic division）                                    | `Div`          |
| `/=`                      | `var /= expr`                                           | 算术除法并赋值（Arithmetic division and assignment）               | `DivAssign`    |
| `:`                       | `pat: type`, `ident: type`                              | 约束（Constraints）                                                |                |
| `:`                       | `ident: expr`                                           | 结构体字段初始化（Struct field initializer）                       |                |
| `:`                       | `'a: loop {...}`                                        | 循环标签（Loop label）                                             |                |
| `;`                       | `expr;`                                                 | 语句与项终止符（Statement and item terminator）                    |                |
| `;`                       | `[...; len]`                                            | 固定大小数组语法的一部分（Part of fixed-size array syntax）        |                |
| `<<`                      | `expr << expr`                                          | 左移（Left-shift）                                                 | `Shl`          |
| `<<=`                     | `var <<= expr`                                          | 左移并赋值（Left-shift and assignment）                            | `ShlAssign`    |
| `<`                       | `expr < expr`                                           | 小于比较（Less than comparison）                                   | `PartialOrd`   |
| `<=`                      | `expr <= expr`                                          | 小于等于比较（Less than or equal to comparison）                   | `PartialOrd`   |
| `=`                       | `var = expr`, `ident = type`                            | 赋值/等价（Assignment/equivalence）                                |                |
| `==`                      | `expr == expr`                                          | 相等比较（Equality comparison）                                    | `PartialEq`    |
| `=>`                      | `pat => expr`                                           | match 分支语法的一部分（Part of match arm syntax）                 |                |
| `>`                       | `expr > expr`                                           | 大于比较（Greater than comparison）                                | `PartialOrd`   |
| `>=`                      | `expr >= expr`                                          | 大于等于比较（Greater than or equal to comparison）                | `PartialOrd`   |
| `>>`                      | `expr >> expr`                                          | 右移（Right-shift）                                                | `Shr`          |
| `>>=`                     | `var >>= expr`                                          | 右移并赋值（Right-shift and assignment）                           | `ShrAssign`    |
| `@`                       | `ident @ pat`                                           | 模式绑定（Pattern binding）                                        |                |
| `^`                       | `expr ^ expr`                                           | 按位异或（Bitwise exclusive OR）                                   | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | 按位异或并赋值（Bitwise exclusive OR and assignment）              | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | 模式备选（Pattern alternatives）                                   |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | 按位或（Bitwise OR）                                               | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | 按位或并赋值（Bitwise OR and assignment）                          | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | 短路逻辑或（Short-circuiting logical OR）                          |                |
| `?`                       | `expr?`                                                 | 错误传播（Error propagation）                                      |                |

### 非运算符符号（Non-operator Symbols）

以下表格包含所有不作为运算符使用的符号；也就是说，它们不像函数或方法调用那样运作。

表 B-2 列出了可以独立出现并在多种位置有效的符号。

<span class="caption">表 B-2：独立语法（Stand-alone Syntax）</span>

| 符号                                                                  | 说明                                                                 |
| --------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `'ident`                                                              | 命名生命周期或循环标签（Named lifetime or loop label）               |
| 数字后紧跟 `u8`、`i32`、`f64`、`usize` 等                             | 特定类型的数字字面量（Numeric literal of specific type）             |
| `"..."`                                                               | 字符串字面量（String literal）                                       |
| `r"..."`, `r#"..."#`, `r##"..."##`, 等等                              | 原始字符串字面量；转义字符不处理（Raw string literal）               |
| `b"..."`                                                              | 字节字符串字面量；构造字节数组而非字符串（Byte string literal）      |
| `br"..."`, `br#"..."#`, `br##"..."##`, 等等                           | 原始字节字符串字面量；原始字符串与字节字符串的组合                   |
| `'...'`                                                               | 字符字面量（Character literal）                                      |
| `b'...'`                                                              | ASCII 字节字面量（ASCII byte literal）                               |
| <code>&vert;...&vert; expr</code>                                     | 闭包（Closure）                                                      |
| `!`                                                                   | 发散函数的空底类型（Always-empty bottom type for diverging functions） |
| `_`                                                                   | "忽略"模式绑定；也用于使整数数字字面量可读（"Ignored" pattern binding） |

表 B-3 列出了在通过模块层次结构访问项的路径（path）上下文中出现的符号。

<span class="caption">表 B-3：路径相关语法（Path-Related Syntax）</span>

| 符号                                    | 说明                                                                                                        |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `ident::ident`                          | 命名空间路径（Namespace path）                                                                              |
| `::path`                                | 相对于 crate 根路径（即显式的绝对路径）                                                                     |
| `self::path`                            | 相对于当前模块的路径（即显式的相对路径）                                                                    |
| `super::path`                           | 相对于当前模块父级的路径                                                                                    |
| `type::ident`, `<type as trait>::ident` | 关联常量、关联函数和关联类型（Associated constants, functions, and types）                                  |
| `<type>::...`                           | 无法直接命名的类型的关联项（例如 `<&T>::...`、`<[T]>::...` 等）                                             |
| `trait::method(...)`                    | 通过指定定义该方法的 trait 来消歧方法调用                                                                   |
| `type::method(...)`                     | 通过指定方法所定义的类型来消歧方法调用                                                                      |
| `<type as trait>::method(...)`          | 通过同时指定 trait 和类型来消歧方法调用                                                                     |

表 B-4 列出了在使用泛型类型参数（generic type parameters）上下文中出现的符号。

<span class="caption">表 B-4：泛型（Generics）</span>

| 符号                           | 说明                                                                                                                                    |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | 在类型中为泛型类型指定参数（例如 `Vec<u8>`）                                                                                            |
| `path::<...>`, `method::<...>` | 在表达式中为泛型类型、函数或方法指定参数；通常称为 turbofish（例如 `"42".parse::<i32>()`）                                              |
| `fn ident<...> ...`            | 定义泛型函数（Define generic function）                                                                                                 |
| `struct ident<...> ...`        | 定义泛型结构体（Define generic structure）                                                                                              |
| `enum ident<...> ...`          | 定义泛型枚举（Define generic enumeration）                                                                                              |
| `impl<...> ...`                | 定义泛型实现（Define generic implementation）                                                                                           |
| `for<...> type`                | 更高阶生命周期约束（Higher ranked lifetime bounds）                                                                                     |
| `type<ident=type>`             | 一个或多个关联类型具有特定赋值的泛型类型（例如 `Iterator<Item=T>`）                                                                     |

表 B-5 列出了在通过 trait 约束（trait bounds）约束泛型类型参数上下文中出现的符号。

<span class="caption">表 B-5：Trait 约束（Trait Bound Constraints）</span>

| 符号                          | 说明                                                                                                                                   |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `T: U`                        | 泛型参数 `T` 被约束为实现 `U` 的类型                                                                                                   |
| `T: 'a`                       | 泛型类型 `T` 必须存活得比生命周期 `'a` 长（即该类型不能传递性地包含任何生命周期短于 `'a` 的引用）                                      |
| `T: 'static`                  | 泛型类型 `T` 不包含除 `'static` 之外的任何借用引用                                                                                     |
| `'b: 'a`                      | 泛型生命周期 `'b` 必须存活得比生命周期 `'a` 长                                                                                         |
| `T: ?Sized`                   | 允许泛型类型参数为动态大小类型（dynamically sized type）                                                                               |
| `'a + trait`, `trait + trait` | 复合类型约束（Compound type constraint）                                                                                               |

表 B-6 列出了在调用或定义宏（macros）以及为项指定属性（attributes）上下文中出现的符号。

<span class="caption">表 B-6：宏与属性（Macros and Attributes）</span>

| 符号                                        | 说明                    |
| ------------------------------------------- | ----------------------- |
| `#[meta]`                                   | 外部属性（Outer attribute）    |
| `#![meta]`                                  | 内部属性（Inner attribute）    |
| `$ident`                                    | 宏替换（Macro substitution）   |
| `$ident:kind`                               | 宏元变量（Macro metavariable） |
| `$(...)...`                                 | 宏重复（Macro repetition）     |
| `ident!(...)`, `ident!{...}`, `ident![...]` | 宏调用（Macro invocation）     |

表 B-7 列出了用于创建注释（comments）的符号。

<span class="caption">表 B-7：注释（Comments）</span>

| 符号       | 说明                         |
| ---------- | ---------------------------- |
| `//`       | 行注释（Line comment）            |
| `//!`      | 内部行文档注释（Inner line doc comment）  |
| `///`      | 外部行文档注释（Outer line doc comment）  |
| `/*...*/`  | 块注释（Block comment）           |
| `/*!...*/` | 内部块文档注释（Inner block doc comment） |
| `/**...*/` | 外部块文档注释（Outer block doc comment） |

表 B-8 列出了使用圆括号（parentheses）的上下文。

<span class="caption">表 B-8：圆括号（Parentheses）</span>

| 符号                     | 说明                                                                                    |
| ------------------------ | --------------------------------------------------------------------------------------- |
| `()`                     | 空元组（即 unit），既是字面量也是类型                                                     |
| `(expr)`                 | 括号表达式（Parenthesized expression）                                                  |
| `(expr,)`                | 单元素元组表达式（Single-element tuple expression）                                     |
| `(type,)`                | 单元素元组类型（Single-element tuple type）                                             |
| `(expr, ...)`            | 元组表达式（Tuple expression）                                                          |
| `(type, ...)`            | 元组类型（Tuple type）                                                                  |
| `expr(expr, ...)`        | 函数调用表达式；也用于初始化元组 `struct` 和元组 `enum` 变体（variants）                |

表 B-9 列出了使用花括号（curly brackets）的上下文。

<span class="caption">表 B-9：花括号（Curly Brackets）</span>

| 上下文        | 说明                  |
| ------------- | --------------------- |
| `{...}`       | 块表达式（Block expression） |
| `Type {...}`  | 结构体字面量（Struct literal）   |

表 B-10 列出了使用方括号（square brackets）的上下文。

<span class="caption">表 B-10：方括号（Square Brackets）</span>

| 上下文                                              | 说明                                                                                                                      |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                             | 数组字面量（Array literal）                                                                                               |
| `[expr; len]`                                       | 包含 `len` 个 `expr` 副本的数组字面量                                                                                     |
| `[type; len]`                                       | 包含 `len` 个 `type` 实例的数组类型                                                                                       |
| `expr[expr]`                                        | 集合索引（Collection indexing）；可重载（`Index`, `IndexMut`）                                                             |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]`  | 集合索引，用作集合切片（collection slicing），使用 `Range`、`RangeFrom`、`RangeTo` 或 `RangeFull` 作为"索引"               |
