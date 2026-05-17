## 构建单线程 Web 服务器

我们将从让一个单线程 Web 服务器开始工作。在开始之前，让我们快速浏览一下构建 Web 服务器所涉及的协议。这些协议的细节超出了本书的范围，但一个简要的概述将为你提供所需的信息。

构建 Web 服务器涉及的两个主要协议是*超文本传输协议（Hypertext Transfer Protocol，HTTP）*和*传输控制协议（Transmission Control Protocol，TCP）*。这两个协议都是*请求-响应（request-response）*协议，意味着*客户端（client）*发起请求，*服务器（server）*监听请求并向客户端提供响应。这些请求和响应的内容由协议定义。

TCP 是较低层的协议，描述了信息如何从一个服务器传输到另一个服务器，但不指定这些信息是什么。HTTP 建立在 TCP 之上，通过定义请求和响应的内容来实现。从技术上讲，可以将 HTTP 与其他协议一起使用，但在绝大多数情况下，HTTP 通过 TCP 发送其数据。我们将处理 TCP 和 HTTP 请求与响应的原始字节。

### 监听 TCP 连接

我们的 Web 服务器需要监听 TCP 连接，因此这是我们首先要处理的部分。标准库提供了一个 `std::net` 模块，让我们可以做到这一点。让我们按照通常的方式创建一个新项目：

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

现在在 _src/main.rs_ 中输入清单 21-1 中的代码来开始。这段代码将在本地地址 `127.0.0.1:7878` 上监听传入的 TCP 流。当它接收到一个传入流时，会打印 `Connection established!`。

<Listing number="21-1" file-name="src/main.rs" caption="监听传入流并在接收到流时打印消息">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

使用 `TcpListener`，我们可以在地址 `127.0.0.1:7878` 上监听 TCP 连接。在地址中，冒号之前的部分是代表你计算机的 IP 地址（这在每台计算机上都是相同的，并不特指作者的计算机），而 `7878` 是端口。我们选择这个端口有两个原因：HTTP 通常不接受此端口上的连接，因此我们的服务器不太可能与你可能在机器上运行的其他 Web 服务器冲突，而且 7878 是电话键盘上拼写为 *rust* 的数字。

这个场景中的 `bind` 函数类似于 `new` 函数，它会返回一个新的 `TcpListener` 实例。这个函数被称为 `bind`，因为在网络中，连接到一个端口进行监听被称为"绑定到一个端口（binding to a port）"。

`bind` 函数返回一个 `Result<T, E>`，这表明绑定可能失败，例如，如果我们运行了两个程序实例并导致两个程序监听同一个端口。由于我们只是为了学习目的而编写一个基础服务器，我们不会担心处理这类错误；相反，如果发生错误，我们使用 `unwrap` 来停止程序。

`TcpListener` 上的 `incoming` 方法返回一个迭代器，为我们提供一连串的流（更具体地说，是 `TcpStream` 类型的流）。一个*流（stream）*表示客户端和服务器之间的开放连接。*连接（connection）*是指完整的请求和响应过程，其中客户端连接到服务器，服务器生成响应，然后服务器关闭连接。因此，我们将从 `TcpStream` 读取以查看客户端发送了什么，然后将我们的响应写入流以将数据发送回客户端。总的来说，这个 `for` 循环将依次处理每个连接，并产生一连串的流供我们处理。

目前，我们对流的处理包括调用 `unwrap` 来在流出现任何错误时终止程序；如果没有错误，程序会打印一条消息。我们将在下一个清单中添加针对成功情况的更多功能。当客户端连接到服务器时，我们可能从 `incoming` 方法收到错误的原因是我们实际上不是在遍历连接。相反，我们是在遍历*连接尝试（connection attempts）*。连接可能由于多种原因而不成功，其中许多原因与操作系统有关。例如，许多操作系统对它们可以同时支持的开放连接数量有限制；超过该数量的新连接尝试将产生错误，直到一些开放连接被关闭。

让我们尝试运行这段代码！在终端中调用 `cargo run`，然后在 Web 浏览器中加载 _127.0.0.1:7878_。浏览器应该显示一条错误消息，比如"Connection reset（连接重置）"，因为服务器目前还没有发送任何数据。但是当你查看终端时，应该会看到浏览器连接到服务器时打印的几条消息！

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

有时你会看到一次浏览器请求打印多条消息；原因可能是浏览器在请求页面的同时，也在请求其他资源，比如浏览器标签中出现的 _favicon.ico_ 图标。

