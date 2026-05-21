# cargo-test(1)
## 名称

cargo-test --- 执行 package 的单元测试与集成测试

## 概要

`cargo test` [_options_] [_testname_] [`--` _test-options_]

## 描述

编译并执行单元测试、集成测试和文档测试。

测试过滤参数 `TESTNAME` 以及双横线（`--`）之后的所有参数，都会传给测试二进制，
并进一步传给 _libtest_（rustc 内置的单元测试与微基准框架）。
如果你同时给 Cargo 和二进制传参，`--` 后面的参数传给二进制，前面的传给 Cargo。
关于 libtest 参数，参见 `cargo test -- --help` 输出以及 rustc 文档测试章节：
<https://doc.rust-lang.org/rustc/tests/index.html>。

例如，下面命令会筛选名称中包含 `foo` 的测试，并用 3 个线程并行运行：

    cargo test foo -- --test-threads 3

测试通过给 `rustc` 传入 `--test` 来构建，这会把你的代码与 libtest 链接成一个特殊
可执行文件。该可执行文件会在多线程中自动运行所有带 `#[test]` 标注的函数。
带 `#[bench]` 标注的函数也会执行一轮以验证其可用性。

如果 package 包含多个测试 target，每个 target 会分别编译为上述特殊可执行文件，
然后串行运行。

可以在 target 的 manifest 设置中将 `harness = false` 来禁用 libtest harness。
此时你的代码需要自行提供 `main` 函数来处理测试执行。

### 文档测试

默认也会运行文档测试，由 `rustdoc` 处理。它会从库 target 的文档注释中提取代码示例并执行。

与普通测试 target 不同，每个代码块会在运行时由 `rustc` 即时编译为一个 doctest 可执行文件。
这些可执行文件会在独立进程中并行运行。代码块编译本质上是由 libtest 控制的测试函数的一部分，
因此 `--jobs` 等部分选项可能不生效。注意 doctest 的执行模型不保证稳定，未来可能变化；
请避免依赖该行为。

