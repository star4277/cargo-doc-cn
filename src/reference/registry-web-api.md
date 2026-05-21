# Web API

registry 可以在 `config.json` 指定的位置提供 Web API，
以支持下列动作。

Cargo 会在需要认证的请求中附带 `Authorization` 头。
其值为 API token。
如果 token 无效，服务端应返回 403。
用户通常通过访问 registry 网站获取 token，
Cargo 可通过 [`cargo login`] 保存该 token，
也可通过命令行直接传入。

成功响应使用 2xx 状态码。
失败应使用合适状态码（如 404）。
失败响应应返回如下 JSON 对象结构：

```javascript
{
    // 向用户展示的错误数组。
    "errors": [
        {
            // 错误消息字符串。
            "detail": "error message text"
        }
    ]
}
```

如果响应体是该结构，即便状态码是 200，Cargo 也会向用户显示详细错误信息。
若状态码表示错误且响应体不是该结构，Cargo 会显示用于帮助排查服务端错误的通用信息。
返回 `errors` 对象可让 registry 提供更详细、面向用户的错误提示。

为保持向后兼容，服务端应忽略未知查询参数或 JSON 字段。
JSON 字段缺失时应视为 `null`。
端点路径中的 `v1` 表示版本，
如果未来需要兼容回退，Cargo 负责处理。

Cargo 会为所有请求设置 `User-Agent`，例如
`cargo/1.32.0 (8610973aa 2019-01-02)`。
用户可通过配置修改该值。该行为自 1.29 起添加。

其他请求头因端点而异，详见下文。

## Publish

- Endpoint: `/api/v1/crates/new`
- Method: PUT
- Authorization: Included
- Headers:
    - `Content-Type`: `application/octet-stream`
    - `Accept`: `application/json`
- Body: Included（见下文）

publish 端点用于发布 crate 新版本。
服务端应校验 crate、使其可下载，并将其加入 index。

成功响应返回前，不要求 index 已完成更新。
成功响应后，Cargo 会在短时间内轮询 index，检查新 crate 是否已加入。
若短时间内仍未出现，Cargo 会提示警告，告知用户新 crate 暂不可用。

Cargo 发送的数据体结构：

- 32 位无符号小端整数：JSON 数据长度。
- package 元数据（JSON 对象）。
- 32 位无符号小端整数：`.crate` 文件长度。
- `.crate` 文件内容。

下面是带注释的 JSON 对象示例。
其中关于 [crates.io] 的限制仅用于举例说明可做哪些校验，
并非 [crates.io] 的完整限制列表。

```javascript
{
    // The name of the package.
    "name": "foo",
    // The version of the package being published.
    "vers": "0.1.0",
    // Array of direct dependencies of the package.
    "deps": [
        {
            // Name of the dependency.
            // If the dependency is renamed from the original package name,
            // this is the original name. The new package name is stored in
            // the `explicit_name_in_toml` field.
            "name": "rand",
            // The semver requirement for this dependency.
            "version_req": "^0.6",
            // Array of features (as strings) enabled for this dependency.
            "features": ["i128_support"],
            // Boolean of whether or not this is an optional dependency.
            "optional": false,
            // Boolean of whether or not default features are enabled.
            "default_features": true,
            // The target platform for the dependency.
            // null if not a target dependency.
            // Otherwise, a string such as "cfg(windows)".
            "target": null,
            // The dependency kind.
            // "dev", "build", or "normal".
            "kind": "normal",
            // The URL of the index of the registry where this dependency is
            // from as a string. If not specified or null, it is assumed the
            // dependency is in the current registry.
            "registry": null,
            // If the dependency is renamed, this is a string of the new
            // package name. If not specified or null, this dependency is not
            // renamed.
            "explicit_name_in_toml": null,
        }
    ],
    // Set of features defined for the package.
    // Each feature maps to an array of features or dependencies it enables.
    // Cargo does not impose limitations on feature names, but crates.io
    // requires alphanumeric ASCII, `_` or `-` characters.
    "features": {
        "extras": ["rand/simd_support"]
    },
    // List of strings of the authors.
    // May be empty.
    "authors": ["Alice <a@example.com>"],
    // Description field from the manifest.
    // May be null. crates.io requires at least some content.
    "description": null,
    // String of the URL to the website for this package's documentation.
    // May be null.
    "documentation": null,
    // String of the URL to the website for this package's home page.
    // May be null.
    "homepage": null,
    // String of the content of the README file.
    // May be null.
    "readme": null,
    // String of a relative path to a README file in the crate.
    // May be null.
    "readme_file": null,
    // Array of strings of keywords for the package.
    "keywords": [],
    // Array of strings of categories for the package.
    "categories": [],
    // String of the license for the package.
    // May be null. crates.io requires either `license` or `license_file` to be set.
    "license": null,
    // String of a relative path to a license file in the crate.
    // May be null.
    "license_file": null,
    // String of the URL to the website for the source repository of this package.
    // May be null.
    "repository": null,
    // Optional object of "status" badges. Each value is an object of
    // arbitrary string to string mappings.
    // crates.io has special interpretation of the format of the badges.
    "badges": {
        "travis-ci": {
            "branch": "master",
            "repository": "rust-lang/cargo"
        }
    },
    // The `links` string value from the package's manifest, or null if not
    // specified. This field is optional and defaults to null.
    "links": null,
    // The minimal supported Rust version (optional)
    // This must be a valid version requirement without an operator (e.g. no `=`)
    "rust_version": null
}
```

