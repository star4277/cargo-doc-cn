# cargo-add(1)
## 名称

cargo-add --- 向 Cargo.toml manifest 添加依赖

## 概要

`cargo add` [_options_] _crate_...\
`cargo add` [_options_] `--path` _path_\
`cargo add` [_options_] `--git` _url_ [_crate_...]


## 描述

该命令可用于添加或修改依赖。

依赖来源可通过以下方式指定：

* _crate_`@`_version_：从 registry 获取，并带上版本约束“_version_”
* `--path` _path_：从指定 _path_ 获取
* `--git` _url_：从 _url_ 对应的 git 仓库拉取

如果未指定来源，Cargo 会尽力自动选择，可能包括：

* 其他依赖表中的已有依赖（如 `dev-dependencies`）
* workspace 成员
* registry 中的最新发布版本

当你添加的 package 已存在时，现有条目会按本次指定的标志进行更新。

命令成功执行后，输出中会列出该依赖被启用（`+`）和禁用（`-`）的 [features]。

[features]: ../reference/features.html

## 选项

### 来源选项

<dl>

<dt class="option-term" id="option-cargo-add---git"><a class="option-anchor" href="#option-cargo-add---git"><code>--git</code> <em>url</em></a></dt>
<dd class="option-desc"><p><a href="../reference/specifying-dependencies.html#specifying-dependencies-from-git-repositories">从该 Git URL 添加指定 crate</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add---branch"><a class="option-anchor" href="#option-cargo-add---branch"><code>--branch</code> <em>branch</em></a></dt>
<dd class="option-desc"><p>从 git 添加时使用的分支。</p>
</dd>


<dt class="option-term" id="option-cargo-add---tag"><a class="option-anchor" href="#option-cargo-add---tag"><code>--tag</code> <em>tag</em></a></dt>
<dd class="option-desc"><p>从 git 添加时使用的 tag。</p>
</dd>


<dt class="option-term" id="option-cargo-add---rev"><a class="option-anchor" href="#option-cargo-add---rev"><code>--rev</code> <em>sha</em></a></dt>
<dd class="option-desc"><p>从 git 添加时使用的指定提交。</p>
</dd>


<dt class="option-term" id="option-cargo-add---path"><a class="option-anchor" href="#option-cargo-add---path"><code>--path</code> <em>path</em></a></dt>
<dd class="option-desc"><p>要添加的本地 crate 的<a href="../reference/specifying-dependencies.html#specifying-path-dependencies">文件系统路径</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add---base"><a class="option-anchor" href="#option-cargo-add---base"><code>--base</code> <em>base</em></a></dt>
<dd class="option-desc"><p>添加本地 crate 时使用的 <a href="../reference/unstable.html#path-bases">path base</a>。</p>
<p><a href="../reference/unstable.html#path-bases">不稳定（仅 nightly）</a></p>
</dd>


<dt class="option-term" id="option-cargo-add---registry"><a class="option-anchor" href="#option-cargo-add---registry"><code>--registry</code> <em>registry</em></a></dt>
<dd class="option-desc"><p>使用的 registry 名称。registry 名称定义在 <a href="../reference/config.html">Cargo 配置文件</a>中。
若未指定，使用默认 registry，由 <code>registry.default</code> 配置键定义，默认为 <code>crates-io</code>。</p>
</dd>


</dl>

### 分区选项

<dl>

<dt class="option-term" id="option-cargo-add---dev"><a class="option-anchor" href="#option-cargo-add---dev"><code>--dev</code></a></dt>
<dd class="option-desc"><p>作为<a href="../reference/specifying-dependencies.html#development-dependencies">开发依赖</a>添加。</p>
</dd>


<dt class="option-term" id="option-cargo-add---build"><a class="option-anchor" href="#option-cargo-add---build"><code>--build</code></a></dt>
<dd class="option-desc"><p>作为<a href="../reference/specifying-dependencies.html#build-dependencies">构建依赖</a>添加。</p>
</dd>


