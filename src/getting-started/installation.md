# 安装

## 安装 Rust 和 Cargo

获取 Cargo 最简单的方式，是通过 [rustup] 安装当前稳定版的 [Rust]。
使用 `rustup` 安装 Rust 时，也会一并安装 `cargo`。

在 Linux 和 macOS 系统上，执行如下命令：

```console
curl https://sh.rustup.rs -sSf | sh
```

它会下载一个脚本并启动安装。如果一切顺利，你会看到：

```console
Rust is installed now. Great!
```

在 Windows 上，下载并运行 [rustup-init.exe]。它会在控制台中启动安装，
成功后会显示与上面相同的提示信息。

完成后，你还可以使用 `rustup` 命令安装 Rust 和 Cargo 的 `beta` 或 `nightly`
通道版本。

更多安装方式和信息，请访问 Rust 官网的
[安装页面][install-rust]。

## 从源码构建并安装 Cargo

你也可以选择[从源码构建 Cargo][compiling-from-source]。

[rust]: https://www.rust-lang.org/
[rustup]: https://rustup.rs/
[rustup-init.exe]: https://win.rustup.rs/
[install-rust]: https://www.rust-lang.org/tools/install
[compiling-from-source]: https://github.com/rust-lang/cargo#compiling-from-source
