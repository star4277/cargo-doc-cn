# Cargo Targets

Cargo package 由多个 *target* 组成，target 对应可被编译为 crate 的源文件。
一个 package 可以包含 [library](#library)、[binary](#binaries)、
[example](#examples)、[test](#tests)、[benchmark](#benchmarks) target。
target 列表可在 `Cargo.toml` 中配置，也常会根据源文件的
[目录布局][package layout]被[自动推断](#target-auto-discovery)。

target 的配置细节见下文 [Configuring a target](#configuring-a-target)。

## Library

library target 定义一个“库”，可被其他库和可执行程序引用与链接。
默认文件名是 `src/lib.rs`，
默认库名为 package 名（把 `-` 替换为 `_`）。
一个 package 只能有一个 library。
library 的设置可在 `Cargo.toml` 的 `[lib]` 表中[自定义][customized]。

```toml
# Example of customizing the library in Cargo.toml.
[lib]
crate-type = ["cdylib"]
bench = false
```

## Binaries

binary target 是编译后可运行的可执行程序。
binary 源文件可以是 `src/main.rs`，也可以放在 [`src/bin/`
目录][package layout] 下。
对 `src/main.rs` 而言，默认 binary 名为 package 名。
每个 binary 的设置可在 `Cargo.toml` 的 `[[bin]]` 表中[自定义][customized]。

binary 可以使用 package library 的公开 API。
它们也会链接 `Cargo.toml` 中定义的 [`[dependencies]`][dependencies]。

你可以用 [`cargo run`] 的 `--bin <bin-name>` 运行指定 binary。
[`cargo install`] 可把可执行文件复制到通用安装位置。

```toml
# Example of customizing binaries in Cargo.toml.
[[bin]]
name = "cool-tool"
test = false
bench = false

[[bin]]
name = "frobnicator"
required-features = ["frobnicate"]
```

## Examples

[`examples` 目录][package layout]下的文件用于展示库功能用法。
编译后会放到 [`target/debug/examples` 目录][build cache]。

example 可以使用 package library 的公开 API。
它们也会链接 `Cargo.toml` 中的 [`[dependencies]`][dependencies]
和 [`[dev-dependencies]`][dev-dependencies]。

默认情况下，example 是可执行 binary（包含 `main()`）。
你也可以通过 [`crate-type` 字段](#the-crate-type-field)
把 example 编译为库：

```toml
[[example]]
name = "foo"
crate-type = ["staticlib"]
```

可执行 example 可通过 [`cargo run`] + `--example <example-name>` 运行。
library example 可通过 [`cargo build`] + `--example <example-name>` 构建。
[`cargo install`] + `--example <example-name>` 可把可执行 example
复制到通用安装位置。
为了避免示例代码“腐烂”，example 默认会被 [`cargo test`] 编译检查。
如果 example 中有希望由 [`cargo test`] 执行的 `#[test]` 函数，
请把 [ `test` 字段](#the-test-field) 设为 `true`。

## Tests

Cargo 项目中有两类测试：

* *单元测试（unit tests）*：
  即在库或 binary（或任何启用了[ `test` 字段](#the-test-field)的 target）内部，
  使用 [`#[test]` attribute][test-attribute] 标记的函数。
  这类测试可访问其所在 target 的私有 API。
* *集成测试（integration tests）*：
  即独立可执行二进制，同样包含 `#[test]` 函数，
  通过链接项目的 library 来测试其*公开* API。

测试通过 [`cargo test`] 运行。
默认情况下，Cargo 和 `rustc` 使用 [libtest harness]。
它负责收集带 [`#[test]` attribute][test-attribute] 的函数，
并行执行它们，并报告每个测试的成功/失败。
若你想换测试驱动或策略，见 [ `harness` 字段](#the-harness-field)。

> **注意**：Cargo 还有一种特殊测试：
> [文档测试][documentation examples]。
> 它由 `rustdoc` 处理，执行模型略有不同。
> 详见 [`cargo test`][cargo-test-documentation-tests]。

[libtest harness]: ../../rustc/tests/index.html
[cargo-test-documentation-tests]: ../commands/cargo-test.md#documentation-tests

### Integration tests

[`tests` 目录][package layout]下的文件即集成测试。
执行 [`cargo test`] 时，Cargo 会把每个文件编译成单独 crate 并执行。

集成测试可使用 package library 的公开 API。
它们也会链接 `Cargo.toml` 中的 [`[dependencies]`][dependencies]
和 [`[dev-dependencies]`][dev-dependencies]。

如果你想在多个集成测试间共享代码，
可放到独立模块（如 `tests/common/mod.rs`），
并在各测试里写 `mod common;` 引入。

每个集成测试都会生成独立可执行文件，
而 [`cargo test`] 会串行运行这些可执行文件。
在某些场景下这可能低效：编译时间更长，
运行时也未必能充分利用多核 CPU。
如果集成测试很多，可考虑只建一个集成测试 target，
再把测试拆到多个模块。
libtest harness 会自动发现所有带 `#[test]` 的函数并并行运行。
你也可以给 [`cargo test`] 传模块名，只跑该模块测试。

只要存在集成测试，binary target 会自动构建。
这样集成测试就能执行 binary 来测试其行为。
在构建与运行集成测试时会设置
`CARGO_BIN_EXE_<name>` [环境变量]，
测试可借助 [`env` macro] 或 [`var` function] 定位可执行文件。

[environment variable]: environment-variables.md#environment-variables-cargo-sets-for-crates
[`env` macro]: ../../std/macro.env.html
[`var` function]: ../../std/env/fn.var.html

## Benchmarks

benchmark 用于通过 [`cargo bench`] 测试代码性能。
它的结构与 [tests](#tests) 类似，
每个 benchmark 函数通过 `#[bench]` 标记。
与测试类似：

* benchmark 放在 [`benches` 目录][package layout]。
* 在库和 binary 中定义的 benchmark 函数可访问该 target 的*私有* API；
  `benches` 目录中的 benchmark 可使用*公开* API。
* 可用 [ `bench` 字段](#the-bench-field) 指定默认参与 benchmark 的 target。
* 可用 [ `harness` 字段](#the-harness-field) 禁用内置 harness。

> **注意**：[`#[bench]`
> attribute](../../unstable-book/library-features/test.html) 当前不稳定，
> 仅在 [nightly channel] 可用。
> 也有一些 [crates.io](https://crates.io/keywords/benchmark) 上的包可帮助你在稳定版做 benchmark，
> 如 [Criterion](https://crates.io/crates/criterion)。

## Configuring a target

`Cargo.toml` 中的 `[lib]`、`[[bin]]`、`[[example]]`、`[[test]]`、`[[bench]]`
都支持类似配置，用于指定 target 的构建方式。
像 `[[bin]]` 这样的双中括号节是
[TOML 的 array-of-table](https://toml.io/en/v1.0.0-rc.3#array-of-tables)，
因此可写多个 `[[bin]]` 来定义多个可执行文件。
而 library 只能有一个，所以 `[lib]` 是普通 TOML table。

下面先概览这些配置项，具体解释见后文。

```toml
[lib]
name = "foo"           # The name of the target.
path = "src/lib.rs"    # The source file of the target.
test = true            # Is tested by default.
doctest = true         # Documentation examples are tested by default.
bench = true           # Is benchmarked by default.
doc = true             # Is documented by default.
proc-macro = false     # Set to `true` for a proc-macro library.
harness = true         # Use libtest harness.
crate-type = ["lib"]   # The crate types to generate.
required-features = [] # Features required to build this target (N/A for lib).
```

### `name` 字段

`name` 指定 target 名称，对应最终产物文件名。
对 library 来说，这也是依赖引用它时使用的 crate 名。

对 library target，默认值是 package 名（`-` 替换为 `_`）。
对默认 binary（`src/main.rs`），默认值同样是 package 名，
但不会替换 `-`。
对[自动发现](#target-auto-discovery)的 target，默认是目录名或文件名。

除 `[lib]` 外，该字段对所有 target 都必填。

### `path` 字段

`path` 指定 crate 源文件位置，相对于 `Cargo.toml`。

若未指定，会根据 target 名使用[推断路径](#target-auto-discovery)。

### `test` 字段

`test` 表示该 target 是否默认由 [`cargo test`] 测试。
对 lib、bin、test，默认 `true`。

> **注意**：example 默认会被 [`cargo test`] 编译，
> 以确保持续可编译，但默认不会执行测试。
> 对 example 设置 `test = true` 后，
> 它也会作为 test 构建，并运行其中定义的
> [`#[test]`][test-attribute] 函数。

### `doctest` 字段

`doctest` 表示是否默认由 [`cargo test`] 测试[文档示例][documentation examples]。
该字段只对 library 有意义，对其他节无效。
library 默认值为 `true`。

### `bench` 字段

`bench` 表示该 target 是否默认由 [`cargo bench`] 进行 benchmark。
对 lib、bin、benchmark，默认值是 `true`。

### `doc` 字段

`doc` 表示该 target 是否默认包含在 [`cargo doc`] 生成的文档中。
对 library 和 binary，默认值为 `true`。

> **注意**：若 binary 名与 lib target 同名，则该 binary 会被跳过。

### `plugin` 字段

该选项已弃用且未使用。

### `proc-macro` 字段

`proc-macro` 表示该库是[过程宏][procedural macro]
（见[参考][proc-macro-reference]）。
该字段仅对 `[lib]` target 有效。

### `harness` 字段

`harness` 表示是否向 `rustc` 传 [`--test` flag]。
传入后会自动包含 libtest 库，
由它负责收集并运行使用 [`#[test]`
attribute][test-attribute] 标注的测试函数，
以及 `#[bench]` 标注的 benchmark 函数。
所有 target 的默认值都是 `true`。

若设为 `false`，则需你自行定义 `main()` 来运行测试/benchmark。

无论 harness 是否启用，测试都会启用
[`cfg(test)` 条件表达式][cfg-test]。

### `crate-type` 字段

`crate-type` 定义 target 将生成的 [crate 类型][crate types]。
它是字符串数组，因此一个 target 可指定多个 crate 类型。
该字段只能用于 library 与 example。
binary、test、benchmark 始终是 `bin` crate 类型。
默认值如下：

Target | Crate Type
-------|-----------
Normal library | `"lib"`
Proc-macro library | `"proc-macro"`
Example | `"bin"`

可选项包括：`bin`、`lib`、`rlib`、`dylib`、`cdylib`、`staticlib`、`proc-macro`。
不同 crate 类型详见 [Rust Reference Manual][crate types]。

### `required-features` 字段

`required-features` 指定构建该 target 所需的 [features]。
若任一必需 feature 未启用，则该 target 会被跳过。
该字段仅对 `[[bin]]`、`[[bench]]`、`[[test]]`、`[[example]]` 有效，
对 `[lib]` 无效。

```toml
[features]
# ...
postgres = []
sqlite = []
tools = []

[[bin]]
name = "my-pg-tool"
required-features = ["postgres", "tools"]
```

### `edition` 字段

`edition` 定义该 target 使用的 [Rust edition]。
若未指定，默认继承 `[package]` 的 [`edition` 字段][package-edition]。

> **注意：**该字段已弃用，并将在未来某个 Edition 中移除。

## 目标自动发现

默认情况下，Cargo 会根据文件系统上的[文件布局][package layout]
自动决定要构建哪些 target。
诸如 `[lib]`、`[[bin]]`、`[[test]]`、`[[bench]]`、`[[example]]`
等 target 配置表，可用于添加不遵循标准目录布局的额外 target。

自动发现也可关闭，让 Cargo 仅构建手工配置的 target。
在 `[package]` 下把 `autolib`、`autobins`、`autoexamples`、`autotests`、`autobenches`
设为 `false`，即可关闭对应 target 类型的自动发现。

```toml
[package]
# ...
autolib = false
autobins = false
autoexamples = false
autotests = false
autobenches = false
```

关闭自动发现通常只在特殊场景需要。
例如你有一个库，里面想定义名为 `bin` 的*模块*。
这会冲突，因为 Cargo 通常会把 `bin` 目录内容当作可执行目标编译。
示例布局如下：

```text
├── Cargo.toml
└── src
    ├── lib.rs
    └── bin
        └── mod.rs
```

为了防止 Cargo 把 `src/bin/mod.rs` 推断为可执行文件，
可在 `Cargo.toml` 设置 `autobins = false` 关闭自动发现：

```toml
[package]
# ...
autobins = false
```

> **注意**：对 2015 edition 的 package，
> 若 `Cargo.toml` 里手工定义了至少一个 target，自动发现默认是 `false`。
> 从 2018 edition 起，默认始终是 `true`。

> **MSRV:** `autobins`、`autoexamples`、`autotests`、`autobenches` 自 1.27 起生效。

> **MSRV:** `autolib` 自 1.83 起生效。

[Build cache]: build-cache.md
[Rust Edition]: ../../edition-guide/index.html
[`--test` flag]: ../../rustc/command-line-arguments.html#option-test
[`cargo bench`]: ../commands/cargo-bench.md
[`cargo build`]: ../commands/cargo-build.md
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo install`]: ../commands/cargo-install.md
[`cargo run`]: ../commands/cargo-run.md
[`cargo test`]: ../commands/cargo-test.md
[cfg-test]: ../../reference/conditional-compilation.html#test
[crate types]: ../../reference/linkage.html
[crates.io]: https://crates.io/
[customized]: #configuring-a-target
[dependencies]: specifying-dependencies.md
[dev-dependencies]: specifying-dependencies.md#development-dependencies
[documentation examples]: ../../rustdoc/documentation-tests.html
[features]: features.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[package layout]: ../guide/project-layout.md
[package-edition]: manifest.md#the-edition-field
[proc-macro-reference]: ../../reference/procedural-macros.html
[procedural macro]: ../../book/ch19-06-macros.html
[test-attribute]: ../../reference/attributes/testing.html#the-test-attribute


