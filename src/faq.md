# 常见问题

## 计划使用 GitHub 作为包仓库吗？

不会。Cargo 的计划是使用 [crates.io]，就像 npm 和 Rubygems
分别使用 [npmjs.com][1] 与 [rubygems.org][3]。

我们会长期支持将 git 仓库作为包来源，
因为这对早期开发和临时补丁都很有价值，
即使人们把 registry 作为主要包来源也是如此。

## 为什么要构建 crates.io，而不是直接把 GitHub 当作 registry？

我们认为支持多种下载包的方式非常重要，
包括从 GitHub 下载以及把包直接拷贝进你的项目。

尽管如此，我们也认为 [crates.io] 有很多关键优势，
并且很可能会成为人们在 Cargo 中下载包的主要方式。

先例方面，Node.js 的 [npm][1] 与 Ruby 的 [bundler][2]
都同时支持中心化 registry 模式与基于 Git 的模式。
在这些生态里，多数包通过 registry 下载，
但也有一部分重要包采用基于 git 的发布方式。

[1]: https://www.npmjs.com
[2]: https://bundler.io
[3]: https://rubygems.org

在其他语言生态中，中心化 registry 受欢迎的一些优势包括：

* **可发现性（Discoverability）**。中心化 registry 提供了便捷入口来查找现有包。
  结合标签系统，还可以提供生态级信息，例如“最流行”或“被依赖最多”的包列表。
* **速度（Speed）**。中心化 registry 能快速高效地仅获取包元数据，
  然后高效下载已发布包本身，而不是仓库里的其他冗余内容。
  这会显著提升依赖解析与下载速度。
  随着依赖图变大，下载全部 git 仓库会很快变慢。
  另外，并非每个人都拥有高速低延迟网络。

## Cargo 能和 C 代码（或其他语言）一起工作吗？

可以！

Cargo 负责编译 Rust 代码，但我们知道很多 Rust 包会链接 C 代码。
我们也知道，围绕 Rust 以外语言编译已经有几十年的工具积累。

我们的方案是：Cargo 允许包在调用 `rustc` 前
[指定一个脚本](reference/build-scripts.md)（用 Rust 编写）先运行。
这样可以利用 Rust 实现平台相关配置，并把包间通用构建逻辑抽离复用。

## Cargo 能在 `make`（或 `ninja` 等）里使用吗？

当然可以。虽然我们希望 Cargo 作为顶层 Rust 包构建工具能独立使用，
但我们也知道有些人希望在其他构建工具中调用 Cargo。

我们在设计 Cargo 时就考虑了这些场景，
例如错误码和机器可读输出模式。
这些方面仍有改进空间，
但在传统脚本环境中使用 Cargo 一直是我们的核心目标。

## Cargo 能处理多平台包或交叉编译吗？

Rust 本身提供了按平台配置代码片段的能力。
Cargo 也支持[平台特定依赖][target-deps]，
未来我们还计划在 `Cargo.toml` 中支持更多按平台配置能力。

[target-deps]: reference/specifying-dependencies.md#platform-specific-dependencies

长期来看，我们也在探索更便捷地用 Cargo 做交叉编译的方式。

## Cargo 是否支持类似 `production` 或 `test` 的环境？

我们通过 [profiles] 来支持不同环境：

[profiles]: reference/profiles.md

* 环境特定编译参数（例如开发用 `-g --opt-level=0`，生产用 `--opt-level=3`）。
* 环境特定依赖（例如测试断言依赖 `hamcrest`）。
* 环境特定 `#[cfg]`。
* `cargo test` 命令。

## Cargo 在 Windows 上可用吗？

可用！

对 Cargo 的每次提交都要求在 Windows 上通过本地测试。
如果你在 Windows 运行时遇到问题，我们会将其视为 bug，
请[提交 issue][cargo-issues]。

[cargo-issues]: https://github.com/rust-lang/cargo/issues

## 为什么要把 `Cargo.lock` 纳入版本控制？

虽然 [`cargo new`] 默认会把 `Cargo.lock` 纳入版本控制，
但是否跟踪仍取决于你的项目需求。

