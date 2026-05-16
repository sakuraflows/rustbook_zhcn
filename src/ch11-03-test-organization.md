## 测试组织（Test Organization）

正如本章开头提到的，测试是一项复杂的学科，不同的人使用不同的术语和组织方式。Rust 社区将测试分为两个主要类别：单元测试（unit tests）和集成测试（integration tests）。*单元测试*规模较小，更专注，一次单独测试一个模块，并且可以测试私有接口。*集成测试*完全位于你的库外部，以任何其他外部代码使用你的代码的相同方式使用你的代码，仅使用公共接口，并且每个测试可能涉及多个模块。

编写这两种测试对于确保库的各个部分（单独地和一起地）按你期望的方式工作都很重要。

### 单元测试

单元测试的目的是在与其余代码隔离的情况下测试每个代码单元，以快速定位代码在哪些地方按预期工作，哪些地方没有。你将单元测试放在 _src_ 目录中，与它们所测试的代码放在同一个文件中。惯例是在每个文件中创建一个名为 `tests` 的模块来包含测试函数，并使用 `cfg(test)` 标注该模块。

#### `tests` 模块和 `#[cfg(test)]`

`tests` 模块上的 `#[cfg(test)]` 注解告诉 Rust，仅当你运行 `cargo test` 时才编译和运行测试代码，而不是在运行 `cargo build` 时。这在你只想构建库时可以节省编译时间，并且在生成的编译产物中节省空间，因为测试不包含在内。你会看到，由于集成测试放在不同的目录中，它们不需要 `#[cfg(test)]` 注解。然而，由于单元测试与代码放在相同的文件中，你将使用 `#[cfg(test)]` 来指定它们不应包含在编译结果中。

回想一下，当我们在本章第一节中生成新的 `adder` 项目时，Cargo 为我们生成了以下代码：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

在自动生成的 `tests` 模块上，属性 `cfg` 代表 *configuration（配置）*，并告诉 Rust 只有在给定特定配置选项时才应包含以下项。在这种情况下，配置选项是 `test`，由 Rust 提供用于编译和运行测试。通过使用 `cfg` 属性，Cargo 仅在我们主动使用 `cargo test` 运行测试时才编译我们的测试代码。这包括此模块中可能存在的任何辅助函数，以及标注了 `#[test]` 的函数。

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### 测试私有函数

在测试社区中，关于是否应该直接测试私有函数存在争议，而其他语言使得测试私有函数变得困难或不可能。无论你遵循哪种测试理念，Rust 的隐私规则确实允许你测试私有函数。考虑示例 11-12 中带有私有函数 `internal_adder` 的代码。

<Listing number="11-12" file-name="src/lib.rs" caption="测试私有函数">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

注意，`internal_adder` 函数未被标记为 `pub`。测试只是 Rust 代码，`tests` 模块只是另一个模块。正如我们在[“用于引用模块树中的项的路径”][paths]<!-- ignore -->中所讨论的，子模块中的项可以使用其祖先模块中的项。在此测试中，我们使用 `use super::*` 将所有属于 `tests` 模块父模块的项引入作用域，然后测试可以调用 `internal_adder`。如果你认为不应测试私有函数，Rust 中没有任何东西会强迫你这样做。

### 集成测试

在 Rust 中，集成测试完全位于你的库外部。它们以任何其他代码使用你的库的相同方式使用你的库，这意味着它们只能调用属于库公共 API 的函数。它们的目的是测试库的多个部分是否能够正确地协同工作。能够单独正确工作的代码单元在集成时可能会出现问题，因此集成代码的测试覆盖范围也很重要。要创建集成测试，你首先需要一个 _tests_ 目录。

#### _tests_ 目录

我们在项目目录的顶层创建一个 _tests_ 目录，与 _src_ 相邻。Cargo 知道要在此目录中查找集成测试文件。然后我们可以根据需要创建任意数量的测试文件，Cargo 会将每个文件编译为单独的 crate。

让我们创建一个集成测试。在 _src/lib.rs_ 文件中仍然保留示例 11-12 中的代码，创建一个 _tests_ 目录，并创建一个名为 _tests/integration_test.rs_ 的新文件。你的目录结构应如下所示：

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

将示例 11-13 中的代码输入到 _tests/integration_test.rs_ 文件中。

<Listing number="11-13" file-name="tests/integration_test.rs" caption="`adder` crate 中某个函数的集成测试">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

_tests_ 目录中的每个文件都是单独的 crate，因此我们需要将库引入每个测试 crate 的作用域。因此，我们在代码顶部添加了 `use adder::add_two;`，这在单元测试中是不需要的。

我们不需要在 _tests/integration_test.rs_ 中使用 `#[cfg(test)]` 标注任何代码。Cargo 特殊处理 _tests_ 目录，并且仅在我们运行 `cargo test` 时才编译此目录中的文件。现在运行 `cargo test`：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

