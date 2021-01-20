# Lint 级别

在 `rustc` 里, lints 被分为4个 *levels* （级别）:

1. allow (允许)
2. warn（警告）
3. deny（拒绝）
4. forbid（禁止）

每一个 lint 都有一个默认级别 (会在本章后面lint列表中解释), 并且编译器有一个默认的警告级别. 首先, 让我们解释一下这些级别的含义, 然后再去讨论关于它们的配置.

[//]: # (Section Separator)

## allow

这个 lints 存在, 但默认情况下不执行任何操作. 例如下面这个源码:

```rust
pub fn foo() {}
```

编译这个文件不会产生任何警告:

```bash
$ rustc lib.rs --crate-type=lib
$
```

但是此代码违反了 `missing_docs` lint.

这些 lints 主要是通过配置手动去开启的, 我们将在本章节后面再讨论.

[//]: # (Section Separator)

## warn

如果你违反了 warn 这种 lint 的级别，将会在编译时产生一些警告. 例如下面的代码与 `unused_variables` lint 发生冲突:

```rust
pub fn foo() {
    let x = 5;
}
```

这将会产生如下所示的警告:

```bash
$ rustc lib.rs --crate-type=lib
warning: unused variable: `x`
 --> lib.rs:2:9
  |
2 |     let x = 5;
  |         ^
  |
  = note: `#[warn(unused_variables)]` on by default
  = note: to avoid this warning, consider using `_x` instead
```

[//]: # (Section Separator)

## deny

一个 'deny' lint 将会在你违反其规则时产生一个错误。例如下面的代码与 `exceeding_bitshifts` lint 相冲突。

```rust,ignore
fn main() {
    100u8 << 10;
}
```

```bash
$ rustc main.rs
error: bitshift exceeds the type's number of bits
 --> main.rs:2:13
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^
  |
  = note: `#[deny(exceeding_bitshifts)]` on by default
```

一个 lint 发出的错误与一个常规错误两者之间有什么不同呢?
Lints 可以通过不同的级别(levels)去配置, 因此与 'allow' lints 类似,默认情况下 '
[//]: # (Section Separator)
deny' 的警告是被允许的。类似的, 你可能希望设置一个默认为 `warn` 等级的 lint 来发出一个错误，此 lint 级别正可以这样让你设置。

[//]: # (Section Separator)

## forbid

'forbid' 是一种特殊的 lint 级别,它比 'deny' 等级更高。 它与 'deny' 一样会发出一个错误，但是与 'deny' 级别不同的是, 'forbid' 级别不能被比错误更低的情况覆盖了。然而, lint 仍然被 `--cap-lints` (参阅下文) 限制, 因此 `rustc --cap-lints warn` 命令将 'forbid' 级别的lints设置为仅有警告信息。

[//]: # (Section Separator)

## 配置 warning 级别

记得我们上面默认级别为 'allow' 的 lint `missing_docs` 的例子的吗?

```bash
$ cat lib.rs
pub fn foo() {}
$ rustc lib.rs --crate-type=lib
$
```

这样可以配置此 lint 在更高级别上运行,两者都使用编译器标志以及源代码中的属性。

你还可以 "限制" lints 以便编译器可以选择忽略某些 lint 级别. 我们将在最后讨论(这种情况)。

[//]: # (Section Separator)

### 通过编译器标志

比如 `-A`, `-W`, `-D`, 和 `-F` 标志可以让你设置一个或多个 lint 为 "allow", "waring", "deny", 或者 "forbid" 级别, 像下面这样:

```bash
$ rustc lib.rs --crate-type=lib -W missing-docs
warning: missing documentation for crate
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^
  |
  = note: requested on the command line with `-W missing-docs`

warning: missing documentation for a function
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^
```

```bash
$ rustc lib.rs --crate-type=lib -D missing-docs
error: missing documentation for crate
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^
  |
  = note: requested on the command line with `-D missing-docs`

error: missing documentation for a function
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^

error: aborting due to 2 previous errors
```

你也可以多次传递每个标签，来修改多个 lints 的级别:

```bash
$ rustc lib.rs --crate-type=lib -D missing-docs -D unused-variables
```

当然, 你也可以将这4种标签混合在一起用:

```bash
$ rustc lib.rs --crate-type=lib -D missing-docs -A unused-variables
```

这些命令行参数的顺序也考虑在内了。以下内容允许 `unused-variables` lint， 是因为它是该 lint 的最后一个参数(译者注: 最后一个参数的设置会覆盖之前的):

```bash
$ rustc lib.rs --crate-type=lib -D unused-variables -A unused-variables
```

您可以通过此举在一组 lints 中覆盖某个特定的 lint 的级别来。以下例子将在 `unused` 组中所有 lint 设置为 拒绝（ denies ）级别， 但是将此组中的 `unused-variables` lint 设置为允许( allows )。
(无论顺序如何，"forbid" 仍然胜过一切):

```bash
$ rustc lib.rs --crate-type=lib -D unused -A unused-variables
```

### 通过属性

你也可以通过设置一个 `crate-wide` 属性来修改 lint 的级别:

```bash
$ cat lib.rs
#![warn(missing_docs)]

pub fn foo() {}
$ rustc lib.rs --crate-type=lib
warning: missing documentation for crate
 --> lib.rs:1:1
  |
1 | / #![warn(missing_docs)]
2 | |
3 | | pub fn foo() {}
  | |_______________^
  |
note: lint level defined here
 --> lib.rs:1:9
  |
1 | #![warn(missing_docs)]
  |         ^^^^^^^^^^^^

warning: missing documentation for a function
 --> lib.rs:3:1
  |
3 | pub fn foo() {}
  | ^^^^^^^^^^^^
```

`warn`, `allow`, `deny`, `forbid` 这四种等级都可以使用这种方式修改。

你还可以为每个属性传递多个 lint ：

```rust
#![warn(missing_docs, unused_variables)]

pub fn foo() {}
```

还有一起使用多个属性：

```rust
#![warn(missing_docs)]
#![deny(unused_variables)]

pub fn foo() {}
```

### Capping lints

`rustc` 支持一个标签, `--cap-lints LEVEL` 用来设置 "lint 级别上限"。即此设置所有的 lint 的最高级别。如下所示， 如果我们采用上面 "deny" lint 级别代码示例:

```rust,ignore
fn main() {
    100u8 << 10;
}
```

然后我们编译它, 限制 lints 为 warn 级别:

```bash
$ rustc lib.rs --cap-lints warn
warning: bitshift exceeds the type's number of bits
 --> lib.rs:2:5
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^
  |
  = note: `#[warn(exceeding_bitshifts)]` on by default

warning: this expression will panic at run-time
 --> lib.rs:2:5
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^ attempt to shift left with overflow
```

现在仅仅只是出现了警告，而不是错误。允许所有的 lint 的话我可以走的更远(译者注:意思是我们可以通过编译继续下去)。

```bash
$ rustc lib.rs --cap-lints allow
$
```

在Cargo中这个特性被使用的很多; 它会在我们编译依赖时设置 `--cap-lints allow` 命令, 以便在有任何警告信息的时候, 不会影响到我们的构建输出。
