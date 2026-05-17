<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## 从单线程到多线程服务器

目前，服务器将依次处理每个请求，这意味着在第一个连接处理完成之前，它不会处理第二个连接。如果服务器收到越来越多的请求，这种串行执行会变得越来越低效。如果服务器收到一个需要很长时间处理的请求，后续请求将不得不等待该长请求完成，即使新请求可以快速处理。我们需要修复这个问题，但首先让我们看一下这个问题的实际表现。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### 模拟慢请求

我们将看看一个处理缓慢的请求如何影响对我们当前服务器实现的其他请求。清单 21-10 实现了对 _/sleep_ 请求的处理，通过模拟慢响应使服务器在响应之前休眠五秒钟。

<Listing number="21-10" file-name="src/main.rs" caption="通过休眠五秒钟来模拟慢请求">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

我们已将 `if` 切换为 `match`，因为现在我们有三种情况。我们需要显式匹配 `request_line` 的切片以与字符串字面量值进行模式匹配；`match` 不会像相等方法那样自动进行引用和解引用。

第一个分支与清单 21-9 中的 `if` 块相同。第二个分支匹配对 _/sleep_ 的请求。当收到该请求时，服务器将在渲染成功的 HTML 页面之前休眠五秒钟。第三个分支与清单 21-9 中的 `else` 块相同。

你可以看到我们的服务器有多原始：真正的库会以更简洁的方式处理多个请求的识别！

使用 `cargo run` 启动服务器。然后，打开两个浏览器窗口：一个用于 _http://127.0.0.1:7878_，另一个用于 _http://127.0.0.1:7878/sleep_。如果你像之前一样多次输入 _/_ URI，你会看到它响应很快。但如果你输入 _/sleep_，然后加载 _/_，你会看到 _/_ 会一直等待，直到 `sleep` 完成了它的完整五秒休眠后才加载。

有几种技术可以用来避免请求在慢请求后面排队，包括使用我们在第 17 章中介绍的 async；我们要实现的一种是线程池（thread pool）。

### 使用线程池提高吞吐量

*线程池（thread pool）*是一组已经生成并准备好等待处理任务的线程。当程序收到新任务时，它会将池中的一个线程分配给该任务，该线程将处理这个任务。池中剩余的其他线程可以处理第一个线程正在处理时传入的任何其他任务。当第一个线程完成其任务的处理时，它会被返回到空闲线程池中，准备处理新任务。线程池允许你并发处理连接，从而提高服务器的吞吐量。

我们将池中的线程数量限制为一个较小的数值，以保护我们免受 DoS 攻击；如果我们的程序为每个传入的请求创建一个新线程，那么有人向我们的服务器发起 1000 万个请求可能会耗尽我们服务器的所有资源，并使请求处理陷入停顿。

因此，我们不会创建无限数量的线程，而是在池中有一个固定数量的线程等待。传入的请求被发送到池中进行处理。池将维护一个传入请求的队列。池中的每个线程都将从该队列中弹出一个请求，处理该请求，然后向队列请求下一个请求。通过这种设计，我们可以同时处理多达 _`N`_ 个请求，其中 _`N`_ 是线程数。如果每个线程都在响应一个长时间运行的请求，后续请求仍然可能在队列中堆积，但我们在达到那个临界点之前能够处理的长时间运行请求的数量增加了。

这种技术只是提高 Web 服务器吞吐量的众多方法之一。你可能探索的其他选项包括 fork/join 模型、单线程异步 I/O 模型和多线程异步 I/O 模型。如果你对这个主题感兴趣，可以阅读更多关于其他解决方案的内容并尝试实现它们；使用像 Rust 这样的底层语言，所有这些选项都是可能的。

在开始实现线程池之前，让我们谈谈使用线程池应该是什么样子。当你尝试设计代码时，首先编写客户端接口可以帮助指导你的设计。编写代码的 API，使其按照你希望调用的方式结构化；然后，在该结构内实现功能，而不是先实现功能再设计公有 API。

类似于我们在第 12 章的项目中使用测试驱动开发的方式，这里我们将使用编译器驱动开发。我们将编写调用我们想要的函数的代码，然后查看编译器的错误，以确定下一步应该更改什么才能使代码工作。不过在此之前，我们将探讨我们不打算使用的技术作为起点。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### 为每个请求生成一个线程