编写文档测试的更多信息见 [rustdoc 手册](https://doc.rust-lang.org/rustdoc/)。

### 测试工作目录

运行每个单元测试和集成测试时，工作目录都会设置为该测试所属 package 的根目录。
这样无论从哪里执行 `cargo test`，测试都能稳定地通过相对路径访问 package 文件。

对于文档测试，调用 `rustdoc` 时的工作目录会被设置为 workspace 根目录，
这也是 `rustdoc` 用作每个文档测试编译目录的位置。
运行每个文档测试时的工作目录会设置为该测试所属 package 的根目录，
并通过 `rustdoc` 的 `--test-run-directory` 选项控制。

## 选项

### 测试选项

<dl>

<dt class="option-term" id="option-cargo-test---no-run"><a class="option-anchor" href="#option-cargo-test---no-run"><code>--no-run</code></a></dt>
<dd class="option-desc"><p>只编译，不运行测试。</p>
</dd>


<dt class="option-term" id="option-cargo-test---no-fail-fast"><a class="option-anchor" href="#option-cargo-test---no-fail-fast"><code>--no-fail-fast</code></a></dt>
<dd class="option-desc"><p>无论是否失败都运行全部测试。若不指定该标志，Cargo 会在第一个可执行文件失败后退出。
Rust 测试 harness 会把单个可执行文件中的所有测试都跑完；该标志仅作用于“可执行文件”这一层。</p>
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

<dt class="option-term" id="option-cargo-test--p"><a class="option-anchor" href="#option-cargo-test--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-test---package"><a class="option-anchor" href="#option-cargo-test---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅测试指定 package。SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。
此标志可多次指定，并支持常见 Unix glob 模式，如 <code>*</code>、<code>?</code> 与 <code>[]</code>。
但为了避免 shell 在 Cargo 处理前提前展开 glob，必须将每个模式放在单引号或双引号中。</p>
</dd>


<dt class="option-term" id="option-cargo-test---workspace"><a class="option-anchor" href="#option-cargo-test---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>测试 workspace 的全部成员。</p>
</dd>


<dt class="option-term" id="option-cargo-test---all"><a class="option-anchor" href="#option-cargo-test---all"><code>--all</code></a></dt>
<dd class="option-desc"><p><code>--workspace</code> 的已弃用别名。</p>
</dd>


<dt class="option-term" id="option-cargo-test---exclude"><a class="option-anchor" href="#option-cargo-test---exclude"><code>--exclude</code> <em>SPEC</em></a></dt>
<dd class="option-desc"><p>排除指定 package。必须与 <code>--workspace</code> 联合使用。
此标志可多次指定，并支持常见 Unix glob 模式，如 <code>*</code>、<code>?</code> 与 <code>[]</code>。
但为了避免 shell 在 Cargo 处理前提前展开 glob，必须将每个模式放在单引号或双引号中。</p>
</dd>


</dl>

### Target 选择

当未给出 target 选择选项时，`cargo test` 会为所选 package 构建以下 target：

- lib：用于与二进制、示例、集成测试和文档测试链接
- bins（仅当构建集成测试且所需 feature 可用时）
- examples：用于确保其可编译
- 作为单元测试构建的 lib
- 作为单元测试构建的 bins
- integration tests
- lib target 的文档测试

默认行为可通过 target 的 manifest 设置中的 `test` 标志修改。
将 example 设置为 `test = true` 会把 example 作为测试构建并运行，
并用 libtest harness 替换 example 的 `main` 函数。
如果你不希望替换 `main`，还应同时设置 `harness = false`，
此时 example 会按原样构建并执行。

将 target 设置为 `test = false` 可让其默认不参与测试。
按名称接收 target 的选择选项（如 `--example foo`）会忽略 `test` 标志，
始终测试给定 target。

可通过在 manifest 中为库设置 `doctest = false` 来禁用文档测试。

按 target 配置的更多信息见
[配置 target](../reference/cargo-targets.html#configuring-a-target)。

如果选择测试某个集成测试或基准，二进制 target 会自动构建。
这使集成测试可执行该二进制来验证其行为。
当集成测试被构建和运行时，会设置
`CARGO_BIN_EXE_<name>`
[环境变量](../reference/environment-variables.html#environment-variables-cargo-sets-for-crates)，
从而可通过 [`env` 宏](https://doc.rust-lang.org/std/macro.env.html) 或
[`var` 函数](https://doc.rust-lang.org/std/env/fn.var.html) 定位可执行文件。

传入 target 选择标志将仅测试指定 target。

注意，`--bin`、`--example`、`--test` 和 `--bench` 也支持常见 Unix glob
模式（如 `*`、`?`、`[]`）。但为避免 shell 在 Cargo 处理前提前展开 glob，必须
将每个 glob 模式放在单引号或双引号中。

<dl>

<dt class="option-term" id="option-cargo-test---lib"><a class="option-anchor" href="#option-cargo-test---lib"><code>--lib</code></a></dt>
<dd class="option-desc"><p>测试该 package 的库。</p>
</dd>


<dt class="option-term" id="option-cargo-test---bin"><a class="option-anchor" href="#option-cargo-test---bin"><code>--bin</code> <em>name</em></a></dt>
<dd class="option-desc"><p>测试指定二进制。此标志可多次指定，并支持常见 Unix glob 模式。</p>
</dd>


<dt class="option-term" id="option-cargo-test---bins"><a class="option-anchor" href="#option-cargo-test---bins"><code>--bins</code></a></dt>
<dd class="option-desc"><p>测试所有二进制 target。</p>
</dd>


<dt class="option-term" id="option-cargo-test---example"><a class="option-anchor" href="#option-cargo-test---example"><code>--example</code> <em>name</em></a></dt>
<dd class="option-desc"><p>测试指定示例。此标志可多次指定，并支持常见 Unix glob 模式。</p>
</dd>


<dt class="option-term" id="option-cargo-test---examples"><a class="option-anchor" href="#option-cargo-test---examples"><code>--examples</code></a></dt>
<dd class="option-desc"><p>测试所有示例 target。</p>
</dd>


<dt class="option-term" id="option-cargo-test---test"><a class="option-anchor" href="#option-cargo-test---test"><code>--test</code> <em>name</em></a></dt>
<dd class="option-desc"><p>测试指定集成测试。此标志可多次指定，并支持常见 Unix glob 模式。</p>
</dd>


<dt class="option-term" id="option-cargo-test---tests"><a class="option-anchor" href="#option-cargo-test---tests"><code>--tests</code></a></dt>
<dd class="option-desc"><p>测试所有在 manifest 中设置了 <code>test = true</code> 的 target。
默认包括作为单元测试构建的库和二进制，以及集成测试。
注意这也会构建所需依赖，因此 lib target 可能会被构建两次
（一次作为单元测试，一次作为二进制、集成测试等的依赖）。
可通过 target 的 manifest 设置中的 <code>test</code> 标志启用或禁用 target。</p>
</dd>


<dt class="option-term" id="option-cargo-test---bench"><a class="option-anchor" href="#option-cargo-test---bench"><code>--bench</code> <em>name</em></a></dt>
<dd class="option-desc"><p>测试指定基准测试。此标志可多次指定，并支持常见 Unix glob 模式。</p>
</dd>


<dt class="option-term" id="option-cargo-test---benches"><a class="option-anchor" href="#option-cargo-test---benches"><code>--benches</code></a></dt>
<dd class="option-desc"><p>测试所有在 manifest 中设置了 <code>bench = true</code> 的 target。
默认包括作为基准测试构建的库和二进制，以及 bench target。
注意这也会构建所需依赖，因此 lib target 可能会被构建两次
（一次作为基准测试，一次作为二进制、基准等的依赖）。
可通过 target 的 manifest 设置中的 <code>bench</code> 标志启用或禁用 target。</p>
</dd>


<dt class="option-term" id="option-cargo-test---all-targets"><a class="option-anchor" href="#option-cargo-test---all-targets"><code>--all-targets</code></a></dt>
<dd class="option-desc"><p>测试全部 target。等价于指定 <code>--lib --bins --tests --benches --examples</code>。</p>
</dd>


</dl>

<dl>

<dt class="option-term" id="option-cargo-test---doc"><a class="option-anchor" href="#option-cargo-test---doc"><code>--doc</code></a></dt>
<dd class="option-desc"><p>仅测试库文档。该选项不能与其他 target 选项混用。</p>
</dd>


</dl>

### Feature 选择

feature 标志用于控制启用哪些 feature。若未给出 feature 选项，
每个选中 package 都会启用 `default` feature。

更多细节见[feature 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-test--F"><a class="option-anchor" href="#option-cargo-test--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-test---features"><a class="option-anchor" href="#option-cargo-test---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表，以空格或逗号分隔。workspace 成员的 feature
可用 <code>package-name/feature-name</code> 语法启用。此标志可多次指定，最终会启用
所有指定 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-test---all-features"><a class="option-anchor" href="#option-cargo-test---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-test---no-default-features"><a class="option-anchor" href="#option-cargo-test---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### 编译选项

<dl>

<dt class="option-term" id="option-cargo-test---target"><a class="option-anchor" href="#option-cargo-test---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>为指定目标架构执行测试。该标志可多次指定。默认是主机架构。
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


<dt class="option-term" id="option-cargo-test--r"><a class="option-anchor" href="#option-cargo-test--r"><code>-r</code></a></dt>
<dt class="option-term" id="option-cargo-test---release"><a class="option-anchor" href="#option-cargo-test---release"><code>--release</code></a></dt>
<dd class="option-desc"><p>使用 <code>release</code> profile 测试优化产物。
如需按名称选择特定 profile，也可使用 <code>--profile</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-test---profile"><a class="option-anchor" href="#option-cargo-test---profile"><code>--profile</code> <em>name</em></a></dt>
<dd class="option-desc"><p>使用给定 profile 执行测试。
关于 profile 的更多细节见<a href="../reference/profiles.html">参考文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-test---timings"><a class="option-anchor" href="#option-cargo-test---timings"><code>--timings</code></a></dt>
<dd class="option-desc"><p>输出每个编译步骤耗时信息，并跟踪并发信息随时间的变化。</p>
<p>构建结束时会在 <code>target/cargo-timings</code> 目录写入 <code>cargo-timing.html</code>。
另外还会生成带时间戳文件名的报告，便于查看历史运行。
这些报告仅适合人工阅读，不提供机器可读的耗时数据。</p>
</dd>



</dl>

### 输出选项

<dl>
<dt class="option-term" id="option-cargo-test---target-dir"><a class="option-anchor" href="#option-cargo-test---target-dir"><code>--target-dir</code> <em>directory</em></a></dt>
<dd class="option-desc"><p>所有生成产物和中间文件的目录。也可通过环境变量
<code>CARGO_TARGET_DIR</code> 或 <code>build.target-dir</code> <a href="../reference/config.html">配置项</a> 指定。
默认是 workspace 根目录下的 <code>target</code>。</p>
</dd>

</dl>

### 显示选项

默认情况下，Rust 测试 harness 会隐藏测试输出，以保持结果可读性。
若要恢复输出（例如调试），可给测试二进制传入 `--no-capture`：

    cargo test -- --no-capture

<dl>

<dt class="option-term" id="option-cargo-test--v"><a class="option-anchor" href="#option-cargo-test--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-test---verbose"><a class="option-anchor" href="#option-cargo-test---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>启用详细输出。可指定两次得到“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-test--q"><a class="option-anchor" href="#option-cargo-test--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-test---quiet"><a class="option-anchor" href="#option-cargo-test---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-test---color"><a class="option-anchor" href="#option-cargo-test---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。有效值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-test---message-format"><a class="option-anchor" href="#option-cargo-test---message-format"><code>--message-format</code> <em>fmt</em></a></dt>
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

<dt class="option-term" id="option-cargo-test---manifest-path"><a class="option-anchor" href="#option-cargo-test---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-test---ignore-rust-version"><a class="option-anchor" href="#option-cargo-test---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 声明。</p>
</dd>


<dt class="option-term" id="option-cargo-test---locked"><a class="option-anchor" href="#option-cargo-test---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言必须使用与现有 <code>Cargo.lock</code> 初次生成时完全一致的依赖与版本。
出现以下任一情况时 Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>因依赖解析不同，Cargo 尝试修改锁文件。</li>
</ul>
<p>可用于追求确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-test---offline"><a class="option-anchor" href="#option-cargo-test---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>阻止 Cargo 以任何理由访问网络。未设置该标志时，如果 Cargo 需要访问网络但网络
不可用，会直接报错。设置后，Cargo 会在可能情况下尝试离线继续。</p>
<p>注意，这可能导致与在线模式不同的依赖解析结果。Cargo 会限制为仅使用本地已下载
crate，即便本地索引副本显示有更新版本。
可先用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖，再转入离线。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-test---frozen"><a class="option-anchor" href="#option-cargo-test---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 与 <code>--offline</code>。</p>
</dd>



</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-test-+toolchain"><a class="option-anchor" href="#option-cargo-test-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>如果 Cargo 通过 rustup 安装，且 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
则会被解释为 rustup 工具链名称（如 <code>+stable</code> 或 <code>+nightly</code>）。
更多覆盖规则见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-test---config"><a class="option-anchor" href="#option-cargo-test---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数应为 TOML 语法的 <code>KEY=VALUE</code>，
或额外配置文件路径。此标志可多次指定。
更多信息见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-test--C"><a class="option-anchor" href="#option-cargo-test--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何指定操作前先切换当前工作目录。这会影响 cargo 默认查找项目
manifest（<code>Cargo.toml</code>）的位置，以及发现 <code>.cargo/config.toml</code> 时搜索的目录等。
该选项必须出现在命令名之前，例如 <code>cargo -C path/to/my-project test</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly channel</a>
可用，并需要 `-Z unstable-options` 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-test--h"><a class="option-anchor" href="#option-cargo-test--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-test---help"><a class="option-anchor" href="#option-cargo-test---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-test--Z"><a class="option-anchor" href="#option-cargo-test--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>传递给 Cargo 的不稳定（仅 nightly）标志。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

### 杂项选项

`--jobs` 会影响测试可执行文件的构建，但不影响测试运行时线程数。
Rust 测试 harness 提供了控制测试线程数的选项：

    cargo test -j 2 -- --test-threads=2

<dl>

<dt class="option-term" id="option-cargo-test--j"><a class="option-anchor" href="#option-cargo-test--j"><code>-j</code> <em>N</em></a></dt>
<dt class="option-term" id="option-cargo-test---jobs"><a class="option-anchor" href="#option-cargo-test---jobs"><code>--jobs</code> <em>N</em></a></dt>
<dd class="option-desc"><p>并行任务数。也可通过 <code>build.jobs</code> <a href="../reference/config.html">配置项</a>
指定。默认值为逻辑 CPU 数。若为负数，则最大并行任务数设为
“逻辑 CPU 数 + 给定值”。若给定字符串 <code>default</code>，则恢复默认值。
不应为 0。</p>
</dd>

<dt class="option-term" id="option-cargo-test---future-incompat-report"><a class="option-anchor" href="#option-cargo-test---future-incompat-report"><code>--future-incompat-report</code></a></dt>
<dd class="option-desc"><p>显示该命令执行期间产生的未来不兼容警告报告。</p>
<p>参见 <a href="cargo-report.html">cargo-report(1)</a></p>
</dd>


</dl>

虽然 `cargo test` 涉及编译，但不提供 `--keep-going` 标志。
如需尽量运行更多测试而不是首个失败即停止，请使用 `--no-fail-fast`。
如需尽量“编译”更多测试，可使用 `--tests` 单独构建测试二进制，例如：

    cargo build --tests --keep-going
    cargo test --tests --no-fail-fast

## 环境

Cargo 读取的环境变量详情见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 执行成功。
* `101`：Cargo 未能完成。

## 示例

1. 执行当前 package 的全部单元测试和集成测试：

       cargo test

2. 仅运行名称匹配过滤字符串的测试：

       cargo test name_filter

3. 在指定集成测试中仅运行某个测试：

       cargo test --test int_test_name -- modname::test_name

## 另请参阅
[cargo(1)](cargo.html), [cargo-bench(1)](cargo-bench.html), [测试类型](../reference/cargo-targets.html#tests), [如何编写测试](https://doc.rust-lang.org/rustc/tests/index.html)
