# 索引格式

下面定义 index 的格式。
Cargo 会不定期增加新特性，这些特性只有在“引入它们的 Cargo 版本及之后”才会被理解。
旧版 Cargo 可能无法使用依赖这些新特性的包。
不过，旧包的格式本身不应变化，因此旧版 Cargo 仍应能使用它们。

## 索引配置
index 根目录包含一个名为 `config.json` 的文件，
其中是 Cargo 访问 registry 所需的 JSON 信息。
下面是 [crates.io] 的配置文件示例：

```javascript
{
    "dl": "https://crates.io/api/v1/crates",
    "api": "https://crates.io"
}
```

字段说明：
- `dl`：用于下载 index 中列出的 crate 的 URL。
  该值可包含以下占位符，运行时会替换为对应值：

  - `{crate}`：crate 名称。
  - `{version}`：crate 版本。
  - `{prefix}`：由 crate 名计算出的目录前缀。
    例如 `cargo` 的前缀是 `ca/rg`。详见下文。
  - `{lowerprefix}`：`{prefix}` 的小写形式。
  - `{sha256-checksum}`：crate 的 sha256 校验和。

  如果未包含任何占位符，会在末尾自动追加
  `/{crate}/{version}/download`。
- `api`：Web API 的基础 URL。
  该键可选；若缺失，像 [`cargo publish`] 这类命令将不可用。
  Web API 见下文。该 URL 不应以斜杠结尾。
- `auth-required`：是否为私有 registry，且要求所有操作都认证，
  包括 API 请求、crate 下载、sparse index 更新。


## 下载端点
下载端点应返回请求 package 的 `.crate` 文件。
Cargo 支持 https、http、file URL，支持 HTTP 重定向、HTTP1、HTTP2。
TLS 支持细节取决于 Cargo 运行平台、Cargo 版本以及编译方式。

若 `config.json` 中设置 `auth-required: true`，
则 http(s) 下载请求会附带 `Authorization` 头。

## 索引文件
index 仓库其余部分由“每个 package 一个文件”组成，
文件名是 package 名的小写形式。
每个 package 版本在该文件中占一行。
文件目录结构分层如下：

- 名称长度为 1 的 package 放在 `1` 目录。
- 名称长度为 2 的 package 放在 `2` 目录。
- 名称长度为 3 的 package 放在 `3/{first-character}` 目录，
  其中 `{first-character}` 是 package 名的第一个字符。
- 其他 package 放在 `{first-two}/{second-two}` 目录：
  顶层目录是前两个字符，下一层子目录是第 3、4 个字符。
  例如 `cargo` 存在 `ca/rg/cargo`。

> 注意：虽然 index 文件名是小写，
> 但 `Cargo.toml` 和 index JSON 数据中包含 package 名的字段大小写敏感，
> 可以含大小写字符。

上面的目录名基于“转换为小写后的 package 名”计算，
对应占位符 `{lowerprefix}`。
若使用“原始 package 名（不做大小写转换）”计算目录，
对应占位符 `{prefix}`。
例如 `MyCrate` 的 `{prefix}` 为 `My/Cr`，
`{lowerprefix}` 为 `my/cr`。
一般推荐优先使用 `{prefix}`，但两者各有利弊。
在大小写不敏感文件系统上使用 `{prefix}` 会出现（无害但不优雅的）目录别名。
例如 `crate` 与 `CrateTwo` 的 `{prefix}` 分别是 `cr/at` 与 `Cr/at`；
在 Unix 上它们不同，在 Windows 上会映射到同一目录。
使用统一大小写目录可避免别名问题；
但在大小写敏感文件系统上，想兼容“不支持 `{prefix}`/`{lowerprefix}` 的旧 Cargo”会更难。
例如 nginx rewrite 规则能轻松构造 `{prefix}`，
却难以做大小写转换来构造 `{lowerprefix}`。

## 名称限制

registry 在向 index 添加 package 名时，应考虑施加限制。
Cargo 本身允许任意 [alphanumeric]、`-`、`_` 字符。
[crates.io] 有自己的限制，包括：

- 仅允许 ASCII 字符。
- 仅允许字母数字、`-`、`_`。
- 首字符必须是字母。
- 大小写不敏感冲突检测。
- 禁止仅 `-` 与 `_` 差异。
- 长度上限（最多 64）。
- 拒绝保留名称，例如 Windows 特殊文件名（如 "nul"）。

