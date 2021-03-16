# JSON 输出

本章节介绍由 `rustc` 所发出的 JSON 的结构。可以通过 [`--error-format=json` 标签][option-error-format] 启用 JSON。可以使用  [`--json` 标签][option-json] 指定其他可以更改生成的消息以及消息的格式的选项。

JSON 消息每行都被发送到 stderr 。

如果使用 Rust 解析输出， [`cargo_metadata`](https://crates.io/crates/cargo_metadata) crate 提供了解析消息的一些支持。

当解析时，应注意与将来的格式更改保持向前兼容（译者注：向前兼容指的是以前的兼容以后的，而向后兼容就是以后的兼容以前的）。可选值可以为 `null` 。新的字段可能会添加。枚举字段如 “level” 或 “suggestion_applicability” 可以添加新的值。

## 诊断 （Diagnostics）

诊断消息会提供错误或在编译中可能产生的问题。 `rustc` 提供有关诊断来源的详细信息，以及提示和建议。

诊断以 parent/child 关系排列，其中 parent 诊断是诊断的核心，附加的 children 提供了其他的上下文，帮助和信息。

诊断有如下格式：

```javascript
{
    /* 主消息 */
    "message": "unused variable: `x`",
    /* 诊断码
       一些消息可能将这个值设置为 null 。
    */
    "code": {
        /* 标识了触发哪个诊断的独一的字符串。 */
        "code": "unused_variables",
        /* 可选字符串，解释关于诊断码的更多细节。*/
        "explanation": null
    },
    /* 诊断信息的重要程度。
       值可能是：
       - "error"： 无法编译的致命错误。
       - "warning": 可能的错误或预警。
       - "note"： 诊断的附加信息或上下文。
       - "help": 解决诊断问题的建议。
       - "failure-note": 为了获取更多信息的附带说明。
       - "error: internal compiler error": 指明编译器内部的错误。
    */
    "level": "warning",
    /* 一个源代码位置数组，指出有关诊断来源的详细信息。这个数组可能是空的，例如，
    对于一些全局消息，或者附加到 parent 的 child 消息，就可能是空的。

       字符偏移量是 Unicode 标量的偏移量。
    */
    "spans": [
        {
            /* span 所在的文件。
               注意，此路径可能不存在。例如，如果路径指向标准库，而
               rust src 在系统根目录中不可用， 那么它可能会指向一个不存在的文件。
               注意，这也可能指向外部 crate 源。
            */
            "file_name": "lib.rs",
            /* span 开始的字节偏移量（基于0且包含0）。 */
            "byte_start": 21,
            /* span 结束的字节偏移量（基于0且不包含0）。 */
            "byte_end": 22,
            /* span 的第一行编号（基于1且包含1）。 */
            "line_start": 2,
            /* span 的最后一行编号（基于1且包含1）。 */
            "line_end": 2,
            /* 行首（line_start）的第一个字符偏移量(基于1且包含1) */
            "column_start": 9,
            /* 行尾（line_end）的最后一个字符偏移量(基于1且不包含1) */
            "column_end": 10,
            /* 不管这是不是 "primary" span。

               这表明这个 span 是该诊断的焦点所在（focal）。

               在少数情况下，多个 span 可能被标记为 primary。 例如 "immutable borrow occurs here" 和
               "mutable borrow ends here" 可以是两个独立的 primary span。

               top (parent) 消息应该有至少一个 rimary span ，除非有 0 个 span 。
               child 消息有 0 个或多个 primary span 。
            */
            "is_primary": true,
            /* 一个对象数组，显示该 span 的原始源代码。这将展示 span 所在的整行文本 。跨多行的 span 对于每行会有一个单独对应的值。
            */
            "text": [
                {
                    /* 完整的原始源代码行。 */
                    "text": "    let x = 123;",
                    /* span 所指行的首字符偏移量(基于1且包含1)。 */
                    "highlight_start": 9,
                    /* span 所指行的最后一个字符偏移量(基于1且不包含1)。 */
                    "highlight_end": 10
                }
            ],
            /* 一个可选消息，用于显示 span 位置。对于主 span ，这通常为 null 。
            */
            "label": null,
            /* 一个可选字符串，对于解决该问题建议的替换。 工具（ Tools ）可能会尝试用此文本替换该 span 的内容 。
            */
            "suggested_replacement": null,
            /* 一个可选字符串，指出 "suggested_replacement" 的置信度。工具（  Tools ）可能会用这个值来决定是否自动应用建议（的替换）。

               可能的值可以是：
               - "MachineApplicable"： 该建议无疑是用户的意图。该建议应该自动被应用。
               - "MaybeIncorrect"： 该建议可能是用户的意图，但不确定。如果应用了该建议，则应该产生有效的 Rust 代码。
               - "HasPlaceholders"： 该建议包含占位符，如
                 `(...)`。 建议不会被自动应用因为它不会产生有效的 Rust 代码。用户需要填充该占位符。
               - "Unspecified": 建议的适用性尚不清楚。
            */
            "suggestion_applicability": null,
            /* 一个可选对象，指出该 span 内的宏展开。

               如果消息发生在宏调用中，该对象将会提供该消息在宏展开所在位置的详细信息。
            */
            "expansion": {
                /* 宏调用的 span 。
                   使用与 "spans" 数组相同的 span 定义。
                */
                "span": {/*...*/}
                /* 宏名称， 例如 "foo!" 或 "#[derive(Eq)]"。 */
                "macro_decl_name": "some_macro!",
                /* 可选的 span ，定义了宏的相关部分 */
                "def_site_span": {/*...*/},
            }
        }
    ],
    /* 附加诊断消息的数组。
       这是一个对象数组，使用与 parent 消息相同的格式。
       children 没有嵌套 （ children 自身不包含 "children" 的定义 ）
    */
    "children": [
        {
            "message": "`#[warn(unused_variables)]` on by default",
            "code": null,
            "level": "note",
            "spans": [],
            "children": [],
            "rendered": null
        },
        {
            "message": "if this is intentional, prefix it with an underscore",
            "code": null,
            "level": "help",
            "spans": [
                {
                    "file_name": "lib.rs",
                    "byte_start": 21,
                    "byte_end": 22,
                    "line_start": 2,
                    "line_end": 2,
                    "column_start": 9,
                    "column_end": 10,
                    "is_primary": true,
                    "text": [
                        {
                            "text": "    let x = 123;",
                            "highlight_start": 9,
                            "highlight_end": 10
                        }
                    ],
                    "label": null,
                    "suggested_replacement": "_x",
                    "suggestion_applicability": "MachineApplicable",
                    "expansion": null
                }
            ],
            "children": [],
            "rendered": null
        }
    ],
    /* 可选字符串，由 rustc 显示的诊断的 rendered 版本。注意这可能会受到 `--json` 标签的影响。
    */
    "rendered": "warning: unused variable: `x`\n --> lib.rs:2:9\n  |\n2 |     let x = 123;\n  |         ^ help: if this is intentional, prefix it with an underscore: `_x`\n  |\n  = note: `#[warn(unused_variables)]` on by default\n\n"
}
```

## 工件通知

当使用 [`--json=artifacts` 标签][option-json] 时就会触发工件通知（ Artifact notification ，译者注，artifact 真的是一个很难恰到其处翻译的词语，所以这里只能约定俗成地翻译为“工件”） 。 它们指明了文件工件已经被保存到了磁盘中。有关发出（工件）类型的更多信息可以在 [`--emit` 标签][option-emit] 文档中找到。

```javascript
{
    /* 生成工件的文件名 */
    "artifact": "libfoo.rlib",
    /* 生成的工件的种类，可能的值有：
       - "link"： 根据 crate 类型指定生成的 crate 。 
       - "dep-info"： 具有依赖信息的 `.d` 文件，所用语法类似于 Makefile 文件的语法。
       - "metadata"： 包含有关 crate 元数据的 Rust `.rmeta` 文件。
       - "save-analysis": 由 `-Zsave-analysis` feature 所发出的 JSON 文件 。
    */
    "emit": "link"
}
```

[option-emit]: command-line-arguments.md#option-emit
[option-error-format]: command-line-arguments.md#option-error-format
[option-json]: command-line-arguments.md#option-json
