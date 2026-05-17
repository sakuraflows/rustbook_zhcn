## 通过自定义命令（Custom Commands）扩展 Cargo

Cargo 的设计允许你使用新的子命令扩展它，而无需修改它本身。如果你的 `$PATH` 中有一个名为 `cargo-something` 的二进制文件，你可以通过运行 `cargo something` 来运行它，就像它是 Cargo 的子命令一样。当你运行 `cargo --list` 时，也会列出这样的自定义命令。能够使用 `cargo install` 安装扩展，然后像内置 Cargo 工具一样运行它们，是 Cargo 设计的一个超级方便的益处！

## 总结

使用 Cargo 和 [crates.io](https://crates.io/)<!-- ignore --> 共享代码是使 Rust 生态系统对许多不同任务有用的部分原因。Rust 的标准库小巧且稳定，但 crate 易于共享、使用和改进，其迭代周期与语言本身不同。不要羞于在 [crates.io](https://crates.io/)<!-- ignore --> 上分享对你有用的代码；很可能它对其他人也有用！
