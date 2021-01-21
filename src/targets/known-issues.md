# 已知问题
本文将告知你一些已知的“陷阱” 。 请记住，本部分是（并且将始终是）不完整的。如有建议和修正，可以随时为本指南做 [贡献](https://doc.rust-lang.org/rustc/contributing.html) 。



## Target Features
当将启用了 target-feature 的代码和禁用 target-feature 的代码混合后，大多数 target-feature 出现了问题。如果想要避免未定义的行为，推荐用一组通用的 target-feature 构建所有的代码。  

默认情况下，用 `-C target-feature` 标签编译代码将不会重新编译整个标准库 和/或（ and/or ） 导入对应 target features 的 crate。因此， target features 通常被认为是不安全的，在单个函数上使用 `#[target_feature]` 属性会使得函数不安全。



例如：

| Target-Feature | 问题 | 所在架构 | 描述 | 细节 |
| -------------- | ----- | ------- | ----------- | ------- |
| `+soft-float` <br> 和 <br> `-sse` | 段错误和 ABI 不匹配 | `x86` 和 `x86-64` | `x86` 和 `x86_64` 架构将 SSE 寄存器 （又名  `xmm`）用于浮点运算。使用软件模拟浮点数 ("soft-floats") 会禁用 `xmm` 寄存器。但 Rust 部分核心库 (例如 `std::f32` 或 `std::f64`) 编译时并没有使用 soft-floats 并且期望参数能传递到 `xmm` 寄存器。 这就导致了 ABI 不匹配。<br><br>  尝试使用禁用的 SSE 编译也会导致此问题。 | [#63466](https://github.com/rust-lang/rust/issues/63466) |

