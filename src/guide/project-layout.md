# 包布局

Cargo 通过约定文件放置位置，让你能够快速理解一个新的 Cargo
[package][def-package]：

```text
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

* `Cargo.toml` 和 `Cargo.lock` 存放在包根目录（*package root*）。
* 源代码放在 `src` 目录。
* 默认库文件是 `src/lib.rs`。
* 默认可执行文件是 `src/main.rs`。
    * 其他可执行文件可放在 `src/bin/`。
* 基准测试放在 `benches` 目录。
* 示例放在 `examples` 目录。
* 集成测试放在 `tests` 目录。

如果某个二进制目标、示例、基准测试或集成测试由多个源文件组成，
请在 `src/bin`、`examples`、`benches` 或 `tests` 下创建子目录，
并在其中放置一个 `main.rs` 以及额外的 [*module*][def-module] 文件。
可执行文件名将使用该子目录名。

> **注意：** 按约定，二进制目标、示例、基准测试和集成测试通常使用 `kebab-case` 命名风格，除非有兼容性原因（例如要兼容既有二进制名）。这些目标中的模块命名则遵循 Rust 标准，使用 `snake_case`。

你可以在[《Rust 程序设计语言》关于模块系统的章节][book-modules]中了解更多。

关于手动配置 target 的细节请见 [Configuring a target]。
关于控制 Cargo 自动推断 target 名称请见 [Target auto-discovery]。

[book-modules]: ../../book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[Configuring a target]: ../reference/cargo-targets.md#configuring-a-target
[def-package]:           ../appendix/glossary.md#package          '"package" (glossary entry)'
[def-module]:            ../appendix/glossary.md#module           '"module" (glossary entry)'
[Target auto-discovery]: ../reference/cargo-targets.md#target-auto-discovery
