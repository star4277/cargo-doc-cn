# 凭据提供器协议（Credential Provider Protocol）
本文描述如何构建 Cargo 凭据提供器。
关于如何配置或使用凭据提供器，请见 [Registry Authentication](registry-authentication.md)。

使用外部凭据提供器时，Cargo 会通过 stdin/stdout 与其通信，
消息以“单行 JSON”形式传递。

Cargo 总是会以 `--cargo-plugin` 参数执行凭据提供器。
这使得凭据提供器可执行文件可以具备 Cargo 需求之外的额外功能。
额外参数会通过 JSON 的 `args` 字段传递。

## JSON 消息
本文中的 JSON 为了可读性加入了换行。
实际消息中不能包含换行。

### 凭据 Hello
* 发送方：凭据提供器
* 目的：进程启动时标识支持的协议版本
```javascript
{
    "v":[1]
}
```

Cargo 发送的请求会带有 `v` 字段，取值为此处列出的某个版本。
如果 Cargo 不支持凭据提供器声明的任何版本，它会报错并关闭凭据进程。

### Registry 信息
* 发送方：Cargo
这不是独立消息，而是作为 Cargo 所有消息中的 `registry` 字段出现。
```javascript
{
    // registry 的 index URL
    "index-url":"https://github.com/rust-lang/crates.io-index",
    // 配置中的 registry 名称（可选）
    "name": "crates-io",
    // 访问需认证 registry 时收到的 HTTP 头（可选）
    "headers": ["WWW-Authenticate: cargo"]
}
```

### Login 请求
* 发送方：Cargo
* 目的：收集并存储凭据
```javascript
{
    // 协议版本
    "v":1,
    // 执行动作：login
    "kind":"login",
    // Registry 信息（见 Registry 信息）
    "registry":{"index-url":"sparse+https://registry-url/index/", "name": "my-registry"},
    // 用户通过 stdin 或命令行指定的 token（可选）
    "token": "<the token value>",
    // 用户可访问以获取 token 的 URL（可选）
    "login-url": "http://registry-url/login",
    // 额外命令行参数（可选）
    "args":[]
}
```

如果设置了 `token` 字段，凭据提供器应使用该 token。
如果未设置 `token`，凭据提供器应提示用户输入 token。

除了配置里给凭据提供器传入的参数外，`cargo login` 还支持通过
`cargo login -- <additional args>` 传递额外命令行参数。
这些参数会在 `args` 字段中出现在配置参数之后。

### Read 请求
* 发送方：Cargo
* 目的：获取“读取 crate 信息”所需凭据
```javascript
{
    // 协议版本
    "v":1,
    // 请求类型：获取凭据
    "kind":"get",
    // 执行动作：读取 crate 信息
    "operation":"read",
    // Registry 信息（见 Registry 信息）
    "registry":{"index-url":"sparse+https://registry-url/index/", "name": "my-registry"},
    // 额外命令行参数（可选）
    "args":[]
}
```

### Publish 请求
* 发送方：Cargo
* 目的：获取“发布 crate”所需凭据
```javascript
{
    // 协议版本
    "v":1,
    // 请求类型：获取凭据
    "kind":"get",
    // 执行动作：发布 crate
    "operation":"publish",
    // crate 名称
    "name":"sample",
    // crate 版本
    "vers":"0.1.0",
    // crate 校验和
    "cksum":"...",
    // Registry 信息（见 Registry 信息）
    "registry":{"index-url":"sparse+https://registry-url/index/", "name": "my-registry"},
    // 额外命令行参数（可选）
    "args":[]
}
```

### Get 成功响应
* 发送方：凭据提供器
* 目的：向 Cargo 返回凭据
```javascript
{"Ok":{
    // 响应类型：对应 get 请求
    "kind":"get",
    // 发送给 registry 的 token
    "token":"...",
    // 缓存控制，可选值：
    // * "never": 不缓存
    // * "session": 当前 cargo 会话内缓存
    // * "expires": 当前 cargo 会话内缓存至过期时间
    "cache":"expires",
    // Unix 时间戳（仅当 "cache": "expires"）
    "expiration":1693942857,
    // token 是否与操作无关
    "operation_independent":true
}}
```

`token` 会作为 `Authorization` HTTP 头的值发送给 registry。

`operation_independent` 表示 token 能否跨不同操作（如发布或拉取）复用缓存。
通常应设为 `true`，除非提供器希望生成“仅针对特定操作”的 token。

### Login 成功响应
* 发送方：凭据提供器
* 目的：表示登录成功
```javascript
{"Ok":{
    // 响应类型：对应 login 请求
    "kind":"login"
}}
```

### Logout 成功响应
* 发送方：凭据提供器
* 目的：表示退出登录成功
```javascript
{"Ok":{
    // 响应类型：对应 logout 请求
    "kind":"logout"
}}
```

### 失败响应（URL 不支持）
* 发送方：凭据提供器
* 目的：向 Cargo 返回错误信息
```javascript
{"Err":{
    "kind":"url-not-supported"
}}
```
如果凭据提供器只处理特定 registry URL，而当前 URL 不受支持，应返回该错误。
如果有其他可用提供器，Cargo 会尝试下一个。

### 失败响应（not found）
* 发送方：凭据提供器
* 目的：向 Cargo 返回错误信息
```javascript
{"Err":{
    // 错误：提供器中找不到凭据
    "kind":"not-found"
}}
```
当找不到凭据时返回。
这在以下场景是预期行为：
- `get` 请求但凭据不可用
- `logout` 请求但没有可删除凭据

### 失败响应（operation not supported）
* 发送方：凭据提供器
* 目的：向 Cargo 返回错误信息
```javascript
{"Err":{
    // 错误：不支持该操作
    "kind":"operation-not-supported"
}}
```
当凭据提供器不支持请求操作时返回。
例如某提供器只支持 `get`，但收到了 `login` 请求。

### 失败响应（other）
* 发送方：凭据提供器
* 目的：向 Cargo 返回错误信息
```javascript
{"Err":{
    // 错误：其他失败
    "kind":"other",
    // 向用户显示的错误信息
    "message": "free form string error message",
    // 错误原因链（可选）
    "caused-by": ["cause 1", "cause 2"]
}}
```

## 请求“读取操作 token”的通信示例
1. Cargo 启动凭据进程，并捕获其 stdin 与 stdout。
2. 凭据进程向 Cargo 发送 Hello 消息。
    ```javascript
    { "v": [1] }
   ```
3. Cargo 向凭据进程发送 CredentialRequest（为便于阅读加入换行）。
    ```javascript
    {
        "v": 1,
        "kind": "get",
        "operation": "read",
        "registry":{"index-url":"sparse+https://registry-url/index/"}
    }
    ```
4. 凭据进程向 Cargo 发送 CredentialResponse（为便于阅读加入换行）。
    ```javascript
    {
        "token": "...",
        "cache": "session",
        "operation_independent": true
    }
    ```
5. Cargo 关闭到凭据提供器的 stdin 管道，凭据进程退出。
6. Cargo 在当前会话剩余生命周期（直到 Cargo 退出）内，
   与该 registry 交互时都使用该 token。