输出的三个部分包括单元测试、集成测试和文档测试。注意，如果某个部分中的任何测试失败，后续部分将不会运行。例如，如果单元测试失败，将不会有集成测试和文档测试的输出，因为这些测试只有在所有单元测试通过时才会运行。

单元测试的第一个部分与我们一直看到的相同：每个单元测试一行（一个名为 `internal` 的测试，我们在示例 11-12 中添加的），然后是单元测试的摘要行。

集成测试部分以 `Running tests/integration_test.rs` 行开头。接下来，是该集成测试中每个测试函数的一行，以及在 `Doc-tests adder` 部分开始之前的集成测试结果摘要行。

每个集成测试文件都有自己的部分，因此如果我们在 _tests_ 目录中添加更多文件，就会有更多的集成测试部分。

我们仍然可以通过将特定测试函数的名称作为参数传递给 `cargo test` 来运行该测试函数。要运行特定集成测试文件中的所有测试，请使用 `cargo test` 的 `--test` 参数，后跟文件名：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

此命令仅运行 _tests/integration_test.rs_ 文件中的测试。

#### 集成测试中的子模块

随着你添加更多的集成测试，你可能希望在 _tests_ 目录中创建更多文件来帮助组织它们；例如，你可以根据测试的功能对测试函数进行分组。如前所述，_tests_ 目录中的每个文件都被编译为其自己的独立 crate，这对于创建单独的作用域以更接近地模仿最终用户使用你的 crate 的方式非常有用。然而，这意味着 _tests_ 目录中的文件与 _src_ 中的文件行为不同，正如你在第 7 章中关于如何将代码分离为模块和文件所了解的那样。

_tests_ 目录文件的不同行为最明显的是，当你有一组辅助函数要在多个集成测试文件中使用，并且你尝试按照第 7 章的[“将模块分隔到不同文件中”][separating-modules-into-files]<!-- ignore -->部分的步骤将它们提取到一个公共模块中时。例如，如果我们创建 _tests/common.rs_ 并在其中放置一个名为 `setup` 的函数，我们可以向 `setup` 添加一些我们想从多个测试文件中的多个测试函数调用的代码：

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

当我们再次运行测试时，即使在输出中会看到 _common.rs_ 文件的新部分，即使此文件不包含任何测试函数，也没有在任何地方调用 `setup` 函数：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

`common` 出现在测试结果中并显示 `running 0 tests` 不是我们想要的。我们只想与其他集成测试文件共享一些代码。为了避免 `common` 出现在测试输出中，我们将创建 _tests/common/mod.rs_ 而不是 _tests/common.rs_。项目目录现在如下所示：

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

这是 Rust 也理解的较旧的命名约定，我们在第 7 章的[“备用文件路径”][alt-paths]<!-- ignore -->中提到过。以这种方式命名文件告诉 Rust 不要将 `common` 模块视为集成测试文件。当我们把 `setup` 函数代码移到 _tests/common/mod.rs_ 并删除 _tests/common.rs_ 文件时，测试输出中的该部分将不再出现。_tests_ 目录的子目录中的文件不会作为单独的 crate 编译，也不会在测试输出中显示为单独的部分。

在创建了 _tests/common/mod.rs_ 之后，我们可以从任何集成测试文件中将其作为模块使用。以下是从 _tests/integration_test.rs_ 中的 `it_adds_two` 测试调用 `setup` 函数的示例：

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

注意，`mod common;` 声明与我们在示例 7-21 中演示的模块声明相同。然后，在测试函数中，我们可以调用 `common::setup()` 函数。

#### 二进制 crate 的集成测试

如果我们的项目是一个仅包含 _src/main.rs_ 文件而没有 _src/lib.rs_ 文件的二进制 crate，我们无法在 _tests_ 目录中创建集成测试，也不能使用 `use` 语句将 _src/main.rs_ 文件中定义的函数引入作用域。只有库 crate 会暴露其他 crate 可以使用的函数；二进制 crate 旨在独立运行。

这就是为什么提供二进制的 Rust 项目会有一个简单的 _src/main.rs_ 文件，该文件调用位于 _src/lib.rs_ 文件中的逻辑的原因之一。使用这种结构，集成测试*可以*使用 `use` 测试库 crate，使重要功能可用。如果重要功能正常，_src/main.rs_ 文件中的少量代码也将正常工作，并且这些少量代码不需要测试。

## 总结

Rust 的测试功能提供了一种指定代码应如何运行的方法，以确保即使在你进行更改时，代码也能继续按预期工作。单元测试分别测试库的不同部分，并且可以测试私有实现细节。集成测试检查库的多个部分是否能够正确协同工作，并且它们使用库的公共 API 以与外部代码将使用它的相同方式测试代码。即使 Rust 的类型系统和所有权规则有助于防止某些类型的 bug，但测试对于减少与代码预期行为方式相关的逻辑 bug 仍然很重要。

让我们将本章以及前几章学到的知识结合起来，着手一个项目！

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
