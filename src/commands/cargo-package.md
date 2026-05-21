# cargo-package(1)
## 名称

cargo-package --- 将本地 package 组装为可分发压缩包

## 概要

`cargo package` [_options_]

## 描述

该命令会把当前目录 package 的源码打包成可分发的压缩 `.crate` 文件。
生成文件会存放在 `target/package` 目录。执行步骤如下：

1. 加载并检查当前 workspace，执行基础校验。
    - 不允许仅有 `path` 且无 `version` 的路径依赖。
      对已发布 package 的依赖，Cargo 会忽略 `path` 键。
      `dev-dependencies` 不受此限制。
2. 创建压缩 `.crate` 文件。
    - 原始 `Cargo.toml` 会被重写并规范化。
    - manifest 中的 `[patch]`、`[replace]`、`[workspace]` 段会被移除。
    - 总是包含 `Cargo.lock`。
      若缺失，会自动生成（除非使用 `--exclude-lockfile`）。
      在使用 `--locked` 时，[cargo-install(1)](cargo-install.html)
      会使用打包内的锁文件。
    - 会包含 `.cargo_vcs_info.json` 文件，
      记录当前 VCS 检出哈希（若可用）以及工作区是否 dirty。
    - 符号链接会被拍平为目标文件。
    - 文件/目录的包含与排除规则遵循
      [manifest 中 `[include]` 与 `[exclude]` 字段](../reference/manifest.html#the-exclude-and-include-fields)。

3. 解压 `.crate` 并构建，验证可构建性。
    - 会从零重建，以确保在干净环境下可构建。
      可使用 `--no-verify` 跳过此步骤。
4. 检查构建脚本未修改任何源码文件。

最终打包文件列表可通过 manifest 中的 `include` 与 `exclude` 字段控制。

打包与发布的更多细节请见[参考文档](../reference/publishing.html)。

### .cargo_vcs_info.json 格式

会生成如下格式的 `.cargo_vcs_info.json`：

```javascript
{
 "git": {
   "sha1": "aac20b6e7e543e6dd4118b246c77225e3a3a1302",
   "dirty": true
 },
 "path_in_vcs": ""
}
```

`dirty` 表示构建 package 时 Git 工作树是否脏。

`path_in_vcs` 对于位于版本库子目录中的 package，
会设置为相对仓库根的路径。

该文件的兼容性策略与 [cargo-metadata(1)](cargo-metadata.html)
的 JSON 输出保持一致。

注意：该文件仅提供“尽力而为”的 VCS 信息快照。
但 package 来源并未被验证，
无法保证 tarball 中源码与 VCS 信息完全一致。

## 选项

### Package 选项

<dl>

<dt class="option-term" id="option-cargo-package--l"><a class="option-anchor" href="#option-cargo-package--l"><code>-l</code></a></dt>
<dt class="option-term" id="option-cargo-package---list"><a class="option-anchor" href="#option-cargo-package---list"><code>--list</code></a></dt>
<dd class="option-desc"><p>仅打印将被打包的文件列表，不实际打包。</p>
</dd>


<dt class="option-term" id="option-cargo-package---no-verify"><a class="option-anchor" href="#option-cargo-package---no-verify"><code>--no-verify</code></a></dt>
<dd class="option-desc"><p>不通过构建来验证内容。</p>
</dd>


<dt class="option-term" id="option-cargo-package---no-metadata"><a class="option-anchor" href="#option-cargo-package---no-metadata"><code>--no-metadata</code></a></dt>
<dd class="option-desc"><p>忽略“缺少面向人类元数据”（如描述或许可证）的警告。</p>
</dd>


<dt class="option-term" id="option-cargo-package---allow-dirty"><a class="option-anchor" href="#option-cargo-package---allow-dirty"><code>--allow-dirty</code></a></dt>
<dd class="option-desc"><p>允许打包包含未提交 VCS 变更的工作目录。</p>
</dd>


<dt class="option-term" id="option-cargo-package---exclude-lockfile"><a class="option-anchor" href="#option-cargo-package---exclude-lockfile"><code>--exclude-lockfile</code></a></dt>
<dd class="option-desc"><p>打包时不包含锁文件。</p>
<p>此参数不建议通用使用。
某些工具可能期望存在锁文件（例如 <code>cargo install --locked</code>）。
使用前请先考虑其他方案。</p>
</dd>


<dt class="option-term" id="option-cargo-package---index"><a class="option-anchor" href="#option-cargo-package---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>要使用的注册表索引 URL。</p>
</dd>


<dt class="option-term" id="option-cargo-package---registry"><a class="option-anchor" href="#option-cargo-package---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要面向其打包的注册表名称；
注册表名配置细节可见 <code>cargo publish --help</code>。
package 不会被发布到该注册表，
但在打包多个相互依赖的 crate 时，
会假定这些依赖将发布到该注册表并据此生成锁文件。</p>
</dd>


<dt class="option-term" id="option-cargo-package---message-format"><a class="option-anchor" href="#option-cargo-package---message-format"><code>--message-format</code> <em>fmt</em></a></dt>
<dd class="option-desc"><p>指定输出消息格式。
目前仅与 <code>--list</code> 配合生效，影响文件列表输出格式。
该能力为不稳定特性，需要 <code>-Zunstable-options</code>。
可用格式：</p>
<ul>
<li><code>human</code>（默认）：每行一个文件。</li>
<li><code>json</code>：输出每个 package 的机器可读 JSON 信息。
每行一个 JSON（Newline delimited JSON）。
<pre><code class="language-javascript">{
  /* package 的 Package ID Spec。 */
  "id": "path+file:///home/foo#0.0.0",
  /* 该 package 的文件 */
  "files" {
    /* 归档文件中的相对路径。 */
    "Cargo.toml.orig": {
      /* 文件来源：
         - "generate" 表示打包过程中生成
         - "copy" 表示从其他位置拷贝
      */
      "kind": "copy",
      /* 对于 "copy"，是实际文件的绝对路径。
         对于 "generate"，是生成文件所基于的原始文件。
      */
      "path": "/home/foo/Cargo.toml"
    },
    "Cargo.toml": {
      "kind": "generate",
      "path": "/home/foo/Cargo.toml"
    },
    "src/main.rs": {
      "kind": "copy",
      "path": "/home/foo/src/main.rs"
    }
  }
}
</code></pre>
</li>
</ul>
</dd>


</dl>

### 包选择

默认情况下（未给出包选择选项时），选中哪些 package 取决于所选 manifest
（若未指定 `--manifest-path`，则基于当前工作目录推断）。
如果 manifest 是 workspace 根，则选择 workspace 的默认成员；
否则仅选择该 manifest 定义的 package。

workspace 默认成员可通过根 manifest 的
`workspace.default-members` 显式设置。
若未设置：
虚拟 workspace 会包含所有成员（等价于 `--workspace`），
非虚拟 workspace 仅包含根 crate 本身。

<dl>

<dt class="option-term" id="option-cargo-package--p"><a class="option-anchor" href="#option-cargo-package--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-package---package"><a class="option-anchor" href="#option-cargo-package---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅打包指定 package。SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。
该参数可重复指定，支持常见 Unix glob（如 <code>*</code>、<code>?</code>、<code>[]</code>）。
为避免 shell 先行展开 glob，必须将每个模式用单引号或双引号包裹。</p>
</dd>


<dt class="option-term" id="option-cargo-package---workspace"><a class="option-anchor" href="#option-cargo-package---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>打包 workspace 的所有成员。</p>
</dd>



<dt class="option-term" id="option-cargo-package---exclude"><a class="option-anchor" href="#option-cargo-package---exclude"><code>--exclude</code> <em>SPEC</em></a></dt>
<dd class="option-desc"><p>排除指定 package。必须与 <code>--workspace</code> 一起使用。
该参数可重复指定，支持常见 Unix glob（如 <code>*</code>、<code>?</code>、<code>[]</code>）。
为避免 shell 先行展开 glob，必须将每个模式用单引号或双引号包裹。</p>
</dd>


</dl>

### 编译选项

<dl>

<dt class="option-term" id="option-cargo-package---target"><a class="option-anchor" href="#option-cargo-package---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>为指定目标架构打包。该参数可重复指定多次。
默认是主机架构。triple 的一般格式为
<code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>。</p>
<p>可选值：</p>
<ul>
<li><code>rustc --print target-list</code> 中列出的任意支持目标。</li>
<li><code>"host-tuple"</code>，它会在内部替换为主机目标。
这在交叉编译部分 crate 时很有用，
尤其是你不想把当前主机目标显式写入 target 列表时
（例如共享项目里会被多种主机执行的 <code>xtask</code>）。</li>
<li>自定义 target 规范文件的路径。更多信息见
<a href="../../rustc/targets/custom.html#custom-target-lookup-path">Custom Target Lookup Path</a>。</li>
</ul>
<p>也可通过 <code>build.target</code> <a href="../reference/config.html">配置值</a> 指定。</p>
<p>注意：指定此参数会让 Cargo 进入另一种模式，
目标产物会放到单独目录。
详情见 <a href="../reference/build-cache.html">build cache</a> 文档。</p>
</dd>


<dt class="option-term" id="option-cargo-package---target-dir"><a class="option-anchor" href="#option-cargo-package---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>所有生成产物与中间文件所在目录。
也可通过环境变量 <code>CARGO_TARGET_DIR</code> 或
<code>build.target-dir</code> <a href="../reference/config.html">配置值</a> 指定。
默认是 workspace 根目录下的 <code>target</code>。</p>
</dd>


</dl>

### Feature 选择

Feature 参数可用于控制启用哪些特性。
当未给出 feature 选项时，所有被选 package 都会启用 `default` feature。

更多细节见
[features 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-package--F"><a class="option-anchor" href="#option-cargo-package--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-package---features"><a class="option-anchor" href="#option-cargo-package---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表（空格或逗号分隔）。
可通过 <code>package-name/feature-name</code> 语法启用 workspace 成员的 feature。
该参数可重复指定，最终会启用所有给出的 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-package---all-features"><a class="option-anchor" href="#option-cargo-package---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-package---no-default-features"><a class="option-anchor" href="#option-cargo-package---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### Manifest 选项

<dl>

<dt class="option-term" id="option-cargo-package---manifest-path"><a class="option-anchor" href="#option-cargo-package---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-package---locked"><a class="option-anchor" href="#option-cargo-package---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-package---offline"><a class="option-anchor" href="#option-cargo-package---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-package---frozen"><a class="option-anchor" href="#option-cargo-package---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>



</dl>

### 其他选项

<dl>
<dt class="option-term" id="option-cargo-package--j"><a class="option-anchor" href="#option-cargo-package--j"><code>-j</code> <em>N</em></a></dt>
<dt class="option-term" id="option-cargo-package---jobs"><a class="option-anchor" href="#option-cargo-package---jobs"><code>--jobs</code> <em>N</em></a></dt>
<dd class="option-desc"><p>并行任务数。也可通过 <code>build.jobs</code> <a href="../reference/config.html">配置值</a> 指定。
默认值为逻辑 CPU 数。
若为负数，则表示“逻辑 CPU 数 + 给定值”。
若为字符串 <code>default</code>，则恢复默认值。
不能为 0。</p>
</dd>

<dt class="option-term" id="option-cargo-package---keep-going"><a class="option-anchor" href="#option-cargo-package---keep-going"><code>--keep-going</code></a></dt>
<dd class="option-desc"><p>尽可能构建依赖图中更多 crate，而不是在第一个失败时立即中止。</p>
<p>例如当前 package 依赖 <code>fails</code> 与 <code>works</code>，其中一个构建失败。
<code>cargo package -j1</code> 可能会也可能不会构建成功的那个
（取决于 Cargo 先选择构建哪一个）；
而 <code>cargo package -j1 --keep-going</code> 一定会尝试两者，
即便先执行的那个失败。</p>
</dd>

</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-package--v"><a class="option-anchor" href="#option-cargo-package--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-package---verbose"><a class="option-anchor" href="#option-cargo-package---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-package--q"><a class="option-anchor" href="#option-cargo-package--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-package---quiet"><a class="option-anchor" href="#option-cargo-package---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-package---color"><a class="option-anchor" href="#option-cargo-package---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。可选值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>

</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-package-+toolchain"><a class="option-anchor" href="#option-cargo-package-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-package---config"><a class="option-anchor" href="#option-cargo-package---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-package--C"><a class="option-anchor" href="#option-cargo-package--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-package--h"><a class="option-anchor" href="#option-cargo-package--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-package---help"><a class="option-anchor" href="#option-cargo-package---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-package--Z"><a class="option-anchor" href="#option-cargo-package--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 为当前 package 创建压缩 `.crate` 文件：

       cargo package

## 另请参见
[cargo(1)](cargo.html), [cargo-publish(1)](cargo-publish.html)
