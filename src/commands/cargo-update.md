# cargo-update(1)

## 名称

cargo-update --- 更新本地锁文件中记录的依赖

## 概要

`cargo update` [_options_] _spec_

## 描述

该命令会把 `Cargo.lock` 中的依赖更新到最新版本。
如果 `Cargo.lock` 不存在，会用最新可用版本创建它。

## 选项

### Update 选项

<dl>

<dt class="option-term" id="option-cargo-update-spec"><a class="option-anchor" href="#option-cargo-update-spec"><em>spec</em></a></dt>
<dd class="option-desc"><p>仅更新指定 package。该参数可重复指定多次。
SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。</p>
<p>如果指定了 <em>spec</em>，会对锁文件执行“保守更新”。
即仅更新 SPEC 指向的依赖；
其传递依赖只会在“不更新它们就无法更新 SPEC”时才会被更新。
其他依赖都会保持当前锁定版本。</p>
<p>若未指定 <em>spec</em>，则更新所有依赖。</p>
</dd>


<dt class="option-term" id="option-cargo-update---recursive"><a class="option-anchor" href="#option-cargo-update---recursive"><code>--recursive</code></a></dt>
<dd class="option-desc"><p>与 <em>spec</em> 一起使用时，会强制同时更新 <em>spec</em> 的依赖。
不能与 <code>--precise</code> 同用。</p>
</dd>


<dt class="option-term" id="option-cargo-update---precise"><a class="option-anchor" href="#option-cargo-update---precise"><code>--precise</code> <em>precise</em></a></dt>
<dd class="option-desc"><p>与 <em>spec</em> 一起使用时，可指定要设置到的精确版本号。
如果 package 来自 git 仓库，也可以指定 git 修订（如 SHA 或 tag）。</p>
<p>虽然不推荐，你也可以指定一个已被 yanked 的版本。
在可行情况下，建议优先选择其他未 yanked 且 SemVer 兼容的版本，
或联系该 package 维护者。</p>
<p>即便 <code>Cargo.toml</code> 的版本要求不含 pre-release 标识，
也可以指定兼容的 <code>pre-release</code> 版本（仅 nightly）。</p>
</dd>


<dt class="option-term" id="option-cargo-update---breaking"><a class="option-anchor" href="#option-cargo-update---breaking"><code>--breaking</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>将 <em>spec</em> 更新到最新的 SemVer 不兼容版本。</p>
<p>版本要求将被修改以允许这次更新。</p>
<p>该选项仅适用于满足以下条件的依赖：</p>
<ul>
<li>该 package 是 workspace 成员的依赖</li>
<li>该依赖未被重命名</li>
<li>存在 SemVer 不兼容的新版本</li>
<li>使用了“SemVer 运算符”（默认的 <code>^</code>）</li>
</ul>
<p>该选项为不稳定特性，仅在
<a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>
可用，并且需要 <code>-Z unstable-options</code>。
详见 <a href="https://github.com/rust-lang/cargo/issues/12425">https://github.com/rust-lang/cargo/issues/12425</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-update--w"><a class="option-anchor" href="#option-cargo-update--w"><code>-w</code></a></dt>
<dt class="option-term" id="option-cargo-update---workspace"><a class="option-anchor" href="#option-cargo-update---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>尝试仅更新 workspace 中定义的 package。
其他 package 仅在锁文件中尚不存在时才会更新。
当你修改了 <code>Cargo.toml</code> 中的版本号后，
该选项可用于更新 <code>Cargo.lock</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-update---dry-run"><a class="option-anchor" href="#option-cargo-update---dry-run"><code>--dry-run</code></a></dt>
<dd class="option-desc"><p>显示将要更新的内容，但不真正写入锁文件。</p>
</dd>


</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-update--v"><a class="option-anchor" href="#option-cargo-update--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-update---verbose"><a class="option-anchor" href="#option-cargo-update---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-update--q"><a class="option-anchor" href="#option-cargo-update--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-update---quiet"><a class="option-anchor" href="#option-cargo-update---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-update---color"><a class="option-anchor" href="#option-cargo-update---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-update---manifest-path"><a class="option-anchor" href="#option-cargo-update---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-update---ignore-rust-version"><a class="option-anchor" href="#option-cargo-update---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 约束。</p>
</dd>


<dt class="option-term" id="option-cargo-update---locked"><a class="option-anchor" href="#option-cargo-update---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-update---offline"><a class="option-anchor" href="#option-cargo-update---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-update---frozen"><a class="option-anchor" href="#option-cargo-update---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>



</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-update-+toolchain"><a class="option-anchor" href="#option-cargo-update-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a> 中的工具链覆盖机制。</p>
</dd>


<dt class="option-term" id="option-cargo-update---config"><a class="option-anchor" href="#option-cargo-update---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-update--C"><a class="option-anchor" href="#option-cargo-update--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly 通道</a>可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-update--h"><a class="option-anchor" href="#option-cargo-update--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-update---help"><a class="option-anchor" href="#option-cargo-update---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-update--Z"><a class="option-anchor" href="#option-cargo-update--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。可运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 更新锁文件中的全部依赖：

       cargo update

2. 仅更新指定依赖：

       cargo update foo bar

3. 将指定依赖设置为指定版本：

       cargo update foo --precise 1.2.3

## 另请参见
[cargo(1)](cargo.html), [cargo-generate-lockfile(1)](cargo-generate-lockfile.html)
