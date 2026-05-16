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
- deepseek-v4-flash
- 工具链：vscode+copilot+DeepSeek V4 for Copilot Chat

提示词：
```
请翻译这个markdown文件：
要求：
- 不翻译代码块，代码块中的代码要保持原样，但可以视情况翻译代码块中大段的注释。
- 对于专业名词，要在翻译的同时，使用括号（）的形式，将英文原文注释在翻译内容的后面。
```

## 构建依赖

构建本书需要 [mdBook]，最好是与本项目中使用的版本相同（mdbook v0.5.2）。


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
