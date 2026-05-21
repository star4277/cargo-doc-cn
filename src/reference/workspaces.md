# Workspaces

*workspace* 是由一个或多个 package 组成的集合，这些 package 称为
*workspace members*，并会一起管理。

workspace 的关键点：

* 可跨所有成员执行通用命令，例如 `cargo check --workspace`。
* 所有 package 共享一个 [`Cargo.lock`]，位于 *workspace root*。
* 所有 package 共享一个[输出目录]，默认是 *workspace root* 下名为 `target` 的目录。
* 可共享 package 元数据，例如通过 [`workspace.package`](#the-package-table)。
* `Cargo.toml` 中的 [`[patch]`][patch]、[`[replace]`][replace]、[`[profile.*]`][profiles]
  仅在 *root* manifest 中生效，成员 crate 的 manifest 里会被忽略。

workspace 根 `Cargo.toml` 支持以下段：

* [`[workspace]`](#the-workspace-section) --- 定义 workspace。
  * [`resolver`](resolver.md#resolver-versions) --- 设置使用的依赖解析器。
  * [`members`](#the-members-and-exclude-fields) --- 包含到 workspace 的 package。
  * [`exclude`](#the-members-and-exclude-fields) --- 从 workspace 排除的 package。
  * [`default-members`](#the-default-members-field) --- 未显式选定 package 时默认操作的 package。
  * [`package`](#the-package-table) --- 可被 package 继承的键。
  * [`dependencies`](#the-dependencies-table) --- 可被 package 依赖继承的键。
  * [`lints`](#the-lints-table) --- 可被 package lint 继承的键。
  * [`metadata`](#the-metadata-table) --- 供外部工具使用的附加配置。
* [`[patch]`](overriding-dependencies.md#the-patch-section) --- 覆盖依赖。
* [`[replace]`](overriding-dependencies.md#the-replace-section) --- 覆盖依赖（已弃用）。
* [`[profile]`](profiles.md) --- 编译器设置与优化。

## `[workspace]` 段

要创建 workspace，可在 `Cargo.toml` 添加 `[workspace]` 表：
```toml
[workspace]
# ...
```

workspace 至少需要一个成员：
要么有根 package，要么是虚拟 manifest。

### Root package

若在已定义 `[package]` 的 `Cargo.toml` 中添加
[`[workspace]` 段](#the-workspace-section)，
该 package 就是 workspace 的 *root package*。
*workspace root* 是 workspace `Cargo.toml` 所在目录。

```toml
[workspace]

[package]
name = "hello_world" # package 名称
version = "0.1.0"    # 当前版本，遵循 semver
```

### Virtual workspace

也可以创建仅含 `[workspace]` 而不含 [`[package]` 段][package] 的 `Cargo.toml`。
这称为 *virtual manifest*。
当没有“主 package”，或希望将所有 package 放在独立目录中时，这通常很有用。

```toml
# [PROJECT_DIR]/Cargo.toml
[workspace]
members = ["hello_world"]
resolver = "3"
```

```toml
# [PROJECT_DIR]/hello_world/Cargo.toml
[package]
name = "hello_world" # package 名称
version = "0.1.0"    # 当前版本，遵循 semver
edition = "2024"     # edition，不会影响 workspace 使用的 resolver 推断
```

在“无根 package”的 workspace 中：

- 必须显式设置 [`resolver`](resolver.md#resolver-versions)，
  因为 virtual workspace 没有 [`package.edition`][package-edition] 可用于推断
  [resolver 版本](resolver.md#resolver-versions)。
- 在 workspace root 执行命令时，默认会作用于所有 workspace members，
  见 [`default-members`](#the-default-members-field)。

## `members` 与 `exclude` 字段

`members` 与 `exclude` 用于定义哪些 package 属于 workspace：

```toml
[workspace]
members = ["member1", "path/to/member2", "crates/*"]
exclude = ["crates/foo", "path/to/other"]
```

位于 workspace 目录内的所有 [`path` 依赖]会自动成为成员。
也可通过 `members` 键显式添加成员，
它应为字符串数组，元素是包含 `Cargo.toml` 的目录。

`members` 也支持 [globs] 匹配多路径，
使用常见文件名模式如 `*` 和 `?`。

`exclude` 可阻止某些路径被纳入 workspace。
当某些 path 依赖不希望成为 workspace 成员，
或使用了 glob 后想剔除个别目录时很有用。

在 workspace 子目录中时，Cargo 会自动向上搜索父目录，
寻找带 `[workspace]` 定义的 `Cargo.toml` 以确定所属 workspace。
成员 crate 可通过 manifest 的 [`package.workspace`] 字段
指定 workspace root，以覆盖自动搜索。
当成员不在 workspace root 的子目录下时，手动指定尤为有用。

### Package 选择

在 workspace 中，像 [`cargo build`] 这样的 package 相关命令
可通过 `-p` / `--package` 或 `--workspace` 选择操作目标。
如果都未指定，Cargo 会使用当前工作目录中的 package。
但如果当前目录是 workspace root，则会使用 [`default-members`](#the-default-members-field)。

## `default-members` 字段

`default-members` 用于指定当位于 workspace root 且未使用 package 选择参数时，
默认操作的 [members](#the-members-and-exclude-fields) 路径：

```toml
[workspace]
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]
default-members = ["path/to/member2", "path/to/member3/foo"]
```

> 注意：当存在 [root package](#root-package) 时，
> 只能通过 `--package` 和 `--workspace` 参数来操作它。

若未指定该字段，则默认使用 [root package](#root-package)。
若是 [virtual workspace](#virtual-workspace)，默认使用全部成员
（等价于命令行指定 `--workspace`）。

## `package` 表

`workspace.package` 表用于定义可由 workspace 成员继承的键。
成员 package 可通过在自身中设置 `{key}.workspace = true` 来继承。

支持继承的键：

|                |                 |
|----------------|-----------------|
| `authors`      | `categories`    |
| `description`  | `documentation` |
| `edition`      | `exclude`       |
| `homepage`     | `include`       |
| `keywords`     | `license`       |
| `license-file` | `publish`       |
| `readme`       | `repository`    |
| `rust-version` | `version`       |

- `license-file` 和 `readme` 路径相对于 workspace root
- `include` 和 `exclude` 路径相对于 package root

示例：
```toml
# [PROJECT_DIR]/Cargo.toml
[workspace]
members = ["bar"]

[workspace.package]
version = "1.2.3"
authors = ["Nice Folks"]
description = "A short description of my package"
documentation = "https://example.com/bar"
```

```toml
# [PROJECT_DIR]/bar/Cargo.toml
[package]
name = "bar"
version.workspace = true
authors.workspace = true
description.workspace = true
documentation.workspace = true
```

> **MSRV:** 需要 1.64+

## `dependencies` 表

`workspace.dependencies` 表用于定义可由 workspace 成员继承的依赖。

声明 workspace 依赖与[普通 package 依赖][specifying-dependencies]类似，但有两点不同：
- 该表中的依赖不能声明为 `optional`
- 该表声明的 [`features`][features] 会与 `[dependencies]` 中的 `features` 叠加

随后你可以[把 workspace 依赖继承为 package 依赖][inheriting-a-dependency-from-a-workspace]。

示例：
```toml
# [PROJECT_DIR]/Cargo.toml
[workspace]
members = ["bar"]

[workspace.dependencies]
cc = "1.0.73"
rand = "0.8.5"
regex = { version = "1.6.0", default-features = false, features = ["std"] }
```

```toml
# [PROJECT_DIR]/bar/Cargo.toml
[package]
name = "bar"
version = "0.2.0"

[dependencies]
regex = { workspace = true, features = ["unicode"] }

[build-dependencies]
cc.workspace = true

[dev-dependencies]
rand.workspace = true
```

> **MSRV:** 需要 1.64+

## `lints` 表

`workspace.lints` 表用于定义可由 workspace 成员继承的 lint 配置。

workspace lint 配置方式与[package lints](manifest.md#the-lints-section)类似。

示例：

```toml
# [PROJECT_DIR]/Cargo.toml
[workspace]
members = ["crates/*"]

[workspace.lints.rust]
unsafe_code = "forbid"
```

```toml
# [PROJECT_DIR]/crates/bar/Cargo.toml
[package]
name = "bar"
version = "0.1.0"

[lints]
workspace = true
```

> **MSRV:** 自 1.74 起生效

## `metadata` 表

`workspace.metadata` 表会被 Cargo 忽略，也不会触发警告。
该段可供外部工具在 `Cargo.toml` 中存储 workspace 配置，例如：

```toml
[workspace]
members = ["member1", "member2"]

[workspace.metadata.webcontents]
root = "path/to/webproject"
tool = ["npm", "run", "build"]
# ...
```

在 package 级别也有对应表：[`package.metadata`][package-metadata]。
虽然 Cargo 不规定这两者内容格式，
但建议外部工具按一致方式使用它们：
例如当 `package.metadata` 缺少数据时，
可回退读取 `workspace.metadata`（若这符合该工具语义）。

[package]: manifest.md#the-package-section
[`Cargo.lock`]: ../guide/cargo-toml-vs-cargo-lock.md
[package-metadata]: manifest.md#the-metadata-table
[package-edition]: manifest.md#the-edition-field
[output directory]: build-cache.md
[patch]: overriding-dependencies.md#the-patch-section
[replace]: overriding-dependencies.md#the-replace-section
[profiles]: profiles.md
[`path` dependencies]: specifying-dependencies.md#specifying-path-dependencies
[`package.workspace`]: manifest.md#the-workspace-field
[globs]: https://docs.rs/glob/0.3.0/glob/struct.Pattern.html
[`cargo build`]: ../commands/cargo-build.md
[specifying-dependencies]: specifying-dependencies.md
[features]: features.md
[inheriting-a-dependency-from-a-workspace]: specifying-dependencies.md#inheriting-a-dependency-from-a-workspace
