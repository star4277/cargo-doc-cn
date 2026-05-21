# cargo-info(1)

## 名称

cargo-info --- 显示 package 信息。

## 概要

`cargo info` [_options_] _spec_

## 描述

该命令用于显示 package 信息。它会获取 package 的 `Cargo.toml` 数据，
并以人类可读的格式展示。

## 选项

### Info 选项

<dl>

<dt class="option-term" id="option-cargo-info-spec"><a class="option-anchor" href="#option-cargo-info-spec"><em>spec</em></a></dt>
<dd class="option-desc"><p>获取指定 package 的信息。<em>spec</em> 可以是 Package ID，
SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。
若指定 package 属于当前 workspace，则会显示本地 Cargo.toml 的信息。
如果 <code>Cargo.lock</code> 不存在，会自动创建。
如果未指定版本，会基于最低支持 Rust 版本（MSRV）选择合适版本。</p>
</dd>

<dt class="option-term" id="option-cargo-info---index"><a class="option-anchor" href="#option-cargo-info---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>要使用的注册表索引 URL。</p>
</dd>

<dt class="option-term" id="option-cargo-info---registry"><a class="option-anchor" href="#option-cargo-info---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要使用的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用默认注册表，该默认值由 <code>registry.default</code> 配置键决定，
其默认值是 <code>crates-io</code>。</p>
</dd>

</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-info--v"><a class="option-anchor" href="#option-cargo-info--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-info---verbose"><a class="option-anchor" href="#option-cargo-info---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-info--q"><a class="option-anchor" href="#option-cargo-info--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-info---quiet"><a class="option-anchor" href="#option-cargo-info---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-info---color"><a class="option-anchor" href="#option-cargo-info---color"><code>--color</code> <em>when</em></a></dt>
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
<dt class="option-term" id="option-cargo-info---locked"><a class="option-anchor" href="#option-cargo-info---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-info---offline"><a class="option-anchor" href="#option-cargo-info---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-info---frozen"><a class="option-anchor" href="#option-cargo-info---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>

</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-info-+toolchain"><a class="option-anchor" href="#option-cargo-info-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-info---config"><a class="option-anchor" href="#option-cargo-info---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-info--C"><a class="option-anchor" href="#option-cargo-info--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-info--h"><a class="option-anchor" href="#option-cargo-info--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-info---help"><a class="option-anchor" href="#option-cargo-info---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-info--Z"><a class="option-anchor" href="#option-cargo-info--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 查看 crates.io 上的 `serde` package：

        cargo info serde
2. 查看版本为 `1.0.0` 的 `serde` package：

        cargo info serde@1.0.0
3. 查看本地注册表中的 `serde` package：

        cargo info serde --registry my-registry

## 另请参见

[cargo(1)](cargo.html), [cargo-search(1)](cargo-search.html)
