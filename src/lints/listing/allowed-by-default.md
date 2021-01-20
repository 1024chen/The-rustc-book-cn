# 默认等级为允许的 lints

默认情况下，这些 lint 都设置为'allow'级别。因此，除非您使用标志或属性将它们设置为更高的 lint 级别，否则它们将不会显示。

## absolute_paths_not_starting_with_crate

`absolute_paths_not_starting_with_crate` lint 检测完全合乎“路径以模块名称开头”的这个要求，即以 `crate` , `self` ，或是以一个 extern crate 名称作为开头。

**样例**

```rust
#![deny(absolute_paths_not_starting_with_crate)]

mod foo {
    pub fn bar() {}
}

fn main() {
    ::foo::bar();
}
```
显示如下结果：
```bash
error: absolute paths must start with `self`, `super`, `crate`, or an external crate name in the 2018 edition
 --> lint_example.rs:8:5
  |
8 |     ::foo::bar();
  |     ^^^^^^^^^^ help: use `crate`: `crate::foo::bar`
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(absolute_paths_not_starting_with_crate)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in the 2018 edition!
  = note: for more information, see issue #53130 <https://github.com/rust-lang/rust/issues/53130>
```

**解释**

Rust [语义版本][edition_guide]允许语言向前发展而不破坏其向后兼容性（译者注：向后兼容指的是新版本可以运行旧代码）。此 lint 捕获使用 2015语义版本路径样式的代码。在2015语义版本中，绝对路径（以 :: 开头）指的是（本 crate 的） crate 根 或者是外部 crate 。应该使用 `crate::`路径前缀指向来自 crate 根的项。

如果在不更新代码的情况下将编译器从2015版切换到2018版，那么如果使用旧样式路径，编译器将无法编译。您可以手动更改路径使用`crate::` 前缀过渡到2018版。

