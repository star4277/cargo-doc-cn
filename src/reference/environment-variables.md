# 环境变量

Cargo 设置并读取您的代码可以检测到的许多环境变量
或覆盖。这是 Cargo 变量集的列表，按交互时间组织
与他们：

## 环境变量 Cargo 读取

您可以覆盖这些环境变量来更改 Cargo 在您的计算机上的行为
如下：

* `CARGO_LOG` --- Cargo 使用 [`tracing`] crate来显示调试日志消息。
可以设置 `CARGO_LOG` 环境变量以启用调试日志记录，
具有`trace`、`debug` 或`warn` 等值。
通常只在调试时使用。欲了解更多详细信息，请参阅
[Debug logging]。
* `CARGO_HOME` --- Cargo 维护注册表索引和的本地缓存
git checkout crate。默认情况下，它们存储在`$HOME/.cargo`下
（`%USERPROFILE%\.cargo` 在 Windows 上），但此变量会覆盖
该目录的位置。一旦crate被缓存，它就不会被删除
干净的命令。
有关更多详细信息，请参阅[guide](../guide/cargo-home.md)。
* `CARGO_TARGET_DIR` --- 放置所有生成工件的位置，
相对于当前工作目录。请参阅[`build.target-dir`]进行设置
通过配置。
* `CARGO` --- 如果设置，Cargo 将转发该值而不是设置它
当它构建crate时以及当它
执行构建脚本和外部子命令。这个值不是
由 Cargo 直接执行，并且应该始终指向一个命令
行为与 `cargo` 完全相同，因为这就是变量的用户
将会期待。
* `RUSTC` --- Cargo 将执行指定的命令，而不是运行 `rustc`
编译器代替。请参阅[`build.rustc`] 通过配置进行设置。
* `RUSTC_WRAPPER` --- Cargo 将执行此命令，而不是简单地运行 `rustc`
指定的包装器，将 rustc 作为其命令行参数传递
调用，第一个参数是实际 rustc 的路径。
对于设置构建缓存工具（例如`sccache`）很有用。看
[`build.rustc-wrapper`] 通过配置进行设置。将其设置为空字符串
覆盖配置并重置Cargo以不使用包装器。
* `RUSTC_WORKSPACE_WRAPPER` --- 对于工作区成员 Cargo 来说，不是简单地运行 `rustc`
执行这个指定的包装器，将 rustc 调用作为其命令行参数传递，
第一个参数是实际 rustc 的路径。构建单包项目时
如果没有工作区，该包将被视为工作区。它影响文件名哈希
以便包装器生成的工件被单独缓存。看
[`build.rustc-workspace-wrapper`] 通过配置进行设置。将其设置为空字符串会覆盖
配置并重置Cargo以不使用工作区成员的包装器。如果两者都`RUSTC_WRAPPER`
和`RUSTC_WORKSPACE_WRAPPER`被设置，然后它们将被嵌套：最终的调用是
`$RUSTC_WRAPPER $RUSTC_WORKSPACE_WRAPPER $RUSTC`。
* `RUSTDOC` --- Cargo 将执行指定的命令，而不是运行 `rustdoc`
相反，`rustdoc` 实例。请参阅[`build.rustdoc`] 通过配置进行设置。
* `RUSTDOCFLAGS` --- 以空格分隔的自定义标志列表，传递给所有`rustdoc`
Cargo 执行的调用。与 [`cargo rustdoc`] 相比，这是
对于将标志传递给*所有* `rustdoc` 实例很有用。看
[`build.rustdocflags`] 了解更多设置标志的方法。这个字符串是
按空格分割；为了对多个参数进行更稳健的编码，
参见`CARGO_ENCODED_RUSTDOCFLAGS`。
* `CARGO_ENCODED_RUSTDOCFLAGS` --- 由`0x1f`分隔的自定义标志列表
（ASCII 单位分隔符）传递给 Cargo 执行的所有 `rustdoc` 调用。
* `RUSTFLAGS` --- 要传递给所有编译器的以空格分隔的自定义标志列表
Cargo 执行的调用。与 [`cargo rustc`] 相比，这是
对于将标志传递给*所有*编译器实例很有用。看
[`build.rustflags`] 了解更多设置标志的方法。这个字符串是
按空格分割；为了对多个参数进行更稳健的编码，
参见`CARGO_ENCODED_RUSTFLAGS`。
* `CARGO_ENCODED_RUSTFLAGS` --- 由`0x1f`分隔的自定义标志列表
（ASCII 单位分隔符）传递给 Cargo 执行的所有编译器调用。
* `CARGO_INCREMENTAL` --- 如果设置为 1 那么 Cargo 将强制 [增量
为当前编译启用，当设置为 0 时
将强制禁用它。如果此环境变量不存在，则为 Cargo 的默认值
否则将被使用。另请参阅[`build.incremental`] 配置值。
* `CARGO_CACHE_RUSTC_INFO` --- 如果设置为 0 那么 Cargo 将不会尝试缓存
编译器版本信息。
* `HTTPS_PROXY` 或 `https_proxy` 或 `http_proxy` --- 要使用的 HTTP 代理，请参阅
[`http.proxy`] 了解更多详情。
* `HTTP_TIMEOUT` --- HTTP 超时（以秒为单位），请参阅[`http.timeout`] 了解更多信息
细节。
* `TERM` --- 如果设置为`dumb`，则会禁用进度条。
* `BROWSER` --- 要执行的 Web 浏览器，以使用 [`cargo 打开文档
doc`]'s `--open` 标志，请参阅 [`doc.browser`] 了解更多详细信息。
* `RUSTFMT` --- 而不是运行`rustfmt`，
[`cargo fmt`](https://github.com/rust-lang/rustfmt)将执行这个指定的
相反，`rustfmt` 实例。

### 配置环境变量

Cargo 读取环境变量中的某些配置值。
有关更多详细信息，请参阅[configuration chapter][config-env]。
总结来说，支持的环境变量有：

* `CARGO_ALIAS_<name>` --- 命令别名，参见[`alias`]。
* `CARGO_BUILD_JOBS` --- 并行作业数，参见[`build.jobs`]。
* `CARGO_BUILD_RUSTC` --- `rustc` 可执行文件，请参阅[`build.rustc`]。
* `CARGO_BUILD_RUSTC_WRAPPER` --- `rustc` 包装器，请参阅[`build.rustc-wrapper`]。
* `CARGO_BUILD_RUSTC_WORKSPACE_WRAPPER` --- 仅适用于工作区成员的`rustc` 包装器，请参阅[`build.rustc-workspace-wrapper`]。
* `CARGO_BUILD_RUSTDOC` --- `rustdoc` 可执行文件，请参阅[`build.rustdoc`]。
* `CARGO_BUILD_TARGET` --- 默认目标平台，参见[`build.target`]。
* `CARGO_BUILD_TARGET_DIR` --- 默认输出目录，参见[`build.target-dir`]。
* `CARGO_BUILD_BUILD_DIR` --- 默认构建目录，请参阅[`build.build-dir`]。
* `CARGO_BUILD_RUSTFLAGS` --- 额外的`rustc` 标志，请参阅[`build.rustflags`]。
* `CARGO_BUILD_RUSTDOCFLAGS` --- 额外的`rustdoc` 标志，请参阅[`build.rustdocflags`]。
* `CARGO_BUILD_INCREMENTAL` --- 增量编译，参见[`build.incremental`]。
* `CARGO_BUILD_DEP_INFO_BASEDIR` --- Dep-info 相对目录，参见[`build.dep-info-basedir`]。
* `CARGO_CACHE_AUTO_CLEAN_FREQUENCY` --- 配置自动缓存清理运行的频率，请参阅[`cache.auto-clean-frequency`]。
* `CARGO_CARGO_NEW_VCS` --- 默认源控制系统[`cargo new`]，参见[`cargo-new.vcs`]。
* `CARGO_FUTURE_INCOMPAT_REPORT_FREQUENCY` --- 我们应该多久生成一次不兼容报告通知，请参阅[`future-incompat-report.frequency`]。
* `CARGO_HTTP_DEBUG` --- 启用 HTTP 调试，请参阅[`http.debug`]。
* `CARGO_HTTP_PROXY` --- 启用 HTTP 代理，请参阅[`http.proxy`]。
* `CARGO_HTTP_TIMEOUT` --- HTTP 超时，参见[`http.timeout`]。
* `CARGO_HTTP_CAINFO` --- TLS 证书证书颁发机构文件，请参阅[`http.cainfo`]。
* `CARGO_HTTP_PROXY_CAINFO` --- 代理 TLS 证书证书颁发机构文件，请参阅[`http.proxy-cainfo`]。
* `CARGO_HTTP_CHECK_REVOKE` --- 禁用 TLS 证书吊销检查，请参阅[`http.check-revoke`]。
* `CARGO_HTTP_SSL_VERSION` --- 要使用的 TLS 版本，请参阅[`http.ssl-version`]。
* `CARGO_HTTP_LOW_SPEED_LIMIT` --- HTTP 低速限制，参见[`http.low-speed-limit`]。
* `CARGO_HTTP_MULTIPLEXING` --- 是否使用HTTP/2复用，参见[`http.multiplexing`]。
* `CARGO_HTTP_USER_AGENT` --- HTTP 用户代理标头，请参阅[`http.user-agent`]。
* `CARGO_INSTALL_ROOT` --- [`cargo install`] 的默认目录，请参阅[`install.root`]。
* `CARGO_NET_RETRY` --- 重试网络错误的次数，参见[`net.retry`]。
* `CARGO_NET_GIT_FETCH_WITH_CLI` --- 允许使用`git` 可执行文件来获取，请参阅[`net.git-fetch-with-cli`]。
* `CARGO_NET_OFFLINE` --- 离线模式，参见[`net.offline`]。
* `CARGO_PROFILE_<name>_BUILD_OVERRIDE_<key>` --- 覆盖构建脚本配置文件，请参阅[`profile.<name>.build-override`]。
* `CARGO_PROFILE_<name>_CODEGEN_UNITS` --- 设置代码生成单元，参见[`profile.<name>.codegen-units`]。
* `CARGO_PROFILE_<name>_DEBUG` --- 要包含什么样的调试信息，请参阅[`profile.<name>.debug`]。
* `CARGO_PROFILE_<name>_DEBUG_ASSERTIONS` --- 启用/禁用调试断言，请参阅[`profile.<name>.debug-assertions`]。
* `CARGO_PROFILE_<name>_INCREMENTAL` --- 启用/禁用增量编译，请参见[`profile.<name>.incremental`]。
* `CARGO_PROFILE_<name>_LTO` --- 链接时优化，请参阅[`profile.<name>.lto`]。
* `CARGO_PROFILE_<name>_OVERFLOW_CHECKS` --- 启用/禁用溢出检查，请参阅[`profile.<name>.overflow-checks`]。
* `CARGO_PROFILE_<name>_OPT_LEVEL` --- 设置优化级别，参见[`profile.<name>.opt-level`]。
* `CARGO_PROFILE_<name>_PANIC` --- 要使用的恐慌策略，请参阅[`profile.<name>.panic`]。
* `CARGO_PROFILE_<name>_RPATH` --- rpath 链接选项，请参阅[`profile.<name>.rpath`]。
* `CARGO_PROFILE_<name>_SPLIT_DEBUGINFO` --- 控制调试文件输出行为，请参阅[`profile.<name>.split-debuginfo`]。
* `CARGO_PROFILE_<name>_STRIP` --- 控制符号和/或调试信息的剥离，请参阅[`profile.<name>.strip`]。
* `CARGO_REGISTRIES_<name>_CREDENTIAL_PROVIDER` --- 注册表的凭证提供程序，请参阅[`registries.<name>.credential-provider`]。
* `CARGO_REGISTRIES_<name>_INDEX` --- 注册表索引的 URL，请参阅[`registries.<name>.index`]。
* `CARGO_REGISTRIES_<name>_TOKEN` --- 注册表的身份验证令牌，请参阅[`registries.<name>.token`]。
* `CARGO_REGISTRY_CREDENTIAL_PROVIDER` --- [crates.io] 的凭证提供者，请参阅[`registry.credential-provider`]。
* `CARGO_REGISTRY_DEFAULT` --- `--registry` 标志的默认注册表，请参阅[`registry.default`]。
* `CARGO_REGISTRY_GLOBAL_CREDENTIAL_PROVIDERS` --- 未定义特定提供程序的注册中心的凭证提供程序。请参阅[`registry.global-credential-providers`]。
* `CARGO_REGISTRY_TOKEN` --- [crates.io] 的身份验证令牌，请参阅[`registry.token`]。
* `CARGO_TARGET_<triple>_LINKER` --- 要使用的链接器，请参阅[`target.<triple>.linker`]。三元组必须是[converted to uppercase and underscores](config.md#environment-variables)。
* `CARGO_TARGET_<triple>_RUNNER` --- 可执行运行程序，请参阅[`target.<triple>.runner`]。
* `CARGO_TARGET_<triple>_RUSTFLAGS` --- 目标的额外`rustc` 标志，请参阅[`target.<triple>.rustflags`]。
* `CARGO_TERM_QUIET` --- 安静模式，参见[`term.quiet`]。
* `CARGO_TERM_VERBOSE` --- 默认终端详细程度，请参阅[`term.verbose`]。
* `CARGO_TERM_COLOR` --- 默认颜色模式，参见[`term.color`]。
* `CARGO_TERM_PROGRESS_WHEN` --- 默认进度条显示方式，参见[`term.progress.when`]。
* `CARGO_TERM_PROGRESS_WIDTH` --- 默认进度条宽度，参见[`term.progress.width`]。

[`cargo doc`]: ../commands/cargo-doc.md
[`cargo install`]: ../commands/cargo-install.md
[`cargo new`]: ../commands/cargo-new.md
[`cargo rustc`]: ../commands/cargo-rustc.md
[`cargo rustdoc`]: ../commands/cargo-rustdoc.md
[config-env]: config.md#environment-variables
[crates.io]: https://crates.io/
[incremental compilation]: profiles.md#incremental
[`alias`]: config.md#alias
[`build.jobs`]: config.md#buildjobs
[`build.rustc`]: config.md#buildrustc
[`build.rustc-wrapper`]: config.md#buildrustc-wrapper
[`build.rustc-workspace-wrapper`]: config.md#buildrustc-workspace-wrapper
[`build.rustdoc`]: config.md#buildrustdoc
[`build.target`]: config.md#buildtarget
[`build.target-dir`]: config.md#buildtarget-dir
[`build.build-dir`]: config.md#buildbuild-dir
[`build.rustflags`]: config.md#buildrustflags
[`build.rustdocflags`]: config.md#buildrustdocflags
[`build.incremental`]: config.md#buildincremental
[`build.dep-info-basedir`]: config.md#builddep-info-basedir
[`doc.browser`]: config.md#docbrowser
[`cache.auto-clean-frequency`]: config.md#cacheauto-clean-frequency
[`cargo-new.name`]: config.md#cargo-newname
[`cargo-new.email`]: config.md#cargo-newemail
[`cargo-new.vcs`]: config.md#cargo-newvcs
[`future-incompat-report.frequency`]: config.md#future-incompat-reportfrequency
[`http.debug`]: config.md#httpdebug
[`http.proxy`]: config.md#httpproxy
[`http.timeout`]: config.md#httptimeout
[`http.cainfo`]: config.md#httpcainfo
[`http.proxy-cainfo`]: config.md#httpproxy-cainfo
[`http.check-revoke`]: config.md#httpcheck-revoke
[`http.ssl-version`]: config.md#httpssl-version
[`http.low-speed-limit`]: config.md#httplow-speed-limit
[`http.multiplexing`]: config.md#httpmultiplexing
[`http.user-agent`]: config.md#httpuser-agent
[`install.root`]: config.md#installroot
[`net.retry`]: config.md#netretry
[`net.git-fetch-with-cli`]: config.md#netgit-fetch-with-cli
[`net.offline`]: config.md#netoffline
[`profile.<name>.build-override`]: config.md#profilenamebuild-override
[`profile.<name>.codegen-units`]: config.md#profilenamecodegen-units
[`profile.<name>.debug`]: config.md#profilenamedebug
[`profile.<name>.debug-assertions`]: config.md#profilenamedebug-assertions
[`profile.<name>.incremental`]: config.md#profilenameincremental
[`profile.<name>.lto`]: config.md#profilenamelto
[`profile.<name>.overflow-checks`]: config.md#profilenameoverflow-checks
[`profile.<name>.opt-level`]: config.md#profilenameopt-level
[`profile.<name>.panic`]: config.md#profilenamepanic
[`profile.<name>.rpath`]: config.md#profilenamerpath
[`profile.<name>.split-debuginfo`]: config.md#profilenamesplit-debuginfo
[`profile.<name>.strip`]: config.md#profilenamestrip
[`registries.<name>.credential-provider`]: config.md#registriesnamecredential-provider
[`registries.<name>.index`]: config.md#registriesnameindex
[`registries.<name>.token`]: config.md#registriesnametoken
[`registry.credential-provider`]: config.md#registrycredential-provider
[`registry.default`]: config.md#registrydefault
[`registry.global-credential-providers`]: config.md#registryglobal-credential-providers
[`registry.token`]: config.md#registrytoken
[`target.<triple>.linker`]: config.md#targettriplelinker
[`target.<triple>.runner`]: config.md#targettriplerunner
[`target.<triple>.rustflags`]: config.md#targettriplerustflags
[`term.quiet`]: config.md#termquiet
[`term.verbose`]: config.md#termverbose
[`term.color`]: config.md#termcolor
[`term.progress.when`]: config.md#termprogresswhen
[`term.progress.width`]: config.md#termprogresswidth

## Cargo 为 crate 设置的环境变量

Cargo 在编译时将这些环境变量公开给您的crate。
请注意，这适用于使用 `cargo run` 和 `cargo test` 运行二进制文件
以及。要获取 Rust 程序中任何这些变量的值，请执行以下操作
这：

```rust,ignore
let version = env!("CARGO_PKG_VERSION");
```

`version` 现在将包含`CARGO_PKG_VERSION` 的值。

请注意，如果manifest中未提供这些值之一，则
相应的环境变量设置为空字符串`""`。

* `CARGO` --- 执行构建的`cargo` 二进制文件的路径。
* `CARGO_MANIFEST_DIR` --- 包含包manifest的目录。
* `CARGO_MANIFEST_PATH` --- 包manifest的路径。
* `CARGO_PKG_VERSION` --- 软件包的完整版本。
* `CARGO_PKG_VERSION_MAJOR` --- 软件包的主要版本。
* `CARGO_PKG_VERSION_MINOR` --- 软件包的次要版本。
* `CARGO_PKG_VERSION_PATCH` --- 软件包的补丁版本。
* `CARGO_PKG_VERSION_PRE` --- 软件包的预发布版本。
* `CARGO_PKG_AUTHORS` --- 用冒号分隔包manifest中的作者列表。
* `CARGO_PKG_NAME` --- 你的包的名称。
* `CARGO_PKG_DESCRIPTION` --- 软件包manifest中的描述。
* `CARGO_PKG_HOMEPAGE` --- 包manifest中的主页。
* `CARGO_PKG_REPOSITORY` --- 包manifest中的存储库。
* `CARGO_PKG_LICENSE` --- 软件包manifest中的许可证。
* `CARGO_PKG_LICENSE_FILE` --- 软件包manifest中的许可证文件。
* `CARGO_PKG_RUST_VERSION` --- 包manifest中的 Rust 版本。
请注意，这是该包支持的最低 Rust 版本，而不是
当前的 Rust 版本。
* `CARGO_PKG_README` --- 包的 README 文件的路径。
* `CARGO_CRATE_NAME` --- 当前正在编译的包的名称。它是[Cargo target] 的名称，`-` 转换为`_`，例如库、二进制文件、示例、集成测试或基准的名称。
* `CARGO_BIN_NAME` --- 当前正在编译的二进制文件的名称。
仅设置[binaries] 或二进制[examples]。该名称不包含任何
文件扩展名，例如`.exe`。
* `OUT_DIR` --- 如果包有构建脚本，则将其设置为文件夹
构建脚本应放置其输出的位置。请参阅下文了解更多信息。
（仅在编译期间设置。）Cargo 不保证该目录
是空的，并且在构建之间不会被清理。
* `CARGO_BIN_EXE_<name>` --- 二进制目标可执行文件的绝对路径。
仅在构建 [integration test] 或基准测试时设置。这可能
与[`env` macro]一起使用来查找要运行以进行测试的可执行文件
目的。 `<name>` 是二进制目标的名称，与原样完全相同。为了
例如，`CARGO_BIN_EXE_my-program` 表示名为 `my-program` 的二进制文件。
构建测试时会自动构建二进制文件，除非二进制文件
具有未启用的必需功能。
* `CARGO_PRIMARY_PACKAGE` --- 如果以下情况将设置此环境变量
正在构建的包是主要的。主包是用户的包
在命令行上选择，使用 `-p` 标志或基于默认值
当前目录和默认工作区成员。
构建依赖项时不会设置该变量，
除非依赖项也是在命令行上选择的工作区成员。
仅在编译包时设置（而不是在运行二进制文件或测试时设置）。
* `CARGO_TARGET_TMPDIR` --- 仅在构建[integration test] 或基准代码时设置。
这是目标目录内目录的路径
集成测试或基准测试可以自由放置所需的任何数据
测试/工作台。 Cargo 最初创建此目录但没有
以任何方式管理其内容，这是测试代码的责任。

[Cargo target]: cargo-targets.md
[binaries]: cargo-targets.md#binaries
[examples]: cargo-targets.md#examples
[integration test]: cargo-targets.md#integration-tests
[`env` macro]: ../../std/macro.env.html

### 动态库路径

Cargo还在编译和运行二进制文件时设置动态库路径
使用 `cargo run` 和 `cargo test` 等命令。这有助于定位
共享库是构建过程的一部分。变量名称取决于
在平台上：

* Windows：`PATH`
* macOS：`DYLD_FALLBACK_LIBRARY_PATH`
* Unix：`LD_LIBRARY_PATH`
* AIX：`LIBPATH`

当 Cargo 启动时，该值将从现有值扩展。 macOS 有
如果 `DYLD_FALLBACK_LIBRARY_PATH` 尚未存在，请特别考虑
设置后，它将添加默认的`$HOME/lib:/usr/local/lib:/usr/lib`。

Cargo包括以下路径：

* 使用 [`rustc-link-search` 搜索任何构建脚本中包含的路径
指令](build-scripts.md#rustc-link-search)。外部路径
`target` 目录已删除。这是用户运行的责任
如果系统上有其他库，则需要正确设置 Cargo 环境
搜索路径中需要。
* 基本输出目录，例如 `target/debug` 和“deps”目录。
这主要是为了支持 proc-macros。
* rustc sysroot 库路径。这对大多数人来说通常并不重要
用户。

## Cargo 为构建脚本设置的环境变量

Cargo 在运行构建脚本时设置几个环境变量。因为这些变量
编译构建脚本时尚未设置，上面使用 `env!` 的示例将不起作用
相反，您需要在运行构建脚本时检索值：

```rust,ignore
use std::env;
let out_dir = env::var("OUT_DIR").unwrap();
```

`out_dir` 现在将包含`OUT_DIR` 的值。

* `CARGO` --- 执行构建的`cargo` 二进制文件的路径。
* `CARGO_MANIFEST_DIR` --- 包含包manifest的目录
正在构建（包含构建脚本的包）。另请注意，这是
构建脚本启动时当前工作目录的值。
* `CARGO_MANIFEST_PATH` --- 包manifest的路径。
* `CARGO_MANIFEST_LINKS` --- manifest`links` 值。
* `CARGO_MAKEFLAGS` --- 包含Cargo的[jobserver]所需的参数
并行化子进程的实现。 Rustc 或来自的Cargo调用
build.rs 已经可以读取 `CARGO_MAKEFLAGS`，但 GNU Make 需要这些标志
直接指定为参数，或通过 `MAKEFLAGS`
环境变量。目前 Cargo 没有设置 `MAKEFLAGS` 变量，
但调用 GNU Make 的构建脚本可以免费将其设置为内容
`CARGO_MAKEFLAGS`。
* `CARGO_FEATURE_<name>` --- 对于正在构建的包的每个激活的功能，
该环境变量将出现，其中 `<name>` 是
功能大写并将`-` 翻译为`_`。
* `CARGO_CFG_<cfg>` --- 对于每个[configuration option][configuration]
正在构建包时，此环境变量将包含
配置，其中 `<cfg>` 是大写的配置名称，
将`-` 翻译为`_`。如果存在，则存在布尔配置
设置，否则不存在。具有多个值的配置是
连接到单个变量，其值由 `,` 分隔。这包括
编译器内置的值（可以通过`rustc --print=cfg`查看）
以及由构建脚本设置的值和传递给 `rustc` 的额外标志（例如
`RUSTFLAGS` 中定义的那些）。这些变量的一些示例：
    * `CARGO_CFG_FEATURE` --- 正在构建的包的每个激活功能。
    * `CARGO_CFG_UNIX` --- 设置[unix-like platforms]。
    * `CARGO_CFG_WINDOWS` --- 设置[windows-like platforms]。
    * `CARGO_CFG_TARGET_FAMILY=unix,wasm` --- [target family]。
    * `CARGO_CFG_TARGET_OS=macos` --- [target operating system]。
    * `CARGO_CFG_TARGET_ARCH=x86_64` --- CPU [target architecture]。
    * `CARGO_CFG_TARGET_VENDOR=apple` --- [target vendor]。
    * `CARGO_CFG_TARGET_ENV=gnu` --- [target environment] ABI。
    * `CARGO_CFG_TARGET_ABI=eabihf` --- [target ABI]。
    * `CARGO_CFG_TARGET_POINTER_WIDTH=64` --- CPU [pointer width]。
    * `CARGO_CFG_TARGET_ENDIAN=little` --- CPU [target endianness]。
    * `CARGO_CFG_TARGET_FEATURE=mmx,sse` --- CPU [target features] 启用的列表。
  > 请注意，不同的 [target triples][Target Triple] 具有不同的 `cfg` 值集，
  > 因此，一个目标三元组中存在的变量可能在另一个目标三元组中不可用。
  >
  > 某些 cfg 值（如 `test`）不可用。
  >
  > **提示：** 对于要读取这些值的类型化 API，请考虑使用 [`build-rs`]
  > crate 而不是手动解析环境变量。另请注意
  > 应使用`CARGO_CFG_*`变量而不是`cfg!`宏或`#@@TOK0@@`
  > 构建脚本中的属性，这些检查*主机*平台，而不是*目标*。
* `OUT_DIR` --- 所有输出和中间工件应在其中的文件夹
被放置。该文件夹位于正在构建的包的构建目录中，
对于相关的包来说它是唯一的。Cargo不清理或重置此
构建之间的目录，其内容可能会在重建过程中保留。建造
脚本不应假设 `OUT_DIR` 为空，并且负责
管理或清理他们创建的任何文件。
* `TARGET` --- 正在编译的目标三元组。本机代码应该是
为此三重编译。有关详细信息，请参阅[Target Triple] 说明。
* `HOST` --- Rust 编译器的主机三元组。
* `NUM_JOBS` --- 指定为顶级并行度的并行度。这个可以
将 `-j` 参数传递给 `make` 等系统很有用。注意护理
解释此环境变量时应考虑。对于历史
目的仍然提供，但 Cargo 的最新版本，例如，
不需要运行 `make -j`，而是可以将 `MAKEFLAGS` 环境变量设置为
`CARGO_MAKEFLAGS` 的内容激活 Cargo 的 GNU Make 兼容功能
[jobserver] 用于子 make 调用。
* `DEBUG` --- `true`（如果将生成任何 [`debug`] 信息），否则`false`。
* `OPT_LEVEL` --- 当前正在构建的配置文件的相应 [`opt-level`] 变量的值。
* `PROFILE` --- `release` 用于发布版本，`debug` 用于其他版本。这是
根据[profile] 是否继承自[`dev`] 或
[`release`] 个人资料。不建议使用此环境变量。
使用其他环境变量如 `OPT_LEVEL` 提供更正确的
查看正在使用的实际设置。
* `DEP_<links>_<key>` --- 有关这组环境变量的更多信息，
请参阅有关 [`links`][links] 的构建脚本文档。
* `RUSTC`, `RUSTDOC` --- Cargo 的编译器和文档生成器
决定使用，传递给构建脚本，以便它也可以使用它。
* `RUSTC_WRAPPER` --- Cargo 正在使用的 `rustc` 包装器（如果有）。请参阅[`build.rustc-wrapper`]。
* `RUSTC_WORKSPACE_WRAPPER` --- Cargo 正在使用的 `rustc` 包装器（如果有）
对于工作区成员。请参阅[`build.rustc-workspace-wrapper`]。
* `RUSTC_LINKER` --- Cargo 已解析使用的链接器二进制文件的路径
对于当前目标（如果指定）。可以通过编辑来更改链接器
`.cargo/config.toml`；请参阅有关 [cargo configuration][cargo-config] 的文档
了解更多信息。
* `CARGO_ENCODED_RUSTFLAGS` --- extra flags that Cargo invokes `rustc` with,
由 `0x1f` 字符（ASCII 单位分隔符）分隔。看
[`build.rustflags`]。请注意，自 Rust 1.55 起，`RUSTFLAGS` 已从
环境；脚本应使用 `CARGO_ENCODED_RUSTFLAGS` 代替。
* `CARGO_PKG_<var>` --- 包信息变量，与[provided during crate building][variables set for crates] 具有相同的名称和值。

[`tracing`]: https://docs.rs/tracing
[debug logging]: https://doc.crates.io/contrib/implementation/debugging.html#logging
[unix-like platforms]: ../../reference/conditional-compilation.html#unix-and-windows
[windows-like platforms]: ../../reference/conditional-compilation.html#unix-and-windows
[target family]: ../../reference/conditional-compilation.html#target_family
[target operating system]: ../../reference/conditional-compilation.html#target_os
[target architecture]: ../../reference/conditional-compilation.html#target_arch
[target vendor]: ../../reference/conditional-compilation.html#target_vendor
[target environment]: ../../reference/conditional-compilation.html#target_env
[target ABI]: ../../reference/conditional-compilation.html#target_abi
[pointer width]: ../../reference/conditional-compilation.html#target_pointer_width
[target endianness]: ../../reference/conditional-compilation.html#target_endian
[target features]: ../../reference/conditional-compilation.html#target_feature
[links]: build-scripts.md#the-links-manifest-key
[configuration]: ../../reference/conditional-compilation.html
[jobserver]: https://www.gnu.org/software/make/manual/html_node/Job-Slots.html
[cargo-config]: config.md
[Target Triple]: ../appendix/glossary.md#target
[variables set for crates]: #environment-variables-cargo-sets-for-crates
[profile]: profiles.md
[`dev`]: profiles.md#dev
[`release`]: profiles.md#release
[`debug`]: profiles.md#debug
[`opt-level`]: profiles.md#opt-level
[`build-rs`]: https://crates.io/crates/build-rs

## Cargo 为 `cargo test` 设置的环境变量

Cargo 在运行测试时设置几个环境变量。
您可以在运行测试时检索值：

```rust,ignore
use std::env;
let out_dir = env::var("CARGO_BIN_EXE_foo").unwrap();
```

* `CARGO_BIN_EXE_<name>` --- 二进制目标可执行文件的绝对路径。
仅在运行 [integration test] 或基准测试时设置。
`<name>` 是二进制目标的名称，与原样完全相同。为了
例如，`CARGO_BIN_EXE_my-program` 表示名为 `my-program` 的二进制文件。
构建测试时会自动构建二进制文件，除非二进制文件
具有未启用的必需功能。

## 第三方子命令的环境变量 Cargo 设置

Cargo 将此环境变量公开给第 3 方子命令
（即名为 `cargo-foobar` 的程序放置在 `$PATH` 中）：

* `CARGO` --- 执行构建的`cargo` 二进制文件的路径。
* `CARGO_MAKEFLAGS` --- 包含Cargo的[jobserver]所需的参数
并行化子进程的实现。
仅当 Cargo 检测到作业服务器存在时才设置。

有关您的环境的更多信息，您可以运行`cargo metadata`。


