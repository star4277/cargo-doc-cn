# 指定依赖

你的 crate 可以依赖来自 [crates.io] 或其他 registry 的库，
也可以依赖 `git` 仓库，或本地文件系统中的子目录。
你还可以临时覆盖依赖的位置，例如为了在本地测试你正在修复的依赖 bug。
此外还可以为不同平台使用不同依赖，
并配置仅开发阶段使用的依赖。
下面逐项说明。

## 从 crates.io 指定依赖

Cargo 默认从 [crates.io] 查找依赖。
此时只需要依赖名和版本字符串。
在 [cargo guide](../guide/index.md) 中，我们曾声明过 `time` 依赖：

```toml
[dependencies]
time = "0.1.12"
```

字符串 `"0.1.12"` 叫做[版本要求](#version-requirement-syntax)。
它指定了[依赖解析](resolver.md)时可选版本范围。
在这里，`"0.1.12"` 表示范围 `>=0.1.12, <0.2.0`。
只要更新后仍在该范围内，就允许升级。
例如执行 `cargo update time`，
若 `0.1.z` 最新是 `0.1.13`，Cargo 会升到 `0.1.13`；
但不会升到 `0.2.0`。

## 版本要求语法

### 默认要求

**默认要求（default requirements）**表示“给定最小版本，并允许更新到 [SemVer] 兼容版本”。
这里的兼容定义是：最左侧非零的 major/minor/patch 分量相同。
这与 [SemVer] 本身不同：SemVer 会把所有 `<1.0.0` 视为不兼容。

`1.2.3` 就是默认要求示例。

```notrust
1.2.3  :=  >=1.2.3, <2.0.0
1.2    :=  >=1.2.0, <2.0.0
1      :=  >=1.0.0, <2.0.0
0.2.3  :=  >=0.2.3, <0.3.0
0.2    :=  >=0.2.0, <0.3.0
0.0.3  :=  >=0.0.3, <0.0.4
0.0    :=  >=0.0.0, <0.1.0
0      :=  >=0.0.0, <1.0.0
```

### 插入号要求

**插入号要求（caret requirements）**是默认版本策略。
该策略允许 [SemVer] 兼容更新。
写法是版本要求前加 `^`。

`^1.2.3` 是 caret 要求示例。

省略 `^` 是 caret 的简写等价语法。
虽然 caret 是默认策略，但建议尽量使用简写。

`log = "^1.2.3"` 与 `log = "1.2.3"` 完全等价。

### 波浪号要求

**波浪号要求（tilde requirements）**指定最小版本并允许有限更新。
如果你给出 major+minor+patch，或只给 major+minor，
则只允许 patch 级变更。
如果只给 major，则允许 minor 与 patch 变更。

`~1.2.3` 是 tilde 要求示例。

```notrust
~1.2.3  := >=1.2.3, <1.3.0
~1.2    := >=1.2.0, <1.3.0
~1      := >=1.0.0, <2.0.0
```

### 通配符要求

**通配符要求（wildcard requirements）**允许通配符位置上的任意版本。

`*`、`1.*`、`1.2.*` 都是通配符示例。

```notrust
*     := >=0.0.0
1.*   := >=1.0.0, <2.0.0
1.2.* := >=1.2.0, <1.3.0
```

> **注意**：[crates.io] 不允许裸 `*` 版本。

### 比较符要求

**比较符要求（comparison requirements）**允许你手工指定版本区间或精确版本。

示例：

```notrust
>= 1.2.0
> 1
< 2
= 1.2.3
```

<span id="multiple-requirements"></span>
### 多个版本要求

如上所示，多个版本要求可用逗号分隔，如 `>= 1.2, < 1.5`。
所有条件必须同时满足。
因此像 `<1.2, ^1.2.2` 这种无交集组合会导致无匹配版本。

### 预发布版本

版本要求默认排除[预发布版本](manifest.md#the-version-field)（如 `1.0.0-alpha`），
除非你显式请求。
例如 `foo` 发布了 `1.0.0-alpha` 时，`foo = "1.0"` *不会* 匹配，
会报错。你必须显式写预发布号，例如 `foo = "1.0.0-alpha"`。
同理，[`cargo install`] 默认也会避开预发布，除非你明确安装它。

Cargo 允许自动升级到“更新的同线预发布版本”。
例如发布了 `1.0.0-beta`，则 `foo = "1.0.0-alpha"` 可升级到 `beta`。
但仅限同一 release 线：
`foo = "1.0.0-alpha"` 不能自动升级到 `foo = "1.0.1-alpha"`
或 `foo = "1.0.1-beta"`。

Cargo 还会把预发布自动升级到 semver 兼容的正式版。
例如 `foo = "1.0.0-alpha"` 可升级到 `foo = "1.0.0"`，
也可升级到 `foo = "1.2.0"`。

注意预发布版本可能不稳定，使用时需谨慎。
有些项目会在预发布之间引入 breaking 变更。
如果你的库本身不是预发布，通常不建议依赖预发布依赖。
更新 `Cargo.lock` 时也需留意，预发布升级可能引发问题。

[`cargo install`]: ../commands/cargo-install.md

### 版本元数据

[版本元数据](manifest.md#the-version-field)（如 `1.0.0+21AF26D3`）
会被忽略，不应用于版本要求。

> **建议：**如果不确定，优先使用默认版本要求运算符。
>
> 少数情况下，某个包若有“公共依赖”（对外 re-export 该依赖，
> 或其公共 API 与该依赖互操作），且它能同时兼容多个“彼此 semver 不兼容”的版本
>（例如仅使用某个在各版本间未变的简单类型，如 `Id`），
> 可能允许用户自行选择公共依赖版本。
> 这时 `">=0.4, <2"` 这类要求可能有价值。
> *但* 使用者很可能遇到错误，并需要手动用 `cargo update`
> 选择“公共依赖”版本，
> 因为 Cargo 在[解析依赖版本](resolver.md)时可能为该公共依赖选出不同版本（见 [#10599]）。
>
> 避免把版本上限限制在“下一个 semver 不兼容版本”之前
>（例如避免 `">=2.0, <2.4"`、`"2.0.*"`、`~2.0`），
> 因为依赖树中其他包可能要求更新版本，导致无法解析（见 [#9029]）。
> 可以评估是否改为通过 [`Cargo.lock`] 控制版本更合适。
>
> 有些场景下这种限制无碍，或收益大于成本，例如：
> - 没有其他人依赖你的包；例如它只是一个 `[[bin]]`
> - 依赖预发布包且希望避免 breaking 变更，此时精确 `"=1.2.3-alpha.3"` 可能合理（见 [#2222]）
> - 库 re-export 一个 proc-macro，而该 proc-macro 会生成调用该库的代码；
>   为确保 proc-macro 不比被 re-export 的库更新、从而生成“当前库版本不存在的 API 调用”，
>   精确 `=1.2.3` 可能合理

[`Cargo.lock`]: ../guide/cargo-toml-vs-cargo-lock.md
[#2222]: https://github.com/rust-lang/cargo/issues/2222
[#9029]: https://github.com/rust-lang/cargo/issues/9029
[#10599]: https://github.com/rust-lang/cargo/issues/10599

## 从其他 registry 指定依赖

若要从非 [crates.io] registry 指定依赖，
可设置 `registry` 键为目标 registry 名：

```toml
[dependencies]
some-crate = { version = "1.0", registry = "my-registry" }
```

其中 `my-registry` 需在 `.cargo/config.toml` 中配置。
更多见 [registries documentation]。

> **注意**：[crates.io] 不允许发布“依赖 [crates.io] 之外代码”的 package。

[registries documentation]: registries.md

## 从 `git` 仓库指定依赖

若依赖位于 `git` 仓库，
最少只需通过 `git` 键指定仓库地址：

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```

Cargo 会拉取该仓库，并遍历文件树，
在仓库任意位置寻找请求 crate 的 `Cargo.toml`。
例如 `regex-lite` 和 `regex-syntax` 都是 `rust-lang/regex` 仓库成员，
无论它们在仓库里哪个子目录，都可只用仓库根 URL 引用：

```toml
regex-lite   = { git = "https://github.com/rust-lang/regex.git" }
regex-syntax = { git = "https://github.com/rust-lang/regex.git" }
```

上面这个“自动遍历”规则不适用于 [`path` dependencies](#specifying-path-dependencies)。

### 提交选择

如果只写仓库 URL（如上），
Cargo 默认使用默认分支最新提交来构建。

你可把 `git` 与 `rev` / `tag` / `branch` 组合，
更精确地指定提交。
例如使用 `next` 分支最新提交：

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git", branch = "next" }
```

凡不属于分支或标签的引用都属于 `rev`。
它可以是提交哈希（如 `rev = "4c59b707"`），
也可以是远端暴露的命名引用（如 `rev = "refs/pull/493/head"`）。

`rev` 可用引用取决于仓库托管平台。
GitHub 会暴露每个 PR 最新提交的引用（如上例）；
其他平台可能有等价机制，但命名不同。

**更多 `git` 依赖示例：**

```toml
# .git suffix can be omitted if the host accepts such URLs - both examples work the same
regex = { git = "https://github.com/rust-lang/regex" }
regex = { git = "https://github.com/rust-lang/regex.git" }

# a commit with a particular tag
regex = { git = "https://github.com/rust-lang/regex.git", tag = "1.10.3" }

# a commit by its SHA1 hash
regex = { git = "https://github.com/rust-lang/regex.git", rev = "0c0990399270277832fbb5b91a1fa118e6f63dba" }

# HEAD commit of PR 493
regex = { git = "https://github.com/rust-lang/regex.git", rev = "refs/pull/493/head" }

# INVALID EXAMPLES

# specifying the commit after # ignores the commit ID and generates a warning
regex = { git = "https://github.com/rust-lang/regex.git#4c59b70" }

# git and path cannot be used at the same time
regex = { git = "https://github.com/rust-lang/regex.git#4c59b70", path = "../regex" }
```

Cargo 会在首次加入 `git` 依赖时，把提交锁定到 `Cargo.lock`。
之后仅在你执行 `cargo update` 时检查更新。

### `version` 键的作用

无论是否同时出现 `git` 或 `path`，
只要出现 `version` 键，就意味着该 package 可从 registry 获取。

`version` 键不会影响 Cargo 从 `git` 依赖中选哪个提交，
但 Cargo 会校验依赖 `Cargo.toml` 中的版本是否与 `version` 兼容，
不兼容则报错。

例如下面会从 Git 获取 `next` 分支 HEAD，
然后检查 crate 版本是否兼容 `version = "1.10.3"`：

```toml
[dependencies]
regex = { version = "1.10.3", git = "https://github.com/rust-lang/regex.git", branch = "next" }
```

`version`、`git`、`path` 被视为依赖解析的不同“位置来源”。
详见下文 [Multiple locations](#multiple-locations)。

> **注意**：[crates.io] 不允许发布依赖 [crates.io] 外部代码的 package
>（`[dev-dependencies]` 会被忽略）。
> `git` 与 `path` 的可发布回退方案见 [Multiple
> locations](#multiple-locations)。

### Git 子模块

克隆 `git` 依赖时，
Cargo 默认会递归拉取其 submodule，
以确保构建所需代码完整。

如果想跳过与构建无关的 submodule 拉取，
可在依赖仓库 `.gitmodules` 中设置
[`submodule.<name>.update = none`][submodule-update]。
这要求你对该仓库有写权限，并会更广泛地禁用 submodule 更新。

[submodule-update]: https://git-scm.com/docs/gitmodules#Documentation/gitmodules.txt-submodulenameupdate

### 访问私有 Git 仓库

私有仓库认证可参考 [Git Authentication](../appendix/git-authentication.md)。

## 指定 path 依赖

随着 [guide](../guide/index.md) 里的 `hello_world` 变大，
你可能希望拆出独立 crate 供他人复用。
Cargo 支持 **path dependencies**，通常用于同一仓库中的子 crate。
先在 `hello_world` 内创建新 crate：

```console
# inside of hello_world/
$ cargo new hello_utils
```

这会创建 `hello_utils` 目录，
其中已有 `Cargo.toml` 和 `src`。
然后在 `hello_world/Cargo.toml` 里把它加为依赖：

```toml
[dependencies]
hello_utils = { path = "hello_utils" }
```

这表示：当前包依赖名为 `hello_utils` 的 crate，
其位置是相对当前 `Cargo.toml` 的 `hello_utils` 目录。

下一次 `cargo build` 会自动构建 `hello_utils` 及其依赖。

### 不支持本地路径遍历

本地路径必须精准指向“依赖的 `Cargo.toml` 所在目录”。
与 `git` 依赖不同，Cargo 不会遍历本地路径。
例如 `regex-lite` 与 `regex-syntax` 是本地克隆的 `rust-lang/regex` 成员时，
必须写完整子路径：

```toml
# git key accepts the repo root URL and Cargo traverses the tree to find the crate
[dependencies]
regex-lite   = { git = "https://github.com/rust-lang/regex.git" }
regex-syntax = { git = "https://github.com/rust-lang/regex.git" }

# path key requires the member name to be included in the local path
[dependencies]
regex-lite   = { path = "../regex/regex-lite" }
regex-syntax = { path = "../regex/regex-syntax" }
```

### 已发布 crate 中的本地路径

只使用 path 指定依赖的 crate 不允许发布到 [crates.io]。

若要发布 `hello_world`，
需先把 `hello_utils` 作为独立 crate 发布到 [crates.io]，
并在 `hello_world` 依赖里写版本：

```toml
[dependencies]
hello_utils = { path = "hello_utils", version = "0.1.0" }
```

`path` 与 `version` 同时使用见下文 [Multiple locations](#multiple-locations)。

> **注意**：[crates.io] 不允许发布依赖 [crates.io] 外部代码的 package，
> 但 [dev-dependencies] 例外。
> `git` 和 `path` 的可发布回退方案见 [Multiple locations](#multiple-locations)。

## 多个位置来源

可以同时指定 registry 版本与 `git` / `path` 位置。
本地开发时使用 `git` 或 `path`（并校验 `version` 与本地副本兼容），
发布到 [crates.io] 等 registry 时使用 registry 版本。
其他组合不允许。示例：

```toml
[dependencies]
# Uses `my-bitflags` when used locally, and uses
# version 1.0 from crates.io when published.
bitflags = { path = "my-bitflags", version = "1.0" }

# Uses the given git repo when used locally, and uses
# version 1.0 from crates.io when published.
smallvec = { git = "https://github.com/servo/rust-smallvec.git", version = "1.0" }

# Note: if a version doesn't match, Cargo will fail to compile!
```

一个常见用途是：
你把一个库拆成同一 workspace 下多个 package。
开发时可用 `path` 指向本地 package，
发布后自动切到 [crates.io] 版本。
这与 [override](overriding-dependencies.md) 类似，
但只作用于这条依赖声明。

## 平台特定依赖

平台特定依赖写法与普通依赖相同，
只是放在 `target` 段下。
通常使用 Rust 风格 [`#[cfg]`
语法](../../reference/conditional-compilation.html)：

```toml
[target.'cfg(windows)'.dependencies]
winhttp = "0.4.0"

[target.'cfg(unix)'.dependencies]
openssl = "1.0.1"

[target.'cfg(target_arch = "x86")'.dependencies]
native-i686 = { path = "native/i686" }

[target.'cfg(target_arch = "x86_64")'.dependencies]
native-x86_64 = { path = "native/x86_64" }
```

和 Rust 一样，这里也支持 `not`、`any`、`all` 组合 `cfg` 条件。

如果你想知道当前平台有哪些 cfg，
可执行 `rustc --print=cfg`。
如果想看其他平台（如 64 位 Windows），
可执行 `rustc --print=cfg --target=x86_64-pc-windows-msvc`。

与 Rust 源码不同，
你不能用 `[target.'cfg(feature = "fancy-feature")'.dependencies]`
基于可选 feature 添加依赖。
应改用 [ `[features]` 段](features.md)：

```toml
[dependencies]
foo = { version = "1.0", optional = true }
bar = { version = "1.0", optional = true }

[features]
fancy-feature = ["foo", "bar"]
```

同理 `cfg(debug_assertions)`、`cfg(test)`、`cfg(proc_macro)` 也不适用。
这些值在这里不会按你预期工作，
会始终是 `rustc --print=cfg` 返回的默认值。
目前没有基于这些配置值添加依赖的方法。

除 `#[cfg]` 语法外，
Cargo 也支持直接写完整 target 三元组：

```toml
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"

[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
```

### 自定义目标规范

若你使用自定义 target 规范（如 `--target foo/bar.json`），
请使用去掉 `.json` 后缀的基础文件名：

```toml
[target.bar.dependencies]
winhttp = "0.4.0"

[target.my-special-i686-platform.dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }
```

> **注意**：自定义 target 规范在 stable channel 不可用。

## 开发依赖

你可以在 `Cargo.toml` 添加 `[dev-dependencies]`，
其格式与 `[dependencies]` 等价：

```toml
[dev-dependencies]
tempdir = "0.3"
```

dev-dependencies 在构建 package 本体时不会使用，
但会用于编译测试、示例、基准。

这些依赖**不会**传递给依赖当前 package 的其他 package。

你也可以写“平台特定 dev 依赖”：
把 target 段里的 `dependencies` 换成 `dev-dependencies`，例如：

```toml
[target.'cfg(unix)'.dev-dependencies]
mio = "0.0.1"
```

> **注意**：发布 package 时，
> 只有显式写了 `version` 的 dev-dependencies 才会包含在发布 crate 中。
> 多数场景下发布时并不需要 dev-dependencies，
> 但有些用户（如发行版打包者）可能会在 crate 内运行测试，
> 因此若可能，给出 `version` 仍有价值。

## 构建依赖

构建脚本可以依赖其他 Cargo crate。
通过 manifest 的 `build-dependencies` 段声明：

```toml
[build-dependencies]
cc = "1.0.3"
```

也可以写“平台特定 build 依赖”：
把 target 段里的 `dependencies` 换成 `build-dependencies`，例如：

```toml
[target.'cfg(unix)'.build-dependencies]
cc = "1.0.3"
```

这种情况下，只有当 host 平台匹配指定 target 时才会构建该依赖。

构建脚本**不能**访问 `dependencies` 或 `dev-dependencies` 里的依赖。
反过来，build dependencies 也不会自动提供给 package 本体，
除非它们也被写进 `dependencies`。
package 本体和其构建脚本是分别构建的，
两者依赖不必一致。
把“不同用途的依赖”拆开可让 Cargo 模型更简洁。

## 选择 feature

如果你依赖的 package 提供条件 feature，可指定要启用哪些：

```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # do not include the default features, and optionally
                         # cherry-pick individual features
features = ["secure-password", "civet"]
```

更多见 [features
chapter](features.md#dependency-features)。

## 在 `Cargo.toml` 中重命名依赖

在 `Cargo.toml` 的 `[dependencies]` 中，
依赖键名通常与代码里导入的 crate 名一致。
但有些项目希望在代码里使用不同名称，
不受其在 crates.io 上发布名影响。
例如你可能想：

* 避免在 Rust 源码里写 `use foo as bar`。
* 同时依赖同一 crate 的多个版本。
* 依赖来自不同 registry、但同名的 crate。

为此，Cargo 支持在 `[dependencies]` 里使用 `package` 键，
显式指定实际依赖的 package：

```toml
[package]
name = "mypackage"
version = "0.0.1"

[dependencies]
foo = "0.1"
bar = { git = "https://github.com/example/project.git", package = "foo" }
baz = { version = "0.1", registry = "custom", package = "foo" }
```

在这个例子里，Rust 代码中会有三个可用 crate 名：

```rust,ignore
extern crate foo; // crates.io
extern crate bar; // git repository
extern crate baz; // registry `custom`
```

这三个 crate 在各自 `Cargo.toml` 里的 package 名都叫 `foo`。
我们用 `package` 显式告诉 Cargo：
虽然本地叫别名，但实际要的都是 `foo` package。
若不写 `package`，默认使用“依赖键名”作为 package 名。

另外，如果你有如下可选依赖：

```toml
[dependencies]
bar = { version = "0.1", package = 'foo', optional = true }
```

你依赖的是 crates.io 上的 `foo` crate，
但当前 crate 拥有的是 `bar` feature，而不是 `foo` feature。
也就是说，feature 命名跟“依赖键名”走，而不是 `package` 名。

启用传递依赖 feature 也是类似逻辑。
例如可在上面 manifest 再加：

```toml
[features]
log-debug = ['bar/log-debug'] # using 'foo/log-debug' would be an error!
```

## 从 workspace 继承依赖

可通过 workspace 继承依赖：
先在 workspace 的 [`[workspace.dependencies]`][workspace.dependencies] 表里定义依赖，
然后在包内 `[dependencies]` 里写 `workspace = true` 引用它。

除 `workspace` 键外，还可附加这些键：
- [`optional`][optional]：注意 `[workspace.dependencies]` 表本身不允许指定 `optional`。
- [`features`][features]：会与 `[workspace.dependencies]` 中声明的 features 叠加。

除了 `optional` 与 `features`，
继承依赖不能再使用其他依赖键（如 `version`、`default-features`）。

`[dependencies]`、`[dev-dependencies]`、`[build-dependencies]`、
`[target."...".dependencies]` 都支持引用 `[workspace.dependencies]`。

```toml
[package]
name = "bar"
version = "0.2.0"

[dependencies]
regex = { workspace = true, features = ["unicode"] }

[build-dependencies]
cc.workspace = true

[dev-dependencies]
rand = { workspace = true, optional = true }
```


[SemVer]: https://semver.org
[crates.io]: https://crates.io/
[dev-dependencies]: #development-dependencies
[workspace.dependencies]: workspaces.md#the-dependencies-table
[optional]: features.md#optional-dependencies
[features]: features.md

