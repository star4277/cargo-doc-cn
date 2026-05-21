# 创建新 Package

要使用 Cargo 创建一个新的 [package][def-package]，执行 `cargo new`：

```console
$ cargo new hello_world --bin
```

这里传入 `--bin` 是因为我们要创建一个二进制程序；如果要创建库，
则应传入 `--lib`。默认情况下，这也会初始化一个新的 `git` 仓库。
如果你不希望这样做，可以加上 `--vcs none`。

看看 Cargo 为我们生成了什么：

```console
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

再仔细看一下 `Cargo.toml`：

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2024"

[dependencies]

```

这被称为 [***manifest***][def-manifest]，其中包含 Cargo 编译你的包所需的
全部元数据。这个文件使用 [TOML] 格式编写。

`src/main.rs` 的内容如下：

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 为你生成了一个“hello world”程序，也就是一个
[*binary crate*][def-crate]。来编译它：

```console
$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

然后运行：

```console
$ ./target/debug/hello_world
Hello, world!
```

你也可以使用 `cargo run` 一步完成编译并运行（如果距离上次编译没有改动，
你不会看到 `Compiling` 这一行）：

```console
$ cargo run
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
     Running `target/debug/hello_world`
Hello, world!
```

此时你会注意到出现了一个新文件 `Cargo.lock`。它记录了依赖信息。
由于当前还没有依赖，因此内容并不多。

当你准备发布时，可以用 `cargo build --release` 以开启优化的方式编译：

```console
$ cargo build --release
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

`cargo build --release` 会把生成的二进制文件放在 `target/release`，
而不是 `target/debug`。

开发阶段默认使用 debug 模式。它编译更快（因为编译器不做优化），
但运行速度更慢。release 模式编译更久，但运行更快。

[TOML]: https://toml.io/
[def-crate]:     ../appendix/glossary.md#crate     '"crate" (glossary entry)'
[def-manifest]:  ../appendix/glossary.md#manifest  '"manifest" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
