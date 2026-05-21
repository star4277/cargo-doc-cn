# Unstable Features

实验性的 Cargo 特性仅在 [nightly channel] 可用。
建议你实际尝试这些特性，评估它们是否满足需求、是否存在问题。
可查看下方关联的 tracking issue 了解每个特性的详细信息；
如果希望跟进后续进展，可在 GitHub 对对应 issue 订阅。

经过一段时间后，如果某特性没有重大问题，就可能被 [stabilized]。
一旦当前 nightly 发布流转到 stable（通常约 6 到 12 周），
该特性就会在 stable 可用。

根据特性类型不同，不稳定特性有三种启用方式：

* 对 `Cargo.toml` 的新语法，需要在 `Cargo.toml` 顶部（任何 table 之前）
  添加 `cargo-features` 键。例如：

  ```toml
  # This specifies which new Cargo.toml features are enabled.
  cargo-features = ["test-dummy-unstable"]

  [package]
  name = "my-package"
  version = "0.1.0"
  im-a-teapot = true  # This is a new option enabled by test-dummy-unstable.
  ```

* 新的命令行 flag / option / 子命令，需要额外带上
  `-Z unstable-options`。例如新的 `--artifact-dir` 仅在 nightly 可用：

  ```cargo +nightly build --artifact-dir=out -Z unstable-options```

