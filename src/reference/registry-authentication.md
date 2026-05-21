# Registry 认证
Cargo 通过凭据提供器（credential provider）对 registry 进行认证。
这些提供器可以是外部可执行程序，也可以是 Cargo 内置提供器，
用于存储和读取凭据。

使用带认证的备用 registry 时，*必须* 配置凭据提供器，
以避免在不知情的情况下把未加密凭据写入磁盘。
出于历史原因，公开（无需认证）registry 不强制要求配置凭据提供器；
如果未配置任何提供器，会使用 `cargo:token`。

Cargo 还内置了依赖操作系统安全存储 token 的平台提供器。
同时也包含 `cargo:token`，其会把凭据以未加密明文写入
[credentials](config.md#credentials) 文件。

## 推荐配置
推荐在 `$CARGO_HOME/config.toml` 中配置全局凭据提供器列表，默认路径为：
* Windows：`%USERPROFILE%\.cargo\config.toml`
* Unix：`~/.cargo/config.toml`

下面的推荐配置优先使用操作系统提供器，并回退到 `cargo:token`，
从 Cargo 的 [credentials](config.md#credentials) 文件或环境变量读取：
```toml
# ~/.cargo/config.toml
[registry]
global-credential-providers = ["cargo:token", "cargo:libsecret", "cargo:macos-keychain", "cargo:wincred"]
```
*注意：后面的条目优先级更高。
更多细节见 [`registry.global-credential-providers`](config.md#registryglobal-credential-providers)。*

某些私有 registry 还可能推荐 registry 专用 credential-provider。
请查看对应 registry 文档。

## 内置提供器
Cargo 内置了多个凭据提供器。
可用的内置提供器未来 Cargo 版本中可能调整（当前暂无此计划）。

### `cargo:token`
使用 Cargo 的 [credentials](config.md#credentials) 文件，以未加密明文存储 token。
读取 token 时，会检查 `CARGO_REGISTRIES_<NAME>_TOKEN` 环境变量。
如果未在提供器列表中包含该项，则 `*_TOKEN` 环境变量不会生效。

### `cargo:wincred`
使用 Windows Credential Manager 存储 token。

凭据会以 `cargo-registry:<index-url>` 的名称存放在 Credential Manager 的
“Windows Credentials”下。

### `cargo:macos-keychain`
使用 macOS Keychain 存储 token。

可通过 Keychain Access 应用查看已保存 token。

### `cargo:libsecret`
使用 [libsecret](https://wiki.gnome.org/Projects/Libsecret) 存储 token。

支持 libsecret 的密码管理器都可查看已保存 token。
以下是部分示例（非完整列表）：

- [GNOME Keyring](https://wiki.gnome.org/Projects/GnomeKeyring)
- [KDE Wallet Manager](https://apps.kde.org/kwalletmanager5/)（自 KDE Frameworks 5.97.0 起）
- [KeePassXC](https://keepassxc.org/)（自 2.5.0 起）

### `cargo:token-from-stdout <command> <args>`
启动一个子进程，从其 stdout 读取 token，并会去除换行。
* 子进程会继承用户的 stdin 和 stderr。
* 成功时应返回 0，失败时返回非 0。
* 不支持 [`cargo login`] 和 [`cargo logout`]；使用时会报错。

执行命令时会提供以下环境变量：

* `CARGO` --- 执行该命令的 `cargo` 二进制路径。
* `CARGO_REGISTRY_INDEX_URL` --- registry 索引 URL。
* `CARGO_REGISTRY_NAME_OPT` --- 可选的 registry 名称，不应作为查找键使用。

命令行参数会继续传给该子命令。

[`cargo login`]: ../commands/cargo-login.md
[`cargo logout`]: ../commands/cargo-logout.md

## 凭据插件
对于遵循 Cargo [凭据提供器协议](credential-provider-protocol.md) 的凭据插件，
配置值应是一个字符串：可执行文件路径（或在 `PATH` 中时直接用可执行名）。

例如，从 crates.io 安装 [cargo-credential-1password](https://crates.io/crates/cargo-credential-1password)
并配置如下：

先安装提供器：`cargo install cargo-credential-1password`

然后在配置里给 `registry.global-credential-providers` 添加（或创建）条目：
```toml
[registry]
global-credential-providers = ["cargo:token", "cargo-credential-1password --account my.1password.com"]
```

`global-credential-providers` 中的值会按空格分割为路径和命令行参数。
如果路径或参数本身包含空格，请使用
[`[credential-alias]` 表](config.md#credential-alias)进行定义。
