# 为什么会有 Cargo

## 预备知识

在 Rust 中，库或可执行程序被称为 [*crate*][def-crate]。crate 通过
Rust 编译器 `rustc` 进行编译。很多人接触 Rust 的第一段代码是经典的
“hello world”，并直接调用 `rustc` 编译：

```console
$ rustc hello.rs
$ ./hello
Hello, world!
```

注意，上面的命令需要你显式指定文件名。如果你改成编译另一个程序，
命令行调用就会不同；如果还要指定编译器选项或引入外部依赖，命令会
更加复杂。

此外，大多数非平凡程序都依赖外部库，并且还会传递依赖于这些库自己的
依赖。若手动获取并维护所有依赖的正确版本，既困难又容易出错。

与其只围绕 crate 和 `rustc` 工作，不如引入更高层的
[“*package*”][def-package] 抽象，并使用
[*包管理器*][def-package-manager]，从而避免上述问题。

## 引入 Cargo

*Cargo* 是 Rust 的包管理器。它让 Rust [*package*][def-package]
能够声明依赖，并确保你始终获得可重复的构建结果。

为实现这一目标，Cargo 做了四件事：

* 引入两个元数据文件，用于记录包信息。
* 获取并构建你的包依赖。
* 使用正确参数调用 `rustc` 或其他构建工具来构建你的包。
* 引入约定，使 Rust 包的开发更容易。

在很大程度上，Cargo 统一了构建程序或库所需的命令；这正是这些约定的价值之一。
正如后文所示，不管构建产物（[*artifact*][def-artifact]）叫什么名字，
都可以通过同样的命令来构建。你不必直接调用 `rustc`，而是执行如
`cargo build` 这样的通用命令，由 Cargo 负责拼装正确的 `rustc` 调用。
此外，Cargo 还会自动从 [*registry*][def-registry] 获取你声明的依赖，
并按需纳入构建过程。

可以说，一旦你学会如何构建一个基于 Cargo 的项目，也就基本会构建所有项目了。

[def-artifact]:         ../appendix/glossary.md#artifact         '"artifact" (glossary entry)'
[def-crate]:            ../appendix/glossary.md#crate            '"crate" (glossary entry)'
[def-package]:          ../appendix/glossary.md#package          '"package" (glossary entry)'
[def-package-manager]:  ../appendix/glossary.md#package-manager  '"package manager" (glossary entry)'
[def-registry]:         ../appendix/glossary.md#registry         '"registry" (glossary entry)'
