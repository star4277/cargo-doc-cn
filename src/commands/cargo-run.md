# cargo-run(1)
## 名称

cargo-run --- 运行当前 package

## 概要

`cargo run` [_options_] [`--` _args_]

## 描述

运行本地 package 的某个二进制目标或示例。

两个短横线（`--`）之后的所有参数都会传给要运行的二进制。
如果同时给 Cargo 和二进制传参，则 `--` 后面的给二进制，前面的给 Cargo。

与 [cargo-test(1)](cargo-test.html) 和 [cargo-bench(1)](cargo-bench.html) 不同，
`cargo run` 会把被执行二进制的工作目录设为“当前工作目录”，
与在 shell 中直接执行一致。

## 选项

### 包选择

默认选择当前工作目录中的 package。
可通过 `-p` 在 workspace 中选择其他 package。

<dl>

<dt class="option-term" id="option-cargo-run--p"><a class="option-anchor" href="#option-cargo-run--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-run---package"><a class="option-anchor" href="#option-cargo-run---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>要运行的 package。SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。</p>
</dd>


</dl>

### 目标选择

当未指定目标选择选项时，`cargo run` 会运行二进制目标。
如果有多个二进制目标，必须传目标参数来选择一个。
或者可在 `Cargo.toml` 的 `[package]` 段设置 `default-run`，
作为默认运行的二进制名称。

<dl>

<dt class="option-term" id="option-cargo-run---bin"><a class="option-anchor" href="#option-cargo-run---bin"><code>--bin</code> <em>name</em></a></dt>
<dd class="option-desc"><p>运行指定二进制。</p>
</dd>


<dt class="option-term" id="option-cargo-run---example"><a class="option-anchor" href="#option-cargo-run---example"><code>--example</code> <em>name</em></a></dt>
<dd class="option-desc"><p>运行指定示例。</p>
</dd>


</dl>

### Feature 选择

Feature 参数可用于控制启用哪些特性。
当未给出 feature 选项时，所有被选 package 都会启用 `default` feature。

