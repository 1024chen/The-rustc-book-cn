# Targets

`rustc` 默认情况下是一个交叉编译器。这意味着你可以使用任意平台的编译器构建任意体系架构（的程序）。 *targets* 清单列出了你可以构建的可能的体系架构目标。

要查看你可以为构建目标设置的所有选项，请[查看](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_target/spec/struct.Target.html)文档。

使用 `--target` 标签将代码编译为特定目标：

```bash
$ rustc src/main.rs --target=wasm32-unknown-unknown
```
## Target Features
`x86` 和 `ARMv8` 是两种流行的 CPU 体系架构。 它们的指令集是大多数 CPU 的通用基准。然而，一些 CPU 在其上扩展了自定义指令集来扩展功能，例如：用于向量（ `AVX` ），位操作（ `BMI` ）或是加密（ `AES` ）。

知道编译的代码将在哪个 CUP 上运行的开发人员，可以通过运行 `-C target-feature=val` 标签添加（或删除） CPU 特定指令集。

请注意，该标签通常被认为是不安全的，更多细节可在[本章](known-issues.md)中找到。
