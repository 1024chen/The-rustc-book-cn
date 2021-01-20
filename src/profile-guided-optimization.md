# Profile Guided Optimization

`rustc` 支持使用 profile-guided optimization (PGO)。（译者注：该名词有时被译作配置文件引导优化/剖面引导优化，意思虽有契合但容易误导，此处以不译为好。）
本章描述 PGO 是什么，有什么优点，以及如何使用。

## 什么是 PGO?

PGO 的基本概念是收集关于一个程序典型的执行数据 （例如，它可能会执行的哪些分支）然后使用该数据来进行如内联，机器码布局，寄存器分配等告知优化。

有多种收集程序执行相关数据的办法。
一种是在分析器中运行程序 （例如 `perf`） 另一种是创建仪器化（instrumented）二进制文件，即已经内置了数据收集的二进制文件，并运行它。后者通常提供的数据更准确，这也是 `rustc` 所支持的。

## 用法

生成 PGO优化程序涉及到以下四个工作流程：:

1. 在开启检测工具的情况下编译程序(例如，`rustc -Cprofile-generate=/tmp/pgo-data main.rs`)
2. 运行仪器化程序 (例如 `./main`) 生成 `default_<id>.profraw` 文件。
3. 使用 LLVM's `llvm-profdata` 工具转化 `.profraw` 文件为 `.profdata` 文件。
4. 再次编译程序，此次使用分析数据(例如 `rustc -Cprofile-use=merged.profdata main.rs`)

一个仪器化程序将会创建一个或多个 `.profraw` 文件，每个 `.profraw` 文件对应一个仪器化程序 。例如，一个加载了两个仪器化动态库的仪器化可执行文件在运行时会生成三个 `.profraw`文件。另一方面，多次运行一个仪器化二进制文件，将会重新使用各自的 `.profraw` 文件，并在适当的地方更新它们。

这些 `.profraw` 文件必须被后期处理后才能被返回到编译器中，这是由 `llvm-profdata` 工具来完成的。该工具可以很轻易地通过以下命令安装：

```bash
rustup component add llvm-tools-preview
```

注意，安装 `llvm-tools-preview` 组件 不会将 `llvm-profdata` 添加到 `PATH` 中。而是可以在以下位置找到此工具：

```bash
~/.rustup/toolchains/<toolchain>/lib/rustlib/<target-triple>/bin/
```

或者 `llvm-profdata` 通常也可以使用最新的 LLVM 或 Clang 版本。

`llvm-profdata` 工具将多个 `.profraw` 文件合成单个的 `.profdata`文件然后就可以通过 `-Cprofile-use` 将其反馈给编译器：

```bash
# STEP 1: 使用工具编译二进制文件。
rustc -Cprofile-generate=/tmp/pgo-data -O ./main.rs

# STEP 2: 运行数次二进制文件，可以使用通用的参数集。
#         每次运行将会创建或更新在 /tmp/pgo-data 下的 `.profraw` 文件
./main mydata1.csv
./main mydata2.csv
./main mydata3.csv

# STEP 3: 合并和处理所有 /tmp/pgo-data 下的 `.profraw` 文件 
llvm-profdata merge -o ./merged.profdata /tmp/pgo-data

# STEP 4: 使用优化过程中合并的 `.profdata` 文件。所有 `rustc`标签
#         必须是相同的
rustc -Cprofile-use=./merged.profdata -O ./main.rs
```

### 完整的 Cargo 工作流程

在 Cargo 上使用此功能与直接使用 `rustc`非常相似。同样我们会生成一个仪表二进制文件 ，运行其以生成数据，合并数据，并将其反馈给编译器。 一些注意事项如下：

- 我们使用 `RUSTFLAGS` 环境变量传递 PGO 编译器标签给程序中所有 crate 的编译。

- 我们通过传递 `--target` 标签给 Cargo以防止 `RUSTFLAGS` 环境变量被传递给 Cargo 构建脚本。我们可不希望构建生成一堆 `.profraw` 文件的脚本。

- 我们通过传递 `--release` 给 Cargo 因为这才是 PGO 最有意义的地方（译者注：也就是说PGO优化本应该用在构建发行版本程序）。
  理论上， PGO 也可以在调试构建（debug builds）时进行，但没理由这么做。

- `-Cprofile-generate` 和变量 `-Cprofile-use` 推荐使用 *绝对路径* 。Cargo 可以在不同工作目录调用 `rustc` ， 这意味着 `rustc` 可能会无法找到提供的 `.profdata` 文件。如果使用绝对路径这将不是问题。

- 确保不会遗留之前编译会话中的分析数据是一种较好实践（good practice）。仅删除（分析数据所在）目录是一种简单的办法 (参考下面的 `STEP 0` )。

整个工作流程如下：

```bash
# STEP 0: 确保没有遗留之前运行产生的分析数据
rm -rf /tmp/pgo-data

# STEP 1: 构建仪器化二进制文件
RUSTFLAGS="-Cprofile-generate=/tmp/pgo-data" \
    cargo build --release --target=x86_64-unknown-linux-gnu

# STEP 2: 可使用一些经典数据
./target/x86_64-unknown-linux-gnu/release/myprogram mydata1.csv
./target/x86_64-unknown-linux-gnu/release/myprogram mydata2.csv
./target/x86_64-unknown-linux-gnu/release/myprogram mydata3.csv

# STEP 3: 合并 `.profraw` 文件进一个 `.profdata` 文件
llvm-profdata merge -o /tmp/pgo-data/merged.profdata /tmp/pgo-data

# STEP 4: 使用 `.profdata` 文件指导优化
RUSTFLAGS="-Cprofile-use=/tmp/pgo-data/merged.profdata" \
    cargo build --release --target=x86_64-unknown-linux-gnu
```

### 故障排除

- 建议在 `-Cprofile-use` 阶段传递 `-Cllvm-args=-pgo-warn-missing-function`。如果找不到给定函数的分析数据，LLVM 默认 不会发出警告。启用此警告将使你更容易发现设置中的错误。

- 在 1.39版本以前的 Cargo 中有一个[已知的问题](https://github.com/rust-lang/cargo/issues/7416)，会阻止 PGO 的正常工作，当执行 PGO 优化的时候，请确保使用 Cargo 1.39 或更新的版本。

## 延伸阅读

`rustc` 的 PGO 支持完全依赖于 LLVM 对该特性的实现，相当于 Clang 通过 `-fprofile-generate` / `-fprofile-use` 标签提供的功能。对于任何想要使用 Rust 进行 PGO优化的人来说，Clang 文档的 [Profile Guided Optimization][clang-pgo] 章节会是次悦读。

[clang-pgo]: https://clang.llvm.org/docs/UsersManual.html#profile-guided-optimization
