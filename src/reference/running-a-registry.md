# 运行一个 Registry

一个最小可用的 registry 可以这样实现：
使用一个包含索引的 git 仓库，再配一个保存压缩 `.crate` 文件（由
[`cargo package`] 生成）的服务器。
用户将无法使用 Cargo 向其发布，但对封闭环境来说这通常已足够。
索引格式见 [Registry Index]。

一个支持发布的完整 registry 还需要额外提供一个 Web API 服务，
并且该 API 需符合 Cargo 使用的接口。
Web API 见 [Registry Web API]。

社区和商业项目中已有可用于构建和运行 registry 的方案。
可用列表见 <https://github.com/rust-lang/cargo/wiki/Third-party-registries>。

[Registry Web API]: registry-web-api.md
[Registry Index]: registry-index.md
[`cargo publish`]: ../commands/cargo-publish.md
[`cargo package`]: ../commands/cargo-package.md
