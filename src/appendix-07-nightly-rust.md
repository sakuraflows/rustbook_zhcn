## 附录 G - Rust 是如何开发的以及"Nightly Rust"

本附录介绍 Rust 是如何开发的，以及这对你作为 Rust 开发者有何影响。

### 稳定而不停滞（Stability Without Stagnation）

作为一门语言，Rust 非常关注代码的**稳定性**。我们希望 Rust 成为你可以依赖的坚如磐石的基础，如果事物不断变化，那将是不可能的。同时，如果我们不能实验新特性，我们可能直到发布之后才会发现重要的缺陷，而那时我们已经无法再更改了。

我们对此问题的解决方案被称为"稳定而不停滞（stability without stagnation）"，我们的指导原则是：你永远不必害怕升级到新版本的稳定版 Rust。每次升级都应该是无痛的，同时还应该为你带来新特性、更少的 bug 和更快的编译时间。

### 嘟——嘟！发布频道与"乘坐火车"（Release Channels and Riding the Trains）

Rust 的开发遵循**列车时刻表（train schedule）**。也就是说，所有开发都在 Rust 仓库的主分支（main branch）中进行。发布遵循软件发布火车模型（software release train model），该模型已被 Cisco IOS 和其他软件项目所使用。Rust 有三个**发布频道（release channels）**：

- Nightly（夜间版）
- Beta（测试版）
- Stable（稳定版）

大多数 Rust 开发者主要使用稳定版频道，但那些想要尝试实验性新特性的人可以使用 nightly 或 beta。

以下是开发和发布流程的一个示例：假设 Rust 团队正在开发 Rust 1.5 的发布。该版本实际上是在 2015 年 12 月发布的，但它能给我们提供实际的版本号。一个新特性被添加到 Rust：一个新的提交（commit）合并到主分支。每晚都会生成一个新的 nightly 版本的 Rust。每天都是发布日，这些发布由我们的发布基础设施自动创建。因此，随着时间的推移，我们的发布看起来像这样，每晚一次：

```text
nightly: * - - * - - *
```

每六周，就到了准备新发布的时候！ Rust 仓库的 `beta` 分支从 nightly 使用的主分支分出。现在有两个发布：

```text
nightly: * - - * - - *
                     |
beta:                *
```

大多数 Rust 用户不会主动使用 beta 版，但会在他们的 CI 系统中针对 beta 版进行测试，以帮助 Rust 发现可能的回归（regression）。与此同时，每晚仍有一个 nightly 发布：

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

假设发现了一个回归。还好我们有一些时间在回归潜入稳定版之前测试 beta 版！修复被应用到主分支，这样 nightly 版得到修复，然后将修复回溯（backport）到 `beta` 分支，并生成一个新的 beta 版发布：

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

在第一个 beta 版创建六周后，是时候发布稳定版了！`stable` 分支由 `beta` 分支生成：

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

万岁！Rust 1.5 完成了！然而，我们忘了一件事：因为六周已经过去了，我们还需要一个 Rust _下一个_版本 1.6 的新 beta 版。因此，在 `stable` 从 `beta` 分出之后，下一版本的 `beta` 再次从 `nightly` 分出：

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

这被称为"火车模型（train model）"，因为每六周，一个发布"离开车站"，但它在作为稳定版发布之前仍需经过 beta 频道的旅程。

Rust 每六周发布一次，像时钟一样准确。如果你知道一次 Rust 发布的日期，你就知道下一次发布的日期：六周后。每六周安排一次发布的一个好处是，下一趟火车很快就会到来。如果一个特性错过了某个特定版本，不必担心：很快就会有另一个版本！这有助于减少在发布日期临近时偷偷塞入可能不够完善的特性的压力。

得益于这一过程，你始终可以查看下一个 Rust 构建版本，并自行验证升级是否容易：如果 beta 版没有按预期工作，你可以向团队报告，并在下一个稳定版发布之前修复它！beta 版中出现问题的概率相对较小，但 `rustc` 仍然是一个软件，bug 确实存在。

