# Cargo Home

“Cargo home” 充当下载与源码缓存目录。
在构建 [crate][def-crate] 时，Cargo 会把下载的构建依赖存放到 Cargo home。
你可以通过设置 `CARGO_HOME` [环境变量][env] 来修改其位置。
如果你需要在 Rust crate 内部获取这个路径，
可以使用 [home](https://crates.io/crates/home) crate 提供的 API。
默认情况下，Cargo home 位于 `$HOME/.cargo/`。

请注意：Cargo home 的内部结构不是稳定接口，可能随时变更。

Cargo home 主要由以下部分组成：

## 文件

* `config.toml`
  Cargo 的全局配置文件，见参考文档中的 [config][config]。

* `credentials.toml`
  由 [`cargo login`] 写入的私有登录凭据，用于登录 [registry][def-registry]。

* `.crates.toml`, `.crates2.json`
  这两个隐藏文件保存通过 [`cargo install`] 安装的 crate
  [package][def-package] 信息。不要手动编辑。

## 目录

* `bin`
  `bin` 目录包含通过 [`cargo install`] 或
  [`rustup`](https://rust-lang.github.io/rustup/) 安装的可执行文件。
  要让这些二进制可直接调用，请将该目录加入 `$PATH` 环境变量。

* `git`
  这里存放 Git 源：

  * `git/db`
    当某个 crate 依赖 Git 仓库时，Cargo 会把该仓库以 bare repo 形式克隆到这里，
    并在需要时更新。

  * `git/checkouts`
    使用 Git 依赖时，Cargo 会从 `git/db` 中的 bare repo 检出依赖所需提交到这里。
    这会为编译器提供对应提交的实际文件内容。
    同一个仓库的不同提交可以同时存在多个 checkout。

* `registry`
  crate 注册表（例如 [crates.io](https://crates.io/)）的包与元数据位于这里。

  * `registry/index`
    索引是一个 bare git 仓库，包含该注册表所有 crate 的元数据
    （版本、依赖等）。

  * `registry/cache`
    下载的依赖会缓存到这里。它们是以 `.crate` 为扩展名的 gzip 压缩包。

  * `registry/src`
    当某个下载的 `.crate` 被需要时，会解压到 `registry/src`，
    供 rustc 读取其中的 `.rs` 文件。

## 在 CI 中缓存 Cargo home

为了避免在持续集成中每次都重新下载所有依赖，你可以缓存 `$CARGO_HOME`。
但缓存整个目录通常效率不高，因为下载的源码会出现“双份”。
例如依赖 `serde 1.0.92` 时，若整体缓存 `$CARGO_HOME`，
会同时缓存 `registry/cache` 中的 `serde-1.0.92.crate`，
以及 `registry/src` 中解压后的 serde `.rs` 文件。
这会不必要地拖慢构建，因为下载、解压、重新压缩和回传缓存都需要时间。

如果你希望缓存通过 [`cargo install`] 安装的二进制，
需要缓存 `bin/` 目录以及 `.crates.toml`、`.crates2.json` 两个文件。

通常跨构建缓存以下内容就够了：

* `.crates.toml`
* `.crates2.json`
* `bin/`
* `registry/index/`
* `registry/cache/`
* `git/db/`

## Vendor 化项目的全部依赖

参见 [`cargo vendor`] 子命令。

## 清理缓存

理论上，你可以删除缓存中的任意部分，Cargo 会尽力恢复：
需要时它会重新解压归档、重新检出 bare repo，或直接从网络重新下载。

另外，[cargo-cache](https://crates.io/crates/cargo-cache) crate 提供了简单的 CLI，
可按需清理缓存的指定部分，或查看各组成部分大小。

[`cargo install`]: ../commands/cargo-install.md
[`cargo login`]: ../commands/cargo-login.md
[`cargo vendor`]: ../commands/cargo-vendor.md
[config]: ../reference/config.md
[def-crate]:     ../appendix/glossary.md#crate     '"crate" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
[def-registry]:  ../appendix/glossary.md#registry  '"registry" (glossary entry)'
[env]: ../reference/environment-variables.md
