# cargo-yank(1)

## 名称

cargo-yank --- 从索引中移除已发布 crate

## 概要

`cargo yank` [_options_] _crate_@_version_\
`cargo yank` [_options_] `--version` _version_ [_crate_]

## 描述

yank 命令会把之前发布的某个 crate 版本从服务器索引中移除。
该命令不会删除数据，该 crate 仍可通过注册表下载链接获取。

对于没有既有锁文件的新项目或新检出，Cargo 不会再使用被 yank 的版本。
如果不存在兼容版本，Cargo 会报错。

该命令要求你已认证：
可通过 `--token` 选项，或先执行 [cargo-login(1)](cargo-login.html)。

若未指定 crate 名称，会使用当前目录 package 的名称。

### yank 的工作方式

例如，`foo` 发布了 `1.5.0`，另一个 crate `bar` 声明依赖 `foo = "1.5"`。
随后 `foo` 发布了不兼容的 `2.0.0`，并发现 `1.5.0` 有严重问题。
如果 yank 掉 `1.5.0`，那么没有既有锁文件的新项目/新检出将无法使用 `bar`，
因为它依赖于 `1.5`。

这种情况下，`foo` 维护者应先发布一个 SemVer 兼容版本（如 `1.5.1`），
再 yank `1.5.0`，这样 `bar` 及依赖 `bar` 的项目仍可正常工作。

再看一个例子：crate `bar` 已发布 `1.5.0`、`1.5.1`、`1.5.2`、`2.0.0`、`3.0.0`。
下表展示在“无锁文件”场景下，不同 SemVer 约束在某个版本被 yank 后，
Cargo 可选择的版本：

| Yanked Version / SemVer requirement | `bar = "1.5.0"`                         | `bar = "=1.5.0"` | `bar = "2.0.0"`  |
|-------------------------------------|-----------------------------------------|------------------|------------------|
| `1.5.0`                             | 使用 `1.5.1` 或 `1.5.2`                 | **返回错误**      | 使用 `2.0.0`      |
| `1.5.1`                             | 使用 `1.5.0` 或 `1.5.2`                 | 使用 `1.5.0`      | 使用 `2.0.0`      |
| `2.0.0`                             | 使用 `1.5.0`、`1.5.1` 或 `1.5.2`        | 使用 `1.5.0`      | **返回错误**      |

### 何时应当 yank

只有在例外场景才应 yank crate，例如误发布、非预期 SemVer 破坏，
或 crate 严重损坏且不可用。
对于安全漏洞，通常 [RustSec] 是更不具破坏性的通知机制，
可提示用户升级，并避免下游在是否受影响之外产生大规模中断。

常见流程是：先发布一个 SemVer 兼容版本，再 yank 有问题版本，
以降低阻断依赖方编译的概率。

若是处理版权、许可证或个人数据问题，仅 yank 可能不足以解决。
这类情况应联系你使用的注册表维护者。
对于 crates.io，请参见其 [policies]，并可联系 <help@crates.io>。

如果凭据泄露，建议立即撤销凭据。
crate 一旦发布，无法确认泄露凭据是否已被复制。
yank 只能阻止 Cargo 在默认依赖解析中选择该版本，
对现有锁文件或直接下载无影响，因此不能阻止泄露凭据继续传播。

[RustSec]: https://rustsec.org/
[policies]: https://crates.io/policies

## 选项

### Yank 选项

<dl>

<dt class="option-term" id="option-cargo-yank---vers"><a class="option-anchor" href="#option-cargo-yank---vers"><code>--vers</code> <em>version</em></a></dt>
<dt class="option-term" id="option-cargo-yank---version"><a class="option-anchor" href="#option-cargo-yank---version"><code>--version</code> <em>version</em></a></dt>
<dd class="option-desc"><p>要 yank 或 un-yank 的版本。</p>
</dd>


<dt class="option-term" id="option-cargo-yank---undo"><a class="option-anchor" href="#option-cargo-yank---undo"><code>--undo</code></a></dt>
<dd class="option-desc"><p>撤销 yank，使该版本重新回到索引中。</p>
</dd>


<dt class="option-term" id="option-cargo-yank---token"><a class="option-anchor" href="#option-cargo-yank---token"><code>--token</code> <em>token</em></a></dt>
<dd class="option-desc"><p>认证时使用的 API token。该值会覆盖凭据文件中的 token
（凭据文件由 <a href="cargo-login.html">cargo-login(1)</a> 创建）。</p>
<p>也可以使用 <a href="../reference/config.html">Cargo 配置</a> 对应的环境变量
覆盖凭据文件中的 token。crates.io 的 token 可通过
<code>CARGO_REGISTRY_TOKEN</code> 指定。
其他注册表可通过形如 <code>CARGO_REGISTRIES_NAME_TOKEN</code> 的环境变量指定，
其中 <code>NAME</code> 是注册表名的大写形式。</p>
</dd>


<dt class="option-term" id="option-cargo-yank---index"><a class="option-anchor" href="#option-cargo-yank---index"><code>--index</code> <em>index</em></a></dt>
<dd class="option-desc"><p>要使用的注册表索引 URL。</p>
</dd>


<dt class="option-term" id="option-cargo-yank---registry"><a class="option-anchor" href="#option-cargo-yank---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>要使用的注册表名称。注册表名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，则使用默认注册表，该默认值由 <code>registry.default</code> 配置键决定，
其默认值是 <code>crates-io</code>。</p>
</dd>


</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo-yank--v"><a class="option-anchor" href="#option-cargo-yank--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-yank---verbose"><a class="option-anchor" href="#option-cargo-yank---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-yank--q"><a class="option-anchor" href="#option-cargo-yank--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-yank---quiet"><a class="option-anchor" href="#option-cargo-yank---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-yank---color"><a class="option-anchor" href="#option-cargo-yank---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-yank-+toolchain"><a class="option-anchor" href="#option-cargo-yank-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-yank---config"><a class="option-anchor" href="#option-cargo-yank---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-yank--C"><a class="option-anchor" href="#option-cargo-yank--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-yank--h"><a class="option-anchor" href="#option-cargo-yank--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-yank---help"><a class="option-anchor" href="#option-cargo-yank---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-yank--Z"><a class="option-anchor" href="#option-cargo-yank--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 从索引 yank 一个 crate：

       cargo yank foo@1.0.7

## 另请参见
[cargo(1)](cargo.html), [cargo-login(1)](cargo-login.html), [cargo-publish(1)](cargo-publish.html)
