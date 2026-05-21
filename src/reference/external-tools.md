# 外部工具

Cargo 的目标之一是与第三方工具（如 IDE 与其他构建系统）进行简单集成。
为简化集成，Cargo 提供了多种能力：

* [`cargo metadata`] 命令，可输出包结构与依赖信息（JSON），

* `--message-format` 标志，可输出某次构建的详细信息，

* 自定义子命令支持。


## 包结构信息

你可以使用 [`cargo metadata`] 获取包结构与依赖信息。
输出格式细节见 [`cargo metadata`] 文档。

该格式是稳定且带版本号的。
调用 `cargo metadata` 时，建议显式传入 `--format-version`，
以避免前向不兼容风险。

如果你使用 Rust，可用 [cargo_metadata] crate 解析输出。

[cargo_metadata]: https://crates.io/crates/cargo_metadata
[`cargo metadata`]: ../commands/cargo-metadata.md

## JSON 消息

传入 `--message-format=json` 时，Cargo 在构建过程中会输出以下信息：

* 编译器错误与警告，

* 产出 artifacts，

* 构建脚本结果（例如本地依赖信息）。

输出写入 stdout，格式为“每行一个 JSON 对象”。
`reason` 字段用于区分消息类型。
`package_id` 字段是引用 package 的唯一标识，
也可作为许多命令的 `--package` 参数。
其语法见 [Package ID Specifications] 章节。

> **注意：** `--message-format=json` 只控制 Cargo 与 Rustc 的输出。
> 它无法控制其他工具输出，
> 例如 `cargo run --message-format=json` 中程序本身输出，
> 或 proc-macro 的任意输出。
> 一个可行变通是：仅当某行以 `{` 开头时才按 JSON 解析。

`--message-format` 还支持附加格式值，
用于调整 JSON 消息的计算与渲染方式。
更多细节见[构建命令文档]中 `--message-format` 描述。

如果你使用 Rust，可用 [cargo_metadata] crate 解析这些消息。

> **MSRV:** `package_id` 采用 Package ID Specification 需要 1.77。
> 在此之前它是一个不透明值。

[build command documentation]: ../commands/cargo-build.md
[cargo_metadata]: https://crates.io/crates/cargo_metadata
[Package ID Specifications]: ./pkgid-spec.md

### 编译器消息

“compiler-message” 消息包含编译器输出（如警告与错误）。
`rustc` 消息格式细节见 [rustc JSON 章节](../../rustc/json.md)，
其内容嵌入在如下结构中：

```javascript
{
    /* The "reason" indicates the kind of message. */
    "reason": "compiler-message",
    /* The Package ID, a unique identifier for referring to the package. */
    "package_id": "file:///path/to/my-package#0.1.0",
    /* Absolute path to the package manifest. */
    "manifest_path": "/path/to/my-package/Cargo.toml",
    /* The Cargo target (lib, bin, example, etc.) that generated the message. */
    "target": {
        /* Array of target kinds.
           - lib targets list the `crate-type` values from the
             manifest such as "lib", "rlib", "dylib",
             "proc-macro", etc. (default ["lib"])
           - binary is ["bin"]
           - example is ["example"]
           - integration test is ["test"]
           - benchmark is ["bench"]
           - build script is ["custom-build"]
        */
        "kind": [
            "lib"
        ],
        /* Array of crate types.
           - lib and example libraries list the `crate-type` values
             from the manifest such as "lib", "rlib", "dylib",
             "proc-macro", etc. (default ["lib"])
           - all other target kinds are ["bin"]
        */
        "crate_types": [
            "lib"
        ],
        /* The name of the target.
           For lib targets, dashes will be replaced with underscores.
        */
        "name": "my_package",
        /* Absolute path to the root source file of the target. */
        "src_path": "/path/to/my-package/src/lib.rs",
        /* The Rust edition of the target.
           Defaults to the package edition.
        */
        "edition": "2018",
        /* Array of required features.
           This property is not included if no required features are set.
        */
        "required-features": ["feat1"],
        /* Whether the target should be documented by `cargo doc`. */
        "doc": true,
        /* Whether or not this target has doc tests enabled, and
           the target is compatible with doc testing.
        */
        "doctest": true
        /* Whether or not this target should be built and run with `--test`
        */
        "test": true
    },
    /* The message emitted by the compiler.

    See https://doc.rust-lang.org/rustc/json.html for details.
    */
    "message": {
        /* ... */
    }
}
```

### Artifact 消息

每个编译步骤都会发出一条 “compiler-artifact” 消息，结构如下：

