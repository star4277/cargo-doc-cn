# 覆盖依赖

在不少场景下，你会希望覆盖依赖。
这些场景大多都归结为：你想在某个 crate 发布到 [crates.io] 之前就先使用它。
例如：

* 你正在开发的某个 crate 也被一个更大的应用使用，你希望在这个大应用里先验证对该库的 bug 修复。
* 你并未直接维护的上游 crate，在其 git 仓库的 master 分支上已有新功能或 bug 修复，你想先试用。
* 你即将发布 crate 的新主版本，但希望先在整个 package 范围做集成测试，确认新主版本可用。
* 你向上游 crate 提交了 bug 修复，但希望你的应用立刻依赖修复后的版本，而不是等待 PR 合并。

这些场景都可以通过 [`[patch]` manifest
段](#the-patch-section)解决。

本章会演示几个不同用例，并说明覆盖依赖的几种方式。

* 示例用例
    * [测试 bug 修复](#testing-a-bugfix)
    * [使用尚未发布的次版本](#working-with-an-unpublished-minor-version)
        * [覆盖仓库 URL](#overriding-repository-url)
    * [预发布破坏性变更](#prepublishing-a-breaking-change)
    * [在多个版本中使用 `[patch]`](#using-patch-with-multiple-versions)
* 参考
    * [`[patch]` 段](#the-patch-section)
    * [`[replace]` 段](#the-replace-section)
    * [`paths` 覆盖](#paths-overrides)

> **注意**：另见在依赖声明中指定 [multiple locations]。
> 这可用于在本地 package 内，只覆盖单个依赖声明的来源。

## 测试 bug 修复

假设你在使用 [`uuid` crate]，开发过程中发现了一个 bug。
你决定自己修它。最初你的 manifest 可能是：

[`uuid` crate]: https://crates.io/crates/uuid

```toml
[package]
name = "my-library"
version = "0.1.0"

[dependencies]
uuid = "1.0"
```

第一步，先在本地 clone [`uuid` 仓库][uuid-repository]：

```console
$ git clone https://github.com/uuid-rs/uuid.git
```

接着把 `my-library` 的 manifest 改为：

```toml
[patch.crates-io]
uuid = { path = "../path/to/uuid" }
```

这里声明的是：我们要用一个新依赖去 *patch* `crates-io` 这个源。
效果上相当于把本地 checkout 的 `uuid`，加入当前 package 看到的 crates.io 注册表视图。

接下来要确保 lock 文件更新为使用这个新版本的 `uuid`，
让 package 使用本地副本而非 crates.io 上的版本。
`[patch]` 的工作方式是：加载 `../path/to/uuid` 这个依赖，
并在查询 crates.io 的 `uuid` 版本时，把“本地版本”也一起返回。

这意味着本地 checkout 的版本号很关键，它会影响 patch 是否会被选中。
我们的 manifest 写的是 `uuid = "1.0"`，
即只会解析到 `>= 1.0.0, < 2.0.0`。
同时 Cargo 的贪心解析策略会尽量选这个范围内的最大版本。
通常问题不大，因为 git 仓库里的版本一般会大于或至少等于 crates.io 已发布最大版本，
但这点一定要记住。

一般来说你现在只需：

```console
$ cargo build
   Compiling uuid v1.0.0 (.../uuid)
   Compiling my-library v0.1.0 (.../my-library)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

完成。现在你已在使用本地 `uuid`（注意输出括号中的路径）。
如果你没看到本地路径版本被构建，可能需要执行
`cargo update uuid --precise $version`，
其中 `$version` 是你本地 checkout 的 `uuid` 版本号。

修好 bug 之后，你大概率会向 `uuid` 提交 PR。
提交后也可以更新 `[patch]` 段。
`[patch]` 内条目和 `[dependencies]` 写法一致，
所以当 PR 合并后，你可以把 `path` 依赖改成：

```toml
[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid.git' }
```

[uuid-repository]: https://github.com/uuid-rs/uuid

## 使用尚未发布的次版本

下面从修 bug 切到加功能。
你在开发 `my-library` 时发现 `uuid` 需要一个全新功能。
你已经实现该功能、通过 `[patch]` 做过本地测试，并提交了 PR。
现在看一下在它正式发布前，如何继续使用并测试它。

假设 crates.io 上当前 `uuid` 版本是 `1.0.0`，
但 git 仓库 master 分支已经更新到 `1.0.1`，
且包含你提交的新功能。
要使用这个仓库，可把 `Cargo.toml` 改成：

```toml
[package]
name = "my-library"
version = "0.1.0"

[dependencies]
uuid = "1.0.1"

[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid.git' }
```

注意我们把本地对 `uuid` 的依赖升级到 `1.0.1`，
因为这就是 crate 发布后真正需要的版本。
该版本虽然还不存在于 crates.io，但可通过 manifest 的 `[patch]` 提供。

这样构建时，库会从 git 仓库拉取 `uuid`，
并解析到仓库中的 `1.0.1`，而不是去 crates.io 下载。
等 `1.0.1` 发布到 crates.io 后，就可以删除 `[patch]` 段。

还要注意 `[patch]` 是*传递生效*的。
假设你在更大的 package 中使用 `my-library`，例如：

```toml
[package]
name = "my-binary"
version = "0.1.0"

[dependencies]
my-library = { git = 'https://example.com/git/my-library' }
uuid = "1.0"

[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid.git' }
```

记住 `[patch]` 虽然是*传递*的，但只能定义在*顶层*，
所以 `my-library` 的使用方如有需要仍需重复写 `[patch]`。
不过在这个例子里，新 `uuid` 会同时作用于：
直接 `uuid` 依赖，以及 `my-library -> uuid` 依赖。
整个 crate 图中的 `uuid` 会统一解析到同一个版本 `1.0.1`，并从 git 仓库获取。

### 覆盖仓库 URL

如果你要覆盖的依赖来源不是 `crates.io`，
那么 `[patch]` 的写法要稍微调整。
例如依赖本身是 git 依赖时，可这样覆盖到本地路径：

```toml
[patch."https://github.com/your/repository"]
my-library = { path = "../my-library/path" }
```

就这么简单。

## 预发布破坏性变更

再来看新主版本场景，通常伴随破坏性变更。
沿用前面的 crate，我们要创建 `uuid` 的 `2.0.0`。
当上游变更都提交后，可把 `my-library` 的 manifest 改成：

```toml
[dependencies]
uuid = "2.0"

[patch.crates-io]
uuid = { git = "https://github.com/uuid-rs/uuid.git", branch = "2.0.0" }
```

完成。和前例一样，`2.0.0` 尚未存在于 crates.io，
但依然可以通过 `[patch]` + git 依赖引入。
再看一次上文的 `my-binary` manifest：

```toml
[package]
name = "my-binary"
version = "0.1.0"

[dependencies]
my-library = { git = 'https://example.com/git/my-library' }
uuid = "1.0"

[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid.git', branch = '2.0.0' }
```

注意这里实际上会解析出两个 `uuid` 版本。
`my-binary` 继续使用 `uuid` 的 1.x.y 系列，
而 `my-library` 会使用 `uuid` 2.0.0。
这让你可以在依赖图中逐步推广破坏性变更，
而不必一次性升级全部依赖。

## 在多个版本中使用 `[patch]`

你可以通过 `package` 键（用于依赖重命名）
同时 patch 同一个 crate 的多个版本。
例如 `serde` 的 1.* 系列有一个你想要的修复，
同时你还想试验 git 仓库里的 `serde` 2.0.0：

```toml
[patch.crates-io]
serde = { git = 'https://github.com/serde-rs/serde.git' }
serde2 = { git = 'https://github.com/example/serde.git', package = 'serde', branch = 'v2' }
```

第一条 `serde = ...` 表示 `serde` 1.* 从 git 仓库获取（拿到所需修复）。
第二条 `serde2 = ...` 表示同时也从 `https://github.com/example/serde` 的 `v2` 分支
拉取 `serde` package。
这里假设该分支的 `Cargo.toml` 标注版本 `2.0.0`。

注意当使用 `package` 键时，这里的 `serde2` 标识符本身会被忽略。
它只需要是一个不与其他 patch crate 冲突的唯一名字。

## `[patch]` 段

`Cargo.toml` 的 `[patch]` 段可用于用“其他副本”覆盖依赖。
语法类似 [`[dependencies]`][dependencies]：

```toml
[patch.crates-io]
foo = { git = 'https://github.com/example/foo.git' }
bar = { path = 'my/local/bar' }

[dependencies.baz]
git = 'https://github.com/example/baz.git'

[patch.'https://github.com/example/baz']
baz = { git = 'https://github.com/example/patched-baz.git', branch = 'my-branch' }
```

> **注意**：`[patch]` 也可作为[配置项](config.md)指定，
> 比如写在 `.cargo/config.toml`，或通过 CLI 参数
> `--config 'patch.crates-io.rand.path="rand"'`。
> 这对“只在本地生效、不想提交”的改动，或临时测试 patch 很有用。

`[patch]` 由“类似依赖声明”的子表构成。
`[patch]` 后面的每个键要么是被 patch 的源 URL，要么是某个 registry 名称。
可用 `crates-io` 表示默认注册表 [crates.io]。
上例第一个 `[patch]` 展示了覆盖 [crates.io]，
第二个 `[patch]` 展示了覆盖 git 源。

这些表中的每个条目，都是普通依赖声明，写法与 manifest 的 `[dependencies]` 相同。
`[patch]` 中列出的依赖会被解析并用于 patch 指定 URL 的源。
上面的片段等价于：
把 `crates-io` 源（即 crates.io）中的 `foo` 与 `bar` 换成指定副本；
并把 `https://github.com/example/baz` 源中的 `baz` 换成来自其他位置的 `my-branch`。

源可以被 patch 成“原本不存在的 crate 版本”，
也可以 patch 成“源中已存在的版本”。
如果 patch 的版本在源中已存在，那么源里的原始 crate 会被替换。

Cargo 只会读取 workspace 根目录 `Cargo.toml` 里的 patch 设置。
在依赖中定义的 patch 设置会被忽略。

## `[replace]` 段

> **注意**：`[replace]` 已弃用。请改用
> [`[patch]`](#the-patch-section)。

`Cargo.toml` 的该段也可用于用其他副本覆盖依赖。
语法类似 `[dependencies]`：

```toml
[replace]
"foo:0.1.0" = { git = 'https://github.com/example/foo.git' }
"bar:1.0.2" = { path = 'my/local/bar' }
```

`[replace]` 表中的每个键都是一个 [package ID
specification](pkgid-spec.md)，
可用于任意选择依赖图中的节点进行覆盖（必须写 3 段版本号）。
每个键的值与 `[dependencies]` 指定依赖的语法一致，
但不能指定 features。
注意：被覆盖 crate 与用于替换的 crate 必须同名同版本，
但来源可不同（如 git 或本地路径）。

Cargo 只会读取 workspace 根目录 `Cargo.toml` 中的 replace 设置。
在依赖里定义的 replace 设置会被忽略。

## `paths` 覆盖

有时你只是临时修改某个 crate，
并不想像上面 `[patch]` 那样改 `Cargo.toml`。
这种场景 Cargo 提供了能力更受限的覆盖方式：**path overrides**。

Path override 配置写在 [`.cargo/config.toml`](config.md)，而不是 `Cargo.toml`。
在 `.cargo/config.toml` 中可指定 `paths` 键：

```toml
paths = ["/path/to/uuid"]
```

该数组应填入包含 `Cargo.toml` 的目录。
在这个例子里我们只添加 `uuid`，
因此只有它会被覆盖。
该路径可以是绝对路径，也可以是相对 `.cargo` 文件夹所在目录的相对路径。

不过 path overrides 比 `[patch]` 限制更多：
它不能改变依赖图结构。
使用 path 替换时，新 `Cargo.toml` 的依赖集合必须与原先完全一致。
例如，这意味着你不能用 path override 来测试“给 crate 新增一个依赖”。
这类场景必须使用 `[patch]`。
因此 path override 通常只适合快速修 bug，而非大改动。

> **注意**：使用本地配置覆盖路径，只对已发布到 [crates.io] 的 crate 生效。
> 你不能用这个功能告诉 Cargo 如何查找本地未发布 crate。


[crates.io]: https://crates.io/
[multiple locations]: specifying-dependencies.md#multiple-locations
[dependencies]: specifying-dependencies.md
