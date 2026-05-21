# cargo-clean(1)
## 名称

cargo-clean --- 删除生成产物

## 概要

`cargo clean` [_options_]

## 描述

删除 Cargo 过去在 target 目录中生成的产物。

不带选项时，`cargo clean` 会删除整个 target 目录。

## 选项

### 包选择

当未选择任何 package 时，会清理 workspace 中所有 package 及其全部依赖。

<dl>

<dt class="option-term" id="option-cargo-clean--p"><a class="option-anchor" href="#option-cargo-clean--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-clean---package"><a class="option-anchor" href="#option-cargo-clean---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅清理指定 package。该参数可重复指定多次。
SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---workspace"><a class="option-anchor" href="#option-cargo-clean---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>清理 workspace 成员的产物。</p>
</dd>


</dl>

### Clean 选项

<dl>

<dt class="option-term" id="option-cargo-clean---dry-run"><a class="option-anchor" href="#option-cargo-clean---dry-run"><code>--dry-run</code></a></dt>
<dd class="option-desc"><p>显示将要删除内容的摘要，但不真正删除。
配合 <code>--verbose</code> 可显示将被删除的实际文件。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---doc"><a class="option-anchor" href="#option-cargo-clean---doc"><code>--doc</code></a></dt>
<dd class="option-desc"><p>仅删除 target 目录中的 <code>doc</code> 子目录。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---release"><a class="option-anchor" href="#option-cargo-clean---release"><code>--release</code></a></dt>
<dd class="option-desc"><p>删除 <code>release</code> 目录中的所有产物。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---profile"><a class="option-anchor" href="#option-cargo-clean---profile"><code>--profile</code> <em>name</em></a></dt>
<dd class="option-desc"><p>删除给定 profile 名称目录中的所有产物。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---target-dir"><a class="option-anchor" href="#option-cargo-clean---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>所有生成产物与中间文件所在目录。
也可通过环境变量 <code>CARGO_TARGET_DIR</code> 或
<code>build.target-dir</code> <a href="../reference/config.html">配置值</a> 指定。
默认是 workspace 根目录下的 <code>target</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---target"><a class="option-anchor" href="#option-cargo-clean---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>针对指定目标架构执行清理。该参数可重复指定多次。
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


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-clean--v"><a class="option-anchor" href="#option-cargo-clean--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-clean---verbose"><a class="option-anchor" href="#option-cargo-clean---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-clean--q"><a class="option-anchor" href="#option-cargo-clean--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-clean---quiet"><a class="option-anchor" href="#option-cargo-clean---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---color"><a class="option-anchor" href="#option-cargo-clean---color"><code>--color</code> <em>when</em></a></dt>
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

### Manifest 选项

<dl>
<dt class="option-term" id="option-cargo-clean---manifest-path"><a class="option-anchor" href="#option-cargo-clean---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---locked"><a class="option-anchor" href="#option-cargo-clean---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---offline"><a class="option-anchor" href="#option-cargo-clean---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---frozen"><a class="option-anchor" href="#option-cargo-clean---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>


</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-clean-+toolchain"><a class="option-anchor" href="#option-cargo-clean-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-clean---config"><a class="option-anchor" href="#option-cargo-clean---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-clean--C"><a class="option-anchor" href="#option-cargo-clean--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-clean--h"><a class="option-anchor" href="#option-cargo-clean--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-clean---help"><a class="option-anchor" href="#option-cargo-clean---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-clean--Z"><a class="option-anchor" href="#option-cargo-clean--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 删除整个 target 目录：

       cargo clean

2. 仅删除 release 产物：

       cargo clean --release

## 另请参见
[cargo(1)](cargo.html), [cargo-build(1)](cargo-build.html)