该 lint 将自动解决此问题。其等级默认为 “allow” 因为此代码在 2015语义版本中是完全有效的。 [`cargo fix`](https://doc.rust-lang.org/cargo/commands/cargo-fix.html)工具带有的 `--edition` 标签会将此 lint 的等级切换为 “warn” 并且自动应用编译器的修改建议。这提供了一种将旧代码完全自动升级到 2018语义版本的方法。

## anonymous-parameters

此 lint 检测匿名参数。一些触发此 lint 的示例代码：
**样例**

```rust
#![deny(anonymous_parameters)]
// edition 2015
pub trait Foo {
    fn foo(usize);
}
fn main() {}

```

显示如下结果：

```text
error: anonymous parameters are deprecated and will be removed in the next edition.
 --> lint_example.rs:4:12
  |
4 |     fn foo(usize);
  |            ^^^^^ help: try naming the parameter or explicitly ignoring it: `_: usize`
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(anonymous_parameters)]
  |         ^^^^^^^^^^^^^^^^^^^^
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in the 2018 edition!
  = note: for more information, see issue #41686 <https://github.com/rust-lang/rust/issues/41686>
```
**解释**

这种语法大多是历史意外，可以通过添加 `_` 模式（译者注：通配符）或描述性标识符很轻松地解决：

```rust
trait Foo {
    fn foo(_: usize);
}
```
这个语法现在在2018语义版本中仍是个固有错误(hard error)。在2015语义版本中，因为旧代码依然有效所以这个 lint 默认级别是 “allow” ，并且对所有旧代码应用 warning 可能会带来干扰（ noisy ）。此 lint 可以通过 [`cargo fix`](https://doc.rust-lang.org/cargo/commands/cargo-fix.html) 工具自带的 `--edition` 标签自动地将旧代码从 2015 版本转换到 2018版本。该工具将会切换此 lint 的等级为 “warn” 并且自动应用编译器的修改建议（会为每个匿名变量添加 `_` ）。这提供了一种将旧代码完全自动升级到新语义版本的方法。更多细节请参考 [issue #41686](https://github.com/rust-lang/rust/issues/41686)。

## box-pointers

`box_pointers` lint 用于 Box 类型。

**样例**

```rust
#![deny(box_pointers)]
struct Foo {
    x: Box<isize>,
}
```

显示如下结果：

```text
error: type uses owned (Box type) pointers: Box<isize>
 --> lint_example.rs:4:5
  |
4 |     x: Box<isize>,
  |     ^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(box_pointers)]
  |         ^^^^^^^^^^^^

```
**解释**

这种 lint 主要是历史性的，并不是特别有用。以前 `Box<T>`是用于构建语言，以及进行堆分配的唯一方法。今天的 Rust 可以调用其他堆分配器等。


## elided-lifetimes-in-paths

`elided_lifetimes_in_paths` lint 用于检测隐藏生命周期参数。
**样例**

```rust
#![deny(elided_lifetimes_in_paths)]
struct Foo<'a> {
    x: &'a u32
}

fn foo(x: &Foo) {
}
```
显示如下结果：
```text
error: hidden lifetime parameters in types are deprecated
 --> lint_example.rs:7:12
  |
7 | fn foo(x: &Foo) {
  |            ^^^- help: indicate the anonymous lifetime: `<'_>`
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(elided_lifetimes_in_paths)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^

```
**解释**

省略的生命周期参数可能使您一眼就看不到发生了借用。该 lint 确保生命周期参数总是被显式指出，即便是`_` [占位符的生命周期](https://doc.rust-lang.org/reference/lifetime-elision.html#lifetime-elision-in-functions)。

该 lint 默认情况下是 “allow” 级别，因为其有一些已知的问题，并且可能需要对旧代码进行重大的转换。

## explicit-outlives-requirements
`explicit_outlives_requirements` lint 检测指出不必要的生命周期约束（bounds）。



**样例**

```rust
#![deny(explicit_outlives_requirements)]

struct SharedRef<'a, T>
where
    T: 'a,
{
    data: &'a T,
}
```
显示如下结果：
```text
error: outlives requirements can be inferred
 --> lint_example.rs:5:24
  |
5 |   struct SharedRef<'a, T>
  |  ________________________^
6 | | where
7 | |     T: 'a,
  | |__________^ help: remove this bound
  |
note: the lint level is defined here
 --> lint_example.rs:2:9
  |
2 | #![deny(explicit_outlives_requirements)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```
**解释**

如果结构体包含引用，例如 `&'a T`，编译器要求 `T` 的生命周期长度比 `'a` 更久。以往要求写上明确的生命周期约束（ bound ）来此满足此条件。然而，这可能过于明显，且会导致混乱和不必要的复杂性。语言已经改进为如果没有指定则会自动推断约束。具体举例来说，如果结构体包含引用，直接或间接地包含 `T` 和生命周期 `'x` ，则它会自动推断要求（requirement） `T: 'x`。

默认情况下该 lint 级别为 “allow”，因为可能会对现有的已经具有这些要求的代码产生干扰。这是一种风格选择，因为明确的约束声明依然是有效的。它依然具有一些可能会引起混乱的误报。

## invalid-html-tags
`invalid_html_tags` lint 检测无效的 HTML 标签。这是一个仅用于 `rustdoc` 的 lint ，请参阅 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#invalid_html_tags)中的文档。

## keyword-idents
`keyword-idents` lint 检测被用作标识符的版本关键字。

**样例**

```rust
#![deny(keyword_idents)]  
// edition 2015  
fn dyn() {}
```
显示如下结果：
```text
error: `dyn` is a keyword in the 2018 edition
 --> lint_example.rs:4:4
  |
4 | fn dyn() {}
  |    ^^^ help: you can use a raw identifier to stay compatible: `r#dyn`
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(keyword_idents)]
  |         ^^^^^^^^^^^^^^
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in the 2018 edition!
  = note: for more information, see issue #49716 <https://github.com/rust-lang/rust/issues/49716>

```

**解释**

Rust [语义版本][edition_guide]允许语言向前发展而不破坏其向后兼容性。此 lint 捕获代码中的被用作标识符（例如变量名、函数名等等）的新增关键。如果你没有更新代码就切换编译器到一个新语义版本，就会在你将新关键字作为标识符的情况下编译失败。

可以手动将标识符改为非关键字，或者使用[原始标识符](https://doc.rust-lang.org/reference/identifiers.html)，例如`r#dyn`，来过渡到新版本。

该 lint 可自动解决该问题，其默认为 “allow”等级，因为该代码在旧版本中完全有效。[`cargo fix`](https://doc.rust-lang.org/cargo/commands/cargo-fix.html)工具自带的`--edition`标签会将此 lint 的等级切换为 “warn” ，并自动应用编译器建议的修复（即使用原始标识符）。这提供了一种完全自动化的方法来将旧代码更新到新版本。

## macro-use-extern-crate
`macro-use-extern-crate` lint 用于检测 [`macro_use`](https://doc.rust-lang.org/reference/macros-by-example.html#the-macro_use-attribute)属性的使用

**样例**

```rust
#![deny(macro_use_extern_crate)]

#[macro_use]
extern crate serde_json;

fn main() {
    let _ = json!{{}};
}

```
显示如下：
```text
error: deprecated `#[macro_use]` attribute used to import macros should be replaced at use sites with a `use` item to import the macro instead
 --> src/main.rs:3:1
  |
3 | #[macro_use]
  | ^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> src/main.rs:1:9
  |
1 | #![deny(macro_use_extern_crate)]
  |         ^^^^^^^^^^^^^^^^^^^^^^

```
**解释**

[`macro_use`](https://doc.rust-lang.org/reference/macros-by-example.html#the-macro_use-attribute)属性放在 `extern crate` 项上使其宏可被使用，而这个外部 crate 可能会被放进该 crate 的路径前缀，导致导入宏在作用域内无处不在。在 [2018 版本][2018 edition]中致力于简化依赖项的处理，`extern crate` 的使用已经淘汰了。要将宏从外部 crate 导入作用域，建议使用 [`use`](https://doc.rust-lang.org/reference/items/use-declarations.html) 导入。

## meta_variable_misuse
`meta_variable_misuse` lint 检测宏定义中可能存在的元变量滥用。

**样例**

```rust
#![deny(meta_variable_misuse)]

macro_rules! foo {
    () => {};
    ($( $i:ident = $($j:ident),+ );*) => { $( $( $i = $k; )+ )* };
}

fn main() {
    foo!();
}
```
显示如下：
```text
error: unknown macro variable `k`
 --> lint_example.rs:5:55
  |
5 |     ($( $i:ident = $($j:ident),+ );*) => { $( $( $i = $k; )+ )* };
  |                                                       ^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(meta_variable_misuse)]
  |         ^^^^^^^^^^^^^^^^^^^^

```

**解释**

[`macro_rules`](https://doc.rust-lang.org/reference/macros-by-example.html)宏有许多不恰当的定义方式，这些错误以前只有在宏被展开或根本不（ not at all ）展开时才能才能检测得到。该 lint 尝试在当定义了宏的时候捕获一些问题。
该 lint 默认等级是 “allow”的，因为其有误报或其他问题。更多细节请参阅 [issue #61053](https://github.com/rust-lang/rust/issues/61053)。


## missing_copy_implementations
`missing_copy_implementations` lint 检测潜在的忘记实现 [`Copy`][Copy trait] trait。

**样例**

```rust
#![deny(missing_copy_implementations)]
pub struct Foo {
    pub field: i32
}
```
显示如下：
```text
error: type could implement `Copy`; consider adding `impl Copy`
 --> lint_example.rs:2:1
  |
2 | / pub struct Foo {
3 | |     pub field: i32
4 | | }
  | |_^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_copy_implementations)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

**解释**

1.0版本以前，类型会被尽可能自动标记为 `Copy`。后面对此进行了更改，并要求实现 `Cpoy` trait 来明确选择添加此 trait。此更改的一部分内容是，如果未为一个可复制类型标记 `Copy`，一个 lint 会发出警告。

该 lint 默认等级是 “allow” 因为其代码并不算糟糕；为了使一个 `Cpoy` 类型不再 `Cpoy`(译者注：即不触发该类型实现的Copy trait那部分代码)而创建新类型（newtypes）是相当常见的，`Cpoy`类型可能会导致意外地复制大量影响性能的数据。

## missing_crate_level_docs
`missing_crate_level_docs` lint 检测 crate 根是否缺失其文档。这是一个仅用于 `rustdoc` 的 lint，请参阅 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#invalid_html_tags)中的文档。

## missing_debug_implementations
`missing_debug_implementations` lint 检测 `fmt::Debug` 的缺失。

**样例**

```rust
#![deny(missing_debug_implementations)]
pub struct Foo;
```
显示如下：
```test
error: type does not implement `Debug`; consider adding `#[derive(Debug)]` or a manual implementation
 --> lint_example.rs:2:1
  |
2 | pub struct Foo;
  | ^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_debug_implementations)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

