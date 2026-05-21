# Cargo.toml 与 Cargo.lock

`Cargo.toml` 和 `Cargo.lock` 的职责不同。先给出结论：

* `Cargo.toml` 用于描述你对依赖的“范围性要求”，由你编写。
* `Cargo.lock` 记录依赖的“精确版本信息”，由 Cargo 维护，不应手动编辑。

如果不确定，通常应把 `Cargo.lock` 提交到版本控制系统（例如 Git）。
关于“为什么要把 Cargo.lock 纳入版本控制”以及可选方案，请看
[FAQ 中的相关条目](../faq.md#why-have-cargolock-in-version-control)。
我们还建议结合阅读
[验证最新依赖](continuous-integration.md#verifying-latest-dependencies)。

下面展开说明。

`Cargo.toml` 是一个 [**manifest**][def-manifest] 文件，你可以在其中声明
各种包元数据。例如，你可以声明依赖另一个包：

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```

这个包只有一个依赖：`regex`。这里指定依赖一个位于 GitHub 上的
Git 仓库。由于没有给出更多信息，Cargo 会假定你希望使用默认分支的
最新提交来构建。

这看起来不错，但有个问题：如果你今天构建一次并把代码发给我，
我明天再构建，期间 `regex` 可能新增提交。那我构建时会包含新提交，
而你不会。最终我们得到的构建结果不同，不利于可复现构建。

你可以在 `Cargo.toml` 里写死 `rev` 来解决：

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git", rev = "9f9f693" }
```

这样构建会一致，但代价很大：每次更新库都要手动处理 SHA-1，
既繁琐又容易出错。

这时 `Cargo.lock` 就派上用场了。因为有它，你不必手工跟踪精确提交，
Cargo 会代劳。当 manifest 如下：

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```

第一次构建时，Cargo 会取最新提交并把精确信息写入 `Cargo.lock`。
文件大致如下：

```toml
[[package]]
name = "hello_world"
version = "0.1.0"
dependencies = [
 "regex 1.5.0 (git+https://github.com/rust-lang/regex.git#9f9f693768c584971a4d53bc3c586c33ed3a6831)",
]

[[package]]
name = "regex"
version = "1.5.0"
source = "git+https://github.com/rust-lang/regex.git#9f9f693768c584971a4d53bc3c586c33ed3a6831"
```

你可以看到这里信息更完整，包括本次构建使用的精确提交。
因此当你把包交给他人时，即使 `Cargo.toml` 没写死提交，
对方也会使用完全相同的 SHA。

当你准备升级到库的新版本时，Cargo 可以帮你重新解析依赖并更新：

```console
$ cargo update         # 更新所有依赖
$ cargo update regex   # 只更新 “regex”
```

这会写出新的 `Cargo.lock`。注意 `cargo update` 的参数本质上是
[Package ID Specification](../reference/pkgid-spec.md)，`regex` 只是简写。

[def-manifest]:  ../appendix/glossary.md#manifest  '"manifest" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
