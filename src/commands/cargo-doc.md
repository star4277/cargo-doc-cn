# cargo-doc(1)
## 名称

cargo-doc --- 构建 package 文档

## 概要

`cargo doc` [_options_]

## 描述

构建本地 package 及其全部依赖的文档。输出会以 rustdoc 的常规格式放在 `target/doc`。

**注意：** 文档生成是累积的：不同 `cargo doc` 调用之间会保留 target 目录中已有文档文件。
若要删除已生成文档，请对 [cargo-clean(1)](cargo-clean.html) 传入 `--doc`。

## 选项

### 文档选项

<dl>

<dt class="option-term" id="option-cargo-doc---open"><a class="option-anchor" href="#option-cargo-doc---open"><code>--open</code></a></dt>
<dd class="option-desc"><p>构建后在浏览器中打开文档。默认使用系统默认浏览器，
除非在环境变量 <code>BROWSER</code> 中另行指定，或使用
<a href="../reference/config.html#docbrowser"><code>doc.browser</code></a> 配置项。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---no-deps"><a class="option-anchor" href="#option-cargo-doc---no-deps"><code>--no-deps</code></a></dt>
<dd class="option-desc"><p>不构建依赖的文档。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---document-private-items"><a class="option-anchor" href="#option-cargo-doc---document-private-items"><code>--document-private-items</code></a></dt>
<dd class="option-desc"><p>在文档中包含非公开项。若文档目标是二进制 target，将默认启用该选项。</p>
</dd>


</dl>

### Package 选择

默认情况下，如果未提供 package 选择选项，具体选择哪些 package 取决于所选
manifest 文件（若未指定 `--manifest-path`，则基于当前工作目录）。如果该
manifest 是 workspace 根目录，则会选择 workspace 的默认成员；否则仅选择该
manifest 定义的 package。

workspace 的默认成员可以通过根 manifest 中的
`workspace.default-members` 键显式设置。若未设置，虚拟 workspace 会包含全部
workspace 成员（等价于传入 `--workspace`），非虚拟 workspace 则仅包含根 crate 自身。

<dl>

<dt class="option-term" id="option-cargo-doc--p"><a class="option-anchor" href="#option-cargo-doc--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-doc---package"><a class="option-anchor" href="#option-cargo-doc---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅为指定 package 生成文档。SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。
此标志可多次指定，并支持常见 Unix glob 模式，如 <code>*</code>、<code>?</code> 与 <code>[]</code>。
但为了避免 shell 在 Cargo 处理前提前展开 glob，必须将每个模式放在单引号或双引号中。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---workspace"><a class="option-anchor" href="#option-cargo-doc---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>为 workspace 中全部成员生成文档。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---all"><a class="option-anchor" href="#option-cargo-doc---all"><code>--all</code></a></dt>
<dd class="option-desc"><p><code>--workspace</code> 的已弃用别名。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---exclude"><a class="option-anchor" href="#option-cargo-doc---exclude"><code>--exclude</code> <em>SPEC</em></a></dt>
<dd class="option-desc"><p>排除指定 package。必须与 <code>--workspace</code> 联合使用。
此标志可多次指定，并支持常见 Unix glob 模式，如 <code>*</code>、<code>?</code> 与 <code>[]</code>。
但为了避免 shell 在 Cargo 处理前提前展开 glob，必须将每个模式放在单引号或双引号中。</p>
</dd>


</dl>

### Target 选择

当未给出 target 选择选项时，`cargo doc` 会为所选 package 的全部二进制和库 target
生成文档。若二进制名称与 lib target 相同，则会跳过该二进制。
若二进制缺少 `required-features`，也会被跳过。

可在 manifest 的 target 设置中将 `doc = false` 以改变默认行为。
使用 target 选择选项时会忽略 `doc` 标志，并始终为给定 target 生成文档。

<dl>
<dt class="option-term" id="option-cargo-doc---lib"><a class="option-anchor" href="#option-cargo-doc---lib"><code>--lib</code></a></dt>
<dd class="option-desc"><p>为该 package 的库生成文档。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---bin"><a class="option-anchor" href="#option-cargo-doc---bin"><code>--bin</code> <em>name</em></a></dt>
<dd class="option-desc"><p>为指定二进制生成文档。此标志可多次指定，并支持常见 Unix glob 模式。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---bins"><a class="option-anchor" href="#option-cargo-doc---bins"><code>--bins</code></a></dt>
<dd class="option-desc"><p>为所有二进制 target 生成文档。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---example"><a class="option-anchor" href="#option-cargo-doc---example"><code>--example</code> <em>name</em></a></dt>
<dd class="option-desc"><p>为指定示例生成文档。此标志可多次指定，并支持常见 Unix glob 模式。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---examples"><a class="option-anchor" href="#option-cargo-doc---examples"><code>--examples</code></a></dt>
<dd class="option-desc"><p>为所有示例 target 生成文档。</p>
</dd>


