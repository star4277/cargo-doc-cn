# Features 示例

下面展示一些在真实项目中使用 feature 的示例。

## 最小化构建时间和文件体积

有些包会通过 feature 让“未启用某些功能”时减少 crate 体积并缩短编译时间。
示例包括：

* [`syn`] 是非常流行的 Rust 代码解析 crate。
  由于它被大量项目使用，缩短它的编译时间会带来广泛收益。
  它提供了[清晰文档化的 feature 列表][syn-features]，可用于最小化包含代码量。
* [`regex`] 有[多个 feature][regex-features]，并且[文档完备][regex-docs]。
  关闭 Unicode 支持可减少生成文件体积，因为可以移除一些大型表。
* [`winapi`] 有[大量 feature][winapi-features]，用于限制支持哪些 Windows API 绑定。
* [`web-sys`] 与 `winapi` 类似，也提供[非常大的 API 绑定面][web-sys-features]，
  通过 feature 来裁剪。

[`winapi`]: https://crates.io/crates/winapi
[winapi-features]: https://github.com/retep998/winapi-rs/blob/0.3.9/Cargo.toml#L25-L431
[`regex`]: https://crates.io/crates/regex
[`syn`]: https://crates.io/crates/syn
[syn-features]: https://docs.rs/syn/1.0.54/syn/#optional-features
[regex-features]: https://github.com/rust-lang/regex/blob/1.4.2/Cargo.toml#L33-L101
[regex-docs]: https://docs.rs/regex/1.4.2/regex/#crate-features
[`web-sys`]: https://crates.io/crates/web-sys
[web-sys-features]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/crates/web-sys/Cargo.toml#L32-L1395

## 扩展行为

[`serde_json`] 提供 [`preserve_order` feature][serde_json-preserve_order]，
可[改变 JSON map 的行为][serde_json-code]，以保留 key 插入顺序。
注意它启用了可选依赖 [`indexmap`] 来实现该行为。

像这样改变行为时，要确保变更是 [SemVer 兼容] 的。
也就是说，开启该 feature 不应破坏通常在“关闭该 feature”下可编译的代码。

[`serde_json`]: https://crates.io/crates/serde_json
[serde_json-preserve_order]: https://github.com/serde-rs/json/blob/v1.0.60/Cargo.toml#L53-L56
[SemVer compatible]: features.md#semver-compatibility
[serde_json-code]: https://github.com/serde-rs/json/blob/v1.0.60/src/map.rs#L23-L26
[`indexmap`]: https://crates.io/crates/indexmap

## `no_std` 支持

有些包希望同时支持 [`no_std`] 与 `std` 环境。
这对嵌入式和资源受限平台很有用，同时仍允许在支持完整标准库的平台上提供扩展能力。

[`wasm-bindgen`] 定义了一个 [`std` feature][wasm-bindgen-std]，
并且[默认启用][wasm-bindgen-default]。
在库顶部，它[无条件启用 `no_std` 属性][wasm-bindgen-no_std]。
这可确保 `std` 与 [`std` prelude] 不会自动进入作用域。
随后在代码多个位置（[example1][wasm-bindgen-cfg1]、[example2][wasm-bindgen-cfg2]）
使用 `#[cfg(feature = "std")]` 条件启用依赖 `std` 的额外能力。

[`no_std`]: ../../reference/names/preludes.html#the-no_std-attribute
[`wasm-bindgen`]: https://crates.io/crates/wasm-bindgen
[`std` prelude]: ../../std/prelude/index.html
[wasm-bindgen-std]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L25
[wasm-bindgen-default]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L23
[wasm-bindgen-no_std]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L8
[wasm-bindgen-cfg1]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L270-L273
[wasm-bindgen-cfg2]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L67-L75

## 重新导出依赖 feature

有时重新导出依赖的 feature 会更方便。
这样依赖你 crate 的用户就能控制这些 feature，而无需直接声明那些依赖。
例如，[`regex`] 会[重新导出 feature][regex-re-export]，
这些 feature 来自 [`regex_syntax`][regex_syntax-features]。
`regex` 的用户无需了解 `regex_syntax`，但仍可访问其 feature。

[regex-re-export]: https://github.com/rust-lang/regex/blob/1.4.2/Cargo.toml#L65-L89
[regex_syntax-features]: https://github.com/rust-lang/regex/blob/1.4.2/regex-syntax/Cargo.toml#L17-L32

## C 库 Vendoring

有些包会为常见 C 库提供绑定（常称为 ["sys" crate][sys]）。
这类包有时允许你选择使用系统已安装 C 库，或从源码构建。
例如，[`openssl`] 有 [`vendored` feature][openssl-vendored]，
它会启用 [`openssl-sys`] 对应的 `vendored` feature。
`openssl-sys` 的构建脚本包含一些[条件逻辑][openssl-sys-cfg]，
使其改为从本地 OpenSSL 源码构建，而不是用系统版本。

