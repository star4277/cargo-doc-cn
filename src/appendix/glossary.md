# 术语表

## Artifact

*Artifact*（构建产物）是编译过程生成的文件或文件集合。
这包括可链接库、可执行二进制以及生成的文档等。

## Cargo

*Cargo* 是 Rust 的[*包管理器*](#package-manager)，也是本书的核心主题。

## Cargo.lock

见 [*lock file*](#lock-file)。

## Cargo.toml

见 [*manifest*](#manifest)。

## Crate

Rust 中的 *crate* 可以是库，也可以是可执行程序，
分别称为 *library crate* 与 *binary crate*。

在 Cargo [package](#package) 中定义的每个 [target](#target) 都是一个 *crate*。

广义上，*crate* 一词既可指 target 的源代码，也可指该 target 生成的编译产物。
它有时也可指从 [registry](#registry) 获取的压缩包。

一个 crate 的源代码可以被拆分为 [*module*](#module)。

## Edition

*Rust edition* 是 Rust 语言演进中的阶段性里程碑。
[package 的 edition][edition-field] 在 `Cargo.toml` [manifest](#manifest)
中指定，单个 target 也可指定所使用的 edition。更多信息见 [Edition Guide]。

## Feature

*feature* 的含义依上下文而定：

- [*feature*][feature] 是一种命名标记，用于条件编译。
  feature 可以表示一个可选依赖，也可以是 `Cargo.toml` [manifest](#manifest)
  中定义的任意名称，并在源码中进行判断。

- Cargo 有[*不稳定特性标记*][cargo-unstable]，用于启用 Cargo 自身的实验性行为。

- Rust 编译器和 Rustdoc 也有各自的不稳定特性（参见
  [The Unstable Book][unstable-book] 与 [The Rustdoc Book][rustdoc-unstable]）。

- CPU target 还包含[*target features*][target-feature]，用于描述 CPU 能力。

## Index

*Index* 是 [*registry*](#registry) 中可搜索的 [*crate*](#crate) 列表。

## Lock file

`Cargo.lock` *lock file* 用于记录 [*workspace*](#workspace)
或 [*package*](#package) 中每个依赖的精确版本。它由 Cargo 自动生成。
参见 [Cargo.toml vs Cargo.lock]。

## Manifest

[*Manifest*][manifest] 是名为 `Cargo.toml` 的文件，
用于描述一个 [package](#package) 或 [workspace](#workspace)。

[*Virtual manifest*][virtual] 是一种只描述 workspace、
不包含 package 定义的 `Cargo.toml`。

## Member

*Member* 是属于某个 [*workspace*](#workspace) 的 [*package*](#package)。

## Module

Rust 的模块系统用于把代码组织为称作 *module* 的逻辑单元，
并在代码中提供隔离命名空间。

一个 [crate](#crate) 的源代码可拆分为一个或多个独立 module。
通常用于按功能区域组织代码，或控制符号（结构体、函数等）
在源码中的可见范围（public/private）。

[`Cargo.toml`](#manifest) 主要关注其定义的 [package](#package)、
包内 crate，以及这些 crate 所依赖包的信息。
不过在使用 Rust 时你会频繁看到 “module” 一词，
因此理解 module 与 crate 的关系很重要。

## Package

*Package* 是一组源文件，加上用于描述该包的 `Cargo.toml`
[*manifest*](#manifest)。package 拥有名称与版本，
用于在包之间声明依赖关系。

一个 package 可包含多个 [*target*](#target)，每个 target 都是
一个 [*crate*](#crate)。`Cargo.toml` 会描述包内 crate 类型
（binary 或 library），以及每个 crate 的元数据，例如如何构建、
直接依赖有哪些等（本书后续会详细说明）。

*Package root* 是 package 的 `Cargo.toml` 所在目录。
（可对照 [*workspace root*](#workspace)。）

[*Package ID specification*][pkgid-spec]（也称 *SPEC*）
是用于唯一标识“某个来源中的某个版本 package”的字符串。

中小型 Rust 项目通常只需要一个 package，但其中包含多个 crate 也很常见。

较大型项目可能涉及多个 package，这时可通过 Cargo
[*workspace*](#workspace) 来统一管理公共依赖和相关元数据。

## Package manager

广义上，*package manager* 是软件生态中的程序（或程序集合），
用于自动化获取、安装和升级构建产物。
在编程语言生态中，包管理器通常是面向开发者的工具，
其核心能力是从某个中心仓库下载库产物及其依赖；
这往往还会结合构建能力（调用语言对应编译器）。

在 Rust 生态中，[*Cargo*](#cargo) 就是包管理器。
Cargo 会下载 Rust [package](#package) 的依赖（即称作
[*crate*](#crate) 的 [*artifact*](#artifact)），编译你的 package，
生成可分发包，并可选上传到 [crates.io][]（Rust 社区的
[*registry*](#registry)）。

## Package registry

见 [*registry*](#registry)。

## Project

[package](#package) 的另一种叫法。

## Registry

*Registry* 是一种服务，包含可下载的 [*crate*](#crate) 集合，
可被安装或作为 [*package*](#package) 的依赖使用。
Rust 生态默认 registry 是 [crates.io](https://crates.io)。
Registry 拥有 [*index*](#index)，其中列出所有 crate，
并告诉 Cargo 如何下载所需 crate。

## Source

*Source* 是可提供 [*crate*](#crate) 的来源，
这些 crate 可作为 [*package*](#package) 的依赖。
source 有多种类型：

- **Registry source**：见 [registry](#registry)。
- **Local registry source**：以压缩文件形式存放在文件系统中的 crate 集合。
  见 [Local Registry Sources]。
- **Directory source**：以未压缩文件形式存放在文件系统中的 crate 集合。
  见 [Directory Sources]。
- **Path source**：位于文件系统中的单个 package（如 [path dependency]），
  或多个 package 的集合（如 [path overrides]）。
- **Git source**：位于 git 仓库中的 package（如 [git dependency]
  或 [git source]）。

更多信息见 [Source Replacement]。

## Spec

见 [package ID specification](#package)。

## Target

*Target* 一词依上下文有不同含义：

- **Cargo Target**：Cargo [*package*](#package) 由多个 *target* 构成，
  对应将被生成的 [*artifact*](#artifact)。package 可包含 library、binary、
  example、test 与 benchmark target。
  [target 列表][targets] 在 `Cargo.toml` [*manifest*](#manifest)
  中配置，通常也会根据源码[目录布局][directory layout]自动推断。
- **Target Directory**：Cargo 会将构建产物放在 *target* 目录。
  默认是 [*workspace*](#workspace) 根目录下名为 `target` 的目录；
  若未使用 workspace，则在 package 根目录。
  该目录可通过 `--target-dir` 命令行选项、`CARGO_TARGET_DIR`
  [环境变量][environment variable]或 `build.target-dir`
  [配置项][config option]修改。
  更多信息见 [build cache] 文档。
- **Target Architecture**：构建产物对应的操作系统与机器架构，
  通常也简称 *target*。
- **Target Triple**：triple 是指定 target 架构的特定格式。
  “target triple” 表示产物目标架构，“host triple” 表示编译器运行架构。
  target triple 可通过 `--target` 命令行选项或 `build.target`
  [配置项][config option]指定。其一般格式为
  `<arch><sub>-<vendor>-<sys>-<abi>`，其中：

  - `arch`：基础 CPU 架构，如 `x86_64`、`i686`、`arm`、`thumb`、`mips` 等。
  - `sub`：CPU 子架构，例如 `arm` 可有 `v7`、`v7s`、`v5te` 等。
  - `vendor`：厂商，如 `unknown`、`apple`、`pc`、`nvidia` 等。
  - `sys`：系统名，如 `linux`、`windows`、`darwin` 等。
    裸机（无操作系统）通常用 `none`。
  - `abi`：ABI，如 `gnu`、`android`、`eabi` 等。

  某些字段可以省略。可执行 `rustc --print target-list` 查看支持的 target 列表。

## Test Targets

Cargo 的 *test target* 会生成用于验证代码行为与正确性的二进制。
测试产物分为两类：

* **Unit test**：*unit test* 是直接从 library 或 binary target 编译得到的
  可执行二进制。它包含库或二进制代码的完整内容，并运行带 `#[test]`
  标注的函数，用于验证代码中的独立单元。
* **Integration test target**：[ *integration test target* ][integration-tests]
  是从 *test target* 编译出的可执行二进制；
  该 test target 是一个独立 [*crate*](#crate)，其源码位于 `tests` 目录，
  或由 `Cargo.toml` [*manifest*](#manifest) 中的
  [`[[test]]` 表][targets]指定。
  它通常用于测试库的公开 API，或执行某个二进制来验证其行为。

## Workspace

[*Workspace*][workspace] 是一个或多个 [*package*](#package) 的集合，
它们共享依赖解析（共享 `Cargo.lock` [*lock file*](#lock-file)）、
输出目录以及 profile 等设置。

[*Virtual workspace*][virtual] 指 workspace 根 `Cargo.toml`
[*manifest*](#manifest) 不定义 package、只列出 workspace
[*member*](#member) 的情况。

*Workspace root* 是 workspace 的 `Cargo.toml` 所在目录。
（可对照 [*package root*](#package)。）


[Cargo.toml vs Cargo.lock]: ../guide/cargo-toml-vs-cargo-lock.md
[Directory Sources]: ../reference/source-replacement.md#directory-sources
[Local Registry Sources]: ../reference/source-replacement.md#local-registry-sources
[Source Replacement]: ../reference/source-replacement.md
[build cache]: ../reference/build-cache.html
[cargo-unstable]: ../reference/unstable.md
[config option]: ../reference/config.md
[crates.io]: https://crates.io/
[directory layout]: ../guide/project-layout.md
[edition guide]: ../../edition-guide/index.html
[edition-field]: ../reference/manifest.md#the-edition-field
[environment variable]: ../reference/environment-variables.md
[feature]: ../reference/features.md
[git dependency]: ../reference/specifying-dependencies.md#specifying-dependencies-from-git-repositories
[git source]: ../reference/source-replacement.md
[integration-tests]: ../reference/cargo-targets.md#integration-tests
[manifest]: ../reference/manifest.md
[path dependency]: ../reference/specifying-dependencies.md#specifying-path-dependencies
[path overrides]: ../reference/overriding-dependencies.md#paths-overrides
[pkgid-spec]: ../reference/pkgid-spec.md
[rustdoc-unstable]: https://doc.rust-lang.org/nightly/rustdoc/unstable-features.html
[target-feature]: ../../reference/attributes/codegen.html#the-target_feature-attribute
[targets]: ../reference/cargo-targets.md#configuring-a-target
[unstable-book]: https://doc.rust-lang.org/nightly/unstable-book/index.html
[virtual]: ../reference/workspaces.md
[workspace]: ../reference/workspaces.md
