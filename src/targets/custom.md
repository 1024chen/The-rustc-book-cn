# 自定义 Targets

如果你想要构建 `rustc` 尚不支持的目标， 你可以用 “自定义目标规范” （"custom target specification"） 来定义目标。这些目标规范文件是 JSON 格式的。要查看主机目标的 JSON，可以运行：

```bash
$ rustc +nightly -Z unstable-options --print target-spec-json
```

要查看其他目标的 JSON，添加 `--target` 标签：

```bash
$ rustc +nightly -Z unstable-options --target=wasm32-unknown-unknown --print target-spec-json
```

要使用自定义目标，查看 `cargo`的 (尚不稳定)  [`build-std` feature](https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#build-std) 。
