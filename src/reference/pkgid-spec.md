# Package ID 规范

## Package ID 规范

Cargo 的子命令经常需要在依赖图中定位某个特定 package，
用于更新、清理、构建等操作。
为此，Cargo 支持 *Package ID 规范（Package ID Specifications）*。
规范本质上是一个字符串，用于在 package 图中唯一标识一个 package。

规范可以是完整形式，例如
`registry+https://github.com/rust-lang/crates.io-index#regex@1.4.3`，
也可以是简写形式，例如 `regex`。
只要简写能在依赖图中唯一标识一个 package，就可以使用。
若存在歧义，可添加限定信息使其唯一。
例如依赖图中存在两个 `regex` 版本时，可加版本限定：
`regex@1.4.3`。

Cargo 输出的 Package ID（例如 [cargo metadata](../commands/cargo-metadata.md)
输出）都是完整形式。

### 规范语法

Package ID 规范的形式化语法如下：

```notrust
spec := pkgname |
        [ kind "+" ] proto "://" hostname-and-path [ "?" query] [ "#" ( pkgname | semver ) ]
query = ( "branch" | "tag" | "rev" ) "=" ref
pkgname := name [ ("@" | ":" ) semver ]
semver := digits [ "." digits [ "." digits [ "-" prerelease ] [ "+" build ]]]

kind = "registry" | "git" | "path"
proto := "http" | "git" | "file" | ...
```

其中方括号表示可选内容。

URL 形式可用于 git 依赖，或用于区分来自不同来源（例如不同 registry）的 package。

### 规范示例

以下示例都指向 `crates.io` 上的 `regex` package：

| Spec                                                              | Name    | Version |
|:------------------------------------------------------------------|:-------:|:-------:|
| `regex`                                                           | `regex` | `*`     |
| `regex@1.4`                                                       | `regex` | `1.4.*` |
| `regex@1.4.3`                                                     | `regex` | `1.4.3` |
| `https://github.com/rust-lang/crates.io-index#regex`              | `regex` | `*`     |
| `https://github.com/rust-lang/crates.io-index#regex@1.4.3`        | `regex` | `1.4.3` |
| `registry+https://github.com/rust-lang/crates.io-index#regex@1.4.3` | `regex` | `1.4.3` |

下面是几个不同 git 依赖的规范示例：

| Spec                                                       | Name             | Version  |
|:-----------------------------------------------------------|:----------------:|:--------:|
| `https://github.com/rust-lang/cargo#0.52.0`                | `cargo`          | `0.52.0` |
| `https://github.com/rust-lang/cargo#cargo-platform@0.1.2`  | <nobr>`cargo-platform`</nobr> | `0.1.2`  |
| `ssh://git@github.com/rust-lang/regex.git#regex@1.4.3`     | `regex`          | `1.4.3`  |
| `git+ssh://git@github.com/rust-lang/regex.git#regex@1.4.3` | `regex`          | `1.4.3`  |
| `git+ssh://git@github.com/rust-lang/regex.git?branch=dev#regex@1.4.3` | `regex`          | `1.4.3`  |

文件系统上的本地 package 可通过 `file://` URL 引用：

| Spec                                        | Name  | Version |
|:--------------------------------------------|:-----:|:-------:|
| `file:///path/to/my/project/foo`            | `foo` | `*`     |
| `file:///path/to/my/project/foo#1.1.8`      | `foo` | `1.1.8` |
| `path+file:///path/to/my/project/foo#1.1.8` | `foo` | `1.1.8` |

### 规范的简洁性

该机制的目标是同时支持“简洁写法”和“完整写法”来引用依赖图中的 package。
有歧义的引用可能对应一个或多个 package。
多数命令在同一规范可对应多个 package 时会报错。