也可能是浏览器试图多次连接服务器，因为服务器没有响应任何数据。当 `stream` 在循环结束时超出作用域并被丢弃时，连接会作为 `drop` 实现的一部分被关闭。浏览器有时会通过重试来处理关闭的连接，因为问题可能是暂时的。

浏览器有时也会打开多个到服务器的连接而不发送任何请求，以便如果它们*确实*稍后发送请求，这些请求可以更快地发生。当这种情况发生时，我们的服务器会看到每个连接，无论该连接上是否有任何请求。例如，许多基于 Chrome 的浏览器版本都会这样做；你可以通过使用隐私浏览模式或使用不同的浏览器来禁用该优化。

关键是我们已经成功获得了一个 TCP 连接的句柄！

请记住在运行完特定版本的代码后按 <kbd>ctrl</kbd>-<kbd>C</kbd> 停止程序。然后，在进行每组代码更改后，通过调用 `cargo run` 命令重新启动程序，以确保你正在运行最新的代码。

### 读取请求

让我们实现从浏览器读取请求的功能！为了分离"先获取连接"和"然后对连接执行某些操作"这两个关注点，我们将启动一个新的函数来处理连接。在这个新的 `handle_connection` 函数中，我们将从 TCP 流中读取数据并打印出来，以便我们可以看到从浏览器发送的数据。将代码改为清单 21-2 所示。

<Listing number="21-2" file-name="src/main.rs" caption="从 `TcpStream` 读取并打印数据">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

我们将 `std::io::BufReader` 和 `std::io::prelude` 引入作用域，以便访问允许我们从流读取和写入的 trait 和类型。在 `main` 函数的 `for` 循环中，我们不再打印表示建立连接的消息，而是调用新的 `handle_connection` 函数并将 `stream` 传递给它。

在 `handle_connection` 函数中，我们创建了一个新的 `BufReader` 实例，它包装了对 `stream` 的引用。`BufReader` 通过为我们管理对 `std::io::Read` trait 方法的调用来添加缓冲功能。

我们创建一个名为 `http_request` 的变量来收集浏览器发送给服务器的请求行。我们通过添加 `Vec<_>` 类型标注来表示我们希望将所有这些行收集到一个向量中。

`BufReader` 实现了 `std::io::BufRead` trait，它提供了 `lines` 方法。`lines` 方法每当遇到换行字节时就将数据流分割，返回一个 `Result<String, std::io::Error>` 的迭代器。为了获取每个 `String`，我们对每个 `Result` 进行 `map` 和 `unwrap`。如果数据不是有效的 UTF-8，或者从流中读取时出现问题，`Result` 可能是错误。同样，生产程序应更优雅地处理这些错误，但为简单起见，我们选择在出错时停止程序。

浏览器通过连续发送两个换行符来表示 HTTP 请求的结束，因此要从流中获取一个请求，我们逐行读取，直到得到一个空字符串行。一旦我们将这些行收集到向量中，就使用漂亮的调试格式将其打印出来，以便查看 Web 浏览器发送给我们的服务器的指令。

让我们尝试这段代码！启动程序，再次在 Web 浏览器中发起请求。请注意，浏览器中仍然会显示错误页面，但我们的程序在终端中的输出现在将类似于这样：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

根据你的浏览器，你可能会得到略有不同的输出。既然我们正在打印请求数据，我们可以通过查看请求第一行中 `GET` 之后的路径，来了解为什么从一个浏览器请求会得到多个连接。如果重复的连接都在请求 _/_，我们就知道浏览器因为未从我们的程序获得响应而重复尝试获取 _/_。

让我们分解这些请求数据，以理解浏览器在向我们的程序请求什么。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### 更仔细地查看 HTTP 请求

HTTP 是一种基于文本的协议，请求采用以下格式：

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

第一行是*请求行（request line）*，包含关于客户端正在请求的信息。请求行的第一部分表示正在使用的方法，例如 `GET` 或 `POST`，它描述了客户端如何发起这个请求。我们的客户端使用了 `GET` 请求，这意味着它在请求信息。

请求行的下一部分是 _/_，表示客户端请求的*统一资源标识符（uniform resource identifier，URI）*：URI 几乎但不完全等同于*统一资源定位符（uniform resource locator，URL）*。URI 和 URL 的区别对于本章的目的来说并不重要，但 HTTP 规范使用了术语 *URI*，因此我们这里可以在心里将 _URL_ 替换为 _URI_。

