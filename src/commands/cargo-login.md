# cargo-login(1)

## 名称

cargo-login --- 登录到注册表

## 概要

`cargo login` [_options_] [`--` _args_]

## 描述

该命令会运行凭据提供器并保存 token，
使需要认证的命令（例如 [cargo-publish(1)](cargo-publish.html)）
可以自动完成认证。

两个短横线（`--`）之后的所有参数都会传递给凭据提供器。

对于默认的 `cargo:token` 凭据提供器，token 会保存到
`$CARGO_HOME/credentials.toml`。
`CARGO_HOME` 默认是用户主目录下的 `.cargo`。

如果某个注册表指定了 credential-provider，则优先使用它。
否则会尝试配置项 `registry.global-credential-providers` 中的提供器，
并从列表末尾开始尝试。

_token_ 会从 stdin 读取。

crates.io 的 API token 可在 <https://crates.io/me> 获取。

请妥善保管 token，不要与他人共享。

## 选项

### 登录选项

<dl>
<dt class="option-term" id="option-cargo-login---registry"><a class="option-anchor" href="#option-cargo-login---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要使用的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用默认注册表，该默认值由 <code>registry.default</code> 配置键决定，
其默认值是 <code>crates-io</code>。</p>
</dd>

</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-login--v"><a class="option-anchor" href="#option-cargo-login--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-login---verbose"><a class="option-anchor" href="#option-cargo-login---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-login--q"><a class="option-anchor" href="#option-cargo-login--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-login---quiet"><a class="option-anchor" href="#option-cargo-login---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-login---color"><a class="option-anchor" href="#option-cargo-login---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-login-+toolchain"><a class="option-anchor" href="#option-cargo-login-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-login---config"><a class="option-anchor" href="#option-cargo-login---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-login--C"><a class="option-anchor" href="#option-cargo-login--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-login--h"><a class="option-anchor" href="#option-cargo-login--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-login---help"><a class="option-anchor" href="#option-cargo-login---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-login--Z"><a class="option-anchor" href="#option-cargo-login--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 为默认注册表保存 token：

       cargo login

2. 为指定注册表保存 token：

       cargo login --registry my-registry

## 另请参见
[cargo(1)](cargo.html), [cargo-logout(1)](cargo-logout.html), [cargo-publish(1)](cargo-publish.html)