```javascript
{
    /* The "reason" indicates the kind of message. */
    "reason": "compiler-artifact",
    /* The Package ID, a unique identifier for referring to the package. */
    "package_id": "file:///path/to/my-package#0.1.0",
    /* Absolute path to the package manifest. */
    "manifest_path": "/path/to/my-package/Cargo.toml",
    /* The Cargo target (lib, bin, example, etc.) that generated the artifacts.
       See the definition above for `compiler-message` for details.
    */
    "target": {
        "kind": [
            "lib"
        ],
        "crate_types": [
            "lib"
        ],
        "name": "my_package",
        "src_path": "/path/to/my-package/src/lib.rs",
        "edition": "2018",
        "doc": true,
        "doctest": true,
        "test": true
    },
    /* The profile indicates which compiler settings were used. */
    "profile": {
        /* The optimization level. */
        "opt_level": "0",
        /* The debug level, an integer of 0, 1, or 2, or a string
           "line-directives-only" or "line-tables-only". If `null`, it implies
           rustc's default of 0.
        */
        "debuginfo": 2,
        /* Whether or not debug assertions are enabled. */
        "debug_assertions": true,
        /* Whether or not overflow checks are enabled. */
        "overflow_checks": true,
        /* Whether or not the `--test` flag is used. */
        "test": false
    },
    /* Array of features enabled. */
    "features": ["feat1", "feat2"],
    /* Array of files generated by this step. */
    "filenames": [
        "/path/to/my-package/target/debug/libmy_package.rlib",
        "/path/to/my-package/target/debug/deps/libmy_package-be9f3faac0a26ef0.rmeta"
    ],
    /* A string of the path to the executable that was created, or null if
       this step did not generate an executable.
    */
    "executable": null,
    /* Whether or not this step was actually executed.
       When `true`, this means that the pre-existing artifacts were
       up-to-date, and `rustc` was not executed. When `false`, this means that
       `rustc` was run to generate the artifacts.
    */
    "fresh": true
}

```

### 构建脚本输出

“build-script-executed” 消息包含构建脚本解析后的输出。
注意即使构建脚本未执行，也会发出该消息；此时显示的是此前缓存值。
构建脚本输出细节见[构建脚本章节](build-scripts.md)。

```javascript
{
    /* The "reason" indicates the kind of message. */
    "reason": "build-script-executed",
    /* The Package ID, a unique identifier for referring to the package. */
    "package_id": "file:///path/to/my-package#0.1.0",
    /* Array of libraries to link, as indicated by the `cargo::rustc-link-lib`
       instruction. Note that this may include a "KIND=" prefix in the string
       where KIND is the library kind.
    */
    "linked_libs": ["foo", "static=bar"],
    /* Array of paths to include in the library search path, as indicated by
       the `cargo::rustc-link-search` instruction. Note that this may include a
       "KIND=" prefix in the string where KIND is the library kind.
    */
    "linked_paths": ["/some/path", "native=/another/path"],
    /* Array of cfg values to enable, as indicated by the `cargo::rustc-cfg`
       instruction.
    */
    "cfgs": ["cfg1", "cfg2=\"string\""],
    /* Array of [KEY, VALUE] arrays of environment variables to set, as
       indicated by the `cargo::rustc-env` instruction.
    */
    "env": [
        ["SOME_KEY", "some value"],
        ["ANOTHER_KEY", "another value"]
    ],
    /* An absolute path which is used as a value of `OUT_DIR` environmental
       variable when compiling current package.
    */
    "out_dir": "/some/path/in/target/dir"
}
```

### 构建结束

“build-finished” 消息会在构建结束时发出。

```javascript
{
    /* The "reason" indicates the kind of message. */
    "reason": "build-finished",
    /* Whether or not the build finished successfully. */
    "success": true,
}
````

该消息有助于工具判断何时停止读取 JSON 消息。
像 `cargo test` 或 `cargo run` 这类命令，构建结束后仍可能继续输出内容。
该消息可告知工具：Cargo 不会再输出 JSON 消息，
但后续可能还有其他输出（例如 `cargo run` 执行程序本身的输出）。

> 注意：nightly 上有实验性的测试 JSON 输出支持，
> 如果启用，在 “build-finished” 之后可能还会收到测试相关 JSON 消息。

## 自定义子命令

Cargo 设计上可通过新子命令扩展，无需修改 Cargo 本体。
其实现方式是将 `cargo (?<command>[^ ]+)` 形式调用
转换为外部工具 `cargo-${command}` 的调用。
该外部工具必须位于用户 `$PATH` 某个目录下。

> **注意：** Cargo 默认会优先查找 `$CARGO_HOME/bin` 中的外部工具，而不是 `$PATH`。
> 用户可通过将 `$CARGO_HOME/bin` 添加到 `$PATH` 来覆盖该优先级。

当 Cargo 调用自定义子命令时，
第一个参数与通常一致，为子命令可执行文件名；
第二个参数为子命令名本身。
例如调用 `cargo-${command}` 时，第二个参数是 `${command}`。
命令行上的其余参数会原样转发。

Cargo 还支持通过 `cargo help ${command}` 显示自定义子命令帮助。
Cargo 假定当第三个参数是 `--help` 时，子命令会输出帮助信息。
因此 `cargo help ${command}` 会调用
`cargo-${command} ${command} --help`。

自定义子命令可使用 `CARGO` 环境变量回调 Cargo。
另一种方式是把 `cargo` crate 当库链接进来，但有两个缺点：

* 作为库的 Cargo 不稳定：API 可能无弃用流程直接变化
* 链接的 Cargo 库版本可能与 Cargo 可执行文件版本不同

因此更推荐通过 CLI 接口驱动 Cargo。
可使用 [`cargo metadata`] 获取当前项目信息
（[`cargo_metadata`] crate 提供该命令的 Rust 接口）。

[`cargo metadata`]: ../commands/cargo-metadata.md
[`cargo_metadata`]: https://crates.io/crates/cargo_metadata
