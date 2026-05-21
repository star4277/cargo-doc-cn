# Lints

注意：[Cargo 的 lint 系统目前不稳定](unstable.md#lintscargo)，只能在 nightly toolchain 上使用。

| Group                | 说明                                                           | 默认级别 |
|----------------------|----------------------------------------------------------------|----------|
| `cargo::complexity`  | 逻辑本身简单，但实现方式过度复杂                               | warn     |
| `cargo::correctness` | 明显错误或无意义的代码                                         | deny     |
| `cargo::nursery`     | 仍在开发中的新 lint                                            | allow    |
| `cargo::pedantic`    | 较严格或偶尔会有误报的 lint                                    | allow    |
| `cargo::perf`        | 可以改写为更高性能的代码                                       | warn     |
| `cargo::restriction` | 限制某些 Cargo 用法的 lint                                     | allow    |
| `cargo::style`       | 不够惯用（idiomatic）的代码风格问题                            | warn     |
| `cargo::suspicious`  | 高概率错误或无意义的代码                                       | warn     |

## 默认允许（Allowed-by-default）

这些 lint 默认级别都为 `allow`。
- [`implicit_minimum_version_req`](#implicit_minimum_version_req)
- [`non_kebab_case_features`](#non_kebab_case_features)
- [`non_kebab_case_packages`](#non_kebab_case_packages)
- [`non_snake_case_features`](#non_snake_case_features)
- [`non_snake_case_packages`](#non_snake_case_packages)

## 默认警告（Warn-by-default）

这些 lint 默认级别都为 `warn`。
- [`blanket_hint_mostly_unused`](#blanket_hint_mostly_unused)
- [`missing_lints_inheritance`](#missing_lints_inheritance)
- [`non_kebab_case_bins`](#non_kebab_case_bins)
- [`redundant_homepage`](#redundant_homepage)
- [`redundant_readme`](#redundant_readme)
- [`unknown_lints`](#unknown_lints)
- [`unused_dependencies`](#unused_dependencies)
- [`unused_workspace_dependencies`](#unused_workspace_dependencies)
- [`unused_workspace_package_fields`](#unused_workspace_package_fields)

## 默认拒绝（Deny-by-default）

这些 lint 默认级别都为 `deny`。
- [`text_direction_codepoint_in_comment`](#text_direction_codepoint_in_comment)
- [`text_direction_codepoint_in_literal`](#text_direction_codepoint_in_literal)

## `blanket_hint_mostly_unused`
Group: `suspicious`

Level: `warn`

MSRV: `1.79.0`

### 作用
检查是否把 `hint-mostly-unused` 应用于“所有依赖”。

### 为什么不好
`hint-mostly-unused` 表示：某个 crate 的大部分 API 对依赖它的代码都不会被使用；
这个提示会尝试减少未使用项的编译成本，从而加快构建。
如果对不符合该条件的 crate 误用，反而会拖慢构建。
因此它应只用于符合条件的依赖，
全局应用几乎总是误用，并且通常会导致构建变慢。

### 示例
```toml
[profile.dev.package."*"]
hint-mostly-unused = true
```

更合理的写法：
```toml
[profile.dev.package.huge-mostly-unused-dependency]
hint-mostly-unused = true
```

## `implicit_minimum_version_req`
Group: `pedantic`

Level: `allow`

### 作用

检查依赖版本要求是否未显式给出完整的 `major.minor.patch`，
例如 `serde = "1"` 或 `serde = "1.0"`。

该 lint 目前只针对 caret 版本要求
（即[默认版本要求](specifying-dependencies.md#default-requirements)）。

### 为什么不好

未写完整版本时，实际最低支持版本容易被误解。
例如 `serde = "1"` 的隐式下界是 `1.0.0`。
如果你的代码实际上需要 `1.0.219` 才有的能力，
那 `1.0.0` 会给出错误的兼容性印象。

写出完整版本有助于：

- 准确记录最低版本要求
- 更好配合 `-Z minimal-versions`
- 给使用方更清晰的依赖约束

### 缺点

即使写了完整版本，如果没有经过验证，最低下界仍可能不正确。
该 lint 只能让版本下界“显式化”，不能保证其正确性。

### 示例

```toml
[dependencies]
serde = "1"
```

建议改为完整具体版本：

```toml
[dependencies]
serde = "1.0.219"
```

## `missing_lints_inheritance`
Group: `suspicious`

Level: `warn`

MSRV: `1.79.0`

### 作用

检查在存在 `workspace.lints` 时，某个 package 是否缺少 `lints` 表。

### 为什么不好

很多人会误以为 `workspace.lints` 会被隐式继承，但实际上不会。

### 缺点

### 示例

```toml
[workspace.lints.cargo]
```

建议写成：

```toml
[workspace.lints.cargo]

[lints]
workspace = true
```

## `non_kebab_case_bins`
Group: `style`

Level: `warn`

MSRV: `1.79.0`

### 作用

检测二进制名称（显式与隐式）是否不是 kebab-case。

### 为什么不好

命令行工具通常约定使用 kebab-case 的二进制名。

### 缺点

改动二进制名可能会影响现有用户。

有些二进制需要遵循外部规范，命名风格可能不是 kebab-case。

GUI 应用可能更希望采用面向用户的命名方式，例如 “Title Case” 或 “Sentence case”。

### 示例

```toml
[[bin]]
name = "foo_bar"
```

建议写成：

```toml
[[bin]]
name = "foo-bar"
```

## `non_kebab_case_features`
Group: `restriction`

Level: `allow`

### 作用

检测不是 kebab-case 的 feature 名称。

### 为什么不好

在同一个 workspace 中混用多种命名风格容易造成困惑。

### 缺点

用户通常会期望“与某依赖紧耦合的 feature”与依赖名称一致。

### 示例

```toml
[features]
foo_bar = []
```

建议写成：

```toml
[features]
foo-bar = []
```

## `non_kebab_case_packages`
Group: `restriction`

Level: `allow`

### 作用

检测不是 kebab-case 的 package 名称。

### 为什么不好

在同一个 workspace 中混用多种命名风格容易造成困惑。

### 缺点

用户需要在脑中把 package 名称映射成 Rust 命名空间风格。

### 示例

```toml
[package]
name = "foo_bar"
```

建议写成：

```toml
[package]
name = "foo-bar"
```

## `non_snake_case_features`
Group: `restriction`

Level: `allow`

### 作用

检测不是 snake_case 的 feature 名称。

### 为什么不好

在同一个 workspace 中混用多种命名风格容易造成困惑。

### 缺点

用户通常会期望“与某依赖紧耦合的 feature”与依赖名称一致。

### 示例

```toml
[features]
foo-bar = []
```

建议写成：

```toml
[features]
foo_bar = []
```

## `non_snake_case_packages`
Group: `restriction`

Level: `allow`

### 作用

检测不是 snake_case 的 package 名称。

### 为什么不好

在同一个 workspace 中混用多种命名风格容易造成困惑。

### 缺点

用户需要在脑中把 package 名称映射成 Rust 命名空间风格。

### 示例

```toml
[package]
name = "foo_bar"
```

建议写成：

```toml
[package]
name = "foo-bar"
```

## `redundant_homepage`
Group: `style`

Level: `warn`

MSRV: `1.79.0`

### 作用

检查 `package.homepage` 的值是否已经被其他字段覆盖。

另见 [`package.homepage` 参考文档](manifest.md#the-homepage-field)。

### 为什么不好

包浏览器在渲染每个链接时，重复链接会增加视觉噪音。

### 缺点

### 示例

```toml
[package]
name = "foo"
homepage = "https://github.com/rust-lang/cargo/"
repository = "https://github.com/rust-lang/cargo/"
```

建议写成：

```toml
[package]
name = "foo"
repository = "https://github.com/rust-lang/cargo/"
```

## `redundant_readme`
Group: `style`

Level: `warn`

MSRV: `1.79.0`

### 作用

检查可自动推断的 `package.readme` 字段。

另见 [`package.readme` 参考文档](manifest.md#the-readme-field)。

### 为什么不好

会增加样板配置。

### 缺点

如果文件命名不规范，用户可能不容易意识到问题。

### 示例

```toml
[package]
name = "foo"
readme = "README.md"
```

建议写成：

```toml
[package]
name = "foo"
```

## `text_direction_codepoint_in_comment`
Group: `correctness`

Level: `deny`

MSRV: `1.79.0`

### 作用
检测 manifest 注释中的 Unicode 控制码点：
它们会改变屏幕上的文本视觉顺序，
并且与内存中的实际字符顺序不一致。

### 为什么不好
Unicode 支持文本从右向左书写时的视觉流控制，
但精心构造的注释可能让“实际会被编译的代码”看起来像注释内容，
具体效果取决于阅读代码的软件。
为避免潜在问题或混淆（如 CVE-2021-42574），
默认拒绝使用此类码点。

## `text_direction_codepoint_in_literal`
Group: `correctness`

Level: `deny`

MSRV: `1.79.0`

### 作用
检测 manifest 字面量中的 Unicode 控制码点：
它们会改变屏幕上的文本视觉顺序，
并且与内存中的实际字符顺序不一致。

### 为什么不好
Unicode 支持文本从右向左书写时的视觉流控制，
但精心构造的字面量可能让“实际会被编译的代码”看起来像字面量的一部分，
具体效果取决于阅读代码的软件。
为避免潜在问题或混淆（如 CVE-2021-42574），
默认拒绝使用此类码点。

## `unknown_lints`
Group: `suspicious`

Level: `warn`

MSRV: `1.79.0`

### 作用
检查 `[lints.cargo]` 表中未知的 lint 名称。

### 为什么不好
- lint 名可能拼写错误，导致行为与预期不符且难以排查。
- 未来如果 `cargo` 引入同名 lint，这个未知项可能变成错误。

### 示例
```toml
[lints.cargo]
this-lint-does-not-exist = "warn"
```

## `unused_dependencies`
Group: `style`

Level: `warn`

MSRV: `1.79.0`

### 作用

检查是否存在未被任何 cargo target 使用的依赖。

### 为什么不好

会拖慢编译时间。

### 缺点

该 lint 只会在特定条件下触发：
因为不同依赖表对应不同 cargo target，必须构建它们才能判断依赖是否未使用。
当前仅检查被选中的 package，不会像多数 lint 那样覆盖所有 `path` 依赖。
而检查哪些依赖表，取决于 cargo target 选择参数，与“选择了哪些 package”并不完全一致。
由于无法一次选择所有会用到 `[dev-dependencies]` 的 cargo target，
因此 `[dev-dependencies]` 目前不会被检查。

示例：
- `cargo check` 会检查 `[build-dependencies]` 和 `[dependencies]`
- `cargo check --all-targets` 仍只检查 `[build-dependencies]` 与 `[dependencies]`，不会检查 `[dev-dependencies]`
- `cargo check --bin foo` 即便 `foo` 是唯一 bin，也不会检查 `[dependencies]`，但会检查 `[build-dependencies]`
- `cargo check -p foo` 不会检查 `path` 依赖 `bar` 的依赖表，即便 `bar` 只有一个 `[lib]`

当你依赖某个传递依赖仅用于激活 feature 时，可能出现误报。

如果是为了在 `Cargo.toml` 固定某个传递依赖版本而产生误报，
可把该依赖移到 `target."cfg(false)".dependencies` 表。

### 示例

```toml
[package]
name = "foo"

[dependencies]
unused = "1"
```

建议写成：

```toml
[package]
name = "foo"
```

## `unused_workspace_dependencies`
Group: `suspicious`

Level: `warn`

MSRV: `1.79.0`

### 作用
检查 `[workspace.dependencies]` 中是否存在未被继承的条目。

### 为什么不好
会给人造成这些依赖已被使用的错误印象。

### 示例
```toml
[workspace.dependencies]
regex = "1"

[dependencies]
```

## `unused_workspace_package_fields`
Group: `suspicious`

Level: `warn`

MSRV: `1.79.0`

### 作用
检查 `[workspace.package]` 中是否存在未被继承的字段。

### 为什么不好
会给人造成这些字段已被使用的错误印象。

### 示例
```toml
[workspace.package]
edition = "2024"

[package]
name = "foo"
```
