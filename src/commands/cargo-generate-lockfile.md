# cargo-generate-lockfile(1)

## 名称

cargo-generate-lockfile --- 为 package 生成锁文件

## 概要

`cargo generate-lockfile` [_options_]

## 描述

该命令会为当前 package 或 workspace 创建 `Cargo.lock` 锁文件。
如果锁文件已存在，会按每个 package 的最新可用版本重新生成。

另见 [cargo-update(1)](cargo-update.html)，
它同样可以创建 `Cargo.lock`，并提供更多控制更新行为的选项。

## 选项

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-generate-lockfile--v"><a class="option-anchor" href="#option-cargo-generate-lockfile--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-generate-lockfile---verbose"><a class="option-anchor" href="#option-cargo-generate-lockfile---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile--q"><a class="option-anchor" href="#option-cargo-generate-lockfile--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-generate-lockfile---quiet"><a class="option-anchor" href="#option-cargo-generate-lockfile---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---color"><a class="option-anchor" href="#option-cargo-generate-lockfile---color"><code>--color</code> <em>when</em></a></dt>
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
<dt class="option-term" id="option-cargo-generate-lockfile---manifest-path"><a class="option-anchor" href="#option-cargo-generate-lockfile---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---ignore-rust-version"><a class="option-anchor" href="#option-cargo-generate-lockfile---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 约束。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---publish-time"><a class="option-anchor" href="#option-cargo-generate-lockfile---publish-time"><code>--publish-time</code> <em>yyyy-mm-ddThh:mm:ssZ</em></a></dt>
<dd class="option-desc"><p>注册表 package 允许的最晚发布时间（不稳定）。</p>
<p>这是对允许 package 的“尽力过滤”，包括：</p>
<ul>
<li>来自不受支持注册表的 package 一律接受</li>
<li>只考虑当前 yank 状态，不考虑 <code>--publish-time</code> 时刻的历史状态</li>
<li>发布时间精度限制</li>
</ul>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---locked"><a class="option-anchor" href="#option-cargo-generate-lockfile---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---offline"><a class="option-anchor" href="#option-cargo-generate-lockfile---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---frozen"><a class="option-anchor" href="#option-cargo-generate-lockfile---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>


</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-generate-lockfile-+toolchain"><a class="option-anchor" href="#option-cargo-generate-lockfile-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile---config"><a class="option-anchor" href="#option-cargo-generate-lockfile---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile--C"><a class="option-anchor" href="#option-cargo-generate-lockfile--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile--h"><a class="option-anchor" href="#option-cargo-generate-lockfile--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-generate-lockfile---help"><a class="option-anchor" href="#option-cargo-generate-lockfile---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-generate-lockfile--Z"><a class="option-anchor" href="#option-cargo-generate-lockfile--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 为当前 package 或 workspace 创建或更新锁文件：

       cargo generate-lockfile

## 另请参见
[cargo(1)](cargo.html), [cargo-update(1)](cargo-update.html)