`Cargo.lock` 的目的是记录一次成功构建时的依赖快照。
Cargo 使用 lockfile 在不同时间、不同系统上提供确定性构建，
确保使用与 `Cargo.lock` 初次生成时完全一致的依赖及版本。

确定性构建有助于：
- 通过 `git bisect` 定位 bug 根因
- 确保 CI 失败只由新提交引起，而不是外部因素
- 降低“我本地和他人/CI 行为不一致”造成的困惑

这个依赖快照在某些场景也很有帮助，例如：
- 验证低于依赖最新版所支持范围的最小 Rust 版本（MSRV）
- 验证不承诺兼容性的可读输出（例如快照测试错误消息可读性）

但这种确定性也可能带来“安全错觉”：
`Cargo.lock` 不影响你包的消费者，真正影响消费者的是 `Cargo.toml`。
例如：
- [`cargo install`] 默认会选最新依赖，除非显式传 `
[`--locked`](commands/cargo.html#option-cargo---locked)`。
- 新增依赖（例如 [`cargo add`]）会锁定到当时最新版本

此外 lockfile 也可能成为合并冲突来源。

关于如何在 CI 中验证更新依赖版本，见
[验证最新依赖](guide/continuous-integration.md#verifying-latest-dependencies)。

[`cargo new`]: commands/cargo-new.md
[`cargo add`]: commands/cargo-add.md
[`cargo install`]: commands/cargo-install.md

## 库可以把依赖版本写成 `*` 吗？

**自 2016 年 1 月 22 日起，[crates.io] 会拒绝所有（不仅是库）包含通配依赖约束的包。**

从语法上讲，库*可以*写 `*`，但不应这样做。
`*` 的含义是“任何历史版本都可用”，这几乎不可能成立。
库应始终给出自己真正兼容的版本范围，
哪怕只是“所有 1.x.y 版本”。

## 为什么叫 `Cargo.toml`？

作为与 Cargo 交互最频繁的文件之一，人们常问为何叫 `Cargo.toml`。
前缀大写 `C` 的选择是为了在目录列表中与类似配置文件分组显示。
很多排序规则会把大写字母排在小写前，
这样 `Makefile` 与 `Cargo.toml` 通常会靠在一起。
后缀 `.toml` 则用于强调文件采用的是
[TOML 配置格式](https://toml.io/)。

Cargo 不接受 `cargo.toml` 或 `Cargofile` 等别名，
这是为了让 Cargo 仓库更容易被识别。
历史经验表明，允许多个文件名往往会造成混乱：
某一种情况被处理了，另一种却被遗漏。

[crates.io]: https://crates.io/

## 如何让 Cargo 离线工作？

[`--offline`](commands/cargo.html#option-cargo---offline)
或 [`--frozen`](commands/cargo.html#option-cargo---frozen)
会告诉 Cargo 不访问网络。
如果它本应访问网络，会直接报错。
你可以先在一个项目里用 [`cargo fetch`] 预下载依赖，
然后在另一个项目里复用这些依赖。
也可通过 [配置项][offline config] 在 Cargo 配置中设置。

Vendoring 也相关，详见[源替换][replace]文档。

[replace]: reference/source-replacement.md
[`cargo fetch`]: commands/cargo-fetch.md
[offline config]: reference/config.md#netoffline

## 为什么 Cargo 会重新编译我的代码？

Cargo 负责项目内 crate 的增量编译。
这意味着如果你连续执行两次 `cargo build`，
第二次通常不应重新编译 crates.io 依赖。
但现实中会有 bug，Cargo 有时会出现“看起来不该重编却重编”的情况。

我们一直[想提供更好的诊断](https://github.com/rust-lang/cargo/issues/2904)，
但这个问题暂时还未取得理想进展。
在此之前，你至少可以通过设置 `CARGO_LOG` 做一些排查：

```sh
$ CARGO_LOG=cargo::core::compiler::fingerprint=info cargo build
```

这会让 Cargo 输出大量与重编译相关的诊断信息。
其中经常能看到线索，但目前可读性还不算好，
往往需要你自行串联信息。
注意 `CARGO_LOG` 必须设置在“你认为不该重编却发生重编”的那次命令上。
Cargo 目前还不支持“事后追踪为什么那次重编了”。

历史上常见会导致重编译的情况包括：

* 构建脚本输出了 `cargo::rerun-if-changed=foo`，
  但 `foo` 文件不存在且没有任何流程会生成它。
  Cargo 会持续重跑构建脚本，误以为它会生成该文件。
  修复方式是避免在这种场景输出 `rerun-if-changed`。

* 两次连续 Cargo 构建中，某些依赖启用的 feature 集不同。
  比如第一次构建整个 workspace，第二次只构建其中一个 crate，
  可能导致 crates.io 上同一依赖启用 feature 不同，
  进而触发它及其上游重编译。
  这个问题很难“彻底修复”，
  最好尽量保证同一 crate 在 workspace 中无论构建什么都启用固定 feature 集。

* 某些文件系统在时间戳行为上比较异常。
  Cargo 主要依赖文件时间戳判断是否需要重建。
  如果使用非标准文件系统，时间戳可能被截断、漂移等，导致误判。
  这种情况下欢迎提 issue，我们可以评估是否能适配。

* 并发构建进程在删除产物或修改文件。
  有时后台进程也在构建/检查项目，可能意外删除产物或触碰文件，
  使重编译看起来“莫名其妙”。
  最好的修复是约束后台进程，避免与当前开发流程冲突。

如果你排查后仍有问题，欢迎[提 issue](https://github.com/rust-lang/cargo/issues/new)！

## “version conflict” 是什么意思，怎么解决？

> failed to select a version for `x` which could resolve this conflict

你见过上面的错误吗？

这是 Cargo 用户最常见、也最烦人的报错之一。
造成版本冲突的情况有很多。下面列出常见原因与排查思路：

- 项目及其依赖通过 [links] 重复链接同一本地库。
  Cargo 禁止在同一依赖图中链接两个提供相同 native 库的 package。
  即便经过多层依赖也不允许。
  这类错误常见提示为：`Only one package in the dependency graph may specify the same links value`。
  你可能需要手动检查并去掉重复 links 值。
  社区也有一些[约定实践][conventions in place]来缓解此问题。

- 项目依赖的多个 crate 间，若共享同一依赖库但版本约束彼此冲突，
  导致无法选出满足条件的统一版本，也会触发冲突。
  常见提示：`all possible versions conflict with previously selected packages`。
  需要调整版本约束使其一致。

- 若项目里有多版本依赖，同时启用 [`direct-minimal-versions`]，
  则可能因最低版本要求无法满足而冲突。
  需相应调整直接依赖的版本要求，使其满足最低 SemVer 版本。

- 如果依赖 crate 不包含你选择的 feature，也会导致冲突。
  这时需要核对该依赖版本及其 feature 列表。

- 合并分支或 PR 时也可能冲突。
  若冲突较复杂，可先重置为“yours”，修完其他冲突后，
  运行 `cargo tree` 或 `cargo check` 等命令，让 lockfile 按本地变更重新更新。
  如果你此前在分支上运行过 `cargo update`，可在此时重跑。
  社区也在推进用[自定义合并工具][custom merge tool]解决 `Cargo.lock` 与 `Cargo.toml` 冲突。


[links]: reference/resolver.md#links
[conventions in place]: reference/build-scripts.md#-sys-packages
[`direct-minimal-versions`]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#direct-minimal-versions
[custom merge tool]: https://github.com/rust-lang/cargo/issues/1818

## 为什么我的构建占用空间这么大？

Cargo 为了提升构建速度，会在磁盘空间与构建时间之间做权衡，包括：
- 维护中间构建产物[缓存][cache]，避免改动一个包就重建全部
- 为不同工具链版本、包版本、feature 组合等维护独立[缓存][cache]条目，
  以便在不同配置间切换时避免重复重建
- 对本地包启用[增量编译][incremental compilation]，加快变更后的重建
- 在 [`dev` profile] 中启用 [debuginfo]，便于调试器使用

[incremental compilation]: reference/profiles.md#incremental
[debuginfo]: reference/profiles.md#debug
[`dev` profile]: reference/profiles.md#dev
[cache]: reference/build-cache.md