<dt class="option-term" id="option-cargo-add---target"><a class="option-anchor" href="#option-cargo-add---target"><code>--target</code> <em>target</em></a></dt>
<dd class="option-desc"><p>作为<a href="../reference/specifying-dependencies.html#platform-specific-dependencies">指定目标平台</a>的依赖添加。</p>
<p>为避免 shell 意外展开，建议给每个 target 加引号，例如 <code>--target 'cfg(unix)'</code>。</p>
</dd>


</dl>

### 依赖选项

<dl>

<dt class="option-term" id="option-cargo-add---dry-run"><a class="option-anchor" href="#option-cargo-add---dry-run"><code>--dry-run</code></a></dt>
<dd class="option-desc"><p>不实际写入 manifest。</p>
</dd>


<dt class="option-term" id="option-cargo-add---rename"><a class="option-anchor" href="#option-cargo-add---rename"><code>--rename</code> <em>name</em></a></dt>
<dd class="option-desc"><p><a href="../reference/specifying-dependencies.html#renaming-dependencies-in-cargotoml">重命名</a>该依赖。</p>
</dd>


<dt class="option-term" id="option-cargo-add---optional"><a class="option-anchor" href="#option-cargo-add---optional"><code>--optional</code></a></dt>
<dd class="option-desc"><p>将该依赖标记为<a href="../reference/features.html#optional-dependencies">可选</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add---no-optional"><a class="option-anchor" href="#option-cargo-add---no-optional"><code>--no-optional</code></a></dt>
<dd class="option-desc"><p>将该依赖标记为<a href="../reference/features.html#optional-dependencies">必需</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add---public"><a class="option-anchor" href="#option-cargo-add---public"><code>--public</code></a></dt>
<dd class="option-desc"><p>将该依赖标记为公开（public）。</p>
<p>该依赖可以在你的库对外 API 中被引用。</p>
<p><a href="../reference/unstable.html#public-dependency">不稳定（仅 nightly）</a></p>
</dd>


<dt class="option-term" id="option-cargo-add---no-public"><a class="option-anchor" href="#option-cargo-add---no-public"><code>--no-public</code></a></dt>
<dd class="option-desc"><p>将该依赖标记为私有（private）。</p>
<p>你仍可在实现中使用该 crate，但不能在公共 API 中引用它。</p>
<p><a href="../reference/unstable.html#public-dependency">不稳定（仅 nightly）</a></p>
</dd>


<dt class="option-term" id="option-cargo-add---no-default-features"><a class="option-anchor" href="#option-cargo-add---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>禁用<a href="../reference/features.html#dependency-features">默认 feature</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add---default-features"><a class="option-anchor" href="#option-cargo-add---default-features"><code>--default-features</code></a></dt>
<dd class="option-desc"><p>重新启用<a href="../reference/features.html#dependency-features">默认 feature</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add--F"><a class="option-anchor" href="#option-cargo-add--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-add---features"><a class="option-anchor" href="#option-cargo-add---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要<a href="../reference/features.html#dependency-features">启用的 feature</a> 列表，
以空格或逗号分隔。添加多个 crate 时，可用 <code>package-name/feature-name</code>
语法为特定 crate 启用 feature。该标志可多次指定，最终会启用所有指定 feature。</p>
</dd>


</dl>


### 显示选项

<dl>
<dt class="option-term" id="option-cargo-add--v"><a class="option-anchor" href="#option-cargo-add--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-add---verbose"><a class="option-anchor" href="#option-cargo-add---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>启用详细输出。可指定两次得到“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-add--q"><a class="option-anchor" href="#option-cargo-add--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-add---quiet"><a class="option-anchor" href="#option-cargo-add---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-add---color"><a class="option-anchor" href="#option-cargo-add---color"><code>--color</code> <em>when</em></a></dt>
<dd class="option-desc"><p>控制何时使用彩色输出。有效值：</p>
<ul>
<li><code>auto</code>（默认）：自动检测终端是否支持颜色。</li>
<li><code>always</code>：始终显示颜色。</li>
<li><code>never</code>：从不显示颜色。</li>
</ul>
<p>也可通过 <code>term.color</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>