最后一部分是客户端使用的 HTTP 版本，然后请求行以 CRLF 序列结束。（*CRLF* 代表*回车（carriage return）*和*换行（line feed）*，这是打字机时代的术语！）CRLF 序列也可以写成 `\r\n`，其中 `\r` 是回车，`\n` 是换行。*CRLF 序列*将请求行与请求数据的其余部分分开。请注意，当 CRLF 被打印时，我们看到的是新行开始，而不是 `\r\n`。

查看我们到目前为止运行程序接收到的请求行数据，我们看到 `GET` 是方法，_/_ 是请求 URI，`HTTP/1.1` 是版本。

在请求行之后，从 `Host:` 开始的剩余行是头部（headers）。`GET` 请求没有主体。

尝试从不同的浏览器发起请求，或者请求不同的地址，例如 _127.0.0.1:7878/test_，看看请求数据如何变化。

既然我们知道浏览器在请求什么，让我们发送回一些数据！

### 编写响应

我们将实现发送数据以响应客户端请求。响应具有以下格式：

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

第一行是*状态行（status line）*，包含响应中使用的 HTTP 版本、总结请求结果的数值状态码（status code），以及提供状态码文本描述的原因短语（reason phrase）。在 CRLF 序列之后是任何头部、另一个 CRLF 序列，以及响应的主体（body）。

这里是一个示例响应，使用 HTTP 版本 1.1，状态码 200，OK 原因短语，没有头部，也没有主体：

```text
HTTP/1.1 200 OK\r\n\r\n
```

状态码 200 是标准的成功响应。这个文本是一个很小的成功 HTTP 响应。让我们将其写入流中，作为对成功请求的响应！从 `handle_connection` 函数中，删除打印请求数据的 `println!`，并替换为清单 21-3 中的代码。

<Listing number="21-3" file-name="src/main.rs" caption="向流中写入一个微小的成功 HTTP 响应">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

第一个新行定义了 `response` 变量，保存成功消息的数据。然后，我们在 `response` 上调用 `as_bytes` 将字符串数据转换为字节。`stream` 上的 `write_all` 方法接受一个 `&[u8]` 并将这些字节直接发送到连接中。因为 `write_all` 操作可能失败，我们在任何错误结果上使用 `unwrap` 如之前一样。同样，在真正的应用程序中，你应该在这里添加错误处理。

通过这些更改，让我们运行代码并发出请求。我们不再向终端打印任何数据，因此除了 Cargo 的输出外，我们看不到任何输出。当你在 Web 浏览器中加载 _127.0.0.1:7878_ 时，应该会看到一个空白页面，而不是错误。你刚刚手动编写代码实现了接收 HTTP 请求和发送响应！

### 返回真实的 HTML

让我们实现返回不仅仅是空白页面的功能。在项目目录的根目录（而不是 _src_ 目录）中创建新文件 _hello.html_。你可以输入任何你想要的 HTML；清单 21-4 展示了一种可能性。

<Listing number="21-4" file-name="hello.html" caption="在响应中返回的示例 HTML 文件">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

这是一个带有标题和一些文本的最小 HTML5 文档。要在收到请求时从服务器返回此内容，我们将修改 `handle_connection`，如清单 21-5 所示，读取 HTML 文件，将其作为响应的主体添加，并发送出去。

<Listing number="21-5" file-name="src/main.rs" caption="将 *hello.html* 的内容作为响应的主体发送">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

我们将 `fs` 添加到 `use` 语句中，以将标准库的文件系统模块引入作用域。将文件内容读取为字符串的代码应该看起来很熟悉；我们在清单 12-4 中为 I/O 项目读取文件内容时使用过它。

接下来，我们使用 `format!` 将文件的内容作为成功响应的主体添加。为了确保有效的 HTTP 响应，我们添加了 `Content-Length` 头部，其值设置为响应主体的大小——在本例中是 `hello.html` 的大小。

使用 `cargo run` 运行此代码，然后在浏览器中加载 _127.0.0.1:7878_；你应该看到你的 HTML 被渲染出来了！

目前，我们忽略了 `http_request` 中的请求数据，无条件地发送回 HTML 文件的内容。这意味着如果你在浏览器中请求 _127.0.0.1:7878/something-else_，你仍然会收到相同的 HTML 响应。目前，我们的服务器非常有限，不像大多数 Web 服务器那样工作。我们希望根据请求自定义响应，并且只在对 _/_ 的正确格式请求时发送回 HTML 文件。

