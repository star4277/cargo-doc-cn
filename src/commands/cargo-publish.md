# cargo-publish(1)
## 名称

cargo-publish --- 将 package 上传到注册表

## 概要

`cargo publish` [_options_]

## 描述

该命令会为当前目录中的 package 创建可分发的压缩 `.crate` 文件，
并上传到注册表。默认注册表是 <https://crates.io>。
执行步骤如下：

1. 先执行若干检查，包括：
   - 检查 manifest 中 `package.publish` 键对可发布注册表的限制。
2. 按 [cargo-package(1)](cargo-package.html) 的流程生成 `.crate` 文件。
3. 把 crate 上传到注册表，服务器会执行额外检查。
4. 客户端会轮询等待 package 出现在索引中，可能超时。
   若超时，你需要手动确认是否完成。超时不影响上传本身。

该命令要求你先完成认证：
可使用 [cargo-login(1)](cargo-login.html)，
或使用配置字段 [`registry.token`](../reference/config.html#registrytoken)
与 [`registries.<name>.token`](../reference/config.html#registriesnametoken)
对应的环境变量。

关于打包与发布的更多细节见[参考文档](../reference/publishing.html)。

## 选项

### Publish 选项

<dl>

<dt class="option-term" id="option-cargo-publish---dry-run"><a class="option-anchor" href="#option-cargo-publish---dry-run"><code>--dry-run</code></a></dt>
<dd class="option-desc"><p>执行所有检查，但不上传。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---no-verify"><a class="option-anchor" href="#option-cargo-publish---no-verify"><code>--no-verify</code></a></dt>
<dd class="option-desc"><p>不通过构建来验证内容。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---allow-dirty"><a class="option-anchor" href="#option-cargo-publish---allow-dirty"><code>--allow-dirty</code></a></dt>
<dd class="option-desc"><p>允许打包包含未提交 VCS 变更的工作目录。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---index"><a class="option-anchor" href="#option-cargo-publish---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>要使用的注册表索引 URL。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---registry"><a class="option-anchor" href="#option-cargo-publish---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要发布到的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo
配置文件</a>中。若未指定，且
<code>Cargo.toml</code> 的
<a href="../reference/manifest.html#the-publish-field"><code>package.publish</code></a>
仅包含一个注册表，则发布到该注册表。
否则使用默认注册表，默认值由
<a href="../reference/config.html#registrydefault"><code>registry.default</code></a>
配置键定义，默认为 <code>crates-io</code>。</p>
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

<dt class="option-term" id="option-cargo-publish--p"><a class="option-anchor" href="#option-cargo-publish--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-publish---package"><a class="option-anchor" href="#option-cargo-publish---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅发布指定 package。SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。
该参数可重复指定，支持常见 Unix glob（如 <code>*</code>、<code>?</code>、<code>[]</code>）。
为避免 shell 先行展开 glob，必须将每个模式用单引号或双引号包裹。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---workspace"><a class="option-anchor" href="#option-cargo-publish---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>发布 workspace 的所有成员。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---all"><a class="option-anchor" href="#option-cargo-publish---all"><code>--all</code></a></dt>
<dd class="option-desc"><p><code>--workspace</code> 的已弃用别名。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---exclude"><a class="option-anchor" href="#option-cargo-publish---exclude"><code>--exclude</code> <em>SPEC</em></a></dt>
<dd class="option-desc"><p>排除指定 package。必须与 <code>--workspace</code> 一起使用。
该参数可重复指定，支持常见 Unix glob（如 <code>*</code>、<code>?</code>、<code>[]</code>）。
为避免 shell 先行展开 glob，必须将每个模式用单引号或双引号包裹。</p>
</dd>


</dl>

### 编译选项

<dl>

<dt class="option-term" id="option-cargo-publish---target"><a class="option-anchor" href="#option-cargo-publish---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>为指定目标架构发布。该参数可重复指定多次。
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


<dt class="option-term" id="option-cargo-publish---target-dir"><a class="option-anchor" href="#option-cargo-publish---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
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

<dt class="option-term" id="option-cargo-publish--F"><a class="option-anchor" href="#option-cargo-publish--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-publish---features"><a class="option-anchor" href="#option-cargo-publish---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表（空格或逗号分隔）。
可通过 <code>package-name/feature-name</code> 语法启用 workspace 成员的 feature。
该参数可重复指定，最终会启用所有给出的 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---all-features"><a class="option-anchor" href="#option-cargo-publish---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---no-default-features"><a class="option-anchor" href="#option-cargo-publish---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### Manifest 选项

<dl>

<dt class="option-term" id="option-cargo-publish---manifest-path"><a class="option-anchor" href="#option-cargo-publish---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---locked"><a class="option-anchor" href="#option-cargo-publish---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---offline"><a class="option-anchor" href="#option-cargo-publish---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---frozen"><a class="option-anchor" href="#option-cargo-publish---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>



</dl>

### 其他选项

<dl>
<dt class="option-term" id="option-cargo-publish--j"><a class="option-anchor" href="#option-cargo-publish--j"><code>-j</code> <em>N</em></a></dt>
<dt class="option-term" id="option-cargo-publish---jobs"><a class="option-anchor" href="#option-cargo-publish---jobs"><code>--jobs</code> <em>N</em></a></dt>
<dd class="option-desc"><p>并行任务数。也可通过 <code>build.jobs</code> <a href="../reference/config.html">配置值</a> 指定。
默认值为逻辑 CPU 数。
若为负数，则表示“逻辑 CPU 数 + 给定值”。
若为字符串 <code>default</code>，则恢复默认值。
不能为 0。</p>
</dd>

<dt class="option-term" id="option-cargo-publish---keep-going"><a class="option-anchor" href="#option-cargo-publish---keep-going"><code>--keep-going</code></a></dt>
<dd class="option-desc"><p>尽可能构建依赖图中更多 crate，而不是在第一个失败时立即中止。</p>
<p>例如当前 package 依赖 <code>fails</code> 与 <code>works</code>，其中一个构建失败。
<code>cargo publish -j1</code> 可能会也可能不会构建成功的那个
（取决于 Cargo 先选择构建哪一个）；
而 <code>cargo publish -j1 --keep-going</code> 一定会尝试两者，
即便先执行的那个失败。</p>
</dd>

</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-publish--v"><a class="option-anchor" href="#option-cargo-publish--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-publish---verbose"><a class="option-anchor" href="#option-cargo-publish---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-publish--q"><a class="option-anchor" href="#option-cargo-publish--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-publish---quiet"><a class="option-anchor" href="#option-cargo-publish---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---color"><a class="option-anchor" href="#option-cargo-publish---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-publish-+toolchain"><a class="option-anchor" href="#option-cargo-publish-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-publish---config"><a class="option-anchor" href="#option-cargo-publish---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-publish--C"><a class="option-anchor" href="#option-cargo-publish--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-publish--h"><a class="option-anchor" href="#option-cargo-publish--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-publish---help"><a class="option-anchor" href="#option-cargo-publish---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-publish--Z"><a class="option-anchor" href="#option-cargo-publish--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 发布当前 package：

       cargo publish

## 另请参见
[cargo(1)](cargo.html), [cargo-package(1)](cargo-package.html), [cargo-login(1)](cargo-login.html)
