# Codegen 选项
> （译者注：Codegen，即 Code generation ，代码生成 ，所以本文是在说各种代码生成选项。）

所有的这些选项都通过 `-C` 标签传递给 `rustc` ，是 "codegen" 的缩写。通过运行 `rustc -C help`

## ar

这个选项是被废弃的并且没什么用。

## code-model

这个选项让你可以选择要使用的代码模型。 \
代码模型对程序及其符号可能使用的地址范围进行了约束。 \
有了更小的地址范围，机器指令就可以使用更紧凑的寻址模式。

该具体范围依赖于目标体系结构及其可用的寻址模式。 \
对于 x86 体系结构，更多细节性的描述可以在 [System V 应用程序二进制接口](https://github.com/hjl-tools/x86-psABI/wiki/x86-64-psABI-1.0.pdf) 规范中找到。

该选项支持的值有：

- `tiny` - 微代码模型。
- `small` - 小代码模型。这是大多数所支持的目标的默认模式。
- `kernel` - 内核代码模式。
- `medium` - 中型代码模式。
- `large` - 大型代码模式。

也可以通过运行 `rustc --print code-models` 查找受支持的值（也就是平台可用的代码模型）。

## codegen-units

该标签控制将 crate 分隔进多少个代码生成单元，这个（代码生成单元数）大于 0 。

当一个 crate 被分割进多个代码生成单元，LLVM 就可以并行处理它们。提高并行性或许会加快编译时间（缩短编译耗时），但是也可能会产生更慢的代码。设置该标签为 1 可能会提升生成代码的性能，但是也可能会编译得更慢。

如果没有指明默认值，对于非增量构建默认值就是 16 。对于增量构建，默认值是 256，可以使缓存更加细粒度。

## control-flow-guard

该标签控制 LLVM 是否启用 Windows 的 [控制流守卫](https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard) 平台安全功能。该标签当前会忽略非 Windows 目标（即在非 Windows 平台上不会生效）。它使用下列值中的一个：

* `y`， `yes`， `on`， `checks`， 或者没有值： 启用控制流守卫。
* `nochecks`： 不进行运行时强制检查的情况下触发控制流守卫元数据（这理应只用于测试目的，因为它不提供安全强制）。
* `n`， `no`， `off`： 不启用控制流守卫（默认值）。

## debug-assertions

该标签让你可以打开或关闭 `cfg(debug_assertions)` [条件编译](https://doc.rust-lang.org/reference/conditional-compilation.html#debug_assertions)。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值 ：开启 debug-assertions。
* `n`， `no`， 或 `off` ： 禁用 debug-assertions。

如果没有指明（ debug-assertions ），仅在 [opt-level](#opt-level) 是 0 的时候开启 debug assertions 。

## debuginfo

该标签控制调试信息的生成。其采用以下值之一：

* `0`: 没有任何调试信息 （默认）。
* `1`: 仅在行表中。
* `2`: 完整的调试信息。

注意：[`-g` 标签][option-g-debug] 是 `-C debuginfo=2` 的别名。

## default-linker-libraries

该标签控制链接器是否包含其默认链接库。其采用以下值之一：

* `y`， `yes`， `on`， 或者没有值 ： 包含默认库（默认值）。
* `n`。 `no`， 或者 `off`： 除开默认库。

例如，对于 gcc flavor 链接器，这会向链接器传递 `-nodefaultlibs` 标签。

## embed-bitcode

该标签控制编译器是否将 LLVM 位码嵌入目标文件（object files）中。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值： 将位码放入 rlib （默认值）。
* `n`， `no`， 或 `off`： 省略 rlib 中的位码。

当 rustc 执行 LTO (link-time optimization ,链接时优化)时需要 LLVM 位码,在像 iOS 这样的目标平台上也需要。被嵌入的位码将会出现在由 rustc 生成的目标文件中，其名称由目标平台定义，大多数时候是 `.llvmbc` 。

如果你的编译实际上不需要位码（例如，如果你不针对 iOS 进行编译或不执行 LTO ），`-C embed-bitcode=no` 的使用可以大大缩短编译时间并且减少生成文件的大小.由于这些原因，Cargo 会尽可能使用 `-C embed-bitcode=no` 。同样，如果你直接用 rustc 进行构建，我们推荐你在不使用 LTO 的时候使用 `-C embed-bitcode=no` 。

如果结合 `-C lto`，`-C embed-bitcode=no` 将会导致 `rustc` 启动时就中止，因为该（标签）组合是无效的。

> **注意**： 如果你使用 LTO 构建 Rust 代码，你就可能甚至不需要打开 `embed-bitcode` 选项。你可能会想要使用 `-Clinker-plugin-lto` 来代替，它会完全跳过生成目标文件，并且用 LLVM 位码来简单替换目标文件。唯一的使用 `-Cembed-bitcode` 的目的是当你要生成同时使用和不使用 LTO 的 rlib 时，例如 Rust 标准库带有嵌入的位码，因为用户可以使用或不使用 LTO 进行链接。
> 
> 这也可能会让你想知道为什么该选项的默认值是 `yes` 。理由是在 1.45 中及更早版本中是这样的。在 1.45 中，将此选项默认值调整为关闭。

## extra-filename

该选项允许你将额外的数据放进每个输出文件。它需要一个字符串作为后缀添加到文件名中。更多信息请参阅 [`--emit` 标签][option-emit] 。

## force-frame-pointers

该标签可以强制栈帧指针的使用。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值： 强制启用栈帧指针。
* `n`， `no`， 或 `off`： 不强制启用栈帧指针，但这并不意味着移除栈帧指针。

如果没有强制启用栈帧指针，则默认行为取决于目标平台。

## force-unwind-tables

该标签强制栈回溯表的生成。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值： 强制生成栈回溯表。
* `n`， `no`， 或 `off`： 不强制生成栈回溯表。如果目标平台或 `-C panic=unwind` 需要栈回溯表，就会触发一个错误。

默认值如果没有指明则取决于目标平台。

## incremental

该标签允许你启用增量编译，这允许 `rustc` 在编译完 crate 之后保存信息，以便重新编译 crate 的时候可以重用，缩短重编译的时间。这将采用一个存放增量文件的目录的路径。

## inline-threshold

该选项使你可以设置内联函数的默认阈值。其值采用无符号整数。内联基于成本模型（ cost model ），较高的阈值将会允许更多的内联。

默认（阈值）取决于 [opt-level](#opt-level):

| opt-level | Threshold |
|-----------|-----------|
| 0         | N/A, 仅内联强制内联函数（ always-inline functions ） |
| 1         | N/A, 仅内联强制内联函数和 LLVM 生命周期内联函数 （ LLVM lifetime intrinsics ） |
| 2         | 225 |
| 3         | 275 |
| s         | 75 |
| z         | 25 |

## link-arg

该标签使你可以在链接器调用后附加一个额外的参数。

“附加” 是很重要的；你可以多次传递该标签以添加多个参数。

## link-args

该标签使你可以在链接器调用后附加多个额外的参数。该选项（后的参数）应该用空格分隔。

## link-dead-code

该标签控制链接器是否保留无效代码。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值： 保留无效代码。
* `n`， `no`， 或 `off`： 移除无效代码（默认值）。

该标签的一个有用的例子是当试图构造代码覆盖率指标的时候（译者注：也就是说试图保留无效代码，咳咳。。）。

## link-self-contained

在支持该标签的目标平台上，该标签控制链接器使用 Rust 附带的库和对象，还是使用系统中的库和对象。其采用以下值之一：

* 无值： 如果系统已经有了必要的工具， rustc 将使用启发式禁用 self-contained 模式。
* `y`， `yes`， `on`： 仅使用 Rust 附带的库/对象。
* `n`， `no`， 或 `off`： 取决于用户或链接器（是否）提供非 Rust 的 库/对象。

当检测失败或用户想要使用附带的库时，这可以覆盖大多数情况。

## linker

该标签控制 `rustc` 调用哪个链接器来链接代码。它接受链接器可执行文件的路径。如果该标签未指明，使用的链接器就会根据目标来推断。另外，请参阅 [linker-flavor](#linker-flavor) 标签 —— 指定链接器的另一种方法。

## linker-flavor

该标签控制 `rustc` 使用的链接器的样式。 如果给编译器附加 [`-C linker` 标签](#linker) ，那么链接器样式将从提供的值中推断出来。如果没有给出链接器，则会使用链接器样式来确定要使用的链接器。每个 `rustc` 目标都默认有一些链接器样式。有效的选项有：

* `em`： 使用 [Emscripten `emcc`](https://emscripten.org/docs/tools_reference/emcc.html).
* `gcc`： 使用 `cc` 可执行文件， 在许多系统上通常是 gcc 或 clang 。
* `ld`： 使用 `ld` 可执行文件
* `msvc`： 使用 Microsoft Visual Studio MSVC 的 `link.exe` 可执行文件。
* `ptx-linker`: 使用 Nvidia NVPTX GPGPU 所支持的 [`rust-ptx-linker`](https://github.com/denzp/rust-ptx-linker) 。
* `wasm-ld`： 使用可执行文件 [`wasm-ld`](https://lld.llvm.org/WebAssembly.html) ，
   一个用于 WebAssembly 的 LLVM `lld` 端口。
* `ld64.lld`： 对于 apple 的 `ld`，使用附加 [`-flavor darwin` 标签][lld-flavor] 的 LLVM `lld` 可执行文件。
* `ld.lld`: 对于 GNU binutil 的 `ld`，使用附加 [`-flavor gnu` 标签][lld-flavor] 的 LLVM `lld` 可执行文件。
* `lld-link`: 对于Microsoft 的 `link.exe`，使用附加 [`-flavor link` 标签][lld-flavor] 的 LLVM `lld` 可执行文件。

[lld-flavor]: https://lld.llvm.org/Driver.html

## linker-plugin-lto

该标签将 LTO 优化推迟到 链接器阶段。 更多细节请参阅 [linker-plugin-LTO](../linker-plugin-lto.md) 。 其采用以下值之一：

* `y`， `yes`， `on`， 或者无值：启用 linker plugin LTO 。
* `n`， `no`， 或 `off`： 禁用 linker plugin LTO （默认）。
* 一个链接器插件（ linker plugin ）的路径。

更具体地说，该标签将使得编译器将其通常的目标文件输出（ typical object file output ）替换为 LLVM 位码文件。 例如，使用 `-Clinker-plugin-lto` 生成的 rlib 会仍然有 `*.o` 文件在其中， 但实际上它们都是 LLVM 位码而非机器码。 预计本平台链接器可以加载这些 LLVM 位码文件并在链接时生成代码（通常在执行优化之后）。

注意， rustc 也可以读取由 `-Clinker-plugin-lto` 生成的它自己的目标文件。如果 rlib 只会在使用 `-Clto` 编译时使用，那么可以传递 `-Clinker-plugin-lto` 标签来加速编译，并避免生成不使用的目标文件。

## llvm-args

该标签可以用来将参数列表直接传递给 LLVM 。

该列表必须用空格来分隔。

传递 `--help` 来查看选项列表。

## lto

该标签控制是否 LLVM 用 [链接时间优化](https://llvm.org/docs/LinkTimeOptimization.html) 来产生更好的优化代码，使用完整的程序分析，以更长的链接时间为代价。其采用以下值之一：

* `y`， `yes`， `on`， `fat`， 或无值：执行 "fat" LTO ，即尝试将依赖图中的所有依赖跨 crate 执行优化。
* `n`， `no`， `off`： 禁用 LTO 。
* `thin`: 执行 ["thin" LTO](http://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html) 。这类似于 "fat" ，但会花费更少的时间（优化），同时仍然可以得到类似于 "fat" 的性能提升。

如果没有指定 `-C lto` ， 那么编译器就会尝试执行 "thin local LTO" ，即只在本地 crate 上跨其[代码生成单元](#codegen-units) 执行 "thin" LTO （which performs "thin" LTO on the local crate only across its [codegen units](#codegen-units) ）。 当未指定 `-C lto` 时，如果代码生成单元是 1 或优化被禁用（[`-C opt-level=0`](#opt-level)），则 LTO 将被禁用。即是：

* 当 `-C lto` 未指定：
  * `codegen-units=1`： 禁用 LTO 。
  * `opt-level=0`： 禁用 LTO。
* 当 `-C lto=true`：
  * `lto=true`： 16 个代码生成单元， 跨 crate 执行 fat LTO 。
  * `codegen-units=1` + `lto=true`： 1 个代码生成单元， 跨 crate 执行 fat LTO 。

跨语言 LTO 请参阅 [linker-plugin-lto](#linker-plugin-lto)。

## metadata

该选项使你可以控制用于符号修饰（symbol mangling）。其采用值为以空格分隔的字符串列表。修饰后的符号将会合并元数据的 hash 。例如，这可以在链接时用来区分相同 crate 的两个不同版本之间符号。

## no-prepopulate-passes

该标签告诉传递管理器（pass manager）使用一个空的传递列表而非通常的预填充了的传递列表。

## no-redzone

该标签允许你禁用 [the red zone](https://en.wikipedia.org/wiki/Red_zone_\(computing\)) 。 其采用以下值之一：

* `y`， `yes`， `on`，或无值：  禁用 the red zone 。
* `n`， `no`， 或 `off`： 启用 the red zone 。

如果该标签未指定，默认行为取决于目标系统。

## no-stack-check

该选项已被弃用，且不执行任何操作。

## no-vectorize-loops

该标签禁用 [循环矢量化](https://llvm.org/docs/Vectorizers.html#the-loop-vectorizer)（ loop vectorization ）。

## no-vectorize-slp

该标签禁用矢量化使用 [superword-level parallelism](https://llvm.org/docs/Vectorizers.html#the-slp-vectorizer) 。

## opt-level

该标签控制优化级别。

* `0`： 没有优化，同时打开 [`cfg(debug_assertions)`](#debug-assertions)（默认值）。
* `1`：基本的优化（basic optimizations）。
* `2`：一些优化（some optimizations）。
* `3`： 全部优化（all optimizations）。
* `s`：优化二进制大小（optimize for binary size）。
* `z`：优化二进制大小，同时关闭循环矢量化。

注意： [`-O` 标签][option-o-optimize] 是 `-C opt-level=2` 的一个别名。

默认值为 `0`。

## overflow-checks

该标签允许你控制 [运行时整数溢出](https://doc.rust-lang.org/reference/expressions/operator-expr.html#overflow)的行为。当 overflow-checks 被启用时，溢出时会发出一个 panic 。该标签采用以下值之一：

* `y`， `yes`， `on`， 或无值：开启 overflow checks 。
* `n`， `no`， or `off`： 禁用 overflow checks 。

倘若未指定 overflow checks，如果 [debug-assertions](#debug-assertions) 是启用的 overflow checks 才会启用，否则是禁用的。

## panic

该标签使你在代码 panic 时控制（程序栈）行为情况。

* `abort`：在 panic 时终止进程。
* `unwind`：在 panic 时对栈进行展开。

如果未指定，默认值取决于目标。

## passes

该标签可以用于添加额外的 [LLVM passes](http://llvm.org/docs/Passes.html) 到编译中。

列表必须以空格分隔。

另请参阅 [`no-prepopulate-passes`](#no-prepopulate-passes) 标签。

## prefer-dynamic

默认情况下，`rustc` 更倾向于静态链接依赖项。该选项表明，如果库的静态和动态版本都是可用的，则应尽可能使用动态链接。有一个内部算法用于确定是否可以静态或动摇地链接依赖项。例如， `cdylib` crate 类型只能使用静态链接。该标采用以下值之一：

* `y`， `yes`， `on`， 或无值： 使用动态链接。
* `n`， `no`， 或 `off`： 使用静态链接（默认值）。

## profile-generate

该标签允许创建用于收集分析数据的仪器化二进制文件（instrumented binaries），用于 PGO 。该标签采用一个可选参数，该参数是一个目录的路径，仪器化二进制文件将会把收集到的数据发送到该目录中。更多信息请参阅 [PGO][profile-guided optimization] 章节。

## profile-use

该标签指定用于 PGO 的分析数据文件。该标签采用一个强制参数，它是一个有效的 `.profdata` 文件的路径。更多信息请参阅 [PGO][profile-guided optimization] 章节。

## relocation-model

该选项控制 [地址无关代码 (PIC)](https://en.wikipedia.org/wiki/Position-independent_code) 的生成。

该选项支持的值有：

#### Primary relocation models

- `static` - 不可重定位代码，机器指令得使用绝对寻址模式。

- `pic` - 完全可重定位独立代码，机器指令需要使用相对寻址模式。  \
等效于在其它编译器 “大写” `-fPIC` 或 `-fPIE` 选项，取决于产生的 crate 类型。  \
这是大多数受支持目标的默认 model。

#### Special relocation models

- `dynamic-no-pic` - 可重定位外部引用，不可重定位代码。  \
仅对 Darwin 操作系统有用，并且很少使用  \
如果 StackOverflow 告诉你将其作为 PIC 或 PIE 的退出选项， 不要相信它，应该使用 `-C relocation-model=static` 。
- `ropi`， `rwpi` 和 `ropi-rwpi` - 可重定位代码和只读数据， 可重定位的读写数据，
以及两者的组合。  \
只对特定的 嵌入式 ARM 目标有意义。.
- `default` -重定位默认模型到当前目标  \
仅有的意义是作为对先前在命令行上设置的其他一些明确指定的重定位模型的替代。

也可以通过运行 `rustc --print relocation-models` 查找支持的值。

#### Linking effects

除了代码生成效果之外， `relocation-model` 在链接期间也有效果。

如果重定位模型是 `pic` ，并且当前目标支持地址无关可执行文件（PIE），则将指示链接器生成一个（地址无关可执行文件）。  \
如果目标不支持地址无关且静态链接的可执行文件，那么 `-C target-feature=+crt-static` 会“超过” `-C relocation-model=pic`，并指示链接器（`-static`）生成静态链接但不是地址无关的可执行文件。

## remark

该标签允许你打印优化传递命令的备注。

传递命令的列表应该用空格分隔。

（如果是）`all` 将会对每个传递命令备注。

## rpath

该标签控制是否启用 [`rpath`](https://en.wikipedia.org/wiki/Rpath) 。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值： 启用 rpath 。
* `n`， `no`， 或 `off`： 禁用 rpath （默认值）。

## save-temps

该标签控制在编译完成后是否删除编译过程中生成的临时文件。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值：保存临时文件。
* `n`， `no`， 或 `off`：删除临时文件（默认值）。

## soft-float

该选项控制是否 `rustc` 生成模拟软件中浮点指令的代码。其采用以下值之一：

* `y`， `yes`， `on`， 或者无值： 使用软浮点数。
* `n`， `no`， 或 `off`： 使用硬件浮点数（默认值）。

## target-cpu

这会指示 `rustc` 生成专用于特定处理器的代码。

你可以运行 `rustc --print target-cpus` 来查看此处有效的传递选项。每个目标都有一个默认的基础 CPU 。 特殊值包括：

* `native` 可以传递给使用该主机处理器（同样的机器）。
* `generic` 指具有最少的 features 但经过现代调优的一个 LLVM 目标。

## target-feature

各个目标会支持不同的 features ； 该标签使你可以控制启用或禁用 feature 。 每个 feature 应该以一个 `+` 或 `-` 来做前缀来启用或禁用 feature 。

多个 features 通过 `-C target-feature` 选项的是可组合的。 \
多个 features 可以在单个选项中使用逗号分隔符指定—— `-C target-feature=+x,-y`。 \
如果某些 feature 同时使用 `+` 和 `-` 指定了一次或多次，那么后面传递的值会覆盖前面传递的值。 \
例如， `-C target-feature=+x,-y,+z -Ctarget-feature=-x,+y`
等价于 `-C target-feature=-x,+y,+z`.

要查看有效的选项和使用示例，请运行 `rustc --print
target-features`。

使用该标签是不安全的，可能会导致 [未定义的运行时行为](../targets/known-issues.md).

同样请参阅 [`target_feature` attribute](../../reference/attributes/codegen.md#the-target_feature-attribute) 控制每个函数的 features 。

这也支持 `+crt-static` 和 `-crt-static` feature 来控制
[静态 C 运行时链接](../../reference/linkage.html#static-and-dynamic-c-runtimes).

每个 target 和 [`target-cpu`](#target-cpu) 都有一组默认启用的 features 。

## tune-cpu

这会指示 `rustc` 为特定处理器调度代码。这不会影响兼容性（指令集或 ABI），但应该会使得代码在所选定的 CPU 上更有效率。

有效的选项与 [`target-cpu`](#target-cpu) 的相同。默认值为 `None`，  LLVM 将其转化为 `target-cpu` 。

这是个不稳定的选项。使用 `-Z tune-cpu=machine` 来指定值。

由于 LLVM （12.0.0-git9218f92）的限制，此选项当前仅针对 x86 目标有效。

[option-emit]: ../command-line-arguments.md#option-emit
[option-o-optimize]: ../command-line-arguments.md#option-o-optimize
[profile-guided optimization]: ../profile-guided-optimization.md
[option-g-debug]: ../command-line-arguments.md#option-g-debug
