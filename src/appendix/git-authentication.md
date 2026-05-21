# Git 认证

当使用 git 依赖和 registry 时，Cargo 支持若干认证方式。
本附录介绍如何配置能与 Cargo 协同工作的 git 认证。

如果你需要其他认证方式，可设置 [`net.git-fetch-with-cli`]
配置项，让 Cargo 调用外部 `git` 可执行程序来拉取远程仓库，
而不是使用内置实现。
也可以通过环境变量 `CARGO_NET_GIT_FETCH_WITH_CLI=true` 启用。

> **注意：** 对于公开的 git 依赖，Cargo 不需要认证。
> 如果在这种场景看到认证失败，请先确认 URL 是否正确。

## HTTPS 认证

HTTPS 认证依赖 [`credential.helper`] 机制。
credential helper 有多种，你需要在全局 git 配置中指定使用哪一个。

```ini
# ~/.gitconfig

[credential]
helper = store
```

Cargo 不会主动提示输入密码，因此对于大多数 helper，
你需要在运行 Cargo 前先把初始用户名/密码交给 helper。
一种做法是先对私有仓库执行一次 `git clone` 并输入用户名/密码。

> **提示：**  
> macOS 用户可考虑 `osxkeychain` helper。  
> Windows 用户可考虑使用 [GCM] helper。

> **注意：** Windows 用户需要确保 `PATH` 中可找到 `sh` shell。
> 通常安装 Git for Windows 后会具备该能力。

## SSH 认证

SSH 认证要求 `ssh-agent` 正在运行，以便获取 SSH 密钥。
请确保相关环境变量已配置（多数类 Unix 系统为 `SSH_AUTH_SOCK`），
并用 `ssh-add` 添加了正确的密钥。

Windows 可以使用 Pageant（[PuTTY] 的一部分）或 `ssh-agent`。
若使用 `ssh-agent`，Cargo 需要使用 Windows 自带的 OpenSSH，
因为 Cargo 不支持 MinGW/Cygwin 使用的“模拟 Unix 域套接字”。
关于 Windows 安装可见 [Microsoft installation documentation]；
[key management] 页面包含启动 `ssh-agent` 与添加密钥的说明。

> **注意：** Cargo 不支持 git 的 SSH 简写 URL（如
> `git@example.com:user/repo.git`）。请使用完整 SSH URL，如
> `ssh://git@example.com/user/repo.git`。

> **注意：** Cargo 内置 SSH 库不会读取 SSH 配置文件（如 OpenSSH 的
> `~/.ssh/config`）。若你有更高级需求，请使用
> [`net.git-fetch-with-cli`]。

### SSH Known Hosts

连接 SSH 主机时，Cargo 需要通过“known hosts”（主机密钥列表）
验证主机身份。Cargo 会在标准位置查找 OpenSSH 风格的 `known_hosts` 文件
（家目录下的 `.ssh/known_hosts`，或类 Unix 平台下的
`/etc/ssh/ssh_known_hosts`，Windows 下的
`%PROGRAMDATA%\ssh\ssh_known_hosts`）。
这些文件的更多信息见 [sshd man page]。
此外，也可以在 Cargo 配置文件中通过 [`net.ssh.known-hosts`] 配置密钥。

如果在 known hosts 尚未配置前连接某 SSH 主机，
Cargo 会报错并提示如何添加主机密钥。
提示还会包含“fingerprint”（主机密钥的短哈希），更便于人工核对。
服务器管理员可对公钥执行 `ssh-keygen` 获取 fingerprint（例如
`ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub`）。
一些知名站点会在网上公布 fingerprint；
例如 GitHub 在
<https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints>
页面发布其 fingerprint 信息。

Cargo 内置了 [github.com](https://github.com) 的主机密钥。
如果这些密钥未来变更，你可以把新密钥加入配置或 known_hosts 文件。

> **注意：** Cargo 不支持 `known_hosts` 文件中的
> `@cert-authority` 或 `@revoked` 标记。
> 若需要这些能力，请使用 [`net.git-fetch-with-cli`]。
> 如果 Cargo 的 SSH 客户端行为不符合你的预期，这通常也是一个好方案。

[`credential.helper`]: https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage
[`net.git-fetch-with-cli`]: ../reference/config.md#netgit-fetch-with-cli
[`net.ssh.known-hosts`]: ../reference/config.md#netsshknown-hosts
[GCM]: https://github.com/microsoft/Git-Credential-Manager-Core/
[PuTTY]: https://www.chiark.greenend.org.uk/~sgtatham/putty/
[Microsoft installation documentation]: https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse
[key management]: https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement
[sshd man page]: https://man.openbsd.org/sshd#SSH_KNOWN_HOSTS_FILE_FORMAT

