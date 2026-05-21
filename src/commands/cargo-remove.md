# cargo-remove(1)
## 名称

cargo-remove --- 从 Cargo.toml manifest 中移除依赖

## 概要

`cargo remove` [_options_] _dependency_...

## 描述

从 `Cargo.toml` manifest 中移除一个或多个依赖。

## 选项

### 分区选项

<dl>

<dt class="option-term" id="option-cargo-remove---dev"><a class="option-anchor" href="#option-cargo-remove---dev"><code>--dev</code></a></dt>
<dd class="option-desc"><p>作为<a href="../reference/specifying-dependencies.html#development-dependencies">开发依赖</a>移除。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---build"><a class="option-anchor" href="#option-cargo-remove---build"><code>--build</code></a></dt>
<dd class="option-desc"><p>作为<a href="../reference/specifying-dependencies.html#build-dependencies">构建依赖</a>移除。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---target"><a class="option-anchor" href="#option-cargo-remove---target"><code>--target</code> <em>target</em></a></dt>
<dd class="option-desc"><p>作为<a href="../reference/specifying-dependencies.html#platform-specific-dependencies">给定目标平台</a>的依赖移除。</p>
<p>为避免 shell 意外展开，建议对目标加引号，例如 <code>--target 'cfg(unix)'</code>。</p>
</dd>


</dl>

### 杂项选项

<dl>

<dt class="option-term" id="option-cargo-remove---dry-run"><a class="option-anchor" href="#option-cargo-remove---dry-run"><code>--dry-run</code></a></dt>
<dd class="option-desc"><p>仅演示变更，不真正写入 manifest。</p>
</dd>


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-remove--v"><a class="option-anchor" href="#option-cargo-remove--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-remove---verbose"><a class="option-anchor" href="#option-cargo-remove---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-remove--q"><a class="option-anchor" href="#option-cargo-remove--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-remove---quiet"><a class="option-anchor" href="#option-cargo-remove---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---color"><a class="option-anchor" href="#option-cargo-remove---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。可选值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>

</dl>

### Manifest 选项

<dl>
<dt class="option-term" id="option-cargo-remove---manifest-path"><a class="option-anchor" href="#option-cargo-remove---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---locked"><a class="option-anchor" href="#option-cargo-remove---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---offline"><a class="option-anchor" href="#option-cargo-remove---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---frozen"><a class="option-anchor" href="#option-cargo-remove---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>


</dl>

### 包选择

<dl>

<dt class="option-term" id="option-cargo-remove--p"><a class="option-anchor" href="#option-cargo-remove--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-remove---package"><a class="option-anchor" href="#option-cargo-remove---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>要移除依赖的 package。</p>
</dd>


</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-remove-+toolchain"><a class="option-anchor" href="#option-cargo-remove-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-remove---config"><a class="option-anchor" href="#option-cargo-remove---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-remove--C"><a class="option-anchor" href="#option-cargo-remove--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-remove--h"><a class="option-anchor" href="#option-cargo-remove--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-remove---help"><a class="option-anchor" href="#option-cargo-remove---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-remove--Z"><a class="option-anchor" href="#option-cargo-remove--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 移除 `regex` 依赖

       cargo remove regex

2. 移除 `trybuild` 开发依赖

       cargo remove --dev trybuild

3. 从 `wasm32-unknown-unknown` 依赖表中移除 `nom`

       cargo remove --target wasm32-unknown-unknown nom

## 另请参见
[cargo(1)](cargo.html), [cargo-add(1)](cargo-add.html)