</dl>

### Feature 选择

feature 标志用于控制启用哪些 feature。若未给出 feature 选项，
每个选中 package 都会启用 `default` feature。

更多细节见[feature 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-doc--F"><a class="option-anchor" href="#option-cargo-doc--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-doc---features"><a class="option-anchor" href="#option-cargo-doc---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表，以空格或逗号分隔。workspace 成员的 feature
可用 <code>package-name/feature-name</code> 语法启用。此标志可多次指定，最终会启用
所有指定 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---all-features"><a class="option-anchor" href="#option-cargo-doc---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---no-default-features"><a class="option-anchor" href="#option-cargo-doc---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### 编译选项

<dl>

<dt class="option-term" id="option-cargo-doc---target"><a class="option-anchor" href="#option-cargo-doc---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>为指定目标架构生成文档。该标志可多次指定。默认是主机架构。
triple 的通用格式为 <code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>。</p>
<p>可选值：</p>
<ul>
<li><code>rustc --print target-list</code> 支持的任意 target。</li>
<li><code>"host-tuple"</code>：内部会替换为主机 target。当你对部分 crate 进行交叉编译，
又不想显式将主机机器也列为 target 时尤其有用（例如共享项目中的 <code>xtask</code>，
可能由多种主机平台共同开发）。</li>
<li>自定义 target 规范文件路径。见 <a href="../../rustc/targets/custom.html#custom-target-lookup-path">Custom Target Lookup Path</a>。</li>
</ul>
<p>也可通过 <code>build.target</code> <a href="../reference/config.html">配置项</a> 指定。</p>
<p>注意，指定该标志会让 Cargo 进入不同模式，target 产物会放入单独目录。
详见<a href="../reference/build-cache.html">构建缓存</a>文档。</p>
</dd>


<dt class="option-term" id="option-cargo-doc--r"><a class="option-anchor" href="#option-cargo-doc--r"><code>-r</code></a></dt>
<dt class="option-term" id="option-cargo-doc---release"><a class="option-anchor" href="#option-cargo-doc---release"><code>--release</code></a></dt>
<dd class="option-desc"><p>使用 <code>release</code> profile 为优化产物生成文档。
如需按名称选择特定 profile，也可使用 <code>--profile</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---profile"><a class="option-anchor" href="#option-cargo-doc---profile"><code>--profile</code> <em>name</em></a></dt>
<dd class="option-desc"><p>使用给定 profile 生成文档。
关于 profile 的更多细节见<a href="../reference/profiles.html">参考文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---timings"><a class="option-anchor" href="#option-cargo-doc---timings"><code>--timings</code></a></dt>
<dd class="option-desc"><p>输出每个编译步骤耗时信息，并跟踪并发信息随时间的变化。</p>
<p>构建结束时会在 <code>target/cargo-timings</code> 目录写入 <code>cargo-timing.html</code>。
另外还会生成带时间戳文件名的报告，便于查看历史运行。
这些报告仅适合人工阅读，不提供机器可读的耗时数据。</p>
</dd>



</dl>

### 输出选项

<dl>
<dt class="option-term" id="option-cargo-doc---target-dir"><a class="option-anchor" href="#option-cargo-doc---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>所有生成产物和中间文件的目录。也可通过环境变量
<code>CARGO_TARGET_DIR</code> 或 <code>build.target-dir</code> <a href="../reference/config.html">配置项</a> 指定。
默认是 workspace 根目录下的 <code>target</code>。</p>
</dd>

</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-doc--v"><a class="option-anchor" href="#option-cargo-doc--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-doc---verbose"><a class="option-anchor" href="#option-cargo-doc---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>启用详细输出。可指定两次得到“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-doc--q"><a class="option-anchor" href="#option-cargo-doc--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-doc---quiet"><a class="option-anchor" href="#option-cargo-doc---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---color"><a class="option-anchor" href="#option-cargo-doc---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。有效值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---message-format"><a class="option-anchor" href="#option-cargo-doc---message-format"><code>--message-format</code> <em>fmt</em></a></dt>
<dd class="option-desc"><p>诊断消息输出格式。可多次指定，由逗号分隔值组成。有效值：</p>
<ul>
<li><code>human</code>（默认）：人类可读文本格式。与 <code>short</code>、<code>json</code> 冲突。</li>
<li><code>short</code>：更短的人类可读文本。与 <code>human</code>、<code>json</code> 冲突。</li>
<li><code>json</code>：向 stdout 输出 JSON 消息。详见
<a href="../reference/external-tools.html#json-messages">参考文档</a>。
与 <code>human</code>、<code>short</code> 冲突。</li>
<li><code>json-diagnostic-short</code>：确保 JSON 消息中 <code>rendered</code> 字段包含 rustc 的
“short”渲染结果。不能与 <code>human</code> 或 <code>short</code> 同用。</li>
<li><code>json-diagnostic-rendered-ansi</code>：确保 JSON 消息中 <code>rendered</code> 字段包含嵌入式
ANSI 颜色码，以遵循 rustc 默认配色。不能与 <code>human</code> 或 <code>short</code> 同用。</li>
<li><code>json-render-diagnostics</code>：指示 Cargo 不在打印的 JSON 消息中直接包含 rustc
诊断，而由 Cargo 自行渲染来自 rustc 的 JSON 诊断。Cargo 自身的 JSON 诊断和
来自 rustc 的其他消息仍会输出。不能与 <code>human</code> 或 <code>short</code> 同用。</li>
</ul>
</dd>

</dl>

### Manifest 选项

<dl>
<dt class="option-term" id="option-cargo-doc---manifest-path"><a class="option-anchor" href="#option-cargo-doc---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---ignore-rust-version"><a class="option-anchor" href="#option-cargo-doc---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 声明。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---locked"><a class="option-anchor" href="#option-cargo-doc---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言必须使用与现有 <code>Cargo.lock</code> 初次生成时完全一致的依赖与版本。
出现以下任一情况时 Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>因依赖解析不同，Cargo 尝试修改锁文件。</li>
</ul>
<p>可用于追求确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---offline"><a class="option-anchor" href="#option-cargo-doc---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>阻止 Cargo 以任何理由访问网络。未设置该标志时，如果 Cargo 需要访问网络但网络
不可用，会直接报错。设置后，Cargo 会在可能情况下尝试离线继续。</p>
<p>注意，这可能导致与在线模式不同的依赖解析结果。Cargo 会限制为仅使用本地已下载
crate，即便本地索引副本显示有更新版本。
可先用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖，再转入离线。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---frozen"><a class="option-anchor" href="#option-cargo-doc---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 与 <code>--offline</code>。</p>
</dd>


</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-doc-+toolchain"><a class="option-anchor" href="#option-cargo-doc-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>如果 Cargo 通过 rustup 安装，且 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
则会被解释为 rustup 工具链名称（如 <code>+stable</code> 或 <code>+nightly</code>）。
更多覆盖规则见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-doc---config"><a class="option-anchor" href="#option-cargo-doc---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数应为 TOML 语法的 <code>KEY=VALUE</code>，
或额外配置文件路径。此标志可多次指定。
更多信息见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-doc--C"><a class="option-anchor" href="#option-cargo-doc--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何指定操作前先切换当前工作目录。这会影响 cargo 默认查找项目
manifest（<code>Cargo.toml</code>）的位置，以及发现 <code>.cargo/config.toml</code> 时搜索的目录等。
该选项必须出现在命令名之前，例如 <code>cargo -C path/to/my-project doc</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly channel</a>
可用，并需要 `-Z unstable-options` 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-doc--h"><a class="option-anchor" href="#option-cargo-doc--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-doc---help"><a class="option-anchor" href="#option-cargo-doc---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-doc--Z"><a class="option-anchor" href="#option-cargo-doc--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>传递给 Cargo 的不稳定（仅 nightly）标志。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

### 杂项选项

<dl>
<dt class="option-term" id="option-cargo-doc--j"><a class="option-anchor" href="#option-cargo-doc--j"><code>-j</code> <em>N</em></a></dt>
<dt class="option-term" id="option-cargo-doc---jobs"><a class="option-anchor" href="#option-cargo-doc---jobs"><code>--jobs</code> <em>N</em></a></dt>
<dd class="option-desc"><p>并行任务数。也可通过 <code>build.jobs</code> <a href="../reference/config.html">配置项</a>
指定。默认值为逻辑 CPU 数。若为负数，则最大并行任务数设为
“逻辑 CPU 数 + 给定值”。若给定字符串 <code>default</code>，则恢复默认值。
不应为 0。</p>
</dd>

<dt class="option-term" id="option-cargo-doc---keep-going"><a class="option-anchor" href="#option-cargo-doc---keep-going"><code>--keep-going</code></a></dt>
<dd class="option-desc"><p>尽可能构建依赖图中的更多 crate，而不是在第一个构建失败时中止。</p>
<p>例如当前 package 依赖 <code>fails</code> 与 <code>works</code>，其中一个构建失败时，
<code>cargo doc -j1</code> 可能会也可能不会构建成功的那个（取决于 Cargo 先选中谁）；
而 <code>cargo doc -j1 --keep-going</code> 即使先执行的构建失败，也一定会执行两者。</p>
</dd>

</dl>

## 环境

Cargo 读取的环境变量详情见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 执行成功。
* `101`：Cargo 未能完成。

## 示例

1. 构建本地 package 及其依赖的文档，输出到 `target/doc`。

       cargo doc

## 另请参阅
[cargo(1)](cargo.html), [cargo-rustdoc(1)](cargo-rustdoc.html), [rustdoc(1)](https://doc.rust-lang.org/rustdoc/index.html)
