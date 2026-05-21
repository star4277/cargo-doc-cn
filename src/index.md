# Cargo 手册

![Cargo Logo](images/Cargo-Logo-Small.png)

Cargo 是 Rust 的[*包管理器*][def-package-manager]。
Cargo 会下载你的 Rust [package][def-package] 所依赖的内容，
编译你的包，生成可分发包，并把它们上传到 [crates.io]
（Rust 社区的[*包注册表*][def-package-registry]）。
你可以在 [GitHub] 上为本书贡献内容。

## 章节

**[快速开始](getting-started/index.md)**

开始使用 Cargo：安装 Cargo（和 Rust），并创建你的第一个[*crate*][def-crate]。

**[Cargo 指南](guide/index.md)**

本指南会告诉你使用 Cargo 开发 Rust 包所需的核心知识。

**[Cargo 参考](reference/index.md)**

参考章节覆盖 Cargo 各个领域的细节。

**[Cargo 命令](commands/index.md)**

命令章节介绍如何通过命令行接口与 Cargo 交互。

**[常见问题](faq.md)**

**附录：**
* [术语表](appendix/glossary.md)
* [Git 认证](appendix/git-authentication.md)

**其他文档：**
* [更新日志](CHANGELOG.md)
  --- 记录 Cargo 每个版本的详细变更说明。
* [Rust 文档站点](https://doc.rust-lang.org/)
  --- Rust 官方文档与工具链接。

[def-crate]:            ./appendix/glossary.md#crate            '"crate" (glossary entry)'
[def-package]:          ./appendix/glossary.md#package          '"package" (glossary entry)'
[def-package-manager]:  ./appendix/glossary.md#package-manager  '"package manager" (glossary entry)'
[def-package-registry]: ./appendix/glossary.md#package-registry '"package registry" (glossary entry)'
[rust]: https://www.rust-lang.org/
[crates.io]: https://crates.io/
[GitHub]: https://github.com/rust-lang/cargo/tree/master/src/doc
