# 命令行参数

下面是 `rustc` 的命令行参数列表与操作。

<a id="option-help"></a>
## `-h`/`--help`: 获取帮助信息

该标签将会打印 `rustc`的帮助信息。

<a id="option-cfg"></a>
## `--cfg`: 配置编译环境

此标签会打开或关闭 `#[cfg]` 变量中的各种 [条件编译](../reference/conditional-compilation.md)设置值.

这个值可以是单标识符，也可以是以 `=` 分隔的双标识符.

例如, `--cfg 'verbose'` 或 `--cfg 'feature="serde"'`。 两者分别对应于 `#[cfg(verbose)]` 和 `#[cfg(feature = "serde")]` .

<a id="option-l-search-path"></a>
## `-L`: 将目录添加到库搜索路径

`-L` 标签添加了搜索外部crate和库的路径。

当 `KIND` 是以下情况之一时，可以使用 `-L
KIND=PATH` 这种形式指定搜索路径的类型：

- `dependency` — 仅在该目录中搜索传递依赖项。
- `crate` — 仅在该目录中搜索此crate的直接依赖项。
- `native` — 仅在该目录中搜索本地库。
- `framework` — 仅在该目录中搜索macOS框架。
- `all` — 搜索此目录中的所有库类型。如果 `KIND` 没有指定，这将是默认值.

<a id="option-l-link-lib"></a>
## `-l`: 将生成的crate链接到本地库

该标签允许您在构建 crate 时指定链接到特定的本地库。

当 `KIND` 是以下情况之一时，库的种类可以使用  `-l KIND=lib` 这种形式指定:

- `dylib` — 本地动态库。
- `static` — 本地静态库 (例如 `.a` archive文件)。（译者注：linux下的静态链接文件格式）
- `framework` — macOS框架。

可以用[`#[link]`][link-attribute]属性指定库的种类。 如果未在  `link` 属性或命令行中指定种类, 它将链接到可用动态库，否则将使用静态库。 如果在命令行中指定了库类型，其将会覆盖 `link` 属性指定的库类型。

`link` 属性中使用的名称可以使用形如 `-l
ATTR_NAME:LINK_NAME` 形式覆盖，其中  `ATTR_NAME` 是 `link` 属性中的名称，`LINK_NAME` 是将要链接到的实际库的名称。

[link-attribute]: ../reference/items/external-blocks.html#the-link-attribute

<a id="option-crate-type"></a>
## `--crate-type`: 编译器将要构建crate的类型列表

这将指示 `rustc` 以何种 crate type 去构建。该标签接接收逗号分隔的值列表，也可以多次指定。有效的 crate type 如下：

- `lib` — 编译器生成的首选库类型， 目前默认为 `rlib`。
- `rlib` — Rust 静态库。
- `staticlib` — 本地静态库。
- `dylib` — Rust 动态库。
- `cdylib` — 本地动态库。
- `bin` — 可执行程序。
- `proc-macro` — 生成格式化且编译器可加载的过程宏库。

可以使用 [`crate_type`][crate_type]属性来指定 crate 类型。 `--crate-type` 命令行的值会覆盖 `crate_type` 属性的值.

更多细节可以参阅 reference 中的 [链接章节][linkage chapter]

[linkage chapter]: ../reference/linkage.html
[crate_type]: ../reference/linkage.html

<a id="option-crate-name"></a>
## `--crate-name`: 指定正在构建的 crate 名称

这将告知 `rustc` 您的 crate 的名称。

<a id="option-edition"></a>
## `--edition`: 指定使用的语义版本

该标签值为 `2015` 或 `2018`。 默认为 `2015`。 更多关于版本的信息可以在 [版本指南][edition guide]中找到。

[edition guide]: ../edition-guide/introduction.html

<a id="option-emit"></a>
## `--emit`: 指定生成输出文件的类型

该标签控制编译器生成的输出文件的类型。其接收以逗号分隔的值列表，也可以多次指定。有效的生成类型有：

