# 面向对象编程特性

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="object-oriented-programming-features-of-rust"></a>

面向对象编程（Object-oriented programming，OOP）是一种对程序进行建模的方式。作为编程概念的对象（Object）最早在 20 世纪 60 年代被引入编程语言 Simula 中。这些对象影响了 Alan Kay 的编程架构，在该架构中对象之间通过传递消息进行通信。为了描述这种架构，他在 1967 年创造了术语 _面向对象编程（object-oriented programming）_。有许多相互竞争的定义描述了什么是 OOP，根据其中一些定义，Rust 是面向对象的，但根据另一些定义，它又不是。在本章中，我们将探讨一些通常被认为是面向对象的特性，以及这些特性如何转化为符合 Rust 惯用写法（idiomatic）的风格。然后，我们将向你展示如何在 Rust 中实现一个面向对象的设计模式（design pattern），并讨论这样做与利用 Rust 的一些优势来提供解决方案之间的权衡。
