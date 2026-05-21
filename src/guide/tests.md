# 测试

你可以使用 `cargo test` 运行测试。Cargo 会在两个位置查找测试：
`src` 下的源文件，以及 `tests/` 目录。
`src` 内通常放单元测试和[文档测试][documentation tests]；
`tests/` 下通常放集成测试。因此在 `tests` 目录里的测试文件中，
你需要显式导入你的 crate。

下面是在当前没有任何测试的 [package][def-package] 中执行 `cargo test`
的示例：

```console
$ cargo test
   Compiling regex v1.5.0 (https://github.com/rust-lang/regex.git#9f9f693)
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
     Running target/test/hello_world-9c2b65bbb79eabce

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

如果你的包中有测试，你会看到对应数量的测试输出。

你也可以通过过滤器运行特定测试：

```console
$ cargo test foo
```

这会运行名字中包含 `foo` 的测试。

`cargo test` 还会做额外检查：
它会编译示例代码以确保仍可编译；
还会运行文档测试以确保文档注释中的代码样例能编译。

关于测试编写和组织的一般方法，请参考 Rust 文档中的
[测试指南][testing]。关于 Cargo 中不同类型测试，请看 [Cargo Targets: Tests]。

[documentation tests]: ../../rustdoc/write-documentation/documentation-tests.html
[def-package]:  ../appendix/glossary.md#package  '"package" (glossary entry)'
[testing]: ../../book/ch11-00-testing.html
[Cargo Targets: Tests]: ../reference/cargo-targets.html#tests
