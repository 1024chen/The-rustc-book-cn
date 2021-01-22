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

`ambiguous_associated_items` lint 检测枚举变量和关联项之间的模糊不清。



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





## arithmetic_overflow





## conflicting_repr_hints



## const_err



## ill_formed_attribute_input



## incomplete_include



## invalid_type_param_default



## macro_expanded_macro_exports_accessed_by_absolute_paths



## missing_fragment_specifier



## mutable_transmutes



## no_mangle_const_items



## order_dependent_trait_objects



## overflowing_literals



## patterns_in_fns_without_body



## pub_use_of_private_extern_crate



## soft_unstable



## unconditional_panic



## unknown_crate_types



## useless_deprecated

