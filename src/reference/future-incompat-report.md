# 未来不兼容报告

Cargo 会检查所有依赖中的“未来不兼容”警告。
这类警告表示某些变更未来可能升级为硬错误，导致该依赖在将来的 rustc 版本中无法构建。
如果发现此类警告，会显示一条简短提示，说明已发现问题，并给出查看完整报告的方法。

例如，构建结束时你可能会看到如下内容：

```text
warning: the following packages contain code that will be rejected by a future
         version of Rust: rental v0.5.5
note: to see what the problems were, use the option `--future-incompat-report`,
      or run `cargo report future-incompatibilities --id 1`
```

可通过 `cargo report future-incompatibilities --id ID` 查看完整报告，
也可以在构建时重新运行并加上 `--future-incompat-report`。
开发者随后应将依赖升级到已修复问题的版本，
或与依赖维护者协作解决问题。

## 配置

该功能可在 `.cargo/config.toml` 的
[`[future-incompat-report]`][config] 段中配置。
当前支持的选项如下：

```toml
[future-incompat-report]
frequency = "always"
```

`frequency` 支持值为 `"always"` 和 `"never"`，
用于控制是否在 `cargo build` / `cargo check` 结束时输出提示消息。

[config]: config.md#future-incompat-report
