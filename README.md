# 《RUst程序设计语言》中文翻译（ai机翻）
正在翻译中.....
20260516




你也可以在线免费阅读这本书。请将本书视为与最新的[稳定版]、[测试版]或[nightly] Rust 发行版一同提供。请注意，这些版本中可能存在的问题可能已经在本仓库中得到了修复，因为这些版本更新频率较低。

[稳定版]: https://doc.rust-lang.org/stable/book/
[测试版]: https://doc.rust-lang.org/beta/book/
[nightly]: https://doc.rust-lang.org/nightly/book/

请查看[发布页面（releases）][releases]以下载本书中所有代码清单（code listings）的代码。

[releases]: https://github.com/rust-lang/book/releases
## 使用的大模型及提示词
- deepseek-v4-flash  (部分长文本使用了pro)
- 工具链：vscode+copilot+DeepSeek V4 for Copilot Chat

提示词：
```
请翻译这个markdown文件：
要求：
- 逐个.md文件进行翻译，翻译完一个md文件结束对一个md文件的翻译，不互相干扰。
- 不翻译代码块，代码块中的代码要保持原样，但可以视情况翻译代码块中大段的注释。
- 对于专业名词，要在翻译的同时，使用括号（）的形式，将英文原文注释在翻译内容的后面。
```

## 构建依赖

构建本书需要 [mdBook]，最好是与本项目中使用的版本相同（mdbook v0.5.2）。或查看


安装方法如下：

[mdBook]: https://github.com/rust-lang/mdBook

```bash
$ cargo install mdbook --locked --version <version_num>
```

## 构建

要构建书籍，请输入:

```bash
$ mdbook build
```

输出将位于 `book` 子目录中。要查看它，请在网页浏览器（web browser）中打开。

_Firefox（火狐浏览器）:_

```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome（谷歌浏览器）:_

```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

要运行测试（tests）：

```bash
$ cd packages/trpl
$ mdbook test --library-path packages/trpl/target/debug/deps
```
## 贡献
本仓库希望并欢迎任何人的贡献.同时,为了对rust-lang社区负责,我们希望新的翻译贡献所使用的翻译模块的能力不低于deepseek-v4.

在贡献前请阅读如下内容:
### 文档结构说明
事实上，通过查看[rust-lang/book]main分支可以发现，这套文档有两个历史版本和一个正在持续更新的版本，这个正在更新的版本的“文档源代码”（即markdown）
在/src下。

本仓库使用ai直接翻译[rust-lang/book](https://github.com/rust-lang/book)，翻译后的文本在/src下,原文在本仓库的/src-en中，

### 翻译流程说明
本仓库会每隔一段时间拉取[rust-lang/book](https://github.com/rust-lang/book)/src与本仓库的/src-en进行同步,如果发现变动,
那么我将会手工使用ai重新翻译发生/src-en下变动的markdown文件,再将翻译好的文件替换本仓库/src中原有旧版本的md文件
```mermaid
graph LR
    A[拉取 rust-lang/src] --> B[同步到本仓库 /src-en 目录]
    B --> C[检测 /src-en 中变动的 .md 文件]
    C --> D[完全重新翻译变动的 .md 文件]
    D --> E[将翻译后的文件替换本仓库 /src 下对应的旧版 .md 文件]
```
### 其它部分的变动
/src下md文件的内嵌内容（如代码等），保存在listings中，在src-en发生变动时，也应当更新此文件夹的内容。

如果[rust-lang/book](https://github.com/rust-lang/book)除了src之外其它的文件(如主题,mdbook等)有了较大的变动,那么本项目则会单独拉取修改.
要注意`.cargo/cargo.toml.txt`与`rustbook`中`cargo.toml`的区别,当rustbook中cargo.toml发生变动时,往往意味着构建版本的或构建依赖组件mdbook的版本更新




# 未来计划
使用手搓+ai写一个基于ii8n.site或git2-rs+similar+pulldown-crmark从头编写一个支持如下功能的翻译模块：
- 检测英文文档是否更新变动
- 将markdown内容解析为对象（文本段落、标题、注释、代码）
- ai翻译

------

<br>
<br>

# 来自rust-lang/book官方仓库的说明
## 贡献指南（Contributing）

我们非常欢迎你的帮助！请查看 [CONTRIBUTING.md][contrib] 了解我们正在寻找的贡献（contributions）类型。

[contrib]: https://github.com/rust-lang/book/blob/main/CONTRIBUTING.md

由于本书是[印刷版][nostarch]，并且我们希望尽可能保持在线版本与印刷版本一致，因此处理你的 issue（问题）或 pull request（拉取请求）可能需要比平时更长的时间。

到目前为止，我们一直在配合 [Rust 版本（Editions）](https://doc.rust-lang.org/edition-guide/)进行较大规模的修订。在这些大规模修订之间，我们只会修正错误。如果你的 issue 或 pull request 严格来说不是修复错误，它可能会被搁置，直到我们下一次进行大规模修订：预计需要数月甚至数年的时间。感谢你的耐心！

### 翻译（Translations）

我们非常希望得到翻译本书的帮助！请查看 [Translations] 标签以加入正在进行的翻译工作。开一个新的 issue 来开始一种新语言的翻译吧！我们正在等待 [mdbook 支持][mdbook support]多语言功能，之后才会合并这些翻译，但请随时开始！

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5

## 拼写检查（Spellchecking）

要扫描源文件中的拼写错误，可以使用 `ci` 目录中的 `spellcheck.sh` 脚本。它需要一个有效单词的字典（dictionary），该字典位于 `ci/dictionary.txt`。如果脚本产生了误报（false positive，例如你使用了 `BTreeMap` 这个词，而脚本认为它无效），你需要将这个单词添加到 `ci/dictionary.txt` 中（保持排序顺序以保持一致性）。