更多细节见
[features 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-run--F"><a class="option-anchor" href="#option-cargo-run--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-run---features"><a class="option-anchor" href="#option-cargo-run---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表（空格或逗号分隔）。
可通过 <code>package-name/feature-name</code> 语法启用 workspace 成员的 feature。
该参数可重复指定，最终会启用所有给出的 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-run---all-features"><a class="option-anchor" href="#option-cargo-run---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-run---no-default-features"><a class="option-anchor" href="#option-cargo-run---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### 编译选项

<dl>

<dt class="option-term" id="option-cargo-run---target"><a class="option-anchor" href="#option-cargo-run---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>为指定目标架构运行。默认是主机架构。
triple 的一般格式为
<code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>。</p>
<p>可选值：</p>
<ul>
<li><code>rustc --print target-list</code> 中列出的任意支持目标。</li>
<li><code>"host-tuple"</code>，它会在内部替换为主机目标。
这在交叉编译部分 crate 时很有用，
尤其是你不想把当前主机目标显式写入 target 列表时
（例如共享项目里会被多种主机执行的 <code>xtask</code>）。</li>
<li>自定义 target 规范文件的路径。更多信息见
<a href="../../rustc/targets/custom.html#custom-target-lookup-path">Custom Target Lookup Path</a>。</li>
</ul>
<p>也可通过 <code>build.target</code> <a href="../reference/config.html">配置值</a> 指定。</p>
<p>注意：指定此参数会让 Cargo 进入另一种模式，
目标产物会放到单独目录。
详情见 <a href="../reference/build-cache.html">build cache</a> 文档。</p>
</dd>


<dt class="option-term" id="option-cargo-run--r"><a class="option-anchor" href="#option-cargo-run--r"><code>-r</code></a></dt>
<dt class="option-term" id="option-cargo-run---release"><a class="option-anchor" href="#option-cargo-run---release"><code>--release</code></a></dt>
<dd class="option-desc"><p>使用 <code>release</code> profile 运行优化后的产物。
若要按名称选择特定 profile，也可使用 <code>--profile</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-run---profile"><a class="option-anchor" href="#option-cargo-run---profile"><code>--profile</code> <em>name</em></a></dt>
<dd class="option-desc"><p>使用给定 profile 运行。
更多信息见 <a href="../reference/profiles.html">参考文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-run---timings"><a class="option-anchor" href="#option-cargo-run---timings"><code>--timings</code></a></dt>
<dd class="option-desc"><p>输出各编译步骤耗时，以及随时间变化的并发信息。</p>
<p>构建结束后会在 <code>target/cargo-timings</code> 目录写入
<code>cargo-timing.html</code>。
此外还会写入一个文件名带时间戳的报告，便于查看历史运行。
这些报告仅适合人工阅读，不提供机器可读的时序数据。</p>
</dd>



</dl>

### 输出选项

<dl>
<dt class="option-term" id="option-cargo-run---target-dir"><a class="option-anchor" href="#option-cargo-run---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>所有生成产物与中间文件所在目录。
也可通过环境变量 <code>CARGO_TARGET_DIR</code> 或
<code>build.target-dir</code> <a href="../reference/config.html">配置值</a> 指定。
默认是 workspace 根目录下的 <code>target</code>。</p>
</dd>

</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo-run--v"><a class="option-anchor" href="#option-cargo-run--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-run---verbose"><a class="option-anchor" href="#option-cargo-run---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-run--q"><a class="option-anchor" href="#option-cargo-run--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-run---quiet"><a class="option-anchor" href="#option-cargo-run---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-run---color"><a class="option-anchor" href="#option-cargo-run---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。可选值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-run---message-format"><a class="option-anchor" href="#option-cargo-run---message-format"><code>--message-format</code> <em>fmt</em></a></dt>
<dd class="option-desc"><p>诊断信息输出格式。
可多次指定，值为逗号分隔。可选值：</p>
<ul>
<li><code>human</code>（默认）：人类可读文本格式。与 <code>short</code>、<code>json</code> 冲突。</li>
<li><code>short</code>：更短的人类可读文本。与 <code>human</code>、<code>json</code> 冲突。</li>
<li><code>json</code>：向 stdout 输出 JSON 消息。
详见 <a href="../reference/external-tools.html#json-messages">参考文档</a>。
与 <code>human</code>、<code>short</code> 冲突。</li>
<li><code>json-diagnostic-short</code>：确保 JSON 消息中的 <code>rendered</code> 字段
包含 rustc 的“short”渲染结果。
不能与 <code>human</code> 或 <code>short</code> 共用。</li>
<li><code>json-diagnostic-rendered-ansi</code>：确保 JSON 消息中的 <code>rendered</code> 字段
包含 ANSI 颜色码，以遵循 rustc 默认配色。
不能与 <code>human</code> 或 <code>short</code> 共用。</li>
<li><code>json-render-diagnostics</code>：让 Cargo 不再直接包含 rustc 的诊断 JSON，
而由 Cargo 自身渲染 rustc 诊断。Cargo 自身诊断和其它来自 rustc 的消息仍会输出。
不能与 <code>human</code> 或 <code>short</code> 共用。</li>
</ul>
</dd>


</dl>

### Manifest 选项

<dl>

<dt class="option-term" id="option-cargo-run---manifest-path"><a class="option-anchor" href="#option-cargo-run---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-run---ignore-rust-version"><a class="option-anchor" href="#option-cargo-run---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 约束。</p>
</dd>


<dt class="option-term" id="option-cargo-run---locked"><a class="option-anchor" href="#option-cargo-run---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-run---offline"><a class="option-anchor" href="#option-cargo-run---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-run---frozen"><a class="option-anchor" href="#option-cargo-run---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>



</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-run-+toolchain"><a class="option-anchor" href="#option-cargo-run-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-run---config"><a class="option-anchor" href="#option-cargo-run---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-run--C"><a class="option-anchor" href="#option-cargo-run--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-run--h"><a class="option-anchor" href="#option-cargo-run--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-run---help"><a class="option-anchor" href="#option-cargo-run---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-run--Z"><a class="option-anchor" href="#option-cargo-run--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

### 其他选项

<dl>
<dt class="option-term" id="option-cargo-run--j"><a class="option-anchor" href="#option-cargo-run--j"><code>-j</code> <em>N</em></a></dt>
<dt class="option-term" id="option-cargo-run---jobs"><a class="option-anchor" href="#option-cargo-run---jobs"><code>--jobs</code> <em>N</em></a></dt>
<dd class="option-desc"><p>并行任务数。也可通过 <code>build.jobs</code> <a href="../reference/config.html">配置值</a> 指定。
默认值为逻辑 CPU 数。
若为负数，则表示“逻辑 CPU 数 + 给定值”。
若为字符串 <code>default</code>，则恢复默认值。
不能为 0。</p>
</dd>

<dt class="option-term" id="option-cargo-run---keep-going"><a class="option-anchor" href="#option-cargo-run---keep-going"><code>--keep-going</code></a></dt>
<dd class="option-desc"><p>尽可能构建依赖图中更多 crate，而不是在第一个失败时立即中止。</p>
<p>例如当前 package 依赖 <code>fails</code> 与 <code>works</code>，其中一个构建失败。
<code>cargo run -j1</code> 可能会也可能不会构建成功的那个
（取决于 Cargo 先选择构建哪一个）；
而 <code>cargo run -j1 --keep-going</code> 一定会尝试两者，
即便先执行的那个失败。</p>
</dd>

</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 构建本地 package 并运行其主目标（假设仅有一个二进制）：

       cargo run

2. 运行示例并传递额外参数：

       cargo run --example exname -- --exoption exarg1 exarg2

## 另请参见
[cargo(1)](cargo.html), [cargo-build(1)](cargo-build.html)
