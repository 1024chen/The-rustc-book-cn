# Warn-by-default lints

这些 lint 的默认等级被设置为 “警告”。

+ [`array_into_iter`](#array_into_iter)
+ [`asm_sub_register`](#asm_sub_register)
+ [`bare_trait_objects`](#bare_trait_objects)
+ [`bindings_with_variant_name`](#bindings_with_variant_name)
+ [`broken_intra_doc_links`](#broken_intra_doc_links)
+ [`cenum_impl_drop_cast`](#cenum_impl_drop_cast)
+ [`clashing_extern_declarations`](#clashing_extern_declarations)
+ [`coherence_leak_check`](#coherence_leak_check)
+ [`confusable_idents`](#confusable_idents)
+ [`const_evaluatable_unchecked`](#const_evaluatable_unchecked)
+ [`const_item_mutation`](#const_item_mutation)
+ [`dead_code`](#dead_code)
+ [`deprecated`](#deprecated)
+ [`drop_bounds`](#drop_bounds)
+ [`ellipsis_inclusive_range_patterns`](#ellipsis_inclusive_range_patterns)
+ [`exported_private_dependencies`](#exported_private_dependencies)
+ [`function_item_references`](#function_item_references)
+ [`illegal_floating_point_literal_pattern`](#illegal_floating_point_literal_pattern)
+ [`improper_ctypes`](#improper_ctypes)
+ [`improper_ctypes_definitions`](#improper_ctypes_definitions)
+ [`incomplete_features`](#incomplete_features)
+ [`indirect_structural_match`](#indirect_structural_match)
+ [`inline_no_sanitize`](#inline_no_sanitize)
+ [`invalid_codeblock_attributes`](#invalid_codeblock_attributes)
+ [`invalid_value`](#invalid_value)
+ [`irrefutable_let_patterns`](#irrefutable_let_patterns)
+ [`late_bound_lifetime_arguments`](#late_bound_lifetime_arguments)
+ [`mixed_script_confusables`](#mixed_script_confusables)
+ [`mutable_borrow_reservation_conflict`](#mutable_borrow_reservation_conflict)
+ [`no_mangle_generic_items`](#no_mangle_generic_items)
+ [`non_autolinks`](#non_autolinks)
+ [`non_camel_case_types`](#non_camel_case_types)
+ [`non_shorthand_field_patterns`](#non_shorthand_field_patterns)
+ [`non_snake_case`](#non_snake_case)
+ [`non_upper_case_globals`](#non_upper_case_globals)
+ [`nontrivial_structural_match`](#nontrivial_structural_match)
+ [`overlapping_patterns`](#overlapping_patterns)
+ [`path_statements`](#path_statements)
+ [`private_in_public`](#private_in_public)
+ [`private_intra_doc_links`](#private_intra_doc_links)
+ [`proc_macro_derive_resolution_fallback`](#proc_macro_derive_resolution_fallback)
+ [`redundant_semicolons`](#redundant_semicolons)
+ [`renamed_and_removed_lints`](#renamed_and_removed_lints)
+ [`safe_packed_borrows`](#safe_packed_borrows)
+ [`stable_features`](#stable_features)
+ [`temporary_cstring_as_ptr`](#temporary_cstring_as_ptr)
+ [`trivial_bounds`](#trivial_bounds)
+ [`type_alias_bounds`](#type_alias_bounds)
+ [`tyvar_behind_raw_pointer`](#tyvar_behind_raw_pointer)
+ [`uncommon_codepoints`](#uncommon_codepoints)
+ [`unconditional_recursion`](#unconditional_recursion)
+ [`uninhabited_static`](#uninhabited_static)
+ [`unknown_lints`](#unknown_lints)
+ [`unnameable_test_items`](#unnameable_test_items)
+ [`unreachable_code`](#unreachable_code)
+ [`unreachable_patterns`](#unreachable_patterns)
+ [`unstable_name_collisions`](#unstable_name_collisions)
+ [`unused_allocation`](#unused_allocation)
+ [`unused_assignments`](#unused_assignments)
+ [`unused_attributes`](#unused_attributes)
+ [`unused_braces`](#unused_braces)
+ [`unused_comparisons`](#unused_comparisons)
+ [`unused_doc_comments`](#unused_doc_comments)
+ [`unused_features`](#unused_features)
+ [`unused_imports`](#unused_imports)
+ [`unused_labels`](#unused_labels)
+ [`unused_macros`](#unused_macros)
+ [`unused_must_use`](#unused_must_use)
+ [`unused_mut`](#unused_mut)
+ [`unused_parens`](#unused_parens)
+ [`unused_unsafe`](#unused_unsafe)
+ [`unused_variables`](#unused_variables)
+ [`warnings`](#warnings)
+ [`where_clauses_object_safety`](#where_clauses_object_safety)
+ [`while_true`](#while_true)



## array_into_iter

`` lint 检测数组中调用 `into_iter`。

**样例**

```rust
# #![allow(unused)]
# fn main() {
# #![allow(unused)]
[1, 2, 3].into_iter().for_each(|n| { *n; });
# }
```
显示如下：
```text
warning: this method call currently resolves to `<&[T; N] as IntoIterator>::into_iter` (due to autoref coercions), but that might change in the future when `IntoIterator` impls for arrays are added.
 --> lint_example.rs:3:11
  |
3 | [1, 2, 3].into_iter().for_each(|n| { *n; });
  |           ^^^^^^^^^ help: use `.iter()` instead of `.into_iter()` to avoid ambiguity: `iter`
  |
  = note: `#[warn(array_into_iter)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #66145 <https://github.com/rust-lang/rust/issues/66145>
```
**解释**  
在将来计划为数组添加一个 `IntoIter` 实现，这样的话它将遍历的是数组的值而不是引用。由于方法解析的工作方式，这将改变在数组上使用 `into_iter` 的那部分现有代码。避免此警告的办法是使用 `iter()` 而不是 `into_iter()`。

这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误（hard error）。更多细节和更全面的 关于此 int 的描述请参阅 [issue #66145](https://github.com/rust-lang/rust/issues/66145)。


## asm_sub_register
`asm_sub_register` lint 检测仅使用寄存器子集进行内联汇编输入。

**样例**  
```rust,ignore
#![feature(asm)]

fn main() {
    #[cfg(target_arch="x86_64")]
    unsafe {
        asm!("mov {0}, {0}", in(reg) 0i16);
    }
}
```
显示如下：
```text
warning: formatting may not be suitable for sub-register argument
 --> src/main.rs:6:19
  |
6 |         asm!("mov {0}, {0}", in(reg) 0i16);
  |                   ^^^  ^^^           ---- for this argument
  |
  = note: `#[warn(asm_sub_register)]` on by default
  = help: use the `x` modifier to have the register formatted as `ax`
  = help: or use the `r` modifier to keep the default formatting of `rax`
```
**解释**  

一些体系结构上的寄存器可以使用不同的名称来引用寄存器的一个子集。默认情况下，编译器会使用该名称来表示整个寄存器大小。想要显式使用寄存器子集，可以通过模板字符串操作数上使用修饰符来指定何时使用子寄存器来覆盖默认值。如果传入的数据类型小于默认寄存器大小的值，就出触发此 lint，以警告你可能使用了错误的（位）宽度。要想修复此问题，向模板添加建议的修饰符，或者或者将值转为正确的大小。

更多细节请参阅[寄存器模板修饰符](https://doc.rust-lang.org/nightly/unstable-book/library-features/asm.html#register-template-modifiers)。

## bare_trait_objects
`bare_trait_objects` lint 暗示为 trait 对象使用 `dyn Trait` 。 

**样例**
```rust
# #![allow(unused)]
# fn main() {
trait Trait { }

fn takes_trait_object(_: Box<Trait>) {
}
# }
```
显示如下：
```text
warning: trait objects without an explicit `dyn` are deprecated
 --> lint_example.rs:4:30
  |
4 | fn takes_trait_object(_: Box<Trait>) {
  |                              ^^^^^ help: use `dyn`: `dyn Trait`
  |
  = note: `#[warn(bare_trait_objects)]` on by default
```
**解释**  

没有 `dyn` 关键字，当阅读代码时你是否是在查看 trait 对象这可能会造成模棱两可或困惑。`dyn` 关键字将其明确，并且添加了对称性来与 [`impl Trait`](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters)对比。

## bindings_with_variant_name
`bindings_with_variant_name` lint 检测与匹配变量之一同名的模式绑定。

**样例**

```rust
# #![allow(unused)]
# fn main() {
pub enum Enum {
    Foo,
    Bar,
}

pub fn foo(x: Enum) {
    match x {
        Foo => {}
        Bar => {}
    }
}
# }
```
显示如下：
```text
warning[E0170]: pattern binding `Foo` is named the same as one of the variants of the type `Enum`
 --> lint_example.rs:9:9
  |
9 |         Foo => {}
  |         ^^^ help: to match on the variant, qualify the path: `Enum::Foo`
  |
  = note: `#[warn(bindings_with_variant_name)]` on by default

```
**解释**  

将枚举变量名称指定为[标识符模式][identifier-pattern]通常是个错误。在上例中，`match` 分支指定了变量名来绑定 `x` 。 第二个分支被忽略，因为第一个分支匹配到了所有的值。可能的意图是第一个分支意在匹配枚举变量。

两个可能的解决办法是：
+ 使用[路径模式][path-pattern]指定枚举变量，例如 `Enum::Foo`。
+ 将枚举变量引入本地作用域，例如上例在 `foo` 函数的开头添加 `use Enum::*;` 。
[path-pattern]:https://doc.rust-lang.org/reference/patterns.html#path-patterns

## broken_intra_doc_links
`broken_intra_doc_links` lint 检测解析内部文档链接目标失败。这是一个仅用于 `rustdoc` 的lint，请查看 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#broken_intra_doc_links)中的文档。

## cenum_impl_drop_cast
`cenum_impl_drop_cast` lint 检测实现了 [`Drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html) trait 的无字段（field-less）枚举体 的 `as` 强制转换。

**样例**

```rust
# #![allow(unused)]
enum E {
    A,
}

impl Drop for E {
    fn drop(&mut self) {
        println!("Drop");
    }
}

fn main() {
    let e = E::A;
    let i = e as u32;
}
```
显示如下：
```text
warning: cannot cast enum `E` into integer `u32` because it implements `Drop`
  --> lint_example.rs:14:13
   |
14 |     let i = e as u32;
   |             ^^^^^^^^
   |
   = note: `#[warn(cenum_impl_drop_cast)]` on by default
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #73333 <https://github.com/rust-lang/rust/issues/73333>
```
**解释**  

将未实现整数的 [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html) 的无字段枚举体将移动值而不调用 `drop` 。如果期望是调用 `drop`，这可能会导致令人惊讶的行为。自动调用 `drop` 将与其他操作不同。由于两种行为都不是清晰或一致，因此决定不允许这种性质的转换。

这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误（hard error）。更多细节请参阅 [issue #73333](https://github.com/rust-lang/rust/issues/73333)。

## clashing_extern_declarations
`clashing_extern_declarations` lint 检测当同名 `extern fn` 被声明但是不同类型的情况。

**样例**

```rust,ignore
# #![allow(unused)]
# fn main() {
mod m {
    extern "C" {
        fn foo();
    }
}

extern "C" {
    fn foo(_: u32);
}
# }
```
显示如下：
```text
warning: `foo` redeclared with a different signature
 --> lint_example.rs:9:5
  |
4 |         fn foo();
  |         --------- `foo` previously declared here
...
9 |     fn foo(_: u32);
  |     ^^^^^^^^^^^^^^^ this signature doesn't match the previous declaration
  |
  = note: `#[warn(clashing_extern_declarations)]` on by default
  = note: expected `unsafe extern "C" fn()`
             found `unsafe extern "C" fn(u32)`
```
**解释**  

因为在链接中不能将同名符号（symbols）解析为两个不同的函数，且一个函数不能有两个类型，一个相冲突的外部声明可以肯定是个错误。检查以确保 `extern` 定义正确且有效，且考虑将它们统一在一个位置。

该 lint 不能跨 crate 运行因为一个项目可能有依赖于相同外部函数的依赖项，但是外部函数以不同（但有效）的方式声明。例如，它们可能都为一个或多个参数声明一个不透明类型（opaque type） （最终会得到不同的类型），或者使用 `extern fn` 定义的语言中有效的转换类型，在这些情况下，编译器不能说冲突声明（clashing declaration）不正确。

## coherence_leak_check
`coherence_leak_check` lint 检测仅能由旧版泄露检查代码（the old leak-check code）区分的 trait 的冲突实现。

**样例**
```rust
# #![allow(unused)]
# fn main() {
trait SomeTrait { }
impl SomeTrait for for<'a> fn(&'a u8) { }
impl<'a> SomeTrait for fn(&'a u8) { }
# }
```
显示如下：
```text
warning: conflicting implementations of trait `main::SomeTrait` for type `for<'a> fn(&'a u8)`:
 --> lint_example.rs:4:1
  |
3 | impl SomeTrait for for<'a> fn(&'a u8) { }
  | ------------------------------------- first implementation here
4 | impl<'a> SomeTrait for fn(&'a u8) { }
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ conflicting implementation for `for<'a> fn(&'a u8)`
  |
  = note: `#[warn(coherence_leak_check)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #56105 <https://github.com/rust-lang/rust/issues/56105>
  = note: this behavior recently changed as a result of a bug fix; see rust-lang/rust#56105 for details
```
**解释**  

过去编译器接受相同功能但唯一不同是生命周期绑定器（lifetime binder）出现的位置的 trait 实现。由于借用检查器的更改实现了多个 bug 的修复，因此不再允许该行为。然而，因为这会影响现有代码，所以这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误（hard error）。
依赖此模式的代码应引入 "[newtypes](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#using-the-newtype-pattern-for-type-safety-and-abstraction)"，例如 `struct Foo(for<'a> fn(&'a u8))` 。

更多细节请参阅 [issue #56105](https://github.com/rust-lang/rust/issues/56105)。

## confusable_idents
`confusable_idents` lint 检测容易混淆的标识符对。

**样例**
```rust
# #![allow(unused)]
#![feature(non_ascii_idents)]

# fn main() {
// Latin Capital Letter E With Caron
pub const Ě: i32 = 1;
// Latin Capital Letter E With Breve
pub const Ĕ: i32 = 2;
# }
```
显示如下：
```text
warning: identifier pair considered confusable between `Ě` and `Ĕ`
 --> lint_example.rs:7:11
  |
5 | pub const Ě: i32 = 1;
  |           - this is where the previous identifier occurred
6 | // Latin Capital Letter E With Breve
7 | pub const Ĕ: i32 = 2;
  |           ^
  |
  = note: `#[warn(confusable_idents)]` on by default
  
```
**解释**  

上面的 [`non_ascii_idents`][non-ascii-idents] 是只能用于 nightly的，其允许使用非 ASCII 字符作为标识符。该 lint 当不同标识符看起来很像的时候发出警告，因为（长得像）这可能会令人困惑。

混淆检测的算法是基于  [Unicode® Technical Standard #39 Unicode Security Mechanisms Section 4 Confusable Detection](https://www.unicode.org/reports/tr39/#Confusable_Detection) 。对每个不同的标识符 X 执行 `skelenton(X)` 函数。如果在一个 crate 中存在两个不同的标识符 X 和 Y 但是却得到 `skeleton(X) = skeleton(Y)` （就会触发警告）。编译器用与此相同的机制来检查标识符与关键字相似。

请注意，易混淆字符集可能会随时间而变化。注意，如果你将该 lint 等级调整为 “禁止”，则现有代码可能在将来会（编译）失败。

## const_evaluatable_unchecked
`const_evaluatable_unchecked` lint 检测类型中使用的泛型常量。

**样例**
```rust
# #![allow(unused)]
# fn main() {
const fn foo<T>() -> usize {
    if std::mem::size_of::<*mut T>() < 8 { // size of *mut T does not depend on T
        4
    } else {
        8
    }
}

fn test<T>() {
    let _ = [0; foo::<T>()];
}
# }
```
显示如下：
```text
warning: cannot use constants which depend on generic parameters in types
  --> lint_example.rs:11:17
   |
11 |     let _ = [0; foo::<T>()];
   |                 ^^^^^^^^^^
   |
   = note: `#[warn(const_evaluatable_unchecked)]` on by default
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #76200 <https://github.com/rust-lang/rust/issues/76200>
```
**解释**  

在 1.43 发行版本，会意外地允许在数组重复表达式中使用泛型参数。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误（hard error）。更多细节描述和可能的修复请参阅 [issue #76200](https://github.com/rust-lang/rust/issues/76200) 。

## const_item_mutation
`const_item_mutation` lint 检测试图更改 `const` 项。

**样例**
```rust
const FOO: [i32; 1] = [0];

fn main() {
    FOO[0] = 1;
    // This will print "[0]".
    println!("{:?}", FOO);
}
```
显示如下：
```text
warning: attempting to modify a `const` item
 --> lint_example.rs:4:5
  |
4 |     FOO[0] = 1;
  |     ^^^^^^^^^^
  |
  = note: `#[warn(const_item_mutation)]` on by default
  = note: each usage of a `const` item creates a new temporary; the original `const` item will not be modified
note: `const` item defined here
 --> lint_example.rs:1:1
  |
1 | const FOO: [i32; 1] = [0];
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^
```
**解释**  

试图直接更改 `const` 项几乎总是个错误。上例中发生的是临时的 `const` 副本被改变，但是原本的 `const` 并没有改变。每次通过名称来引用 `const`（例如上例中的 `FOO`）时，该值的一个单独副本都会内联到该位置。
该 lint 检查直接写入字段（`FOO.field = some_value`） 或数组输入（`FOO[0] = val`），或对 `const` 进行可变引用（`&mut FOO`），包括通过自动解引用（`FOO.some_mut_self_method()`）。

以下有多种选择取决于你想要完成的方式：
+ 首先，总是重新考虑是否使用可变全局变量，因为它们可能很难正确使用，而且会使得代码更难以使用或理解。
+ 如果你试图对一个全局变量进行一次性初始化：
  - 如果想要该值可以在编译器计算，请考虑使用常量兼容（const-compatible）的值（请参阅[常量值计算](https://doc.rust-lang.org/reference/const_eval.html) ）。
  - 对于更复杂的单次初始化（single-intialization），可以考虑使用第三方 crate ，例如 [`lazy_static`](https://crates.io/crates/lazy_static)或 [`once_cell`](https://crates.io/crates/once_cell) 。
  - 如果你在使用 [nightly channel](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html)，考虑使用标准库中的新 [`lazy`](https://doc.rust-lang.org/nightly/std/lazy/index.html) 模块。
+ 如果确实需要可变的全局变量，请考虑使用具有多种选择的 [`static`](https://doc.rust-lang.org/reference/items/static-items.html) 。
  - 可以直接定义简单数据类型并可用 [原子类型](https://doc.rust-lang.org/std/sync/atomic/index.html)进行转变。
  - 更复杂的类型可以放在如 [`mutex`](https://doc.rust-lang.org/std/sync/struct.Mutex.html) 之类的同步原语中，可以使用上面列出的选项之一对其进行初始化。
  - [可变静态变量](https://doc.rust-lang.org/reference/items/static-items.html#mutable-statics) 是低级原语，要求使用 unsafe。通常情况下应该避免该情况，使用上述之一。

## dead_code
`dead_code` lint 检测未使用，未导出的代码。

**样例**
```rust
# #![allow(unused)]
# fn main() {
fn foo() {}
# }
```
显示如下：
```text
warning: function is never used: `foo`
 --> lint_example.rs:2:4
  |
2 | fn foo() {}
  |    ^^^
  |
  = note: `#[warn(dead_code)]` on by default
```
**解释**  

Dead code 表示错误或未完成的代码。要使单个项的警告静默，在名称前加上下划线例如 `_foo`。如果打算将项导出 crate 之外，考虑添加可见修饰符如 `pub` 。否则请考虑移除未使用的代码。

## deprecated
`deprecated` lint 检测不推荐使用的项。

**样例**
```rust
# #![allow(unused)]
# fn main() {
#[deprecated]
fn foo() {}

fn bar() {
    foo();
}
# }
```
显示如下：
```text
warning: use of deprecated function `main::foo`
 --> lint_example.rs:6:5
  |
6 |     foo();
  |     ^^^
  |
  = note: `#[warn(deprecated)]` on by default
```
**解释**  

项可以被标记为 “被遗弃的”，通过 [`deprecated`](https://doc.rust-lang.org/reference/attributes/diagnostics.html#the-deprecated-attribute) 属性指明不该再使用。通常该属性应包含如何使用替换的提示或者检查文档。

## drop_bounds
`drop_bounds` lint 检查使用 `std::ops::Drop` 作为约束的泛型。

**样例**
```rust
# #![allow(unused)]
# fn main() {
fn foo<T: Drop>() {}
# }
```
显示如下：
```text
warning: bounds on `T: Drop` are useless, consider instead using `std::mem::needs_drop` to detect if a type has a destructor
 --> lint_example.rs:2:11
  |
2 | fn foo<T: Drop>() {}
  |           ^^^^
  |
  = note: `#[warn(drop_bounds)]` on by default
```
**解释**  

`Drop` 约束并没有真正完成（accomplish）任何事。一个类型可能由编译器生成 drop 而没有实现 `Drop` trait 本身。`Drop` trait 也只有一个方法 `Drop::drop` ，而且该函数在用户代码中不可调用。所以实际上没有实际使用 `Drop` trait 的用例。

drop trait 最有可能的用例是区分有析构函数和没有析构函数的类型。结合泛型特化，初级程序员会编写一个实现并认为类型会被简单 drop，然后为 `T:Drop` 写了泛型特化 实际上调用了析构函数。实际上这是不正确的，例如，String 实际上并没有实现 Drop，但因为 String 包含 Vec，假设其可以被简单丢弃将会造成内存泄漏。



## ellipsis_inclusive_range_patterns
`ellipsis_inclusive_range_patterns` lint 检测 `...` 这种已经被遗弃的[范围模式](https://doc.rust-lang.org/reference/patterns.html#range-patterns)。  

**样例**

```rust
# #![allow(unused)]
# fn main() {
let x = 123;
match x {
    0...100 => {}
    _ => {}
}
# }
```
显示如下：
```text
warning: `...` range patterns are deprecated
 --> lint_example.rs:4:6
  |
4 |     0...100 => {}
  |      ^^^ help: use `..=` for an inclusive range
  |
  = note: `#[warn(ellipsis_inclusive_range_patterns)]` on by default
```
**解释**  

`...` 范围模式语法已经被改为 `..=`，以避免和 [`..` 范围表达式](https://doc.rust-lang.org/reference/expressions/range-expr.html)的潜在混乱。请使用新形式。

## exported_private_dependencies
`exported_private_dependencies` lint 检测在公共接口公开的私有依赖。

**样例**
```rust,ignore
pub fn foo() -> Option<some_private_dependency::Thing> {
    None
}
```
显示如下：
```text
warning: type `bar::Thing` from private dependency 'bar' in public interface
 --> src/lib.rs:3:1
  |
3 | pub fn foo() -> Option<bar::Thing> {
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(exported_private_dependencies)]` on by default
```
**解释**  

依赖可以被标记为 “私有” 以指明其不能在 crate 中的公共接口公开。可以被 Cargo 独立地解析这些依赖项，因为可以假定它不需要使用相同的依赖将它们与其他包统一。该 lint 指明违反了该规则。
要修复此问题，应避免在公共接口中公开此依赖，或将此依赖切换为公有依赖。
注意仅在 nightly channel 中支持此 lint 。更多细节请参阅 [RFC 1977](https://github.com/rust-lang/rfcs/blob/master/text/1977-public-private-dependencies.md)，以及 [Cargo 文档](https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#public-dependency)。

## function_item_references
`function_item_references` lint 检测使用 [`fmt::Pointer`][fmt-Pointer]格式化或转换的函数引用。

**样例**
```rust
fn foo() { }

fn main() {
    println!("{:p}", &foo);
}
```
显示如下：
```text
warning: taking a reference to a function item does not give a function pointer
 --> lint_example.rs:4:22
  |
4 |     println!("{:p}", &foo);
  |                      ^^^^ help: cast `foo` to obtain a function pointer: `foo as fn()`
  |
  = note: `#[warn(function_item_references)]` on by default
```
**解释**  

引用一个函数可能被误认为是获取函数指针的一种方式。将引用格式化为指针或对其进行转换的时候可能会产生意外的结果。当函数引用被格式化为指针，作为 [`fmt::Pointer`][fmt-Pointer] 约束的参数传递或转换时，就会触发该 lint。

[fmt-Pointer]:https://doc.rust-lang.org/std/fmt/trait.Pointer.html

## illegal_floating_point_literal_pattern
`illegal_floating_point_literal_pattern` lint 检测用于模式中的浮点数字面量。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let x = 42.0;

match x {
    5.0 => {}
    _ => {}
}
# }
```
显示如下：
```text
warning: floating-point types cannot be used in patterns
 --> lint_example.rs:5:5
  |
5 |     5.0 => {}
  |     ^^^
  |
  = note: `#[warn(illegal_floating_point_literal_pattern)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #41620 <https://github.com/rust-lang/rust/issues/41620>
```
**解释**  

早期版本的编译器接受用于模式中的浮点数字面量，但是后来确定了是个错误。当与”结构相等“进行对比时，比较浮点数值的语义可能不会再模式中明确。通常你可以通过使用 match 守卫（match guard） 来解决此问题，例如：
```rust
# #![allow(unused)]
# fn main() {
# let x = 42.0;

match x {
    y if y == 5.0 => {}
    _ => {}
}
# }
```
这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #41620](https://github.com/rust-lang/rust/issues/41620)。

## improper_ctypes
`improper_ctypes` lint 检测在外部模块中错误使用的类型。

**样例**

```rust
# #![allow(unused)]
# fn main() {
extern "C" {
    static STATIC: String;
}
# }
```
显示如下：
```text
warning: `extern` block uses type `String`, which is not FFI-safe
 --> lint_example.rs:3:20
  |
3 |     static STATIC: String;
  |                    ^^^^^^ not FFI-safe
  |
  = note: `#[warn(improper_ctypes)]` on by default
  = help: consider adding a `#[repr(C)]` or `#[repr(transparent)]` attribute to this struct
  = note: this struct has unspecified layout
```
**解释**  

编译器做了几项检查以验证外部块中使用的类型是安全的，并遵循某些规则以确保与外部接口的适当兼容性。当其在定义中检测到可能的错误时，将触发此 lint。该 lint 通常应该提供问题描述，并尽可能提示如何解决。

## improper_ctypes_definitions
`improper_ctypes_definitions` lint 检测对 [`extern` 函数](https://doc.rust-lang.org/reference/items/functions.html#extern-function-qualifier) 定义的错误使用。 

**样例**
```rust
# #![allow(unused)]
# fn main() {
# #![allow(unused)]
pub extern "C" fn str_type(p: &str) { }
# }
```
显示如下：
```text
warning: `extern` fn uses type `str`, which is not FFI-safe
 --> lint_example.rs:3:31
  |
3 | pub extern "C" fn str_type(p: &str) { }
  |                               ^^^^ not FFI-safe
  |
  = note: `#[warn(improper_ctypes_definitions)]` on by default
  = help: consider using `*const u8` and a length instead
  = note: string slices have no C equivalent
```
**解释**  

在 `extern` 函数中可能指定了许多与给定的 ABI 不兼容的参数和返回类型。该 lint 是一个关于这些类型都不应该使用的警告。该 lint 应该提供问题的描述，并尽可能提示如何解决问题。

## incomplete_features
`incomplete_features` lint 检测使用 [`feature`](https://doc.rust-lang.org/nightly/unstable-book/) 属性启用的不稳定 feature，这可能在一些或全部情况下不能正常工作。

**样例**
```rust
# #![allow(unused)]
#![feature(generic_associated_types)]
# fn main() {
# }
```
显示如下：
```text
warning: the feature `generic_associated_types` is incomplete and may not be safe to use and/or cause compiler crashes
 --> lint_example.rs:1:12
  |
1 | #![feature(generic_associated_types)]
  |            ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(incomplete_features)]` on by default
  = note: see issue #44265 <https://github.com/rust-lang/rust/issues/44265> for more information
```
**解释**  

尽管鼓励人们尝试不稳定的性能，其中的一些已知不完整或有缺陷。该 lint 是一个关于 feature 尚未完成的信号，并且你可能会遇到一些问题。

## indirect_structural_match
`indirect_structural_match` lint 检测手动实现的 [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html) 和 [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html) 的模式中的 `const`。

**样例**
```rust
#![deny(indirect_structural_match)]

struct NoDerive(i32);
impl PartialEq for NoDerive { fn eq(&self, _: &Self) -> bool { false } }
impl Eq for NoDerive { }
#[derive(PartialEq, Eq)]
struct WrapParam<T>(T);
const WRAP_INDIRECT_PARAM: & &WrapParam<NoDerive> = & &WrapParam(NoDerive(0));
fn main() {
    match WRAP_INDIRECT_PARAM {
        WRAP_INDIRECT_PARAM => { }
        _ => { }
    }
}
```
显示如下：
```text
error: to use a constant of type `NoDerive` in a pattern, `NoDerive` must be annotated with `#[derive(PartialEq, Eq)]`
  --> lint_example.rs:11:9
   |
11 |         WRAP_INDIRECT_PARAM => { }
   |         ^^^^^^^^^^^^^^^^^^^
   |
note: the lint level is defined here
  --> lint_example.rs:1:9
   |
1  | #![deny(indirect_structural_match)]
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #62411 <https://github.com/rust-lang/rust/issues/62411>
```

**解释**  

编译器过去无意间接受了此种形式。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。完整的问题描述和一些可能的解决办法请参阅 [issue #62411](https://github.com/rust-lang/rust/issues/62411)。

## inline_no_sanitize
`inline_no_sanitize` lint 检测 [`#[inline(always)]`][inline-always]和[`#[no_sanitize(...)]`][no-sanitize] 之间的不兼容性。

**样例**
```rust
#![feature(no_sanitize)]

#[inline(always)]
#[no_sanitize(address)]
fn x() {}

fn main() {
    x()
}
```
显示如下：
```text
warning: `no_sanitize` will have no effect after inlining
 --> lint_example.rs:4:1
  |
4 | #[no_sanitize(address)]
  | ^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(inline_no_sanitize)]` on by default
note: inlining requested here
 --> lint_example.rs:3:1
  |
3 | #[inline(always)]
  | ^^^^^^^^^^^^^^^^^
```
**解释**  

 [`#[inline(always)]`][inline-always] 属性的使用会阻止 [`#[no_sanitize(...)]`][no-sanitize] 属性正常工作。考虑暂时移除 `inline` 属性。

[inline-always]:https://doc.rust-lang.org/reference/attributes/codegen.html#the-inline-attribute
[no-sanitize]:https://doc.rust-lang.org/nightly/unstable-book/language-features/no-sanitize.html

## invalid_codeblock_attributes
`invalid_codeblock_attributes` lint 检测文档示例中那些可能有类型错误的代码块。这是个仅用于 `rustdoc` 的 lint ，请参阅 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#broken_intra_doc_links) 中的文档。

## invalid_value
`invalid_value` lint 检测创建的无效值，例如 NULL 引用。

**样例**
```rust
# #![allow(unused)]
# fn main() {
# #![allow(unused)]
unsafe {
    let x: &'static i32 = std::mem::zeroed();
}
# }
```
显示如下：
```text
warning: the type `&i32` does not permit zero-initialization
 --> lint_example.rs:4:27
  |
4 |     let x: &'static i32 = std::mem::zeroed();
  |                           ^^^^^^^^^^^^^^^^^^
  |                           |
  |                           this code causes undefined behavior when executed
  |                           help: use `MaybeUninit<T>` instead, and only call `assume_init` after initialization is done
  |
  = note: `#[warn(invalid_value)]` on by default
  = note: references must be non-null
```
**解释**  

一些情况下，编译器可以检测到代码创建了无效的值，这应该是避免的。
特别地，该 lint 会检测是否有不恰当的使用 [`mem::zeroed`](https://doc.rust-lang.org/std/mem/fn.zeroed.html)，[`mem::uninitialized`](https://doc.rust-lang.org/std/mem/fn.uninitialized.html)，[`mem::transmute`](https://doc.rust-lang.org/std/mem/fn.transmute.html)和[`MaybeUninit::assume_init`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#method.assume_init) 可能造成[未定义行为](https://doc.rust-lang.org/reference/behavior-considered-undefined.html)。

## irrefutable_let_patterns
`irrefutable_let_patterns` lint 检测 在 [if-let][https://doc.rust-lang.org/reference/expressions/if-expr.html#if-let-expressions] 和 [while-let][https://doc.rust-lang.org/reference/expressions/loop-expr.html#predicate-pattern-loops] 语句中的[不可辩驳模式][https://doc.rust-lang.org/reference/patterns.html#refutability] 。

**样例**
```rust
# #![allow(unused)]
# fn main() {
if let _ = 123 {
    println!("always runs!");
}
# }
```
显示如下：
```text
warning: irrefutable if-let pattern
 --> lint_example.rs:2:1
  |
2 | / if let _ = 123 {
3 | |     println!("always runs!");
4 | | }
  | |_^
  |
  = note: `#[warn(irrefutable_let_patterns)]` on by default
```
**解释**  

通常没理由在 if-let 或 while-let 语句中使用不可辩驳模式，因为这样的话模式总是会匹配成功，要是这样的话 [`let`](https://doc.rust-lang.org/reference/statements.html#let-statements) 或 [`loop`](https://doc.rust-lang.org/reference/expressions/loop-expr.html#infinite-loops) 语句就够了。然而，当用宏生成代码时，在宏不知道模式是否是可辨驳的情况下，禁止不可辩驳模式是一种笨拙的解决办法。该 lint 允许宏接受此形式，并警告普通代码这可能是不正确的使用。
更多细节请参阅 [RFC 2086](https://github.com/rust-lang/rfcs/blob/master/text/2086-allow-if-let-irrefutables.md)。

## late_bound_lifetime_arguments
`late_bound_lifetime_arguments` lint 检测后绑定生命周期参数路径段中的泛型生命周期参数。

**样例**
```rust
struct S;

impl S {
    fn late<'a, 'b>(self, _: &'a u8, _: &'b u8) {}
}

fn main() {
    S.late::<'static>(&0, &0);
}
```
显示如下：
```text
warning: cannot specify lifetime arguments explicitly if late bound lifetime parameters are present
 --> lint_example.rs:8:14
  |
4 |     fn late<'a, 'b>(self, _: &'a u8, _: &'b u8) {}
  |             -- the late bound lifetime parameter is introduced here
...
8 |     S.late::<'static>(&0, &0);
  |              ^^^^^^^
  |
  = note: `#[warn(late_bound_lifetime_arguments)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #42868 <https://github.com/rust-lang/rust/issues/42868>
```
**解释**  

如果将先绑定生命周期参数与同一参数列表中的后绑定生命周期参数混合在一起，则不清楚如何为其提供参数。目前，如果存在后绑定参数，提供显式参数将触发此 lint。因此将来解决方案可以被采用而不会遇到向后兼容性问题。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节以及先绑定和后绑定之间差异的描述请参阅 [issue #42868](https://github.com/rust-lang/rust/issues/42868)。

## mixed_script_confusables
`mixed_script_confusables` lint 检测在不同[脚本](https://en.wikipedia.org/wiki/Script_(Unicode))标识符(identifiers between different scripts)间的可视性易混淆的字符。

**样例**
```rust
# #![allow(unused)]
#![feature(non_ascii_idents)]

# fn main() {
// The Japanese katakana character エ can be confused with the Han character 工.
const エ: &'static str = "アイウ";
# }
```
显示如下：
```text
warning: The usage of Script Group `Japanese, Katakana` in this crate consists solely of mixed script confusables
 --> lint_example.rs:5:7
  |
5 | const エ: &'static str = "アイウ";
  |       ^^
  |
  = note: `#[warn(mixed_script_confusables)]` on by default
  = note: The usage includes 'エ' (U+30A8).
  = note: Please recheck to make sure their usages are indeed what you want.
```
**解释**  

上面的 [`non_ascii_idents`][non-ascii-idents] 是只能用于 nightly的，其允许使用非 ASCII 字符作为标识符。该 lint 当不同脚本字符在视觉上看起来很像的时候发出警告，因为（长得像）这可能会造成混乱。
如果 crate包含在相同的脚本中不会引起混淆的字符，那么此 lint 将不会被触发。例如，如果上例还有另一个带有片假名字符的标识符（例如 `let カタカナ = 123;`），然后就会表明你在有意使用片假名，并且不会对此发出警告。
请注意，易混淆字符集可能会随时间而变化。注意，如果你将该 lint 等级调整为 “禁止”，则现有代码可能在将来会（编译）失败。

## mutable_borrow_reservation_conflict
`mutable_borrow_reservation_conflict` lint 检测与其他共享借用相冲突的第二阶段借用这种保留。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let mut v = vec![0, 1, 2];
let shared = &v;
v.push(shared.len());
# }
```
显示如下：
```text
warning: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> lint_example.rs:4:1
  |
3 | let shared = &v;
  |              -- immutable borrow occurs here
4 | v.push(shared.len());
  | ^      ------ immutable borrow later used here
  | |
  | mutable borrow occurs here
  |
  = note: `#[warn(mutable_borrow_reservation_conflict)]` on by default
  = warning: this borrowing pattern was not meant to be accepted, and may become a hard error in the future
  = note: for more information, see issue #59159 <https://github.com/rust-lang/rust/issues/59159>
```
**解释**  

这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。完整问题描述和一些可能的解决办法请参阅 [issue #59159](https://github.com/rust-lang/rust/issues/59159)。

## no_mangle_generic_items
`no_mangle_generic_items` lint 检测必须使用 改发（mangle）的泛型项。

**样例**
```rust
# #![allow(unused)]
# fn main() {
#[no_mangle]
fn foo<T>(t: T) {

}
# }
```
显示如下：
```text
warning: functions generic over types or consts must be mangled
 --> lint_example.rs:3:1
  |
2 |   #[no_mangle]
  |   ------------ help: remove this attribute
3 | / fn foo<T>(t: T) {
4 | |
5 | | }
  | |_^
  |
  = note: `#[warn(no_mangle_generic_items)]` on by default
```
**解释**  

泛型函数必须改发其符号以适应泛型参数。[`no_mangle`](https://doc.rust-lang.org/reference/abi.html#the-no_mangle-attribute) 属性对此无效，应该被移除。

## non_autolinks
`non_autolinks` lint 检测何时仅能用尖括号写入 URL 。这是一个仅用于 `rustdoc` 的lint，请查看 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#broken_intra_doc_links)中的文档。

## non_camel_case_types
`non_camel_case_types` lint 检测没有使用驼峰命名的类型、变量、trait 和类型参数。

**样例**
```rust
# #![allow(unused)]
# fn main() {
struct my_struct;
# }
```
显示如下：
```text
warning: type `my_struct` should have an upper camel case name
 --> lint_example.rs:2:8
  |
2 | struct my_struct;
  |        ^^^^^^^^^ help: convert the identifier to upper camel case: `MyStruct`
  |
  = note: `#[warn(non_camel_case_types)]` on by default
```
**解释**  

标识符的首选样式是使用 ”驼峰大小写“，例如 `MyStruct`，其中首字母不应小写，且字母之间不应使用下划线。在标识符的开头和结尾以及非字母之间（例如 `X86_64`），都可以使用下划线。

## non_shorthand_field_patterns
`non_shorthand_field_patterns` lint 检测在模式中使用 `Struct { x: x }` 而非 `Struct { x }`。

**样例**
```rust
struct Point {
    x: i32,
    y: i32,
}


fn main() {
    let p = Point {
        x: 5,
        y: 5,
    };

    match p {
        Point { x: x, y: y } => (),
    }
}
```
显示如下：
```text
warning: the `x:` in this pattern is redundant
  --> lint_example.rs:14:17
   |
14 |         Point { x: x, y: y } => (),
   |                 ^^^^ help: use shorthand field pattern: `x`
   |
   = note: `#[warn(non_shorthand_field_patterns)]` on by default
```
**解释**  

首选的样式是避免在两个标识符相同的情况下重复指定字段名和绑定名。

## non_snake_case
`non_snake_case` lint 检测没有使用蛇形命名的变量、方法、函数、生命周期参数和模块。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let MY_VALUE = 5;
# }
```
显示如下：
```text
warning: variable `MY_VALUE` should have a snake case name
 --> lint_example.rs:2:5
  |
2 | let MY_VALUE = 5;
  |     ^^^^^^^^ help: convert the identifier to snake case: `my_value`
  |
  = note: `#[warn(non_snake_case)]` on by default
```
**解释**  
标识符首选样式是使用蛇形命名，即所有字符均为小写，单词之间用单个下划线分隔，例如 `my_value`。

## non_upper_case_globals
`non_upper_case_globals` lint 检测没有使用大写标识符的 static 项。

**样例**

```rust
# #![allow(unused)]
# fn main() {
static max_points: i32 = 5;
# }
```
显示如下：
```text
warning: static variable `max_points` should have an upper case name
 --> lint_example.rs:2:8
  |
2 | static max_points: i32 = 5;
  |        ^^^^^^^^^^ help: convert the identifier to upper case: `MAX_POINTS`
  |
  = note: `#[warn(non_upper_case_globals)]` on by default
```
**解释**  

静态项命名的首选样式是都使用大写，例如 `MAX_POINTS`。

## nontrivial_structural_match
`nontrivial_structural_match` lint 检测用于模式中的常量，该常量类型是非结构化匹配（not structural match）的且初始化体实际上所使用的值是非结构化匹配的。所以如果常数仅是 `None`，`Option<NotStruturalMatch>` 是对的。

**样例**
```rust
#![deny(nontrivial_structural_match)]

#[derive(Copy, Clone, Debug)]
struct NoDerive(u32);
impl PartialEq for NoDerive { fn eq(&self, _: &Self) -> bool { false } }
impl Eq for NoDerive { }
fn main() {
    const INDEX: Option<NoDerive> = [None, Some(NoDerive(10))][0];
    match None { Some(_) => panic!("whoops"), INDEX => dbg!(INDEX), };
}
```
显示如下：
```text
error: to use a constant of type `NoDerive` in a pattern, the constant's initializer must be trivial or `NoDerive` must be annotated with `#[derive(PartialEq, Eq)]`
 --> lint_example.rs:9:47
  |
9 |     match None { Some(_) => panic!("whoops"), INDEX => dbg!(INDEX), };
  |                                               ^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(nontrivial_structural_match)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #73448 <https://github.com/rust-lang/rust/issues/73448>
```
**解释**  

早期的 Rust 接受在模式中使用常量，甚至就算常量类型没有派生 `PartialEq` 。因此编译器会回退到 `PartialEq` 执行的运行时，即使两个常量位等效该运行时也报告为不等。

## overlapping_patterns
`overlapping_patterns` lint 检测具有重叠的[范围模式](https://doc.rust-lang.org/nightly/reference/patterns.html#range-patterns)的 `match` 分支。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let x = 123u8;
match x {
    0..=100 => { println!("small"); }
    100..=255 => { println!("large"); }
}
# }
```
显示如下：
```text
warning: multiple patterns covering the same range
 --> lint_example.rs:5:5
  |
4 |     0..=100 => { println!("small"); }
  |     ------- this range overlaps on `100_u8`
5 |     100..=255 => { println!("large"); }
  |     ^^^^^^^^^ overlapping patterns
  |
  = note: `#[warn(overlapping_patterns)]` on by default
```
**解释**  

 在 match 表达式中重叠的范围模式可能是错误的。检查开始和结束值是否符合你的期望，并记住，使用 `..=` 时，左边界和右边界是包括在内的。

## path_statements
`path_statements` lint 检测无效的路径语句（path statements）。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let x = 42;

x;
# }
```
显示如下：
```text
warning: path statement with no effect
 --> lint_example.rs:4:1
  |
4 | x;
  | ^^
  |
  = note: `#[warn(path_statements)]` on by default
```
**解释**  

无效的语句通常是个错误。

## private_in_public
`private_in_public` lint 检测以前的实现没有捕获的公有接口中的私有项。

**样例**
```rust
# #![allow(unused)]
struct SemiPriv;

mod m1 {
    struct Priv;
    impl super::SemiPriv {
        pub fn f(_: Priv) {}
    }
}
# fn main() {}
```
显示如下：
```text
warning: private type `Priv` in public interface (error E0446)
 --> lint_example.rs:7:9
  |
7 |         pub fn f(_: Priv) {}
  |         ^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(private_in_public)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #34537 <https://github.com/rust-lang/rust/issues/34537>
```
**解释**  

可见性规则旨在防止公共接口公开私有项。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #34537](https://github.com/rust-lang/rust/issues/34537)。

## private_intra_doc_links
该 lint 是当从公有项链接到私有项时会发出警告的 `broken_intra_doc_links` 的子集。这是一个仅用于 `rustdoc` 的lint，请查看 [rustdoc book](https://doc.rust-lang.org/rustdoc/lints.html#broken_intra_doc_links)中的文档。


## proc_macro_derive_resolution_fallback
`proc_macro_derive_resolution_fallback` lint 检测使用父模块中无法访问的名称的 proc 宏派生。

**样例**
```rust,ignore
// foo.rs
#![crate_type = "proc-macro"]

extern crate proc_macro;

use proc_macro::*;

#[proc_macro_derive(Foo)]
pub fn foo1(a: TokenStream) -> TokenStream {
    drop(a);
    "mod __bar { static mut BAR: Option<Something> = None; }".parse().unwrap()
}
```

```rust,ignore
// bar.rs
#[macro_use]
extern crate foo;

struct Something;

#[derive(Foo)]
struct Another;

fn main() {}
```
显示如下：
```text
warning: cannot find type `Something` in this scope
 --> src/main.rs:8:10
  |
8 | #[derive(Foo)]
  |          ^^^ names from parent modules are not accessible without an explicit import
  |
  = note: `#[warn(proc_macro_derive_resolution_fallback)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #50504 <https://github.com/rust-lang/rust/issues/50504>
```
**解释**  

如果 proc-macro 生成了一个模块，则编译器会无意地允许该模块中的项引用 crate 根中的项而不用导入。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #50504](https://github.com/rust-lang/rust/issues/50504)。

## redundant_semicolons
`redundant_semicolons` lint 检测不必要的尾部分号。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let _ = 123;;
# }
```
显示如下：
```text
warning: unnecessary trailing semicolon
 --> lint_example.rs:2:13
  |
2 | let _ = 123;;
  |             ^ help: remove this semicolon
  |
  = note: `#[warn(redundant_semicolons)]` on by default
```
**解释**  

多余的分号是不需要的，可以将其删除以避免混淆和视觉混乱。

## renamed_and_removed_lints
`renamed_and_removed_lints` lint 检测已经呗重命名或移除的 lint 。

**样例**
```rust
# #![allow(unused)]
#![deny(raw_pointer_derive)]
# fn main() {
# }
```
显示如下：
```text
warning: lint `raw_pointer_derive` has been removed: `using derive with raw pointers is ok`
 --> lint_example.rs:1:9
  |
1 | #![deny(raw_pointer_derive)]
  |         ^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(renamed_and_removed_lints)]` on by default
```
**解释**  

要修复此问题，要么移除此 lint，要么使用推荐的新名字。这可以帮助避免与不再有效的 lint 的混淆，并且保持重命名后的 lint 的一致性。

## safe_packed_borrows
`safe_packed_borrows` lint 检测借用包装结构体对齐方式不是1 的字段。

**样例**
```rust
#[repr(packed)]
pub struct Unaligned<T>(pub T);

pub struct Foo {
    start: u8,
    data: Unaligned<u32>,
}

fn main() {
    let x = Foo { start: 0, data: Unaligned(1) };
    let y = &x.data.0;
}
```
显示如下：
```text
warning: borrow of packed field is unsafe and requires unsafe function or block (error E0133)
  --> lint_example.rs:11:13
   |
11 |     let y = &x.data.0;
   |             ^^^^^^^^^
   |
   = note: `#[warn(safe_packed_borrows)]` on by default
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #46043 <https://github.com/rust-lang/rust/issues/46043>
   = note: fields of packed structs might be misaligned: dereferencing a misaligned pointer or even just creating a misaligned reference is undefined behavior
```

**解释**  

这种借用方式是不安全的，且可能再某些平台上导致错误和违反编译器所做的某些假设，以前无意中会允许这样做。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。有关如何解决该问题的引导和更多细节请参阅 [issue #46043](https://github.com/rust-lang/rust/issues/46043)。

## stable_features
`stable_features` lint 检测已经变为 stable 的 [`feature`](https://doc.rust-lang.org/nightly/unstable-book/) 属性。

**样例**
```rust
#![feature(test_accepted_feature)]
fn main() {}
```
显示如下：
```text
warning: the feature `test_accepted_feature` has been stable since 1.0.0 and no longer requires an attribute to enable
 --> lint_example.rs:1:12
  |
1 | #![feature(test_accepted_feature)]
  |            ^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(stable_features)]` on by default
```
**解释**  

当一个 feature 稳定后，就不再需要包含 `#![feature]` 属性了。要修复此问题，只需简单地移除 `#![feature]` 属性就行。

## temporary_cstring_as_ptr
`temporary_cstring_as_ptr` lint 检测临时获取 `CString` 的内部指针。

**样例**
```rust
# #![allow(unused)]
# fn main() {
# #![allow(unused)]
# use std::ffi::CString;
let c_str = CString::new("foo").unwrap().as_ptr();
# }
```
显示如下：
```text
warning: getting the inner pointer of a temporary `CString`
 --> lint_example.rs:4:42
  |
4 | let c_str = CString::new("foo").unwrap().as_ptr();
  |             ---------------------------- ^^^^^^ this pointer will be invalid
  |             |
  |             this `CString` is deallocated at the end of the statement, bind it to a variable to extend its lifetime
  |
  = note: `#[warn(temporary_cstring_as_ptr)]` on by default
  = note: pointers do not have a lifetime; when calling `as_ptr` the `CString` will be deallocated at the end of the statement because nothing is referencing it as far as the type system is concerned
  = help: for more information, see https://doc.rust-lang.org/reference/destructors.html
```
**解释**  

 `CString` 的内部指针 存活时间和其指向的 `CString` 一样长。获取临时的 `CString` 的内部指针允许在语句结尾释放 `CString` ，因为就类型系统而言，其并未被引用。这意味着在语句之外，该指针将会指向已释放的内存，如果之后解引用指针会导致未定义的行为。

## trivial_bounds
`trivial_bounds` lint 检测没有依赖任何参数的 triat 约束。

**样例**
```rust
# #![allow(unused)]
#![feature(trivial_bounds)]
# fn main() {
pub struct A where i32: Copy;
# }
```
显示如下：
```text
warning: Trait bound i32: Copy does not depend on any type or lifetime parameters
 --> lint_example.rs:3:25
  |
3 | pub struct A where i32: Copy;
  |                         ^^^^
  |
  = note: `#[warn(trivial_bounds)]` on by default
```
**解释**  

通常，你不会写出一个你知道它永远是对的，或者永远不对的 trait 约束。然而，使用宏时，宏可能不知道在生成代码时约束是否成立。当前，如果约束始终正确，编译器不会警告你；如果约束不对，编译器会生成错误。在这两种情况下， `trivial_bounds` feature 都将其更改为警告，从而使得宏有更大的自由度和灵活性来生成代码，同时在产生存在问题的非宏代码时会发出相应信号表明存在问题。
更多细节请参阅 [RFC 2056](https://github.com/rust-lang/rfcs/blob/master/text/2056-allow-trivial-where-clause-constraints.md)。该 feature 目前仅在 nightly channel 有效，跟踪问题请参阅 [issue #48214](https://github.com/rust-lang/rust/issues/48214)。

## type_alias_bounds
`type_alias_bounds` lint 检测类型别名中的约束。

**样例**
```rust
# #![allow(unused)]
# fn main() {
type SendVec<T: Send> = Vec<T>;
# }
```
显示如下：
```text
warning: bounds on generic parameters are not enforced in type aliases
 --> lint_example.rs:2:17
  |
2 | type SendVec<T: Send> = Vec<T>;
  |                 ^^^^
  |
  = note: `#[warn(type_alias_bounds)]` on by default
help: the bound will not be checked when the type alias is used, and should be removed
  |
2 | type SendVec<T> = Vec<T>;
  |              --
```
**解释**  

类型别名中的 trait 约束 当前是被忽略的，并且不应该包含在内以免造成混淆。以前会无意中允许这样做，这在将来可能会转换为固有错误。

## tyvar_behind_raw_pointer
`tyvar_behind_raw_pointer` lint 检测指向推断变量（inference variable）的原生指针（raw pointer）。

**样例**
```rust
# #![allow(unused)]
# fn main() {
// edition 2015
let data = std::ptr::null();
let _ = &data as *const *const ();

if data.is_null() {}
# }
```
显示如下：
```text
warning: type annotations needed
 --> lint_example.rs:6:9
  |
6 | if data.is_null() {}
  |         ^^^^^^^
  |
  = note: `#[warn(tyvar_behind_raw_pointer)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in the 2018 edition!
  = note: for more information, see issue #46906 <https://github.com/rust-lang/rust/issues/46906>
```
**解释**  

这种推断以前是允许的，但是随着将来 [arbitrary self type](https://github.com/rust-lang/rust/issues/44874)的引入，这可能会引起歧义。要解决此问题，请使用显式类型而非依赖类型推导。
这是个[将来不兼容][future-incompatible] 的 lint ，在 2018 版本中会转化为固有错误。更多细节请参阅 [issue #46906](https://github.com/rust-lang/rust/issues/46906) 。目前在 2018 版本中是个固有错误，在2018版本中默认等级是警告。

## uncommon_codepoints
`uncommon_codepoints` lint 检测在标识符中不常见的 Unicode 字符码（codepoint）。

**样例**
```rust
# #![allow(unused)]
# fn main() {
# #![allow(unused)]
#![feature(non_ascii_idents)]
const µ: f64 = 0.000001;
# }
```
显示如下：
```text
warning: identifier contains uncommon Unicode codepoints
 --> lint_example.rs:4:7
  |
4 | const µ: f64 = 0.000001;
  |       ^
  |
  = note: `#[warn(uncommon_codepoints)]` on by default
```
**解释**  

上面的 [`non_ascii_idents`][non-ascii-idents] 是只能用于 nightly的，其允许使用非 ASCII 字符作为标识符。该 lint发出警告不要使用不常用字符，并且可能会引起视觉混乱。
该 lint 由包含不属于 “Allowed” 字符码集的字符码的标识符所触发，该 “Allowed” 字符码集被描述为 [Unicode® Technical Standard #39 Unicode Security Mechanisms Section 3.1 General Security Profile for Identifiers](https://www.unicode.org/reports/tr39/#General_Security_Profile)。

## unconditional_recursion
`unconditional_recursion` lint 检测不调用自身无法返回的函数。

**样例**
```rust
# #![allow(unused)]
# fn main() {
fn foo() {
    foo();
}
# }
```
显示如下：
```text
warning: function cannot return without recursing
 --> lint_example.rs:2:1
  |
2 | fn foo() {
  | ^^^^^^^^ cannot return without recursing
3 |     foo();
  |     ----- recursive call site
  |
  = note: `#[warn(unconditional_recursion)]` on by default
  = help: a `loop` may express intention better if this is on purpose
```
**解释**  

进行没有一定条件终止的递归调用通常是个错误。如果确实想要进行无限循环，推荐使用 `loop` 表达式。

## uninhabited_static
`uninhabited_static` lint 检测 uninhabited  静态项。（译者想把 uninhabited 翻译为空巢，但是想想还是算了，太花哨了）

**样例**

```rust
# #![allow(unused)]
# fn main() {
enum Void {}
extern {
    static EXTERN: Void;
}
# }
```
显示如下：
```text
warning: static of uninhabited type
 --> lint_example.rs:4:5
  |
4 |     static EXTERN: Void;
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(uninhabited_static)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #74840 <https://github.com/rust-lang/rust/issues/74840>
  = note: uninhabited statics cannot be initialized, and any access would be an immediate error
```
**解释**  

一个  uninhabited 的 static 类型永远不会被初始化，因此无法定义。然而，该问题可以使用 `extern static` 来避开。（uninhabited static）假定其没有初始化的 uninhabited 的地方（例如本地或静态变量）。（uninhabited static）确实被允许这么做，但是其正在被淘汰。

## unknown_lints
`unknown_lints` lint 无法识别的 lint 属性。

**样例**
```rust
# #![allow(unused)]
#![allow(not_a_real_lint)]
# fn main() {
# }
```
显示如下：
```text
warning: unknown lint: `not_a_real_lint`
 --> lint_example.rs:1:10
  |
1 | #![allow(not_a_real_lint)]
  |          ^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unknown_lints)]` on by default
```
**解释**  

指定一个不存在的 lint 通常是个错误。检查拼写和 lint 列表中的正确名称是否相同。同时考虑是否是在使用旧版本的编译器，而此 lint 仅在新版本中可用。

## unnameable_test_items
`unnameable_test_items` lint 检测不能由测试工具运行的 [`#[test]`](https://doc.rust-lang.org/reference/attributes/testing.html#the-test-attribute) 函数，因为它们处于不可命名的地方。

**样例**
```rust
fn main() {
    #[test]
    fn foo() {
        // This test will not fail because it does not run.
        assert_eq!(1, 2);
    }
}
```
显示如下：
```text
warning: cannot test inner items
 --> lint_example.rs:2:5
  |
2 |     #[test]
  |     ^^^^^^^
  |
  = note: `#[warn(unnameable_test_items)]` on by default
  = note: this warning originates in an attribute macro (in Nightly builds, run with -Z macro-backtrace for more info)
```
**解释**  

为了让测试工具能够进行测试，测试函数必须位于可以从 crate 根访问的位置。通常来说，这意味着必须在模块中定义它，而不是在其他地方，例如在另一个函数中定义。编译器以前允许这样做而没有发出错误消息，因此添加了一个 lint 发出未使用测试的警告。如今尚未确定是否应该允许这么做，请参阅 [RFC 2471](https://github.com/rust-lang/rfcs/pull/2471#issuecomment-397414443) 和 [issue #36629](https://github.com/rust-lang/rust/issues/36629) 。

## unreachable_code
`unreachable_code` lint 检测无法到达的代码路径。

**样例**
```rust,ignore
# #![allow(unused)]
# fn main() {
panic!("we never go past here!");

let x = 5;
# }
```
显示如下：
```text
warning: unreachable statement
 --> lint_example.rs:4:1
  |
2 | panic!("we never go past here!");
  | --------------------------------- any code following this expression is unreachable
3 | 
4 | let x = 5;
  | ^^^^^^^^^^ unreachable statement
  |
  = note: `#[warn(unreachable_code)]` on by default
```
**解释**  

无法到达的代码可能意味着是个错误或代码未完成。如果代码不再使用，请考虑移除它。

## unreachable_patterns
`unreachable_patterns` lint 检测无法到达的模式。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let x = 5;
match x {
    y => (),
    5 => (),
}
# }
```
显示如下：
```text
warning: unreachable pattern
 --> lint_example.rs:5:5
  |
4 |     y => (),
  |     - matches any value
5 |     5 => (),
  |     ^ unreachable pattern
  |
  = note: `#[warn(unreachable_patterns)]` on by default
```
**解释**  

这通常意味着模式的指定或顺序有误。在上例中，`y` 模式总是会匹配，所以 5 是不可能到达的。记住，match 分支是按顺序匹配的，你可以将 `5` 调整到 `y` 的上面。

## unstable_name_collisions
`unstable_name_collisions` lint 检测使用了标准库计划在将来添加的名称。

**样例**
```rust
# #![allow(unused)]
# fn main() {
trait MyIterator : Iterator {
    // is_sorted is an unstable method that already exists on the Iterator trait
    fn is_sorted(self) -> bool where Self: Sized {true}
}

impl<T: ?Sized> MyIterator for T where T: Iterator { }

let x = vec![1,2,3];
let _ = x.iter().is_sorted();
# }
```
显示如下：
```text
warning: a method with this name may be added to the standard library in the future
  --> lint_example.rs:10:18
   |
10 | let _ = x.iter().is_sorted();
   |                  ^^^^^^^^^
   |
   = note: `#[warn(unstable_name_collisions)]` on by default
   = warning: once this method is added to the standard library, the ambiguity may cause an error or change in behavior!
   = note: for more information, see issue #48919 <https://github.com/rust-lang/rust/issues/48919>
   = help: call with fully qualified syntax `MyIterator::is_sorted(...)` to keep using the current method
   = help: add `#![feature(is_sorted)]` to the crate attributes to enable `is_sorted`
```
**解释**  

当标准库中的 trait 添加了新方法之时，它们通常在具有 `feature` 属性的 nightly channel 中以 “unstable” 形式添加。如果有任何之前已存在的代码扩展了具有同名方法的 trait，则这些名称将会发生冲突。将来，当方法稳定后，由于歧义将会造成错误。该 lint 是一个让你知道将来可能会发生碰撞的预警。可以通过添加类型注解来消除要调用的 trait 方法的歧义避免此歧义，例如 `MyIterator::is_sorted(my_iter)` 或重名或删除该方法。

## unused_allocation
`unused_allocation` lint 检测可以被消除的不必要的（内存）分配。

**样例**
```rust
#![feature(box_syntax)]
fn main() {
    let a = (box [1,2,3]).len();
}
```
显示如下：
```text
warning: unnecessary allocation, use `&` instead
 --> lint_example.rs:3:13
  |
3 |     let a = (box [1,2,3]).len();
  |             ^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_allocation)]` on by default
```
**解释**  

当一个 `box` 表达式立即被强转为引用时，说明这个分配时不必要的，且应该使用引用（ 使用 `&` 或 `&mut` ）来避免引用。

## unused_assignments
`unused_assignments` lint 检测从未被读取的赋值。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let mut x = 5;
x = 6;
# }
```
显示如下：
```text
warning: value assigned to `x` is never read
 --> lint_example.rs:3:1
  |
3 | x = 6;
  | ^
  |
  = note: `#[warn(unused_assignments)]` on by default
  = help: maybe it is overwritten before being read?
```
**解释**  

未使用的赋值可能意味着是个错误或未完成的代码。如果变量自赋值之后就从未被使用，那么这个赋值也可以被移除。带有下划线前缀的变量例如 `_x` 将不会触发此 lint 。

## unused_attributes
`unused_attributes` lint 检测编译器未使用的属性。

**样例**
```rust
# #![allow(unused)]
#![ignore]
# fn main() {
# }
```
显示如下：
```text
warning: unused attribute
 --> lint_example.rs:1:1
  |
1 | #![ignore]
  | ^^^^^^^^^^
  |
  = note: `#[warn(unused_attributes)]` on by default
```
**解释**  

未使用的[属性](https://doc.rust-lang.org/reference/attributes.html)可能意味着属性放在了错误的位置。考虑移除它，或将其放在正确的地方。还应考虑是否使用属性所在项的内部属性（用 `!`，例如`#![allow(unused)]`），或者应用于属性后面项的外部属性（没有 `!`，例如 `[allow(unused)]`）。

## unused_braces
`unused_braces` lint 检测表达式周边不必要的大括号。

**样例**
```rust
# #![allow(unused)]
# fn main() {
if { true } {
    // ...
}
# }
```
显示如下：
```text
warning: unnecessary braces around `if` condition
 --> lint_example.rs:2:4
  |
2 | if { true } {
  |    ^^^^^^^^ help: remove these braces
  |
  = note: `#[warn(unused_braces)]` on by default
```
**解释**  

该大括号是不需要的，应该将其移除。这是编写这些表达式的首选样式。

## unused_comparisons
`unused_comparisons` lint 检测由于所涉及类型的限制而变得无用的比较。

**样例**
```rust
# #![allow(unused)]
# fn main() {
fn foo(x: u8) {
    x >= 0;
}
# }
```
显示如下：
```text
warning: comparison is useless due to type limits
 --> lint_example.rs:3:5
  |
3 |     x >= 0;
  |     ^^^^^^
  |
  = note: `#[warn(unused_comparisons)]` on by default
```
**解释**  

一个无用的比较或许表明是一个错误，或者应该被修复或移除。

## unused_doc_comments
`unused_doc_comments` lint 检测并未用于 `rustdoc` 的文档注释。

**样例**
```rust
# #![allow(unused)]
# fn main() {
/// docs for x
let x = 12;
# }
```
显示如下：
```text
warning: unused doc comment
 --> lint_example.rs:2:1
  |
2 | /// docs for x
  | ^^^^^^^^^^^^^^
3 | let x = 12;
  | ----------- rustdoc does not generate documentation for statements
  |
  = note: `#[warn(unused_doc_comments)]` on by default
```
**解释**  

`rustdoc` 并不会使用所有地方的文档注释，因此某些文档注释将会被忽略。尝试使用 // 将其改为普通注释，以免出现警告。

## unused_features
`unused_features` lint 检测未使用或未知的在 crate-level [`feature`](https://doc.rust-lang.org/nightly/unstable-book/) 属性找到的 feature 。

注意：该 lint 目前还无法运作，更多细节请参阅 [issue #44232](https://github.com/rust-lang/rust/issues/44232)。

## unused_imports
`unused_imports` lint 检测从未被使用的 import 项。

**样例**
```rust
# #![allow(unused)]
# fn main() {
use std::collections::HashMap;
# }
```
显示如下：
```text
warning: unused import: `std::collections::HashMap`
 --> lint_example.rs:2:5
  |
2 | use std::collections::HashMap;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default
```
**解释**  

未使用的 import 项可能意味着是个错误或未完成的代码，并且会使代码混乱，应该将其移除。如果要重导出项使得在模块外可用，请添加可见修饰符如 `pub` 。

## unused_labels
`unused_labels` lint  检测未使用的 [标记](https://doc.rust-lang.org/reference/expressions/loop-expr.html#loop-labels)。 

**样例**

```rust
# #![allow(unused)]
# fn main() {
'unused_label: loop {}
# }
```
显示如下：
```text
warning: unused label
 --> lint_example.rs:2:1
  |
2 | 'unused_label: loop {}
  | ^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_labels)]` on by default
```
**解释**  

未使用的标记可能意味着是个错误或未完成的代码。要使单个标记的该警告沉默，在前面添加下划线，如 `'_my_label:`。

## unused_macros
`unused_macros` lint 检测从未使用的宏。

**样例**
```rust
macro_rules! unused {
    () => {};
}

fn main() {
}
```
显示如下：
```text
warning: unused macro definition
 --> lint_example.rs:1:1
  |
1 | / macro_rules! unused {
2 | |     () => {};
3 | | }
  | |_^
  |
  = note: `#[warn(unused_macros)]` on by default
```
**解释**  

未使用的宏可能意味着是个错误或未完成的代码。使单个宏的该警告沉默，在前面添加下划线，如 `'_my_macro`。如果想要导出宏以使其在 crate 之外可用，请使用 [`macro_export`](https://doc.rust-lang.org/reference/macros-by-example.html#path-based-scope) 属性。

## unused_must_use
`unused_must_use` lint 检测被标记为 `#[must_use]` 却没被使用的类型的 result 。

**样例**
```rust
fn returns_result() -> Result<(), ()> {
    Ok(())
}

fn main() {
    returns_result();
}
```
显示如下：
```text
warning: unused `std::result::Result` that must be used
 --> lint_example.rs:6:5
  |
6 |     returns_result();
  |     ^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: this `Result` may be an `Err` variant, which should be handled
```
**解释**  

`#[must_use]` 属性表明忽略返回值是个错误。更多细节请参阅 [reference](https://doc.rust-lang.org/reference/attributes/diagnostics.html#the-must_use-attribute)。

## unused_mut
`unused_mut` lint 不需要可变的可变变量。

**样例**
```rust
# #![allow(unused)]
# fn main() {
let mut x = 5;
# }
```
显示如下：
```text
warning: variable does not need to be mutable
 --> lint_example.rs:2:5
  |
2 | let mut x = 5;
  |     ----^
  |     |
  |     help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```
**解释**  

首选样式是仅在需要时才将变量标记为 mut 。

## unused_parens
`unused_parens` lint 检测 `if`，`match`，`while`和`return` 带有圆括号，它们不需要圆括号。

**样例**
```rust
# #![allow(unused)]
# fn main() {
if(true) {}
# }
```
显示如下：
```text
warning: unnecessary parentheses around `if` condition
 --> lint_example.rs:2:3
  |
2 | if(true) {}
  |   ^^^^^^ help: remove these parentheses
  |
  = note: `#[warn(unused_parens)]` on by default
```
**解释**  

圆括号是不需要的，应该被移除。这是这些表达式的首选样式。

## unused_unsafe
`unused_unsafe` lint 检测不必要的 `unsafe` 块的使用。

**样例**
```rust
# #![allow(unused)]
# fn main() {
unsafe {}
# }
```
显示如下：
```text
warning: unnecessary `unsafe` block
 --> lint_example.rs:2:1
  |
2 | unsafe {}
  | ^^^^^^ unnecessary `unsafe` block
  |
  = note: `#[warn(unused_unsafe)]` on by default
```
**解释**  

如果块中没有内容需要 `unsafe`，应该移除 `unsafe` 标记，因为其不是必要并且可能会造成混乱。

## unused_variables
`unused_variables` lint 检测未以任何方式使用的变量。

**样例**
```rust

# #![allow(unused)]
# fn main() {
let x = 5;
# }
```
显示如下：
```text
warning: unused variable: `x`
 --> lint_example.rs:2:5
  |
2 | let x = 5;
  |     ^ help: if this is intentional, prefix it with an underscore: `_x`
  |
  = note: `#[warn(unused_variables)]` on by default
```
**解释**  

未使用变量可能意味着是个错误或未完成的代码。要对单个变量使该警告沉默，前缀加上下划线例如 `_x` 。

## warnings
`warnings` lint 允许你更改那些将会产生警告的 lint 的等级 为其他等级。

**样例**
```rust
#![deny(warnings)]
# fn main() {
fn foo() {}
}
```
显示如下：
```text
error: function is never used: `foo`
 --> lint_example.rs:3:4
  |
3 | fn foo() {}
  |    ^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(warnings)]
  |         ^^^^^^^^
  = note: `#[deny(dead_code)]` implied by `#[deny(warnings)]`
```
**解释**  

`warning` lint 有点特殊；通过改变其等级，可以改变所有其他会根据你想要的任何值生成警告的警告。这样说来，你永远不会在代码中直接触发此 lint 。

## where_clauses_object_safety
`where_clauses_object_safety` lint 检测 [where 子句](https://doc.rust-lang.org/reference/items/generics.html#where-clauses)的 [对象安全](https://doc.rust-lang.org/reference/items/traits.html#object-safety)。

**样例**
```rust
trait Trait {}

trait X { fn foo(&self) where Self: Trait; }

impl X for () { fn foo(&self) {} }

impl Trait for dyn X {}

// Segfault at opt-level 0, SIGILL otherwise.
pub fn main() { <dyn X as X>::foo(&()); }
```
显示如下：
```text
warning: the trait `X` cannot be made into an object
 --> lint_example.rs:3:14
  |
3 | trait X { fn foo(&self) where Self: Trait; }
  |              ^^^
  |
  = note: `#[warn(where_clauses_object_safety)]` on by default
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #51443 <https://github.com/rust-lang/rust/issues/51443>
note: for a trait to be "object safe" it needs to allow building a vtable to allow the call to be resolvable dynamically; for more information visit <https://doc.rust-lang.org/reference/items/traits.html#object-safety>
 --> lint_example.rs:3:14
  |
3 | trait X { fn foo(&self) where Self: Trait; }
  |       -      ^^^ ...because method `foo` references the `Self` type in its `where` clause
  |       |
  |       this trait cannot be made into an object...
  = help: consider moving `foo` to another trait
```

**解释**  

编译器以前会允许这些对象不安全的约束，这是不安全的。这是个[将来不兼容][future-incompatible] 的 lint ，将来会转化为固有错误。更多细节请参阅 [issue #51443](https://github.com/rust-lang/rust/issues/51443)。

## while_true
`while_true` lint 检测 `while true { }`。

**样例**
```rust

# #![allow(unused)]
# fn main() {
while true {

}
# }
```
显示如下：
```text
warning: denote infinite loops with `loop { ... }`
 --> lint_example.rs:2:1
  |
2 | while true {
  | ^^^^^^^^^^ help: use `loop`
  |
  = note: `#[warn(while_true)]` on by default
```

**解释**  
`while true` 应该被 `loop` 所代替。`loop` 表达式是编写无限循环的首选方法因为其直接表达了无限循环的意图。



[future-incompatible]:https://doc.rust-lang.org/rustc/lints/index.html#future-incompatible-lints
[identifier-pattern]:https://doc.rust-lang.org/reference/patterns.html#identifier-patterns
[non-ascii-idents]:https://doc.rust-lang.org/nightly/unstable-book/language-features/non-ascii-idents.html