### 验证请求并有选择地响应

目前，无论客户端请求什么，我们的 Web 服务器都会返回 HTML 文件。让我们添加功能，在返回 HTML 文件之前检查浏览器是否在请求 _/_，如果浏览器请求其他内容，则返回错误。为此，我们需要修改 `handle_connection`，如清单 21-6 所示。这个新代码将收到的请求内容与我们已知的 _/_ 请求的样子进行比较，并添加 `if` 和 `else` 块来区别对待请求。

<Listing number="21-6" file-name="src/main.rs" caption="区别对待对 */* 的请求与其他请求">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

我们只查看 HTTP 请求的第一行，因此不将整个请求读取到向量中，而是调用 `next` 从迭代器中获取第一个元素。第一个 `unwrap` 处理 `Option`，如果迭代器没有元素则停止程序。第二个 `unwrap` 处理 `Result`，效果与清单 21-2 的 `map` 中添加的 `unwrap` 相同。

接下来，我们检查 `request_line` 是否等于对 _/_ 路径的 GET 请求的请求行。如果是，`if` 块返回我们 HTML 文件的内容。

如果 `request_line` *不*等于对 _/_ 路径的 GET 请求，意味着我们收到了其他请求。稍后我们将向 `else` 块添加代码以响应所有其他请求。

现在运行此代码，请求 _127.0.0.1:7878_；你应该会得到 _hello.html_ 中的 HTML。如果你发出任何其他请求，例如 _127.0.0.1:7878/something-else_，你会看到与运行清单 21-1 和清单 21-2 中的代码时类似的连接错误。

现在让我们将清单 21-7 中的代码添加到 `else` 块中，以返回状态码 404 的响应，表示未找到请求的内容。我们还将返回一些 HTML，用于在浏览器中渲染一个页面，向最终用户指示响应。

<Listing number="21-7" file-name="src/main.rs" caption="如果请求的不是 */*，则响应状态码 404 和错误页面">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

这里，我们的响应有一个状态行，状态码为 404，原因短语为 `NOT FOUND`。响应的主体将是文件 _404.html_ 中的 HTML。你需要在 _hello.html_ 旁边创建一个 _404.html_ 文件用于错误页面；同样，随意使用任何你想要的 HTML，或使用清单 21-8 中的示例 HTML。

<Listing number="21-8" file-name="404.html" caption="用于任何 404 响应发送回的页面示例内容">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

通过这些更改，再次运行你的服务器。请求 _127.0.0.1:7878_ 应返回 _hello.html_ 的内容，而任何其他请求（如 _127.0.0.1:7878/foo_）应返回 _404.html_ 中的错误 HTML。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="a-touch-of-refactoring"></a>

### 重构

目前，`if` 和 `else` 块有很多重复：它们都在读取文件并将文件内容写入流。唯一的区别是状态行和文件名。让我们通过将这些差异提取到单独的 `if` 和 `else` 行中（这些行将状态行和文件名的值赋给变量），使代码更简洁；然后，我们可以在读取文件和写入响应的代码中无条件地使用这些变量。清单 21-9 显示了替换掉大型 `if` 和 `else` 块后的结果代码。

<Listing number="21-9" file-name="src/main.rs" caption="重构 `if` 和 `else` 块，使其仅包含两种情况之间不同的代码">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

现在 `if` 和 `else` 块只返回元组中状态行和文件名的适当值；然后，我们使用解构来通过 `let` 语句中的模式将这两个值赋给 `status_line` 和 `filename`，如第 19 章所述。

之前重复的代码现在位于 `if` 和 `else` 块之外，并使用 `status_line` 和 `filename` 变量。这使得更容易看到两种情况之间的区别，并且意味着如果我们想要更改文件读取和响应写入的工作方式，我们只有一个地方需要更新代码。清单 21-9 中代码的行为将与清单 21-7 相同。

太棒了！我们现在有了一个用大约 40 行 Rust 代码编写的简单 Web 服务器，它用一个页面内容响应一个请求，并用 404 响应响应所有其他请求。

目前，我们的服务器在单线程中运行，意味着它一次只能处理一个请求。让我们通过模拟一些慢请求来检查这如何成为一个问题。然后，我们将修复它，使我们的服务器能够同时处理多个请求。