</dl>

### Manifest 选项

<dl>
<dt class="option-term" id="option-cargo-add---manifest-path"><a class="option-anchor" href="#option-cargo-add---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-add--p"><a class="option-anchor" href="#option-cargo-add--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-add---package"><a class="option-anchor" href="#option-cargo-add---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅为指定 package 添加依赖。</p>
</dd>


<dt class="option-term" id="option-cargo-add---ignore-rust-version"><a class="option-anchor" href="#option-cargo-add---ignore-rust-version"><code>--ignore-rust-version</code></a></dt>
<dd class="option-desc"><p>忽略 package 中的 <code>rust-version</code> 声明。</p>
</dd>


<dt class="option-term" id="option-cargo-add---locked"><a class="option-anchor" href="#option-cargo-add---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言必须使用与现有 <code>Cargo.lock</code> 初次生成时完全一致的依赖与版本。
出现以下任一情况时 Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>因依赖解析不同，Cargo 尝试修改锁文件。</li>
</ul>
<p>可用于追求确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-add---offline"><a class="option-anchor" href="#option-cargo-add---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>阻止 Cargo 以任何理由访问网络。未设置该标志时，如果 Cargo 需要访问网络但网络
不可用，会直接报错。设置后，Cargo 会在可能情况下尝试离线继续。</p>
<p>注意，这可能导致与在线模式不同的依赖解析结果。Cargo 会限制为仅使用本地已下载
crate，即便本地索引副本显示有更新版本。
可先用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖，再转入离线。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-add---frozen"><a class="option-anchor" href="#option-cargo-add---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 与 <code>--offline</code>。</p>
</dd>


</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-add-+toolchain"><a class="option-anchor" href="#option-cargo-add-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>如果 Cargo 通过 rustup 安装，且 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
则会被解释为 rustup 工具链名称（如 <code>+stable</code> 或 <code>+nightly</code>）。
更多覆盖规则见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo-add---config"><a class="option-anchor" href="#option-cargo-add---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数应为 TOML 语法的 <code>KEY=VALUE</code>，
或额外配置文件路径。此标志可多次指定。
更多信息见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo-add--C"><a class="option-anchor" href="#option-cargo-add--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何指定操作前先切换当前工作目录。这会影响 cargo 默认查找项目
manifest（<code>Cargo.toml</code>）的位置，以及发现 <code>.cargo/config.toml</code> 时搜索的目录等。
该选项必须出现在命令名之前，例如 <code>cargo -C path/to/my-project add</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly channel</a>
可用，并需要 `-Z unstable-options` 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-add--h"><a class="option-anchor" href="#option-cargo-add--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-add---help"><a class="option-anchor" href="#option-cargo-add---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-add--Z"><a class="option-anchor" href="#option-cargo-add--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>传递给 Cargo 的不稳定（仅 nightly）标志。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 执行成功。
* `101`：Cargo 未能完成。

## 示例

1. 添加 `regex` 作为依赖

       cargo add regex

2. 添加 `trybuild` 作为开发依赖

       cargo add --dev trybuild

3. 添加较旧版本的 `nom` 作为依赖

       cargo add nom@5

4. 添加 `serde` 与 `serde_json`，并开启 `derive` 以支持 JSON 序列化

       cargo add serde serde_json -F serde/derive

5. 添加 `windows` 作为 `cfg(windows)` 的平台特定依赖

       cargo add windows --target 'cfg(windows)'

## 另请参阅
[cargo(1)](cargo.html), [cargo-remove(1)](cargo-remove.html)