**解释**

## missing_doc_code_examples
`missing_doc_code_examples` lint 检测文档中缺失代码样例的公开导出项。这是一个仅用于 `rustdoc` 的 lint，请参阅 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#missing_doc_code_examples) 中的文档。

## missing_docs
`missing_docs` lint 检测缺失文档的公有项目。(译者注：`missing_docs` 与  `missing_doc_code_examples` ，一个检测有没有，一个检测文档有没有代码样例)

**样例**

```rust
#![deny(missing_docs)]
pub fn foo() {}
```
显示如下：
```text
error: missing documentation for the crate
 --> lint_example.rs:1:1
  |
1 | / #![deny(missing_docs)]
2 | | fn main() {
3 | | pub fn foo() {}
4 | | }
  | |_^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_docs)]
  |         ^^^^^^^^^^^^

```
**解释**

该 lint 旨在确保一个库有良好的文档记录。没有文档的项目对于用户来说很难理解如何正确使用。
该lint 默认等级是 “allow”是因为其可能会造成干扰，并且不是所有项目都需要强制将一切用文档记录。

## non_ascii_idents
`non_ascii_idents` lint 检测非 ascii 标识符。

**样例**

```rust
#![feature(non_ascii_idents)]
#![deny(non_ascii_idents)]
fn main() {
    let föö = 1;
}
```
显示如下：
```text
error: identifier contains non-ASCII characters
 --> lint_example.rs:5:9
  |
5 |     let föö = 1;
  |         ^^^
  |
note: the lint level is defined here
 --> lint_example.rs:3:9
  |
3 | #![deny(non_ascii_idents)]
  |         ^^^^^^^^^^^^^^^^

```
**解释**

