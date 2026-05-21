# 依赖

[crates.io] 是 Rust 社区的中央[*包注册表*][def-package-registry]，
用于发现和下载 [package][def-package]。`cargo` 默认配置为从它查找
你请求的包。

如果要依赖托管在 [crates.io] 上的库，请把它添加到 `Cargo.toml`。

[crates.io]: https://crates.io/

## 添加依赖

如果你的 `Cargo.toml` 还没有 `[dependencies]` 段，先添加该段，
然后列出要使用的 [crate][def-crate] 名称和版本。下面示例添加 `time` 依赖：

```toml
[dependencies]
time = "0.1.12"
```

这个版本字符串是一个 [SemVer] 版本要求。关于可选写法的更多信息，
请参考[指定依赖](../reference/specifying-dependencies.md)。

[SemVer]: https://semver.org

如果你还想添加 `regex` 依赖，不需要为每个 crate 重复写 `[dependencies]`。
下面是同时依赖 `time` 和 `regex` 时整个 `Cargo.toml` 的样子：

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2024"

[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

重新执行 `cargo build`，Cargo 会下载新依赖及其所有依赖、全部编译，
并更新 `Cargo.lock`：

```console
$ cargo build
      Updating crates.io index
   Downloading memchr v0.1.5
   Downloading libc v0.1.10
   Downloading regex-syntax v0.2.1
   Downloading memchr v0.1.5
   Downloading aho-corasick v0.3.0
   Downloading regex v0.1.41
     Compiling memchr v0.1.5
     Compiling libc v0.1.10
     Compiling regex-syntax v0.2.1
     Compiling memchr v0.1.5
     Compiling aho-corasick v0.3.0
     Compiling regex v0.1.41
     Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

`Cargo.lock` 记录了所有这些依赖所使用的精确版本信息。

现在即使 `regex` 以后发布了更新，在你主动执行 `cargo update` 之前，
构建仍会使用当前锁定的版本。

此时你就可以在 `main.rs` 中使用 `regex` 库：

```rust,ignore
use regex::Regex;

fn main() {
    let re = Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    println!("Did our date match? {}", re.is_match("2014-01-01"));
}
```

运行后会看到：

```console
$ cargo run
   Running `target/hello_world`
Did our date match? true
```

[def-crate]:             ../appendix/glossary.md#crate             '"crate" (glossary entry)'
[def-package]:           ../appendix/glossary.md#package           '"package" (glossary entry)'
[def-package-registry]:  ../appendix/glossary.md#package-registry  '"package-registry" (glossary entry)'
