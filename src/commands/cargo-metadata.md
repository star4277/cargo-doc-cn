# cargo-metadata(1)

## 名称

cargo-metadata --- 当前 package 的机器可读元数据

## 概要

`cargo metadata` [_options_]

## 描述

向 stdout 输出 JSON，包含当前 package 的 workspace 成员信息与已解析依赖。

输出格式在未来 Cargo 版本中可能变化。
建议加上 `--format-version` 标志，以提升代码的前向兼容性，
并确保输出格式符合你的预期。
关于兼容性约定，见[“兼容性”](#兼容性)。

如需在 Rust 中读取这些元数据，可使用
[cargo_metadata crate](https://crates.io/crates/cargo_metadata) 提供的 API。

## 输出格式

### 兼容性

在相同输出格式版本内，兼容性通常会保持。
但以下变更不被视为不兼容（非完整列表）：

* **新增字段** — 会按需新增字段。这有助于 Cargo 演进，而无需频繁提升格式版本。
* **为枚举类字段新增取值** — 与新增字段类似，可避免元数据演进停滞。
* **变更不透明表示形式** — 某些字段的内部表示属于实现细节。
  例如与 “Source ID” 相关字段会被当作不透明标识符，用于区分 package 或来源。
  除非文档明确保证，否则消费方不应依赖其内部格式。

### JSON 格式

JSON 输出格式如下：

```javascript
{
    /* Array of all packages in the workspace.
       It also includes all feature-enabled dependencies unless --no-deps is used.
    */
    "packages": [
        {
            /* The name of the package. */
            "name": "my-package",
            /* The version of the package. */
            "version": "0.1.0",
            /* The Package ID for referring to the
               package within the document and as the `--package` argument to many commands
            */
            "id": "file:///path/to/my-package#0.1.0",
            /* The license value from the manifest, or null. */
            "license": "MIT/Apache-2.0",
            /* The license-file value from the manifest, or null. */
            "license_file": "LICENSE",
            /* The description value from the manifest, or null. */
            "description": "Package description.",
            /* The source ID of the package, an "opaque" identifier representing
               where a package is retrieved from. See "Compatibility" above for
               the stability guarantee.

               This is null for path dependencies and workspace members.

               For other dependencies, it is a string with the format:
               - "registry+URL" for registry-based dependencies.
                 Example: "registry+https://github.com/rust-lang/crates.io-index"
               - "git+URL" for git-based dependencies.
                 Example: "git+https://github.com/rust-lang/cargo?rev=5e85ba14aaa20f8133863373404cb0af69eeef2c#5e85ba14aaa20f8133863373404cb0af69eeef2c"
               - "sparse+URL" for dependencies from a sparse registry
                 Example: "sparse+https://my-sparse-registry.org"

               The value after the `+` is not explicitly defined, and may change
               between versions of Cargo and may not directly correlate to other
               things, such as registry definitions in a config file. New source
               kinds may be added in the future which will have different `+`
               prefixed identifiers.
            */
            "source": null,
            /* Array of dependencies declared in the package's manifest. */
            "dependencies": [
                {
                    /* The name of the dependency. */
                    "name": "bitflags",
                    /* The source ID of the dependency. May be null, see
                       description for the package source.
                    */
                    "source": "registry+https://github.com/rust-lang/crates.io-index",
                    /* The version requirement for the dependency.
                       Dependencies without a version requirement have a value of "*".
                    */
                    "req": "^1.0",
                    /* The dependency kind.
                       "dev", "build", or null for a normal dependency.
                    */
                    "kind": null,
                    /* If the dependency is renamed, this is the new name for
                       the dependency as a string.  null if it is not renamed.
                    */
                    "rename": null,
                    /* Boolean of whether or not this is an optional dependency. */
                    "optional": false,
                    /* Boolean of whether or not default features are enabled. */
                    "uses_default_features": true,
                    /* Array of features enabled. */
                    "features": [],
                    /* The target platform for the dependency.
                       null if not a target dependency.
                    */
                    "target": "cfg(windows)",
                    /* The file system path for a local path dependency.
                       not present if not a path dependency.
                    */
                    "path": "/path/to/dep",
                    /* A string of the URL of the registry this dependency is from.
                       If not specified or null, the dependency is from the default
                       registry (crates.io).
                    */
                    "registry": null,
                    /* (unstable) Boolean flag of whether or not this is a pulbic
                       dependency. This field is only present when
                       `-Zpublic-dependency` is enabled.
                    */
                    "public": false
                }
            ],
            /* Array of Cargo targets. */
            "targets": [
                {
                    /* Array of target kinds.
                       - lib targets list the `crate-type` values from the
                         manifest such as "lib", "rlib", "dylib",
                         "proc-macro", etc. (default ["lib"])
                       - binary is ["bin"]
                       - example is ["example"]
                       - integration test is ["test"]
                       - benchmark is ["bench"]
                       - build script is ["custom-build"]
                    */
                    "kind": [
                        "bin"
                    ],
                    /* Array of crate types.
                       - lib and example libraries list the `crate-type` values
                         from the manifest such as "lib", "rlib", "dylib",
                         "proc-macro", etc. (default ["lib"])
                       - all other target kinds are ["bin"]
                    */
                    "crate_types": [
                        "bin"
                    ],
                    /* The name of the target.
                       For lib targets, dashes will be replaced with underscores.
                    */
                    "name": "my-package",
                    /* Absolute path to the root source file of the target. */
                    "src_path": "/path/to/my-package/src/main.rs",
                    /* The Rust edition of the target.
                       Defaults to the package edition.
                    */
                    "edition": "2018",
                    /* Array of required features.
                       This property is not included if no required features are set.
                    */
                    "required-features": ["feat1"],
                    /* Whether the target should be documented by `cargo doc`. */
                    "doc": true,
                    /* Whether or not this target has doc tests enabled, and
                       the target is compatible with doc testing.
                    */
                    "doctest": false,
                    /* Whether or not this target should be built and run with `--test`
                    */
                    "test": true
                }
            ],
            /* Set of features defined for the package.
               Each feature maps to an array of features or dependencies it
               enables.
            */
            "features": {
                "default": [
                    "feat1"
                ],
                "feat1": [],
                "feat2": []
            },
            /* Absolute path to this package's manifest. */
            "manifest_path": "/path/to/my-package/Cargo.toml",
            /* Package metadata.
               This is null if no metadata is specified.
            */
            "metadata": {
                "docs": {
                    "rs": {
                        "all-features": true
                    }
                }
            },
            /* List of registries to which this package may be published.
               Publishing is unrestricted if null, and forbidden if an empty array. */
            "publish": [
                "crates-io"
            ],
            /* Array of authors from the manifest.
               Empty array if no authors specified.
            */
            "authors": [
                "Jane Doe <user@example.com>"
            ],
            /* Array of categories from the manifest. */
            "categories": [
                "command-line-utilities"
            ],
            /* Optional string that is the default binary picked by cargo run. */
            "default_run": null,
            /* Optional string that is the minimum supported rust version */
            "rust_version": "1.56",
            /* Array of keywords from the manifest. */
            "keywords": [
                "cli"
            ],
            /* The readme value from the manifest or null if not specified. */
            "readme": "README.md",
            /* The repository value from the manifest or null if not specified. */
            "repository": "https://github.com/rust-lang/cargo",
            /* The homepage value from the manifest or null if not specified. */
            "homepage": "https://rust-lang.org",
            /* The documentation value from the manifest or null if not specified. */
            "documentation": "https://doc.rust-lang.org/stable/std",
            /* The default edition of the package.
               Note that individual targets may have different editions.
            */
            "edition": "2018",
            /* Optional string that is the name of a native library the package
               is linking to.
            */
            "links": null,
        }
    ],
    /* Array of members of the workspace.
       Each entry is the Package ID for the package.
    */
    "workspace_members": [
        "file:///path/to/my-package#0.1.0",
    ],
    /* Array of default members of the workspace.
       Each entry is the Package ID for the package.
    */
    "workspace_default_members": [
        "file:///path/to/my-package#0.1.0",
    ],
    // The resolved dependency graph for the entire workspace. The enabled
    // features are based on the enabled features for the "current" package.
    // Inactivated optional dependencies are not listed.
    //
    // This is null if --no-deps is specified.
    //
    // By default, this includes all dependencies for all target platforms.
    // The `--filter-platform` flag may be used to narrow to a specific
    // target triple.
    "resolve": {
        /* Array of nodes within the dependency graph.
           Each node is a package.
        */
        "nodes": [
            {
                /* The Package ID of this node. */
                "id": "file:///path/to/my-package#0.1.0",
                /* The dependencies of this package, an array of Package IDs. */
                "dependencies": [
                    "https://github.com/rust-lang/crates.io-index#bitflags@1.0.4"
                ],
                /* The dependencies of this package. This is an alternative to
                   "dependencies" which contains additional information. In
                   particular, this handles renamed dependencies.
                */
                "deps": [
                    {
                        /* The name of the dependency's library target.
                           If this is a renamed dependency, this is the new
                           name.
                        */
                        "name": "bitflags",
                        /* The Package ID of the dependency. */
                        "pkg": "https://github.com/rust-lang/crates.io-index#bitflags@1.0.4"
                        /* Array of dependency kinds. Added in Cargo 1.40. */
                        "dep_kinds": [
                            {
                                /* The dependency kind.
                                   "dev", "build", or null for a normal dependency.
                                */
                                "kind": null,
                                /* The target platform for the dependency.
                                   null if not a target dependency.
                                */
                                "target": "cfg(windows)"
                            }
                        ]
                    }
                ],
                /* Array of features enabled on this package. */
                "features": [
                    "default"
                ]
            }
        ],
        /* The package in the current working directory (if --manifest-path is not given).
           This is null if there is a virtual workspace. Otherwise it is
           the Package ID of the package.
        */
        "root": "file:///path/to/my-package#0.1.0",
    },
    /* The absolute path to the target directory where Cargo places its output. */
    "target_directory": "/path/to/my-package/target",
    /* The absolute path to the build directory where Cargo places intermediate build artifacts. (unstable) */
    "build_directory": "/path/to/my-package/build-dir",
    /* The version of the schema for this metadata structure.
       This will be changed if incompatible changes are ever made.
    */
    "version": 1,
    /* The absolute path to the root of the workspace. */
    "workspace_root": "/path/to/my-package"
    /* Workspace metadata.
       This is null if no metadata is specified. */
    "metadata": {
        "docs": {
            "rs": {
                "all-features": true
            }
        }
    }
}
````

说明：
- 关于 `"id"` 字段语法，请见参考文档中的 [Package ID Specifications]。

## 选项

### 输出选项

<dl>

<dt class="option-term" id="option-cargo-metadata---no-deps"><a class="option-anchor" href="#option-cargo-metadata---no-deps"><code>--no-deps</code></a></dt>
<dd class="option-desc"><p>只输出 workspace 成员信息，不拉取依赖。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---format-version"><a class="option-anchor" href="#option-cargo-metadata---format-version"><code>--format-version</code> <em>version</em></a></dt>
<dd class="option-desc"><p>指定输出格式版本。目前仅支持 <code>1</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---filter-platform"><a class="option-anchor" href="#option-cargo-metadata---filter-platform"><code>--filter-platform</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>过滤 <code>resolve</code> 输出，仅包含指定 <a href="../appendix/glossary.html#target">target triple</a> 的依赖。
可使用字面值 <code>"host-tuple"</code>，其会在内部替换为主机 target。
不加该参数时，resolve 会包含所有 targets。</p>
<p>注意，<code>"packages"</code> 数组列出的依赖仍是全量依赖。
每个 package 定义旨在原样复现 <code>Cargo.toml</code> 中的信息。</p>
</dd>


</dl>

### Feature 选择

feature 标志用于控制启用哪些 feature。若未给出 feature 选项，
每个选中 package 都会启用 `default` feature。

更多细节见[feature 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-metadata--F"><a class="option-anchor" href="#option-cargo-metadata--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-metadata---features"><a class="option-anchor" href="#option-cargo-metadata---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表，以空格或逗号分隔。workspace 成员的 feature
可用 <code>package-name/feature-name</code> 语法启用。此标志可多次指定，最终会启用
所有指定 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---all-features"><a class="option-anchor" href="#option-cargo-metadata---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---no-default-features"><a class="option-anchor" href="#option-cargo-metadata---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-metadata--v"><a class="option-anchor" href="#option-cargo-metadata--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-metadata---verbose"><a class="option-anchor" href="#option-cargo-metadata---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>启用详细输出。可指定两次得到“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata--q"><a class="option-anchor" href="#option-cargo-metadata--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-metadata---quiet"><a class="option-anchor" href="#option-cargo-metadata---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---color"><a class="option-anchor" href="#option-cargo-metadata---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。有效值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>

</dl>

### Manifest 选项

<dl>
<dt class="option-term" id="option-cargo-metadata---manifest-path"><a class="option-anchor" href="#option-cargo-metadata---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---locked"><a class="option-anchor" href="#option-cargo-metadata---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言必须使用与现有 <code>Cargo.lock</code> 初次生成时完全一致的依赖与版本。
出现以下任一情况时 Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>因依赖解析不同，Cargo 尝试修改锁文件。</li>
</ul>
<p>可用于追求确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---offline"><a class="option-anchor" href="#option-cargo-metadata---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>阻止 Cargo 以任何理由访问网络。未设置该标志时，如果 Cargo 需要访问网络但网络
不可用，会直接报错。设置后，Cargo 会在可能情况下尝试离线继续。</p>
<p>注意，这可能导致与在线模式不同的依赖解析结果。Cargo 会限制为仅使用本地已下载
crate，即便本地索引副本显示有更新版本。
可先用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖，再转入离线。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---frozen"><a class="option-anchor" href="#option-cargo-metadata---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 与 <code>--offline</code>。</p>
</dd>


</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-metadata-+toolchain"><a class="option-anchor" href="#option-cargo-metadata-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>如果 Cargo 通过 rustup 安装，且 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
则会被解释为 rustup 工具链名称（如 <code>+stable</code> 或 <code>+nightly</code>）。
更多覆盖规则见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata---config"><a class="option-anchor" href="#option-cargo-metadata---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数应为 TOML 语法的 <code>KEY=VALUE</code>，
或额外配置文件路径。此标志可多次指定。
更多信息见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata--C"><a class="option-anchor" href="#option-cargo-metadata--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何指定操作前先切换当前工作目录。这会影响 cargo 默认查找项目
manifest（<code>Cargo.toml</code>）的位置，以及发现 <code>.cargo/config.toml</code> 时搜索的目录等。
该选项必须出现在命令名之前，例如 <code>cargo -C path/to/my-project metadata</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly channel</a>
可用，并需要 `-Z unstable-options` 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata--h"><a class="option-anchor" href="#option-cargo-metadata--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-metadata---help"><a class="option-anchor" href="#option-cargo-metadata---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-metadata--Z"><a class="option-anchor" href="#option-cargo-metadata--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>传递给 Cargo 的不稳定（仅 nightly）标志。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 执行成功。
* `101`：Cargo 未能完成。

## 示例

1. 输出当前 package 的 JSON 元数据：

       cargo metadata --format-version=1

## 另请参阅

[cargo(1)](cargo.html), [cargo-pkgid(1)](cargo-pkgid.html), [Package ID Specifications], [JSON messages]

[Package ID Specifications]: ../reference/pkgid-spec.html
[JSON messages]: ../reference/external-tools.html#json-messages