在稳定版的 Rust 上，标识符必须包含 ASCII 字符。 `non_ascii_idents` 只在 nightly feature 允许标识符包含非 ASCII 字符。该 lint 允许项目希望切换该 lint 等级为 “forbid” 以保持只使用 ASCII 字符的限制（例如，简化协作或是为了安全）。更多细节请参阅 [RFC 2457](https://github.com/rust-lang/rfcs/blob/master/text/2457-non-ascii-idents.md)。

## pointer_structural_match
`pointer_structural_match` lint 检测那些在不同编译器版本和优化级别依赖上不能用于模式中的指针。

**样例**

```rust
#![deny(pointer_structural_match)]
fn foo(a: usize, b: usize) -> usize { a + b }
const FOO: fn(usize, usize) -> usize = foo;
fn main() {
    match FOO {
        FOO => {},
        _ => {},
    }
}
```

显示如下：
```text
error: function pointers and unsized pointers in patterns behave unpredictably and should not be relied upon. See https://github.com/rust-lang/rust/issues/70861 for details.
 --> lint_example.rs:6:9
  |
6 |         FOO => {},
  |         ^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(pointer_structural_match)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #62411 <https://github.com/rust-lang/rust/issues/70861>
```

**解释**

早期 Rust 版本允许在模式中使用函数指针和泛（wide）原始指针。尽管许多情况下可以按用户期望的方式运行，但由于编译器进行优化，在运行时，指针可能已经 “不等于自身” 或者是指向不同函数的函数指针相等。这是因为如果函数体相等， LLVM 会优化掉重复函数（译者注：即保留一个），因此也会使得这些指向他们的函数指针指向同一位置。另外，如果重复的函数在不同 crate 中，且又没有通过 LTO 进行优化（删除相同代码数据），那么就会造成重复。

## private_doc_tests
`private_doc_tests` lint 检测私有项中的文档测试。这是个只用在 `rustdoc` 中的 lint，请参阅 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#private_doc_tests) 中的文档。

## single_use_lifetimes
`single_use_lifetimes` lint 检测只使用一次的生命期。

**样例**

```rust
#![deny(single_use_lifetimes)]

fn foo<'a>(x: &'a u32) {}
```
显示如下：
```text
error: lifetime parameter `'a` only used once
 --> lint_example.rs:4:8
  |
4 | fn foo<'a>(x: &'a u32) {}
  |        ^^      -- ...is used only here
  |        |
  |        this lifetime...
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(single_use_lifetimes)]
  |         ^^^^^^^^^^^^^^^^^^^^