### 维护时间（Maintenance time）

Rust 项目支持最新的稳定版本。当新的稳定版本发布时，旧版本达到生命周期终点（end of life, EOL）。这意味着每个版本的支持周期为六周。

### 不稳定特性（Unstable Features）

这个发布模型还有一个问题：不稳定特性。Rust 使用一种称为"特性标志（feature flags）"的技术来确定在给定版本中启用哪些特性。如果某个新特性正在积极开发中，它会合并到主分支，因此也会出现在 nightly 中，但会隐藏在**特性标志（feature flag）**之后。如果你，作为用户，希望尝试正在开发中的特性，你可以这样做，但你必须使用 nightly 版 Rust 并在源代码中使用相应的标志来启用。

如果你使用的是 beta 或稳定版 Rust，则无法使用任何特性标志。这是关键所在，它让我们可以在将新特性永久声明为稳定之前先进行实际使用。那些希望尝鲜的人可以选择使用最新技术，而那些想要坚如磐石体验的人可以坚持使用稳定版，并知道他们的代码不会出问题。稳定而不停滞。

本书仅包含稳定特性的信息，因为正在开发中的特性仍在变化，并且它们肯定会在本书编写完成与它们在稳定版构建中启用之间有所不同。你可以在线查找仅用于 nightly 版的文档。

### Rustup 与 Rust Nightly 的作用

Rustup 可以轻松地在不同的 Rust 发布频道之间切换，无论是全局还是按项目设置。默认情况下，你将安装稳定版 Rust。例如，要安装 nightly 版：

```console
$ rustup toolchain install nightly
```

你也可以使用 `rustup` 查看所有已安装的**工具链（toolchains）**（Rust 版本及相关组件）。以下是在作者之一的 Windows 电脑上的示例：

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

如你所见，稳定版工具链是默认的。大多数 Rust 用户大部分时间使用稳定版。你可能大部分时间使用稳定版，但在特定项目中使用 nightly 版，因为你关心前沿特性。为此，你可以使用 `rustup override` 在该项目的目录中设置 nightly 工具链，以便 `rustup` 在该目录下使用它：

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

现在，每当你进入 `~/projects/needs-nightly/` 目录并调用 `rustc` 或 `cargo` 时，`rustup` 将确保你使用的是 nightly Rust，而不是默认的稳定版 Rust。当你有很多 Rust 项目时，这非常方便！

### RFC 流程与团队（The RFC Process and Teams）

那么，你如何了解这些新特性呢？Rust 的开发模式遵循 **RFC（Request For Comments，意见征求）流程**。如果你希望改进 Rust，你可以编写一份提案，称为 RFC。

任何人都可以编写 RFC 来改进 Rust，这些提案由 Rust 团队（由许多主题子团队组成）进行审查和讨论。在 [Rust 官网](https://www.rust-lang.org/governance) 上有完整的团队列表，包括项目各个领域的团队：语言设计、编译器实现、基础设施、文档等。相应的团队阅读提案和评论，撰写他们自己的评论，最终达成共识以接受或拒绝该特性。

如果特性被接受，就会在 Rust 仓库中开启一个 issue（问题），然后有人可以实现它。实现它的人很可能不是最初提出该特性的人！当实现准备好后，它会合并到主分支中，并置于特性门控（feature gate）之后，正如我们在 ["Unstable Features"](#unstable-features)<!-- ignore --> 部分讨论的那样。

一段时间后，一旦使用 nightly 版的 Rust 开发者能够试用新特性，团队成员将讨论该特性、它在 nightly 版上的表现，并决定是否应将其纳入稳定版 Rust。如果决定推进，特性门控将被移除，该特性现在被认为是稳定的！它将乘坐火车进入新的 Rust 稳定版发布。
