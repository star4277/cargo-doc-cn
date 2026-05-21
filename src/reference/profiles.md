# Profiles

Profile 用于调整编译器设置，从而影响优化等级、调试符号等行为。

Cargo 内置 4 个 profile：`dev`、`release`、`test`、`bench`。
如果命令行未显式指定 profile，Cargo 会根据所执行命令自动选择。
除了内置 profile，也可以定义自定义 profile。

profile 设置可在 [`Cargo.toml`](manifest.md) 的 `[profile]` 表中修改。
在每个具名 profile 下，可通过如下键值对单独调整设置：

```toml
[profile.dev]
opt-level = 1               # Use slightly better optimizations.
overflow-checks = false     # Disable integer overflow checks.
```

Cargo 只会读取 workspace 根目录 `Cargo.toml` 中的 profile 设置。
依赖中的 profile 设置会被忽略。

此外，也可以在 [config] 中覆盖 profile。
如果在配置文件或环境变量里指定了 profile，
会覆盖 `Cargo.toml` 中对应设置。

[config]: config.md

## Profile 设置项

以下是可在 profile 中控制的设置。

### opt-level

`opt-level` 控制 [`-C opt-level` flag]，即优化级别。
更高优化级别可能带来更快运行速度，但会增加编译时间。
同时更高优化也可能重排代码，影响调试体验。

可选值：

* `0`：无优化
* `1`：基础优化
* `2`：一定程度优化
* `3`：尽可能全部优化
* `"s"`：优化二进制体积
* `"z"`：优化二进制体积，并关闭循环向量化

建议在项目中实际测试不同级别，找到平衡点。
有时会出现直觉外结果，例如 `3` 反而比 `2` 慢，或 `"s"` / `"z"` 不一定更小。
随着 `rustc` 版本演进，优化行为也会变化，建议定期复查设置。

进阶优化可参考 [Profile Guided Optimization]。

[`-C opt-level` flag]: ../../rustc/codegen-options/index.html#opt-level
[Profile Guided Optimization]: ../../rustc/profile-guided-optimization.html

### debug

`debug` 控制 [`-C debuginfo` flag]，即生成到二进制中的调试信息量。

可选值：