首先，让我们探讨如果为每个连接创建一个新线程，代码会是什么样子。如前所述，由于可能生成无限数量的线程的问题，这并非我们的最终计划，但它是让一个多线程服务器先工作的起点。然后，我们将添加线程池作为改进，对比这两种解决方案会更容易。

清单 21-11 显示了对 `main` 所做的更改，以便在 `for` 循环中生成一个新线程来处理每个流。

<Listing number="21-11" file-name="src/main.rs" caption="为每个流生成一个新线程">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

如你在第 16 章中所学，`thread::spawn` 将创建一个新线程，然后在新线程中运行闭包中的代码。如果你运行此代码，在浏览器中加载 _/sleep_，然后在另外两个浏览器标签页中加载 _/_，你会看到对 _/_ 的请求不必等待 _/sleep_ 完成。然而，正如我们提到的，这最终会使系统不堪重负，因为你会无限制地创建新线程。

你可能还从第 17 章中记得，这正是 async 和 await 真正发光发热的情况！在我们构建线程池时请记住这一点，并思考如果使用 async，事情会有什么不同或相同之处。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### 创建有限数量的线程

我们希望线程池以类似且熟悉的方式工作，以便从线程切换到线程池不需要对使用我们 API 的代码进行大量更改。清单 21-12 展示了我们想要用来替代 `thread::spawn` 的 `ThreadPool` 结构体的假设接口。

<Listing number="21-12" file-name="src/main.rs" caption="我们理想的 `ThreadPool` 接口">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

我们使用 `ThreadPool::new` 创建一个具有可配置数量线程的新线程池，这里指定为四个。然后，在 `for` 循环中，`pool.execute` 具有与 `thread::spawn` 类似的接口，它接受一个闭包，池应该为每个流运行该闭包。我们需要实现 `pool.execute`，使其接受闭包并将其交给池中的一个线程运行。这段代码还不能编译，但我们会尝试，以便编译器可以指导我们如何修复它。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### 使用编译器驱动开发构建 `ThreadPool`

将清单 21-12 中的更改应用到 _src/main.rs_，然后让我们使用 `cargo check` 的编译器错误来驱动我们的开发。这是我们得到的第一个错误：

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

太好了！这个错误告诉我们，我们需要一个 `ThreadPool` 类型或模块，所以我们现在就构建一个。我们的 `ThreadPool` 实现将独立于我们的 Web 服务器所执行的工作类型。因此，让我们将 `hello` crate 从二进制 crate 转换为库 crate，以保存我们的 `ThreadPool` 实现。当我们改为库 crate 后，我们还可以将单独的线程池库用于任何我们想要使用线程池的工作，而不仅仅是处理 Web 请求。

创建一个 _src/lib.rs_ 文件，其中包含以下内容，这是我们现在能拥有的最简单的 `ThreadPool` 结构体定义：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

然后，编辑 _main.rs_ 文件，通过将以下代码添加到 _src/main.rs_ 顶部，将 `ThreadPool` 从库 crate 引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

这段代码仍然不能工作，但让我们再次检查以获取我们需要处理的下一个错误：

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

这个错误表明，接下来我们需要为 `ThreadPool` 创建一个名为 `new` 的关联函数。我们也知道 `new` 需要有一个可以接受 `4` 作为参数的参数，并应返回一个 `ThreadPool` 实例。让我们实现最简单的具有这些特征的 `new` 函数：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

我们选择 `usize` 作为 `size` 参数的类型，因为我们知道负数个线程没有意义。我们也知道我们将使用这个 `4` 作为线程集合中的元素数量，这正是 `usize` 类型的用途，如第 3 章中的["整数类型"][integer-types]<!-- ignore -->部分所述。

让我们再次检查代码：

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

