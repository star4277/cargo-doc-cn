# 在现有 Cargo 包上工作

如果你下载了一个使用 Cargo 的现有 [package][def-package]，上手会非常容易。

首先，从某处获取这个包。这个例子里，我们使用从 GitHub 仓库克隆的 `regex`：

```console
$ git clone https://github.com/rust-lang/regex.git
$ cd regex
```

构建时，执行 `cargo build`：

```console
$ cargo build
   Compiling regex v1.5.0 (file:///path/to/package/regex)
```

Cargo 会先拉取所有依赖，然后编译依赖和该 package 本身。

[def-package]:  ../appendix/glossary.md#package  '"package" (glossary entry)'