* `0`、`false` 或 `"none"`：不包含调试信息（[`release`](#release) 默认）
* `"line-directives-only"`：仅行指令信息。对 nvptx* 目标可启用 [profiling]。
  其他场景一般更建议 `line-tables-only`，兼容性更好。
* `"line-tables-only"`：仅行号表。可用于带文件名/行号的回溯，不含变量/函数参数信息。
* `1` 或 `"limited"`：不含类型级或变量级信息，但比 `line-tables-only` 提供更详细的模块级信息。
* `2`、`true` 或 `"full"`：完整调试信息（[`dev`](#dev) 默认）

各选项细节见 `rustc` 的 [debuginfo] 文档。

你也可能需要结合 [`split-debuginfo`](#split-debuginfo) 一起配置。

> **MSRV:** `none`、`limited`、`full`、`line-directives-only`、`line-tables-only`
> 需要 Rust 1.71。

[`-C debuginfo` flag]: ../../rustc/codegen-options/index.html#debuginfo
[debuginfo]: ../../rustc/codegen-options/index.html#debuginfo
[profiling]: https://reviews.llvm.org/D46061

### split-debuginfo

`split-debuginfo` 控制 [`-C split-debuginfo` flag]，
决定调试信息是放在可执行文件内，还是拆分到旁边的独立文件。

此选项为字符串，可选值与编译器支持值一致（见
[`-C split-debuginfo` flag]）。
若该 profile 启用了调试信息，macOS 默认值是 `unpacked`。
其他平台默认值以 rustc 文档（[`-C split-debuginfo` flag]）为准，
并且与平台相关。部分值仅在 [nightly channel] 可用。
Cargo 的默认值未来可能调整（例如在更多测试完成、DWARF 支持稳定后）。

请注意 Cargo 与 rustc 在此选项上的默认值并不相同。
该选项存在的目的之一，就是让 Cargo 可以实验更合适的 flag 组合，
改善调试与开发体验。

[nightly channel]: ../../book/appendix-07-nightly-rust.html
[`-C split-debuginfo` flag]: ../../rustc/codegen-options/index.html#split-debuginfo

### strip

`strip` 控制 [`-C strip` flag]，用于让 rustc 从二进制中剥离符号或调试信息。
示例：

```toml
[package]
# ...

[profile.release]
strip = "debuginfo"
```

`strip` 可选字符串值：`"none"`、`"debuginfo"`、`"symbols"`。
默认是 `"none"`。

也可使用布尔值 `true` / `false`：
`strip = true` 等价于 `strip = "symbols"`；
`strip = false` 等价于 `strip = "none"`，即完全禁用 strip。

[`-C strip` flag]: ../../rustc/codegen-options/index.html#strip

### debug-assertions

`debug-assertions` 控制 [`-C debug-assertions` flag]，
用于开启/关闭 `cfg(debug_assertions)` [条件编译]。
Debug assertions 通常用于开发/调试构建中的运行时校验；
这些校验在发布构建中可能过于昂贵或不希望开启。
开启后会启用标准库的 [`debug_assert!` macro]。

可选值：

* `true`：启用
* `false`：禁用

[`-C debug-assertions` flag]: ../../rustc/codegen-options/index.html#debug-assertions
[conditional compilation]: ../../reference/conditional-compilation.md#debug_assertions
[`debug_assert!` macro]: ../../std/macro.debug_assert.html

### overflow-checks

`overflow-checks` 控制 [`-C overflow-checks` flag]，
用于指定[整数溢出运行时行为][runtime integer overflow]。
开启后发生溢出会触发 panic。

可选值：

* `true`：启用
* `false`：禁用

[`-C overflow-checks` flag]: ../../rustc/codegen-options/index.html#overflow-checks
[runtime integer overflow]: ../../reference/expressions/operator-expr.md#overflow

### lto

`lto` 控制 rustc 的 [`-C lto`]、[`-C linker-plugin-lto`]、[`-C embed-bitcode`]，
用于控制 LLVM 的 [link time optimizations]。
LTO 可通过全程序分析获得更优性能，但会增加链接时间。

可选值：

* `true` 或 `"fat"`：执行“fat LTO”，尝试跨依赖图所有 crate 做优化。
* `"thin"`：执行 ["thin" LTO]。效果接近 fat，但耗时明显更低。
* `false`：执行“thin local LTO”，只在本地 crate 的
  [codegen units](#codegen-units) 间做 thin LTO。
  若 codegen units 为 1 或 [opt-level](#opt-level) 为 0，则不做 LTO。
* `"off"`：禁用 LTO。

若你关注跨语言 LTO，可阅读 [linker-plugin-lto chapter]。
Cargo 目前尚未原生支持该能力，但可通过 `RUSTFLAGS` 实现。

[`-C lto`]: ../../rustc/codegen-options/index.html#lto
[link time optimizations]: https://llvm.org/docs/LinkTimeOptimization.html
[`-C linker-plugin-lto`]: ../../rustc/codegen-options/index.html#linker-plugin-lto
[`-C embed-bitcode`]: ../../rustc/codegen-options/index.html#embed-bitcode
[linker-plugin-lto chapter]: ../../rustc/linker-plugin-lto.html
["thin" LTO]: http://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html

### panic

`panic` 控制 [`-C panic` flag]，决定 panic 策略。

可选值：

* `"unwind"`：panic 时展开栈。
* `"abort"`：panic 时直接终止进程。

设置为 `"unwind"` 时，实际行为仍受目标平台默认值影响。
例如 NVPTX 平台不支持 unwind，因此始终使用 `"abort"`。

`test`、`bench`、构建脚本、proc macro 会忽略 `panic` 设置。
`rustc` 的测试 harness 当前要求 `unwind` 行为。
可参考不稳定选项 [`panic-abort-tests`] 来启用 `abort` 行为。

另外，当使用 `abort` 策略并执行测试构建时，
所有依赖也会被强制以 `unwind` 策略构建。

[`-C panic` flag]: ../../rustc/codegen-options/index.html#panic
[`panic-abort-tests`]: unstable.md#panic-abort-tests

### incremental

`incremental` 控制 [`-C incremental` flag]，
用于启用/禁用增量编译。
增量编译会让 `rustc` 把额外信息写入磁盘，在后续重编译时复用，
从而提升二次编译速度。额外信息存放于 `target` 目录。

可选值：

* `true`：启用
* `false`：禁用

增量编译仅用于 workspace 成员以及 `path` 依赖。

此值可被全局的 `CARGO_INCREMENTAL` [环境变量]或
[`build.incremental`] 配置覆盖。

[`-C incremental` flag]: ../../rustc/codegen-options/index.html#incremental
[environment variable]: environment-variables.md
[`build.incremental`]: config.md#buildincremental

### codegen-units

`codegen-units` 控制 [`-C codegen-units` flag]，
决定 crate 被拆分成多少“代码生成单元”。
单元越多，越有利于并行编译，可能缩短编译时间，
但也可能生成更慢的代码。

该选项需设置为大于 0 的整数。

默认值：增量构建为 256；非增量构建为 16。

[`-C codegen-units` flag]: ../../rustc/codegen-options/index.html#codegen-units

### rpath

`rpath` 控制 [`-C rpath` flag]，决定是否启用 [`rpath`]。

[`-C rpath` flag]: ../../rustc/codegen-options/index.html#rpath
[`rpath`]: https://en.wikipedia.org/wiki/Rpath

## 默认 profile

### dev

`dev` profile 用于日常开发与调试。
它是 [`cargo build`] 之类构建命令的默认 profile，
也是 `cargo install --debug` 使用的 profile。

`dev` 默认设置：

```toml
[profile.dev]
opt-level = 0
debug = true
split-debuginfo = '...'  # Platform-specific.
strip = "none"
debug-assertions = true
overflow-checks = true
lto = false
panic = 'unwind'
incremental = true
codegen-units = 256
rpath = false
```

### release

`release` profile 用于发布和生产场景的优化构建产物。
使用 `--release` 时会启用此 profile，
它也是 [`cargo install`] 的默认 profile。

`release` 默认设置：

```toml
[profile.release]
opt-level = 3
debug = false
split-debuginfo = '...'  # Platform-specific.
strip = "none"
debug-assertions = false
overflow-checks = false
lto = false
panic = 'unwind'
incremental = false
codegen-units = 16
rpath = false
```

### test

`test` 是 [`cargo test`] 的默认 profile。
`test` 继承 [`dev`](#dev) profile 的设置。

### bench

`bench` 是 [`cargo bench`] 的默认 profile。
`bench` 继承 [`release`](#release) profile 的设置。

### Build Dependencies

为了加快编译，默认情况下所有 profile 都不会优化构建期依赖
（构建脚本、proc macro 及其依赖），
并尽量在“构建期依赖不作为运行期依赖”时不计算调试信息。
build override 的默认设置为：

```toml
[profile.dev.build-override]
opt-level = 0
codegen-units = 256
debug = false # when possible

[profile.release.build-override]
opt-level = 0
codegen-units = 256
```

但如果运行构建期依赖时出现错误，打开完整调试信息能显著改善回溯与可调试性：

```toml
debug = true
```

除上述覆盖外，构建期依赖会继承当前活动 profile 的设置，
见 [Profile selection](#profile-selection)。

## 自定义 profile

除内置 profile 外，也可以定义额外的自定义 profile。
这对建立多工作流、多构建模式很有帮助。
定义自定义 profile 时，必须指定 `inherits`，
用于在某项未设置时从哪个 profile 继承。

例如你想比较“普通 release 构建”和“启用 [LTO](#lto) 的 release 构建”，
可在 `Cargo.toml` 写：

```toml
[profile.release-lto]
inherits = "release"
lto = true
```

然后通过 `--profile` 选择该自定义 profile：

```console
cargo build --profile release-lto
```

每个 profile 的输出会放到 [`target` directory] 中同名子目录。
例如上例输出目录为 `target/release-lto`。

[`target` directory]: build-cache.md

## Profile 选择规则

最终使用哪个 profile，取决于命令本身、命令行参数（如 `--release` / `--profile`）
以及 package（在使用[覆盖](#overrides)时）。
未显式指定时，默认 profile 如下：

| Command | Default Profile |
|---------|-----------------|
| [`cargo run`], [`cargo build`],<br>[`cargo check`], [`cargo rustc`] | [`dev` profile](#dev) |
| [`cargo test`] | [`test` profile](#test)
| [`cargo bench`] | [`bench` profile](#bench)
| [`cargo install`] | [`release` profile](#release)

可用 `--profile=NAME` 切换到指定 profile。
`--release` 等价于 `--profile=release`。

所选 profile 会应用于所有 Cargo target，
包括 [library](./cargo-targets.md#library)、
[binary](./cargo-targets.md#binaries)、
[example](./cargo-targets.md#examples)、
[test](./cargo-targets.md#tests)、
以及 [benchmark](./cargo-targets.md#benchmarks)。

特定 package 的 profile 可通过下文[覆盖](#overrides)指定。

[`cargo bench`]: ../commands/cargo-bench.md
[`cargo build`]: ../commands/cargo-build.md
[`cargo check`]: ../commands/cargo-check.md
[`cargo install`]: ../commands/cargo-install.md
[`cargo run`]: ../commands/cargo-run.md
[`cargo rustc`]: ../commands/cargo-rustc.md
[`cargo test`]: ../commands/cargo-test.md

## Overrides

可以为特定 package 或构建期 crate 覆盖 profile 设置。
要覆盖某个 package，可使用 `package` 表指定该 package：

```toml
# The `foo` package will use the -Copt-level=3 flag.
[profile.dev.package.foo]
opt-level = 3
```

这里的 package 名实际是 [Package ID Spec](pkgid-spec.md)，
因此你可以用如 `[profile.dev.package."foo:2.1.0"]` 的语法
精确命中特定版本。

要覆盖所有依赖（不含 workspace 成员），使用 `"*"`：

```toml
# Set the default for dependencies.
[profile.dev.package."*"]
opt-level = 2
```

要覆盖构建脚本、proc macro 及其依赖，使用 `build-override` 表：

```toml
# Set the settings for build scripts and proc-macros.
[profile.dev.build-override]
opt-level = 3
```

> 注意：当某依赖既是普通依赖又是构建依赖，且未指定 `--target` 时，
> Cargo 会尽量只构建一次。使用 `build-override` 后，该依赖可能需要构建两次：
> 一次按普通依赖，一次按覆盖后的构建设置。这会增加首次构建时间。

值选择优先级如下（先匹配先使用）：

1. `[profile.dev.package.name]` --- 指定包名。
2. `[profile.dev.package."*"]` --- 任意非 workspace 成员。
3. `[profile.dev.build-override]` --- 仅构建脚本、proc macro 及其依赖。
4. `[profile.dev]` --- `Cargo.toml` 中该 profile 设置。
5. Cargo 内置默认值。

覆盖项不能指定 `panic`、`lto`、`rpath`。

### Overrides 与泛型

泛型代码实例化发生的位置，会影响其使用的优化设置。
这会导致一个细微但常见的问题：
当你用 profile override 提高某个 crate 的优化等级时，
该 crate 中的泛型函数在你本地 crate 中实例化后，可能并不会按“被提高后的设置”优化。
因为最终代码可能是在“实例化发生的 crate”中生成，
于是会使用那个 crate 的优化设置。

例如 [nalgebra] 大量使用泛型参数定义向量与矩阵。
如果你的本地代码定义了 `Vector4<f64>` 等具体类型并调用其方法，
对应 nalgebra 代码会在你的 crate 内完成实例化与构建。
因此即便你通过 profile override 提高 `nalgebra` 的优化等级，
也可能看不到预期性能提升。

问题还会更复杂：`rustc` 会在某些情况下尝试跨 crate 共享单态化后的泛型。
当 opt-level 是 2 或 3 时，一个 crate 不会使用其他 crate 的单态化泛型，
也不会导出本地单态化项供其他 crate 共享。
如果你在开发中尝试优化依赖，
可以考虑 opt-level 1：它能提供一部分优化，同时仍允许共享单态化项。

[nalgebra]: https://crates.io/crates/nalgebra
