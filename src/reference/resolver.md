# 依赖解析

Cargo 的核心任务之一，是根据各 package 指定的版本要求，
决定实际应使用哪些依赖版本。
这个过程称为“依赖解析（dependency resolution）”，
由“resolver（解析器）”执行。
解析结果会写入 [`Cargo.lock` file]，把依赖“锁定”为具体版本并在后续保持稳定。
可用 [`cargo tree`] 命令可视化解析结果。

[`Cargo.lock` file]: ../guide/cargo-toml-vs-cargo-lock.md
[dependency specifications]: specifying-dependencies.md
[dependency specification]: specifying-dependencies.md
[`cargo tree`]: ../commands/cargo-tree.md

## 约束与启发式规则

很多情况下并不存在唯一“最佳”的依赖解析结果。
resolver 会在多种约束与启发式规则下，寻找一个普遍可用的解。
理解这些规则如何交互，先要对解析流程有一个粗粒度认识。

下面的伪代码近似描述了 Cargo resolver 的行为：
```rust
pub fn resolve(workspace: &[Package], policy: Policy) -> Option<ResolveGraph> {
    let dep_queue = Queue::new(workspace);
    let resolved = ResolveGraph::new();
    resolve_next(dep_queue, resolved, policy)
}

fn resolve_next(dep_queue: Queue, resolved: ResolveGraph, policy: Policy) -> Option<ResolveGraph> {
    let Some(dep_spec) = policy.pick_next_dep(&mut dep_queue) else {
        // Done
        return Some(resolved);
    };

    if let Some(resolved) = policy.try_unify_version(dep_spec, resolved.clone()) {
        return Some(resolved);
    }

    let dep_versions = dep_spec.lookup_versions()?;
    let mut dep_versions = policy.filter_versions(dep_spec, dep_versions);
    while let Some(dep_version) = policy.pick_next_version(&mut dep_versions) {
        if policy.needs_version_unification(&dep_version, &resolved) {
            continue;
        }

        let mut dep_queue = dep_queue.clone();
        dep_queue.enqueue(&dep_version.dependencies);
        let mut resolved = resolved.clone();
        resolved.register(dep_version);
        if let Some(resolved) = resolve_next(dep_queue, resolved, policy) {
            return Some(resolved);
        }
    }

    // No valid solution found, backtrack and `pick_next_version`
    None
}
```

关键步骤：
- 遍历依赖（`pick_next_dep`）：
  遍历顺序会影响同一依赖的相关版本要求如何收敛（见版本统一），
  也会影响回溯次数，从而影响解析性能。
- 版本统一（`try_unify_version`、`needs_version_unification`）：
  Cargo 会尽可能复用同一版本，以减少构建时间并让公共依赖类型可在 API 间传递。
  如果本可统一但被[依赖声明][dependency specifications]冲突阻止，
  Cargo 会回溯；若无解，则报错，而不是无条件选择多版本。
  同时，[依赖声明][dependency specification]或 Cargo 本身也可能认定某版本“不理想”，
  从而倾向回溯或报错，而不是使用该版本。
- 版本偏好（`pick_next_version`）：
  Cargo 可能优先尝试某个版本，
  回溯时再退到下一个版本。

### 版本号

通常 Cargo 会优先选择当前可用的最高版本。

例如解析图中有：
```toml
[dependencies]
bitflags = "*"
```
当 `Cargo.lock` 生成时，若 `bitflags` 最高版本是 `1.2.1`，
则会使用 `1.2.1`。

