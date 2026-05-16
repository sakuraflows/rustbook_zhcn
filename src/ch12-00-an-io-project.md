# I/O 项目：构建命令行程序（Building a Command Line Program）

本章是对你到目前为止学到的许多技能的回顾，以及对更多标准库功能的探索。我们将构建一个与文件和命令行输入/输出交互的命令行工具，以实践你现在掌握的 Rust 概念。

Rust 的速度、安全性、单一二进制输出和跨平台支持使其成为创建命令行工具的理想语言，因此对于我们的项目，我们将制作经典命令行搜索工具 `grep` 的自己的版本（**g**lobally search a **r**egular **e**xpression and **p**rint，即全局搜索正则表达式并打印）。在最简单的用例中，`grep` 在指定文件中搜索指定的字符串。为此，`grep` 以文件路径和一个字符串作为参数。然后，它读取文件，找到包含该字符串参数的行，并打印这些行。

在此过程中，我们将展示如何使我们的命令行工具使用许多其他命令行工具使用的终端功能。我们将读取环境变量的值，以允许用户配置我们工具的行为。我们还将错误消息打印到标准错误控制台流（`stderr`）而不是标准输出（`stdout`），以便例如用户可以将成功输出重定向到文件，同时仍然在屏幕上看到错误消息。

一位 Rust 社区成员 Andrew Gallant 已经创建了一个功能齐全、速度极快的 `grep` 版本，称为 `ripgrep`。相比之下，我们的版本将相当简单，但本章将为你提供一些理解诸如 `ripgrep` 等真实世界项目所需的背景知识。

我们的 `grep` 项目将结合你到目前为止学到的许多概念：

- 组织代码（[第 7 章][ch7]<!-- ignore -->）
- 使用向量和字符串（[第 8 章][ch8]<!-- ignore -->）
- 处理错误（[第 9 章][ch9]<!-- ignore -->）
- 在适当的位置使用 trait 和生命周期（[第 10 章][ch10]<!-- ignore -->）
- 编写测试（[第 11 章][ch11]<!-- ignore -->）

我们还将简要介绍闭包（closure）、迭代器（iterator）和 trait 对象（trait object），[第 13 章][ch13]<!-- ignore -->和[第 18 章][ch18]<!-- ignore -->将详细涵盖这些内容。

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
