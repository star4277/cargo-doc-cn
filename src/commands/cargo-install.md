# cargo-install(1)
## 名称

cargo-install --- 构建并安装 Rust 二进制

## 概要

`cargo install` [_options_] _crate_[@_version_]...\
`cargo install` [_options_] `--path` _path_\
`cargo install` [_options_] `--git` _url_ [_crate_...]\
`cargo install` [_options_] `--list`

## 描述

该命令用于管理 Cargo 本地已安装的二进制 crate 集合。
只有包含可执行 `[[bin]]` 或 `[[example]]` 目标的 package 才能安装，
所有可执行文件会安装到安装根目录的 `bin` 文件夹。
默认只安装 binaries，不安装 examples。

安装根目录优先级如下：

- `--root` 选项
- `CARGO_INSTALL_ROOT` 环境变量
- `install.root` Cargo [配置值](../reference/config.html)
- `CARGO_HOME` 环境变量
- `$HOME/.cargo`

crate 可从多个来源安装。
默认来源是 crates.io，但可通过 `--git`、`--path`、`--registry` 改变来源。
如果来源中包含多个 package（例如 crates.io 或含多个 crate 的 git 仓库），
则必须通过 _crate_ 参数指定要安装的 crate。

来自 crates.io 的 crate 可通过 `--version` 指定安装版本；
来自 git 仓库的 package 可通过分支、tag 或 revision 指定。
若 crate 含多个二进制，可用 `--bin` 选择其中一个；
若想安装示例，可用 `--example`。

若 package 已安装，且当前安装看起来不是最新状态，Cargo 会重新安装。
当以下任一项变化时，Cargo 会重装：

- package 版本与来源。
- 已安装二进制名称集合。
- 所选 features。
- profile（`--profile`）。
- target（`--target`）。

使用 `--path` 安装时通常总会构建并安装，
除非与其他 package 的二进制产生冲突。
可用 `--force` 强制总是重装。

若来源是 crates.io 或 `--git`，默认会在临时 target 目录构建。
如需避免，可设置环境变量 `CARGO_TARGET_DIR` 指向指定路径。
这在 CI 中缓存构建产物时尤其有用。

### Lockfile 处理

默认会忽略 package 内附带的 `Cargo.lock`。
这意味着 Cargo 会重新计算依赖版本，可能使用发布后出现的较新版本。
可使用 `--locked` 强制 Cargo 使用打包内 `Cargo.lock`（若存在）。
这有助于可复现构建，使用发布时同一组依赖版本。
若某依赖新版本在你的系统上无法构建或有其他问题，也有帮助。
但缺点是你将收不到依赖的后续修复与更新。
注意 Cargo 从 1.37 起才开始随发布包携带 `Cargo.lock`，
更早版本发布的包可能没有该文件。

### 配置发现

该命令在系统/用户层级工作，而非项目层级。
这意味着会忽略本地的[配置发现]。
配置发现会从 `$CARGO_HOME/config.toml` 开始。
若使用 `--path $PATH` 安装 package，
则会使用本地配置，从 `$PATH/.cargo/config.toml` 开始发现。

[配置发现]: ../reference/config.html#hierarchical-structure

## 选项

### Install 选项

<dl>

<dt class="option-term" id="option-cargo-install---vers"><a class="option-anchor" href="#option-cargo-install---vers"><code>--vers</code> <em>version</em></a></dt>
<dt class="option-term" id="option-cargo-install---version"><a class="option-anchor" href="#option-cargo-install---version"><code>--version</code> <em>version</em></a></dt>
<dd class="option-desc"><p>指定要安装的版本。
可使用<a href="../reference/specifying-dependencies.html">版本要求</a>（如 <code>~1.2</code>），
让 Cargo 选择满足条件的最新版本。
若版本不含要求运算符（如 <code>^</code>、<code>~</code>），
则必须是 <em>MAJOR.MINOR.PATCH</em> 形式，并安装“精确版本”；
它<em>不会</em>像普通依赖那样按 caret 规则解释。</p>
</dd>


<dt class="option-term" id="option-cargo-install---git"><a class="option-anchor" href="#option-cargo-install---git"><code>--git</code> <em>url</em></a></dt>
<dd class="option-desc"><p>从指定 git URL 安装 crate。</p>
</dd>


<dt class="option-term" id="option-cargo-install---branch"><a class="option-anchor" href="#option-cargo-install---branch"><code>--branch</code> <em>branch</em></a></dt>
<dd class="option-desc"><p>从 git 安装时使用的分支。</p>
</dd>


<dt class="option-term" id="option-cargo-install---tag"><a class="option-anchor" href="#option-cargo-install---tag"><code>--tag</code> <em>tag</em></a></dt>
<dd class="option-desc"><p>从 git 安装时使用的 tag。</p>
</dd>