现在错误发生，因为我们没有 `ThreadPool` 上的 `execute` 方法。回想一下["创建有限数量的线程"](#creating-a-finite-number-of-threads)<!-- ignore -->部分，我们决定线程池应该有一个类似于 `thread::spawn` 的接口。此外，我们将实现 `execute` 函数，使其接受给定的闭包并将其交给池中的一个空闲线程来运行。

我们将在 `ThreadPool` 上定义 `execute` 方法，接受一个闭包作为参数。回顾第 13 章中的["将捕获的值移出闭包"][moving-out-of-closures]<!-- ignore -->，我们可以使用三种不同的 trait 将闭包作为参数：`Fn`、`FnMut` 和 `FnOnce`。我们需要决定在这里使用哪种闭包。我们知道最终会做与标准库 `thread::spawn` 实现类似的事情，因此我们可以查看 `thread::spawn` 签名对其参数有哪些约束。文档向我们显示以下内容：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`F` 类型参数是我们这里关心的；`T` 类型参数与返回值有关，我们不关心它。我们可以看到 `spawn` 使用 `FnOnce` 作为 `F` 上的 trait 约束。这可能也是我们想要的，因为我们最终会将 `execute` 中获得的参数传递给 `spawn`。我们可以进一步确信 `FnOnce` 是我们想用的 trait，因为运行请求的线程只会执行该请求的闭包一次，这与 `FnOnce` 中的 `Once` 匹配。

`F` 类型参数也有 trait 约束 `Send` 和生命周期约束 `'static`，这些在我们的场景中很有用：我们需要 `Send` 来将闭包从一个线程转移到另一个线程，以及 `'static` 因为我们不知道线程需要多长时间来执行。让我们在 `ThreadPool` 上创建一个 `execute` 方法，该方法将接受具有这些约束的泛型参数 `F`：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

我们仍然在 `FnOnce` 后面使用 `()`，因为这个 `FnOnce` 表示一个不带参数并返回单元类型 `()` 的闭包。就像函数定义一样，返回类型可以从签名中省略，但即使我们没有参数，我们仍然需要圆括号。

同样，这是 `execute` 方法最简单的实现：它什么也不做，但我们只是试图使我们的代码编译。让我们再次检查：

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

它编译了！但请注意，如果你尝试 `cargo run` 并在浏览器中发出请求，你会看到本章开头我们在浏览器中看到的错误。我们的库实际上还没有调用传递给 `execute` 的闭包！

> 注意：关于具有严格编译器的语言（如 Haskell 和 Rust），你可能听到过一种说法："如果代码能编译，它就能工作。"但这个说法并非普遍正确。我们的项目能编译，但它什么也不做！如果我们在构建一个真实的、完整的项目，现在正是开始编写单元测试以检查代码是否编译*并且*具有我们想要的行为的好时机。

思考一下：如果我们执行 future 而不是闭包，这里会有什么不同？

#### 在 `new` 中验证线程数量

我们没有对 `new` 和 `execute` 的参数做任何操作。让我们用我们想要的行为来实现这些函数的主体。首先，让我们考虑一下 `new`。之前我们为 `size` 参数选择了无符号类型，因为负数个线程的池没有意义。然而，零个线程的池也同样没有意义，但零是完全有效的 `usize`。我们将添加代码来检查 `size` 是否大于零，然后再返回 `ThreadPool` 实例，如果收到零，我们将使用 `assert!` 宏使程序 panic，如清单 21-13 所示。

<Listing number="21-13" file-name="src/lib.rs" caption="如果 `size` 为零，实现 `ThreadPool::new` 使其 panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

我们还为我们的 `ThreadPool` 添加了一些带有文档注释的文档。请注意，我们遵循了良好的文档实践，添加了一个章节来指出我们的函数可能 panic 的情况，如第 14 章所述。尝试运行 `cargo doc --open` 并单击 `ThreadPool` 结构体，以查看为 `new` 生成的文档是什么样子！

除了像这里一样添加 `assert!` 宏，我们也可以将 `new` 改为 `build` 并返回 `Result`，就像我们在第 12 章 I/O 项目的清单 12-9 中对 `Config::build` 所做的那样。但我们在这个案例中决定，尝试创建一个没有任何线程的线程池应该是一个不可恢复的错误。如果你有雄心壮志，尝试编写一个名为 `build` 的具有以下签名的函数，与 `new` 函数进行比较：

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### 创建空间来存储线程

现在我们有了知道在池中存储有效数量线程的方式，我们可以创建这些线程并将它们存储到 `ThreadPool` 结构体中，然后返回该结构体。但我们如何"存储"一个线程呢？让我们再看看 `thread::spawn` 的签名：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` 函数返回一个 `JoinHandle<T>`，其中 `T` 是闭包返回的类型。让我们也尝试使用 `JoinHandle`，看看会发生什么。在我们的案例中，我们传递给线程池的闭包将处理连接并且不返回任何内容，因此 `T` 将是单元类型 `()`。

清单 21-14 中的代码将编译，但它还没有创建任何线程。我们更改了 `ThreadPool` 的定义，使其持有一个 `thread::JoinHandle<()>` 实例的向量，使用 `size` 的容量初始化了该向量，设置了一个 `for` 循环来运行一些创建线程的代码，并返回一个包含这些线程的 `ThreadPool` 实例。

<Listing number="21-14" file-name="src/lib.rs" caption="为 `ThreadPool` 创建一个向量来持有线程">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

我们将 `std::thread` 引入到库 crate 的作用域中，因为我们在 `ThreadPool` 中使用 `thread::JoinHandle` 作为向量中项目的类型。

一旦收到有效的大小，我们的 `ThreadPool` 将创建一个可以容纳 `size` 个项目的新向量。`with_capacity` 函数执行与 `Vec::new` 相同的任务，但有一个重要的区别：它在向量中预分配了空间。因为知道我们需要在向量中存储 `size` 个元素，提前进行这种分配比使用 `Vec::new`（它在插入元素时会调整自身大小）稍微高效一些。

当你再次运行 `cargo check` 时，它应该会成功。

<!-- 旧标题。请勿删除，否则链接可能失效。 -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### 将代码从 `ThreadPool` 发送到线程

我们在清单 21-14 的 `for` 循环中留下了一条关于创建线程的注释。这里，我们将看看如何实际创建线程。标准库提供了 `thread::spawn` 作为创建线程的方式，并且 `thread::spawn` 期望在创建线程时立即获得一些线程应该运行的代码。然而，在我们的情况下，我们希望创建线程并让它们*等待*我们稍后发送的代码。标准库的线程实现不包括任何方式来做这件事；我们必须手动实现它。

我们将通过引入一个新的数据结构（位于 `ThreadPool` 和线程之间）来管理这个新行为。我们将这个数据结构称为 *Worker*，这是池化实现中常用的术语。`Worker` 拾取需要运行的代码并在其线程中运行该代码。

可以把这想象成在餐厅厨房工作的人：工人们一直等侯，直到顾客的订单到达，然后他们负责取走这些订单并完成它们。

我们将不在线程池中存储 `JoinHandle<()>` 实例的向量，而是存储 `Worker` 结构体的实例。每个 `Worker` 将存储一个单独的 `JoinHandle<()>` 实例。然后，我们将在 `Worker` 上实现一个方法，该方法接受要运行的闭包代码并将其发送到已经运行的线程执行。我们还为每个 `Worker` 赋予一个 `id`，以便在记录日志或调试时能够区分池中不同的 `Worker` 实例。

以下是我们创建 `ThreadPool` 时将发生的新过程。在以这种方式设置 `Worker` 之后，我们将实现将闭包发送到线程的代码：

1. 定义一个 `Worker` 结构体，它持有一个 `id` 和一个 `JoinHandle<()>`。
2. 将 `ThreadPool` 改为持有一个 `Worker` 实例的向量。
3. 定义一个 `Worker::new` 函数，它接受一个 `id` 编号，并返回一个持有该 `id` 和以空闭包生成的线程的 `Worker` 实例。
4. 在 `ThreadPool::new` 中，使用 `for` 循环计数器生成一个 `id`，使用该 `id` 创建一个新的 `Worker`，并将 `Worker` 存储在向量中。

如果你准备好接受挑战，在查看清单 21-15 中的代码之前，尝试自己实现这些更改。

准备好了吗？这里是清单 21-15，其中包含进行上述修改的一种方法。

<Listing number="21-15" file-name="src/lib.rs" caption="修改 `ThreadPool` 以持有 `Worker` 实例，而不是直接持有线程">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

我们将 `ThreadPool` 上的字段名称从 `threads` 改为 `workers`，因为它现在持有的是 `Worker` 实例而不是 `JoinHandle<()>` 实例。我们使用 `for` 循环中的计数器作为 `Worker::new` 的参数，并将每个新 `Worker` 存储在名为 `workers` 的向量中。

外部代码（如我们在 _src/main.rs_ 中的服务器）不需要知道在 `ThreadPool` 内部使用 `Worker` 结构体的实现细节，因此我们将 `Worker` 结构体及其 `new` 函数设为私有。`Worker::new` 函数使用我们提供给它的 `id`，并存储一个通过使用空闭包生成新线程而创建的 `JoinHandle<()>` 实例。

> 注意：如果由于系统资源不足而无法创建线程，`thread::spawn` 将会 panic。这会导致整个服务器 panic，即使某些线程的创建可能成功。为简单起见，这种行为是可以接受的，但在生产线程池实现中，你可能想要使用 [`std::thread::Builder`][builder]<!-- ignore --> 及其返回 `Result` 的 [`spawn`][builder-spawn]<!-- ignore --> 方法。

这段代码将编译，并将存储我们指定为 `ThreadPool::new` 参数数量的 `Worker` 实例。但我们*仍然*没有处理在 `execute` 中获得的闭包。接下来让我们看看如何做到这一点。

#### 通过通道将请求发送到线程

我们接下来要解决的问题是，传递给 `thread::spawn` 的闭包绝对不做任何事情。目前，我们在 `execute` 方法中获得了要执行的闭包。但是我们需要在创建 `ThreadPool` 时创建每个 `Worker` 时，给 `thread::spawn` 一个闭包来运行。

我们希望刚刚创建的 `Worker` 结构体从 `ThreadPool` 持有的队列中获取要运行的代码，并将该代码发送到其线程运行。

我们在第 16 章学到的通道——一种在两个线程之间通信的简单方式——非常适合这种用例。我们将使用一个通道来充当任务队列，`execute` 将任务从 `ThreadPool` 发送到 `Worker` 实例，而 `Worker` 实例将任务发送到其线程。以下是计划：

1. `ThreadPool` 将创建一个通道并持有发送端（sender）。
2. 每个 `Worker` 将持有接收端（receiver）。
3. 我们将创建一个新的 `Job` 结构体，它将保存我们想要发送到通道中的闭包。
4. `execute` 方法将通过发送端发送它想要执行的任务。
5. 在其线程中，`Worker` 将循环处理其接收端，并执行它收到的任何任务的闭包。

让我们首先在 `ThreadPool::new` 中创建一个通道，并将发送端保存在 `ThreadPool` 实例中，如清单 21-16 所示。`Job` 结构体目前不持有任何东西，但将是我们通过通道发送的项目的类型。

<Listing number="21-16" file-name="src/lib.rs" caption="修改 `ThreadPool` 以存储传输 `Job` 实例的通道的发送端">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们创建新的通道，并且池持有发送端。这将成功编译。

让我们在创建通道时尝试将通道的接收端传递给每个 `Worker`。我们知道要在 `Worker` 实例生成的线程中使用接收端，因此我们将在闭包中引用 `receiver` 参数。清单 21-17 中的代码还不能编译。

<Listing number="21-17" file-name="src/lib.rs" caption="将接收端传递给每个 `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

我们做了一些小而直接的更改：我们将接收端传递到 `Worker::new` 中，然后在闭包内部使用它。

当我们尝试检查此代码时，会得到这个错误：

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

代码试图将 `receiver` 传递给多个 `Worker` 实例。这行不通，正如你从第 16 章回忆起的：Rust 提供的通道实现是多个*生产者（producer）*、单个*消费者（consumer）*。这意味着我们不能简单地克隆通道的消费端来修复此代码。我们也不希望向多个消费者多次发送消息；我们希望有一个消息列表，由多个 `Worker` 实例处理，每个消息只被处理一次。

此外，从通道队列中取出任务涉及修改 `receiver`，因此线程需要一种安全的方式来共享和修改 `receiver`；否则，我们可能会遇到竞争条件（如第 16 章所述）。

回想一下第 16 章中讨论的线程安全智能指针：为了跨多个线程共享所有权并允许线程修改该值，我们需要使用 `Arc<Mutex<T>>`。`Arc` 类型将允许多个 `Worker` 实例拥有接收端，而 `Mutex` 将确保一次只有一个 `Worker` 从接收端获取任务。清单 21-18 显示了我们需要的更改。

<Listing number="21-18" file-name="src/lib.rs" caption="使用 `Arc` 和 `Mutex` 在 `Worker` 实例之间共享接收端">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们将接收端放入 `Arc` 和 `Mutex` 中。对于每个新的 `Worker`，我们克隆 `Arc` 以增加引用计数，以便 `Worker` 实例可以共享接收端的所有权。

通过这些更改，代码编译了！我们快到了！

#### 实现 `execute` 方法

让我们最终实现 `ThreadPool` 上的 `execute` 方法。我们还将把 `Job` 从一个结构体改为一个类型别名（type alias），用于保存 `execute` 接收的闭包类型的 trait 对象。如第 20 章的["类型同义词与类型别名"][type-aliases]<!-- ignore -->部分所述，类型别名允许我们将长类型缩短以方便使用。请看清单 21-19。

<Listing number="21-19" file-name="src/lib.rs" caption="为持有每个闭包的 `Box` 创建 `Job` 类型别名，然后将任务通过通道发送">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

在使用 `execute` 中获得的闭包创建新的 `Job` 实例后，我们将该任务通过通道的发送端发送出去。我们在 `send` 上调用 `unwrap` 以处理发送失败的情况。例如，如果我们停止了所有线程的执行，意味着接收端已停止接收新消息，就可能会发生这种情况。目前，我们不能停止线程的执行：只要池存在，我们的线程就会继续执行。我们使用 `unwrap` 的原因是我们知道失败情况不会发生，但编译器不知道这一点。

但我们还没有完全完成！在 `Worker` 中，我们传递给 `thread::spawn` 的闭包仍然只*引用*了通道的接收端。相反，我们需要闭包永远循环，向通道的接收端请求一个任务，并在获得任务时运行它。让我们对 `Worker::new` 进行清单 21-20 中所示的更改。

<Listing number="21-20" file-name="src/lib.rs" caption="在 `Worker` 中永久循环，并在 `recv` 收到任务时执行该任务">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

这里，我们首先在 `receiver` 上调用 `lock` 以获取互斥锁（mutex），然后调用 `unwrap` 以在出现任何错误时 panic。获取互斥锁可能会失败，如果互斥锁处于一种称为*中毒（poisoned）*的状态，其他线程在持有锁时 panic 而没有释放锁就会发生这种情况。在这些情况下，直接调用 `unwrap` 让该线程 panic 是可行的行动方案。你可以随意将 `unwrap` 改为对你更有意义的 `expect`。

如果我们在互斥锁上获得锁，我们就调用 `recv` 从通道中接收一个 `Job`。最后的 `unwrap` 也会处理任何错误，如果持有发送端的线程已经关闭就可能发生这种情况，类似于当 `recv` 返回 `Err` 时 `join` 方法处理错误的方式。

调用 `recv` 会阻塞，因此如果还没有任务，当前线程将一直等待，直到有可用的任务。`Mutex<T>` 确保一次只有一个 `Worker` 线程尝试请求一个任务。

我们的线程池现在可以工作了！执行 `cargo run` 并发出一些请求：

```text
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `thread` is never read
  --> src/lib.rs:53:5
   |
53 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^
   |
   = note: `#[warn(dead_code)]` on by default

warning: field `id` is never read
  --> src/lib.rs:54:5
   |
54 |     id: usize,
   |     ^^
```

我们看到了关于 `Worker` 的 `id` 和 `thread` 字段未被直接使用的警告，这是一个提醒，我们实际上没有清理任何东西。当我们使用不那么优雅的 <kbd>ctrl</kbd>-<kbd>C</kbd> 方式停止主线程时，所有其他线程也会立即停止，即使它们正在处理请求。

接下来，我们将实现 `Drop` trait 来对池中的每个线程调用 `join`，以便它们在关闭之前可以完成正在处理的请求。然后，我们将实现一种方式来告诉线程它们应该停止接受新请求并关闭。

[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[type-aliases]: ch20-03-advanced-types.html#creating-type-synonyms-with-type-aliases
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
