# cargo-owner(1)

## 名称

cargo-owner --- 管理注册表上 crate 的所有者

## 概要

`cargo owner` [_options_] `--add` _login_ [_crate_]\
`cargo owner` [_options_] `--remove` _login_ [_crate_]\
`cargo owner` [_options_] `--list` [_crate_]

## 描述

该命令用于修改注册表上某个 crate 的所有者。
crate 的所有者可以上传新版本并 yank 旧版本。
非团队所有者也可以修改所有者列表，请谨慎操作。

该命令要求你已认证：
可通过 `--token` 选项，或先执行 [cargo-login(1)](cargo-login.html)。

若未指定 crate 名称，会使用当前目录 package 的名称。

关于所有者与发布的更多信息，见
[参考文档](../reference/publishing.html#cargo-owner)。

## 选项

### Owner 选项

<dl>

<dt class="option-term" id="option-cargo-owner--a"><a class="option-anchor" href="#option-cargo-owner--a"><code>-a</code></a></dt>
<dt class="option-term" id="option-cargo-owner---add"><a class="option-anchor" href="#option-cargo-owner---add"><code>--add</code> <em>login</em></a></dt>
<dd class="option-desc"><p>邀请给定用户或团队成为所有者。</p>
</dd>


<dt class="option-term" id="option-cargo-owner--r"><a class="option-anchor" href="#option-cargo-owner--r"><code>-r</code></a></dt>
<dt class="option-term" id="option-cargo-owner---remove"><a class="option-anchor" href="#option-cargo-owner---remove"><code>--remove</code> <em>login</em></a></dt>
<dd class="option-desc"><p>移除给定用户或团队的所有者权限。</p>
</dd>


<dt class="option-term" id="option-cargo-owner--l"><a class="option-anchor" href="#option-cargo-owner--l"><code>-l</code></a></dt>
<dt class="option-term" id="option-cargo-owner---list"><a class="option-anchor" href="#option-cargo-owner---list"><code>--list</code></a></dt>
<dd class="option-desc"><p>列出 crate 的所有者。</p>
</dd>


<dt class="option-term" id="option-cargo-owner---token"><a class="option-anchor" href="#option-cargo-owner---token"><code>--token</code> <em>token</em></a></dt>
<dd class="option-desc"><p>认证时使用的 API token。该值会覆盖凭据文件中的 token
（凭据文件由 <a href="cargo-login.html">cargo-login(1)</a> 创建）。</p>
<p>也可以使用 <a href="../reference/config.html">Cargo 配置</a> 对应的环境变量
覆盖凭据文件中的 token。crates.io 的 token 可通过
<code>CARGO_REGISTRY_TOKEN</code> 指定。
其他注册表可通过形如 <code>CARGO_REGISTRIES_NAME_TOKEN</code> 的环境变量指定，
其中 <code>NAME</code> 是注册表名的大写形式。</p>
</dd>


<dt class="option-term" id="option-cargo-owner---index"><a class="option-anchor" href="#option-cargo-owner---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>要使用的注册表索引 URL。</p>
</dd>


<dt class="option-term" id="option-cargo-owner---registry"><a class="option-anchor" href="#option-cargo-owner---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要使用的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用默认注册表，该默认值由 <code>registry.default</code> 配置键决定，
其默认值是 <code>crates-io</code>。</p>
</dd>


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-owner--v"><a class="option-anchor" href="#option-cargo-owner--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-owner---verbose"><a class="option-anchor" href="#option-cargo-owner---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-owner--q"><a class="option-anchor" href="#option-cargo-owner--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-owner---quiet"><a class="option-anchor" href="#option-cargo-owner---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-owner---color"><a class="option-anchor" href="#option-cargo-owner---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-owner-+toolchain"><a class="option-anchor" href="#option-cargo-owner-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-owner---config"><a class="option-anchor" href="#option-cargo-owner---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-owner--C"><a class="option-anchor" href="#option-cargo-owner--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-owner--h"><a class="option-anchor" href="#option-cargo-owner--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-owner---help"><a class="option-anchor" href="#option-cargo-owner---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-owner--Z"><a class="option-anchor" href="#option-cargo-owner--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 列出 package 的所有者：

       cargo owner --list foo

2. 向 package 邀请所有者：

       cargo owner --add username foo

3. 从 package 移除所有者：

       cargo owner --remove username foo

## 另请参见
[cargo(1)](cargo.html), [cargo-login(1)](cargo-login.html), [cargo-publish(1)](cargo-publish.html)