<dt class="option-term" id="option-cargo-install---rev"><a class="option-anchor" href="#option-cargo-install---rev"><code>--rev</code> <em>sha</em></a></dt>
<dd class="option-desc"><p>从 git 安装时使用的指定提交。</p>
</dd>


<dt class="option-term" id="option-cargo-install---path"><a class="option-anchor" href="#option-cargo-install---path"><code>--path</code> <em>path</em></a></dt>
<dd class="option-desc"><p>从本地 crate 路径安装。</p>
</dd>


<dt class="option-term" id="option-cargo-install---list"><a class="option-anchor" href="#option-cargo-install---list"><code>--list</code></a></dt>
<dd class="option-desc"><p>列出所有已安装 package 及版本。</p>
</dd>


<dt class="option-term" id="option-cargo-install--n"><a class="option-anchor" href="#option-cargo-install--n"><code>-n</code></a></dt>
<dt class="option-term" id="option-cargo-install---dry-run"><a class="option-anchor" href="#option-cargo-install---dry-run"><code>--dry-run</code></a></dt>
<dd class="option-desc"><p>（不稳定）执行所有检查但不安装。</p>
</dd>


<dt class="option-term" id="option-cargo-install--f"><a class="option-anchor" href="#option-cargo-install--f"><code>-f</code></a></dt>
<dt class="option-term" id="option-cargo-install---force"><a class="option-anchor" href="#option-cargo-install---force"><code>--force</code></a></dt>
<dd class="option-desc"><p>强制覆盖已有 crate 或二进制。
当某 package 安装了与其他 package 同名二进制时可使用。
当系统环境变化（例如 `rustc` 升级）需要重建时也有用。</p>
</dd>


<dt class="option-term" id="option-cargo-install---no-track"><a class="option-anchor" href="#option-cargo-install---no-track"><code>--no-track</code></a></dt>
<dd class="option-desc"><p>默认情况下 Cargo 会在安装根目录维护元数据文件以跟踪已安装包。
该参数会禁止使用或创建该文件。
启用后，除非使用 <code>--force</code>，否则 Cargo 会拒绝覆盖任何已有文件。
同时也会禁用 Cargo 对“并发安装调用冲突”的保护能力。</p>
</dd>


<dt class="option-term" id="option-cargo-install---bin"><a class="option-anchor" href="#option-cargo-install---bin"><code>--bin</code> <em>name</em></a></dt>
<dd class="option-desc"><p>仅安装指定二进制。</p>
</dd>


<dt class="option-term" id="option-cargo-install---bins"><a class="option-anchor" href="#option-cargo-install---bins"><code>--bins</code></a></dt>
<dd class="option-desc"><p>安装全部二进制。这是默认行为。</p>
</dd>


<dt class="option-term" id="option-cargo-install---example"><a class="option-anchor" href="#option-cargo-install---example"><code>--example</code> <em>name</em></a></dt>
<dd class="option-desc"><p>仅安装指定示例。</p>
</dd>


<dt class="option-term" id="option-cargo-install---examples"><a class="option-anchor" href="#option-cargo-install---examples"><code>--examples</code></a></dt>
<dd class="option-desc"><p>安装全部示例。</p>
</dd>


<dt class="option-term" id="option-cargo-install---root"><a class="option-anchor" href="#option-cargo-install---root"><code>--root</code> <em>dir</em></a></dt>
<dd class="option-desc"><p>安装目标目录。</p>
</dd>


<dt class="option-term" id="option-cargo-install---registry"><a class="option-anchor" href="#option-cargo-install---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要使用的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用默认注册表，该默认值由 <code>registry.default</code> 配置键决定，
其默认值是 <code>crates-io</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-install---index"><a class="option-anchor" href="#option-cargo-install---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>要使用的注册表索引 URL。</p>
</dd>


</dl>

### Feature 选择

Feature 参数可用于控制启用哪些特性。
当未给出 feature 选项时，所有被选 package 都会启用 `default` feature。

