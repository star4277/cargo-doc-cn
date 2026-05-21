# Rust 版本

`rust-version` 字段是一个可选键，用于告诉 Cargo 你的 package 支持哪个 Rust 工具链版本。

```toml
[package]
# ...
rust-version = "1.56"
```

Rust 版本必须是纯版本号，且至少包含一个组件；
不能包含 semver 运算符或预发布标识。
检查 Rust 版本时会忽略编译器预发布标识（如 `-nightly`）。

> **MSRV:** 自 1.56 起受支持

## 用途

**诊断：**

当你的 package 在不受支持的工具链上编译时，Cargo 会向用户报错。
这能明确支持范围，避免只看到“语法无效”或“标准库缺功能”之类不够直接的报错。
它影响 package 中所有 [Cargo targets](cargo-targets.md)，
包括二进制、示例、测试套件、基准等。
用户可通过 `--ignore-rust-version` 显式选择继续构建不受支持版本。


**开发辅助：**

`cargo add` 会自动为依赖选择与当前 `rust-version` 兼容的最新版本要求。
如果这不是最新版本，`cargo add` 会提示用户，
让用户决定保留该选择还是升级你的 `rust-version`。

[resolver](resolver.md#rust-version) 在选择依赖时也可能考虑 Rust 版本。

其他工具也可能利用该字段，例如 `cargo clippy` 的
[`incompatible_msrv` lint](https://rust-lang.github.io/rust-clippy/stable/index.html#incompatible_msrv)。

> **注意：** 可通过 `--ignore-rust-version` 忽略 `rust-version`。

## 支持预期

以下是一般预期；部分 package 可能在文档中声明不遵循其中某些项。

**完整性（Complete）：**

在所有受支持 Rust 版本下、所有 [feature](features.md) 组合中，
包括二进制与 API 在内的全部功能都可用。

**已验证（Verified）：**

package 功能已在其声明支持的 Rust 版本上验证，包括自动化测试。
另见
[Rust 版本 CI 指南](../guide/continuous-integration.md#verifying-rust-version)。

**可打补丁（Patchable）：**

在许可证允许时，
用户应能通过 [覆盖本地依赖](overriding-dependencies.md)
指向你的 package 的 fork。
此时 Cargo 可能会加载该依赖的整个 workspace，
即使同一 workspace 里其他 package 的支持 Rust 版本不同，
也应保证在声明支持的 Rust 版本上可工作。

**依赖支持（Dependency Support）：**

为满足上述目标，
通常预期每个依赖的版本要求至少支持一个与你 `rust-version` 兼容的版本。
但**不要求**依赖声明必须排除所有不兼容你 `rust-version` 的版本。
实际上，同时兼容两类用户（支持旧 Rust 与不支持旧 Rust）往往更有价值。

## 设置与更新 Rust 版本

支持哪些 Rust 版本是以下因素之间的权衡：
- 维护者成本：无法使用更新工具链或依赖的新特性
- 用户成本：若 package 不能利用新工具链特性，会失去收益（例如用标准库新特性替代 polyfill 以缩短构建时间）
- 对仍使用旧 Rust 用户的可用性

> **注意：** [修改 `rust-version`](semver.md#env-new-rust) 被视为轻微不兼容

> **建议：** 制定并公开你的 Rust 版本支持策略，以及何时调整。
> 用户可据此与自己的策略比较，
> 若不兼容，再决定是接受功能改进损失，还是接受潜在阻塞性 bug 无法修复的风险。
>
> 最简单可维护的策略是始终使用最新 Rust 版本。
>
> 根据风险偏好，更稳妥的次简策略是继续维护支持旧 Rust 的旧主/次版本分支。

### 选择支持的 Rust 版本

你的用户很可能按以下方式跟踪可支持的 Rust 版本：
- 跟随其 Rust 工具链发行方的支持策略，例如 Rust 项目或 Linux 发行版
  - 注意：Rust 项目只为最新版本提供 bug 修复和安全更新。
- 按固定节奏重新验证包与新工具链兼容性，例如每年首个版本、每 5 个版本一次

此外，用户通常不会立即使用新 Rust 版本，
他们需要时间发现并完成再验证，也可能与他人不在同一升级节奏。

示例策略：
- “N-2”：即“跟随最新版本，并给 2 个发布版本的升级宽限”
- 每个偶数版本，并给 2 个发布版本宽限
- 支持本日历年内的所有版本，并给 1 年升级宽限

> **注意：** 若要找出当前项目可兼容的最低 `rust-version`，
> 可使用第三方工具如 [`cargo-msrv`](https://crates.io/crates/cargo-msrv)。

### 更新时间线

当你的策略表明某 Rust 版本不再需要支持时，
可立即更新 `rust-version`，也可按需更新。

让 `rust-version` 相对策略滞后，可以给用户更多升级缓冲。
但这种滞后不可预测，无法作为用户与自身版本策略对齐的可靠依据。

`rust-version` 与既定策略偏离越大，
用户越可能推断出你本无意表达的策略，最终因预期不符而受挫。

若允许偏离，就会出现“什么理由足以放弃某支持版本”的问题。
每个人都可能给出不同且合理的判断，
围绕此讨论常常让相关参与者感到挫败。
这会削弱那些希望避免此类冲突的人，
尤其是新贡献者或偶发贡献者，他们可能觉得自己不适合提出该问题，
或担心冲突影响其改动合并机会。

### 一个 Workspace 中的多策略

Cargo 允许在一个 workspace 内支持多种策略。

在特定 Rust 版本下验证特定 package 可能较复杂。
可借助 [`cargo-hack`](https://crates.io/crates/cargo-hack) 等工具。

对跨策略共享的依赖，必须使用最低公共版本，因为 Cargo 会
[统一 SemVer 兼容版本](resolver.md#semver-compatibility)，
这可能限制高 `rust-version` 成员对共享依赖新特性的使用。

若要让用户 patch 某个 workspace 成员的依赖，
workspace 中每个 package 都需要能在该 workspace 支持的最旧 Rust 版本上被加载。

当使用 [`incompatible-rust-versions = "fallback"`](config.md#resolverincompatible-rust-versions) 时，
某个 package 的 Rust 版本会影响另一个 Rust 版本不同的 package 的依赖选择。
详见 [resolver](resolver.md#rust-version) 章节。

### 单一策略或多策略

缓解支持旧 Rust 成本的一种方式是：
将你的策略应用到仍在维护的旧主/次版本分支。
你通常仍需要定义策略，说明开发分支相对于这些发布分支支持哪些 Rust 版本。

仅在“需要时”更新开发分支，有助于减少需维护的发布分支数量。

这还涉及“哪些改动可以回移（backport）到发布分支”的问题。
若在次版本间回移新功能，下一个可用版本若缺少该功能，
可能被视为破坏性变化，违反 SemVer。
回移本身也存在引入 bug 的风险。

支持旧版本有成本。
该成本取决于 package 中 bug 的风险与影响，以及可接受的回移范围。
按需创建发布分支，并让社区承担一部分回移工作，
是平衡成本的方式。

目前依赖管理工具还不能直接报告“非最新版本仍受支持”，
因此用户仍需从文档中自行识别。

例如，一个 Rust 版本支持策略可以是：
- 开发分支跟踪 Rust 项目最新稳定版，按需更新
  - 每次提升 `rust-version` 时提升次版本号
- 项目支持本日历年的所有 Rust 版本，并额外给 1 年宽限
  - 仍支持某 Rust 版本的最后一个次版本，会接收社区提供的 bug 修复
  - 修复必须回移到开发分支与所需受支持 Rust 版本之间的所有受支持次版本
