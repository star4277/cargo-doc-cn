# cargo-uninstall(1)

## 名称

cargo-uninstall --- 删除 Rust 二进制程序

## 概要

`cargo uninstall` [_options_] [_spec_...]

## 描述

该命令会移除通过 [cargo-install(1)](cargo-install.html) 安装的 package。
参数 _spec_ 是要移除 package 的 Package ID 规范（见
[cargo-pkgid(1)](cargo-pkgid.html)）。

默认会移除该 crate 的全部二进制文件，
但可以使用 `--bin` 和 `--example` 仅移除指定二进制。

安装根目录按以下优先级决定：

- `--root` 选项
- `CARGO_INSTALL_ROOT` 环境变量
- `install.root` Cargo [配置值](../reference/config.html)
- `CARGO_HOME` 环境变量
- `$HOME/.cargo`

## 选项

### 卸载选项

<dl>

<dt class="option-term" id="option-cargo-uninstall--p"><a class="option-anchor" href="#option-cargo-uninstall--p"><code>-p</code></a></dt>
<dt class="option-term" id="option-cargo-uninstall---package"><a class="option-anchor" href="#option-cargo-uninstall---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>要卸载的 package。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall---bin"><a class="option-anchor" href="#option-cargo-uninstall---bin"><code>--bin</code> <em>name</em></a></dt>
<dd class="option-desc"><p>仅卸载名为 <em>name</em> 的二进制。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall---root"><a class="option-anchor" href="#option-cargo-uninstall---root"><code>--root</code> <em>dir</em></a></dt>
<dd class="option-desc"><p>执行卸载的目录。</p>
</dd>


</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo-uninstall--v"><a class="option-anchor" href="#option-cargo-uninstall--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-uninstall---verbose"><a class="option-anchor" href="#option-cargo-uninstall---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall--q"><a class="option-anchor" href="#option-cargo-uninstall--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-uninstall---quiet"><a class="option-anchor" href="#option-cargo-uninstall---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall---color"><a class="option-anchor" href="#option-cargo-uninstall---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-uninstall-+toolchain"><a class="option-anchor" href="#option-cargo-uninstall-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall---config"><a class="option-anchor" href="#option-cargo-uninstall---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall--C"><a class="option-anchor" href="#option-cargo-uninstall--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall--h"><a class="option-anchor" href="#option-cargo-uninstall--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-uninstall---help"><a class="option-anchor" href="#option-cargo-uninstall---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-uninstall--Z"><a class="option-anchor" href="#option-cargo-uninstall--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 卸载一个先前安装的 package：

       cargo uninstall ripgrep

## 另请参见
[cargo(1)](cargo.html), [cargo-install(1)](cargo-install.html)
