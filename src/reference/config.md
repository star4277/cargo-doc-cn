# 配置

本文档解释了 Cargo 的配置系统如何工作，以及
可用的密钥或配置。通过其配置包
清单，请参见[清单格式](manifest.md)。

## 层次结构

Cargo 允许对特定包进行本地配置以及全局配置
配置。它在当前目录中查找配置文件并
所有父目录。例如，如果调用 Cargo
`/projects/foo/bar/baz`，那么以下配置文件将是
按以下顺序进行探测和统一：

* `/projects/foo/bar/baz/.cargo/config.toml`
* `/projects/foo/bar/.cargo/config.toml`
* `/projects/foo/.cargo/config.toml`
* `/projects/.cargo/config.toml`
* `/.cargo/config.toml`
* `$CARGO_HOME/config.toml` 默认为：
    * Windows: `%USERPROFILE%\.cargo\config.toml`
    * Unix: `$HOME/.cargo/config.toml`

使用这种结构，您可以指定每个包的配置，甚至
可能会将其签入版本控制。您还可以指定个人默认值
在你的主目录中有一个配置文件。

如果在多个配置文件中指定了一个键，则这些值将被合并
一起。数字、字符串和布尔值将在更深层次中使用该值
config 目录优先于祖先目录，其中
主目录的优先级最低。数组将连接在一起
with higher precedence items being placed later in the merged array.

目前，当从工作区调用时，Cargo 不会读取配置
工作区中 crate 中的文件。即，如果工作空间中有两个板条箱
它名为“/projects/foo/bar/baz/mylib”和“/projects/foo/bar/baz/mybin”，以及
`/projects/foo/bar/baz/mylib/.cargo/config.toml` 有 Cargo 配置
和 `/projects/foo/bar/baz/mybin/.cargo/config.toml`，Cargo 不读取
如果从工作区根调用这些配置文件
（`/projects/foo/bar/baz/`）。

> **注意：** Cargo 还会读取不带 `.toml` 扩展名的配置文件，例如
> `.cargo/config`。 1.39 版本中添加了对“.toml”扩展名的支持
> 是首选形式。如果两个文件都存在，Cargo 将使用该文件
> 没有扩展名。

## 配置格式

配置文件以[TOML格式][toml]编写（就像
清单），在部分（表）内有简单的键值对。这
以下是所有设置的快速概述，并附有详细说明
下面找到了。

```toml
paths = ["/path/to/override"] # path dependency overrides

[alias]     # command aliases
b = "build"
c = "check"
t = "test"
r = "run"
rr = "run --release"
recursive_example = "rr --example recursions"
space_example = ["run", "--release", "--", "\"command list\""]

[build]
warnings = "warn"             # adjust the effective lint level for warnings
jobs = 1                      # number of parallel jobs, defaults to # of CPUs
rustc = "rustc"               # the rust compiler tool
rustc-wrapper = "…"           # run this wrapper instead of `rustc`
rustc-workspace-wrapper = "…" # run this wrapper instead of `rustc` for workspace members
rustdoc = "rustdoc"           # the doc generator tool
target = "triple"             # build for the target triple (ignored by `cargo install`)
target-dir = "target"         # path of where to place generated artifacts
build-dir = "target"          # path of where to place intermediate build artifacts
rustflags = ["…", "…"]        # custom flags to pass to all compiler invocations
rustdocflags = ["…", "…"]     # custom flags to pass to rustdoc
incremental = true            # whether or not to enable incremental compilation
dep-info-basedir = "…"        # path for the base directory for targets in depfiles

[credential-alias]
# Provides a way to define aliases for credential providers.
my-alias = ["/usr/bin/cargo-credential-example", "--argument", "value", "--flag"]

[doc]
browser = "chromium"          # browser to use with `cargo doc --open`,
                              # overrides the `BROWSER` environment variable

[env]
# Set ENV_VAR_NAME=value for any process run by Cargo
ENV_VAR_NAME = "value"
# Set even if already present in environment
ENV_VAR_NAME_2 = { value = "value", force = true }
# `value` is relative to the parent of `.cargo/config.toml`, env var will be the full absolute path
ENV_VAR_NAME_3 = { value = "relative/path", relative = true }

[future-incompat-report]
frequency = 'always' # when to display a notification about a future incompat report

[cache]
auto-clean-frequency = "1 day"   # How often to perform automatic cache cleaning

[cargo-new]
vcs = "none"              # VCS to use ('git', 'hg', 'pijul', 'fossil', 'none')

[http]
debug = false               # HTTP debugging
proxy = "host:port"         # HTTP proxy in libcurl format
ssl-version = "tlsv1.3"     # TLS version to use
ssl-version.max = "tlsv1.3" # maximum TLS version
ssl-version.min = "tlsv1.1" # minimum TLS version
timeout = 30                # timeout for each HTTP request, in seconds
low-speed-limit = 10        # network timeout threshold (bytes/sec)
cainfo = "cert.pem"         # path to Certificate Authority (CA) bundle
proxy-cainfo = "cert.pem"   # path to proxy Certificate Authority (CA) bundle
check-revoke = true         # check for SSL certificate revocation
multiplexing = true         # HTTP/2 multiplexing
user-agent = "…"            # the user-agent header

[install]
root = "/some/path"         # `cargo install` destination directory

[net]
retry = 3                   # network retries
git-fetch-with-cli = true   # use the `git` executable for git operations
offline = true              # do not access the network

[net.ssh]
known-hosts = ["..."]       # known SSH host keys

[patch.<registry>]
# Same keys as for [patch] in Cargo.toml

[profile.<name>]         # Modify profile settings via config.
inherits = "dev"         # Inherits settings from [profile.dev].
opt-level = 0            # Optimization level.
debug = true             # Include debug info.
split-debuginfo = '...'  # Debug info splitting behavior.
strip = "none"           # Removes symbols or debuginfo.
debug-assertions = true  # Enables debug assertions.
overflow-checks = true   # Enables runtime integer overflow checks.
lto = false              # Sets link-time optimization.
panic = 'unwind'         # The panic strategy.
incremental = true       # Incremental compilation.
codegen-units = 16       # Number of code generation units.
rpath = false            # Sets the rpath linking option.
[profile.<name>.build-override]  # Overrides build-script settings.
# Same keys for a normal profile.
[profile.<name>.package.<name>]  # Override profile for a package.
# Same keys for a normal profile (minus `panic`, `lto`, and `rpath`).

[resolver]
lockfile-path = "…"  # Overrides the path used for
incompatible-rust-versions = "allow"  # Specifies how resolver reacts to these

[registries.<name>]  # registries other than crates.io
index = "…"          # URL of the registry index
token = "…"          # authentication token for the registry
credential-provider = "cargo:token" # The credential provider for this registry.

[registries.crates-io]
protocol = "sparse"  # The protocol to use to access crates.io.

[registry]
default = "…"        # name of the default registry
token = "…"          # authentication token for crates.io
credential-provider = "cargo:token"           # The credential provider for crates.io.
global-credential-providers = ["cargo:token"] # The credential providers to use by default.

[source.<name>]      # source definition and replacement
replace-with = "…"   # replace this source with the given named source
directory = "…"      # path to a directory source
registry = "…"       # URL to a registry source
local-registry = "…" # path to a local registry source
git = "…"            # URL of a git repository source
branch = "…"         # branch name for the git repository
tag = "…"            # tag name for the git repository
rev = "…"            # revision for the git repository

[target.<triple>]
linker = "…"              # linker to use
runner = "…"              # wrapper to run executables
rustflags = ["…", "…"]    # custom flags for `rustc`
rustdocflags = ["…", "…"] # custom flags for `rustdoc`

[target.<cfg>]
linker = "…"            # linker to use
runner = "…"            # wrapper to run executables
rustflags = ["…", "…"]  # custom flags for `rustc`

[target.<triple>.<links>] # `links` build script override
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = "-L /some/path"
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"

[term]
quiet = false                    # whether cargo output is quiet
verbose = false                  # whether cargo provides verbose output
color = 'auto'                   # whether cargo colorizes output
hyperlinks = true                # whether cargo inserts links into output
unicode = true                   # whether cargo can render output using non-ASCII unicode characters
progress.when = 'auto'           # whether cargo shows progress bar
progress.width = 80              # width of progress bar
progress.term-integration = true # whether cargo reports progress to terminal emulator
```