- `asm` — 生成在 crate 中的一个汇编代码文件。 默认的输出文件是 `CRATE_NAME.s`。
- `dep-info` — 生成一个包含Makefile语法的文件，指示加载以生成crate的所有源文件。 默认输出文件是 `CRATE_NAME.d`。
- `link` — 生成由 `--crate-type` 指定的 crates 。 默认输出文件取决于平台和 crate 类型。 如果未指定 `--emit` 这将是默认值。
- `llvm-bc` — 生成一个包含 [LLVM bitcode]的二进制文件。默认输出文件是 `CRATE_NAME.bc`。
- `llvm-ir` — 生成一个包含 [LLVM IR]（ LLVM  中间语言）的文件。默认的输出文件是 `CRATE_NAME.ll`。
- `metadata` — 生成一个关于该 crate 的元数据的文件。 默认输出文件是 `CRATE_NAME.rmeta`。
- `mir` — 生成一个包含 Rust 中级中间表示（即中级中间语言）的文件. 默认输出文件名是  `CRATE_NAME.mir`。
- `obj` — 生成一个本地对象文件，默认输出文件是
  `CRATE_NAME.o`。

输出文件名可以用 [`-o` flag](#option-o-output) 进行设置。使用[`-C extra-filename`
flag](codegen-options/index.md#extra-filename)可以添加文件名后缀。文件将被写入当前目录除非使用 [`--out-dir` flag](#option-out-dir) 标签。 每一个生成类型也可以使用 `KIND=PATH`的形式指定输出文件名，它优先于 `-o` 标签。

[LLVM bitcode]: https://llvm.org/docs/BitCodeFormat.html
[LLVM IR]: https://llvm.org/docs/LangRef.html

<a id="option-print"></a>
## `--print`: 打印编译器信息

该标签打印有关编译器的各种信息。 该标签可以多次指定，并且信息按标志指定的顺序打印。指定 `--print` 标签通常会禁用 [`--emit`](#option-emit) 步骤且只打印请求的信息。打印的有效值类型为：

- `crate-name` — crate的名称。
- `file-names` — 文件名由 `link` 命令执行的种类所决定。
- `sysroot` — 系统根目录路径。（译者注：此处指的是所使用rust根目录即.rustup下的toolchains文件夹）
- `target-libdir` - 目标 lib 文件夹路径（译者注：同上）。
- `cfg` — 条件编译值列表。 了解更多条件编译值信息，请参阅 [条件编译][conditional compilation]。
- `target-list` — 已知目标列表。 可以使用
 `--target` 标签选择目标。
- `target-cpus` — 当前目标的可用CPU值列表。可以使用 [`-C target-cpu=val`
  flag](codegen-options/index.md#target-cpu)标签选择目标。
- `target-features` — 当前目标的可用 目标  features 列表。目标 features 可以使用 [`-C target-feature=val`](codegen-options/index.md#target-feature) 标签启用。该标签是不安全（ unsafe ）的。 已知问题详细信息请参阅
  [known issues](targets/known-issues.md)。
- `relocation-models` — 重定位模型列表。重定位模型可以用 [`-C relocation-model=val`
  flag](codegen-options/index.md#relocation-model) 标签选择。
- `code-models` — 代码模型列表。代码模型可以用 [`-C code-model=val` flag](codegen-options/index.md#code-model)进行设置。
- `tls-models` — 支持的线程本地存储模型列表。 模型可以用 `-Z tls-model=val` 标签来选择。
- `native-static-libs` — 当创建一个 `staticlib` crate 类型时可以使用此选项。 如果这是唯一的标志，它将执行一个完整的编译，并包含一个指出链接生成静态库时要使用的链接器标签（flags）的诊断说明。该说明以文本 `native-static-libs:` 开始，以便更容易获取输出信息。

[conditional compilation]: ../reference/conditional-compilation.html

<a id="option-g-debug"></a>
## `-g`: 包含调试信息

[`-C debuginfo=2`](codegen-options/index.md#debuginfo) 的同义词。

<a id="option-o-optimize"></a>
## `-O`: 优化代码

[`-C opt-level=2`](codegen-options/index.md#opt-level) 的同义词。

<a id="option-o-output"></a>
## `-o`: 输出的文件名

该标签控制输出的文件名。

<a id="option-out-dir"></a>
## `--out-dir`: 将输出写入的目录

输出的crate将会被写入此目录。 如果使用 [`-o`](#option-o-output) 标签将会忽略此标签。

<a id="option-explain"></a>
## `--explain`: 提供错误消息的详细说明

`rustc` 对于每一个（检测到的）错误都会返回一个错误码； 这将打印给定错误的较长说明。

<a id="option-test"></a>
## `--test`: 构建测试工具

当编译此crate，`rustc` 将会忽略 `main` 函数并且生成测试工具。

<a id="option-target"></a>
## `--target`: 选择要构建目标的三要素

这将控制将要生成的 [target](targets/index.md)。

<a id="option-w-warn"></a>
## `-W`: 设置 lint 警告等级

该标签将会设置哪些 lint 的[警告等级](lints/levels.md#warn).

_注:_ 这些 lint 级别参数的顺序将被考虑进去，更多信息请参见[lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-a-allow"></a>
## `-A`: 设置 lint 许可等级

该标签将会设置哪些 lint 应该被设置的 [许可等级](lints/levels.md#allow)。

_Note:_ 这些 lint 级别参数的顺序， 更多信息请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-d-deny"></a>
## `-D`: 设置 lint 拒绝等级

该标签将会设置哪些 lint 应该被设置的 [拒绝等级](lints/levels.md#deny).

_Note:_ 这些 lint 级别参数的顺序， 更多信息请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-f-forbid"></a>
## `-F`: 设置 lint 禁止等级

该标签将会设置哪些 lint 应该被设置的 [forbid level](lints/levels.md#forbid).

_Note:_ 这些 lint 级别参数的顺序， 更多信息请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-z-unstable"></a>
## `-Z`: 设置不稳定选项

该标签允许你设置不稳定 rustc 选项。该标签可以被使用多次以设置多个选项。 例如： `rustc -Z verbose -Z time`。
使用 -Z 指定选项只能在 nightly 版本中可用。浏览所有可用选项可以运行： `rustc -Z help`。

<a id="option-cap-lints"></a>
## `--cap-lints`: 设置最严格的 lint 等级

该标签让你 限制 lints 的等级， 更多请 [参见](lints/levels.md#capping-lints)。

<a id="option-codegen"></a>
## `-C`/`--codegen`: 代码生成选项

该标签允许你设置 [生成选项](codegen-options/index.md)。

<a id="option-version"></a>
## `-V`/`--version`: 打印版本

该标签将会打印 `rustc` 的版本。

<a id="option-verbose"></a>
## `-v`/`--verbose`: 详细输出

该标签与其他标签结合使用时，会生成额外的输出信息。

<a id="option-extern"></a>
## `--extern`: 指定外部库的位置

该标签允许您传递直接依赖项的外部 crate 的名称和位置。 间接依赖项(依赖项的依赖项)
使用 [`-L`](#option-l-search-path) 标签来定位。给定的 crate 名称被添加到  [extern prelude]中， 类似于在根模块中指定 `extern crate` 。给定的 crate 名称不需要与库构建时使用的名称匹配。

要指出 `--extern` 与 `extern crate` 有一个行为差异： `--extern` 仅仅将 crate  成为链接的一个候选项；不经常指定是不会将其真正链接的。在少数情况下，你可能希望确保一个 crate 是被链接的，即使你不经常在你的代码中使用它：例如， 如果它改变了全局分配器（allocator），或者它包含' #[no_mangle] '符号以供其他编程语言使用，在这种情况下你需要使用 `extern crate`。

该标签可以多次指定。该标签接受下列格式之一的参数：

* `CRATENAME=PATH` — 指定在给定路径上可找到的 crate 。
* `CRATENAME` — 指定在搜索路径上可以被找到的 crate ，例如用 sysroot 或由 `-L`  标签。

同一个 crate 名称可以被不同 crate types 指定多次。如果同时找到 `rlib` 和 `dylib` （同名）文件， 则使用一个内部算法来决定使用哪一个进行链接。 [`-C prefer-dynamic`][prefer-dynamic] 标签可以被用来影响使用谁。

如果指定了一个带有路径和一个不带路径的同名 crate，则带路径的那个被使用且不带路径的那个无效。

[extern prelude]: ../reference/items/extern-crates.html#extern-prelude
[prefer-dynamic]: codegen-options/index.md#prefer-dynamic

<a id="option-sysroot"></a>
## `--sysroot`: 覆盖系统根目录

"sysroot" 是 `rustc` 寻找 Rust 发行 crate 的位置；该标签允许被重写。

<a id="option-error-format"></a>
## `--error-format`: 控制如何产生错误

该标签允许你控制消息的格式。 消息被打印到 stderr 。有效的选项有：

- `human` — 人可读的输出格式，此为默认选项。
- `json` — 结构化Json输出。 更多细节请参阅 [ json 章节][the JSON chapter]。
- `short` — 简短，单行信息。

<a id="option-color"></a>
## `--color`: 配置输出的颜色

该标签允许你配置输出的颜色。有效选项包括：

- `auto` — 如果颜色输出到终端（tty）使用颜色，这是默认选项。
- `always` — 始终使用颜色。
- `never` — 始终不对输出进行着色。

<a id="option-remap-path-prefix"></a>
## `--remap-path-prefix`: 输出中重映射源名称

在所有输出中重新映射源路径前缀，包括编译器诊断、调试信息、宏扩展等。它以  `FROM=TO` 形式接收一个值，其中 `FROM` 的路径前缀被重写为值 `TO` 。 `FROM` 本身可能包含 `=` 符号， 但是 `TO` 值不可能包含。该标签可以被多次指定。

这对于规范化构建产品是很有用的，例如，通过从发送到对象文件的路径名中删除当前目录。该替换是纯文本的，不考虑当前系统的路径名语法。例如 `--remap-path-prefix foo=bar` 将会匹配 `foo/lib.rs` 而不是 `./foo/lib.rs`。

<a id="option-json"></a>
## `--json`: 配置编译器打印json的消息

当 [`--error-format=json`](#option-error-format) 选项 传递到 rustc 时，编译器所有的 诊断输出都将以 JSON blobs 的形式发出。 `--json` 参数可以与 `--error-format=json` 参数一起使用以配置 JSON blobs 包含的内容以及发出的内容。

使用 `--error-format=json` 编译器 会将所有编译器错误以 JSON blob 格式发出，但  `--json` 标签也可以用以下选项来自定义输出：

- `diagnostic-short` -  用于诊断消息的json blobs应该使用“ short ”形式呈现，而不是平时的“human”默认值。这意味着 `--error-format=short` 的输出将被嵌入到JSON诊断中，而不是默认的 `--error-format=human` 。

- `diagnostic-rendered-ansi` - 默认情况下，JSON blob在其 `rendered` 字段中将包含诊断的纯文本呈现。相反，该选项指示诊断应该嵌入ANSI颜色代码用于为消息着色，就像rustc通常对终端输出所做的那样。注意通常与像 [`fwdansi`](https://crates.io/crates/fwdansi) 这类的 crate 结合以在 windows console 命令行转换 ANSI 码；或者如果你想在其后选择性地移除 ansi 颜色可以结合 [`strip-ansi-escapes`](https://crates.io/crates/strip-ansi-escapes)。

- `artifacts` - 这指示rustc为所发出的每个部件发出一个JSON blob对象。部件对应于来自 [`--emit` CLI](#option-emit) 参数的请求，一旦部件在文件系统上可用，就会发出相应通知（译者注：即JSON blob对象）。

请注意，`--json` 参数与 [`--color`](#option-color) 参数组合是无效的，需要将 `--json` 与 `--error format=json` 组合。
更多细节请参阅[json 章节][the JSON chapter]。

<a id="at-path"></a>
## `@path`: 从路径加载命令行标签

如果在命令行中指定 `@path`，则它将打开 `path` 并读取命令行选项。这些选项每行一个；空行表示一个空选项。文件可以使用Unix或Windows样式的行尾，并且必须编码为UTF-8。

[the JSON chapter]: json.md