[`curl-sys`] 也是类似示例，其 [`static-curl` feature][curl-sys-static]
会让它从源码构建 libcurl。
同时它还有 [`force-system-lib-on-osx`][curl-sys-macos] feature，
可[强制使用系统 libcurl][curl-sys-macos-code]，覆盖 static-curl 设置。

[`openssl`]: https://crates.io/crates/openssl
[`openssl-sys`]: https://crates.io/crates/openssl-sys
[sys]: build-scripts.md#-sys-packages
[openssl-vendored]: https://github.com/sfackler/rust-openssl/blob/openssl-v0.10.31/openssl/Cargo.toml#L19
[build script]: build-scripts.md
[openssl-sys-cfg]: https://github.com/sfackler/rust-openssl/blob/openssl-v0.10.31/openssl-sys/build/main.rs#L47-L54
[`curl-sys`]: https://crates.io/crates/curl-sys
[curl-sys-static]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/Cargo.toml#L49
[curl-sys-macos]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/Cargo.toml#L52
[curl-sys-macos-code]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/build.rs#L15-L20

## Feature 优先级

某些包的 feature 彼此互斥。
一种处理方式是指定 feature 的优先级。
[`log`] 就是一个例子。
它有[多个 feature][log-features]用于在编译期选择最大日志级别（文档见[这里][log-docs]）。
它通过 [`cfg-if`] 来[实现优先级选择][log-cfg-if]。
如果启用了多个 feature，会优先选择更高的 "max" 级别。

[`log`]: https://crates.io/crates/log
[log-features]: https://github.com/rust-lang/log/blob/0.4.11/Cargo.toml#L29-L42
[log-docs]: https://docs.rs/log/0.4.11/log/#compile-time-filters
[log-cfg-if]: https://github.com/rust-lang/log/blob/0.4.11/src/lib.rs#L1422-L1448
[`cfg-if`]: https://crates.io/crates/cfg-if

## Proc-macro 配套包

有些包的 proc-macro 与本体高度绑定。
但并非所有用户都需要 proc-macro。
将 proc-macro 设为可选依赖，可让用户按需选择是否包含。
这很有帮助，因为 proc-macro 版本有时必须与主包同步，
你不希望强迫用户同时声明两个依赖并手动保持同步。

例如 [`serde`] 提供 [`derive`][serde-derive] feature，
用于启用 [`serde_derive`] proc-macro。
`serde_derive` 与 `serde` 紧密耦合，因此采用了[等号版本要求][serde-equals] 来确保同步。

[`serde`]: https://crates.io/crates/serde
[`serde_derive`]: https://crates.io/crates/serde_derive
[serde-derive]: https://github.com/serde-rs/serde/blob/v1.0.118/serde/Cargo.toml#L34-L35
[serde-equals]: https://github.com/serde-rs/serde/blob/v1.0.118/serde/Cargo.toml#L17

## 仅 nightly 的 feature

有些包希望实验只在 Rust [nightly channel] 提供的 API 或语言特性。
但它们可能不想要求所有用户也使用 nightly。
例如 [`wasm-bindgen`] 提供 [`nightly` feature][wasm-bindgen-nightly]，
用于启用[扩展 API][wasm-bindgen-unsize]，
其依赖 [`Unsize`] 标记 trait（撰写本文时仅 nightly 可用）。

注意该 crate 根部通过 [`cfg_attr` 启用 nightly feature][wasm-bindgen-cfg_attr]。
同时要记住，[`feature` attribute] 与 Cargo feature 无关，
它用于启用实验性语言特性。

[`rand`] 的 [`simd_support` feature][rand-simd_support] 也是类似示例，
其依赖只可在 nightly 上构建。

[`wasm-bindgen`]: https://crates.io/crates/wasm-bindgen
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[wasm-bindgen-nightly]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L27
[wasm-bindgen-unsize]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/closure.rs#L257-L269
[`Unsize`]: ../../std/marker/trait.Unsize.html
[wasm-bindgen-cfg_attr]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L11
[`feature` attribute]: ../../unstable-book/index.html
[`rand`]: https://crates.io/crates/rand
[rand-simd_support]: https://github.com/rust-random/rand/blob/0.7.3/Cargo.toml#L40

## 实验性 feature

有些包希望在不承诺 API 稳定性的前提下尝试新功能。
这类 feature 通常会注明“实验性”，未来可能变化甚至失效，
即使在次版本发布中也可能如此。
例如 [`async-std`] 的 [`unstable` feature][async-std-unstable]，
用于[门控新 API][async-std-gate]。
用户可主动选择启用，但这些 API 可能尚未成熟到可以稳定依赖。

[`async-std`]: https://crates.io/crates/async-std
[async-std-unstable]: https://github.com/async-rs/async-std/blob/v1.8.0/Cargo.toml#L38-L42
[async-std-gate]: https://github.com/async-rs/async-std/blob/v1.8.0/src/macros.rs#L46
