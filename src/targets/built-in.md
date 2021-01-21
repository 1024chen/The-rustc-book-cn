# 内置 Targets

`rustc` 能够自动编译代码为多个目标，我们称之为“内置” （"built-in"） targets，他们通常与团队直接支持的目标相对应。 要查看内置 targets 的清单，可以运行 `rustc --print target-list`。


通常，目标 需要 Rust 标准库的编译副本才能工作。如果使用 [rustup]，想要了解如何下载由官方 Rust 发行版 构建的预构建标准库，请参阅关于
[交叉编译][rustup-cross] 的文档。大多数目标需要系统链接器 （system linker），还有可能需要其他的东西。

[rustup]: https://github.com/rust-lang/rustup
[rustup-cross]: https://github.com/rust-lang/rustup#cross-compilation
