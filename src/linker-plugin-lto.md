# Linker-plugin-LTO

`-C linker-plugin-lto` 标签允许将 LTO （译者注：Linking Technology and Order）优化推迟到实际链接步骤中，如果要链接的所有目标文件（object files） 都是基于 LLVM 工具链优化的，则此标签又允许跨编程语言边界执行过程间优化，本文示例主要展示如何将 Rust 代码与使用 Clang 编译的 C/C++ 代码链接在一起。

## 用法

链接基于LTO的插件有两种主要的情况:

 - 编译 Rust `staticlib` 用于 C ABI 的依赖项。
 - 在 `rustc` 调用链接器的地方编译 Rust 二进制文件。

在这两种情况下，Rust代码需要用 `-C linker-plugin-lto` 编译，并且 c/c++ 代码使用 `-flto` 或 `-flto=thin` 以便将目标文件生成 LLVM 位码。

### Rust `staticlib` 作为 C/C++ 程序中的依赖项

在这种情况下，Rust 编译器只需要确保 `staticlib` 中的目标文件格式正确即可。
要想进行链接，必须使用带有 LLVM 插件的链接器 （例如 LLD）。

直接使用 `rustc` ：

```bash
# 编译 Rust staticlib
rustc --crate-type=staticlib -Clinker-plugin-lto -Copt-level=2 ./lib.rs
# 用 `-flto=thin` 编译 C代码
clang -c -O2 -flto=thin -o main.o ./main.c
# 链接起来,得先确保我们使用的是合适的链接器。
clang -flto=thin -fuse-ld=lld -L . -l"name-of-your-rust-lib" -o main -O2 ./cmain.o
```

使用 `cargo`:

```bash
# 编译 Rust staticlib
RUSTFLAGS="-Clinker-plugin-lto" cargo build --release
# 用 `-flto=thin` 编译 C代码
clang -c -O2 -flto=thin -o main.o ./main.c
# 链接起来,得先确保我们使用的是合适的链接器。
clang -flto=thin -fuse-ld=lld -L . -l"name-of-your-rust-lib" -o main -O2 ./cmain.o
```

### C/C++ 代码作为 Rust 中的依赖项

在这种情况下，将由 `rustc` 调用链接器。我们再次必须确保使用合适的链接器。

直接使用 `rustc` ：

```bash
# 用 `-flto=thin` 编译 C代码
clang ./clib.c -flto=thin -c -o ./clib.o -O2
# 从 C代码中创建静态库
ar crus ./libxyz.a ./clib.o

# 使用其他参数一起调用 `rustc`
rustc -Clinker-plugin-lto -L. -Copt-level=2 -Clinker=clang -Clink-arg=-fuse-ld=lld ./main.rs
```

直接使用 `cargo`：

```bash
# 用 `-flto` 编译 C代码
clang ./clib.c -flto=thin -c -o ./clib.o -O2
# 从 C代码中创建静态库
ar crus ./libxyz.a ./clib.o

# 通过 RUSTFLAGS 设置链接参数
RUSTFLAGS="-Clinker-plugin-lto -Clinker=clang -Clink-arg=-fuse-ld=lld" cargo build --release
```

### 明确指定 `rustc` 要使用的链接器插件

如果想要使用 LLD 以外的链接器，需要明确指定使用的 LLVM 链接器插件。否则链接器无法读取目标文件。插件的路径通过作为 `-Clinker-plugin-lto` 选项的参数传递：

```bash
rustc -Clinker-plugin-lto="/path/to/LLVMgold.so" -L. -Copt-level=2 ./main.rs
```


## 工具链的兼容性

<!-- NOTE: 要更新下表，你可以使用以下 shell 脚本:

```sh
rustup toolchain install --profile minimal nightly
MINOR_VERSION=$(rustc +nightly --version | cut -d . -f 2)
LOWER_BOUND=44

llvm_version() {
    toolchain="$1"
    printf "Rust $toolchain    |    Clang "
    rustc +"$toolchain" -Vv | grep LLVM | cut -d ':' -f 2 | tr -d ' '
}

for version in `seq $LOWER_BOUND $((MINOR_VERSION - 2))`; do
    toolchain=1.$version.0
    rustup toolchain install --no-self-update --profile  minimal $toolchain >/dev/null 2>&1
    llvm_version $toolchain
done
```

-->

为了使这种 LTO 生效，LLVM 链接器插件必须能够处理由 `rustc` 和 `clang` 产生的 LLVM 位码。

通过使用基于相同版本的 LLVM 的 `rustc` 和 `clang` 实现可获得最佳结果。可以使用 `rustc -vV` 来查看给定 `rustc` 版本所使用的 LLVM。注意因为Rust有时会使用 LLVM 的不稳定版本，所以此处所给出的版本号只是一个近似值，但是这种近似通常是可靠的。

下表展示了已知工具链的良好组合。

| Rust 版本 | Clang 版本 |
|--------------|---------------|
| Rust 1.34    |    Clang 8    |
| Rust 1.35    |    Clang 8    |
| Rust 1.36    |    Clang 8    |
| Rust 1.37    |    Clang 8    |
| Rust 1.38    |    Clang 9    |
| Rust 1.39    |    Clang 9    |
| Rust 1.40    |    Clang 9    |
| Rust 1.41    |    Clang 9    |
| Rust 1.42    |    Clang 9    |
| Rust 1.43    |    Clang 9    |
| Rust 1.44    |    Clang 9    |
| Rust 1.45    |    Clang 10   |
| Rust 1.46    |    Clang 10   |

注意，此处的兼容性策略将来可能会更改。
