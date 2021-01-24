# 默认等级为拒绝的 lints

默认情况下，这些 lint 都设置为 'deny' 级别。

- [`ambiguous_associated_items`](#ambiguous_associated_items)
- [`arithmetic_overflow`](#arithmetic_overflow)
- [`conflicting_repr_hints`](#conflicting_repr_hints)
- [`const_err`](#const_err)
- [`ill_formed_attribute_input`](#ill_formed_attribute_input)
- [`incomplete_include`](#incomplete_include)
- [`invalid_type_param_default`](#invalid_type_param_default)
- [`macro_expanded_macro_exports_accessed_by_absolute_paths`](#macro_expanded_macro_exports_accessed_by_absolute_paths)
- [`missing_fragment_specifier`](#missing_fragment_specifier)
- [`mutable_transmutes`](#mutable_transmutes)
- [`no_mangle_const_items`](#no_mangle_const_items)
- [`order_dependent_trait_objects`](#order_dependent_trait_objects)
- [`overflowing_literals`](#overflowing_literals)
- [`patterns_in_fns_without_body`](#patterns_in_fns_without_body)
- [`pub_use_of_private_extern_crate`](#pub_use_of_private_extern_crate)
- [`soft_unstable`](#soft_unstable)
- [`unconditional_panic`](#unconditional_panic)
- [`unknown_crate_types`](#unknown_crate_types)
- [`useless_deprecated`](#useless_deprecated)



## ambiguous_associated_items

`ambiguous_associated_items` lint 检测枚举变量和关联项之间的不确定项。  

**样例**

```rust
# #![allow(unused)]
# fn main() {
enum E {
    V
}

trait Tr {
    type V;
    fn foo() -> Self::V;
}

impl Tr for E {
    type V = u8;
    // `Self::V` is ambiguous because it may refer to the associated type or
    // the enum variant.
    fn foo() -> Self::V { 0 }
}
# }
```

显示如下：

```text
error: ambiguous associated item
  --> lint_example.rs:15:17
   |
15 |     fn foo() -> Self::V { 0 }
   |                 ^^^^^^^ help: use fully-qualified syntax: `<E as Tr>::V`
   |
   = note: `#[deny(ambiguous_associated_items)]` on by default
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #57644 <https://github.com/rust-lang/rust/issues/57644>
note: `V` could refer to the variant defined here
  --> lint_example.rs:3:5
   |
3  |     V
   |     ^
note: `V` could also refer to the associated type defined here
  --> lint_example.rs:7:5
   |
7  |     type V;
   |     ^^^^^^^
   
```

**解释**  

早期 Rust 版本不允许通过类型别名访问枚举变量，当添加此功能时（请参阅 [RFC 2338][RFC-2338]），这引入了某些情况，即类型所指的可能不明确。

要解决该歧义，应使用[路径限定][qualified path]明确声明要使用的类型。例如，以上示例中函数可以被写作`fn f() -> <Self as Tr>::V { 0 } ` 以明确引用关联的类型。

这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #57644][issue-#57644]


[RFC-2338]:https://github.com/rust-lang/rfcs/blob/master/text/2338-type-alias-enum-variants.md
[qualified path]:https://doc.rust-lang.org/reference/paths.html#qualified-paths
[issue-#57644]:https://github.com/rust-lang/rust/issues/57644

## arithmetic_overflow
`arithmetic_overflow` lint 检测会溢出的算术运算

**样例**  

```rust
# #![allow(unused)]
# fn main() {
1_i32 << 32;
# }
```
显示如下：
```text
error: this arithmetic operation will overflow
 --> lint_example.rs:2:1
  |
2 | 1_i32 << 32;
  | ^^^^^^^^^^^ attempt to shift left by `32_i32`, which would overflow
  |
  = note: `#[deny(arithmetic_overflow)]` on by default
```
**解释**  

执行值溢出运算很可能是错误，如果编译器能够在编译时检测到这些溢出，就会触发这个 lint。请考虑调整表达式避免溢出，或者使用不会溢出的数据类型。

## conflicting_repr_hints
`conflicting_repr_hints` lint 检测带有冲突提示的 [`repr`](https://doc.rust-lang.org/reference/type-layout.html#representations) 属性。

**样例**  

```rust

# #![allow(unused)]
# fn main() {
#[repr(u32, u64)]
enum Foo {
    Variant1,
}
# }
```
显示如下：
```text
error[E0566]: conflicting representation hints
 --> lint_example.rs:2:8
  |
2 | #[repr(u32, u64)]
  |        ^^^  ^^^
  |
  = note: `#[deny(conflicting_repr_hints)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #68585 <https://github.com/rust-lang/rust/issues/68585>

```
**解释**  

过去编译器错误的接受了这些冲突的表示形式。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。了解更多细节请参阅 [issue #68585](https://github.com/rust-lang/rust/issues/68585)

想要更正该问题，请移除冲突的提示之一。

## const_err
`const_err` lint 检测常量求值时的错误表达式。

**样例**  

```rust
# #![allow(unused)]
#![allow(unconditional_panic)]
# fn main() {
let x: &'static i32 = &(1 / 0);
# }
```
显示如下：
```text
error: reaching this expression at runtime will panic or abort
 --> lint_example.rs:3:24
  |
3 | let x: &'static i32 = &(1 / 0);
  |                       -^^^^^^^
  |                        |
  |                        dividing by zero
  |
  = note: `#[deny(const_err)]` on by default
```
**解释**  

该 lint 检测很可能错误的代码，如果将该 lint 等级变为允许，那么代码将不会在编译时进行计算，而是继续生成代码，在运行时计算，这可能会在运行时 panic 。
注意，该 lint 可以在 [const 上下文][const-context]内部或外部触发。在 [const 上下文][const-context]外部，编译器有时会在编译时对表达式求值以生成更高效的代码。如果编译器要在这方面做得更好，它需要决定当遇到肯定会panic 或是不正确的代码时应该怎么做。将此设置为固有错误（hard error）将阻止表现出此行为的现有代码编译，破坏向后兼容性。然而，这肯定是不正确的代码，因此这是一个默认等级为拒绝的 lint。更多细节请参阅 [RFC 1229][RFC-1229] 和 [issue #28238][issue-28238]。
注意还有几个其他更特定的编译时计算相关的 lint，例如：[`arithmetic_overflow`][arithmetic-overflow]，[`unconditional_panic`][unconditional-panic] 。

[const-context]:https://doc.rust-lang.org/reference/const_eval.html#const-context
[RFC-1229]:https://github.com/rust-lang/rfcs/blob/master/text/1229-compile-time-asserts.md
[issue-28238]:https://github.com/rust-lang/rust/issues/28238
[arithmetic-overflow]:https://doc.rust-lang.org/rustc/lints/listing/deny-by-default.html#arithmetic-overflow
[unconditional-panic]:https://doc.rust-lang.org/rustc/lints/listing/deny-by-default.html#unconditional-panic

## ill_formed_attribute_input
`ill_formed_attribute_input` lint 检测以前被接收并且用于实践中的不良格式的属性输入。  

**样例**  

```rust
# #![allow(unused)]
# fn main() {
#[inline = "this is not valid"]
fn foo() {}
# }
```
这会显示：
```text
error: attribute must be of the form `#[inline]` or `#[inline(always|never)]`
 --> lint_example.rs:2:1
  |
2 | #[inline = "this is not valid"]
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[deny(ill_formed_attribute_input)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #57571 <https://github.com/rust-lang/rust/issues/57571>

```
**解释**  

以前，许多内置属性的输入没有经过验证，无意义的属性输入被接收。在添加了验证之后，明确了一些现有的项目使用了这些无效的格式。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #57571](https://github.com/rust-lang/rust/issues/57571) 。
有关有效输入的属性，更多细节请参阅 [attribute reference](https://doc.rust-lang.org/nightly/reference/attributes.html) 。

## incomplete_include
`incomplete_include` lint 检测一个文件包含多于一个表达式的 [`include!`][include-macro] 宏。

**样例**  
```rust,igonre
fn main() {
    include!("foo.txt");
}
```
当 `foo.txt`文件包含以下内容：
```text
println!("hi!");
```
显示如下：
```text
error: include macro expected single expression in source
 --> foo.txt:1:14
  |
1 | println!("1");
  |              ^
  |
  = note: `#[deny(incomplete_include)]` on by default
```
**解释**  

[`include!`][include-macro] 宏当前仅打算用于单个[表达式][expression]或多个[项][item]。从以前看，它会忽略第一个表达式之后的任何内容，但这可能会令人困惑。在上例中，println! 表达式（ println! expression ）刚好在分号之前结束，从而使分号成为多余的信息而被忽略，更令人惊讶的是，如果包含的文件有多个打印语句，后续的语句将被忽略!
一个解决办法是将内容放在大括号中创建[块表达式][block-expression]。还可以考虑其他办法，例如函数封装表达式或者使用[过程宏][proc-macros]。
这是个 lint 而不是固有错误是因为现有项目已经发现并报过错。谨慎起见，它现在还是个 lint 。`include!` 宏的未来语义还不确定，请参阅 [issue #35560](https://github.com/rust-lang/rust/issues/35560)。

[include-macro]:https://doc.rust-lang.org/std/macro.include.html
[block-expression]:https://doc.rust-lang.org/reference/expressions/block-expr.html
[proc-macros]:https://doc.rust-lang.org/reference/procedural-macros.html

## invalid_type_param_default
`invalid_type_param_default` lint 检测在无效位置中错误地允许 (allowed) 使用类型参数默认值。
**样例**  
```rust
# #![allow(unused)]
# fn main() {
fn foo<T=i32>(t: T) {}
# }
```
显示如下：
```text
error: defaults for type parameters are only allowed in `struct`, `enum`, `type`, or `trait` definitions.
 --> lint_example.rs:2:8
  |
2 | fn foo<T=i32>(t: T) {}
  |        ^
  |
  = note: `#[deny(invalid_type_param_default)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #36887 <https://github.com/rust-lang/rust/issues/36887>

```
**解释**  

默认类型参数仅在某些情况下才允许使用，但是以前编译器在任何地方都允许使用。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #36887](https://github.com/rust-lang/rust/issues/36887) 。

## macro_expanded_macro_exports_accessed_by_absolute_paths
`macro_expanded_macro_exports_accessed_by_absolute_paths` lint 检测当前 crate 中不能被绝对路径引用的 [`macro_export`][macro-export]宏的宏展开。
**样例**  
```rust
macro_rules! define_exported {
    () => {
        #[macro_export]
        macro_rules! exported {
            () => {};
        }
    };
}

define_exported!();

fn main() {
    crate::exported!();
}
```
显示如下：
```text
error: macro-expanded `macro_export` macros from the current crate cannot be referred to by absolute paths
  --> lint_example.rs:13:5
   |
13 |     crate::exported!();
   |     ^^^^^^^^^^^^^^^
   |
   = note: `#[deny(macro_expanded_macro_exports_accessed_by_absolute_paths)]` on by default
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #52234 <https://github.com/rust-lang/rust/issues/52234>
note: the macro is defined here
  --> lint_example.rs:4:9
   |
4  | /         macro_rules! exported {
5  | |             () => {};
6  | |         }
   | |_________^
...
10 |   define_exported!();
   |   ------------------- in this macro invocation
   = note: this error originates in a macro (in Nightly builds, run with -Z macro-backtrace for more info)

```
**解释**  

我们的目的是所有使用 `#[macro_export]` 属性的宏在 crate 根是可用的。然而，当一个 `macro_rules!` 定义由另一个宏生成之时，宏展开是无法遵循该规则的。
这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #53495](https://github.com/rust-lang/rust/issues/53495) 。

[macro-export]:https://doc.rust-lang.org/reference/macros-by-example.html#path-based-scope

## missing_fragment_specifier
`missing_fragment_specifier` lint 当有一个未使用的模式在未跟有片段说明符 (fragment specifier) (例如：`:expr`)的元变量(例如：`$e`)的 `macro_rules!` 宏定义中时被触发。

始终可以通过移除 `macro_rules!` 宏定义中未使用的模式来解决此警告。

**样例**  
```rust
macro_rules! foo {
   () => {};
   ($name) => { };
}

fn main() {
   foo!();
}
```
显示如下：
```text
error: missing fragment specifier
 --> lint_example.rs:3:5
  |
3 |    ($name) => { };
  |     ^^^^^
  |
  = note: `#[deny(missing_fragment_specifier)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #40107 <https://github.com/rust-lang/rust/issues/40107>
```
**解释**  
要修复此问题，从 `macro_rules!` 定义中移除此未使用模式：
```rust
macro_rules! foo {
    () => {};
}
fn main() {
    foo!();
}
```

## mutable_transmutes
`mutable_transmutes` lint 捕捉从 `&T` 到 `&mut T` 这种[未定义行为][undefined-behavior] 的转换。
**样例**  
```rust
# #![allow(unused)]
# fn main() {
unsafe {
    let y = std::mem::transmute::<&i32, &mut i32>(&5);
}
# }
```
显示如下：
```text
error: mutating transmuted &mut T from &T may cause undefined behavior, consider instead using an UnsafeCell
 --> lint_example.rs:3:13
  |
3 |     let y = std::mem::transmute::<&i32, &mut i32>(&5);
  |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[deny(mutable_transmutes)]` on by default
```
**解释**  

我们对数据别名做出了一些假设，而这种转换是违反这些假设的。考虑使用 [`UnsafeCell`](https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html)。

[undefined-behavior]:https://doc.rust-lang.org/reference/behavior-considered-undefined.html

## no_mangle_const_items
`no_mangle_const_items` lint 检测 [`no_mangle`](https://doc.rust-lang.org/reference/abi.html#the-no_mangle-attribute)属性的所有 `const` 项。
**样例**  
```rust
# #![allow(unused)]
# fn main() {
#[no_mangle]
const FOO: i32 = 5;
# }
```
显示如下：
```text
error: const items should never be `#[no_mangle]`
 --> lint_example.rs:3:1
  |
3 | const FOO: i32 = 5;
  | -----^^^^^^^^^^^^^^
  | |
  | help: try a static value: `pub static`
  |
  = note: `#[deny(no_mangle_const_items)]` on by default
```
**解释**  

常量没有其导出符号，因此这可能意味着你得用 [`static`][static] 而不是 [`const`][const]。

[static]:https://doc.rust-lang.org/reference/items/static-items.html
[const]:https://doc.rust-lang.org/reference/items/constant-items.html

## order_dependent_trait_objects
`order_dependent_trait_objects` lint 检测一种 trait 一致性冲突，该冲突即为同一个包含标记 trait （marker traits）的 dynamic trait object 创建两个 trait 实现。

**样例**  
```rust
# #![allow(unused)]
# fn main() {
pub trait Trait {}

impl Trait for dyn Send + Sync { }
impl Trait for dyn Sync + Send { }
# }
```
显示如下：
```text
error: conflicting implementations of trait `main::Trait` for type `(dyn std::marker::Send + std::marker::Sync + 'static)`: (E0119)
 --> lint_example.rs:5:1
  |
4 | impl Trait for dyn Send + Sync { }
  | ------------------------------ first implementation here
5 | impl Trait for dyn Sync + Send { }
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ conflicting implementation for `(dyn std::marker::Send + std::marker::Sync + 'static)`
  |
  = note: `#[deny(order_dependent_trait_objects)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #56484 <https://github.com/rust-lang/rust/issues/56484>

```
**解释**  

以前的一个 bug 导致编译器将不同顺序的 trait （例如 `Send + Sync` 和 `Sync + Send`）解释为不同的类型，然而它们应该被认为是相同的。这允许代码在出现一致性错误的时候定义单独的 trait 实现。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #56484](https://github.com/rust-lang/rust/issues/56484) 。

## overflowing_literals
`overflowing_literals` lint 检测超出其所属类型范围的字面量。
**样例**  
```rust
# #![allow(unused)]
# fn main() {
let x: u8 = 1000;
# }
```
显示如下：
```text
error: literal out of range for `u8`
 --> lint_example.rs:2:13
  |
2 | let x: u8 = 1000;
  |             ^^^^
  |
  = note: `#[deny(overflowing_literals)]` on by default
  = note: the literal `1000` does not fit into the type `u8` whose range is `0..=255`
```
**解释**  

使用溢出其所用类型的字面量通常是错误。要么就使用在其类型范围内的字面量，要么就更改其类型以能容纳该字面量。

## patterns_in_fns_without_body
`patterns_in_fns_without_body` lint 检测 `mut` [标识符模式][identifier-patterns]用于没有函数体的函数的参数。
**样例**  
```rust
# #![allow(unused)]
# fn main() {
trait Trait {
    fn foo(mut arg: u8);
}
# }
```
显示如下：
```text
error: patterns aren't allowed in functions without bodies
 --> lint_example.rs:3:12
  |
3 |     fn foo(mut arg: u8);
  |            ^^^^^^^
  |
  = note: `#[deny(patterns_in_fns_without_body)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #35203 <https://github.com/rust-lang/rust/issues/35203>
```
**解释**  

要想修复此问题， trait 定义中从参数移除 `mut` ；也可以使用默认实现。也就是说，以下两种都行：
```rust
# #![allow(unused)]
# fn main() {
trait Trait {
    fn foo(arg: u8); // Removed `mut` here
}

impl Trait for i32 {
    fn foo(mut arg: u8) { // `mut` here is OK

    }
}
# }
```
trait 定义中可以定义没有函数体的函数以指定实现必须实现的函数体。无函数体的函数形参名仅允许是 `_` 或为了文档目的的（仅类型相关）的[标识符](https://doc.rust-lang.org/reference/identifiers.html)。以前的编译器版本错误地允许[标识符模式][identifier-patterns]使用 `mut` 关键字，但这是不被允许的。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #35203](https://github.com/rust-lang/rust/issues/35203) 。

[identifier-patterns]:https://doc.rust-lang.org/reference/patterns.html#identifier-patterns

## pub_use_of_private_extern_crate
`pub_use_of_private_extern_crate` lint 检测私有 `extern crate` 重导出的具体情况。

**样例**  
```rust,ignore
# #![allow(unused)]
# fn main() {
extern crate core;
pub use core as reexported_core;
# }
```
显示如下：
```text
error: extern crate `core` is private, and cannot be re-exported (error E0365), consider declaring with `pub`
 --> lint_example.rs:3:9
  |
3 | pub use core as reexported_core;
  |         ^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[deny(pub_use_of_private_extern_crate)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #34537 <https://github.com/rust-lang/rust/issues/34537>
```
**解释**  

一个公开的 `use` 声明不应该用于 公开性地重导出私有 `extern crate`。应该使用 `pub extern crate`。
过去是允许该行为的，但是根据可见性规则这是不符合预期的。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #34537](https://github.com/rust-lang/rust/issues/34537) 。

## soft_unstable
`soft_unstable` lint 检测 在 stable 上无意间允许（allowed）的 unstable feature。
**样例**  
```rust
# #![allow(unused)]
# fn main() {
#[cfg(test)]
extern crate test;

#[bench]
fn name(b: &mut test::Bencher) {
    b.iter(|| 123)
}
# }
```
显示如下：
```text
error: use of unstable library feature 'test': `bench` is a part of custom test frameworks which are unstable
 --> lint_example.rs:5:3
  |
5 | #[bench]
  |   ^^^^^
  |
  = note: `#[deny(soft_unstable)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #64266 <https://github.com/rust-lang/rust/issues/64266>
```
**解释**  

[`bench`](https://doc.rust-lang.org/nightly/unstable-book/library-features/test.html) 属性意外地在 [stable release channel](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html) 上被指定。将此转化为固有错误会破坏一些（现有）项目。当使用 `--cap-lints` 时该 lint 允许项目正确地构建，否则会发出一个错误提示。`#[bench]` 不应该被用在 stable channel。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #64266](https://github.com/rust-lang/rust/issues/64266) 。

## unconditional_panic
`unconditional_panic` lint 检测将在运行时引起 panic 的操作。

**样例**  
```rust
# #![allow(unused)]
# fn main() {
# #![allow(unused)]
let x = 1 / 0;
# }
```
显示如下：
```text
error: this operation will panic at runtime
 --> lint_example.rs:3:9
  |
3 | let x = 1 / 0;
  |         ^^^^^ attempt to divide `1_i32` by zero
  |
  = note: `#[deny(unconditional_panic)]` on by default
```
**解释**  

该 lint 检测很可能不正确的代码。如果可能，编译器将尝试检测代码能够在编译时进行计算的情况，以生成更高效的代码。在计算这类代码时，如果检测到代码会无条件地 panic ，通常表示（要执行的计算）在做一些错误的事情。如果该 lint 等级被改为允许，然后此代码将不会在编译时被计算，而是继续生成代码在运行时计算，这也可能会在运行时 panic。

## unknown_crate_types
`unknown_crate_types` lint 检测在 [`crate_type`](https://doc.rust-lang.org/reference/linkage.html)属性中找到的未知 crate 类型。

**样例**  
```rust
#![crate_type="lol"]
fn main() {}
```
显示如下:
```text
error: invalid `crate_type` value
 --> lint_example.rs:1:15
  |
1 | #![crate_type="lol"]
  |               ^^^^^
  |
  = note: `#[deny(unknown_crate_types)]` on by default
```
**解释**  

给 `crate_type` 属性赋未知值可以肯定说是一个错误。

## useless_deprecated
`useless_deprecated` lint 检测无效且弃用的属性。

**样例**  
```rust
# #![allow(unused)]
# fn main() {
struct X;

#[deprecated = "message"]
impl Default for X {
    fn default() -> Self {
        X
    }
}
# }
```
显示如下：
```text
error: this `#[deprecated]` annotation has no effect
 --> lint_example.rs:4:1
  |
4 | #[deprecated = "message"]
  | ^^^^^^^^^^^^^^^^^^^^^^^^^ help: remove the unnecessary deprecation attribute
  |
  = note: `#[deny(useless_deprecated)]` on by default
```
**解释**  
弃用属性对 trait 实现是无影响的。

[future-incompatible]:https://doc.rust-lang.org/rustc/lints/index.html#future-incompatible-lints
[expression]:https://doc.rust-lang.org/reference/expressions.html
[item]:https://doc.rust-lang.org/reference/items.html
