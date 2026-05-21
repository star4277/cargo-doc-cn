# Build Scripts

有些 package 需要编译第三方非 Rust 代码，例如 C 库。
还有些 package 需要链接 C 库，这些库可能已在系统中，也可能要从源码构建。
另外一些场景还需要在构建前先做代码生成（例如语法解析器生成器）。

Cargo 的目标不是替代在这些任务上已经高度优化的工具，
而是通过自定义构建脚本与它们集成。
在 package 根目录放置名为 `build.rs` 的文件，
Cargo 就会在构建 package 前先编译并执行该脚本。

```rust,ignore
// Example custom build script.
fn main() {
    // Tell Cargo that if the given file changes, to rerun this build script.
    println!("cargo::rerun-if-changed=src/hello.c");
    // Use the `cc` crate to build a C file and statically link it.
    cc::Build::new()
        .file("src/hello.c")
        .compile("hello");
}
```

构建脚本的典型用途包括：

* 构建内置的 C 库。
* 在宿主系统查找 C 库。
* 从规范文件生成 Rust 模块。
* 执行 crate 所需的平台特定配置。

下文介绍构建脚本的工作方式，[示例章节](build-script-examples.md)
给出了多种编写脚本的实例。

> 注意：可通过 [`package.build` manifest 键](manifest.md#the-build-field)
> 修改构建脚本文件名，或完全禁用构建脚本。

## Life Cycle of a Build Script

在构建 package 之前，Cargo 会先把构建脚本编译为可执行文件
（若尚未编译过）。然后运行该脚本，脚本可执行任意任务。
脚本可通过向 stdout 输出以 `cargo::` 开头的特定格式命令与 Cargo 通信。

当构建脚本的任意源码文件或其依赖发生变化时，脚本会被重新编译。

默认情况下，只要 package 内任意文件变化，Cargo 就会重跑构建脚本。
通常应使用下文[变更检测](#change-detection)中的 `rerun-if` 指令，
把重跑触发范围收窄到真正相关的输入。

构建脚本成功结束后，Cargo 才会继续编译 package 其他部分。
若脚本出错，应以非零退出码退出以中止构建，
此时构建脚本输出会显示在终端。

## Inputs to the Build Script

构建脚本运行时会收到一组输入，
全部通过[环境变量][build-env]传入。

除环境变量外，构建脚本的当前工作目录是其所属 package 的根目录。

[build-env]: environment-variables.md#environment-variables-cargo-sets-for-build-scripts

> **注意：**在构建脚本中检查 `target_os`、`target_arch` 等[配置项]时，
> 不要使用 `cfg!` 宏或 `#[cfg]` 属性，
> 因为它们检查的是**host** 机器（构建脚本运行环境），
> 不是你正在编译的**target** 平台。
> 交叉编译时这个区别尤其关键。
>
> 应改为读取对应的 [`CARGO_CFG_*`][build-env] 环境变量，
> 它们会正确反映 target 配置。
> 若希望使用类型化 API，可考虑 [`build-rs`] crate。
> 详见 [build script examples]。

[configuration options]: ../../reference/conditional-compilation.html
[`build-rs`]: https://crates.io/crates/build-rs
[build script examples]: build-script-examples.md#conditional-compilation

## Outputs of the Build Script

构建脚本可以把输出文件和中间产物保存到
[`OUT_DIR` 环境变量][build-env]指定的目录。
脚本不应修改该目录之外的文件。

> **注意：**Cargo 不会在两次构建之间清空或重置 `OUT_DIR`。
> 其内容可能跨多次重建保留，即使构建脚本重跑也一样。
> 这是刻意设计，用于支持增量构建（例如本地代码编译）。
>
> 构建脚本不应假设 `OUT_DIR` 为空。
> 如果脚本需要干净目录，当前应自行管理/清理所创建的文件或子目录。
> 该领域的改进仍在讨论中（见
> [#16427](https://github.com/rust-lang/cargo/issues/16427) 与
> [#9661](https://github.com/rust-lang/cargo/issues/9661)）。

构建脚本通过向 stdout 打印内容与 Cargo 通信。
Cargo 会把每一行以 `cargo::` 开头的输出解释为“会影响包编译”的指令。
其他行会被忽略。

> 构建脚本输出 `cargo::` 指令的顺序，*可能*
> 影响 `cargo` 传给 `rustc` 的参数顺序，
> 进而影响 `rustc` 传给链接器的参数顺序。
> 因此需要关注指令顺序。
> 例如对象 `foo` 需要依赖库 `bar` 链接时，
> 通常希望 `bar` 的 [`cargo::rustc-link-lib`](#rustc-link-lib)
> 出现在链接 `foo` 相关指令之后。

正常编译时，脚本输出不会直接显示在终端。
若要在终端直接查看，使用 `-vv`（very verbose）运行 Cargo。
仅当构建脚本实际运行时才会显示；
若 Cargo 判断无需重跑，则不会再次执行脚本。
详见下文[变更检测](#change-detection)。

构建脚本 stdout 的所有行都会写入类似
`target/debug/build/<pkg>/output` 的文件
（具体路径受配置影响）。
stderr 也保存在同目录。

下面是 Cargo 可识别指令概览，后文有细节说明：

* [`cargo::rerun-if-changed=PATH`](#rerun-if-changed) --- 指定何时重跑脚本。
* [`cargo::rerun-if-env-changed=VAR`](#rerun-if-env-changed) --- 指定何时因环境变量变化重跑。
* [`cargo::rustc-link-arg=FLAG`](#rustc-link-arg) --- 为 benchmark、binary、`cdylib`、example、test 传递链接器参数。
* [`cargo::rustc-link-arg-cdylib=FLAG`](#rustc-cdylib-link-arg) --- 为 `cdylib` 传递链接器参数。
* [`cargo::rustc-link-arg-bin=BIN=FLAG`](#rustc-link-arg-bin) --- 为指定 binary `BIN` 传递链接器参数。
* [`cargo::rustc-link-arg-bins=FLAG`](#rustc-link-arg-bins) --- 为所有 binary 传递链接器参数。
* [`cargo::rustc-link-arg-tests=FLAG`](#rustc-link-arg-tests) --- 为 test 传递链接器参数。
* [`cargo::rustc-link-arg-examples=FLAG`](#rustc-link-arg-examples) --- 为 example 传递链接器参数。
* [`cargo::rustc-link-arg-benches=FLAG`](#rustc-link-arg-benches) --- 为 benchmark 传递链接器参数。
* [`cargo::rustc-link-lib=LIB`](#rustc-link-lib) --- 添加待链接库。
* [`cargo::rustc-link-search=[KIND=]PATH`](#rustc-link-search) --- 添加库搜索路径。
* [`cargo::rustc-flags=FLAGS`](#rustc-flags) --- 给编译器传递特定参数。
* [`cargo::rustc-cfg=KEY[="VALUE"]`](#rustc-cfg) --- 启用编译期 `cfg` 设置。
* [`cargo::rustc-check-cfg=CHECK_CFG`](#rustc-check-cfg) --- 注册“预期 cfg”以用于编译期配置检查。
* [`cargo::rustc-env=VAR=VALUE`](#rustc-env) --- 设置环境变量。
- [`cargo::error=MESSAGE`](#cargo-error) --- 在终端显示错误。
* [`cargo::warning=MESSAGE`](#cargo-warning) --- 在终端显示警告。
* [`cargo::metadata=KEY=VALUE`](#the-links-manifest-key) --- 元数据，用于 `links` 脚本。

> **MSRV:** `cargo::KEY=VALUE` 语法需要 Rust 1.77。
> 若需兼容更老版本，请使用 `cargo:KEY=VALUE`。

### `cargo::rustc-link-arg=FLAG`
`rustc-link-arg` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG` 选项][link-arg]。
但仅对受支持 target 生效（benchmark、binary、`cdylib`、example、test）。
该用法高度平台相关，常用于设置共享库版本或链接脚本。

[link-arg]: ../../rustc/codegen-options/index.md#link-arg

### `cargo::rustc-link-arg-cdylib=FLAG`
`rustc-link-arg-cdylib` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG`][link-arg]，
但仅在构建 `cdylib` 库 target 时生效。
该用法高度平台相关，常用于设置共享库版本或 runtime-path。

出于历史原因，`cargo::rustc-cdylib-link-arg` 是
`cargo::rustc-link-arg-cdylib` 的别名，语义相同。

### `cargo::rustc-link-arg-bin=BIN=FLAG`
`rustc-link-arg-bin` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG`][link-arg]，
但仅在构建名为 `BIN` 的 binary target 时生效。
该用法高度平台相关，常用于设置链接脚本或其他链接器选项。

### `cargo::rustc-link-arg-bins=FLAG`
`rustc-link-arg-bins` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG`][link-arg]，
但仅在构建 binary target 时生效。
该用法高度平台相关，常用于设置链接脚本或其他链接器选项。

### `cargo::rustc-link-arg-tests=FLAG`
`rustc-link-arg-tests` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG`][link-arg]，
但仅在构建 test target 时生效。

### `cargo::rustc-link-arg-examples=FLAG`
`rustc-link-arg-examples` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG`][link-arg]，
但仅在构建 example target 时生效。

### `cargo::rustc-link-arg-benches=FLAG`
`rustc-link-arg-benches` 指令告诉 Cargo：
向编译器传递 [`-C link-arg=FLAG`][link-arg]，
但仅在构建 benchmark target 时生效。

### `cargo::rustc-link-lib=LIB`
`rustc-link-lib` 指令告诉 Cargo：
使用编译器 [`-l` 选项][option-link]链接指定库。
通常用于通过 [FFI] 链接本地库。

`LIB` 字符串会原样传给 rustc，因此支持 `-l` 的全部语法。
当前完整支持的 `LIB` 语法是 `[KIND[:MODIFIERS]=]NAME[:RENAME]`。

`-l` 参数默认只传给 package 的 library target。
若 package 没有 library target，才会传给所有 target。
这样做是因为其他 target 都隐式依赖 library target，
该库只应链接一次。
这意味着：若 package 同时有 library 和 binary，
*library* 可直接访问该库符号，
binary 应通过 library 的公开 API 间接访问。

可选 `KIND` 为 `dylib`、`static` 或 `framework`。
详见 [rustc book][option-link]。

[option-link]: ../../rustc/command-line-arguments.md#option-l-link-lib
[FFI]: ../../nomicon/ffi.md

### `cargo::rustc-link-search=[KIND=]PATH`
`rustc-link-search` 指令告诉 Cargo：
向编译器传递 [`-L` 参数][option-search]，
把目录加入库搜索路径。

可选 `KIND` 为 `dependency`、`crate`、`native`、`framework`、`all`。
详见 [rustc book][option-search]。

如果这些路径位于 `OUT_DIR` 内，也会被加入[动态库搜索路径环境变量](environment-variables.md#dynamic-library-paths)。
不建议依赖该行为，因为这会让产物更难使用。
通常最好避免在构建脚本中生成动态库（使用系统已有动态库没问题）。

[option-search]: ../../rustc/command-line-arguments.md#option-l-search-path

### `cargo::rustc-flags=FLAGS`
`rustc-flags` 指令告诉 Cargo：
向编译器传递给定的空格分隔参数。
这里只允许 `-l` 与 `-L`，
等价于使用 [`rustc-link-lib`](#rustc-link-lib)
和 [`rustc-link-search`](#rustc-link-search)。

### `cargo::rustc-cfg=KEY[="VALUE"]`
`rustc-cfg` 指令告诉 Cargo：
把给定值传给编译器 [`--cfg` 参数][option-cfg]。
可用于编译期能力探测并启用[条件编译]。
自定义 cfg 必须通过 [`cargo::rustc-check-cfg`](#rustc-check-cfg)
声明为预期值，否则需要放宽 [`unexpected_cfgs`][unexpected-cfgs]
lint 才能避免“unexpected cfg”警告。

注意：这**不会**影响 Cargo 的依赖解析。
不能用它来启用可选依赖，也不能启用其他 Cargo feature。

还要注意 [Cargo features] 使用 `feature="foo"` 形式。
而通过该指令传入的 cfg 不受此限制：
既可仅传单个标识符，也可传任意键值对。
例如输出 `cargo::rustc-cfg=abc` 后，可在代码里用 `#[cfg(abc)]`
（注意没有 `feature=`）。
也可输出 `cargo::rustc-cfg=my_component="foo"` 这种任意键值对。
其中 key 应为 Rust 标识符，value 应为字符串。

[cargo features]: features.md
[conditional compilation]: ../../reference/conditional-compilation.md
[option-cfg]: ../../rustc/command-line-arguments.md#option-cfg
[unexpected-cfgs]: ../../rustc/lints/listing/warn-by-default.md#unexpected-cfgs

### `cargo::rustc-check-cfg=CHECK_CFG`
将“预期配置名和值”加入列表，
该列表会用于 [`unexpected_cfgs`][unexpected-cfgs]
lint 对 _reachable_ cfg 表达式的检查。

`CHECK_CFG` 语法与 rustc 的 [`--check-cfg` 参数][option-check-cfg]一致。
详见 [Checking conditional configurations][checking-conditional-configurations]。

使用示例：

```rust,no_run
// build.rs
println!("cargo::rustc-check-cfg=cfg(foo, values(\"bar\"))");
if foo_bar_condition {
    println!("cargo::rustc-cfg=foo=\"bar\"");
}
```

注意应定义“所有可能 cfg”，
不论它当前是否启用；
这包括同一 cfg 名的所有可能值。

建议把 `cargo::rustc-check-cfg` 与
[`cargo::rustc-cfg`][option-cfg] 指令尽量写在一起，
以减少拼写错误、漏写 check-cfg、遗留 cfg 等问题。

另见[条件编译][conditional-compilation-example]示例。

> **MSRV:** 自 1.80 起生效。

[checking-conditional-configurations]: ../../rustc/check-cfg.html
[option-check-cfg]: ../../rustc/command-line-arguments.md#option-check-cfg
[conditional-compilation-example]: build-script-examples.md#conditional-compilation

### `cargo::rustc-env=VAR=VALUE`
`rustc-env` 指令告诉 Cargo：
在编译 package 时设置指定环境变量。
编译后的 crate 可通过 [`env!` 宏][env-macro]读取该值。
常见用途是把额外元数据嵌入代码，
例如 git HEAD 哈希或 CI 服务器唯一 ID。

另见 [Cargo 自动注入的环境变量][env-cargo]。

> **注意**：这些环境变量在 `cargo run` / `cargo test`
> 运行可执行文件时也会被设置。
> 但不建议依赖这种运行时行为，
> 因为这会把可执行程序绑定到 Cargo 运行环境。
> 通常应只在编译期通过 `env!` 宏读取这些变量。

[env-macro]: ../../std/macro.env.html
[env-cargo]: environment-variables.md#environment-variables-cargo-sets-for-crates

### `cargo::error=MESSAGE`
`error` 指令告诉 Cargo：
在构建脚本运行结束后显示错误，并使构建失败。

 > 注意：构建脚本库需谨慎选择使用 `cargo::error`
 > 还是返回 `Result`。
 > 更好的做法通常是返回 `Result`，
 > 由调用方决定错误是否致命。
 > 调用方再决定是否通过 `cargo::error` 展示 `Err`。

> **MSRV:** 自 1.84 起生效。

### `cargo::warning=MESSAGE`
`warning` 指令告诉 Cargo：
在构建脚本结束后显示警告。
默认仅对 `path` 依赖显示警告（即本地正在开发的依赖）。
因此 [crates.io] 上依赖打印的 warning 默认不会显示，
除非构建失败。
可通过 `-vv` 让 Cargo 显示所有 crate 的 warning。

## Build Dependencies

构建脚本也可以依赖其他基于 Cargo 的 crate。
这些依赖通过 manifest 的 `build-dependencies` 声明。

```toml
[build-dependencies]
cc = "1.0.46"
```

构建脚本**不能**访问 `dependencies` / `dev-dependencies`
里的依赖（它们尚未构建完成）。
此外，build dependency 也不会自动提供给 package 运行时代码，
除非你也显式把它加到 `[dependencies]`。

建议谨慎添加每个依赖，
权衡其对编译时间、许可证、维护成本等影响。
Cargo 会尽量复用“同时出现在 build 依赖和普通依赖”的 crate，
但并非总能做到（例如交叉编译场景）。
因此仍需关注它对编译时间的影响。

## Change Detection

重建 package 时，Cargo 不一定知道是否需要重跑构建脚本。
默认策略较保守：只要 package 内任一文件变化
（或由 [`exclude` 与 `include` 字段]控制的文件集合变化）就重跑。
这在多数情况下不是理想选择。
建议每个构建脚本至少输出一个 `rerun-if` 指令（见下）。
一旦输出了这些指令，Cargo 就只会在对应值变化时重跑脚本。
如果你发现 Cargo 在重跑你自己的 crate 或依赖的构建脚本，
但不清楚原因，可看 FAQ：
[Why is Cargo rebuilding my code?](../faq.md#why-is-cargo-rebuilding-my-code)

[`exclude` and `include` fields]: manifest.md#the-exclude-and-include-fields

### `cargo::rerun-if-changed=PATH`
`rerun-if-changed` 指令告诉 Cargo：
当给定路径文件变化时重跑构建脚本。
当前 Cargo 仅使用文件系统“最后修改时间（mtime）”判断变化，
并与上次运行构建脚本时记录的时间戳比较。

若路径指向目录，Cargo 会扫描整个目录并检测任意修改。

若构建脚本本质上“永远不需要重跑”，
可输出 `cargo::rerun-if-changed=build.rs`，
这是阻止重跑的简单方法。
（否则在未输出任何 `rerun-if` 指令时，默认会扫描整个 package 目录变化。）
Cargo 会自动处理“脚本本身是否需要重新编译”；
脚本一旦被重编译，当然会再次运行。
除这类场景外，显式写 `build.rs` 往往是多余的。

### `cargo::rerun-if-env-changed=NAME`
`rerun-if-env-changed` 指令告诉 Cargo：
当指定环境变量值变化时重跑构建脚本。

这里的环境变量指全局变量（如 `CC`）。
不能用它监听像 `TARGET` 这种由 [Cargo 为构建脚本设置的变量][build-env]。
它监控的是 `cargo` 进程接收到的环境变量，
不是构建脚本可执行文件进程再继承到的那一层。

自 1.46 起，源码中使用 [`env!`][env-macro] 和 [`option_env!`][option-env-macro]
会自动检测变化并触发重建。
对这些宏已引用的变量，就不再需要 `rerun-if-env-changed`。

[option-env-macro]: ../../std/macro.option_env.html

## The `links` Manifest Key

可在 `Cargo.toml` 中设置 `package.links`，
声明该 package 会链接指定本地库。
该键的作用是让 Cargo 理解 package 的本地依赖集合，
并为构建脚本之间传递元数据提供规范机制。

```toml
[package]
# ...
links = "foo"
```

这表示 package 会链接 `libfoo` 本地库。
当使用 `links` 键时，package 必须有构建脚本，
且构建脚本应通过 [`rustc-link-lib` 指令](#rustc-link-lib)执行链接。

Cargo 的核心约束是：同一个 `links` 值最多只能由一个 package 使用。
换言之，禁止两个 package 同时声明链接同一本地库。
这有助于避免 crate 之间符号重复。
不过也有一套[约定](#-sys-packages)用于缓解该限制。

构建脚本可生成任意键值对元数据，
通过 `cargo::metadata=KEY=VALUE` 指令设置。

这些元数据会传递给**直接依赖方**的构建脚本。
例如 package `foo` 依赖 `bar`，而 `bar` 链接 `baz`。
若 `bar` 的构建脚本生成 `key=value`，
则 `foo` 的构建脚本会看到环境变量 `DEP_BAZ_KEY=value`
（注意这里使用 `links` 值，并对 key 做大小写转换）。
具体用法示例见 ["Using another `sys` crate"][using-another-sys]。

注意：元数据只传给直接依赖，不会传给传递依赖。

> **MSRV:** `cargo::metadata=KEY=VALUE` 需要 1.77。
> 若需支持旧版本，请使用 `cargo:KEY=VALUE`
> （不支持的指令会被当作 metadata 键处理）。

[using-another-sys]: build-script-examples.md#using-another-sys-crate

## `*-sys` Packages

有些链接系统库的 Cargo package 采用 `-sys` 后缀命名约定。
名为 `foo-sys` 的 package 通常应提供两类能力：

* 库 crate 应链接本地库 `libfoo`。
  通常会先探测系统是否已有 `libfoo`，找不到再从源码构建。
* 库 crate 应提供 `libfoo` 的类型/函数**声明**，
  但**不**提供高层抽象。

`*-sys` 约定为链接本地库提供了一套通用依赖。
其好处包括：

* 统一依赖 `foo-sys` 可缓解“每个 `links` 值只能一个 package”的规则限制。
* 其他 `-sys` 包可利用 `DEP_LINKS_KEY=value` 环境变量与其更好集成。
  见 ["Using another `sys` crate"][using-another-sys]。
* 可把 `libfoo` 的发现/构建逻辑集中维护。
* 这些依赖易于[覆盖](#overriding-build-scripts)。

常见模式是再提供一个不带 `-sys` 后缀的配套包，
在 sys 包之上提供安全高层抽象。
例如 [`git2` crate] 就是 [`libgit2-sys` crate] 的高层接口。

[`git2` crate]: https://crates.io/crates/git2
[`libgit2-sys` crate]: https://crates.io/crates/libgit2-sys

## Overriding Build Scripts

若 manifest 含 `links` 键，Cargo 支持用自定义库覆盖其构建脚本。
该能力的目的是“彻底跳过原构建脚本执行”，
并提前提供所需元数据。

要覆盖构建脚本，可在任意合法 [`config.toml`](config.md) 中加入：

```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = "-L /some/path"
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["..."]
metadata_key1 = "value"
metadata_key2 = "value"
```

有了该配置后，只要某 package 声明链接 `foo`，
其构建脚本就**不会**再被编译/运行，
改用这里提供的元数据。

`warning`、`rerun-if-changed`、`rerun-if-env-changed`
这些键不应在此使用，且会被忽略。

## Jobserver

Cargo 与 `rustc` 使用 GNU make 的 [jobserver protocol] 协调跨进程并发。
它本质是一个信号量，用于控制并发任务数。
并发度可由 `--jobs` 指定，默认是逻辑 CPU 数。

每个构建脚本会从 Cargo 继承一个 job slot，
因此脚本运行时应尽量只占用一个 CPU。
若脚本希望并行使用更多 CPU，
应使用 [`jobserver` crate] 与 Cargo 协调。

例如 [`cc` crate] 可启用可选 `parallel` feature，
它会通过 jobserver 协议尝试并行编译多个 C 文件。

[`cc` crate]: https://crates.io/crates/cc
[`jobserver` crate]: https://crates.io/crates/jobserver
[jobserver protocol]: http://make.mad-scientist.net/papers/jobserver-implementation/
[crates.io]: https://crates.io/