## 环境变量

除了以下内容之外，还可以通过环境变量来配置 Cargo
TOML 配置文件。对于“foo.bar”形式的每个配置键
环境变量“CARGO_FOO_BAR”也可用于定义该值。
键将转换为大写，点和破折号将转换为下划线。
例如，“target.x86_64-unknown-linux-gnu.runner”密钥也可以是
由“CARGO_TARGET_X86_64_UNKNOWN_LINUX_GNU_RUNNER”环境定义
多变的。

环境变量将优先于 TOML 配置文件。
目前仅支持整数、布尔值、字符串和一些数组值
由环境变量定义。 [以下说明](#configuration-keys)
指示哪些键支持环境变量，否则不支持
由于[技术问题](https://github.com/rust-lang/cargo/issues/5416)而受到支持。

除了上述系统之外，Cargo 还识别其他一些特定的系统
[环境变量][env]。

## 命令行覆盖

Cargo 还可以通过以下方式接受任意配置覆盖
`--config` 命令行选项。参数应采用以下 TOML 语法：
`KEY=VALUE` 或作为额外配置文件的路径提供：

```console
# With `KEY=VALUE` in TOML syntax
cargo --config net.git-fetch-with-cli=true fetch

# With a path to a configuration file
cargo --config ./path/to/my/extra-config.toml fetch
```

`--config` 选项可以被指定多次，在这种情况下
使用相同的合并逻辑，按从左到右的顺序合并值
当应用多个配置文件时使用。配置
以这种方式指定的值优先于环境变量，
它优先于配置文件。

当“--config”选项作为额外的配置文件提供时，
通过这种方式加载的配置文件遵循相同的优先规则
与直接使用“--config”指定的其他选项一样。

使用 Bourne shell 语法的一些示例：

```console
# Most shells will require escaping.
cargo --config http.proxy=\"http://example.com\" …

# Spaces may be used.
cargo --config "net.git-fetch-with-cli = true" …

# TOML array example. Single quotes make it easier to read and write.
cargo --config 'build.rustdocflags = ["--html-in-header", "header.html"]' …

# Example of a complex TOML key.
cargo --config "target.'cfg(all(target_arch = \"arm\", target_os = \"none\"))'.runner = 'my-runner'" …

# Example of overriding a profile setting.
cargo --config profile.dev.package.image.opt-level=3 …
```

## 包括额外的配置文件

配置可以使用顶级“include”键包含其他配置文件。
这允许跨多个项目共享配置
或将复杂的配置拆分为多个文件。

### `包含`

* 类型：字符串或表格数组
* 默认值：无
* 环境：不支持

加载额外的配置文件。
路径与包含它们的配置文件相关。
仅接受以“.toml”结尾的路径。

支持以下格式：

```toml
# array of paths
include = [
    "frodo.toml",
    "samwise.toml",
]

# inline tables for more control
include = [
    { path = "required.toml" },
    { path = "optional.toml", optional = true },
]
```

> **注意：** 为了更好的可读性并避免混淆，建议：
> - 将 `include` 放在配置文件的顶部
> - 每行放置一个 include 以获得更清晰的版本控制差异
> - 当需要可选包含时使用内联表语法

使用表语法时，支持以下字段：

* `path` （字符串，必需）：要包含的配置文件的路径。
* `可选`（布尔值，默认值： false）：如果为`true`，丢失的文件将被静默处理
  跳过而不是导致错误。

`include` 的合并行为与其他配置值不同：

1. 首先从“include”路径加载配置值。
    * 包含的文件从左到右加载，
      后面的文件中的值优先于前面的文件。
    * 如果包含的配置文件也包含“include”键，则此步骤会递归。
2. 然后，配置文件自己的值被合并到包含的配置之上，
   具有最高优先权。

## 配置相对路径

配置文件中的路径可以是绝对路径、相对路径或不带任何路径分隔符的裸名称。
没有路径分隔符的可执行文件的路径将使用“PATH”环境变量来搜索可执行文件。
非可执行文件的路径将与定义配置值的位置相关。

具体来说，规则是：

* 对于环境变量，路径是相对于当前工作目录的。
* 对于直接从 [`--config KEY=VALUE`](#command-line-overrides) 选项加载的配置值，
  路径是相对于当前工作目录的。
* 对于配置文件，路径是相对于定义配置文件的目录的父目录，
  无论这些文件来自[分层探测](#hierarchical-struction)
  或 [`--config <path>`](#command-line-overrides) 选项。

> **注意：** 为了保持与现有 `.cargo/config.toml` 探测行为的一致性，
> 按照设计，配置文件中的路径通过 `--config <path>` 传递
> 也相对于配置文件本身向上两级。
>
> 为了避免意外结果，经验法则是放置额外的配置文件
> 在项目中发现的 `.cargo/config.toml` 的同一级别。
> 例如，给定一个项目 `/my/project`，
> 建议将配置文件放在 `/my/project/.cargo` 下
> 或同一级别的新目录，例如“/my/project/.config”。

```toml
# Relative path examples.

[target.x86_64-unknown-linux-gnu]
runner = "foo"  # Searches `PATH` for `foo`.

[source.vendored-sources]
# Directory is relative to the parent where `.cargo/config.toml` is located.
# For example, `/my/project/.cargo/config.toml` would result in `/my/project/vendor`.
directory = "vendor"
```

## 带参数的可执行路径

一些 Cargo 命令调用外部程序，可以将其配置为路径
和一些参数。

该值可以是字符串数组，例如“['/path/to/program', 'somearg']”或
以空格分隔的字符串，例如“/path/to/program somearg”。如果路径为
可执行文件包含空格，必须使用列表形式。

如果 Cargo 将其他参数传递给程序，例如打开或的路径
运行时，它们将在值的最后一个指定参数之后传递
该格式的选项。如果指定的程序没有路径分隔符，
Cargo 将在“PATH”中搜索其可执行文件。

#＃ 证书

具有敏感信息的配置值存储在
`$CARGO_HOME/credentials.toml` 文件。该文件是自动创建和更新的
使用 [`cargo:token`] 凭证提供程序时，通过 [`cargo login`] 和 [`cargo logout`]。

某些 Cargo 命令使用令牌，例如 [`cargopublish`]
使用远程注册表进行身份验证。应注意保护
令牌并对其保密。

它遵循与 Cargo 配置文件相同的格式。

```toml
[registry]
token = "…"   # Access token for crates.io

[registries.<name>]
token = "…"   # Access token for the named registry
```

与大多数其他配置值一样，可以使用环境指定令牌
变量。 [crates.io] 的令牌可以通过以下方式指定
`CARGO_REGISTRY_TOKEN` 环境变量。其他注册中心的令牌可能
使用以下形式的环境变量指定
`CARGO_REGISTRIES_<name>_TOKEN` 其中 `<name>` 是注册表的名称
全部大写字母。

> **注意：** Cargo 还可以读取和写入没有 `.toml` 的凭证文件
> 扩展名，例如“.cargo/credentials”。支持 `.toml` 扩展名
> 是在1.39版本中添加的。在版本 1.68 中，Cargo 使用以下内容写入文件
> 默认扩展名。然而，出于向后兼容性的原因，当两者
> 文件存在，Cargo 将读取和写入不带扩展名的文件。

## 配置键

本节记录了所有配置键。键的描述为
变量部分用尖括号注释，如“target.<triple>”，其中
`<triple>` 部分可以是任何[目标三元组]，例如
`target.x86_64-pc-windows-msvc`。

### `[paths]`
* Type: array of strings (paths)
* Default: none
* Environment: not supported

本地包的路径数组，将用作覆盖
依赖关系。有关详细信息，请参阅[覆盖依赖项
指南]（overriding-dependency.md#paths-overrides）。

### `[alias]`
* Type: string or array of strings
* Default: see below
* Environment: `CARGO_ALIAS_<name>`

`[alias]` 表定义 CLI 命令别名。例如，运行“cargo”
b` 是运行 `cargo build` 的别名。表中的每个键都是
子命令，值是实际要运行的命令。该值可能是一个
字符串数组，其中第一个元素是命令，以下是
论据。它也可能是一个字符串，它将被空格分割成
子命令和参数。 Cargo 内置了以下别名：

```toml
[alias]
b = "build"
c = "check"
d = "doc"
t = "test"
r = "run"
rm = "remove"
```

别名不允许重新定义现有的内置命令。

别名是递归的：

```toml
[alias]
rr = "run --release"
recursive_example = "rr --example recursions"
```

### `[build]`

“[build]”表控制构建时操作和编译器设置。

#### `build.warnings`
* Type: string
* Default: `"warn"`
* Environment: `CARGO_BUILD_WARNINGS`

调整本地包的 lint 警告的有效级别。
允许的级别是：
* `"warn"`：继续发出 lints 作为警告（默认）。
* `"allow"`：隐藏 lints。
* `"deny"`：对于有 lint 警告的 crate 发出错误。
  使用 `--keep-going` 查看所有依赖包的 lint 警告。

仅棉绒警告（即级别可调）受到影响，
例如保留原样的非 lint 警告或来自依赖项的警告，通过 `--verbose --verbose` 可见。

> **MSRV:** Respected as of 1.97.

#### `build.jobs`
* Type: integer or string
* Default: number of logical CPUs
* Environment: `CARGO_BUILD_JOBS`

设置并行运行的编译器进程的最大数量。如果为负数，
它将编译器进程的最大数量设置为逻辑 CPU 的数量
加上提供的价值。不应为 0。如果提供了字符串“default”，则它会设置
值恢复为默认值。

可以使用 `--jobs` CLI 选项覆盖。

#### `build.rustc`
* Type: string (program path)
* Default: `"rustc"`
* Environment: `CARGO_BUILD_RUSTC` or `RUSTC`

设置用于“rustc”的可执行文件。

#### `build.rustc-wrapper`
* Type: string (program path)
* Default: none
* Environment: `CARGO_BUILD_RUSTC_WRAPPER` or `RUSTC_WRAPPER`

设置要执行的包装器而不是“rustc”。第一个参数传递给
包装器是要使用的实际可执行文件的路径
（即“build.rustc”，如果已设置，则为“rustc”，否则）。

#### `build.rustc-workspace-wrapper`
* Type: string (program path)
* Default: none
* Environment: `CARGO_BUILD_RUSTC_WORKSPACE_WRAPPER` or `RUSTC_WORKSPACE_WRAPPER`

设置一个包装器来执行，而不是“rustc”，仅适用于工作区成员。当建造一个
没有工作区的单包项目，该包被视为工作区。第一个
传递给包装器的参数是要使用的实际可执行文件的路径（即“build.rustc”，如果
已设置，否则为“rustc”）。它会影响文件名哈希，以便由
包装器单独缓存。

如果同时设置了“rustc-wrapper”和“rustc-workspace-wrapper”，那么它们将被嵌套：
最终调用是“$RUSTC_WRAPPER $RUSTC_WORKSPACE_WRAPPER $RUSTC”。

#### `build.rustdoc`
* Type: string (program path)
* Default: `"rustdoc"`
* Environment: `CARGO_BUILD_RUSTDOC` or `RUSTDOC`

设置用于“rustdoc”的可执行文件。

#### `build.target`
* Type: string or array of strings
* Default: host platform
* Environment: `CARGO_BUILD_TARGET`

要编译到的默认[target platform triples][target triple] 。

Possible values:
- `rustc --print target-list` 中任何支持的目标。
- `“host-tuple”`，它将在内部被主机的目标替换。如果您正在交叉编译一些包，并且不想将主机的计算机指定为目标（例如，可能由许多主机处理的共享项目中的“xtask”），那么这可能特别有用。
- 自定义目标规范的路径。有关更多信息，请参阅[自定义目标查找路径](../../rustc/targets/custom.html#custom-target-lookup-path)。

可以使用 `--target` CLI 选项覆盖。

```toml
[build]
target = ["x86_64-unknown-linux-gnu", "i686-unknown-linux-gnu"]
```

#### `build.target-dir`
* Type: string (path)
* Default: `"target"`
* Environment: `CARGO_BUILD_TARGET_DIR` or `CARGO_TARGET_DIR`

所有编译器输出放置的路径。如果不指定则默认
是位于工作空间根目录的名为“target”的目录。

可以使用 `--target-dir` CLI 选项覆盖。

有关更多信息，请参阅[构建缓存文档](../reference/build-cache.md)。

#### `build.build-dir`

* Type: string (path)
* Default: Defaults to the value of `build.target-dir`
* Environment: `CARGO_BUILD_BUILD_DIR`

将存储中间构建工件的目录。
中间工件是由 Rustc/Cargo 在构建过程中生成的。

此选项支持路径模板。

可用的模板变量：
* `{workspace-root}` 解析为当前工作空间的根目录。
* `{cargo-cache-home}` 解析为 `CARGO_HOME`
* `{workspace-path-hash}` 解析为清单路径的哈希值

有关更多信息，请参阅[构建缓存文档](../reference/build-cache.md)。

#### `build.rustflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_BUILD_RUSTFLAGS` or `CARGO_ENCODED_RUSTFLAGS` or `RUSTFLAGS`

要传递给“rustc”的额外命令行标志。该值可能是一个数组
字符串或空格分隔的字符串。

额外标志有四个互斥的来源。他们已登记入住
顺序，使用第一个：

1. `CARGO_ENCODED_RUSTFLAGS` 环境变量。
2. `RUSTFLAGS` 环境变量。
3. 所有匹配的 `target.<triple>.rustflags` 和 `target.<cfg>.rustflags`
   配置条目连接在一起。
4. `build.rustflags` 配置值。

还可以使用 [`cargo rustc`] 命令传递其他标志。

如果使用了 `--target` 标志（或 [`build.target`](#buildtarget)），那么
标志只会传递给目标的编译器。正在建造的东西
对于主机，例如构建脚本或 proc 宏，将不会收到参数。
如果没有“--target”，标志将传递给所有编译器调用
（包括构建脚本和过程宏）因为依赖项是共享的。如果
您不想将参数传递给构建脚本或过程宏，并且
正在为主机构建，请使用 [主机三元组][目标三元组] 传递 `--target`。

不建议传入 Cargo 本身通常管理的标志。为了
例如，由 [profiles](profiles.md) 驱动的标志最好通过设置
适当的配置文件设置。

> **警告**：由于直接将标志传递给的低级性质
> 编译器，这可能会导致与 Cargo 的未来版本发生冲突，从而可能
> 自行发出相同或相似的标志，这可能会干扰
> 您指定的标志。在这个领域，Cargo 可能并不总是落后
> 兼容。

#### `build.rustdocflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_BUILD_RUSTDOCFLAGS` or `CARGO_ENCODED_RUSTDOCFLAGS` or `RUSTDOCFLAGS`

要传递给“rustdoc”的额外命令行标志。该值可能是一个数组
字符串或空格分隔的字符串。

额外标志有四个互斥的来源。他们已登记入住
顺序，使用第一个：

1. `CARGO_ENCODED_RUSTDOCFLAGS` 环境变量。
2. `RUSTDOCFLAGS` 环境变量。
3. 所有匹配的 `target.<triple>.rustdocflags` 和 `target.<cfg>.rustdocflags`
  配置条目连接在一起。
4. `build.rustdocflags` 配置值。

还可以使用 [`cargo rustdoc`] 命令传递其他标志。

> **警告**：由于直接将标志传递给的低级性质
> 编译器，这可能会导致与 Cargo 的未来版本发生冲突，从而可能
> 自行发出相同或相似的标志，这可能会干扰
> 您指定的标志。在这个领域，Cargo 可能并不总是落后
> 兼容。

#### `build.incremental`
* Type: bool
* Default: from profile
* Environment: `CARGO_BUILD_INCREMENTAL` or `CARGO_INCREMENTAL`

是否执行【增量编译】。如果不设置则默认为
使用[配置文件](profiles.md#incremental)中的值。否则这会覆盖设置
所有配置文件。

可以将“CARGO_INCRMENTAL”环境变量设置为“1”以强制启用
所有配置文件的增量编译，或“0”以禁用它。这个环境变量
覆盖配置设置。

#### `build.dep-info-basedir`
* Type: string (path)
* Default: none
* Environment: `CARGO_BUILD_DEP_INFO_BASEDIR`

从 [dep 中删除给定的路径前缀
info](../reference/build-cache.md#dep-info-files) 文件路径。这个配置设置
旨在将绝对路径转换为需要的工具的相对路径
相对路径。

设置本身是配置相对路径。因此，例如，值为
`"."` 将删除以 `.cargo` 的父目录开头的所有路径
目录。

#### `build.pipelining`

此选项已弃用且未使用。 Cargo 始终启用管道传输。

### `[credential-alias]`
* Type: string or array of strings
* Default: empty
* Environment: `CARGO_CREDENTIAL_ALIAS_<name>`

“[credential-alias]”表定义凭证提供者别名。
这些别名可以作为“registry.global-credential-providers”的元素进行引用
数组，或作为特定注册表的凭据提供程序
在“registries.<NAME>.credential-provider”下。

如果指定为字符串，则该值将按空格拆分为路径和参数。

例如，定义一个名为“my-alias”的别名：

```toml
[credential-alias]
my-alias = ["/usr/bin/cargo-credential-example", "--argument", "value", "--flag"]
```
更多信息请参见[Registry Authentication](registry-authentication.md)。

### `[doc]`

`[doc]` 表定义了 [`cargo doc`] 命令的选项。

#### `doc.browser`

* Type: string or array of strings ([program path with args])
* Default: `BROWSER` 环境变量，或者，如果缺少该变量，
  以系统特定方式打开链接

此选项设置 [`cargo doc`] 使用的浏览器，覆盖
使用“--open”打开文档时的“BROWSER”环境变量
选项。

### `[cargo-new]`

`[cargo-new]` 表定义了 [`cargo new`] 命令的默认值。

#### `cargo-new.name`

此选项已弃用且未使用。

#### `cargo-new.email`

此选项已弃用且未使用。

#### `cargo-new.vcs`
* Type: string
* Default: `"git"` or `"none"`
* Environment: `CARGO_CARGO_NEW_VCS`

指定用于初始化新存储库的源代码控制系统。
有效值为“git”、“hg”（对于 Mercurial）、“pijul”、“fossil” 或“none”
禁用此行为。默认为“git”，如果已在 VCS 内，则默认为“none”
存储库。可以使用 `--vcs` CLI 选项覆盖。

### `[env]`

`[env]` 部分允许您设置额外的环境变量
构建脚本、rustc 调用、“cargo run”和“cargo build”。

```toml
[env]
OPENSSL_DIR = "/opt/openssl"
```

默认情况下，指定的变量不会覆盖已经存在的值
在环境中。可以通过设置“force”标志来更改此行为。

设置“relative”标志会将值评估为配置相对路径
相对于包含以下内容的 `.cargo` 目录的父目录
`config.toml` 文件。环境变量的值将是完整的
绝对路径。

```toml
[env]
TMPDIR = { value = "/home/tmp", force = true }
OPENSSL_DIR = { value = "vendor/openssl", relative = true }
```

### `[future-incompat-report]`

`[future-incompat-report]` 表控制 [future incompat reporting](future-incompat-report.md) 的设置

#### `future-incompat-report.frequency`
* Type: string
* Default: `"always"`
* Environment: `CARGO_FUTURE_INCOMPAT_REPORT_FREQUENCY`

控制当将来有不兼容报告可用时向终端显示通知的频率。可能的值：

* `always`（默认）：当命令（例如`cargo build`）生成未来不兼容报告时始终显示通知
* `never`: 从不显示通知

### `[cache]`

`[cache]` 表定义了货物缓存的设置。

#### 全局缓存

运行“cargo”命令时，Cargo 将自动跟踪您在全局缓存中使用的文件。
Cargo 会定期删除一段时间未使用的文件。
如果3个月内没有使用过，它将删除必须从网络下载的文件。无需网络访问即可生成的文件，如果 1 个月内未使用，将被删除。

仅当运行已经执行大量工作的命令时才会自动删除文件，例如所有构建命令（“cargo build”、“cargo test”、“cargo check”等）和“cargo fetch”。

如果货物离线，则禁用自动删除，例如使用“--offline”或“--frozen”，以避免删除长时间离线时可能需要使用的工件。

> **注意**：此跟踪目前仅针对 Cargo 主目录中的全局缓存实现。
> 这包括注册表索引和从注册表下载的源文件以及 git 依赖项。
> 对跟踪构建工件的支持尚未实现，并在 [cargo#13136](https://github.com/rust-lang/cargo/issues/13136) 中进行跟踪。
>
> 此外，还有一个不稳定的功能，支持“手动”触发缓存清理，并进一步自定义配置选项。
> 更多信息请参见[不稳定章节](unstable.md#gc)。

#### `cache.auto-clean-frequency`
* Type: string
* Default: `"1 day"`
* Environment: `CARGO_CACHE_AUTO_CLEAN_FREQUENCY`

此选项定义 Cargo 自动删除全局缓存中未使用文件的频率。
这*不*定义文件必须有多老，这些阈值在[上面]（#global-caches）中进行了描述。

它支持以下设置：

* `"never"` --- 从不删除旧文件。
* `"always"` --- 每次 Cargo 运行时检查是否删除旧文件。
* 一个整数，后跟“秒”、“分钟”、“小时”、“天”、“周”或“月” --- 检查最多在给定时间范围内删除旧文件。

### `[http]`

`[http]` 表定义 HTTP 行为的设置。这包括获取
crate 依赖项并访问远程 git 存储库。

#### `http.debug`
* Type: boolean
* Default: false
* Environment: `CARGO_HTTP_DEBUG`

如果为“true”，则启用 HTTP 请求的调试。调试信息可以是
通过设置`CARGO_LOG=network=debug`环境看到
变量（或使用“network=trace”获取更多信息）。

在公共位置发布此输出的日志时要小心。输出
可能包含带有您不想泄漏的身份验证令牌的标头！
在发布日志之前请务必查看日志。

#### `http.proxy`
* Type: string
* Default: none
* Environment: `CARGO_HTTP_PROXY` or `HTTPS_PROXY` or `https_proxy` or `http_proxy`

设置要使用的 HTTP 和 HTTPS 代理。格式为 [libcurl format] 如下
`[协议://]主机[:端口]`。如果未设置，Cargo 还将检查 `http.proxy`
在全局 git 配置中进行设置。如果这些都没有设置，
`HTTPS_PROXY` 或 `https_proxy` 环境变量设置 HTTPS 代理
请求，“http_proxy”将其设置为 HTTP 请求。

#### `http.timeout`
* Type: integer
* Default: 30
* Environment: `CARGO_HTTP_TIMEOUT` or `HTTP_TIMEOUT`

设置每个 HTTP 请求的超时时间（以秒为单位）。

#### `http.cainfo`
* Type: string (path)
* Default: none
* Environment: `CARGO_HTTP_CAINFO`

证书颁发机构 (CA) 捆绑文件的路径，用于验证 TLS
证书。如果未指定，Cargo 会尝试使用系统证书。

#### `http.proxy-cainfo`
* Type: string (path)
* Default: falls back to `http.cainfo` if not set
* Environment: `CARGO_HTTP_PROXY_CAINFO`

证书颁发机构 (CA) 捆绑文件的路径，用于验证代理 TLS
证书。

#### `http.check-revoke`
* Type: boolean
* Default: true (Windows) false (all others)
* Environment: `CARGO_HTTP_CHECK_REVOKE`

这决定是否应进行 TLS 证书吊销检查
执行。这仅适用于 Windows。

#### `http.ssl-version`
* Type: string or min/max table
* Default: none
* Environment: `CARGO_HTTP_SSL_VERSION`

这将设置要使用的最低 TLS 版本。它需要一个字符串，其中一个
可能的值为 `"default"`、`"tlsv1"`、`"tlsv1.0"`、`"tlsv1.1"`、`"tlsv1.2"` 或
`“tlsv1.3”`。

这也可以采用带有两个键“min”和“max”的表，每个键
取指定最小值和最大值的同类字符串值
要使用的 TLS 版本范围。

默认最低版本为“tlsv1.0”，最高版本为最新版本
您的平台支持，通常为“tlsv1.3”。

#### `http.low-speed-limit`
* Type: integer
* Default: 10
* Environment: `CARGO_HTTP_LOW_SPEED_LIMIT`

此设置控制慢速连接的超时行为。如果平均
传输速度（以字节/秒为单位）低于给定值
[`http.timeout`](#httptimeout) 秒（默认 30 秒），然后
连接被认为太慢，Cargo 将中止并重试。

#### `http.multiplexing`
* Type: boolean
* Default: true
* Environment: `CARGO_HTTP_MULTIPLEXING`

当为“true”时，Cargo 将尝试使用具有多路复用功能的 HTTP2 协议。
这允许多个请求使用同一个连接，通常会改进
获取多个文件时的性能。如果为 false，Cargo 将使用 HTTP 1.1
无需流水线。

#### `http.user-agent`
* Type: string
* Default: Cargo's version
* Environment: `CARGO_HTTP_USER_AGENT`

指定要使用的自定义用户代理标头。如果未指定，则默认为
包含 Cargo 版本的字符串。

### `[install]`

`[install]` 表定义了 [`cargo install`] 命令的默认值。

#### `install.root`
* Type: string (path)
* Default: Cargo's home directory
* Environment: `CARGO_INSTALL_ROOT`

设置根目录的路径，用于安装 [`cargo
安装`]。可执行文件进入根目录下的“bin”目录。

为了跟踪已安装的可执行文件的信息，一些额外的文件，例如
`.crates.toml` 和 `.crates2.json` 也是在此根目录下创建的。

如果未指定，则默认为 Cargo 的主目录（默认为“.cargo”）
您的主目录）。

可以使用“--root”命令行选项覆盖。

### `[net]`

`[net]` 表控制网络配置。

#### `net.retry`
* Type: integer
* Default: 3
* Environment: `CARGO_NET_RETRY`

重试可能的虚假网络错误的次数。

#### `net.git-fetch-with-cli`
* Type: boolean
* Default: false
* Environment: `CARGO_NET_GIT_FETCH_WITH_CLI`

如果这是“true”，那么 Cargo 将使用“git”可执行文件来获取注册表
索引和 git 依赖项。如果为“false”，则它使用内置的“git”
图书馆。

如果您有特殊的身份验证，将其设置为“true”会很有帮助
Cargo 不支持的要求。请参阅[Git
身份验证](../appendix/git-authentication.md) 了解更多信息
设置 git 身份验证。

#### `net.offline`
* Type: boolean
* Default: false
* Environment: `CARGO_NET_OFFLINE`

如果这是“true”，那么 Cargo 将避免访问网络，并尝试
继续处理本地缓存的数据。如果为“false”，Cargo 将按照以下方式访问网络
需要，如果遇到网络错误则生成错误。

可以使用“--offline”命令行选项覆盖。

#### `net.ssh`

`[net.ssh]` 表包含 SSH 连接的设置。

#### `net.ssh.known-hosts`
* Type: array of strings
* Default: see description
* Environment: not supported

`known-hosts` 数组包含应该是的 SSH 主机密钥列表
连接到 SSH 服务器时被接受为有效（例如 SSH git
依赖项）。每个条目应该是格式类似于 OpenSSH 的字符串
`known_hosts` 文件。每个字符串应以一个或多个主机名开头
用逗号、空格、键类型名称、空格和
Base64 编码的密钥。例如：

```toml
[net.ssh]
known-hosts = [
    "example.com ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFO4Q5T0UV0SQevair9PFwoxY9dl4pQl3u5phoqJH3cF"
]
```

Cargo 将尝试从支持的常见位置加载已知主机密钥
OpenSSH，并将加入 Cargo 配置文件中列出的任何内容。
如果任何匹配条目具有正确的密钥，则将允许连接。

Cargo 附带内置 [github.com][github-keys] 的主机密钥。如果
如果这些更改发生变化，您可以将新密钥添加到 config 或known_hosts 文件中。

请参阅[Git 身份验证](../appendix/git-authentication.md#ssh-known-hosts)
了解更多详情。

[github-keys]: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints

### `[patch]`

正如您可以使用 [`[patch]` 覆盖依赖项
`Cargo.toml`](overriding-dependency.md#the-patch-section)，你可以
在货物配置文件中覆盖它们以应用这些补丁
任何受影响的构建。格式与中使用的格式相同
`Cargo.toml`。

由于 `.cargo/config.toml` 文件通常不会签入源代码
控制，您应该尽可能使用“Cargo.toml”进行修补
确保其他开发人员可以在自己的包中编译您的包
环境。通过货物配置文件进行修补通常是
仅当补丁部分由自动生成时才适用
外部构建工具。

如果给定的依赖项在货物配置文件和
`Cargo.toml` 文件，使用配置文件中的补丁。如果
多个配置文件修补相同的依赖项，标准货物
使用配置合并，优先选择最接近定义的值
到当前目录，使用 `$HOME/.cargo/config.toml`
最低优先级。

这样的“[patch]”部分中的相对“path”依赖关系已解决
相对于它们出现的配置文件。

### `[profile]`

`[profile]` 表可用于全局更改配置文件设置，并且
覆盖 `Cargo.toml` 中指定的设置。它具有相同的语法和
选项如“Cargo.toml”中指定的配置文件。请参阅[配置文件章节]
有关选项的详细信息。

[Profiles chapter]: profiles.md

#### `[profile.<name>.build-override]`
* Environment: `CARGO_PROFILE_<name>_BUILD_OVERRIDE_<key>`

构建覆盖表覆盖构建脚本、过程宏、
以及他们的依赖关系。它具有与普通配置文件相同的键。请参阅
[覆盖部分](profiles.md#overrides) 了解更多详细信息。

#### `[profile.<name>.package.<name>]`
* Environment: not supported

包表会覆盖特定包的设置。它有相同的
键作为普通配置文件，减去“panic”、“lto”和“rpath”设置。看
[覆盖部分](profiles.md#overrides) 了解更多详细信息。

#### `profile.<name>.codegen-units`
* Type: integer
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_CODEGEN_UNITS`

请参阅 [codegen-units](profiles.md#codegen-units)。

#### `profile.<name>.debug`
* Type: integer or boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_DEBUG`

请参阅[调试](profiles.md#debug)。

#### `profile.<name>.split-debuginfo`
* Type: string
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_SPLIT_DEBUGINFO`

请参阅 [split-debuginfo](profiles.md#split-debuginfo)。

#### `profile.<name>.debug-assertions`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_DEBUG_ASSERTIONS`

请参阅[调试断言](profiles.md#debug-assertions)。

#### `profile.<name>.incremental`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_INCREMENTAL`

请参阅[增量](profiles.md#incremental)。

#### `profile.<name>.lto`
* Type: string or boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_LTO`

请参阅 [lto](profiles.md#lto)。

#### `profile.<name>.overflow-checks`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_OVERFLOW_CHECKS`

请参阅[溢出检查](profiles.md#overflow-checks)。

#### `profile.<name>.opt-level`
* Type: integer or string
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_OPT_LEVEL`

请参阅 [opt-level](profiles.md#opt-level)。

#### `profile.<name>.panic`
* Type: string
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_PANIC`

请参阅[恐慌](profiles.md#panic)。

#### `profile.<name>.rpath`
* Type: boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_RPATH`

请参阅 [rpath](profiles.md#rpath)。

#### `profile.<name>.strip`
* Type: string or boolean
* Default: See profile docs.
* Environment: `CARGO_PROFILE_<name>_STRIP`

请参阅 [strip](profiles.md#strip)。

### `[resolver]`

`[resolver]` 表会覆盖本地开发的 [依赖关系解析行为](resolver.md)（例如，排除 `cargo install`）。

#### `resolver.lockfile-path`
* Type: string (path)
* Default: `<workspace_root>/Cargo.lock`
* Environment: `CARGO_RESOLVER_LOCKFILE_PATH`

指定解析依赖项时要使用的锁定文件的路径。
当使用只读源目录时，此选项非常有用。

该路径必须以“Cargo.lock”结尾。

> **MSRV:** Requires 1.97+

#### `resolver.incompatible-rust-versions`
* Type: string
* Default: See [`resolver`](resolver.md#resolver-versions) docs
* Environment: `CARGO_RESOLVER_INCOMPATIBLE_RUST_VERSIONS`

解析要使用的依赖项版本时，选择如何处理具有不兼容 `package.rust-version` 的版本。
值包括：
- `allow`：像任何其他版本一样对待`rust-version`不兼容的版本
- `fallback`：如果没有其他版本匹配，则仅考虑 `rust-version` 不兼容的版本

可以被覆盖
- `--ignore-rust-version` CLI 选项
- 将依赖项的版本要求设置为高于具有兼容“rust-version”的任何版本
- 使用“--precise”指定“cargo update”的版本

有关更多详细信息，请参阅 [resolver](resolver.md#rust-version) 章节。

> **MSRV:**
> - `allow` is supported on any version
> - `fallback` is respected as of 1.84

### `[registries]`

`[registries]` 表用于指定附加的 [registries]。它
由每个命名注册表的子表组成。

#### `registries.<name>.index`
* Type: string (url)
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_INDEX`

指定注册表索引的 URL。

#### `registries.<name>.token`
* Type: string
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_TOKEN`

指定给定注册表的身份验证令牌。这个值应该
仅出现在 [credentials](#credentials) 文件中。这是用于注册表的
像 [`cargopublish`] 这样的命令需要身份验证。

可以使用“--token”命令行选项覆盖。

#### `registries.<name>.credential-provider`
* Type: string or array of path and arguments
* Default: none
* Environment: `CARGO_REGISTRIES_<name>_CREDENTIAL_PROVIDER`

指定给定注册表的凭据提供程序。如果未设置，则
[`registry.global-credential-providers`](#registryglobal-credential-providers) 中的提供程序
将被使用。

如果指定为字符串，路径和参数将按空格分隔。为了
包含空格的路径或参数，请使用数组。

如果该值存在于 [`[credential-alias]`](#credential-alias) 表中，则将使用别名。

更多信息请参见[注册表认证](registry-authentication.md)。

#### `registries.crates-io.protocol`
* Type: string
* Default: `"sparse"`
* Environment: `CARGO_REGISTRIES_CRATES_IO_PROTOCOL`

指定用于访问 crates.io 的协议。允许的值为“git”或“sparse”。

`git` 使 Cargo 克隆曾经从 <https://github.com/rust-lang/crates.io-index/> 发布到 [crates.io] 的所有包的整个索引。
由于索引的大小，这可能会对性能产生影响。
`sparse` 是一种较新的协议，它使用 HTTPS 仅从 <https://index.crates.io/> 下载必需的内容。
在大多数情况下，这可以显着提高解决新依赖项的性能。

有关注册表协议的更多信息可以在[注册表章节](registries.md)中找到。

### `[registry]`

“[registry]”表控制未使用时使用的默认注册表
指定的。

#### `registry.index`

该值不再被接受并且不应使用。

#### `registry.default`
* Type: string
* Default: `"crates-io"`
* Environment: `CARGO_REGISTRY_DEFAULT`

要使用的注册表名称（来自 [`registries` 表](#registries)）
默认情况下，对于诸如 [`cargopublish`] 之类的注册表命令。

可以使用“--registry”命令行选项覆盖。

#### `registry.credential-provider`
* Type: string or array of path and arguments
* Default: none
* Environment: `CARGO_REGISTRY_CREDENTIAL_PROVIDER`

指定 [crates.io] 的凭证提供程序。如果未设置，则
[`registry.global-credential-providers`](#registryglobal-credential-providers) 中的提供程序
将被使用。

如果指定为字符串，路径和参数将按空格分隔。为了
包含空格的路径或参数，请使用数组。

如果该值存在于“[credential-alias]”表中，则将使用该别名。

更多信息请参见[注册表认证](registry-authentication.md)。

#### `registry.token`
* Type: string
* Default: none
* Environment: `CARGO_REGISTRY_TOKEN`

指定 [crates.io] 的身份验证令牌。该值应该只
出现在 [credentials](#credentials) 文件中。这是用于注册表的
像 [`cargopublish`] 这样的命令需要身份验证。

可以使用“--token”命令行选项覆盖。

#### `registry.global-credential-providers`
* Type: array
* Default: `["cargo:token"]`
* Environment: `CARGO_REGISTRY_GLOBAL_CREDENTIAL_PROVIDERS`

指定全局凭据提供程序的列表。如果未设置凭证提供者
对于使用 `registries.<name>.credential-provider` 的特定注册表，Cargo 将使用
此列表中的凭证提供者。列表末尾的提供者具有优先权。

路径和参数按空格分开。如果路径或参数包含空格，则凭证
提供程序应在 [`[credential-alias]`](#credential-alias) 表中定义，并且
此处通过其别名引用。

更多信息请参见[注册表认证](registry-authentication.md)。

### `[source]`

“[source]”表定义可用的注册表源。参见[来源
更换]了解更多信息。它由每个命名的子表组成
来源。一个源应该只定义一种（目录、注册表、
本地注册表或 git）。

#### `source.<name>.replace-with`
* Type: string
* Default: none
* Environment: not supported

如果设置，请将此源替换为给定的命名源或命名注册表。

#### `source.<name>.directory`
* Type: string (path)
* Default: none
* Environment: not supported

设置用作目录源的目录的路径。

#### `source.<name>.registry`
* Type: string (url)
* Default: none
* Environment: not supported

设置用于注册表源的 URL。

#### `source.<name>.local-registry`
* Type: string (path)
* Default: none
* Environment: not supported

设置用作本地注册表源的目录的路径。

#### `source.<name>.git`
* Type: string (url)
* Default: none
* Environment: not supported


设置用于 git 存储库源的 URL。

#### `source.<name>.branch`
* Type: string
* Default: none
* Environment: not supported

设置用于 git 存储库的分支名称。

如果没有设置“branch”、“tag”或“rev”，则默认为“master”分支。

#### `source.<name>.tag`
* Type: string
* Default: none
* Environment: not supported

设置用于 git 存储库的标签名称。

如果没有设置“branch”、“tag”或“rev”，则默认为“master”分支。

#### `source.<name>.rev`
* Type: string
* Default: none
* Environment: not supported

设置用于 git 存储库的 [revision]。

如果没有设置“branch”、“tag”或“rev”，则默认为“master”分支。


### `[target]`

`[target]` 表用于指定特定平台的设置
目标。它由一个子表组成，该子表可以是[平台三元组][目标三元组]
或 [`cfg()` 表达式]。如果目标平台将使用给定值
匹配“<triple>”值或“<cfg>”表达式。

```toml
[target.thumbv7m-none-eabi]
linker = "arm-none-eabi-gcc"
runner = "my-emulator"
rustflags = ["…", "…"]

[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "my-arm-wrapper"
rustflags = ["…", "…"]
```

`cfg` 值来自编译器内置的值（运行 `rustc --print=cfg`
查看）和额外的 `--cfg` 标志传递给 `rustc` （例如那些定义在
`铁锈标志`）。不要尝试匹配 `debug_assertions`、`test`、Cargo 功能
像`feature =“foo”`，或由[构建脚本]设置的值。

如果使用目标规范 JSON 文件，则 [`<triple>`] 值是文件名主干。
例如 `--target foo/bar.json` 将匹配 `[target.bar]`。

#### `target.<triple>.ar`

此选项已弃用且未使用。

#### `target.<triple>.linker`
* Type: string (program path)
* Default: none
* Environment: `CARGO_TARGET_<triple>_LINKER`

当正在编译 [`<triple>`]。默认情况下，链接器不会被覆盖。

#### `target.<cfg>.linker`
这与 [target linker](#targettriplelinker) 类似，但使用
[`cfg()` 表达式]。如果 [`<triple>`] 和 `<cfg>` 链接器都匹配，
`<triple>` 将优先。如果超过一个则出现错误
`<cfg>` 链接器与当前目标匹配。

#### `target.<triple>.runner`
* Type: string or array of strings ([program path with args])
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUNNER`

如果提供了运行程序，则目标 [`<triple>`] 的可执行文件将是
通过调用指定的运行程序来执行，并将实际可执行文件传递为
一个论点。这适用于[`cargo run`]、[`cargo test`]和[`cargo bench`]
命令。默认情况下，直接执行编译后的可执行文件。

#### `target.<cfg>.runner`

这与 [target runner](#targettriplerunner) 类似，但使用
[`cfg()` 表达式]。如果 [`<triple>`] 和 `<cfg>` 跑步者都匹配，
`<triple>` 将优先。如果超过一个则出现错误
`<cfg>` 运行程序与当前目标匹配。

#### `target.<triple>.rustflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUSTFLAGS`

为此 [`<triple>`] 将一组自定义标志传递给编译器。
该值可以是字符串数组或空格分隔的字符串。

有关不同版本的更多详细信息，请参阅 [`build.rustflags`](#buildrustflags)
特定额外标志的方法。

#### `target.<cfg>.rustflags`

这与 [target rustflags](#targettriplerustflags) 类似，但是
使用 [`cfg()` 表达式]。如果有多个 `<cfg>` 和 [`<triple>`] 条目
匹配当前目标，标志连接在一起。

#### `target.<triple>.rustdocflags`
* Type: string or array of strings
* Default: none
* Environment: `CARGO_TARGET_<triple>_RUSTDOCFLAGS`

为此 [`<triple>`] 将一组自定义标志传递给编译器。
该值可以是字符串数组或空格分隔的字符串。

有关不同版本的更多详细信息，请参阅 [`build.rustdocflags`](#buildrustdocflags)
特定额外标志的方法。

#### `target.<cfg>.rustdocflags`

这与 [target rustdocflags](#targettriplerustdocflags) 类似，但是
使用 [`cfg()` 表达式]。如果有多个 `<cfg>` 和 [`<triple>`] 条目
匹配当前目标，标志连接在一起。

#### `target.<triple>.<links>`

links 子表提供了一种[覆盖构建脚本]的方法。什么时候
指定后，给定“links”库的构建脚本将不会
运行，并且将使用给定的值。

```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = "-L /some/path"
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"
```

### `[term]`

`[term]` 表控制终端输出和交互。

#### `term.quiet`
* Type: boolean
* Default: false
* Environment: `CARGO_TERM_QUIET`

控制 Cargo 是否显示日志消息。

指定“--quiet”标志将覆盖并强制安静输出。
指定“--verbose”标志将覆盖并禁用安静输出。

#### `term.verbose`
* Type: boolean
* Default: false
* Environment: `CARGO_TERM_VERBOSE`

控制 Cargo 是否显示额外详细的消息。

指定“--quiet”标志将覆盖并禁用详细输出。
指定“--verbose”标志将覆盖并强制详细输出。

#### `term.color`
* Type: string
* Default: `"auto"`
* Environment: `CARGO_TERM_COLOR`

控制终端中是否使用彩色输出。可能的值：

* `auto`（默认）：自动检测颜色支持是否可用
  终端。
* `always`：始终显示颜色。
* `never`：从不显示颜色。

可以使用“--color”命令行选项覆盖。

#### `term.hyperlinks`
* Type: bool
* Default: auto-detect
* Environment: `CARGO_TERM_HYPERLINKS`

控制是否在终端中使用超链接。

#### `term.unicode`
* Type: bool
* Default: auto-detect
* Environment: `CARGO_TERM_UNICODE`

控制是否可以使用非 ASCII unicode 字符呈现输出。

#### `term.progress.when`
* Type: string
* Default: `"auto"`
* Environment: `CARGO_TERM_PROGRESS_WHEN`

控制是否在终端中显示进度条。可能的值：

* `auto` (默认): 智能猜测是否显示进度条。
* `always`: 始终显示进度条。
* `never`: 从不显示进度条。

#### `term.progress.width`
* Type: integer
* Default: none
* Environment: `CARGO_TERM_PROGRESS_WIDTH`

设置进度条的宽度。

#### `term.progress.term-integration`
* Type: bool
* Default: auto-detect
* Environment: `CARGO_TERM_PROGRESS_TERM_INTEGRATION`

向终端模拟器报告进度，以便在任务栏等位置显示。

[`cargo bench`]: ../commands/cargo-bench.md
[`cargo login`]: ../commands/cargo-login.md
[`cargo logout`]: ../commands/cargo-logout.md
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo new`]: ../commands/cargo-new.md
[`cargo publish`]: ../commands/cargo-publish.md
[`cargo run`]: ../commands/cargo-run.md
[`cargo rustc`]: ../commands/cargo-rustc.md
[`cargo test`]: ../commands/cargo-test.md
[`cargo rustdoc`]: ../commands/cargo-rustdoc.md
[`cargo install`]: ../commands/cargo-install.md
[env]: environment-variables.md
[`cfg()` expression]: ../../reference/conditional-compilation.html
[build scripts]: build-scripts.md
[`-C linker`]: ../../rustc/codegen-options/index.md#linker
[override a build script]: build-scripts.md#overriding-build-scripts
[toml]: https://toml.io/
[incremental compilation]: profiles.md#incremental
[program path with args]: #executable-paths-with-arguments
[libcurl format]: https://everything.curl.dev/transfers/conn/proxies#proxy-types
[source replacement]: source-replacement.md
[revision]: https://git-scm.com/docs/gitrevisions
[registries]: registries.md
[`cargo:token`]: registry-authentication.md#cargotoken
[crates.io]: https://crates.io/
[target triple]: ../appendix/glossary.md#target '"target" (glossary)'
[`<triple>`]: ../appendix/glossary.md#target '"target" (glossary)'
