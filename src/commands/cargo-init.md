# cargo-init(1)

## 名称

cargo-init --- 在现有目录中创建新的 Cargo package

## 概要

`cargo init` [_options_] [_path_]

## 描述

该命令会在当前目录创建新的 Cargo manifest。
也可以通过参数传入路径，在指定目录创建。

如果目录中已存在符合常见命名的 Rust 源文件，将直接使用这些文件。
若不存在，则会创建示例 `src/main.rs`；若传入 `--lib`，则创建 `src/lib.rs`。

如果该目录当前不在 VCS 仓库中，则会创建新仓库（见下方 `--vcs`）。

与之类似的命令是 [cargo-new(1)](cargo-new.html)，
它会在新目录中创建新的 package。

## 选项

### Init 选项

<dl>

<dt class="option-term" id="option-cargo-init---bin"><a class="option-anchor" href="#option-cargo-init---bin"><code>--bin</code></a></dt>
<dd class="option-desc"><p>创建带二进制 target（<code>src/main.rs</code>）的 package。
这是默认行为。</p>
</dd>


<dt class="option-term" id="option-cargo-init---lib"><a class="option-anchor" href="#option-cargo-init---lib"><code>--lib</code></a></dt>
<dd class="option-desc"><p>创建带库 target（<code>src/lib.rs</code>）的 package。</p>
</dd>


<dt class="option-term" id="option-cargo-init---edition"><a class="option-anchor" href="#option-cargo-init---edition"><code>--edition</code> <em>edition</em></a></dt>
<dd class="option-desc"><p>指定要使用的 Rust edition。默认是 2024。
可选值：2015、2018、2021、2024。</p>
</dd>


<dt class="option-term" id="option-cargo-init---name"><a class="option-anchor" href="#option-cargo-init---name"><code>--name</code> <em>name</em></a></dt>
<dd class="option-desc"><p>设置 package 名称。默认使用目录名。</p>
</dd>


<dt class="option-term" id="option-cargo-init---vcs"><a class="option-anchor" href="#option-cargo-init---vcs"><code>--vcs</code> <em>vcs</em></a></dt>
<dd class="option-desc"><p>为给定版本控制系统初始化新仓库（git、hg、pijul 或 fossil），
或不初始化版本控制（none）。
若未指定，默认使用 <code>git</code>，或配置项 <code>cargo-new.vcs</code>，
如果当前目录已在 VCS 仓库中则默认为 <code>none</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-init---registry"><a class="option-anchor" href="#option-cargo-init---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>将 <code>Cargo.toml</code> 中的 <code>publish</code> 字段设置为给定注册表名，
从而限制只能发布到该注册表。</p>
<p>注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用 <code>registry.default</code> 配置键定义的默认注册表。
若默认注册表未设置且未使用 <code>--registry</code>，则不会设置 <code>publish</code> 字段，
也就不会限制发布目标。</p>
</dd>


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-init--v"><a class="option-anchor" href="#option-cargo-init--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-init---verbose"><a class="option-anchor" href="#option-cargo-init---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-init--q"><a class="option-anchor" href="#option-cargo-init--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-init---quiet"><a class="option-anchor" href="#option-cargo-init---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-init---color"><a class="option-anchor" href="#option-cargo-init---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-init-+toolchain"><a class="option-anchor" href="#option-cargo-init-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-init---config"><a class="option-anchor" href="#option-cargo-init---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-init--C"><a class="option-anchor" href="#option-cargo-init--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-init--h"><a class="option-anchor" href="#option-cargo-init--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-init---help"><a class="option-anchor" href="#option-cargo-init---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-init--Z"><a class="option-anchor" href="#option-cargo-init--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 在当前目录创建二进制 Cargo package：

       cargo init

## 另请参见
[cargo(1)](cargo.html), [cargo-new(1)](cargo-new.html)
