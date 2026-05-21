# cargo-locate-project(1)

## 名称

cargo-locate-project --- 以 JSON 形式输出 Cargo.toml 文件位置

## 概要

`cargo locate-project` [_options_]

## 描述

该命令会向 stdout 打印一个 JSON 对象，其中包含 manifest 的完整路径。
manifest 的定位方式是：从当前工作目录开始向上搜索名为 `Cargo.toml` 的文件。

如果当前项目属于某个 workspace，默认输出的是“当前项目的 manifest”，
而不是 workspace 根的 manifest。可通过 `--workspace` 覆盖该行为。
workspace 根可以在向上继续遍历中找到，
也可在定位到某个 workspace 成员的 manifest 后，
通过 `package.workspace` 字段确定。

## 选项

<dl>

<dt class="option-term" id="option-cargo-locate-project---workspace"><a class="option-anchor" href="#option-cargo-locate-project---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>定位 workspace 根的 <code>Cargo.toml</code>，而不是当前 workspace 成员。</p>
</dd>


</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo-locate-project---message-format"><a class="option-anchor" href="#option-cargo-locate-project---message-format"><code>--message-format</code> <em>fmt</em></a></dt>
<dd class="option-desc"><p>输出项目位置的表示格式。可选值：</p>
<ul>
<li><code>json</code>（默认）：JSON 对象，路径放在键 <code>root</code> 下。</li>
<li><code>plain</code>：仅输出路径。</li>
</ul>
</dd>


<dt class="option-term" id="option-cargo-locate-project--v"><a class="option-anchor" href="#option-cargo-locate-project--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-locate-project---verbose"><a class="option-anchor" href="#option-cargo-locate-project---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-locate-project--q"><a class="option-anchor" href="#option-cargo-locate-project--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-locate-project---quiet"><a class="option-anchor" href="#option-cargo-locate-project---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-locate-project---color"><a class="option-anchor" href="#option-cargo-locate-project---color"><code>--color</code> <em>when</em></a></dt>
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
<dt class="option-term" id="option-cargo-locate-project---manifest-path"><a class="option-anchor" href="#option-cargo-locate-project---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>

</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-locate-project-+toolchain"><a class="option-anchor" href="#option-cargo-locate-project-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-locate-project---config"><a class="option-anchor" href="#option-cargo-locate-project---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-locate-project--C"><a class="option-anchor" href="#option-cargo-locate-project--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-locate-project--h"><a class="option-anchor" href="#option-cargo-locate-project--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-locate-project---help"><a class="option-anchor" href="#option-cargo-locate-project---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-locate-project--Z"><a class="option-anchor" href="#option-cargo-locate-project--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 基于当前目录显示 manifest 路径：

       cargo locate-project

## 另请参见
[cargo(1)](cargo.html), [cargo-metadata(1)](cargo-metadata.html)
