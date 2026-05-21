# Cargo 入门第一步

本节会快速介绍 `cargo` 命令行工具。我们将演示它如何为我们创建一个新的
[***package***][def-package]，如何编译该 package 中的 [***crate***][def-crate]，
以及如何运行最终生成的程序。

要用 Cargo 创建一个新 package，使用 `cargo new`：

```console
$ cargo new hello_world
```

Cargo 默认使用 `--bin` 来创建二进制程序。如果要创建库，则传入 `--lib`。

下面看看 Cargo 为我们生成了什么：

```console
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

这就是开始所需的全部内容。先看一下 `Cargo.toml`：

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2024"

[dependencies]
```

这被称为 [***manifest***][def-manifest]，其中包含 Cargo 编译你的 package
所需的全部元数据。

`src/main.rs` 的内容如下：

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 为我们生成了一个“hello world”程序，也就是一个
[***binary crate***][def-crate]。现在来编译它：

```console
$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

然后运行它：

```console
$ ./target/debug/hello_world
Hello, world!
```

我们也可以使用 `cargo run`，一步完成编译并运行：

```console
$ cargo run
     Fresh hello_world v0.1.0 (file:///path/to/package/hello_world)
   Running `target/hello_world`
Hello, world!
```

## 继续深入

如需了解更多 Cargo 的使用细节，请查看 [Cargo 指南](../guide/index.md)

[def-crate]:     ../appendix/glossary.md#crate     '"crate"（术语表条目）'
[def-manifest]:  ../appendix/glossary.md#manifest  '"manifest"（术语表条目）'
[def-package]:   ../appendix/glossary.md#package   '"package"（术语表条目）'
