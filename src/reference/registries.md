# Registries

Cargo 通过“registry”来安装 crate 与获取依赖。
默认 registry 是 [crates.io]。
registry 包含一个“index”，其中有可检索的可用 crate 列表。
registry 还可以提供 Web API，以支持从 Cargo 直接发布新 crate。

> 注意：如果你想镜像或 vendor 一个现有 registry，
> 请参见 [Source Replacement]。

如果你正在实现 registry 服务器，请查看 [Running a Registry]，
了解 Cargo 与 registry 之间协议细节。

如果你使用的 registry 需要认证，请见 [Registry Authentication]。
如果你在实现凭据提供器，请见 [Credential Provider Protocol]。

## 使用备用 Registry

要使用 [crates.io] 之外的 registry，
需在 [`.cargo/config.toml` 文件][config]中添加该 registry 的名称与索引 URL。
`registries` 表会为每个 registry 设置一个键，例如：

```toml
[registries]
my-registry = { index = "https://my-intranet:8080/git/index" }
```

`index` 键应是 registry 索引的 git 仓库 URL，
或带 `sparse+` 前缀的 Cargo sparse registry URL。

之后即可在 `Cargo.toml` 的依赖条目中通过 `registry` 键指定 registry 名称，
从其他 registry 依赖 crate：

```toml
# 示例 Cargo.toml
[package]
name = "my-project"
version = "0.1.0"
edition = "2024"

[dependencies]
other-crate = { version = "1.0", registry = "my-registry" }
```

和大多数配置项一样，index 也可以通过环境变量指定而不是配置文件。
例如，设置如下环境变量与上述配置等价：

```ignore
CARGO_REGISTRIES_MY_REGISTRY_INDEX=https://my-intranet:8080/git/index
```

> 注意：[crates.io] 不接受依赖其他 registry crate 的 package。

## 发布到备用 Registry

如果该 registry 支持 Web API 访问，
则可以直接通过 Cargo 发布 package 到该 registry。
Cargo 的多个命令（如 [`cargo publish`]）支持 `--registry` 参数来指定使用哪个 registry。
例如，发布当前目录中的 package：

1. `cargo login --registry=my-registry`

    这个步骤只需执行一次。
    你需要输入从 registry 网站获取的私密 API token。
    也可以通过 `publish` 命令的 `--token` 参数直接传入 token，
    或设置包含 registry 名称的环境变量，例如
    `CARGO_REGISTRIES_MY_REGISTRY_TOKEN`。

2. `cargo publish --registry=my-registry`

如果不想每次都传 `--registry`，可在 [`.cargo/config.toml`][config]
通过 `registry.default` 设置默认 registry，例如：

```toml
[registry]
default = "my-registry"
```

在 `Cargo.toml` 中设置 `package.publish` 可限制该 package 允许发布到哪些 registry。
这可防止误把闭源 package 发布到 [crates.io]。
其值可以是 registry 名称列表，例如：

```toml
[package]
# ...
publish = ["my-registry"]
```

`publish` 也可以设为 `false`，表示禁止所有发布（等同空列表）。

[`cargo login`] 保存的认证信息位于 Cargo home 目录（默认 `$HOME/.cargo`）中的
`credentials.toml` 文件。
其中每个 registry 都有独立表项，例如：

```toml
[registries.my-registry]
token = "854DvwSlUwEHtIo3kWy6x7UCPKHfzCmy"
```

## Registry 协议
Cargo 支持两种远程 registry 协议：`git` 与 `sparse`。
如果 registry index URL 以 `sparse+` 开头，Cargo 使用 sparse 协议；
否则使用 `git` 协议。

`git` 协议将索引元数据存放在 git 仓库中，并要求 Cargo 克隆整个仓库。

`sparse` 协议通过普通 HTTP 请求拉取单个元数据文件。
由于 Cargo 只下载相关 crate 的元数据，`sparse` 协议可显著节省时间和带宽。

[crates.io] 同时支持两种协议。
crates.io 使用哪种协议由 [`registries.crates-io.protocol`] 配置键控制。

[Source Replacement]: source-replacement.md
[Running a Registry]: running-a-registry.md
[Credential Provider Protocol]: credential-provider-protocol.md
[Registry Authentication]: registry-authentication.md
[`cargo publish`]: ../commands/cargo-publish.md
[`cargo package`]: ../commands/cargo-package.md
[`cargo login`]: ../commands/cargo-login.md
[config]: config.md
[crates.io]: https://crates.io/
[`registries.crates-io.protocol`]: config.md#registriescrates-ioprotocol