更多细节见
[features 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-install--F"><a class="option-anchor" href="#option-cargo-install--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-install---features"><a class="option-anchor" href="#option-cargo-install---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表（空格或逗号分隔）。
可通过 <code>package-name/feature-name</code> 语法启用 workspace 成员的 feature。
该参数可重复指定，最终会启用所有给出的 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-install---all-features"><a class="option-anchor" href="#option-cargo-install---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-install---no-default-features"><a class="option-anchor" href="#option-cargo-install---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### 编译选项

<dl>

<dt class="option-term" id="option-cargo-install---target"><a class="option-anchor" href="#option-cargo-install---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>为指定目标架构安装。默认是主机架构。
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


<dt class="option-term" id="option-cargo-install---target-dir"><a class="option-anchor" href="#option-cargo-install---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>所有生成产物与中间文件所在目录。
也可通过环境变量 <code>CARGO_TARGET_DIR</code> 或
<code>build.target-dir</code> <a href="../reference/config.html">配置值</a> 指定。
默认会在系统临时目录下创建新的临时文件夹。</p>
<p>使用 <code>--path</code> 时，默认会使用本地 crate 所在 workspace 的
<code>target</code> 目录，除非显式指定 <code>--target-dir</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-install---debug"><a class="option-anchor" href="#option-cargo-install---debug"><code>--debug</code></a></dt>
<dd class="option-desc"><p>使用 <code>dev</code> profile 构建，而非 <code>release</code> profile。
若要按名称选择特定 profile，也可使用 <code>--profile</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-install---profile"><a class="option-anchor" href="#option-cargo-install---profile"><code>--profile</code> <em>name</em></a></dt>
<dd class="option-desc"><p>使用给定 profile 安装。
更多信息见 <a href="../reference/profiles.html">参考文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-install---timings"><a class="option-anchor" href="#option-cargo-install---timings"><code>--timings</code></a></dt>
<dd class="option-desc"><p>输出各编译步骤耗时，以及随时间变化的并发信息。</p>
<p>构建结束后会在 <code>target/cargo-timings</code> 目录写入
<code>cargo-timing.html</code>。
此外还会写入一个文件名带时间戳的报告，便于查看历史运行。
这些报告仅适合人工阅读，不提供机器可读的时序数据。</p>
</dd>



</dl>

### Manifest 选项

<dl>
<dt class="option-term" id="option-cargo-install---ignore-rust-version"><a class="option-anchor" href="#option-cargo-install---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 约束。</p>
</dd>


<dt class="option-term" id="option-cargo-install---locked"><a class="option-anchor" href="#option-cargo-install---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-install---offline"><a class="option-anchor" href="#option-cargo-install---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-install---frozen"><a class="option-anchor" href="#option-cargo-install---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>

</dl>

### 其他选项

<dl>
<dt class="option-term" id="option-cargo-install--j"><a class="option-anchor" href="#option-cargo-install--j"><code>-j</code> <em>N</em></a></dt>
<dt class="option-term" id="option-cargo-install---jobs"><a class="option-anchor" href="#option-cargo-install---jobs"><code>--jobs</code> <em>N</em></a></dt>
<dd class="option-desc"><p>并行任务数。也可通过 <code>build.jobs</code> <a href="../reference/config.html">配置值</a> 指定。
默认值为逻辑 CPU 数。
若为负数，则表示“逻辑 CPU 数 + 给定值”。
若为字符串 <code>default</code>，则恢复默认值。
不能为 0。</p>
</dd>

<dt class="option-term" id="option-cargo-install---keep-going"><a class="option-anchor" href="#option-cargo-install---keep-going"><code>--keep-going</code></a></dt>
<dd class="option-desc"><p>尽可能构建依赖图中更多 crate，而不是在第一个失败时立即中止。</p>
<p>例如当前 package 依赖 <code>fails</code> 与 <code>works</code>，其中一个构建失败。
<code>cargo install -j1</code> 可能会也可能不会构建成功的那个
（取决于 Cargo 先选择构建哪一个）；
而 <code>cargo install -j1 --keep-going</code> 一定会尝试两者，
即便先执行的那个失败。</p>
</dd>

</dl>

### 显示选项

<dl>
<dt class="option-term" id="option-cargo-install--v"><a class="option-anchor" href="#option-cargo-install--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-install---verbose"><a class="option-anchor" href="#option-cargo-install---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-install--q"><a class="option-anchor" href="#option-cargo-install--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-install---quiet"><a class="option-anchor" href="#option-cargo-install---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-install---color"><a class="option-anchor" href="#option-cargo-install---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。可选值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-install---message-format"><a class="option-anchor" href="#option-cargo-install---message-format"><code>--message-format</code> <em>fmt</em></a></dt>
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

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-install-+toolchain"><a class="option-anchor" href="#option-cargo-install-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-install---config"><a class="option-anchor" href="#option-cargo-install---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-install--C"><a class="option-anchor" href="#option-cargo-install--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-install--h"><a class="option-anchor" href="#option-cargo-install--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-install---help"><a class="option-anchor" href="#option-cargo-install---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-install--Z"><a class="option-anchor" href="#option-cargo-install--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 从 crates.io 安装或升级 package：

       cargo install ripgrep

2. 安装或重装当前目录中的 package：

       cargo install --path .

3. 查看已安装 package 列表：

       cargo install --list

## 另请参见
[cargo(1)](cargo.html), [cargo-uninstall(1)](cargo-uninstall.html), [cargo-search(1)](cargo-search.html), [cargo-publish(1)](cargo-publish.html)
