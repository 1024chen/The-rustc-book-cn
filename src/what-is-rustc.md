# 什么是 rustc?

欢迎来到 "The rustc book"! `rustc` 是Rust程序语言的编译器, 它由项目本身提供。 编译器将获取您的源代码并以库或可执行文件的形式生成二进制代码。

大多数Rust程序员并不会直接使用 `rustc` 命令, 取而代之的是使用
[Cargo](../cargo/index.html) 。 不过所有的服务都通过 `rustc` 来完成！如果你想查看 Cargo 是如何使用 `rustc` 的, 你可以使用如下命令来查看：

```bash
$ cargo build --verbose
```

[//]: # (Section Separator)

它将打印出每个 `rustc` 的调用. 这本书可以帮忙你来理解这些选项的作用. 另外, 虽然大多数
 Rustaceans 使用 Cargo, 但并不是所有的 Rustaceans 都这样做: 有时候他们将 `rustc` 集成到其它构建系统中。 本书会为你提供执行这些操作的所有选项的指南。


[//]: # (Section Separator)


## 基本用法

假设你在 `hello.rs`文件中有一个小的 hello world 程序:

```rust
fn main() {
    println!("Hello, world!");
}
```

你可以使用 `rustc`命令将此源代码转换成可执行的文件：

```bash
$ rustc hello.rs
$ ./hello # 在linux上
$ .\hello.exe # 在windows上
```

请注意，我们只是向 `rustc` 传递了 crate 根目录， 而不是我们想要去编译第一个文件。例如我们有一个如下所示的 `main.rs` 文件：

```rust,ignore
mod foo;

fn main() {
    foo::hello();
}
```

创建 `foo.rs` 并添加如下内容:

```rust,ignore
pub fn hello() {
    println!("Hello, world!");
}
```

我们使用下面的命令去编译它:

```bash
$ rustc main.rs
```

[//]: # (Section Separator)

你不需要告诉 `rustc` 关于 `foo.rs`的信息;  `mod` 语句会给出需要关联的一切。这与你使用C编译器有所不同， c编译器中，你可以在每个文件上单独调用编译器，然后将他们都链接起来。换句话说， *crate* 是一个翻译单元，而不是特定的模块。