registry 应考虑采用类似限制，并评估安全影响，
例如 [IDN 同形异义攻击](https://en.wikipedia.org/wiki/IDN_homograph_attack)
及 [UTR36](https://www.unicode.org/reports/tr36/)、
[UTS39](https://www.unicode.org/reports/tr39/) 中的其他问题。

## 版本唯一性

index **必须**保证每个 package 的每个版本只出现一次。
这也包括忽略 SemVer build metadata。
例如 index **不得**同时出现 `1.0.7` 与 `1.0.7+extra` 两个条目。

## JSON Schema

package 文件中每一行都是一个 JSON 对象，描述该 package 的一个已发布版本。
下面是带注释的美化示例：

```javascript
{
    // The name of the package.
    // This must only contain alphanumeric, `-`, or `_` characters.
    "name": "foo",
    // The version of the package this row is describing.
    // This must be a valid version number according to the Semantic
    // Versioning 2.0.0 spec at https://semver.org/.
    "vers": "0.1.0",
    // Array of direct dependencies of the package.
    "deps": [
        {
            // Name of the dependency.
            // If the dependency is renamed from the original package name,
            // this is the new name. The original package name is stored in
            // the `package` field.
            "name": "rand",
            // The SemVer requirement for this dependency.
            // This must be a valid version requirement defined at
            // https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html.
            "req": "^0.6",
            // Array of features (as strings) enabled for this dependency.
            // Since Cargo 1.84, defaults to `[]` if not specified.
            "features": ["i128_support"],
            // Boolean of whether or not this is an optional dependency.
            // Since Cargo 1.84, defaults to `false` if not specified.
            "optional": false,
            // Boolean of whether or not default features are enabled.
            // Since Cargo 1.84, defaults to `true` if not specified.
            "default_features": true,
            // The target platform for the dependency.
            // If not specified or `null`, it is not a target dependency.
            // Otherwise, a string such as "cfg(windows)".
            "target": null,
            // The dependency kind.
            // "dev", "build", or "normal".
            // If not specified or `null`, it defaults to "normal".
            "kind": "normal",
            // The URL of the index of the registry where this dependency is
            // from as a string. If not specified or `null`, it is assumed the
            // dependency is in the current registry.
            "registry": null,
            // If the dependency is renamed, this is a string of the actual
            // package name. If not specified or `null`, this dependency is not
            // renamed.
            "package": null,
        }
    ],
    // A SHA256 checksum of the `.crate` file.
    "cksum": "d867001db0e2b6e0496f9fac96930e2d42233ecd3ca0413e0753d4c7695d289c",
    // Set of features defined for the package.
    // Each feature maps to an array of features or dependencies it enables.
    // Since Cargo 1.84, defaults to `{}` if not specified.
    "features": {
        "extras": ["rand/simd_support"]
    },
    // Boolean of whether or not this version has been yanked.
    "yanked": false,
    // The `links` string value from the package's manifest, or null if not
    // specified. This field is optional and defaults to null.
    "links": null,
    // An unsigned 32-bit integer value indicating the schema version of this
    // entry.
    //
    // If this is not specified, it should be interpreted as the default of 1.
    //
    // Cargo (starting with version 1.51) will ignore versions it does not
    // recognize. This provides a method to safely introduce changes to index
    // entries and allow older versions of cargo to ignore newer entries it
    // doesn't understand. Versions older than 1.51 ignore this field, and
    // thus may misinterpret the meaning of the index entry.
    //
    // The current values are:
    //
    // * 1: The schema as documented here, not including newer additions.
    //      This is honored in Rust version 1.51 and newer.
    // * 2: The addition of the `features2` field.
    //      This is honored in Rust version 1.60 and newer.
    "v": 2,
    // This optional field contains features with new, extended syntax.
    // Specifically, namespaced features (`dep:`) and weak dependencies
    // (`pkg?/feat`).
    //
    // This is separated from `features` because versions older than 1.19
    // will fail to load due to not being able to parse the new syntax, even
    // with a `Cargo.lock` file.
    //
    // Cargo will merge any values listed here with the "features" field.
    //
    // If this field is included, the "v" field should be set to at least 2.
    //
    // Registries are not required to use this field for extended feature
    // syntax, they are allowed to include those in the "features" field.
    // Using this is only necessary if the registry wants to support cargo
    // versions older than 1.19, which in practice is only crates.io since
    // those older versions do not support other registries.
    "features2": {
        "serde": ["dep:serde", "chrono?/serde"]
    }
    // The minimal supported Rust version (optional)
    // This must be a valid version requirement without an operator (e.g. no `=`)
    "rust_version": "1.60",
    // The publish time of this package version (optional).
    //
    // The format is a subset of ISO8601:
    // - `yyyy-mm-ddThh:mm:ssZ`
    // - no fractional seconds
    // - always `Z` for UTC timezone, no timezone offsets supported
    // - fields are 0-padded
    //
    // Example: 2025-11-12T19:30:12Z
    //
    // This should be the original publish time and not changed on any status changes,
    // like `yanked`.
    "pubtime": "2025-11-12T19:30:12Z"
}
```

这些 JSON 对象一旦添加后，不应修改，
唯一允许随时变化的是 `yanked` 字段。

> **注意**：index JSON 格式与 [Publish API] 及 [`cargo metadata`] 的 JSON 格式存在细微差异。
> 若你基于它们来生成 index 条目，建议仔细核对文档差异。
>
> 对于 [Publish API]，差异包括：
>
> * `deps`
>     * `name` --- 当依赖在 `Cargo.toml` 中被[重命名][renamed]时，
>       publish API 把原始包名放在 `name` 字段，别名放在 `explicit_name_in_toml` 字段。
>       index 则把别名放在 `name`，原始包名放在 `package`。
>     * `req` --- Publish API 中该字段名是 `version_req`。
> * `cksum` --- publish API 不提供校验和；registry 在写入 index 前需自行计算。
> * `features` --- 部分 feature 可能放到 `features2` 字段。
>   注意：这仅是 [crates.io] 的遗留兼容要求；其他 registry 通常无需调整 features map。
>   `v` 字段用于指示是否存在 `features2`。
> * publish API 还包含一些 index 不含字段，如 `description`、`readme`。
>   这些字段用于帮助 registry 在无需解包解析 `.crate` 的情况下获取展示元数据。
>   这些附加信息通常会写入 registry 服务端数据库。
> * 虽然此处有 `rust_version`，但 [crates.io] 会忽略该字段，
>   转而读取 `.crate` 内 `Cargo.toml` 的值。
>
> 对于 [`cargo metadata`]，差异包括：
>
> * `vers` --- `cargo metadata` 中字段名为 `version`。
> * `deps`
>   * `name` --- 当依赖在 `Cargo.toml` 中被[重命名][renamed]时，
>     `cargo metadata` 把原始包名放在 `name`，别名放在 `rename`。
>     index 则把别名放在 `name`，原始包名放在 `package`。
>   * `default_features` --- `cargo metadata` 中字段名为 `uses_default_features`。
>   * `registry` --- `cargo metadata` 用 `null` 表示该依赖来自 [crates.io]。
>     index 用 `null` 表示该依赖来自“当前 index 对应的同一 registry”。
>     当生成非 [crates.io] registry 的 index 条目时，
>     应把 `null` 转成 `https://github.com/rust-lang/crates.io-index`，
>     并把与当前 index 匹配的 URL 转回 `null`。
>   * `cargo metadata` 还包含一些额外字段，如 `source`、`path`。
> * index 包含额外字段，如 `yanked`、`cksum`、`v`。

[renamed]: specifying-dependencies.md#renaming-dependencies-in-cargotoml
[Publish API]: registry-web-api.md#publish
[`cargo metadata`]: ../commands/cargo-metadata.md

## 索引协议
Cargo 支持两种远程 registry 协议：`git` 和 `sparse`。
`git` 协议将 index 文件保存在 git 仓库中，
`sparse` 协议则通过 HTTP 拉取单个文件。

### Git 协议
git 协议的 index URL 不带协议前缀。
例如 [crates.io] 的 git index URL 是
`https://github.com/rust-lang/crates.io-index`。

Cargo 会在本地缓存该 git 仓库，以便高效增量拉取更新。

### Sparse 协议
sparse 协议在 registry URL 中使用 `sparse+` 前缀。
例如 [crates.io] 的 sparse index URL 是
`sparse+https://index.crates.io/`。

sparse 协议通过独立 HTTP 请求下载每个 index 文件。
由于会产生大量小请求，若服务端支持流水线和 HTTP/2，性能会显著提升。

#### Sparse 认证
Cargo 会先拉取 `config.json`，再拉取其他文件。
如果服务器返回 HTTP 401，Cargo 会认为该 registry 需要认证，
并携带认证 token 重新请求 `config.json`。

认证失败（或缺少认证 token）时，
服务端可在 `www-authenticate` 头里返回
`Cargo login_url="<URL>"` challenge，告知用户去哪里获取 token。

需要认证的 registry 必须在 `config.json` 设置 `auth-required: true`。

#### 缓存
Cargo 会缓存 crate 元数据文件，
并保存服务端返回的 `ETag` 或 `Last-Modified` 头。
刷新元数据时，Cargo 会发送 `If-None-Match` 或 `If-Modified-Since`，
让服务端在本地缓存有效时返回 HTTP 304 “Not Modified”，以节省时间与带宽。
若两者同时存在，Cargo 只使用 `ETag`。

#### 缓存失效
如果 registry 使用 CDN 或代理缓存 index 文件，
建议在文件更新时实现某种缓存失效机制。
否则用户可能在缓存清理前无法访问新 crate。

#### 不存在的 Crate
对不存在的 crate，registry 应返回
404 “Not Found”、410 “Gone” 或 451 “Unavailable For Legal Reasons”。

#### Sparse 限制
由于 lockfile 会存储 registry URL，
不建议同一 registry 同时提供两种协议。
关于迁移方案仍在 [#10964] 讨论中。
[crates.io] 是例外，因为当使用 sparse 协议时，
Cargo 内部会替换为等价 git URL。

如果某 registry 仍同时提供两种协议，
当前建议选定其中一种作为规范协议（canonical protocol），
另一种通过 [source replacement] 处理。


[`cargo publish`]: ../commands/cargo-publish.md
[alphanumeric]: ../../std/primitive.char.html#method.is_alphanumeric
[crates.io]: https://crates.io/
[source replacement]: ../reference/source-replacement.md
[#10964]: https://github.com/rust-lang/cargo/issues/10964
