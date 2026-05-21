# 优化构建性能

Cargo 的配置选项与源码组织模式可以帮助提升构建性能。
这通常意味着你会优先考虑“构建速度”，而不是某些在你场景中不那么重要的维度。

和优化运行时性能一样，请务必针对你真正关心的工作流做测量。
本文给出的是通用建议；在你的具体场景里，某些做法可能反而变慢。

建议关注的工作流示例：
- 开发期间的编译反馈（代码改动后执行 `cargo check`）
- 开发期间的测试反馈（代码改动后执行 `cargo test`）
- CI 构建

## Cargo 与编译器配置

Cargo 默认配置会在可调试性、运行时性能、构建性能、二进制体积等方面做平衡。
本节介绍一些偏向“最大化构建性能”的调整方式。

常见的覆盖默认值位置：
- [`Cargo.toml` manifest](../reference/profiles.md)
  - 对所有协作者可见
  - 支持的配置范围有限（扩展见 [#12738](https://github.com/rust-lang/cargo/issues/12738)）
- [`$WORKSPACE_ROOT/.cargo/config.toml` 配置文件](../reference/config.md)
  - 对所有协作者可见
  - 与 `Cargo.toml` 不同，它受你从哪个目录调用 `cargo` 影响（见 [#2930](https://github.com/rust-lang/cargo/issues/2930)）
- [`$CARGO_HOME/.cargo/config.toml` 配置文件](../reference/config.md)
  - 供单个开发者控制本地默认行为

### 减少生成的调试信息

建议：在 `Cargo.toml` 或 `.cargo/config.toml` 中添加：

```toml
[profile.dev]
debug = "line-tables-only"

[profile.dev.package."*"]
debug = false

[profile.debugging]
inherits = "dev"
debug = true
```

这会：
- 调整 [`dev` profile](../reference/profiles.md#dev)（开发命令默认 profile）：
  - 对工作区成员只保留有用 panic 回溯所需的[调试信息](../reference/profiles.md#debug)
  - 对依赖不生成调试信息
- 提供一个可选 profile：通过 [`--profile debugging`](../reference/profiles.md#custom-profiles)
  获取完整调试信息

> **注：** 关于重新评估 `dev` profile 的工作，见 [#15931](https://github.com/rust-lang/cargo/issues/15931)。

权衡：
- 优点：代码生成更快（`cargo build`）
- 优点：链接更快
- 优点：`target` 目录磁盘占用更小
- 缺点：若要高质量调试体验，通常需要一次完整重建

### 使用替代代码生成后端

建议：

- 安装 Cranelift 代码生成后端组件：
  ```console
  $ rustup component add rustc-codegen-cranelift-preview --toolchain nightly
  ```
- 在 `Cargo.toml` 或 `.cargo/config.toml` 中添加：
  ```toml
  [profile.dev]
  codegen-backend = "cranelift"
  ```
- 运行 Cargo 时加上 `-Z codegen-backend`，或在 `.cargo/config.toml` 中启用
  [`codegen-backend`](../reference/unstable.md#codegen-backend) 功能。
  - 这是必须的，因为该特性仍不稳定。

这会让 [`dev` profile](../reference/profiles.md#dev) 使用
[Cranelift codegen backend](https://github.com/rust-lang/rustc_codegen_cranelift)
而非默认 LLVM 来生成机器码。Cranelift 的代码生成通常更快，
因此有望提升构建性能。

权衡：
- 优点：代码生成更快（`cargo build`）
- 缺点：**需要 nightly Rust 和 [不稳定 Cargo 特性][codegen-backend-feature]**
- 缺点：生成代码的运行时性能通常更差
  - 会加速 `cargo test` 的构建部分，但可能拉长测试执行部分
- 缺点：仅支持[部分目标平台](https://github.com/rust-lang/rustc_codegen_cranelift?tab=readme-ov-file#platform-support)
- 缺点：可能不支持全部 Rust 特性（例如 unwinding）

[codegen-backend-feature]: ../reference/unstable.md#codegen-backend

### 启用实验性的并行前端

建议：在 `.cargo/config.toml` 中添加：

```toml
[build]
rustflags = "-Zthreads=8"
```

这个 [`rustflags`][build.rustflags] 会启用 Rust 编译器的
[并行前端][parallel-frontend-blog]，并指定 `n` 个线程。
`n` 应根据 CPU 核心数选择，但收益会递减。建议最多使用 `8` 线程。

权衡：
- 优点：构建更快（`cargo check` 与 `cargo build`）
- 缺点：**需要 nightly Rust 和 [不稳定 Rust 特性][parallel-frontend-issue]**

[parallel-frontend-blog]: https://blog.rust-lang.org/2023/11/09/parallel-rustc/
[parallel-frontend-issue]: https://github.com/rust-lang/rust/issues/113349
[build.rustflags]: ../reference/config.md#buildrustflags

### 使用替代链接器

可考虑安装并配置替代链接器，例如
[LLD](https://lld.llvm.org/)、[mold](https://github.com/rui314/mold)
或 [wild](https://github.com/davidlattimore/wild)。
例如在 Linux 上配置 mold，可在 `.cargo/config.toml` 中写：

```toml
[target.'cfg(target_os = "linux")']
# mold, if you have GCC 12+
rustflags = ["-C", "link-arg=-fuse-ld=mold"]

# mold, otherwise
linker = "clang"
rustflags = ["-C", "link-arg=-fuse-ld=/path/to/mold"]
```

虽然依赖可并行构建，但链接通常集中发生在构建末尾，
在增量重建时尤为可能成为瓶颈。
很多时候 Rust 默认链接器已经够快，切换收益不一定值得；
但并非总是如此。例如除 `x86_64-unknown-linux-gnu` 外的一些 Linux 目标，
仍使用速度较慢的系统链接器（细节见 [rust#39915](https://github.com/rust-lang/rust/issues/39915)）。

权衡：
- 优点：链接更快
- 缺点：某些用例可能不支持，尤其是依赖 C/C++ 组件时

### 在整个工作区统一特性解析

可考虑：在项目的 `.cargo/config.toml` 中添加

```toml
[resolver]
feature-unification = "workspace"
```

调用 `cargo` 时，[启用的 features][resolver-features]
会受你当前选择的 workspace 成员影响。
但在大型应用协作中，你可能需要在多个包间频繁构建测试，
这会因公共依赖被激活的 feature 集不同而触发额外重建。
使用 [`feature-unification`][feature-unification] 后，
不论你当前构建哪个包，依赖 feature 集都尽量一致，
从而复用更多构建结果。

权衡：
- 优点：在 workspace 中切换构建不同包时，重建更少
- 缺点：**需要 nightly Rust 和 [不稳定 Cargo 特性][feature-unification]**
- 缺点：某个包激活了 feature，可能掩盖其他包本应激活但未激活的 bug
- 缺点：如果 `--workspace` 下的 feature 统一策略不适用，你这里也不会适用

[resolver-features]: ../reference/resolver.md#features
[feature-unification]: ../reference/unstable.md#feature-unification

## 减少被构建的代码量

### 移除未使用依赖

建议：定期用以下命令检查并移除未使用依赖：

```console
$ cargo +nightly check -Zcargo-lints --workspace --all-targets
```

它可能出现误报，例如：
- 依赖使用是否生效由 `build.rs` 或 `RUSTFLAGS` 动态控制

另外，也建议定期查看隐藏的 [`cargo::unused_dependencies`] 结果：

```console
$ CARGO_LOG=cargo::core::compiler::unused_deps=debug cargo +nightly check -Zcargo-lints --workspace --all-targets
```

它会展示潜在未使用依赖，场景包括：
- registry 与 git 依赖
- 你的 [`package.rust-version`] 太旧，无法使用 `[lints.cargo]`
- 某依赖“可能”仅用于约束传递依赖版本（可改用 `[target."cfg(false)".dependencies]`）
- 某依赖“可能”仅用于激活传递依赖 feature
- `[dev-dependencies]`（目前还无法确保其所有消费者都被覆盖构建）

代码变更后，很容易忽视某些依赖已不再使用、可以删除。

权衡：
- 优点：完整构建和链接更快
- 缺点：**检查未使用依赖时需要 nightly Rust 与 [不稳定特性][cargo-lints]**
- 缺点：从误报中识别真正未使用依赖需要额外精力

[cargo-lints]: ../reference/unstable.md#lintscargo
[`cargo::unused_dependencies`]: ../reference/lints.md#unused_dependencies
[`package.rust-version`]: ../reference/rust-version.md

### 移除依赖中未使用的 features

建议：定期借助第三方工具排查并移除依赖中未使用的 features，例如：
[cargo-features-manager](https://crates.io/crates/cargo-features-manager)、
[cargo-unused-features](https://crates.io/crates/cargo-unused-features)。

代码变更后，依赖 feature 已不再需要的情况很容易被忽略。
移除后可能减少传递依赖数量，或减少单个 crate 需要编译的代码量。
但删除 feature 时要更谨慎，因为 feature 也可能用于期望的行为或性能调整，
这些影响未必能仅靠编译或测试立刻看出来。

权衡：
- 优点：完整构建和链接更快
- 缺点：工具可能会误判某些 feature 为“未使用”
