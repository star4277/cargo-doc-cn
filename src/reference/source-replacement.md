# 源替换

本文介绍如何将与 [registries] 或 [git-based dependencies] 仓库的通信重定向到其他数据源，
例如镜像原始 registry 的服务器，或精确的本地副本。

如果你想单独 patch 某个依赖，请见文档中的 [overriding dependencies] 章节。
如果你想控制 Cargo 的网络请求方式，请见 [`[http]`](config.md#http)
和 [`[net]`](config.md#net) 配置。

*source*（源）是一个提供器，包含可作为 package 依赖的 crate。
Cargo 支持将**一个源替换为另一个源**，用于表达如下策略：

* Vendoring --- 可定义代表本地文件系统 crate 的自定义源。
  这些源是其所替换源的子集，并且在需要时可直接提交到 package 中。

* Mirroring --- 可将源替换为等价镜像，作为 crates.io 的缓存。

Cargo 对源替换有一个核心假设：两个源提供的源码完全一致。
这也意味着替换源不允许包含原始源中不存在的 crate。

因此，源替换不适用于 patch 依赖或私有 registry 等场景。
Cargo 对 patch 依赖使用 [the `[patch]` key][overriding dependencies]；
私有 registry 支持见 [the Registries chapter][registries]。

使用源替换时，运行那些需要直接联系 registry 的命令[^1] 时，
必须传入 `--registry` 选项。
这样可以避免联系哪个 registry 的歧义，并使用对应 registry 的认证 token。

[^1]: 这类命令示例见 [Publishing Commands]。

[Publishing Commands]: ../commands/publishing-commands.md
[overriding dependencies]: overriding-dependencies.md
[registries]: registries.md

## 配置

替换源通过 [`.cargo/config.toml`][config] 配置，
可用键如下：

```toml
# `source` 表用于存放所有与 source-replacement 相关配置
[source]

# 在 `source` 表下，可定义多个来源表，其键为来源名。
# 例如这里定义一个新来源 `my-vendor-source`，
# 其路径是相对于该 `.cargo/config.toml` 所在目录的 `vendor`
[source.my-vendor-source]
directory = "vendor"

# crates.io 的默认源名称是 "crates-io"，
# 这里通过 `replace-with` 指示它被上面的来源替换。
#
# `replace-with` 也可以引用 `[registries]` 表中定义的备用 registry 名称。
[source.crates-io]
replace-with = "my-vendor-source"

# 每个 source 都有各自的表，键名为 source 名称
[source.the-source-name]

# 表示 `the-source-name` 将被定义在其他位置的 `another-source` 替换
replace-with = "another-source"

# 可指定多种 source（详见下文）
registry = "https://example.com/path/to/index"
local-registry = "path/to/registry"
directory = "path/to/vendor"

# Git source 还可选指定 branch/tag/rev
git = "https://example.com/path/to/repo"
# branch = "master"
# tag = "v1.0.1"
# rev = "313f44e8"
```

[config]: config.md

## 注册表源

“registry source” 的工作方式与 crates.io 类似。
它是一个索引，符合 https://doc.rust-lang.org/cargo/reference/registry-index.html 规范，
并带有一个配置文件，用于指示从何处下载 crate。

registry source 可使用 [git 或 sparse HTTP 协议][protocols]：

```toml
# Git 协议
registry = "ssh://git@example.com/path/to/index.git"

# Sparse HTTP 协议  
registry = "sparse+https://example.com/path/to/index"

# HTTPS git 协议
registry = "https://example.com/path/to/index"
```

[protocols]: registries.md#registry-protocols

[crates.io index]: registry-index.md

## 本地注册表源

“local registry source” 设计为另一个 registry source 的子集，
但可在本地文件系统使用（即 vendoring）。
local registry 一般会提前下载，通常与 `Cargo.lock` 同步，
由一组 `*.crate` 文件以及与普通 registry 相同格式的索引组成。

管理和创建 local registry source 的主要方式是使用
[`cargo-local-registry`][cargo-local-registry] 子命令。
该命令 [可在 crates.io 获取][cargo-local-registry]，可通过
`cargo install cargo-local-registry` 安装。

[cargo-local-registry]: https://crates.io/crates/cargo-local-registry

local registry 位于单个目录中，包含若干从 crates.io 下载的 `*.crate` 文件，
以及一个 `index` 目录。其格式与 crates.io-index 项目一致，
但只包含当前存在的 crate 条目。

## 目录源

“directory source” 与 local registry source 类似，
也是本地文件系统上的多个 crate，适合用于 vendor 依赖。
directory source 主要由 `cargo vendor` 子命令管理。

不过它和 local registry 的区别在于：
directory source 存放的是 `*.crate` 的解包内容，
在某些场景下更适合整体纳入版本控制。
一个 directory source 本质上是一个目录，
其中包含多个子目录，每个子目录是某个 crate 的源码
（即 `*.crate` 的解包结果）。
当前对子目录命名没有限制。

directory source 中每个 crate 还会带一个关联元数据文件
`.cargo-checksum.json`，用于防止意外修改。
它不是安全机制，不能防御恶意修改。

## Git 源

Git source 表示 [git-based dependencies] 使用的仓库。
它用于指定哪些 git 依赖应被替换为其他来源。

Git source 与 [git registry][protocols] 无关，
不能用于替换 registry source。

[git-based dependencies]: specifying-dependencies.md#specifying-dependencies-from-git-repositories