成功响应包含如下 JSON 对象：

```javascript
{
    // Optional object of warnings to display to the user.
    "warnings": {
        // Array of strings of categories that are invalid and ignored.
        "invalid_categories": [],
        // Array of strings of badge names that are invalid and ignored.
        "invalid_badges": [],
        // Array of strings of arbitrary warnings to display to the user.
        "other": []
    }
}
```

## Yank

- Endpoint: `/api/v1/crates/{crate_name}/{version}/yank`
- Method: DELETE
- Authorization: Included
- Headers:
    - `Accept`: `application/json`
- Body: None

yank 端点会把 index 中该 crate 指定版本的 `yank` 字段设为 `true`。

成功响应包含：

```javascript
{
    // Indicates the yank succeeded, always true.
    "ok": true,
}
```

## Unyank

- Endpoint: `/api/v1/crates/{crate_name}/{version}/unyank`
- Method: PUT
- Authorization: Included
- Headers:
    - `Accept`: `application/json`
- Body: None

unyank 端点会把 index 中该 crate 指定版本的 `yank` 字段设为 `false`。

成功响应包含：

```javascript
{
    // Indicates the unyank succeeded, always true.
    "ok": true,
}
```

## Owners

Cargo 本身并不内建用户与 owner 概念，
但提供了 `owner` 命令来协助管理谁有权限控制某个 crate。
具体如何处理用户与 owner，取决于 registry。
关于 [crates.io] 如何通过 GitHub 用户与团队处理 owner，
见[发布文档]。

### Owners: List

- Endpoint: `/api/v1/crates/{crate_name}/owners`
- Method: GET
- Authorization: Included
- Headers:
    - `Accept`: `application/json`
- Body: None

owners 端点返回该 crate 的 owner 列表。

成功响应包含：

```javascript
{
    // Array of owners of the crate.
    "users": [
        {
            // Unique unsigned 32-bit integer of the owner.
            "id": 70,
            // The unique username of the owner.
            "login": "github:rust-lang:core",
            // Name of the owner.
            // This is optional and may be null.
            "name": "Core",
        }
    ]
}
```

### Owners: Add

- Endpoint: `/api/v1/crates/{crate_name}/owners`
- Method: PUT
- Authorization: Included
- Headers:
    - `Content-Type`: `application/json`
    - `Accept`: `application/json`
- Body: Included（见下文）

PUT 请求会向 registry 提交“为 crate 添加 owner”的请求。
具体处理方式由 registry 决定。
例如 [crates.io] 会向用户发送邀请，用户接受后才会加入。

请求体应包含：

```javascript
{
    // Array of `login` strings of owners to add.
    "users": ["login_name"]
}
```

成功响应包含：

```javascript
{
    // Indicates the add succeeded, always true.
    "ok": true,
    // A string to be displayed to the user.
    "msg": "user ehuss has been invited to be an owner of crate cargo"
}
```

### Owners: Remove

- Endpoint: `/api/v1/crates/{crate_name}/owners`
- Method: DELETE
- Authorization: Included
- Headers:
    - `Content-Type`: `application/json`
    - `Accept`: `application/json`
- Body: Included（见下文）

DELETE 请求会从 crate 中移除 owner。
请求体应包含：

```javascript
{
    // Array of `login` strings of owners to remove.
    "users": ["login_name"]
}
```

成功响应包含：

```javascript
{
    // Indicates the remove succeeded, always true.
    "ok": true
    // A string to be displayed to the user. Currently ignored by cargo.
    "msg": "owners successfully removed",
}
```

## Search

- Endpoint: `/api/v1/crates`
- Method: GET
- Authorization: Not Included
- Headers:
    - `Accept`: `application/json`
- Body: None
- Query Parameters:
    - `q`: 搜索查询字符串。
    - `per_page`: 结果数量，默认 10，最大 100。

search 请求会按照服务端定义的条件搜索 crate。

成功响应包含：

```javascript
{
    // Array of results.
    "crates": [
        {
            // Name of the crate.
            "name": "rand",
            // The highest version available.
            "max_version": "0.6.1",
            // Textual description of the crate.
            "description": "Random number generators and other randomness functionality.\n",
        }
    ],
    "meta": {
        // Total number of results available on the server.
        "total": 119
    }
}
```

## Login

- Endpoint: `/me`

“login” 端点并不是真正的 API 请求。
它仅用于 [`cargo login`] 命令展示一个 URL，
引导用户在浏览器中登录并获取 API token。

[`cargo login`]: ../commands/cargo-login.md
[`cargo package`]: ../commands/cargo-package.md
[`cargo publish`]: ../commands/cargo-publish.md
[alphanumeric]: ../../std/primitive.char.html#method.is_alphanumeric
[config]: config.md
[crates.io]: https://crates.io/
[publishing documentation]: publishing.md#cargo-owner
