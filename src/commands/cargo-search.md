# cargo-search(1)

## 名称

cargo-search --- 在注册表中搜索 package。默认注册表是 crates.io

## 概要

`cargo search` [_options_] [_query_...]

## 描述

在 <https://crates.io> 上执行文本搜索。
匹配的 crate 会连同其描述一起展示，输出格式为适合复制到
`Cargo.toml` manifest 的 TOML 片段。

## 选项

### 搜索选项

<dl>

<dt class="option-term" id="option-cargo-search---limit"><a class="option-anchor" href="#option-cargo-search---limit"><code>--limit</code> <em>limit</em></a></dt>
<dd class="option-desc"><p>限制结果数量（默认：10，最大：100）。</p>
</dd>


<dt class="option-term" id="option-cargo-search---index"><a class="option-anchor" href="#option-cargo-search---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>使用的注册表索引 URL。</p>
</dd>


<dt class="option-term" id="option-cargo-search---registry"><a class="option-anchor" href="#option-cargo-search---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要使用的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用默认注册表，该默认值由 <code>registry.default</code> 配置键决定，
其默认值是 <code>crates-io</code>。</p>
</dd>


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-search--v"><a class="option-anchor" href="#option-cargo-search--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-search---verbose"><a class="option-anchor" href="#option-cargo-search---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-search--q"><a class="option-anchor" href="#option-cargo-search--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-search---quiet"><a class="option-anchor" href="#option-cargo-search---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-search---color"><a class="option-anchor" href="#option-cargo-search---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。可选值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>

</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-search-+toolchain"><a class="option-anchor" href="#option-cargo-search-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-search---config"><a class="option-anchor" href="#option-cargo-search---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-search--C"><a class="option-anchor" href="#option-cargo-search--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-search--h"><a class="option-anchor" href="#option-cargo-search--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-search---help"><a class="option-anchor" href="#option-cargo-search---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-search--Z"><a class="option-anchor" href="#option-cargo-search--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 从 crates.io 搜索 package：

       cargo search serde

## 另请参见

[cargo(1)](cargo.html), [cargo-install(1)](cargo-install.html), [cargo-publish(1)](cargo-publish.html)