* `-Z` 命令行 flag 用于启用一些新能力：
  这些能力可能尚无稳定接口、接口尚未定型，或属于影响 Cargo 多个子系统的复杂特性。
  例如可用以下方式启用 [mtime-on-use](#mtime-on-use)：

  ```cargo +nightly build -Z mtime-on-use```

  运行 `cargo -Z help` 可查看可用 flag 列表。

  任何可通过 `-Z` 配置的项，也可在 cargo [config file]
  （`.cargo/config.toml`）的 `unstable` 表中设置。例如：

  ```toml
  [unstable]
  mtime-on-use = true
  build-std = ["core", "alloc"]
  ```

下文每个新特性都应包含对应使用方式说明。

*若要查看最新 nightly 文档，请参阅本页的 [nightly version]。*

[config file]: config.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[stabilized]: https://doc.crates.io/contrib/process/unstable.html#stabilization
[nightly version]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html

## List of unstable features

* 不稳定专用特性
    * [-Z allow-features](#allow-features) --- 限制可使用的不稳定特性集合。
* 构建脚本与链接
    * [Metabuild](#metabuild) --- 提供声明式构建脚本。
    * [Multiple Build Scripts](#multiple-build-scripts) --- 允许使用多个构建脚本。
    * [Any Build Script Metadata](#any-build-script-metadata) --- 允许任意构建脚本通过 `cargo::metadata=key=value` 指定环境变量。
* 解析器与特性
    * [no-index-update](#no-index-update) --- 阻止 cargo 更新索引缓存。
    * [avoid-dev-deps](#avoid-dev-deps) --- 在解析时避免纳入 dev-dependencies。
    * [minimal-versions](#minimal-versions) --- 强制解析器使用最低兼容版本，而非最高版本。
    * [direct-minimal-versions](#direct-minimal-versions) — 仅对直接依赖强制使用最低兼容版本，而非最高版本。
    * [public-dependency](#public-dependency) --- 允许将依赖标记为 public 或 private。
    * [msrv-policy](#msrv-policy) --- MSRV 感知的解析与版本选择。
    * [precise-pre-release](#precise-pre-release) --- 允许通过 `update --precise` 选择预发布版本。
    * [sbom](#sbom) --- 为编译产物生成 SBOM 前置文件。
    * [update-breaking](#update-breaking) --- 允许使用 `update --breaking` 升级到破坏性版本。
    * [feature-unification](#feature-unification) --- 在工作区中启用新的 feature 统一模式。
    * [lockfile-publish-time](#lockfile-publish-time) --- 将解析范围限制为指定时间之前发布的包。
* 输出行为
    * [artifact-dir](#artifact-dir) --- 增加一个用于复制构建产物的目录。
    * [build-dir-new-layout](#build-dir-new-layout) --- 启用新的 build-dir 文件系统布局。
    * [Different binary name](#different-binary-name) --- 允许构建出的二进制名与 crate 名不同。
    * [root-dir](#root-dir) --- 控制输出路径相对于哪个根目录打印。
* 编译行为
    * [mtime-on-use](#mtime-on-use) --- 每次使用依赖时都更新其最后修改时间，为删除未使用产物提供依据。
    * [build-std](#build-std) --- 构建标准库，而不是使用预编译二进制。
    * [build-std-features](#build-std-features) --- 设置标准库构建时使用的 feature。
    * [binary-dep-depinfo](#binary-dep-depinfo) --- 让 dep-info 文件跟踪二进制依赖。
    * [checksum-freshness](#checksum-freshness) --- 判断 crate 是否需要重建时使用文件校验和而非 mtime。
    * [panic-abort-tests](#panic-abort-tests) --- 允许以 `abort` panic 策略运行测试。
    * [host-config](#host-config) --- 允许为主机构建目标设置类似 `[target]` 的配置。
    * [no-embed-metadata](#no-embed-metadata) --- 向编译器传 `-Zembed-metadata=no`，避免在 rlib/dylib 中嵌入元数据以节省磁盘空间。
    * [target-applies-to-host](#target-applies-to-host) --- 调整某些 flag 是否会传给主机构建目标。
    * [gc](#gc) --- 全局缓存垃圾回收。
    * [open-namespaces](#open-namespaces) --- 允许多个包参与同一 API 命名空间。
    * [panic-immediate-abort](#panic-immediate-abort) --- 向编译器传 `-Cpanic=immediate-abort`。
    * [compile-time-deps](#compile-time-deps) --- rust-analyzer 使用的永久不稳定特性。
    * [fine-grain-locking](#fine-grain-locking) --- 使用细粒度锁，而不是锁住整个构建缓存。
    * [json-target-spec](#json-target-spec) --- 允许使用 `.json` 自定义目标规格。
* rustdoc
    * [rustdoc-map](#rustdoc-map) --- 为文档提供映射，以链接到 [docs.rs](https://docs.rs/) 等外部站点。
    * [scrape-examples](#scrape-examples) --- 在文档中展示示例。
    * [output-format](#output-format-for-rustdoc) --- 允许同时输出实验性的 [JSON format](https://doc.rust-lang.org/nightly/nightly-rustc/rustdoc_json_types/) 文档。
    * [rustdoc-depinfo](#rustdoc-depinfo) --- 在 rustdoc 重建检测中使用 dep-info 文件。
    * [rustdoc-mergeable-info](#rustdoc-mergeable-info) --- 使用 rustdoc 可合并的跨 crate 信息文件。
* `Cargo.toml` 扩展
    * [Profile `rustflags` option](#profile-rustflags-option) --- 直接传给 rustc。
    * [Profile `hint-mostly-unused` option](#profile-hint-mostly-unused-option) --- 提示某依赖“多数未使用”，以优化编译时间。
    * [codegen-backend](#codegen-backend) --- 选择 rustc 使用的代码生成后端。
    * [per-package-target](#per-package-target) --- 为每个包单独设置 `--target`。
    * [artifact dependencies](#artifact-dependencies) --- 允许将构建产物作为其他构建产物的输入，并可针对不同目标构建。
    * [Profile `trim-paths` option](#profile-trim-paths-option) --- 控制构建输出中文件路径的清理。
    * [`[lints.cargo]`](#lintscargo) --- 允许配置 Cargo 相关 lint。
    * [path bases](#path-bases) --- 为路径依赖定义命名基目录。
    * [`unstable-editions`](#unstable-editions) --- 允许使用尚未稳定的 edition。
* 信息与元数据
    * [unit-graph](#unit-graph) --- 输出 Cargo 内部图结构的 JSON。
    * [`cargo rustc --print`](#rustc---print) --- 调用 rustc 的 `--print` 输出信息。
    * [Build analysis](#build-analysis) --- 跨多次构建记录并持久化详细指标，并提供新命令查询历史构建。
    * [`rustc-unicode`](#rustc-unicode) --- 在 Cargo 报错中启用 rustc 的 Unicode 错误格式。
* 配置
    * [`cargo config`](#cargo-config) --- 新增查看配置文件的子命令。
* 注册表
    * [publish-timeout](#publish-timeout) --- 控制 crate 上传后到索引可见之间的超时时间。
    * [asymmetric-token](#asymmetric-token) --- 增加对非对称加密认证令牌的支持（`cargo:paseto` provider）。
* 其他
    * [gitoxide](#gitoxide) --- 在一组操作中用 `gitoxide` 替代 `git2`。
    * [script](#script) --- 启用单文件 `.rs` 包支持。
    * [native-completions](#native-completions) --- 将 cargo shell 补全迁移为原生补全。
    * [Package message format](#package-message-format) --- `cargo package` 的消息格式。
    * [`fix-edition`](#fix-edition) --- 永久不稳定的 edition 迁移助手。
    * [Plumbing subcommands](https://github.com/crate-ci/cargo-plumbing) --- 低层命令，可作为 Cargo API 使用（如 `cargo metadata`）。

## allow-features

这个永久不稳定 flag 用于限制“仅允许使用指定的不稳定特性”。
具体来说，如果你传 `-Zallow-features=foo,bar`，
那么仍可给 `cargo` 传 `-Zfoo` 与 `-Zbar`，
但不能再传 `-Zbaz`。
你也可以传空字符串（`-Zallow-features=`）来禁止所有不稳定特性。

`-Zallow-features` 还会限制 `Cargo.toml` 的 `cargo-features`
里可用的不稳定特性。
例如你想允许：

```toml
cargo-features = ["test-dummy-unstable"]
```

其中 `test-dummy-unstable` 是不稳定特性；
它会在 `-Zallow-features=` 下被禁止，
在下面参数下被允许：
`-Zallow-features=test-dummy-unstable`.

传给 cargo 的 `-Zallow-features` 特性列表也会传递给 Cargo 调用的 Rust 工具
（如 `rustc`、`rustdoc`）。
因此当你执行 `cargo -Zallow-features=` 时，
不论是 Cargo 还是 Rust 的不稳定特性都无法使用。

## no-index-update
* Original Issue: [#3479](https://github.com/rust-lang/cargo/issues/3479)
* Tracking Issue: [#7404](https://github.com/rust-lang/cargo/issues/7404)

`-Z no-index-update` 会确保 Cargo 不尝试更新 registry 索引。
这主要用于像 Crater 这类会批量执行大量 Cargo 命令的工具，
可避免每次都更新索引带来的网络延迟。

## mtime-on-use
* Original Issue: [#6477](https://github.com/rust-lang/cargo/pull/6477)
* Cache usage meta tracking issue: [#7150](https://github.com/rust-lang/cargo/issues/7150)

`-Z mtime-on-use` 是一个实验性功能：
让 Cargo 在文件被使用时更新其 mtime，
从而让 cargo-sweep 这类工具更容易识别哪些文件已陈旧。
在很多工作流中，这需要对 *所有* Cargo 调用都启用。
为便于使用，可以在 `.cargo/config.toml` 中设置 `unstable.mtime_on_use`：
或使用对应环境变量，把 `-Z mtime-on-use` 应用于所有 nightly cargo 调用
（该配置在 stable 中会被忽略）。

## avoid-dev-deps
* Original Issue: [#4988](https://github.com/rust-lang/cargo/issues/4988)
* Tracking Issue: [#5133](https://github.com/rust-lang/cargo/issues/5133)

在运行 `cargo install`、`cargo build` 等命令时，
Cargo 当前即使不使用 dev-dependencies 也会下载它们。
`-Z avoid-dev-deps` 允许在不需要时跳过 dev-dependencies 下载。
如果跳过了 dev-dependencies，则不会生成 `Cargo.lock`。

## minimal-versions
* Original Issue: [#4100](https://github.com/rust-lang/cargo/issues/4100)
* Tracking Issue: [#5657](https://github.com/rust-lang/cargo/issues/5657)

> Note: 不建议使用此特性。
> 它会对所有传递依赖强制最小版本，而许多外部依赖并未正确声明下界版本，实用性有限。
> 该特性计划在未来调整为“仅对直接依赖强制最小版本”。

生成 `Cargo.lock` 时，`-Z minimal-versions` 会把依赖解析到“满足要求的最小 SemVer 版本”
（而非最大版本）。

这个 flag 的目标用例是：在 CI 中验证 `Cargo.toml` 指定版本是否真实反映了你实际使用的最小版本。
也就是说，如果 `Cargo.toml` 写的是 `foo = "1.0.0"`，
就不应意外依赖只在 `foo 1.5.0` 才引入的功能。

## direct-minimal-versions
* Original Issue: [#4100](https://github.com/rust-lang/cargo/issues/4100)
* Tracking Issue: [#5657](https://github.com/rust-lang/cargo/issues/5657)

生成 `Cargo.lock` 时，`-Z direct-minimal-versions` 会仅对“直接依赖”解析到满足要求的最小 SemVer 版本
（而非最大版本）。

这个 flag 的目标用例同样是：在 CI 中验证 `Cargo.toml` 指定版本是否真实反映了你实际使用的最小版本。
即若写了 `foo = "1.0.0"`，不应意外依赖只在 `foo 1.5.0` 中新增的功能。

间接依赖仍按常规方式解析，以免被其最小版本校验阻塞。

## artifact-dir
* Original Issue: [#4875](https://github.com/rust-lang/cargo/issues/4875)
* Tracking Issue: [#6790](https://github.com/rust-lang/cargo/issues/6790)

该特性允许指定“构建完成后产物复制到哪里”。
通常产物只会写入 `target/release` 或 `target/debug`。
但要确定精确文件名通常较麻烦，因为常需解析 JSON 输出。
`--artifact-dir` 可让你更可预测地访问产物。
注意这里是“复制”，原始产物仍保留在 `target` 目录。示例：

```sh
cargo +nightly build --artifact-dir=out -Z unstable-options
```

也可在 `.cargo/config.toml` 中配置：

```toml
[build]
artifact-dir = "out"
```

## root-dir
* Original Issue: [#9887](https://github.com/rust-lang/cargo/issues/9887)
* Tracking Issue: None (not currently slated for stabilization)

`-Zroot-dir` 用于设置路径打印时的基准根目录。
它同时影响诊断信息与 `file!()` 宏输出路径。

## Metabuild
* Tracking Issue: [rust-lang/rust#49803](https://github.com/rust-lang/rust/issues/49803)
* RFC: [#2196](https://github.com/rust-lang/rfcs/blob/master/text/2196-metabuild.md)

Metabuild 用于实现声明式构建脚本。
你无需手写 `build.rs`，而是在 `Cargo.toml` 的 `metabuild` 键里列出构建依赖。
Cargo 会自动生成构建脚本，按顺序运行各构建依赖。
Metabuild 包可再从 `Cargo.toml` 读取元数据来决定行为。

在 `Cargo.toml` 顶部加入 `cargo-features`，
在 `package` 中加入 `metabuild` 键，
在 `build-dependencies` 列出依赖，
并在 `package.metadata` 下填写 metabuild 包需要的元数据。示例：

```toml
cargo-features = ["metabuild"]

[package]
name = "mypackage"
version = "0.0.1"
metabuild = ["foo", "bar"]

[build-dependencies]
foo = "1.0"
bar = "1.0"

[package.metadata.foo]
extra-info = "qwerty"
```

Metabuild 包应提供一个名为 `metabuild` 的公开函数，
其执行行为应等价于常规 `build.rs` 脚本。

## Multiple Build Scripts
* Tracking Issue: [#14903](https://github.com/rust-lang/cargo/issues/14903)
* Original Pull Request: [#15630](https://github.com/rust-lang/cargo/pull/15630)

Multiple Build Scripts 特性允许你在同一个包中使用多个构建脚本。

在 `Cargo.toml` 顶部加入 `cargo-features`，并添加 `multiple-build-scripts` 以启用该特性。
将构建脚本路径以数组形式写入 `package.build`。例如：

```toml
cargo-features = ["multiple-build-scripts"]

[package]
name = "mypackage"
version = "0.0.1"
build = ["foo.rs", "bar.rs"]
```

**访问输出目录**：每个构建脚本的输出目录可通过 `<script-name>_OUT_DIR` 访问
  where the `<script-name>` is the file-stem of the build script, exactly as-is.
  For example, `bar_OUT_DIR` for script at `foo/bar.rs`. (Only set during compilation, can be accessed via `env!` macro)

## Any Build Script Metadata
* Tracking Issue: [#14903](https://github.com/rust-lang/cargo/issues/3544)

允许任意构建脚本通过 `cargo::metadata=key=value` 指定环境变量。

依赖方构建脚本可在运行时读取 `CARGO_DEP_<dep>_<key>` 环境变量来获取这些键值对。
对于带 `links` 的 crate 构建脚本，会同时设置 `DEP_<links>_<key>` 和 `CARGO_DEP_<dep>_<key>`。

注意：`CARGO_DEP_<dep>_<key>` 中的 `dep` 与 `key` 会转为大写，连字符（`-`）会替换为下划线（`_`）。

## public-dependency
* Tracking Issue: [#44663](https://github.com/rust-lang/rust/issues/44663)

`public-dependency` 特性允许将依赖标记为 `public` 或 `private`。
启用后，Cargo 会向 rustc 传递额外信息，使 [exported_private_dependencies](../../rustc/lints/listing/warn-by-default.html#exported-private-dependencies) lint 正常工作。

要启用该特性，可以使用 `-Zpublic-dependency`

```sh
cargo +nightly run -Zpublic-dependency
```

或在 `[unstable]` 表中设置，例如：

```toml
# .cargo/config.toml
[unstable]
public-dependency = true
```

`public-dependency` could also be enabled in `cargo-features`, **though this is deprecated and will be removed soon**.

```toml
cargo-features = ["public-dependency"]

[dependencies]
my_dep = { version = "1.2.3", public = true }
private_dep = "2.0.0" # Will be 'private' by default
```

文档更新：
- 对 workspace 的 “`dependencies` 表” 小节，将 `public` 列为 `workspace.dependencies` 的不支持字段。
- 使用 `public` 所需的最低 MSRV 为 1.83（见 [#14507](https://github.com/rust-lang/cargo/pull/14507)）。

## msrv-policy
- [RFC: MSRV-aware Resolver](https://rust-lang.github.io/rfcs/3537-msrv-resolver.html)
- [#9930](https://github.com/rust-lang/cargo/issues/9930) (MSRV-aware resolver)

用于承载 [RFC 2495](https://github.com/rust-lang/rfcs/pull/2495) 下所有 MSRV 感知 Cargo 特性的通用不稳定特性。

### MSRV-aware cargo add

该功能已在 1.79 版本通过 [#13608](https://github.com/rust-lang/cargo/pull/13608) 稳定。

### MSRV-aware resolver

该功能已在 1.84 版本通过 [#14639](https://github.com/rust-lang/cargo/pull/14639) 稳定。

### Convert `incompatible_toolchain` error into a lint

未实现。

### `--update-rust-version` flag for `cargo add`, `cargo update`

未实现。

### `package.rust-version = "toolchain"`

未实现。

### Update `cargo new` template to set `package.rust-version = "toolchain"`

未实现。

## precise-pre-release

* Tracking Issue: [#13290](https://github.com/rust-lang/cargo/issues/13290)
* RFC: [#3493](https://github.com/rust-lang/rfcs/pull/3493)

`precise-pre-release` 特性允许在 `update --precise` 中选择预发布版本，
即使项目的 `Cargo.toml` 没有显式指定预发布版本也可以。

Take for example this `Cargo.toml`.

```toml
[dependencies]
my-dependency = "0.1.1"
```

可以通过 `update -Zunstable-options my-dependency --precise 0.1.2-pre.0`
将 `my-dependency` 更新到预发布版本。
这是因为 `0.1.2-pre.0` 被视为与 `0.1.1` 兼容。
但不能用同样方式从 `0.1.1` 升级到 `0.2.0-pre.0`。

## sbom
* Tracking Issue: [#13709](https://github.com/rust-lang/cargo/pull/13709)
* RFC: [#3553](https://github.com/rust-lang/rfcs/pull/3553)

`sbom` 构建配置允许在每个编译产物旁生成所谓的 SBOM 前置文件。
软件物料清单（SBOM）工具可以利用这些生成文件，收集 Cargo 构建过程中的重要信息，
而这些信息往往难以或无法通过其他方式获取。

要启用该特性，可在 `.cargo/config.toml` 中设置 `sbom` 字段

```toml
[unstable]
sbom = true

[build]
sbom = true
```

或者将环境变量 `CARGO_BUILD_SBOM` 设为 `true`。
该功能通过 `-Z sbom` 开关启用。

生成的输出文件为 JSON 格式，命名规则为 `<artifact>.cargo-sbom.json`。
该 JSON 文件包含依赖、目标、features 以及所用 rustc 编译器信息。

SBOM 前置文件会为所有被复制到 target 或 artifact 目录中的可执行/可链接输出生成。

### Environment variables Cargo sets for crates

* `CARGO_SBOM_PATH` -- 生成的 SBOM 前置文件列表，以平台 PATH 分隔符分隔；可用 `std::env::split_paths` 拆分。

### SBOM pre-cursor schema

```json5
{
  // Schema version.
  "version": 1,
  // Index into the crates array for the root crate.
  "root": 0,
  // Array of all crates. There may be duplicates of the same crate if that
  // crate is compiled differently (different opt-level, features, etc).
  "crates": [
    {
      // Fully qualified package ID specification
      "id": "path+file:///sample-package#0.1.0",
      // List of target kinds: bin, lib, rlib, dylib, cdylib, staticlib, proc-macro, example, test, bench, custom-build
      "kind": ["bin"],
      // Enabled feature flags.
      "features": [],
      // Dependencies for this crate.
      "dependencies": [
        {
          // Index in to the crates array.
          "index": 1,
          // Dependency kind: 
          // Normal: A dependency linked to the artifact produced by this crate.
          // Build: A compile-time dependency used to build this crate (build-script or proc-macro).
          "kind": "normal"
        },
        {
          // A crate can depend on another crate with both normal and build edges.
          "index": 1,
          "kind": "build"
        }
      ]
    },
    {
      "id": "registry+https://github.com/rust-lang/crates.io-index#zerocopy@0.8.16",
      "kind": ["bin"],
      "features": [],
      "dependencies": []
    }
  ],
  // Information about rustc used to perform the compilation.
  "rustc": {
    // Compiler version
    "version": "1.86.0-nightly",
    // Compiler wrapper
    "wrapper": null,
    // Compiler workspace wrapper
    "workspace_wrapper": null,
    // Commit hash for rustc
    "commit_hash": "bef3c3b01f690de16738b1c9f36470fbfc6ac623",
    // Host target triple
    "host": "x86_64-pc-windows-msvc",
    // Verbose version string: `rustc -vV`
    "verbose_version": "rustc 1.86.0-nightly (bef3c3b01 2025-02-04)\nbinary: rustc\ncommit-hash: bef3c3b01f690de16738b1c9f36470fbfc6ac623\ncommit-date: 2025-02-04\nhost: x86_64-pc-windows-msvc\nrelease: 1.86.0-nightly\nLLVM version: 19.1.7\n"
  }
}
```

## update-breaking

* Tracking Issue: [#12425](https://github.com/rust-lang/cargo/issues/12425)

允许使用 `--breaking` 标志，将 `Cargo.toml` 中的依赖版本要求升级到 SemVer 不兼容版本。

这仅适用于满足以下条件的依赖：
- 该包是某个 workspace 成员的依赖
- 该依赖未被重命名
- 存在 SemVer 不兼容的新版本
- 使用了 “SemVer operator”（`^`，即默认值）

用户还可通过命令行显式指定包名，以进一步限制哪些包会被升级。

示例：
```console
$ cargo +nightly -Zunstable-options update --breaking
$ cargo +nightly -Zunstable-options update --breaking clap
```

*其目标是扮演与 [cargo-upgrade](https://github.com/killercup/cargo-edit/) 类似的角色。*

## build-std
* Tracking Repository: <https://github.com/rust-lang/wg-cargo-std-aware>

`build-std` 特性允许 Cargo 在 crate 图编译的一部分中自行编译标准库。
它历史上也被称为 “std-aware Cargo”。
该特性仍处于非常早期的开发阶段，也可能成为 Cargo 的一项大型功能扩展。
即使只按当前最简形式记录它，也已经是很大的文档量；因此如果你想持续关注进展，应跟踪 [tracking repository](https://github.com/rust-lang/wg-cargo-std-aware) 及其相关 issues。

当前已实现的功能由 `-Z build-std` 标志启用。
该标志表示 Cargo 应使用与主构建相同的 profile，从源码编译标准库。
注意：要让它工作，你必须能获取标准库源码；目前唯一受支持的方式是安装 `rust-src` rustup 组件：

```console
$ rustup component add rust-src --toolchain nightly
```

Usage looks like:

```console
$ cargo new foo
$ cd foo
$ cargo +nightly run -Z build-std --target x86_64-unknown-linux-gnu
   Compiling core v0.0.0 (...)
   ...
   Compiling foo v0.1.0 (...)
    Finished dev [unoptimized + debuginfo] target(s) in 21.00s
     Running `target/x86_64-unknown-linux-gnu/debug/foo`
Hello, world!
```

这里我们以 debug 模式、开启 debug assertions 重新编译了标准库
（类似 `src/main.rs` 的编译方式），并在最后把所有内容链接到一起。

使用 `-Z build-std` 会隐式编译稳定 crate `core`、`std`、`alloc` 和 `proc_macro`。
若使用 `cargo test`，还会编译 `test` crate。
如果你的环境不支持其中某些 crate，也可以给 `-Zbuild-std` 传入参数：

```console
$ cargo +nightly build -Z build-std=core,alloc
```

这里的值是要构建的标准库 crate 的逗号分隔列表。

### Requirements

简而言之，当前使用 `-Z build-std` 的要求如下：

* 必须通过 `rustup component add rust-src` 安装 libstd 源码
* 必须同时使用 nightly Cargo 和 nightly rustc
* 所有 `cargo` 调用都必须传入 `-Z build-std`

### Reporting bugs and helping out

`-Z build-std` 特性仍处于非常早期的开发阶段！
这个 Cargo 特性历史很长、范围很大，而现在才刚开始。
如果你想反馈 bug，请提交到：

* Cargo --- <https://github.com/rust-lang/cargo/issues/new> --- 用于实现 bug
* 跟踪仓库 ---
  <https://github.com/rust-lang/wg-cargo-std-aware/issues/new> --- 用于更大的设计问题。

如果你想查看尚未实现的功能，或某些行为不如预期，也可以查看跟踪仓库的
[issue tracker](https://github.com/rust-lang/wg-cargo-std-aware/issues)；
如果没有对应 issue，请直接新建。

## build-std-features
* Tracking Repository: <https://github.com/rust-lang/wg-cargo-std-aware>

该 flag 与 `-Zbuild-std` 同级，用于配置构建标准库时标准库自身启用的 feature。
当前默认启用的 feature 是 `backtrace` 和 `panic-unwind`。
该 flag 接受逗号分隔列表，若提供则会覆盖默认启用列表。

## binary-dep-depinfo
* Tracking rustc issue: [#63012](https://github.com/rust-lang/rust/issues/63012)

`-Z binary-dep-depinfo` 会让 Cargo 将同名 flag 转发给 `rustc`，
从而让 rustc 在 “dep info” 文件（`.d` 扩展名）中包含所有二进制依赖的路径。
Cargo 随后会利用这些信息进行变更检测（如果任何二进制依赖变化，就会重建该 crate）。
其主要用例是构建编译器本身，因为它对标准库有隐式依赖，而这些依赖本来无法被变更检测追踪。

## checksum-freshness
* Tracking issue: [#14136](https://github.com/rust-lang/cargo/issues/14136)

`-Z checksum-freshness` 会把 Cargo fingerprints 中使用的文件 mtime 改为文件校验和值。
这在 mtime 实现较差的系统，或 CI/CD 中最有用。
校验和算法可能在 Cargo 版本间无预警变更。
Cargo 通过 fingerprints 决定 crate 何时需要重建。

目前，即使启用了 `checksum-freshness`，构建脚本摄取的文件仍会继续使用 mtime。
这并非长期方案。

## panic-abort-tests
* Tracking Issue: [#67650](https://github.com/rust-lang/rust/issues/67650)
* Original Pull Request: [#7460](https://github.com/rust-lang/cargo/pull/7460)

`-Z panic-abort-tests` 允许在 nightly 下把测试 harness crate 编译为 `-Cpanic=abort`。
没有这个 flag 时，Cargo 会把测试以及其所有依赖都用 `-Cpanic=unwind` 编译，
因为这是 `test` crate 目前唯一知道如何工作的方式。
不过从 [rust-lang/rust#64158] 开始，`test` crate 已支持以“每个测试进程一个”方式使用 `-C panic=abort`，
这有助于避免重复编译 crate 图。

目前尚不清楚 Cargo 会如何稳定这个特性，但我们希望最终能以某种方式稳定它。

[rust-lang/rust#64158]: https://github.com/rust-lang/rust/pull/64158

## target-applies-to-host
* Original Pull Request: [#9322](https://github.com/rust-lang/cargo/pull/9322)
* Tracking Issue: [#9453](https://github.com/rust-lang/cargo/issues/9453)

历史上，Cargo 对来自环境变量和 [`[target]`](config.md#target) 的 `linker`、`rustflags`
配置，在构建脚本、插件以及其他*始终为主机平台构建*的产物上是否生效，行为并不一致。
当未传 `--target` 时，Cargo 会让构建脚本像其他编译产物一样使用同一组 `linker` 和 `rustflags`。
但当传入 `--target` 时，Cargo 会使用 [`[target.<host triple>]`](config.md#targettriplelinker) 中的 `linker`，
并且不会读取任何 `rustflags` 配置。
这种双重行为既令人困惑，也让“宿主 triple 与 [target triple] 恰好相同，但运行在构建宿主上的产物仍需不同配置”的场景更难正确配置。

`-Ztarget-applies-to-host` 会启用 Cargo 配置文件中的顶层 `target-applies-to-host` 设置，
让用户可以选择更一致的行为。
当 `target-applies-to-host` 未设置或设为 `true` 时，Cargo 保持现有行为（但请注意 `-Zhost-config` 会改变该默认值）。
当设为 `false` 时，无论是否传入 `--target`，`[target.<host triple>]`、`RUSTFLAGS` 或 `[build]` 的选项都不会再作用于主机产物。
若要自定义在主机上运行的产物，请使用 `[host]`（[`host-config`](#host-config)）。

未来，`target-applies-to-host` 可能会默认变为 `false`，以提供更合理且一致的默认行为。

```toml
# config.toml
target-applies-to-host = false
```

```console
cargo +nightly -Ztarget-applies-to-host build --target x86_64-unknown-linux-gnu
```

## host-config
* Original Pull Request: [#9322](https://github.com/rust-lang/cargo/pull/9322)
* Tracking Issue: [#9452](https://github.com/rust-lang/cargo/issues/9452)

配置文件中的 `host` 键可用于向主机构建目标传递参数，例如在交叉编译时必须运行于宿主系统而非目标系统的构建脚本。
它同时支持通用表与按宿主架构划分的专用表，匹配到的宿主架构表优先于通用宿主表。

它要求同时设置 `-Zhost-config` 与 `-Ztarget-applies-to-host`
命令行选项，并且在 Cargo 配置文件中设定 `target-applies-to-host = false`。

```toml
# config.toml
[host]
linker = "/path/to/host/linker"
runner = "host-runner"
[host.x86_64-unknown-linux-gnu]
linker = "/path/to/host/arch/linker"
runner = "host-arch-runner"
rustflags = ["-Clink-arg=--verbose"]
[target.x86_64-unknown-linux-gnu]
linker = "/path/to/target/linker"
```

`host.runner` 设置会包裹主机构建目标的执行，比如构建脚本；类似于 `target.<triple>.runner` 包裹 `cargo run`/`test`/`bench`。

当在 `x86_64-unknown-linux-gnu` 宿主上构建时，上面的通用 `host` 表会被完全忽略，因为 `host.x86_64-unknown-linux-gnu` 表优先。

设置 `-Zhost-config` 会把 `target-applies-to-host` 的默认值从 `true` 改为 `false`。

```console
cargo +nightly -Ztarget-applies-to-host -Zhost-config build --target x86_64-unknown-linux-gnu
```

## unit-graph
* Tracking Issue: [#8002](https://github.com/rust-lang/cargo/issues/8002)

`--unit-graph` 可以传给任意构建命令（`build`、`check`、`run`、`test`、`bench`、`doc` 等），
它会向 stdout 输出一个表示 Cargo 内部 unit 图的 JSON 对象。
实际上不会执行构建，命令在打印后立即返回。
每个 “unit” 对应一次编译器执行；这些对象还会包含每个 unit 依赖哪些其他 unit。

```
cargo +nightly build --unit-graph -Z unstable-options
```

该结构比 Cargo 视角下的依赖关系更完整。
特别是其中的 `features` 字段支持新的 feature resolver，在该模式下一个依赖可因不同 feature 被构建多次。
`cargo metadata` 从根本上无法表示不同依赖类型之间的 feature 关系，而 feature 现在还取决于运行了哪个命令、选择了哪些包与目标。
此外，它还能提供包内依赖细节，例如构建脚本或测试。

下面是 JSON 结构说明：

```javascript
{
  /* Version of the JSON output structure. If any backwards incompatible
     changes are made, this value will be increased.
  */
  "version": 1,
  /* Array of all build units. */
  "units": [
    {
      /* An opaque string which indicates the package.
         Information about the package can be obtained from `cargo metadata`.
      */
      "pkg_id": "my-package 0.1.0 (path+file:///path/to/my-package)",
      /* The Cargo target. See the `cargo metadata` documentation for more
         information about these fields.
         https://doc.rust-lang.org/cargo/commands/cargo-metadata.html
      */
      "target": {
        "kind": ["lib"],
        "crate_types": ["lib"],
        "name": "my_package",
        "src_path": "/path/to/my-package/src/lib.rs",
        "edition": "2018",
        "test": true,
        "doctest": true
      },
      /* The profile settings for this unit.
         These values may not match the profile defined in the manifest.
         Units can use modified profile settings. For example, the "panic"
         setting can be overridden for tests to force it to "unwind".
      */
      "profile": {
        /* The profile name these settings are derived from. */
        "name": "dev",
        /* The optimization level as a string. */
        "opt_level": "0",
        /* The LTO setting as a string. */
        "lto": "false",
        /* The codegen units as an integer.
           `null` if it should use the compiler's default.
        */
        "codegen_units": null,
        /* The debug information level as an integer.
           `null` if it should use the compiler's default (0).
        */
        "debuginfo": 2,
        /* Whether or not debug-assertions are enabled. */
        "debug_assertions": true,
        /* Whether or not overflow-checks are enabled. */
        "overflow_checks": true,
        /* Whether or not rpath is enabled. */
        "rpath": false,
        /* Whether or not incremental is enabled. */
        "incremental": true,
        /* The panic strategy, "unwind" or "abort". */
        "panic": "unwind"
      },
      /* Which platform this target is being built for.
         A value of `null` indicates it is for the host.
         Otherwise it is a string of the target triple (such as
         "x86_64-unknown-linux-gnu").
      */
      "platform": null,
      /* The "mode" for this unit. Valid values:

         * "test" --- Build using `rustc` as a test.
         * "build" --- Build using `rustc`.
         * "check" --- Build using `rustc` in "check" mode.
         * "doc" --- Build using `rustdoc`.
         * "doctest" --- Test using `rustdoc`.
         * "run-custom-build" --- Represents the execution of a build script.
      */
      "mode": "build",
      /* Array of features enabled on this unit as strings. */
      "features": ["somefeat"],
      /* Whether or not this is a standard-library unit,
         part of the unstable build-std feature.
         If not set, treat as `false`.
      */
      "is_std": false,
      /* Array of dependencies of this unit. */
      "dependencies": [
        {
          /* Index in the "units" array for the dependency. */
          "index": 1,
          /* The name that this dependency will be referred as. */
          "extern_crate_name": "unicode_xid",
          /* Whether or not this dependency is "public",
             part of the unstable public-dependency feature.
             If not set, the public-dependency feature is not enabled.
          */
          "public": false,
          /* Whether or not this dependency is injected into the prelude,
             currently used by the build-std feature.
             If not set, treat as `false`.
          */
          "noprelude": false
        }
      ]
    },
    // ...
  ],
  /* Array of indices in the "units" array that are the "roots" of the
     dependency graph.
  */
  "roots": [0],
}
```

## Profile `rustflags` option
* Original Issue: [rust-lang/cargo#7878](https://github.com/rust-lang/cargo/issues/7878)
* Tracking Issue: [rust-lang/cargo#10271](https://github.com/rust-lang/cargo/issues/10271)

该特性在 `[profile]` 段新增了可直接传给 rustc 的参数选项。
可按如下方式启用：

```toml
cargo-features = ["profile-rustflags"]

[package]
# ...

[profile.release]
rustflags = [ "-C", "..." ]
```

若要在 Cargo 配置的 profile 中设置该项，需要通过
`-Z profile-rustflags` 或 `[unstable]` 表启用。例如：

```toml
# .cargo/config.toml
[unstable]
profile-rustflags = true

[profile.release]
rustflags = [ "-C", "..." ]
```

## Profile `hint-mostly-unused` option
* Tracking Issue: [#15644](https://github.com/rust-lang/cargo/issues/15644)

该特性在 `[profile]` 段新增了用于启用 rustc `hint-mostly-unused` 的选项。
它主要适合对特定依赖启用：

```toml
[profile.dev.package.huge-mostly-unused-dependency]
hint-mostly-unused = true
```

要启用该特性，请传入 `-Zprofile-hint-mostly-unused`。
不过该选项只是提示；如果未传这个 flag，Cargo 只会给出 warning 并忽略该 profile 选项。
早于该特性引入版本的 Cargo 会给出 “unused manifest key” 警告，但不会报错。
这意味着你可以在 crate 的 `Cargo.toml` 里写这个提示，而不强制要求所有构建方都升级到更新的 Cargo。

crate 还可以通过 `[hints]` 表为依赖它的 crate 自动提供该提示
（老版本 Cargo 同样会忽略此表）：

```toml
[hints]
mostly-unused = true
```

这会让该 crate 默认启用 hint-mostly-unused。
若 `profile` 中有显式设置，则以 `profile` 为准（其优先级更高，且只能在顶层被构建 crate 中指定）。

## rustdoc-map
* Tracking Issue: [#8296](https://github.com/rust-lang/cargo/issues/8296)

该特性增加会传给 `rustdoc` 的配置项：
当依赖未在本地生成文档时，它可为“托管在其他位置”的依赖文档生成链接。
先在 `.cargo/config` 中添加：

```toml
[doc.extern-map.registries]
crates-io = "https://docs.rs/"
```

然后构建文档时使用以下参数，让依赖链接指向 [docs.rs](https://docs.rs/)：

```
cargo +nightly doc --no-deps -Zrustdoc-map
```

`registries` 表是“注册表名 -> 链接 URL”的映射。
URL 可包含 `{pkg_name}` 和 `{version}` 占位符，Cargo 会替换为实际值。
若两者都未出现，Cargo 默认会在 URL 末尾追加 `{pkg_name}/{version}/`。

另一个配置项可用于重定向标准库链接。
默认情况下，rustdoc 会链接到 <https://doc.rust-lang.org/nightly/>。
要修改该行为，请使用 `doc.extern-map.std`：

```toml
[doc.extern-map]
std = "local"
```

值为 `"local"` 表示链接到 `rustc` sysroot 中的本地文档。
若你使用 rustup，可通过 `rustup component add rust-docs` 安装该文档。

默认值是 `"remote"`。

该值也可以是自定义 URL。

## per-package-target
* Tracking Issue: [#9406](https://github.com/rust-lang/cargo/pull/9406)
* Original Pull Request: [#9030](https://github.com/rust-lang/cargo/pull/9030)
* Original Issue: [#7004](https://github.com/rust-lang/cargo/pull/7004)

`per-package-target` 特性给 manifest 增加了两个键：
`package.default-target` 和 `package.forced-target`。
前者用于指定该包“默认”（即未传 `--target`）要编译到的目标；
后者用于指定该包“始终”编译到的目标。

示例：

```toml
[package]
forced-target = "wasm32-unknown-unknown"
```

在此示例中，crate 会始终构建为 `wasm32-unknown-unknown`，
例如因为它将作为插件供运行在宿主（或命令行指定）目标上的主程序使用。

## artifact-dependencies

* Tracking Issue: [#9096](https://github.com/rust-lang/cargo/pull/9096)
* Original Pull Request: [#9992](https://github.com/rust-lang/cargo/pull/9992)

Artifact dependencies 允许 Cargo 包依赖 `bin`、`cdylib`、`staticlib` 类型 crate，
并在编译期使用这些 crate 生成的产物。

使用 `-Z bindeps` 运行 `cargo` 以启用该功能。

### artifact-dependencies: Dependency declarations

Artifact-dependencies 为 `Cargo.toml` 依赖声明增加了以下键：

- `artifact` --- 指定要构建的 [Cargo Target](cargo-targets.md)。
  通常没有该字段时，Cargo 只会构建依赖的 `[lib]` 目标。
  该字段允许指定应构建哪个目标，并在构建期以二进制产物形式提供：

  * `"bin"` --- 编译可执行二进制，对应依赖 manifest 中所有 `[[bin]]` 段。
  * `"bin:<bin-name>"` --- 编译指定名称 `<bin-name>` 的单个二进制目标。
  * `"cdylib"` --- C 兼容动态库，对应依赖 manifest 中 `[lib]` 的 `crate-type = ["cdylib"]`。
  * `"staticlib"` --- C 兼容静态库，对应依赖 manifest 中 `[lib]` 的 `crate-type = ["staticlib"]`。

  `artifact` 可以是字符串，也可以是字符串数组以指定多个目标。

  Example:

  ```toml
  [dependencies]
  bar = { version = "1.0", artifact = "staticlib" }
  zoo = { version = "1.0", artifact = ["bin:cat", "bin:dog"]}
  ```

- `lib` --- 布尔值，表示是否额外把该依赖库按普通 Rust `lib` 依赖构建。
  该字段只能在指定了 `artifact` 时使用。

  当指定 `artifact` 时，该字段默认是 `false`。
  若设为 `true`，则依赖的 `[lib]` 目标也会按声明包的构建平台目标构建。
  这样包就能在使用 artifact dependency 的同时，也像普通依赖一样在 Rust 代码中使用该依赖。

  Example:

  ```toml
  [dependencies]
  bar = { version = "1.0", artifact = "bin", lib = true }
  ```

- `target` --- 构建该依赖所用的平台目标。
  该字段只能在指定了 `artifact` 时使用。

  若未指定该字段，其默认值取决于依赖类型。
  对 build-dependencies，会为 host target 构建；
  对其他依赖，会为声明包构建时使用的同一目标构建。

  对 build dependency，该字段还可取特殊值 `"target"`，表示按“包当前构建目标”来构建该依赖。

  ```toml
  [build-dependencies]
  bar = { version = "1.0", artifact = "cdylib", target = "wasm32-unknown-unknown"}
  same-target = { version = "1.0", artifact = "bin", target = "target" }
  ```

### artifact-dependencies: Environment variables

构建完 artifact dependency 后，Cargo 会提供以下环境变量用于访问产物：

- `CARGO_<ARTIFACT-TYPE>_DIR_<DEP>` --- 该依赖所有产物所在目录。

  `<ARTIFACT-TYPE>` 是依赖声明中的 `artifact`（大写形式，如 `CDYLIB`、`STATICLIB`、`BIN`），`<DEP>` 是依赖名。
  与其他 Cargo 环境变量一致，依赖名会转为大写，连字符会替换为下划线。

  若 manifest 对依赖做了重命名，`<DEP>` 对应你指定的名称，而不是原包名。

- `CARGO_<ARTIFACT-TYPE>_FILE_<DEP>_<NAME>` --- 该产物的完整路径。

  `<ARTIFACT-TYPE>` 为上文同义，`<DEP>` 为转换后的依赖名，`<NAME>` 为该依赖产物名。

  注意 `<NAME>` 不会做任何改写：若产物提供 crate 指定了 `name` 就用该值，否则用 crate 名；因此它可能是小写，也可能包含连字符。

  为方便起见，若产物名与原包名一致，Cargo 还会额外提供一个去掉 `_<NAME>` 后缀的同义变量。
  例如 `cmake` crate 提供名为 `cmake` 的二进制时，Cargo 会同时提供 `CARGO_BIN_FILE_CMAKE` 和 `CARGO_BIN_FILE_CMAKE_cmake`。

针对不同依赖类型，这些变量会注入到其对应可访问该依赖的构建阶段：

- 对 build-dependencies：变量提供给 `build.rs`，可通过 [`std::env::var_os`](https://doc.rust-lang.org/std/env/fn.var_os.html) 读取。
  （与任意 OS 文件路径一样，它们可能不是合法 UTF-8。）
- 对普通 dependencies：变量在 crate 编译期间提供，可通过 [`env!`] 宏读取。
- 对 dev-dependencies：变量在 example/test/benchmark 编译期间提供，也可通过 [`env!`] 宏读取。

[`env!`]: https://doc.rust-lang.org/std/macro.env.html

### artifact-dependencies: Examples

#### Example: use a binary executable from a build script

在 `Cargo.toml` 中，你可以声明一个二进制依赖供构建脚本使用：

```toml
[build-dependencies]
some-build-tool = { version = "1.0", artifact = "bin" }
```

然后在构建脚本中即可在构建期执行该二进制：

```rust
fn main() {
    let build_tool = std::env::var_os("CARGO_BIN_FILE_SOME_BUILD_TOOL").unwrap();
    let status = std::process::Command::new(build_tool)
        .arg("do-stuff")
        .status()
        .unwrap();
    if !status.success() {
        eprintln!("failed!");
        std::process::exit(1);
    }
}
```

#### Example: use _cdylib_ artifact in build script

消费方包中的 `Cargo.toml`：将 `bar` 库按 `cdylib` 为指定构建目标构建……

```toml
[build-dependencies]
bar = { artifact = "cdylib", version = "1.0", target = "wasm32-unknown-unknown" }
```

……并配合 `build.rs` 中的构建脚本。

```rust
fn main() {
    wasm::run_file(std::env::var("CARGO_CDYLIB_FILE_BAR").unwrap());
}
```

#### Example: use _binary_ artifact and its library in a binary

消费方包中的 `Cargo.toml`：将 `bar` 二进制作为 artifact 引入，同时也把它作为库可用……

```toml
[dependencies]
bar = { artifact = "bin", version = "1.0", lib = true }
```

……并配合使用 `main.rs` 的可执行程序。

```rust
fn main() {
    bar::init();
    command::run(env!("CARGO_BIN_FILE_BAR"));
}
```

## publish-timeout
* Tracking Issue: [11222](https://github.com/rust-lang/cargo/issues/11222)

配置文件中的 `publish.timeout` 可用于控制 `cargo publish` 从上传包到注册表到本地索引可见之间的等待时长。

超时设为 `0` 表示不执行任何检查。当前默认值是 `60` 秒。

需要同时设置命令行选项 `-Zpublish-timeout`。

```toml
# config.toml
[publish]
timeout = 300  # in seconds
```

## asymmetric-token
* Tracking Issue: [10519](https://github.com/rust-lang/cargo/issues/10519)
* RFC: [#3231](https://github.com/rust-lang/rfcs/pull/3231)

`-Z asymmetric-token` 会启用 `cargo:paseto` 凭据提供器，使 Cargo 可在不通过网络发送密钥明文的情况下对注册表认证。

[`config.toml`](config.md) 与 `credentials.toml` 中有 `private-key` 字段，
它是使用 [`PASERK` secret 子集格式](https://github.com/paseto-standard/paserk/blob/master/types/secret.md) 的私钥，用于签名非对称令牌。

可通过 `cargo login --generate-keypair` 生成密钥对，它会：
- 按当前推荐方式生成公私钥对。
- 将私钥保存到 `credentials.toml`。
- 以 [PASERK public](https://github.com/paseto-standard/paserk/blob/master/types/public.md) 格式输出公钥。

建议将 `private-key` 存在 `credentials.toml` 中。
它也支持写在 `config.toml`，主要为了可通过对应环境变量注入；这也是 CI 场景下推荐的提供方式。
这一模式与 `token` 字段配置密钥令牌的方式一致。

还有一个可选字段 `private-key-subject`，是由注册表选择的字符串。
该字符串会被包含进非对称令牌中，因此不应视为密钥。
它适用于少见场景，例如“用密码学方式证明中央 CA 服务器授权了此次操作”。
Cargo 要求它必须是非空白可打印 ASCII；若注册表需要非 ASCII 数据，应先做 base64 编码。

这两个字段都可通过 `cargo login --registry=name --private-key --private-key-subject="subject"` 设置，命令会提示你输入密钥值。

一个注册表最多只能设置 `private-key` 或 `token` 其中之一。

所有 PASETO 都会包含 `iat`（ISO 8601 格式的当前时间）。
Cargo 还会在适用时包含以下 claim：
- `sub`：可选、非密钥字符串，由注册表选择，预期每次请求都携带。其值来自 `config.toml` 的 `private-key-subject`。
- `mutation`：若存在，表示此次请求为写操作（若不存在则视为只读）；取值必须是 `publish`、`yank`、`unyank` 之一。
  - `name`：本次请求关联的 crate 名称。
  - `vers`：本次请求关联的 crate 版本字符串。
  - `cksum`：crate 内容的 SHA256（64 位小写十六进制字符串），仅当 `mutation = publish` 时必须出现。
- `challenge`：本会话中从该服务器 401/403 响应收到的 challenge 字符串。
  发行 challenge 的注册表必须跟踪 challenge 的签发/使用状态，并且在同一有效期内同一个 challenge 只能接受一次（避免记录历史所有 challenge）。

“footer”（签名的一部分）是 UTF-8 JSON 字符串，包含：
- `url`：Cargo 获取 `config.json` 文件的 RFC 3986 合规 URL。
  - 若注册表使用 HTTP 索引，这里是所有索引查询的基准 URL。
  - 若注册表使用 GIT 索引，这里是 Cargo 克隆索引所用 URL。
- `kid`：用于签名请求的私钥标识，遵循 [PASERK IDs](https://github.com/paseto-standard/paserk/blob/master/operations/ID.md) 标准。

PASETO 会包含被签名的消息本体，因此服务器无需从请求重建原始字符串再验签。
服务器仍需校验：签名对 PASETO 中字符串有效，且该字符串内容与请求一致。
若某请求理应包含某 claim，但 PASETO 中缺失，则必须拒绝该请求。

## `cargo config`

* Original Issue: [#2362](https://github.com/rust-lang/cargo/issues/2362)
* Tracking Issue: [#9301](https://github.com/rust-lang/cargo/issues/9301)

`cargo config` 子命令用于显示 Cargo 加载的配置文件内容。
当前提供 `get` 子命令，可选择指定某个配置值进行展示。

```console
cargo +nightly -Zunstable-options config get build.rustflags
```

若未指定配置值，则会显示所有配置项。
更多可用选项见 `--help`。

## rustc `--print`

* Tracking Issue: [#9357](https://github.com/rust-lang/cargo/issues/9357)

`cargo rustc --print=VAL` 会把 `--print` 转发给 `rustc` 以提取其信息。
它会带对应的
[`--print`](https://doc.rust-lang.org/rustc/command-line-arguments.html#--print-print-compiler-information)
参数运行 rustc，并立即退出而不执行编译。
将其暴露为 Cargo 参数后，Cargo 可基于当前配置注入正确的 target 与 RUSTFLAGS。

主要用例是执行 `cargo rustc --print=cfg`，
以获取“针对正确目标且受当前 RUSTFLAGS 影响”的配置值。


## Different binary name

* Tracking Issue: [#9778](https://github.com/rust-lang/cargo/issues/9778)
* PR: [#9627](https://github.com/rust-lang/cargo/pull/9627)

`different-binary-name` 特性允许设置二进制文件名，而无需遵守 crate 名称限制。
例如 crate 名必须仅由字母数字、`-`、`_` 组成且不能为空。

`filename` 参数**不应**包含二进制扩展名，`cargo` 会自行选择并附加合适扩展名。

`filename` 参数仅在 manifest 的 `[[bin]]` 段可用。

```toml
cargo-features = ["different-binary-name"]

[package]
name =  "foo"
version = "0.0.1"

[[bin]]
name = "foo"
filename = "007bar"
path = "src/main.rs"
```

## scrape-examples

* RFC: [#3123](https://github.com/rust-lang/rfcs/pull/3123)
* Tracking Issue: [#9910](https://github.com/rust-lang/cargo/issues/9910)

`-Z rustdoc-scrape-examples` 会让 Rustdoc 在当前工作区 crate 中搜索函数调用点，
并把这些调用点纳入文档。可这样使用：

```
cargo doc -Z unstable-options -Z rustdoc-scrape-examples
```

默认情况下，Cargo 会从被文档化包的 example targets 抓取示例。
你可以通过 `doc-scrape-examples` 为每个 target 单独启用或禁用抓取，例如：

```toml
# Enable scraping examples from a library
[lib]
doc-scrape-examples = true

# Disable scraping examples from an example target
[[example]]
name = "my-example"
doc-scrape-examples = false
```

**关于 tests 的说明：** 目前在 test targets 上启用 `doc-scrape-examples` 不会生效。
从测试中抓取示例仍在开发中。

**关于 dev-dependencies 的说明：** 给库生成文档通常不需要该 crate 的 dev-dependencies。
但 example targets 需要 dev-deps。为保持向后兼容，`-Z rustdoc-scrape-examples` *不会* 给 `cargo doc` 引入 dev-deps 要求。
因此在以下条件下，不会从 example targets 抓取示例：

1. No target being documented requires dev-deps, AND
2. At least one crate with targets being documented has dev-deps, AND
3. The `doc-scrape-examples` parameter is unset or false for all `[[example]]` targets.

如果你希望从 example targets 抓取示例，就必须不满足上述条件之一。
例如，你可以把某个 example target 的 `doc-scrape-examples` 设为 true，以告知 Cargo：你接受 `cargo doc` 构建 dev-deps。

## output-format for rustdoc

* Tracking Issue: [#13283](https://github.com/rust-lang/cargo/issues/13283)

该参数用于决定 `cargo rustdoc` 的输出格式，可选 `html` 或 `json`，
为工具链提供利用 [rustdoc 实验性 JSON 格式](https://doc.rust-lang.org/nightly/nightly-rustc/rustdoc_json_types/) 的方式。

可这样使用该标志：

```
cargo rustdoc -Z unstable-options --output-format json
```

## codegen-backend

`codegen-backend` 特性允许通过 profile 选择 rustc 使用的代码生成后端。

示例：

```toml
[package]
name = "foo"

[dependencies]
serde = "1.0.117"

[profile.dev.package.foo]
codegen-backend = "cranelift"
```

若要在 Cargo 配置里的 profile 中设置它，需要通过 `-Z codegen-backend` 或 `[unstable]` 表启用。例如：

```toml
# .cargo/config.toml
[unstable]
codegen-backend = true

[profile.dev.package.foo]
codegen-backend = "cranelift"
```

## gitoxide

* Tracking Issue: [#11813](https://github.com/rust-lang/cargo/issues/11813)

启用 `gitoxide` 不稳定特性后，全部或指定的 git 操作会由 `gitoxide` crate 执行，而不是 `git2`。

`-Zgitoxide` 会启用当前所有已实现功能；你也可以使用 `-Zgitoxide=operation[,operationN]` 单独选择由 `gitoxide` 执行的操作。

可用操作如下：

* `fetch` - 所有 fetch 都由 `gitoxide` 完成，包括 git 依赖和 crates 索引。
* `checkout` *(planned)* - 检出工作树，支持过滤器和子模块。

## git

* Tracking Issue: [#13285](https://github.com/rust-lang/cargo/issues/13285)

启用 `git` 不稳定特性后，`gitoxide` 与 `git2` 都会对 crate 索引和 git 依赖执行浅抓取（shallow fetch）。

`-Zgit` 会启用当前所有已实现功能；你也可以通过 `-Zgit=operation[,operationN]` 单独选择何时执行浅抓取。

可用操作如下：

* `shallow-index` - 对索引执行浅克隆。
* `shallow-deps` - 对 git 依赖执行浅克隆。

**浅克隆细节**

* 启用浅克隆：拉取 git 依赖用 `-Zgit=shallow-deps`，拉取注册表索引用 `-Zgit=shallow-index`。
* 浅克隆与浅检出的 git 仓库会放在带 `-shallow` 后缀的独立目录中，即：
  - `~/.cargo/registry/index/*-shallow`
  - `~/.cargo/git/db/*-shallow`
  - `~/.cargo/git/checkouts/*-shallow`
* 启用该不稳定特性后，抓取/克隆 git 仓库始终采用浅抓取，近似等价于处处执行 `git fetch --depth 1`。
* 即便存在 `Cargo.lock` 或指定提交 `{ rev = "…" }`，gitoxide 与 libgit2 仍会尽量浅抓取，而不会把已有仓库“去浅化”（unshallow）。

## script

* Tracking Issue: [#12207](https://github.com/rust-lang/cargo/issues/12207)

Cargo 可直接运行 `.rs` 文件：
```console
$ cargo +nightly -Zscript file.rs
```
其中 `file.rs` 可以简单到：
```rust
fn main() {}
```

用户还可以选择在模块级注释中，通过 `cargo` 代码围栏嵌入 manifest，例如：
````rust
#!/usr/bin/env -S cargo +nightly -Zscript
---cargo
[dependencies]
clap = { version = "4.2", features = ["derive"] }
---

use clap::Parser;

#[derive(Parser, Debug)]
#[clap(version)]
struct Args {
    #[clap(short, long, help = "Path to config")]
    config: Option<std::path::PathBuf>,
}

fn main() {
    let args = Args::parse();
    println!("{:?}", args);
}
````

### Single-file packages

除现有多文件包（`Cargo.toml` + 其他 `.rs` 文件）外，我们新增“单文件包”概念，它可以包含嵌入式 manifest。
单文件 `.rs` 包与普通 `.rs` 文件在文件形式上没有强制区分。

可通过 `--manifest-path` 指定单文件包，例如 `cargo test --manifest-path foo.rs`。
与 `Cargo.toml` 不同，这类文件不会被自动发现。

单文件包可包含嵌入式 manifest。
嵌入式 manifest 以 `TOML` 存放在 Rust “frontmatter” 中，也就是文件顶部 info string 以 `cargo` 开头的 Markdown 代码围栏。

推断/默认 manifest 字段：
- `package.name = <slugified file stem>`
- `package.edition = <current>` to avoid always having to add an embedded
  这样可避免总是写嵌入式 manifest，但代价是 Rust 升级时脚本可能受影响
  - 当 `edition` 未指定时给出 warning，以提醒这一点

不允许的 manifest 字段：
- `[workspace]`, `[lib]`, `[[bin]]`, `[[example]]`, `[[test]]`, `[[bench]]`
- `package.workspace`, `package.build`, `package.links`, `package.autolib`, `package.autobins`, `package.autoexamples`, `package.autotests`, `package.autobenches`

单文件包默认 `CARGO_TARGET_DIR` 位于 `$CARGO_HOME/target/<hash>`：
- Avoid conflicts from multiple single-file packages being in the same directory
- Avoid problems with the single-file package's parent directory being read-only
- Avoid cluttering the user's directory

单文件包的 lockfile 会放在 `CARGO_TARGET_DIR`。
未来支持 workspace 后，这将允许用户获得持久化 lockfile。

### Manifest-commands

你可以不带子命令，直接把 manifest 传给 `cargo`，例如 `foo/Cargo.toml` 或 `foo.rs` 这类单文件包。
这主要用于写在 `#!` 行中。

`cargo <subcommand>` 的解释优先级为：
1. Built-in xor single-file packages
2. Aliases
3. External subcommands

若参数满足以下任一条件，会被识别为 manifest-command：
- Path separators
- A `.rs` extension
- The file name is `Cargo.toml`

`cargo run --manifest-path <path>` 与 `cargo <path>` 的区别：
- `cargo <path>` 使用 `<path>` 对应配置而非当前目录配置，行为更接近 `cargo install --path <path>`。
- `cargo <path>` 的默认日志级别低于常规默认值；传 `-v` 可恢复常规输出。

运行带嵌入式 manifest 的包时，
[`arg0`](https://doc.rust-lang.org/std/os/unix/process/trait.CommandExt.html#tymethod.arg0) 将是脚本路径。
若要获取可执行文件路径，见 [`current_exe`](https://doc.rust-lang.org/std/env/fn.current_exe.html)。

### Documentation Updates

## Profile `trim-paths` option

* Tracking Issue: [rust-lang/cargo#12137](https://github.com/rust-lang/cargo/issues/12137)
* Tracking Rustc Issue: [rust-lang/rust#111540](https://github.com/rust-lang/rust/issues/111540)

该特性新增了 profile 配置，用于控制生成二进制中路径的清理方式。
可按如下方式启用：

```toml
cargo-features = ["trim-paths"]

[package]
# ...

[profile.release]
trim-paths = ["diagnostics", "object"]
```

若要在 Cargo 配置中的 profile 里设置它，
需要使用 `-Z trim-paths` 或 `[unstable]` 表启用。
例如：

```toml
# .cargo/config.toml
[unstable]
trim-paths = true

[profile.release]
trim-paths = ["diagnostics", "object"]
```

### Documentation updates

#### trim-paths

*as a new ["Profiles settings" entry](./profiles.html#profile-settings)*

`trim-paths` is a profile setting which enables and controls the sanitization of file paths in build outputs.
它支持以下取值：

- `"none"` and `false` --- disable path sanitization
- `"macro"` --- sanitize paths in the expansion of `std::file!()` macro.
    This is where paths in embedded panic messages come from
- `"diagnostics"` --- sanitize paths in printed compiler diagnostics
- `"object"` --- sanitize paths in compiled executables or libraries
- `"all"` and `true` --- sanitize paths in all possible locations

它也接受由 `"macro"`、`"diagnostics"`、`"object"` 组合而成的数组值。

`dev` profile 默认是 `none`，`release` profile 默认是 `object`。
你可以在 `Cargo.toml` 中手动覆盖：

```toml
[profile.dev]
trim-paths = "all"

[profile.release]
trim-paths = ["object", "diagnostics"]
```

默认 `release` 配置（`object`）只会清理输出的可执行/库文件中的路径。
它总会影响来自宏（如 panic 消息）的路径；
对于调试信息，仅在其与二进制一并嵌入时生效
（ELF 平台默认如此，如 Linux 和 windows-gnu）；
若调试信息在独立文件中（Windows MSVC 与 macOS 默认），则不会改写其内容，
但指向这些独立文件的路径会被清理。

如果 `trim-paths` 不是 `none` 或 `false`，则在被选中的作用域中出现以下路径时会被清理：

1. Path to the source files of the standard and core library (sysroot) will begin with `/rustc/[rustc commit hash]`,
   e.g. `/home/username/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/result.rs` ->
   `/rustc/fe72845f7bb6a77b9e671e6a4f32fe714962cec4/library/core/src/result.rs`
2. Path to the current package will be stripped, relatively to the current workspace root, e.g. `/home/username/crate/src/lib.rs` -> `src/lib.rs`.
3. Path to dependency packages will be replaced with `[package name]-[version]`. E.g. `/home/username/deps/foo/src/lib.rs` -> `foo-0.1.0/src/lib.rs`

当标准库和 core 库源码路径*不在*清理范围内时，
输出路径是否指向本地文件取决于 `rust-src` 组件是否存在。
若存在，一些路径会指向你文件系统中的源码副本；
若不存在，则会显示为 `/rustc/[rustc commit hash]/library/...`
（与被选中清理时的表现一致）。
其他源码文件路径不受影响。

这不会影响源码中硬编码的路径（例如字符串字面量中的路径）。

#### Environment variable

*as a new entry of ["Environment variables Cargo sets for build scripts"](./environment-variables.md#environment-variables-cargo-sets-for-crates)*

* `CARGO_TRIM_PATHS` --- The value of `trim-paths` profile option.
    `false`, `"none"`, and empty arrays would be converted to `none`.
    `true` and `"all"` become `all`.
    Values in a non-empty array would be joined into a comma-separated list.
    If the build script introduces absolute paths to built artifacts (such as by invoking a compiler),
    the user may request them to be sanitized in different types of artifacts.
    Common paths requiring sanitization include `OUT_DIR`, `CARGO_MANIFEST_DIR` and `CARGO_MANIFEST_PATH`,
    plus any other introduced by the build script, such as include directories.

## gc

* Tracking Issue: [#12633](https://github.com/rust-lang/cargo/issues/12633)

`-Zgc` 用于启用与 Cargo home 目录内全局缓存垃圾回收相关的功能。

#### Automatic gc configuration

`-Zgc` 会让 Cargo 读取与垃圾回收相关的额外配置项。
可用设置如下：

```toml
# Example config.toml file.

# Sub-table for defining specific settings for cleaning the global cache.
[cache.global-clean]
# Anything older than this duration will be deleted in the source cache.
max-src-age = "1 month"
# Anything older than this duration will be deleted in the compressed crate cache.
max-crate-age = "3 months"
# Any index older than this duration will be deleted from the index cache.
max-index-age = "3 months"
# Any git checkout older than this duration will be deleted from the checkout cache.
max-git-co-age = "1 month"
# Any git clone older than this duration will be deleted from the git cache.
max-git-db-age = "3 months"
```

Note that the [`cache.auto-clean-frequency`] option was stabilized in Rust 1.88.

[`cache.auto-clean-frequency`]: config.md#cacheauto-clean-frequency

### Manual garbage collection with `cargo clean`

Manual deletion can be done with the `cargo clean gc -Zgc` command.
Deletion of cache contents can be performed by passing one of the cache options:

- `--max-src-age=DURATION` --- Deletes source cache files that have not been used since the given age.
- `--max-crate-age=DURATION` --- Deletes crate cache files that have not been used since the given age.
- `--max-index-age=DURATION` --- Deletes registry indexes that have not been used since then given age (including their `.crate` and `src` files).
- `--max-git-co-age=DURATION` --- Deletes git dependency checkouts that have not been used since then given age.
- `--max-git-db-age=DURATION` --- Deletes git dependency clones that have not been used since then given age.
- `--max-download-age=DURATION` --- Deletes any downloaded cache data that has not been used since then given age.
- `--max-src-size=SIZE` --- Deletes the oldest source cache files until the cache is under the given size.
- `--max-crate-size=SIZE` --- Deletes the oldest crate cache files until the cache is under the given size.
- `--max-git-size=SIZE` --- Deletes the oldest git dependency caches until the cache is under the given size.
- `--max-download-size=SIZE` --- Deletes the oldest downloaded cache data until the cache is under the given size.

DURATION 采用 `"N seconds/minutes/days/weeks/months"` 形式，其中 N 为整数。

SIZE 采用 `"N *suffix*"` 形式，其中 *suffix* 可为 B、kB、MB、GB、kiB、MiB、GiB，N 可为整数或浮点数。若无后缀则按字节计。

```sh
cargo clean gc -Zgc
cargo clean gc -Zgc --max-download-age=1week
cargo clean gc -Zgc --max-git-size=0 --max-download-size=100MB
```

## open-namespaces

* Tracking Issue: [#13576](https://github.com/rust-lang/cargo/issues/13576)

允许多个包参与同一个 API 命名空间。

可按如下方式启用：
```toml
cargo-features = ["open-namespaces"]

[package]
# ...
```

## panic-immediate-abort

* Tracking Issue: [#16042](https://github.com/rust-lang/cargo/issues/16042)
* Upstream Tracking Issue: [rust-lang/rust#147286](https://github.com/rust-lang/rust/issues/147286)

扩展 `panic` profile 设置，支持
[`immediate-abort`](../../rustc/codegen-options/index.html#panic) panic 策略。
可按如下方式启用：

```toml
# Cargo.toml
cargo-features = ["panic-immediate-abort"]

[package]
# ...

[profile.release]
panic = "immediate-abort"
```

若要在 Cargo 配置的 profile 中设置该项，
需要通过 `-Z panic-immediate-abort` CLI 标志
或 `[unstable]` 表启用。
例如：

```toml
# .cargo/config.toml
[unstable]
panic-immediate-abort = true

[profile.release]
panic = "immediate-abort"
```

## fine-grain-locking

* Tracking Issue: [#4282](https://github.com/rust-lang/cargo/issues/4282)

使用细粒度锁，而不是锁住整个构建缓存。

注意：Fine grain locking 会隐式启用 [build-dir-new-layout](#build-dir-new-layout)，因为它构建于该目录重组之上。

## `[lints.cargo]`

* Tracking Issue: [#12235](https://github.com/rust-lang/cargo/issues/12235)

新增 `cargo` 的 `lints` 工具表，可在使用 `-Zcargo-lints` 时配置 Cargo 自身发出的 lint。
```toml
[lints.cargo]
implicit-features = "warn"
```

可与下述机制配合使用：
[RFC 2906 `workspace-deduplicate`](https://rust-lang.github.io/rfcs/2906-cargo-workspace-deduplicate.html):
```toml
[workspace.lints.cargo]
implicit-features = "warn"

[lints]
workspace = true
```

## Path Bases

* Tracking Issue: [#14355](https://github.com/rust-lang/cargo/issues/14355)

`path` 依赖可通过设置 `base` 键来指定基路径；
其值可来自[配置](config.md)中的 `[path-bases]` 表，或[内置 path base](#built-in-path-bases)。
该 base 的值会拼接到 `path` 值前面（必要时加路径分隔符），形成 Cargo 实际查找依赖的位置。

例如，如果 `Cargo.toml` 包含：

```toml
cargo-features = ["path-bases"]

[dependencies]
foo = { base = "dev", path = "foo" }
```

若配置中的 `[path-bases]` 表包含：

```toml
[path-bases]
dev = "/home/user/dev/rust/libraries/"
```

则会生成位于下列位置的 `path` 依赖 `foo`：
`/home/user/dev/rust/libraries/foo`.

Path bases can be either absolute or relative. Relative path bases are relative
to the parent directory of the configuration file that declared that path base.

path base 名称只能使用 [alphanumeric](https://doc.rust-lang.org/std/primitive.char.html#method.is_alphanumeric) 字符、`-` 或 `_`，
必须以 [alphabetic](https://doc.rust-lang.org/std/primitive.char.html#method.is_alphabetic) 字符开头，且不能为空。

如果依赖中使用的 path base 名既不在配置中，也不是内置 path base，Cargo 会报错。

#### Built-in path bases

Cargo 提供了隐式 path base，可无需在 `[path-bases]` 中显式声明直接使用。

* `workspace` - If a project is [a workspace or workspace member](workspaces.md)
则该 path base 定义为该 workspace 根 `Cargo.toml` 的父目录。

如果配置中声明了与内置同名的 path base，Cargo 会优先使用配置值。
这使得 Cargo 可以在不引发兼容性问题的情况下新增内置 path base
（因为已有同名配置会覆盖内置值）。

## native-completions
* Original Issue: [#6645](https://github.com/rust-lang/cargo/issues/6645)
* Tracking Issue: [#14520](https://github.com/rust-lang/cargo/issues/14520)

该特性将手写补全脚本迁移为 Rust 原生实现，
让我们更容易新增、扩展和测试补全功能。
该特性在 nightly 下可直接使用，无需额外 `-Z` 选项。

特别希望收到反馈的方面：
- Arguments that need escaping or quoting that aren't handled correctly
- Inaccuracies in the information
- Bugs in parsing of the command-line
- Arguments that don't report their completions
- If a known issue is being problematic

反馈可分为：
- What completion candidates are reported
  - Known issues: [#14520](https://github.com/rust-lang/cargo/issues/14520), [`A-completions`](https://github.com/rust-lang/cargo/labels/A-completions)
  - [Report an issue](https://github.com/rust-lang/cargo/issues/new) or [discuss the behavior](https://github.com/rust-lang/cargo/issues/14520)
- Shell integration, command-line parsing, and completion filtering
  - Known issues: [clap#3166](https://github.com/clap-rs/clap/issues/3166), [clap's `A-completions`](https://github.com/clap-rs/clap/labels/A-completion)
  - [Report an issue](https://github.com/clap-rs/clap/issues/new/choose) or [discuss the behavior](https://github.com/clap-rs/clap/discussions/new/choose)

如有疑问，可在 [#14520](https://github.com/rust-lang/cargo/issues/14520) 或 [zulip](https://rust-lang.zulipchat.com/#narrow/stream/246057-t-cargo) 讨论。

### How to use native-completions feature:
- bash:
  Add `source <(CARGO_COMPLETE=bash cargo +nightly)` to `~/.local/share/bash-completion/completions/cargo`.

- zsh:
  Add `source <(CARGO_COMPLETE=zsh cargo +nightly)` to your `.zshrc`.
  
- fish:
  Add `source (CARGO_COMPLETE=fish cargo +nightly | psub)` to `$XDG_CONFIG_HOME/fish/completions/cargo.fish`

- elvish:
  Add `eval (E:CARGO_COMPLETE=elvish cargo +nightly | slurp)` to `$XDG_CONFIG_HOME/elvish/rc.elv`

- powershell:
  Add `CARGO_COMPLETE=powershell cargo +nightly | Invoke-Expression` to `$PROFILE`.

## feature unification

* RFC: [#3692](https://github.com/rust-lang/rfcs/blob/master/text/3692-feature-unification.md)
* Tracking Issue: [#14774](https://github.com/rust-lang/cargo/issues/14774)

`-Z feature-unification` 会启用 `resolver.feature-unification` 配置项，
用于控制工作区内 feature 的统一方式。
如果未启用该不稳定标志，则 `resolver.feature-unification` 配置会被忽略。

### `resolver.feature-unification`

* Type: string
* Default: `"selected"`
* Environment: `CARGO_RESOLVER_FEATURE_UNIFICATION`

指定哪些包参与 [feature unification](../reference/features.html#feature-unification)。

* `selected`: Merge dependency features from all packages specified for the current build.
* `workspace`: Merge dependency features across all workspace members,
  regardless of which packages are specified for the current build.
* `package`: Dependency features are considered on a package-by-package basis,
  preferring duplicate builds of dependencies when different sets of features are activated by the packages.

## lockfile-publish-time

* Original Issue: [#5221](https://github.com/rust-lang/cargo/issues/5221)
* Tracking Issue: [#16271](https://github.com/rust-lang/cargo/issues/16271)

使用 `cargo generate-lockfile -Zunstable-options --publish-time <time>` 时，
依赖解析不会考虑晚于指定时间发布的包。

## Package message format

* Original Issue: [#11666](https://github.com/rust-lang/cargo/issues/11666)
* Tracking Issue: [#15353](https://github.com/rust-lang/cargo/issues/15353)

`cargo package` 中的 `--message-format` 用于控制输出消息格式。
当前仅与 `--list` 一起生效，并影响文件列表格式。
需要 `-Zunstable-options`。
更多信息见 [`cargo package --message-format`](../commands/cargo-package.md#option-cargo-package---message-format)。

## rustdoc depinfo

* Original Issue: [#12266](https://github.com/rust-lang/cargo/issues/12266)
* Tracking Issue: [#15370](https://github.com/rust-lang/cargo/issues/15370)

`-Z rustdoc-depinfo` 会利用 rustdoc 的 dep-info 文件来判断文档是否需要重新生成。
它可以与 `-Z checksum-freshness` 结合使用，以通过校验和变化而不是文件 mtime 来检测变更。

## no-embed-metadata
* Original Pull Request: [#15378](https://github.com/rust-lang/cargo/pull/15378)
* Tracking Issue: [#15495](https://github.com/rust-lang/cargo/issues/15495)

Rust 的默认行为是将 crate 元数据嵌入到 `rlib` 与 `dylib` 产物中。
由于 Cargo 也会对这些中间产物传入 `--emit=metadata` 以支持流水线编译，
这会导致大量元数据在磁盘上重复，占用 target 目录空间。

该特性会让 Cargo 向编译器传入 `-Zembed-metadata=no`，
从而指示其不要在 rlib/dylib 产物中嵌入元数据。
此时元数据只会保存在 `.rmeta` 文件中。

```console
cargo +nightly -Zno-embed-metadata build
```

## `unstable-editions`

`cargo-features` 列表中的 `unstable-editions` 值允许 `Cargo.toml` 声明尚未稳定的 edition。

```toml
cargo-features = ["unstable-editions"]

[package]
name = "my-package"
edition = "future"
```

当引入新的 edition 时，在该 edition 稳定前都需要 `unstable-editions` 特性。

特殊的 `"future"` edition 是为开发中的新功能准备的永久不稳定占位版。
`future` edition 本身不会引入新行为；其中的每项变更都需要显式 opt-in，例如使用 `#![feature(...)]` 属性。

## `fix-edition`

`-Zfix-edition` 是一个永久不稳定 flag，用于辅助测试 edition 迁移，尤其适合配合 crater 使用。
它只对 `cargo fix` 子命令生效，并有两种形式：

- `-Zfix-edition=start=$INITIAL` --- 该形式会检查当前 edition 是否等于给定编号；若不是，则直接成功退出（因为我们想忽略更旧的 edition）；若相等，则运行等价于 `cargo check` 的操作。它用于配合 crater 的 “start” 工具链，为 “before” 工具链建立基线。
- `-Zfix-edition=end=$INITIAL,$NEXT` --- 该形式会检查当前 edition 是否等于给定的 `$INITIAL`；若不是，则直接成功退出；若相等，则执行迁移到 `$NEXT` 指定 edition。随后它会修改 `Cargo.toml`，加入相应的 `cargo-features = ["unstable-edition"]`，更新 `edition` 字段，并运行等价于 `cargo check` 的操作，验证迁移在新 edition 上可用。

例如：

```console
cargo +nightly fix -Zfix-edition=end=2024,future
```

## section-timings
* Original Pull Request: [#15780](https://github.com/rust-lang/cargo/pull/15780)
* Tracking Issue: [#15817](https://github.com/rust-lang/cargo/issues/15817)

该特性可用于扩展 `cargo build --timings` 的输出。
它会让 rustc 产出各个编译阶段的耗时，随后显示在 timings 的 HTML/JSON 输出中。

```console
cargo +nightly -Zsection-timings build --timings
```

## Build analysis

* Original Issue: [rust-lang/rust-project-goals#332](https://github.com/rust-lang/rust-project-goals/pull/332)
* Tracking Issue: [#15844](https://github.com/rust-lang/cargo/issues/15844)

`-Zbuild-analysis` 会把详细构建指标记录并持久化到磁盘，
并提供新命令查询历史构建。

启用后，Cargo 会将构建日志以 JSONL 格式写入 `$CARGO_HOME/log/` 目录。
每次 cargo 调用都会生成一个带唯一 session ID 的日志文件。
这些日志包含耗时信息、重建原因以及其他构建元数据，可通过 `cargo report` 子命令分析。

要启用 build analysis，可添加如下 [Cargo configuration](config.md)：

```toml
# Example config.toml file.

[unstable]
build-analysis = true

# Enable the build metric collection
[build.analysis]
enabled = true
```

在 stable 工具链上设置它只会产生 unknown config warning，
因此可以安全保留在 Cargo 配置中。

### `cargo report` commands

`-Zbuild-analysis` 下提供以下命令：

- `cargo report sessions` --- 列出历史构建会话。
  可用来查找其他 report 命令所需的 session ID。
- `cargo report timings` --- 基于历史会话生成 HTML timing 报告，
  类似 `cargo build --timings`，但不会重新构建。
- `cargo report rebuilds` --- 报告 crate 被重新构建的原因，
  用于诊断意外的重复编译。

## build-dir-new-layout

* Tracking Issue: [#15010](https://github.com/rust-lang/cargo/issues/15010)

启用新的 build-dir 文件系统布局。
这一布局调整为缓存和锁改进工作扫清障碍。


## compile-time-deps

这个永久不稳定 flag 仅构建 proc-macro 与 build script（及其所需依赖），并执行 build script。

它面向 rust-analyzer 之类工具，且永不稳定化。

示例：

```console
cargo +nightly build --compile-time-deps -Z unstable-options
cargo +nightly check --compile-time-deps --all-targets -Z unstable-options
```

## `rustc-unicode`
* Tracking Issue: [rust#148607](https://github.com/rust-lang/rust/issues/148607)

在 Cargo 错误消息中启用 `rustc` 的 Unicode 错误格式。

## rustdoc mergeable info

* Original Pull Request: [#16309](https://github.com/rust-lang/cargo/pull/16309)
* Tracking issue: [#16306](https://github.com/rust-lang/cargo/issues/16306)
* Tracking rustc issue: [rust-lang/rust#130676](https://github.com/rust-lang/rust/issues/130676)

`-Z rustdoc-mergeable-info` 利用 rustdoc 的可合并 crate 信息，
使 `cargo doc` 能从多个输出目录合并跨 crate 信息（如搜索索引、源码索引等），
并支持并行运行 rustdoc。

## json-target-spec
* Tracking Issue: [rust-lang/rust#151528](https://github.com/rust-lang/rust/issues/151528)

`-Z json-target-spec` CLI flag 允许使用 [自定义 target spec JSON 文件](https://doc.rust-lang.org/nightly/rustc/targets/custom.html) 作为目标。

```console
cargo +nightly build --target my-target.json -Z json-target-spec
```

这通常需要与 [build-std](#build-std) 结合使用。

# Stabilized and removed features

## Compile progress

compile-progress 特性已在 1.30 版本稳定。
进度条现已默认启用。
关于控制该特性的更多信息，见 [`term.progress`](config.md#termprogresswhen)。

## Edition

在 `Cargo.toml` 中指定 `edition` 已在 1.31 版本稳定。
关于该字段的更多说明，见 [the edition field](manifest.md#the-edition-field)。

## rename-dependency

在 `Cargo.toml` 中指定重命名依赖已在 1.31 版本稳定。
关于依赖重命名的更多信息，见 [renaming dependencies](specifying-dependencies.md#renaming-dependencies-in-cargotoml)。

## Alternate Registries

对替代注册表的支持已在 1.34 版本稳定。
更多信息见 [Registries chapter](registries.md)。

## Offline Mode

offline 特性已在 1.36 版本稳定。
关于离线模式使用方式，见 [`--offline` flag](../commands/cargo.md#option-cargo---offline)。

## publish-lockfile

`publish-lockfile` 特性已在 1.37 版本移除。
若包包含二进制 target，发布时始终会包含 `Cargo.lock`。
`cargo install` 需要 `--locked` 才会使用 `Cargo.lock` 文件。
更多信息见 [`cargo package`](../commands/cargo-package.md) 与 [`cargo install`](../commands/cargo-install.md)。

## default-run

`default-run` 特性已在 1.37 版本稳定。
关于指定默认运行目标的更多信息，见 [the `default-run` field](manifest.md#the-default-run-field)。

## cache-messages

编译器消息缓存已在 1.40 版本稳定。
编译器 warning 现在默认会被缓存，并在重新运行 Cargo 时自动回放。

## install-upgrade

`install-upgrade` 特性已在 1.41 版本稳定。
[`cargo install`] 现在会在包看起来过期时自动升级。
更多信息见 [`cargo install`] 文档。

[`cargo install`]: ../commands/cargo-install.md

## Profile Overrides

Profile overrides 已在 1.41 版本稳定。
关于 override 的使用，见 [Profile Overrides](profiles.md#overrides)。

## Config Profiles

在 Cargo 配置文件和环境变量中指定 profile 已在 1.43 版本稳定。
关于在配置文件中指定 [profiles](profiles.md) 的更多信息，见 [config `[profile]` table](config.md#profile)。

## crate-versions

`-Z crate-versions` 已在 1.47 版本稳定。
crate 版本现在会自动显示在 [`cargo doc`](../commands/cargo-doc.md) 文档侧边栏中。

## Features

`-Z features` 已在 1.51 版本稳定。
关于使用新 feature resolver 的更多信息，见 [feature resolver version 2](features.md#feature-resolver-version-2)。

## package-features

`-Z package-features` 已在 1.51 版本稳定。
关于 features CLI 选项的更多信息，见 [resolver version 2 command-line flags](features.md#resolver-version-2-command-line-flags)。

## Resolver

`Cargo.toml` 中的 `resolver` 特性已在 1.51 版本稳定。
关于 resolver 指定方式的更多信息，见 [resolver versions](resolver.md#resolver-versions)。

## extra-link-arg

用于在构建脚本中指定额外链接器参数的 `extra-link-arg` 特性已在 1.56 版本稳定。
更多信息见 [build script documentation](build-scripts.md#outputs-of-the-build-script)。

## configurable-env

用于在 Cargo 配置中指定环境变量的 `configurable-env` 特性已在 1.56 版本稳定。
更多信息见 [config documentation](config.html#env)。

## rust-version

`Cargo.toml` 中的 `rust-version` 字段已在 1.56 版本稳定。
关于 `rust-version` 字段及 `--ignore-rust-version` 选项的更多信息，见 [rust-version field](manifest.html#the-rust-version-field)。

## patch-in-config

`-Z patch-in-config` 标志以及 Cargo 配置文件中 `[patch]` 段支持已在 1.56 版本稳定。
更多信息见 [patch field](config.html#patch)。

## edition 2021

2021 edition 已在 1.56 版本稳定。
关于 edition 设置，见 [`edition` field](manifest.md#the-edition-field)。
关于现有项目迁移，见 [`cargo fix --edition`](../commands/cargo-fix.md) 与 [The Edition Guide](../../edition-guide/index.html)。


## Custom named profiles

自定义命名 profile 已在 1.57 版本稳定。
更多信息见 [profiles chapter](profiles.md#custom-profiles)。

## Profile `strip` option

profile `strip` 选项已在 1.59 版本稳定。
更多信息见 [profiles chapter](profiles.md#strip)。

## Future incompat report

生成 future-incompat 报告的支持已在 1.59 版本稳定。
更多信息见 [future incompat report chapter](future-incompat-report.md)。

## Namespaced features

Namespaced features 已在 1.60 版本稳定。
更多信息见 [Features chapter](features.md#optional-dependencies)。

## Weak dependency features

Weak dependency features 已在 1.60 版本稳定。
更多信息见 [Features chapter](features.md#dependency-features)。

## timings

`-Ztimings` 已在 1.60 版本稳定为 `--timings`。
timings 输出格式选项
（例如 `--timings=html` 与机器可读的 `--timings=json`）
已在 1.94.0-nightly 中移除。

## config-cli

`--config` CLI 选项已在 1.63 版本稳定。
更多信息见 [config documentation](config.html#command-line-overrides)。

## multitarget

`-Z multitarget` 选项已在 1.64 版本稳定。
关于设置默认[目标平台三元组][target triple]，见 [`build.target`](config.md#buildtarget)。

## crate-type

`cargo rustc` 的 `--crate-type` 标志已在 1.64 版本稳定。
更多信息见 [`cargo rustc` documentation](../commands/cargo-rustc.md)。


## Workspace Inheritance

Workspace Inheritance 已在 1.64 版本稳定。
更多信息见 [workspace.package](workspaces.md#the-package-table)、
[workspace.dependencies](workspaces.md#the-dependencies-table)、
[inheriting-a-dependency-from-a-workspace](specifying-dependencies.md#inheriting-a-dependency-from-a-workspace)。

## terminal-width

`-Z terminal-width` 选项已在 1.68 版本稳定。
当在 Cargo 可自动检测终端宽度的终端中运行时，终端宽度会始终传给编译器。

## sparse-registry

Sparse registry 支持已在 1.68 版本稳定。
更多信息见 [Registry Protocols](registries.md#registry-protocols)。

### `cargo logout`

[`cargo logout`] 命令已在 1.70 版本稳定。

[target triple]: ../appendix/glossary.md#target '"target" (glossary)'
[`cargo logout`]: ../commands/cargo-logout.md

## `doctest-in-workspace`

`cargo test` 的 `-Z doctest-in-workspace` 选项已在 1.72 版本稳定并默认启用。
关于编译和运行测试时的工作目录，见 [`cargo test` documentation](../commands/cargo-test.md#working-directory-of-tests)。

## keep-going

`--keep-going` 选项已在 1.74 版本稳定。
更多细节可参考 `cargo build` 中的 [`--keep-going` flag](../commands/cargo-build.html#option-cargo-build---keep-going)。

## `[lints]`

[`[lints]`](manifest.html#the-lints-section) (enabled via `-Zlints`) has been stabilized in the 1.74 release.

## credential-process

`-Z credential-process` 特性已在 1.74 版本稳定。

详情见 [Registry Authentication](registry-authentication.md) 文档。

## registry-auth

`-Z registry-auth` 特性已在 1.74 版本稳定，额外要求是必须配置 credential-provider。

详情见 [Registry Authentication](registry-authentication.md) 文档。

## check-cfg

`-Z check-cfg` 特性已在 1.80 版本通过“设为默认行为”方式稳定。

关于自定义 cfg 指定方式，见 [build script documentation](build-scripts.md#rustc-check-cfg)。

## Edition 2024

2024 edition 已在 1.85 版本稳定。
关于 edition 设置，见 [`edition` field](manifest.md#the-edition-field)。
关于现有项目迁移，见 [`cargo fix --edition`](../commands/cargo-fix.md) 与 [The Edition Guide](../../edition-guide/index.html)。

## Automatic garbage collection

自动删除旧文件支持已在 Rust 1.88 稳定。
更多信息见 [config chapter](config.md#cache)。

## doctest-xcompile

从 Rust 1.89 开始，doctest 交叉编译已无条件启用。
现在使用 `cargo test` 运行 doctest 会遵循 `--target` 标志。

## package-workspace

多包发布已在 Rust 1.90.0 稳定。

## build-dir

`build.build-dir` 支持已在 1.91 版本稳定。
关于修改 build-dir，见 [config documentation](config.md#buildbuild-dir)。

## Build-plan

`build` 命令的 `--build-plan` 参数已在 1.93.0-nightly 移除。
移除原因见 <https://github.com/rust-lang/cargo/issues/7614>。

## config-include

通过 `include` 配置键包含额外配置文件的支持已在 1.93.0 稳定。
更多信息见 [`include` config documentation](config.md#include)。

## pubtime

`pubtime` 索引字段已在 Rust 1.94.0 稳定。

## lockfile-path

`resolver.lockfile-path` 配置字段支持已在 Rust 1.97.0 稳定。

## warnings

`build.warnings` 配置字段已在 Rust 1.97 稳定。
