# 持续集成

## 快速开始

一个基础 CI 通常会对项目执行构建和测试。

### GitHub Actions

如果你在 GitHub Actions 上测试 package，可以使用如下 `.github/workflows/ci.yml`：

```yaml
name: Cargo Build & Test

on:
  push:
  pull_request:

env:
  CARGO_TERM_COLOR: always

jobs:
  build_and_test:
    name: Rust project - latest
    runs-on: ubuntu-latest
    strategy:
      matrix:
        toolchain:
          - stable
          - beta
          - nightly
    steps:
      - uses: actions/checkout@v4
      - run: rustup update ${{ matrix.toolchain }} && rustup default ${{ matrix.toolchain }}
      - run: cargo build --verbose
      - run: cargo test --verbose
```

这会测试三个发布通道（注意：任一 toolchain 失败都会导致整个 job 失败）。
你也可以在 GitHub 界面中点击 `Actions > new workflow`，选择 Rust，
把[默认配置](https://github.com/actions/starter-workflows/blob/main/ci/rust.yml)
加入仓库。更多信息请参考
[GitHub Actions 文档](https://docs.github.com/en/actions)。

### GitLab CI

如果你在 GitLab CI 上测试 package，可以使用如下 `.gitlab-ci.yml`：

```yaml
stages:
  - build

rust-latest:
  stage: build
  image: rust:latest
  script:
    - cargo build --verbose
    - cargo test --verbose

rust-nightly:
  stage: build
  image: rustlang/rust:nightly
  script:
    - cargo build --verbose
    - cargo test --verbose
  allow_failure: true
```

这会在 stable 和 nightly 上测试，但 nightly 的失败不会导致整体构建失败。
更多信息请参考 [GitLab CI 文档](https://docs.gitlab.com/ce/ci/yaml/index.html)。

### builds.sr.ht

如果你在 sr.ht 上测试 package，可以使用如下 `.build.yml`。
请将 `<your repo>` 和 `<your project>` 替换为要克隆的仓库地址与目录名。

```yaml
image: archlinux
packages:
  - rustup
sources:
  - <your repo>
tasks:
  - setup: |
      rustup toolchain install nightly stable
      cd <your project>/
      rustup run stable cargo fetch
  - stable: |
      rustup default stable
      cd <your project>/
      cargo build --verbose
      cargo test --verbose
  - nightly: |
      rustup default nightly
      cd <your project>/
      cargo build --verbose ||:
      cargo test --verbose  ||:
  - docs: |
      cd <your project>/
      rustup run stable cargo doc --no-deps
      rustup run nightly cargo doc --no-deps ||:
```

这会在 stable 和 nightly 上测试并构建文档，但 nightly 失败不会导致整体构建失败。
更多信息请参考 [builds.sr.ht 文档](https://man.sr.ht/builds.sr.ht/)。

### CircleCI

如果你在 CircleCI 上测试 package，可以使用如下 `.circleci/config.yml`：

```yaml
version: 2.1
jobs:
  build:
    docker:
      # check https://circleci.com/developer/images/image/cimg/rust#image-tags for latest
      - image: cimg/rust:1.77.2
    steps:
      - checkout
      - run: cargo test
```

如果你需要更复杂的流水线（如 flaky 测试检测、缓存、制品管理），
请参考 [CircleCI Configuration Reference](https://circleci.com/docs/configuration-reference/)。

## 验证最新依赖

在 `Cargo.toml` 中[指定依赖](../reference/specifying-dependencies.md)时，
通常会匹配一个版本范围。穷举测试所有版本组合并不现实。
至少验证最新版本，能够覆盖使用 [`cargo add`] 或 [`cargo install`]
安装时的常见用户场景。

测试最新版本时可考虑：
- 尽量减少影响本地开发或 CI 的外部因素
- 新依赖版本发布的频率
- 项目可接受的风险水平
- CI 成本，包括间接成本（例如并行 runner 上限导致任务排队）

可选方案包括：
- [不提交 `Cargo.lock`](../faq.md#why-have-cargolock-in-version-control)
  - 取决于 PR 速度，可能有很多版本未经测试
  - 代价是牺牲确定性
- 在 CI 中校验最新依赖，但将 job 标记为“失败不阻断”
  - 取决于 CI 平台，失败可能不够显眼
  - 取决于 PR 速度，可能比必要消耗更多资源
- 使用定时 CI 任务验证最新依赖
  - 托管 CI 可能会对长期无变更仓库停用定时任务，影响被动维护项目
  - 通知可能无法发送给能处理失败的人
  - 若频率与依赖发布速率不匹配，可能测得不够或重复测试
- 通过 PR 定期更新依赖，例如使用 [Dependabot] 或 [RenovateBot]
  - 可以把依赖拆成独立 PR，也可合并为单个 PR
  - 只使用必要资源
  - 可配置更新频率以平衡资源与覆盖

下面是一个使用 GitHub Actions 验证最新依赖的示例：

```yaml
jobs:
  latest_deps:
    name: Latest Dependencies
    runs-on: ubuntu-latest
    continue-on-error: true
    env:
      CARGO_RESOLVER_INCOMPATIBLE_RUST_VERSIONS: allow
    steps:
      - uses: actions/checkout@v4
      - run: rustup update stable && rustup default stable
      - run: cargo update --verbose
      - run: cargo build --verbose
      - run: cargo test --verbose
```

说明：
- 设置 [`CARGO_RESOLVER_INCOMPATIBLE_RUST_VERSIONS`](../reference/config.md#resolverincompatible-rust-versions)
  是为了确保 [resolver](../reference/resolver.md) 不会因为你项目的
  [Rust 版本](../reference/rust-version.md) 而限制依赖选择。

对于更容易出现“平台相关”或“Rust 版本相关”失败的项目，
可能需要测试更多组合。

## 验证 `rust-version`

如果发布的包声明了 [`rust-version`](../reference/manifest.md#the-rust-version-field)，
应验证该字段是否正确。

可用的第三方工具示例：
- [`cargo-msrv`](https://crates.io/crates/cargo-msrv)
- [`cargo-hack`](https://crates.io/crates/cargo-hack)

下面是一个 GitHub Actions 示例：

```yaml
jobs:
  msrv:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: taiki-e/install-action@cargo-hack
    - run: cargo hack check --rust-version --workspace --all-targets --ignore-private
```

这个方案在完整性与反馈速度间做了平衡：
- 只用单个平台，因为多数项目平台无关，并假定平台相关依赖会自行验证行为。
- 使用 `cargo check`，因为贡献者最常遇到的是 API 可用性问题而非运行行为问题。
- 跳过未发布包，因为这里假设只有通过注册表消费已发布包的用户会关注 `rust-version`。

## 检查警告

通常项目希望在官方分支上保持“无 warning”，但本地开发可相对宽松。
可以使用 [`build.warnings = "deny"`] 在存在 warning 时让 CI 失败。

下面是一个 GitHub Actions 示例：

```yaml
jobs:
  warnings:
    runs-on: ubuntu-latest
    env:
      CARGO_BUILD_WARNINGS: deny
    steps:
      - uses: actions/checkout@v4
      - run: rustup update stable && rustup default stable
      - run: rustup component add clippy
      - run: cargo clippy --all-targets --all-features --keep-going
```

注意事项：
- 由于 warning 的兼容性保证有限，工具链新版本可能导致 CI 失败。
  可考虑固定工具链版本，并用自动化任务在新版本发布时创建升级 PR。
- 选择要检查的平台、特性与包/构建目标组合时，要在覆盖度和反馈速度间平衡。
- 一些 CI 系统支持直接集成 lint 报告，例如 GitHub 上可结合 [`clippy-sarif`]。

[`build.warnings = "deny"`]: ../reference/config.md#buildwarnings
[`cargo add`]: ../commands/cargo-add.md
[`cargo install`]: ../commands/cargo-install.md
[`clippy-sarif`]: https://crates.io/crates/clippy-sarif
[Dependabot]: https://docs.github.com/en/code-security/dependabot/working-with-dependabot
[RenovateBot]: https://renovatebot.com/
