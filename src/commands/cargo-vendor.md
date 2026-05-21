# cargo-vendor(1)

## 名称

cargo-vendor --- 将所有依赖本地 Vendor 化

## 概要

`cargo vendor` [_options_] [_path_]

## 描述

该 Cargo 子命令会把项目中所有 crates.io 和 git 依赖
vendor 到 `<path>` 指定目录。
命令完成后，`<path>` 目录会包含依赖中所有远程来源代码。
除默认 manifest 外，也可通过 `-s` 指定其他 manifest。

`cargo vendor` 完成后，会把使用这些 vendored 源所需的配置打印到 stdout。
你需要把这些配置添加到（或重定向到）Cargo 配置文件中，
通常是当前 package 下的 `.cargo/config.toml`。

Cargo 会像对待 registry 和 git 源一样，将 vendored 源视为只读。
如果你打算修改远程来源中的 crate，
请使用 `[patch]` 或指向该 crate 本地副本的 `path` 依赖。
这样 Cargo 在增量构建时才能正确处理，
因为它知道该依赖不再是只读来源。

## 选项

### Vendor 选项

<dl>

<dt class="option-term" id="option-cargo-vendor--s"><a class="option-anchor" href="#option-cargo-vendor--s"><code>-s</code> <em>manifest</em></a></dt>
<dt class="option-term" id="option-cargo-vendor---sync"><a class="option-anchor" href="#option-cargo-vendor---sync"><code>--sync</code> <em>manifest</em></a></dt>
<dd class="option-desc"><p>指定额外的 <code>Cargo.toml</code> manifest（通常来自其他 workspace），
这些依赖也会被 vendor 并同步到输出目录。
该参数可重复指定多次。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---no-delete"><a class="option-anchor" href="#option-cargo-vendor---no-delete"><code>--no-delete</code></a></dt>
<dd class="option-desc"><p>vendor 时不删除 <code>vendor</code> 目录，
而是保留其中已有内容。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---respect-source-config"><a class="option-anchor" href="#option-cargo-vendor---respect-source-config"><code>--respect-source-config</code></a></dt>
<dd class="option-desc"><p>默认情况下会忽略 <code>.cargo/config.toml</code> 中的 <code>[source]</code> 配置。
加上该参数后会读取并使用这些配置（例如下载 crates.io crate 时）。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---versioned-dirs"><a class="option-anchor" href="#option-cargo-vendor---versioned-dirs"><code>--versioned-dirs</code></a></dt>
<dd class="option-desc"><p>默认仅在同名 package 多版本并存时才在目录名中加版本号。
该参数会让 <code>vendor</code> 目录中的所有目录都带版本号，
便于跟踪 vendored package 的历史变化，
也有助于在只变动部分 package 时提升重复 vendor 的性能。</p>
</dd>


</dl>

### Manifest 选项

<dl>

<dt class="option-term" id="option-cargo-vendor---manifest-path"><a class="option-anchor" href="#option-cargo-vendor---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---locked"><a class="option-anchor" href="#option-cargo-vendor---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---offline"><a class="option-anchor" href="#option-cargo-vendor---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---frozen"><a class="option-anchor" href="#option-cargo-vendor---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>



</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo-vendor--v"><a class="option-anchor" href="#option-cargo-vendor--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-vendor---verbose"><a class="option-anchor" href="#option-cargo-vendor---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor--q"><a class="option-anchor" href="#option-cargo-vendor--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-vendor---quiet"><a class="option-anchor" href="#option-cargo-vendor---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---color"><a class="option-anchor" href="#option-cargo-vendor---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-vendor-+toolchain"><a class="option-anchor" href="#option-cargo-vendor-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor---config"><a class="option-anchor" href="#option-cargo-vendor---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor--C"><a class="option-anchor" href="#option-cargo-vendor--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor--h"><a class="option-anchor" href="#option-cargo-vendor--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-vendor---help"><a class="option-anchor" href="#option-cargo-vendor---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-vendor--Z"><a class="option-anchor" href="#option-cargo-vendor--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 将所有依赖 vendor 到本地 `vendor` 目录

       cargo vendor

2. 将所有依赖 vendor 到本地 `third-party/vendor` 目录

       cargo vendor third-party/vendor

3. 将当前 workspace 以及另一个 workspace 一并 vendor 到 `vendor`

       cargo vendor -s ../path/to/Cargo.toml

4. vendor 并把所需配置重定向到配置文件

       cargo vendor > path/to/my/cargo/config.toml

## 另请参见
[cargo(1)](cargo.html)
