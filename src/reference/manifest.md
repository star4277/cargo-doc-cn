# Manifest 格式

每个 package 的 `Cargo.toml` 文件称为该 package 的 *manifest*。
它使用 [TOML] 格式编写，包含编译该 package 所需的元数据。
关于 Cargo 如何定位 manifest，可参考 `cargo locate-project` 对应章节。

每个 manifest 文件由以下部分组成：

* [`cargo-features`](unstable.md) --- 不稳定、仅 nightly 可用的特性。
* [`[package]`](#the-package-section) --- 定义 package。
  * [`name`](#the-name-field) --- package 名称。
  * [`version`](#the-version-field) --- package 版本。
  * [`authors`](#the-authors-field) --- package 作者。
  * [`edition`](#the-edition-field) --- Rust edition。
  * [`rust-version`](rust-version.md) --- 最低支持 Rust 版本。
  * [`description`](#the-description-field) --- package 描述。
  * [`documentation`](#the-documentation-field) --- 文档 URL。
  * [`readme`](#the-readme-field) --- package README 文件路径。
  * [`homepage`](#the-homepage-field) --- package 主页 URL。
  * [`repository`](#the-repository-field) --- 源码仓库 URL。
  * [`license`](#the-license-and-license-file-fields) --- package 许可证。
  * [`license-file`](#the-license-and-license-file-fields) --- 许可证文本文件路径。
  * [`keywords`](#the-keywords-field) --- package 关键词。
  * [`categories`](#the-categories-field) --- package 分类。
  * [`workspace`](#the-workspace-field) --- package 所属 workspace 路径。
  * [`build`](#the-build-field) --- package 构建脚本路径。
  * [`links`](#the-links-field) --- package 链接到的本地库名称。
  * [`exclude`](#the-exclude-and-include-fields) --- 发布时排除的文件。
  * [`include`](#the-exclude-and-include-fields) --- 发布时包含的文件。
  * [`publish`](#the-publish-field) --- 可用于阻止发布该 package。
  * [`metadata`](#the-metadata-table) --- 外部工具的扩展配置。
  * [`default-run`](#the-default-run-field) --- [`cargo run`] 默认运行的二进制。
  * [`autolib`](cargo-targets.md#target-auto-discovery) --- 禁用 library 自动发现。
  * [`autobins`](cargo-targets.md#target-auto-discovery) --- 禁用 binary 自动发现。
  * [`autoexamples`](cargo-targets.md#target-auto-discovery) --- 禁用 example 自动发现。
  * [`autotests`](cargo-targets.md#target-auto-discovery) --- 禁用 test 自动发现。
  * [`autobenches`](cargo-targets.md#target-auto-discovery) --- 禁用 bench 自动发现。
  * [`resolver`](resolver.md#resolver-versions) --- 设置使用的依赖解析器。
* Target 表：（配置项见 [configuration](cargo-targets.md#configuring-a-target)）
  * [`[lib]`](cargo-targets.md#library) --- 库目标配置。
  * [`[[bin]]`](cargo-targets.md#binaries) --- 二进制目标配置。
  * [`[[example]]`](cargo-targets.md#examples) --- 示例目标配置。
  * [`[[test]]`](cargo-targets.md#tests) --- 测试目标配置。
  * [`[[bench]]`](cargo-targets.md#benchmarks) --- 基准目标配置。
* 依赖表：
  * [`[dependencies]`](specifying-dependencies.md) --- 运行库依赖。
  * [`[dev-dependencies]`](specifying-dependencies.md#development-dependencies) --- 示例/测试/基准依赖。
  * [`[build-dependencies]`](specifying-dependencies.md#build-dependencies) --- 构建脚本依赖。
  * [`[target]`](specifying-dependencies.md#platform-specific-dependencies) --- 平台特定依赖。
* [`[badges]`](#the-badges-section) --- 在注册表显示的徽章。
* [`[features]`](features.md) --- 条件编译 feature。
* [`[lints]`](#the-lints-section) --- 为本 package 配置 lint。
* [`[hints]`](#the-hints-section) --- 为本 package 提供编译提示。
* [`[patch]`](overriding-dependencies.md#the-patch-section) --- 覆盖依赖。
* [`[replace]`](overriding-dependencies.md#the-replace-section) --- 覆盖依赖（已弃用）。
* [`[profile]`](profiles.md) --- 编译器设置与优化。
* [`[workspace]`](workspaces.md) --- workspace 定义。

## `[package]` 段

`Cargo.toml` 的第一部分通常是 `[package]`。

```toml
[package]
name = "hello_world" # the name of the package
version = "0.1.0"    # the current version, obeying semver
```

Cargo 仅强制要求 [`name`](#the-name-field)。
若要发布到注册表，注册表可能要求额外字段。
具体见下文说明，以及[发布章节][publishing]中关于发布到
[crates.io] 的要求。

### `name` 字段

package 名是引用该 package 的标识符。
它用于在其他 package 中声明依赖，也会作为推断出的 lib / bin 目标默认名称。

名称只能包含[字母数字字符][alphanumeric]、`-`、`_`，且不能为空。

注意 [`cargo new`] 与 [`cargo init`] 还会施加一些额外限制，
例如必须是合法 Rust 标识符，且不能是关键字。
[crates.io] 限制更多，例如：

- 仅允许 ASCII 字符。
- 不可使用保留名称。
- 不可使用如 `nul` 这类 Windows 特殊名。
- 长度最多 64 个字符。

[alphanumeric]: ../../std/primitive.char.html#method.is_alphanumeric

### `version` 字段

`version` 字段遵循 [SemVer] 规范。

版本必须包含三个数字部分：主版本、次版本、补丁版本。

可在 `-` 后追加预发布标识，例如 `1.0.0-alpha`。
预发布部分可再用 `.` 分隔多个组件。
其中数字组件按数值比较，其他组件按字典序比较。
例如 `1.0.0-alpha.11` 高于 `1.0.0-alpha.4`。

可在 `+` 后追加元数据部分，例如 `1.0.0+21AF26D3`。
它仅用于信息标注，Cargo 通常会忽略。

Cargo 内建了[语义化版本](https://semver.org/)概念：
只要最左侧非零的 major/minor/patch 分量一致，就被视为[兼容](semver.md)。
Cargo 如何用版本解析依赖，见 [Resolver] 章节。

该字段可选，默认 `0.0.0`。
但发布 package 时必须提供此字段。

> **MSRV:** 在 Rust 1.75 之前，此字段是必填项。

[SemVer]: https://semver.org
[Resolver]: resolver.md
[SemVer compatibility]: semver.md

### `authors` 字段

> **警告**：该字段已弃用。

`authors` 是可选字段，用数组列出被视为 package “作者”的个人或组织。
每个作者条目末尾可选附带尖括号邮箱。

```toml
[package]
# ...
authors = ["Graydon Hoare", "Fnu Lnu <no-reply@rust-lang.org>"]
```

为了向后兼容，该字段仍会出现在 package 元数据中，
并通过 `build.rs` 中的 `CARGO_PKG_AUTHORS` 环境变量暴露。

### `edition` 字段

`edition` 是可选键，用于指定 package 使用的 [Rust Edition]。
在 `[package]` 里设置后，会影响该 package 的全部 target/crate，
包括测试套件、基准、二进制、示例等。

```toml
[package]
# ...
edition = '2024'
```

大多数 manifest 的 `edition` 会由 [`cargo new`] 自动填充为最新稳定 edition。
当前 `cargo new` 默认创建 2024 edition。

若 `Cargo.toml` 未提供 `edition`，
为兼容历史行为会假定为 2015 edition。
但由 [`cargo new`] 创建的 manifest 不会触发该回退，
因为它会显式写入较新的 `edition`。

### `rust-version` 字段

`rust-version` 用于告知 Cargo：
该 package 支持的 Rust toolchain 最低版本。
详见 [Rust version 章节](rust-version.md)。

### `description` 字段

`description` 是 package 的简短说明。
[crates.io] 会在页面显示它。
该字段应为纯文本（不是 Markdown）。

```toml
[package]
# ...
description = "A short description of my package"
```

> **注意**：[crates.io] 要求必须设置 `description`。

### `documentation` 字段

`documentation` 用于指定托管 crate 文档的网站 URL。
若 manifest 中未指定该 URL，当文档已构建并可用时（见 [docs.rs queue]），
[crates.io] 会自动将 crate 链接到对应 [docs.rs] 页面。

```toml
[package]
# ...
documentation = "https://docs.rs/bitflags"
```

[docs.rs queue]: https://docs.rs/releases/queue

### `readme` 字段

`readme` 应为 package 根目录中某文件路径（相对当前 `Cargo.toml`），
用于提供 package 的通用说明。
发布时该文件会上传到注册表，[crates.io] 会将其按 Markdown 渲染到 crate 页面。

```toml
[package]
# ...
readme = "README.md"
```

若该字段未设置，且包根存在 `README.md`、`README.txt` 或 `README`，
会自动使用该文件名。
可通过设为 `false` 关闭此行为。
若设为 `true`，则默认视为 `README.md`。

### `homepage` 字段

`homepage` 应为 package 主页 URL。

```toml
[package]
# ...
homepage = "https://serde.rs"
```

仅当 crate 有独立于源码仓库/API 文档的专门网站时才建议设置。
不要让 `homepage` 与 `documentation` 或 `repository` 重复。

### `repository` 字段

`repository` 应为 package 源码仓库 URL。

```toml
[package]
# ...
repository = "https://github.com/rust-lang/cargo"
```

### `license` 与 `license-file` 字段

`license` 填写该 package 使用的软件许可证名称。
`license-file` 填写许可证文本文件路径（相对当前 `Cargo.toml`）。

[crates.io] 将 `license` 解释为 [SPDX 2.3 许可证表达式][spdx-2.3-license-expressions]。
名称必须来自 [SPDX 许可证列表 3.20][spdx-license-list-3.20]。
更多见 [SPDX site]。

SPDX 许可证表达式支持 `AND`、`OR` 组合多个许可证。[^slash]

```toml
[package]
# ...
license = "MIT OR Apache-2.0"
```

使用 `OR` 表示用户可二选一。
使用 `AND` 表示用户必须同时遵守所有许可证。
`WITH` 表示带有特殊例外条款的许可证。
示例：

* `MIT OR Apache-2.0`
* `LGPL-2.1-only AND MIT AND BSD-2-Clause`
* `GPL-2.0-or-later WITH Bison-exception-2.2`

若使用非标准许可证，可不写 `license`，改写 `license-file`：

```toml
[package]
# ...
license-file = "LICENSE.txt"
```

> **注意**：[crates.io] 要求 `license` 与 `license-file` 至少设置一个。

[^slash]: 旧语法曾允许用 `/` 分隔多个许可证，但已弃用。

### `keywords` 字段

`keywords` 是描述该 package 的字符串数组。
它有助于在注册表中搜索 package；可填写任何能帮助用户发现该 crate 的词。

```toml
[package]
# ...
keywords = ["gamedev", "graphics"]
```

> **注意**：[crates.io] 最多允许 5 个关键词。每个关键词必须是 ASCII 文本，
> 最长 20 个字符，以字母数字字符开头，并且只能包含字母、数字、`_`、`-`、`+`。

### `categories` 字段

`categories` 是该 package 所属分类的字符串数组。

```toml
categories = ["command-line-utilities", "development-tools::cargo-plugins"]
```

> **注意**：[crates.io] 最多允许 5 个分类。
> 每个分类必须精确匹配 <https://crates.io/category_slugs> 中提供的字符串。

### `workspace` 字段

`workspace` 可用于配置该 package 所属 workspace。
若未指定，Cargo 会向上查找文件系统中第一个带 `[workspace]` 的 `Cargo.toml` 并推断。
当成员不在 workspace 根的子目录中时，这个字段很有用。

```toml
[package]
# ...
workspace = "path/to/workspace/root"
```

若当前 manifest 已定义 `[workspace]` 表，则不能再指定该字段。
即一个 crate 不能同时既是 workspace 根 crate（含 `[workspace]`），
又是另一个 workspace 的成员 crate（含 `package.workspace`）。

更多见 [workspaces 章节](workspaces.md)。

### `build` 字段

`build` 指定 package 根目录中的一个文件，作为构建本地代码所用的 [build
script]。详见 [build script 指南][build script]。

[build script]: build-scripts.md

```toml
[package]
# ...
build = "build.rs"
```

默认值是 `"build.rs"`，即从 package 根目录 `build.rs` 加载脚本。
可用 `build = "custom_build_name.rs"` 指向其他文件，
或 `build = false` 禁用构建脚本自动检测。

### `links` 字段

`links` 指定要链接的本地库名称。
详见构建脚本指南中的 [`links`][links] 小节。

[links]: build-scripts.md#the-links-manifest-key

例如，一个链接名为 `git2` 的本地库（Linux 下如 `libgit2.a`）的 crate，
可以这样写：

```toml
[package]
# ...
links = "git2"
```

### `exclude` 与 `include` 字段

`exclude` 与 `include` 可显式指定：
项目打包发布（[published][publishing]）时包含哪些文件，
以及某些变更跟踪场景（见下文）要跟踪哪些文件。
`exclude` 中的模式定义“排除集”；`include` 中的模式定义“显式包含集”。

```toml
[package]
# ...
exclude = ["/ci", "images/", ".*"]
```

```toml
[package]
# ...
include = ["/src", "COPYRIGHT", "/examples", "!/examples/big_example"]
```

> **注意：**可运行 [`cargo package --list`][`cargo package`]
> 验证最终会被打包的文件。

若两者都未指定，默认包含 package 根下所有文件，
但会应用下述默认排除规则。

若未指定 `include`，则以下文件会被排除：

* 若 package 不在 git 仓库中：跳过所有以 `.` 开头的隐藏文件。
* 若 package 在 git 仓库中：跳过仓库 [gitignore] 与全局 git 配置忽略的文件。

若指定了 `include`，则不再应用仓库和全局 git 的 gitignore 规则。

无论是否指定 `exclude`/`include`，以下文件始终会被排除：

* 任意子 package（即任何包含 `Cargo.toml` 的子目录）。
* package 根目录下名为 `target` 的目录。

以下文件始终会被包含：

* package 自身的 `Cargo.toml` 总会被包含，无需在 `include` 中列出。
* 会自动包含一个最小化的 `Cargo.lock`。详见 [`cargo package`]。
* 若指定了 [`license-file`](#the-license-and-license-file-fields)，也总会被包含。

`include` 与 `exclude` 互斥；设置 `include` 会覆盖 `exclude`。
若你需要在 `include` 集合内再排除部分文件，请使用下文的 `!` 操作符。

模式语法采用 [gitignore] 风格。简要规则：

- `foo`：匹配 package 任意位置名为 `foo` 的文件或目录，等价于 `**/foo`。
- `/foo`：仅匹配 package 根目录下名为 `foo` 的文件或目录。
- `foo/`：匹配 package 任意位置名为 `foo` 的*目录*。
- 支持常见 glob：`*`、`?`、`[]`：
  - `*`：匹配除 `/` 外的 0 个或多个字符。
    例如 `*.html` 匹配 package 任意位置扩展名为 `.html` 的文件/目录。
  - `?`：匹配除 `/` 外任意一个字符。
    例如 `foo?` 可匹配 `food`，但不匹配 `foo`。
  - `[]`：匹配字符范围。
    例如 `[ab]` 匹配 `a` 或 `b`；`[a-z]` 匹配 `a` 到 `z`。
- `**/` 前缀：匹配任意目录层级。
  例如 `**/foo/bar` 匹配任意位置中位于 `foo` 目录下的 `bar` 文件/目录。
- `/**` 后缀：匹配目录内全部内容。
  例如 `foo/**` 匹配 `foo` 下所有文件（含子目录）。
- `/**/`：匹配 0 层或多层目录。
  例如 `a/**/b` 可匹配 `a/b`、`a/x/b`、`a/x/y/b` 等。
- `!` 前缀：取反。
  例如模式 `src/*.rs` 配合 `!foo.rs`，表示匹配 `src` 下所有 `.rs`，
  但排除任意名为 `foo.rs` 的文件。

在某些场景下，include/exclude 列表也用于变更跟踪。
对使用 `rustdoc` 构建的 target，它用于决定“哪些文件变化会触发重建”。
若 package 有 [build script] 且未发出任何 `rerun-if-*` 指令，
则会用 include/exclude 列表跟踪这些文件变化，以判断是否重跑构建脚本。

[gitignore]: https://git-scm.com/docs/gitignore

### `publish` 字段

`publish` 可用于控制该 package 允许发布到哪些 registry：
```toml
[package]
# ...
publish = ["some-registry-name"]
```

若想避免误发布到 registry（如 crates.io），例如公司内部私有包，
可以不写 [`version`](#the-version-field) 字段。
若希望显式禁止发布，可设置：
```toml
[package]
# ...
publish = false
```

如果 `publish` 数组只包含一个 registry，且 `cargo publish` 未指定 `--registry`，
则会默认使用该 registry。

### `metadata` 表

Cargo 默认会对 `Cargo.toml` 里未使用的键发出警告，以帮助发现拼写错误等问题。
但 `package.metadata` 表会被 Cargo 完全忽略，不会产生未使用警告。
该区块可供外部工具把配置放进 `Cargo.toml`。例如：

```toml
[package]
name = "..."
# ...

# Metadata used when generating an Android APK, for example.
[package.metadata.android]
package-name = "my-awesome-android-app"
assets = "path/to/static"
```

如何使用该字段取决于具体工具文档。
使用 `package.metadata` 的 Rust 项目示例可见：
- [docs.rs](https://docs.rs/about/metadata)

workspace 级别还有类似表：[`workspace.metadata`][workspace-metadata]。
虽然 Cargo 不规定这两类表的内容格式，但建议外部工具尽量保持一致使用方式，
例如当 `package.metadata` 缺失某项时，可回退读取 `workspace.metadata`
（若这符合该工具语义）。

[workspace-metadata]: workspaces.md#the-metadata-table

### `default-run` 字段

manifest 的 `[package]` 段中，`default-run` 可用于指定 [`cargo run`]
默认选择的二进制。例如同时存在 `src/bin/a.rs` 与 `src/bin/b.rs` 时：

```toml
[package]
default-run = "a"
```

## `[lints]` 段

可通过表项把不同工具的 lint 默认级别覆盖为新级别，例如：
```toml
[lints.rust]
unsafe_code = "forbid"
```

它等价于：
```toml
[lints.rust]
unsafe_code = { level = "forbid", priority = 0 }
```

`level` 对应 `rustc` 的 [lint levels](https://doc.rust-lang.org/rustc/lints/levels.html)：
- `forbid`
- `deny`
- `warn`
- `allow`

`priority` 是有符号整数，用于控制 lint 或 lint 组之间的覆盖优先级：
- 更低（尤其负数）优先级更低，会被更高数字覆盖；
  同时会更早出现在 `rustc` 等工具的命令行参数中。

要判断某个 lint 应写在 `[lints]` 下哪个子表，看 lint 名 `::` 前面的部分。
若没有 `::`，工具名视为 `rust`。
例如 `unsafe_code` 对应 `lints.rust.unsafe_code`，
而 `clippy::enum_glob_use` 对应 `lints.clippy.enum_glob_use`。

例如：
```toml
[lints.rust]
unsafe_code = "forbid"

[lints.clippy]
enum_glob_use = "deny"
```

通常这些配置只影响当前 package 的本地开发。
Cargo 只对当前 package 应用，不会应用到依赖。
对依赖方而言，Cargo 会借助如 [`--cap-lints`](../../rustc/lints/levels.html#capping-lints)
之类机制抑制非 path 依赖的 lint。

> **MSRV:** 自 Rust 1.74 起生效。

## `[hints]` 段

`[hints]` 允许为本 package 提供编译提示。
Cargo 在编译本 package 时默认会遵循这些提示，
但顶层正在构建的 package 可通过 `[profile]` 机制覆盖这些值。
按设计，hint 永远应当是“可被 Cargo 安全忽略”的：
若 Cargo 遇到不认识的 hint，或值不认识，会给出 warning 而非 error。
因此在 crate 中声明 hints 不会影响该 crate 的 MSRV。

某些 hint 可能绑定了不稳定 feature gate，
需要显式开启该 gate 才会应用相应配置；
若未开启，依旧只会 warning，不会 error。

当前没有稳定的 hints。
不稳定 hint 可见 [hint-mostly-unused
documentation](unstable.md#profile-hint-mostly-unused-option)。

> **MSRV:** 自 Rust 1.90 起生效。

## `[badges]` 段

`[badges]` 用于指定 package 发布后可在注册表网站展示的状态徽章。

> 注意：[crates.io] 过去会在网站上为 crate 显示徽章，
> 但该功能已移除。建议把徽章放到 README，
> [crates.io] 会展示 README（见[readme 字段](#the-readme-field)）。

```toml
[badges]
# The `maintenance` table indicates the status of the maintenance of
# the crate. This may be used by a registry, but is currently not
# used by crates.io. See https://github.com/rust-lang/crates.io/issues/2437
# and https://github.com/rust-lang/crates.io/issues/2438 for more details.
#
# The `status` field is required. Available options are:
# - `actively-developed`: New features are being added and bugs are being fixed.
# - `passively-maintained`: There are no plans for new features, but the maintainer intends to
#   respond to issues that get filed.
# - `as-is`: The crate is feature complete, the maintainer does not intend to continue working on
#   it or providing support, but it works for the purposes it was designed for.
# - `experimental`: The author wants to share it with the community but is not intending to meet
#   anyone's particular use case.
# - `looking-for-maintainer`: The current maintainer would like to transfer the crate to someone
#   else.
# - `deprecated`: The maintainer does not recommend using this crate (the description of the crate
#   can describe why, there could be a better solution available or there could be problems with
#   the crate that the author does not want to fix).
# - `none`: Displays no badge on crates.io, since the maintainer has not chosen to specify
#   their intentions, potential crate users will need to investigate on their own.
maintenance = { status = "..." }
```

## 依赖段

关于 `[dependencies]`、`[dev-dependencies]`、`[build-dependencies]`、
以及目标特定 `[target.*.dependencies]` 的说明，
见[依赖声明页面](specifying-dependencies.md)。

## `[profile.*]` 段

`[profile]` 表可用于自定义编译器设置（如优化和调试选项）。
详见 [Profiles 章节](profiles.md)。



[`cargo init`]: ../commands/cargo-init.md
[`cargo new`]: ../commands/cargo-new.md
[`cargo package`]: ../commands/cargo-package.md
[`cargo run`]: ../commands/cargo-run.md
[crates.io]: https://crates.io/
[docs.rs]: https://docs.rs/
[publishing]: publishing.md
[Rust Edition]: ../../edition-guide/index.html
[spdx-2.3-license-expressions]: https://spdx.github.io/spdx-spec/v2.3/SPDX-license-expressions/
[spdx-license-list-3.20]: https://github.com/spdx/license-list-data/tree/v3.20
[SPDX site]: https://spdx.org
[TOML]: https://toml.io/
