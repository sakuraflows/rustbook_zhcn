# 高级特性

到目前为止，你已经学习了 Rust 编程语言中最常用的部分。在第 21 章开始另一个项目之前，我们将探讨你偶尔会遇到但可能不会每天都用到的一些语言方面。你可以将本章作为遇到任何未知时的参考。这里介绍的特性在非常特定的情况下非常有用。尽管你可能不会经常用到它们，但我们希望确保你掌握 Rust 所提供的所有特性。

在本章中，我们将涵盖：

- Unsafe Rust：如何选择退出 Rust 的某些保证，并自行负责手动维护这些保证
- 高级 Trait（Advanced traits）：关联类型（associated types）、默认类型参数（default type parameters）、完全限定语法（fully qualified syntax）、超 trait（supertraits）以及与 trait 相关的新类型模式（newtype pattern）
- 高级类型（Advanced types）：关于新类型模式、类型别名（type aliases）、永不返回类型（never type）和动态大小类型（dynamically sized types）的更多内容
- 高级函数与闭包（Advanced functions and closures）：函数指针（function pointers）和返回闭包（returning closures）
- 宏（Macros）：定义在编译时生成更多代码的代码的方法

这是 Rust 特性的一个盛宴，每个人都能找到适合自己的内容！让我们开始吧。