help: elide the single-use lifetime
  |
4 | fn foo(x: &u32) {}
  |      --   --

```
**解释**

显式指定一个生命周期，例如在函数或 `impl` 中的 `'a`应该用来链接这两者。否则，应该使用 `'_` 表明生命周期并未链接到两者，或者如果有可能的话干脆直接省略生命周期。

该 lint 默认等级为 “allow” ，因为它是在 `'_` 和省略生命周期第一次被引入的时候引入的，而且这个 lint 可能会有很多干扰（ too noisy ）。此外，它还会产生一些已知的误报，了解历史内容请参阅 [RFC 2115](https://github.com/rust-lang/rfcs/blob/master/text/2115-argument-lifetimes.md)，更多细节请参阅 [issue #44752](https://github.com/rust-lang/rust/issues/44752)

## trivial_casts
`trivial_casts` lint 检测可以被强制类型转换替代的平凡类型转换，这可能需要[类型归因](https://github.com/rust-lang/rust/issues/23416)（type ascription）或临时变量。

**样例**

```rust
#![deny(trivial_casts)]
let x: &u32 = &42;
let y = x as *const u32;
```
显示如下：
```text
error: trivial cast: `&u32` as `*const u32`
 --> lint_example.rs:4:9
  |
4 | let y = x as *const u32;
  |         ^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(trivial_casts)]
  |         ^^^^^^^^^^^^^
  = help: cast can be replaced by coercion; this might require a temporary variable

```
**解释**

平凡类型转换是一种 `e` 含有 `U` 类型，而且 `U` 是 `T` 的一个子类型的 `e as T` 转换。这种类型转换通常是不必要的，其通常可以被推断出来。

该 lint 默认等级为 “allow” ，因为在某些情况下，例如 FFI 接口或者是 复杂类型别名，可能会被不正确地触发，或者是在一些难以表达清楚意图的情况下。在未来它可能会成为一个 警告（warning），可能会因为类型归因提供了一种解决当前问题的方法。历史内容请参阅 [RFC 401](https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md)

## trivial_numeric_casts
`trivial_numeric_casts` lint 检测可能已经被移除的平凡数值类型转换。

**样例**

```rust
#![deny(trivial_numeric_casts)]
let x = 42_i32 as i32;
```
显示如下：
```text
error: trivial numeric cast: `i32` as `i32`
 --> lint_example.rs:3:9
  |
3 | let x = 42_i32 as i32;
  |         ^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(trivial_numeric_casts)]
  |         ^^^^^^^^^^^^^^^^^^^^^
  = help: cast can be replaced by coercion; this might require a temporary variable
```
**解释**

平凡数值类型转换指的是将数值类型转换为相同数值类型的转换。这种转换通常是不必要的。

该 lint 默认等级为 “allow” ，因为在某些情况下，例如 FFI 接口或者是 复杂类型别名，可能会被不正确地触发，或者是在一些难以表达清楚意图的情况下。在未来它可能会成为一个 警告（warning），可能会因为类型归因提供了一种解决当前问题的方法。历史内容请参阅 [RFC 401](https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md)

## unaligned_references
`unaligned_references` lint 检测对包装结构体字段的未对其引用。

**样例**

```rust
#![deny(unaligned_references)]

#[repr(packed)]
pub struct Foo {
    field1: u64,
    field2: u8,
}

fn main() {
    unsafe {
        let foo = Foo { field1: 0, field2: 0 };
        let _ = &foo.field1;
    }
}
```
显示如下：
```text
error: reference to packed field is unaligned
  --> lint_example.rs:12:17
   |
12 |         let _ = &foo.field1;
   |                 ^^^^^^^^^^^
   |
note: the lint level is defined here
  --> lint_example.rs:1:9
   |
1  | #![deny(unaligned_references)]
   |         ^^^^^^^^^^^^^^^^^^^^
   = note: fields of packed structs are not properly aligned, and creating a misaligned reference is undefined behavior (even if that reference is never dereferenced)

```
**解释**

创建对未充分对其的包装字段的引用是一种[未定义的行为](https://doc.rust-lang.org/reference/behavior-considered-undefined.html)并且应该被禁止。
默认情况下，此 lint 等级为 “allow” ，因为没有稳定的替代方法，并且尚不确定现有代码将如何触发此 lint 。有关更多讨论请参阅 [issue #27060](https://github.com/rust-lang/rust/issues/27060) 。

## unreachable_pub
`unreachable_pub` lint 触发无法从 crate 根到达的 pub 项。
**样例**

```rust
#![deny(unreachable_pub)]
mod foo {
    pub mod bar {

    }
}
```
显示如下：
```text
error: unreachable `pub` item
 --> lint_example.rs:4:5
  |
4 |     pub mod bar {
  |     ---^^^^^^^^
  |     |
  |     help: consider restricting its visibility: `pub(crate)`
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unreachable_pub)]
  |         ^^^^^^^^^^^^^^^
  = help: or consider exporting it for use by other crates

```
**解释**

一个裸（ bare ） pub 项的可见性可能会因为该项无法从 crate 导出而被误导。该 `pub(crate)` 可见性建议用仅可见性仅在其自身 crate 这种清晰的表达来替代。

默认情况下，此 lint 等级为 “allow” ，因为它会触发大量现有的 Rust 代码，并且会有一些误报。最终我们希望它成为一个默认警告。

## unsafe_code
`unsafe_code` lint 捕捉 `unsafe` 代码的使用。

**样例**

```rust
#![deny(unsafe_code)]
fn main() {
    unsafe {

    }
}
```
显示如下：
```text
error: usage of an `unsafe` block
 --> lint_example.rs:3:5
  |
3 | /     unsafe {
4 | |
5 | |     }
  | |_____^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unsafe_code)]
  |         ^^^^^^^^^^^

```
**解释**

该 lint 意在限制 `unsafe` 的使用，这很难被正确使用。（译者注：此处的“很难被正确使用”，一者指的是 unsafe 代码的不安全操作，一者指的是对 unsafe 代码的严格限制 lint 很难说是正确的）。

## unsafe_op_in_unsafe_fn
`unsafe_op_in_unsafe_fn` lint 检测非 unsafe 块中
 unsafe 函数中的 unsafe 操作。该 lint 仅在  [nightly 通道](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html)( nightly channel )中使用`#![feature(unsafe_block_in_unsafe_fn)]`时有效。

**样例**

```rust
#![feature(unsafe_block_in_unsafe_fn)]
#![deny(unsafe_op_in_unsafe_fn)]

unsafe fn foo() {}

unsafe fn bar() {
    foo();
}

fn main() {}
```

显示如下：
```text
error: call to unsafe function is unsafe and requires unsafe block (error E0133)
 --> lint_example.rs:7:5
  |
7 |     foo();
  |     ^^^^^ call to unsafe function
  |
note: the lint level is defined here
 --> lint_example.rs:2:9
  |
2 | #![deny(unsafe_op_in_unsafe_fn)]
  |         ^^^^^^^^^^^^^^^^^^^^^^
  = note: consult the function's documentation for information on how to avoid undefined behavior

```
**解释**

当前，unsafe 函数允许在其中进行任何的 unsafe 操作。然而，这可能会因为需要对代码行为进行适当仔细的检查而增加代码体积。 `unsafe` 块提供了一种简便的，可以清楚说明代码的哪部分正在进行 unsafe 操作。在未来，我们希望修改它以便不能在一个非 unsafe 块的 `unsafe 函数`中执行 unsafe 操作。

解决此问题的方法是将将此 unsafe 代码包装进 unsafe 块中。

该 lint 默认等级为 “allow” ，因为其尚未稳定，也尚未完成。更多细节请参阅 [RFC #2585](https://github.com/rust-lang/rfcs/blob/master/text/2585-unsafe-block-in-unsafe-fn.md) 和 [issue #71668](https://github.com/rust-lang/rust/issues/71668)。

## unstable_features
`unstable_features` lint 已被废弃，不应再使用。

## unused_crate_dependencies
`unused_crate_dependencies` lint 检测未被使用的 crate 依赖。

**样例**

```rust
#![deny(unused_crate_dependencies)]
```
显示如下：
```text
error: external crate `regex` unused in `lint_example`: remove the dependency or add `use regex as _;`
  |
note: the lint level is defined here
 --> src/lib.rs:1:9
  |
1 | #![deny(unused_crate_dependencies)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^

```
**解释**

将使用了依赖项的代码移除之后，通常需要从构建配置中删除依赖。然而，有时可能会忘记这一步，导致浪费时间来构建不再使用的依赖项。该 lint 可以被用来检测从未使用的依赖项（更具体地说，那些从未被 [`use`](https://doc.rust-lang.org/reference/items/use-declarations.html) , [`extern crate`](https://doc.rust-lang.org/reference/items/extern-crates.html) ,或者任何[路径](https://doc.rust-lang.org/reference/paths.html)指向的，通过 `--extern`命令行标签指定的依赖）

该 lint 默认等级为 “allow” ，因为根据构建系统的配置不同可能会产生误报。例如，当使用 Cargo 时，一个 “包”（“package”）包含了多个 crate （例如一个库 crate 和一个二进制 crate ），但是这个包的依赖是为整体而定义的，如果有一个依赖仅在二进制 crate 中使用，在库 crate 中未使用，那么该 lint 将会在库（译者注：在库 crate 运行的时候）错误地被发出。

## unused_extern_crates
`unused_extern_crates` lint 谨防从未被使用的 `extern crate` 项。

**样例**

```rust
#![deny(unused_extern_crates)]
extern crate proc_macro;
```
显示如下：
```text
error: unused extern crate
 --> lint_example.rs:3:1
  |
3 | extern crate proc_macro;
  | ^^^^^^^^^^^^^^^^^^^^^^^^ help: remove it
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_extern_crates)]
  |         ^^^^^^^^^^^^^^^^^^^^

```
**解释**

未使用的 `extern crate` 项是无效的应该被删除。请注意，在某些情况下，需要指定 `extern crate` 以确保他们被 crate 所链接，即使没有直接引用它。可以通过为 crate 取一个下划线别名来消除检测，例如 `extern crate foo as _`。还要注意的是 `extern crate` 在 [2018 语义版本](https://doc.rust-lang.org/edition-guide/rust-2018/module-system/path-clarity.html#no-more-extern-crate)中已经不常用，因为现在外部 crate（译者注：被指定的）会被自动添加到域中。

该 lint 默认等级为 “allow” ，因为其可能会造成干扰和误报。如果要从项目中移除依赖，推荐在构建配置中将其删除（例如 Cargo.toml）确保编译时不会留下陈旧的构建条目。

## unused_import_braces
`unused_import_braces` lint 捕捉导入项中不必要的大括号。

**样例**

```rust
#![deny(unused_import_braces)]
use test::{A};

pub mod test {
    pub struct A;
}
```
显示如下：
```text
error: braces around A is unnecessary
 --> lint_example.rs:2:1
  |
2 | use test::{A};
  | ^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_import_braces)]
  |         ^^^^^^^^^^^^^^^^^^^^

```
**解释**

如果仅有单个项，应该移除大括号（例如 `use test::A;`）。

该 lint 默认等级为 “allow” ，因为其只是强制执行样式选择。

## unused_lifetimes
`unused_lifetimes` lint 检测未使用的生命周期参数。

**样例**

```rust
#[deny(unused_lifetimes)]

pub fn foo<'a>() {}
```
显示如下：
```text
error: lifetime parameter `'a` never used
 --> lint_example.rs:4:12
  |
4 | pub fn foo<'a>() {}
  |           -^^- help: elide the unused lifetime
  |
note: the lint level is defined here
 --> lint_example.rs:2:8
  |
2 | #[deny(unused_lifetimes)]
  |        ^^^^^^^^^^^^^^^^
```
**解释**

未使用的生命周期参数可能是个错误或是代码未完成。（译者注：如果是错误）应该考虑删除该参数

## unused_qualifications
`unused_qualifications` lint 检测不必要的限定名。

**样例**

```rust
#![deny(unused_qualifications)]
mod foo {
    pub fn bar() {}
}

fn main() {
    use foo::bar;
    foo::bar();
}
```
显示如下：
```text
error: unnecessary qualification
 --> lint_example.rs:8:5
  |
8 |     foo::bar();
  |     ^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_qualifications)]
  |         ^^^^^^^^^^^^^^^^^^^^^

```
**解释**

如果来自另一个模块的项已经被导入了这个域，在这种情况下无需为该项加限定名，你可以不加 `foo::` 直接调用 `bar()`。

该 lint 默认等级为 “allow” ，因为这有些花哨（ pedantic ），并不表示实际问题，而且是一种风格选择，并且当重构或移动代码的时候可能会带来干扰。

## unused_results
`unused_results` lint 检查语句中表达式未使用的 result 。

**样例**

```rust
#![deny(unused_results)]
fn foo<T>() -> T { panic!() }

fn main() {
    foo::<usize>();
}
```
显示如下：
```text
error: unused result
 --> lint_example.rs:5:5
  |
5 |     foo::<usize>();
  |     ^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_results)]
  |         ^^^^^^^^^^^^^^
```
**解释**

忽略的函数返回值可能会表明一个错误。在可以确定使用 result 的情况下推荐使用 [`must_use`](https://doc.rust-lang.org/reference/attributes/diagnostics.html#the-must_use-attribute)属性来注解函数。如果不使用此类返回值将会触发默认为警告级别的 `unused_must_use` lint。`unused_results` lint 本质上是一样的，但是其被所有的返回值触发。

该 lint 默认等级为 “allow” ，因为其可能会带来干扰，且可能并不是一个真正的问题。例如，调用 `Vec` 或 `HashMap` 的  `remove` 方法会返回先前的值（译者注：也就是已经被 remove 的那个值），你可能并不关心这个值，使用这个 lint 将会要求显式地忽略或丢弃这些值。

## variant_size_differences
`variant_size_differences` lint 检测具有不同变量大小的枚举。
**样例**

```rust
#![deny(variant_size_differences)]
enum En {
    V0(u8),
    VBig([u8; 1024]),
}
```
显示如下：
```text
error: enum variant is more than three times larger (1024 bytes) than the next largest
 --> lint_example.rs:5:5
  |
5 |     VBig([u8; 1024]),
  |     ^^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(variant_size_differences)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^

```
**解释**

向枚举中添加一个比其他变量大得多的变量可能是个错误，这会增加所有变量所需空间的总大小。这可能会影响性能和内存使用。如果第一大的变量比第二大的变量所需空间大三倍以上，就会触发这个 lint。

可以考虑将较大变量的内容放在堆上（例如通过 Box ），以保持枚举体整体大小处在较小量上。

该 lint 默认等级为 “allow” ，因为其可能会造成干扰，且可能并不是一个真正的问题。应通过基准测试和分析指导来考虑这个问题。

[edition_guide]: https://doc.rust-lang.org/edition-guide/
[2018 edition]: https://doc.rust-lang.org/edition-guide/rust-2018/module-system/path-clarity.html#no-more-extern-crate
[Copy trait]:https://doc.rust-lang.org/std/marker/trait.Copy.html