可能的例外见 [Rust version](#rust-version)。

### 版本要求

package 通过[版本要求][version requirements]声明可接受版本并拒绝其他版本。

例如：
```toml
[dependencies]
bitflags = "1.0"  # meaning `>=1.0.0,<2.0.0`
```
当 `Cargo.lock` 生成时，若 `bitflags` 最高版本是 `1.2.1`，
则会选 `1.2.1`，因为它在兼容范围内且最高。
即使 `2.0.0` 已发布，也仍会使用 `1.2.1`，因为 `2.0.0` 不兼容。

[version requirements]: specifying-dependencies.md#version-requirement-syntax

### SemVer 兼容性

Cargo 假设 package 遵循 [SemVer]，
并按 [Caret version requirements] 判定“可兼容”时进行版本统一。
若两个兼容版本因版本要求冲突无法统一，Cargo 会报错。

关于“什么算兼容变更”，见 [SemVer Compatibility] 章节。

示例：

以下两个 package 对 `bitflags` 的依赖会被统一，
因为任意被选版本都互相兼容：
```toml
# Package A
[dependencies]
bitflags = "1.0"  # meaning `>=1.0.0,<2.0.0`

# Package B
[dependencies]
bitflags = "1.1"  # meaning `>=1.1.0,<2.0.0`
```

以下会报错，因为版本要求冲突，无法在“兼容版本范围内”统一为单一版本：
```toml
# Package A
[dependencies]
log = "=0.4.11"

# Package B
[dependencies]
log = "=0.4.8"
```

以下两个 package 的 `rand` 不会统一，
因为各自只能用互不兼容的版本。
最终会解析并构建两个版本（如 0.6.5 与 0.7.3）。
这可能带来风险，详见 [Version-incompatibility hazards]。
```toml
# Package A
[dependencies]
rand = "0.7"  # meaning `>=0.7.0,<0.8.0`

# Package B
[dependencies]
rand = "0.6"  # meaning `>=0.6.0,<0.7.0`
```

通常，下面这两个 package 也不会统一，
因为存在满足要求但互不兼容的版本：
最终会解析并构建两个版本（如 0.6.5 与 0.7.3）。
但其他约束/启发式也可能使其被统一到一个版本（如 0.6.5）。
```toml
# Package A
[dependencies]
rand = ">=0.6,<0.8.0"

# Package B
[dependencies]
rand = "0.6"  # meaning `>=0.6.0,<0.7.0`
```

[SemVer]: https://semver.org/
[SemVer Compatibility]: semver.md
[Caret version requirements]: specifying-dependencies.md#default-requirements
[Version-incompatibility hazards]: #version-incompatibility-hazards

#### 版本不兼容风险

当解析图中出现同一 crate 的多个版本时，
如果这些 crate 暴露的类型在上层交叉使用，就可能出问题。
原因是 Rust 编译器会把它们视为不同类型/项，哪怕名字相同。
库在发布 SemVer 不兼容版本时要格外谨慎（例如 `1.0.0` 已广泛使用后发布 `2.0.0`），
尤其是生态中使用广泛的库。

“[semver trick]”是一个常见规避手法：
在发布 breaking 版本时，保持对旧版本的兼容体验。
链接文档有完整说明。简而言之：
发布新大版本后，再发布旧大版本的一个补丁版本，
由它重新导出新版本中的类型。

这类不兼容通常体现为编译错误，
但有时会在运行时才表现为异常行为。
例如公共库 `foo` 同时以 `1.0.0` 和 `2.0.0` 出现在解析图中。
若对“由依赖 `foo` 1.0.0 的库创建的对象”调用 [`downcast_ref`]，
而调用侧按 `foo` 2.0.0 的类型去 downcast，
运行时 downcast 会失败。

因此若你的图中确实有多版本并存，
务必确认使用方式正确，尤其当不同版本类型可能交汇时。
可用 [`cargo tree -d`][`cargo tree`] 定位重复版本及其来源。
同样，若你为热门库发布 SemVer 不兼容版本，也要评估其生态影响。

[semver trick]: https://github.com/dtolnay/semver-trick
[`downcast_ref`]: ../../std/any/trait.Any.html#method.downcast_ref

### 锁文件

在使用 [`Cargo.lock` file] 时，Cargo 会优先采用其中记录的版本。
这用于平衡“可复现构建”与“manifest 变化带来的调整”。

例如：
```toml
[dependencies]
bitflags = "*"
```
当 `Cargo.lock` 生成时，若 `bitflags` 最高版本是 `1.2.1`，
则会使用并记录 `1.2.1`。

之后即便发布了 `bitflags` `1.3.5`，
下次解析仍会继续用 `1.2.1`，因为它已在 `Cargo.lock` 中。

若随后把 package 改成：
```toml
[dependencies]
bitflags = "1.3.0"
```
此时 `1.2.1` 不满足新要求，
`Cargo.lock` 中该条目会被忽略，
最终会选并记录 `1.3.5`。

### Rust 版本

为了支持“按最低 [Rust version] 开发软件”，
resolver 可把“依赖版本对 Rust 版本的兼容性”纳入考虑。
该行为由配置项 [`resolver.incompatible-rust-versions`] 控制。

在 `fallback` 设置下，resolver 会优先选择 Rust 版本
小于等于你当前 Rust 版本的依赖。
例如你使用 Rust 1.85 开发：
```toml
[package]
name = "my-cli"
rust-version = "1.62"

[dependencies]
clap = "4.0"  # resolves to 4.0.32
```
resolver 会选 `4.0.32`，因为它的 Rust 版本是 1.60.0。
- 不选 4.0.0，因为虽然同为 Rust 1.60.0，但它是[更低版本号](#version-numbers)。
- 不选 4.5.20，因为它对 `my-cli` 的 Rust 1.62 不兼容，
  即便它[版本更高](#version-numbers)，且对你本机 1.85 toolchain 兼容（Rust 1.74.0）。

如果某个版本要求下“没有任何兼容你 Rust 版本的依赖版本”，
resolver 不会报错，而是会退而选一个版本（即便可能次优）。
例如把 `clap` 改为：
```toml
[package]
name = "my-cli"
rust-version = "1.62"

[dependencies]
clap = "4.2"  # resolves to 4.5.20
```
在该[版本要求](#version-requirements)下，
不存在与 Rust 1.62 兼容的 `clap` 版本。
resolver 会因此选择不兼容版本，比如 Rust 1.74 的 4.5.20。

resolver 在为某 package 选择依赖版本时，
并不知道未来哪些 workspace 成员会经由传递依赖使用该版本，
因此无法仅针对“与该依赖真正相关的 Rust 版本”做最优判断。
当 workspace 成员 Rust 版本不一致时，
resolver 会使用启发式找一个“足够好”的解。
即便某些包没声明 Rust 版本，也受此影响。

当 workspace 成员 Rust 版本不同，resolver 可能会选得偏低。
例如：
```toml
[package]
name = "a"
rust-version = "1.62"

[package]
name = "b"

[dependencies]
clap = "4.2"  # resolves to 4.5.20
```
虽然 `b` 未声明 Rust 版本，理论上可用更高版本如 4.5.20，
但会因为 `a` 的 Rust 1.62 而选中 4.0.32。

也可能选得偏高。
例如：
```toml
[package]
name = "a"
rust-version = "1.62"

[dependencies]
clap = "4.2"  # resolves to 4.5.20

[package]
name = "b"

[dependencies]
clap = "4.5"  # resolves to 4.5.20
```
虽然每个 package 单独看都有满足自身 Rust 版本的 `clap` 要求，
但因为[版本统一](#version-numbers)，
resolver 必须选一个两边都共用的版本，最终会选类似 4.5.20。

[Rust version]: rust-version.md
[`resolver.incompatible-rust-versions`]: config.md#resolverincompatible-rust-versions

### 特性

为了生成 `Cargo.lock`，resolver 会按“启用所有 [workspace] 成员的所有 [features]”来构建依赖图。
这可确保在通过[`--features` 参数](features.md#command-line-feature-options)
增减 feature 时，可选依赖仍能与整张依赖图正确解析。
随后 resolver 会再运行一次，根据命令行实际选择的 feature，
确定*编译*时真正使用的 feature。

依赖的 feature 会按“并集”解析。
例如一个 package 依赖 [`im`] 并启用其 [`serde` dependency]，
另一个 package 依赖 `im` 并启用其 [`rayon` dependency]，
那么 `im` 会在两个 feature 都启用的情况下构建，
`serde` 与 `rayon` 也会被纳入解析图。
若没有 package 启用这些 feature，对应可选依赖会被忽略，不影响解析。

当一次构建 workspace 内多个 package（如 `--workspace` 或多个 `-p`）时，
这些 package 的依赖 feature 会被统一。
若你希望不同 workspace 成员避免 feature 统一，
需要拆成多次独立 `cargo` 调用。

resolver 会跳过“缺少必需 feature”的包版本。
例如某 package 依赖 [`regex`] `^1` 且要求 [`perf` feature]，
那么它可选的最旧版本是 `1.3.0`，
因为更早版本没有 `perf` feature。
类似地，如果新版本删除某 feature，
要求该 feature 的 package 会被迫停在旧版本。
不建议在 SemVer 兼容发布中删除 feature。
还需注意：可选依赖也会隐式定义同名 feature，
因此删除可选依赖或把它改为非可选都可能出问题，
见 [removing an optional dependency]。

[`im`]: https://crates.io/crates/im
[`perf` feature]: https://github.com/rust-lang/regex/blob/1.3.0/Cargo.toml#L56
[`rayon` dependency]: https://github.com/bodil/im-rs/blob/v15.0.0/Cargo.toml#L47
[`regex`]: https://crates.io/crates/regex
[`serde` dependency]: https://github.com/bodil/im-rs/blob/v15.0.0/Cargo.toml#L46
[features]: features.md
[removing an optional dependency]: semver.md#cargo-remove-opt-dep
[workspace]: workspaces.md

#### 特性解析器版本 2

当在 `Cargo.toml` 指定 `resolver = "2"`（见下文 [resolver
versions](#resolver-versions)）时，
会使用不同的 feature resolver 与不同的 feature 统一算法。
`"1"` 版会不分场景地统一某 package 的 feature。
`"2"` 版会在以下场景避免统一：

* 若目标特定依赖对应的目标平台当前未构建，
  则其 feature 不会启用。例如：

  ```toml
  [dependencies.common]
  version = "1.0"
  features = ["f1"]

  [target.'cfg(windows)'.dependencies.common]
  version = "1.0"
  features = ["f2"]
  ```

  在非 Windows 平台构建时，`f2` *不会* 启用。

* 若同一依赖既作为普通依赖又作为 [build-dependencies]/proc-macro 依赖，
  则构建依赖路径上启用的 feature 不会与普通依赖统一。例如：

  ```toml
  [dependencies]
  log = "0.4"

  [build-dependencies]
  log = {version = "0.4", features=['std']}
  ```

  构建构建脚本时，`log` 会带 `std` feature 构建；
  构建 package 的 library 时不会启用该 feature。

* 若同一依赖既作为普通依赖又作为 [dev-dependencies]，
  则 dev 路径上启用的 feature 不会统一，
  除非这些 dev-dependencies 当前确实在被构建。例如：

  ```toml
  [dependencies]
  serde = {version = "1.0", default-features = false}

  [dev-dependencies]
  serde = {version = "1.0", features = ["std"]}
  ```

  此例中，library 通常会在不启用 `std` 的情况下链接 `serde`。
  但当以 test/example 方式构建时，会包含 `std` feature。
  例如 `cargo test` 或 `cargo build --all-targets` 会统一这些 feature。
  注意：依赖的 dev-dependencies 始终会被忽略，
  此处仅针对顶层 package 或 workspace 成员。

[build-dependencies]: specifying-dependencies.md#build-dependencies
[dev-dependencies]: specifying-dependencies.md#development-dependencies
[resolver-field]: features.md#resolver-versions

### `links` 字段

[`links` field] 用于确保二进制中同一本地库只链接一份。
resolver 会尝试找到满足“每个 `links` 名只出现一次”的图。
若找不到满足该约束的图，就会报错。

例如，若一个 package 依赖 [`libgit2-sys`] `0.11`，
另一个依赖 `0.12`，会报错。
因为 Cargo 无法把它们统一，
但两者都链接到本地 `git2` 库。
因此如果你的库被广泛使用，且使用了 `links` 字段，
发布 SemVer 不兼容版本时应格外谨慎。

[`links` field]: manifest.md#the-links-field
[`libgit2-sys`]: https://crates.io/crates/libgit2-sys

### 已 yank 的版本

[Yanked releases][yank] 是被标记为“不应继续使用”的发布版本。
resolver 构图时会忽略所有 yank 版本，
除非它们已在 `Cargo.lock` 中，
或被 `cargo update` 的 [`--precise`] 显式请求。

[yank]: publishing.md#cargo-yank
[`--precise`]: ../commands/cargo-update.md#option-cargo-update---precise

## 依赖更新

所有需要依赖图信息的 Cargo 命令都会自动执行依赖解析。
例如 [`cargo build`] 会先运行 resolver，找出需要构建的依赖。
首次解析后结果写入 `Cargo.lock`。
后续命令会再次运行 resolver，并在可能时保持 `Cargo.lock` 中已锁定版本。

如果 `Cargo.toml` 依赖列表变化，
例如把某依赖从 `1.0` 改成 `2.0`，
resolver 会选择满足新要求的新版本。
若新依赖又引入新的要求，可能触发更多连锁更新。
最终 `Cargo.lock` 会更新为新结果。
使用 `--locked` 或 `--frozen` 可阻止这种自动更新，
在要求变化时改为直接报错。

[`cargo update`] 可在新版本发布后更新 `Cargo.lock` 条目。
不带参数时会尝试更新 lock 文件中所有 package。
可用 `-p` 定向更新某个 package，
也可用 `--recursive`、`--precise` 等控制版本选择策略。

[`cargo build`]: ../commands/cargo-build.md
[`cargo update`]: ../commands/cargo-update.md

## 覆盖机制

Cargo 提供多种机制在依赖图内覆盖依赖。
具体用法见 [Overriding Dependencies] 章节。
覆盖会作为 registry 的一层 overlay 生效，
把被 patch 的版本替换为新条目，其余解析逻辑与平常一致。

[Overriding Dependencies]: overriding-dependencies.md

## 依赖类型

package 里有三类依赖：普通依赖、[build] 依赖、[dev][dev-dependencies] 依赖。
从 resolver 角度看，大多数行为一致。
一个差异是：非 workspace 成员的 dev-dependencies 总是被忽略，不影响解析。

`[target]` 表里的[平台特定依赖]会按“所有平台都启用”来解析。
也就是 resolver 会忽略平台和 `cfg` 表达式。

[build]: specifying-dependencies.md#build-dependencies
[dev-dependencies]: specifying-dependencies.md#development-dependencies
[Platform-specific dependencies]: specifying-dependencies.md#platform-specific-dependencies

### dev-dependency 循环

通常 resolver 不允许图中有环，
但对 [dev-dependencies] 允许。
例如项目 `foo` 的 dev-dependency 是 `bar`，
而 `bar` 普通依赖 `foo`（通常是 `path` 依赖）。
这是允许的，因为从构建产物角度看并不形成真实循环：
先构建 `foo` library（不需要 `bar`，因 `bar` 仅测试使用），
再构建依赖 `foo` 的 `bar`，最后构建链接 `bar` 的 `foo` 测试。

但这可能导致令人困惑的错误。
以 library 单元测试为例，最终测试二进制里实际上会链接两份库：
一份是和 `bar` 链接的 `foo`，另一份是包含单元测试代码的 `foo`。
与 [Version-incompatibility hazards] 中的问题类似，
这两份之间的类型互不兼容。
在这种情况下，若 `bar` 暴露了 `foo` 的类型，
`foo` 的单元测试里它们不会被视为本地同一类型。

如果可能，尽量把 package 拆分重构，保持严格无环。

## 解析器版本

可在 `Cargo.toml` 通过 resolver 版本指定不同行为：

```toml
[package]
name = "my-package"
version = "1.0.0"
resolver = "2"
```
- `"1"`（默认）
- `"2"`（[`edition = "2021"`](manifest.md#the-edition-field) 默认）：
  引入 [feature 统一](#features) 相关变化。
  详见 [features chapter][features-2]。
- `"3"`（[`edition = "2024"`](manifest.md#the-edition-field) 默认，需 Rust 1.84+）：
  将 [`resolver.incompatible-rust-versions`] 的默认值从 `allow` 改为 `fallback`。

resolver 是影响整个 workspace 的全局选项。
依赖里的 `resolver` 值会被忽略，只使用顶层 package 的值。
若使用 [virtual workspace]，应在 `[workspace]` 表里指定，例如：

```toml
[workspace]
members = ["member1", "member2"]
resolver = "2"
```

> **MSRV:** 需要 1.51+

[virtual workspace]: workspaces.md#virtual-workspace
[features-2]: features.md#feature-resolver-version-2

## 建议

下面是关于“设置包版本”与“编写依赖要求”的建议。
这些是适用于常见场景的通用指导，但特殊场景可能需要非常规要求。

* 决定如何更新版本号、是否需要 SemVer breaking 版本时，遵循 [SemVer guidelines]。
* 大多数场景下依赖建议使用 caret 要求（如 `"1.2.3"`）。
  这样 resolver 在保持构建兼容前提下，拥有最大的版本选择灵活性。
  * 建议写全三段版本号，并以当前实际使用版本为准。
    这有助于设定最小版本，避免其他用户被解析到过旧依赖而缺少你需要的能力。
  * 避免 `*` 要求：它在 [crates.io] 不允许，且普通 `cargo update` 时可能拉入 SemVer breaking 变更。
  * 避免过宽版本要求。例如 `>=2.0.0` 可能拉入任意不兼容版本（如 `5.0.0`），
    将来容易导致构建损坏。
  * 若可能也避免过窄要求。
    例如你写 `bar="~1.3"`，而另一个 package 写 `bar="1.4"`，
    即使次版本理论兼容，也会解析失败。
* 尽量让依赖版本要求与库“实际最低需要版本”保持同步。
  例如你原先 `bar="1.0.12"`，后续版本开始使用 `bar` `1.1.0` 新能力，
  就应把要求更新到 `bar="1.1.0"`。

  否则问题可能不易立刻暴露：
  因为你本地执行全量 `cargo update` 时，Cargo 可能机会性选择最新版本。
  但若他人依赖你的库并执行 `cargo update your-library`，
  如果 `bar` 在其 `Cargo.lock` 已被锁定，则不会自动更新。
  只有依赖声明也更新时，`bar` 才会更新。
  若未同步更新依赖声明，用户可能遇到难以理解的构建错误。
* 若两个 package 紧耦合，可考虑使用 `=` 依赖要求确保同步。
  例如某库与其配套 proc-macro 库常有隐含耦合，
  若二者版本不同步可能出问题（且通常也不预期被独立使用）。
  父库可对 proc-macro 使用 `=` 约束，并重新导出宏以便使用。
* 对长期不稳定 package，可使用 `0.0.x` 版本线。

总体而言：依赖要求越严格，resolver 失败概率越高；
要求过松，则未来新版本更可能引入破坏构建的变化。

[SemVer guidelines]: semver.md
[crates.io]: https://crates.io/

## 故障排查

下面给出一些常见问题与可能解决办法。

### 为什么会包含这个依赖？

假设你在 `cargo check` 输出看到依赖 `rand`，
但觉得不该出现，想查它为什么被引入。

可以运行：
```console
$ cargo tree --workspace --target all --all-features --invert rand
rand v0.8.5
├── ...

rand v0.8.5
├── ...
```

### 为什么该依赖上的这个 Feature 会被启用？

你可能发现是某个 feature 被启用才导致 `rand` 出现。
**要定位是哪个 package 启用了该 feature，可加 `--edges features`：**
```console
$ cargo tree --workspace --target all --all-features --edges features --invert rand
rand v0.8.5
├── ...

rand v0.8.5
├── ...
```

### 非预期的依赖重复

当你运行：
```console
$ cargo tree --workspace --target all --all-features --duplicates
rand v0.7.3
├── ...

rand v0.8.5
├── ...
```

若看到多个 `rand` 实例，
说明 resolver 收敛到了“一个依赖出现两份”的解，
即使理论上一份也许够用。
例如：

```toml
# Package A
[dependencies]
rand = "0.7"

# Package B
[dependencies]
rand = ">=0.6"  # note: open requirements such as this are discouraged
```

在该例中，Cargo 可能会构建两份 `rand`，
尽管单用 `0.7.3` 也能满足所有要求。
原因是 resolver 倾向给 Package B 选择当下最新 `rand`（编写本文时是 `0.8.5`），
而它与 Package A 的要求不兼容。
当前算法不会在这种场景主动做“去重收敛”。

Cargo 不鼓励使用 `>=0.6` 这种开放式版本要求。
但若你遇到此情况，可用 [`cargo update`] + `--precise`
手动去除这类重复。

[`cargo update`]: ../commands/cargo-update.md

### 为什么没有选择更新版本？

如果你执行：
```console
$ cargo update
```
后发现并未选中依赖最新版本，
可开启额外日志查看原因：
```console
$ env CARGO_LOG=cargo::core::resolver=trace cargo update
```
**注意：**Cargo 日志 target 与级别可能随时间变化。

### 破坏 SemVer 的补丁发布导致构建失败

有时项目会误发布包含 SemVer breaking 变更的补丁版本。
用户执行 `cargo update` 后会拉到该版本，导致构建失败。
这种情况建议项目方先 [yank] 该版本，
然后要么移除 breaking 变更，要么改为提升 SemVer 主版本发布。

如果问题来自第三方项目，
尽量（礼貌地）与对方协作解决。

在版本被 yank 之前，可根据场景采用临时方案：

* 若你的项目是最终产物（如二进制），
  就先不要更新 `Cargo.lock` 里出问题的包。
  可通过 [`cargo update`] 的 `--precise` 实现。
* 若你在 [crates.io] 发布二进制，
  可临时加 `=` 约束，强制依赖到某个已知正常版本。
  * 二进制项目也可建议用户使用 [`cargo install`] 的 `--locked`，
    以使用原始 `Cargo.lock` 中的已知可用版本。
* 库项目可考虑临时发一个新小版本，
  收紧依赖要求以避开问题版本。
  你可以考虑使用区间要求（而不是 `=`）避免过严约束，
  以免与其他 package 发生冲突。
  问题解决后再发一个小版本，把依赖放宽回 caret 要求。
* 若第三方项目看起来无法/不愿 yank，
  一个方案是更新你的代码以兼容其变更，
  并把依赖最小版本提升到该新版本。
  同时也要评估这是否构成你自己库的 SemVer breaking 变更，
  例如你对外暴露了该依赖的类型。

[`cargo install`]: ../commands/cargo-install.md
