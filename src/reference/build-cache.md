# 构建缓存

Cargo 会把构建输出存放到 `target` 与 `build` 目录。
默认情况下，两者都指向 [*workspace*][def-workspace] 根目录下名为 `target` 的目录。
若要修改 target-dir，可设置 `CARGO_TARGET_DIR` [环境变量]、
[`build.target-dir`] 配置项，或使用 `--target-dir` 命令行参数。
若要修改 build-dir，可设置 `CARGO_BUILD_BUILD_DIR` [环境变量]
或 [`build.build-dir`] 配置项。

构建产物分为两类：
* 最终构建产物
  * 面向 Cargo 最终用户的输出
  * 例如 bin crate 的可执行文件、`cargo doc` 输出、Cargo `--timings` 报告
  * 存储于 target-dir
* 中间构建产物
  * Cargo 与 Rust 编译器内部使用的产物
  * 最终用户通常不需要直接操作
  * 存储于 Cargo build-dir

目录布局取决于是否使用 `--target` 为特定平台构建。
若未指定 `--target`，Cargo 以主机架构模式构建。
输出位于 target 目录根部，每个 [profile] 位于单独子目录：

Directory | Description
----------|------------
<code style="white-space: nowrap">target/debug/</code> | `dev` profile 输出。
<code style="white-space: nowrap">target/release/</code> | `release` profile 输出（使用 `--release` 选项）。
<code style="white-space: nowrap">target/foo/</code> | `foo` profile 输出（使用 `--profile=foo` 选项）。

由于历史原因，`dev` 与 `test` profile 存在 `debug` 目录，
`release` 与 `bench` profile 存在 `release` 目录。
用户自定义 profile 存在与 profile 同名目录。

使用 `--target` 为其他目标构建时，输出会放在以 [target] 命名的目录中：

Directory | Example
----------|--------
<code style="white-space: nowrap">target/&lt;triple&gt;/debug/</code> | <code style="white-space: nowrap">target/thumbv7em-none-eabihf/debug/</code>
<code style="white-space: nowrap">target/&lt;triple&gt;/release/</code> | <code style="white-space: nowrap">target/thumbv7em-none-eabihf/release/</code>

> **注意**：不使用 `--target` 时，Cargo 会让构建脚本与 proc macro 共享你的依赖。
> [`RUSTFLAGS`] 也会被每次 `rustc` 调用共享。
> 使用 `--target` 时，构建脚本与 proc macro 会单独为主机架构构建，
> 且不会共享 `RUSTFLAGS`。

在 profile 目录（如 `debug` 或 `release`）内，产物放在以下目录：

Directory | Description
----------|------------
<code style="white-space: nowrap">target/debug/</code> | 正在构建的 package 输出（[二进制可执行文件] 与 [库 target]）。
<code style="white-space: nowrap">target/debug/examples/</code> | [示例 target] 输出。

部分命令会在 `target` 顶层专用目录写入输出：

Directory | Description
----------|------------
<code style="white-space: nowrap">target/doc/</code> | rustdoc 文档（[`cargo doc`]）。
<code style="white-space: nowrap">target/package/</code> | [`cargo package`] 的输出。

Cargo 还会在 build-dir 中创建若干构建过程所需目录和文件。
build-dir 布局属于 Cargo 内部实现，未来可能变化。
其中部分目录如下：

Directory | Description
----------|------------
<code style="white-space: nowrap">\<build-dir>/debug/deps/</code> | 依赖及其他产物。
<code style="white-space: nowrap">\<build-dir>/debug/incremental/</code> | `rustc` [增量输出]，用于加速后续构建。
<code style="white-space: nowrap">\<build-dir>/debug/build/</code> | [构建脚本]输出。

## Dep-info 文件

每个已编译产物旁边都会有一个后缀为 `.d` 的“dep info”文件。
该文件采用类似 Makefile 的语法，记录重建该产物所需的全部文件依赖。
它主要供外部构建系统使用，以检测是否需要重新执行 Cargo。
文件中的路径默认是绝对路径。
如需相对路径，见 [`build.dep-info-basedir`] 配置项。

```Makefile
# 在 target/debug/foo.d 中找到的 dep-info 示例
/path/to/myproj/target/debug/foo: /path/to/myproj/src/lib.rs /path/to/myproj/src/main.rs
```

## 共享缓存

可使用第三方工具 [sccache] 在不同 workspace 之间共享构建依赖。

要配置 `sccache`，先通过 `cargo install sccache` 安装，
然后在运行 Cargo 前把环境变量 `RUSTC_WRAPPER` 设为 `sccache`。
如果你使用 bash，可把 `export RUSTC_WRAPPER=sccache` 写入 `.bashrc`。
也可在 [Cargo 配置][config] 中设置 [`build.rustc-wrapper`]。
更多细节请参阅 sccache 文档。

[`RUSTFLAGS`]: ../reference/config.md#buildrustflags
[`build.dep-info-basedir`]: ../reference/config.md#builddep-info-basedir
[`build.rustc-wrapper`]: ../reference/config.md#buildrustc-wrapper
[`build.target-dir`]: ../reference/config.md#buildtarget-dir
[`build.build-dir`]: ../reference/config.md#buildbuild-dir
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo package`]: ../commands/cargo-package.md
[`cargo publish`]: ../commands/cargo-publish.md
[build scripts]: ../reference/build-scripts.md
[config]: ../reference/config.md
[def-workspace]:  ../appendix/glossary.md#workspace  '"workspace" (glossary entry)'
[target]: ../appendix/glossary.md#target '"target" (glossary entry)'
[environment variable]: ../reference/environment-variables.md
[incremental output]: ../reference/profiles.md#incremental
[sccache]: https://github.com/mozilla/sccache
[profile]: ../reference/profiles.md
[binary executables]: ../reference/cargo-targets.md#binaries
[library targets]: ../reference/cargo-targets.md#library
[example targets]: ../reference/cargo-targets.md#examples
