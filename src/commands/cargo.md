# cargo(1)

## 名称

cargo --- Rust 包管理器

## 概要

`cargo` [_options_] _command_ [_args_]\
`cargo` [_options_] `--version`\
`cargo` [_options_] `--list`\
`cargo` [_options_] `--help`\
`cargo` [_options_] `--explain` _code_

## 描述

这是 Rust 语言的包管理与构建工具，可在 <https://rust-lang.org> 获取。

_command_ 可以是：
- 内建命令（见下文）
- [别名]
- [外部工具]

[别名]: ../reference/config.html#alias
[外部工具]: ../reference/external-tools.html#custom-subcommands

## 命令

### 构建命令

[cargo-bench(1)](cargo-bench.html)\
&nbsp;&nbsp;&nbsp;&nbsp;执行 package 的基准测试。

[cargo-build(1)](cargo-build.html)\
&nbsp;&nbsp;&nbsp;&nbsp;编译 package。

[cargo-check(1)](cargo-check.html)\
&nbsp;&nbsp;&nbsp;&nbsp;检查本地 package 及其全部依赖中的错误。

[cargo-clean(1)](cargo-clean.html)\
&nbsp;&nbsp;&nbsp;&nbsp;删除 Cargo 过去生成的产物。

[cargo-doc(1)](cargo-doc.html)\
&nbsp;&nbsp;&nbsp;&nbsp;构建 package 文档。

[cargo-fetch(1)](cargo-fetch.html)\
&nbsp;&nbsp;&nbsp;&nbsp;从网络获取 package 依赖。

[cargo-fix(1)](cargo-fix.html)\
&nbsp;&nbsp;&nbsp;&nbsp;自动修复 rustc 报告的 lint 警告。

[cargo-run(1)](cargo-run.html)\
&nbsp;&nbsp;&nbsp;&nbsp;运行本地 package 的二进制或示例。

[cargo-rustc(1)](cargo-rustc.html)\
&nbsp;&nbsp;&nbsp;&nbsp;编译 package，并向编译器传递额外选项。

[cargo-rustdoc(1)](cargo-rustdoc.html)\
&nbsp;&nbsp;&nbsp;&nbsp;使用指定自定义标志构建 package 文档。

[cargo-test(1)](cargo-test.html)\
&nbsp;&nbsp;&nbsp;&nbsp;执行 package 的单元测试与集成测试。

### Manifest 命令

[cargo-add(1)](cargo-add.html)\
&nbsp;&nbsp;&nbsp;&nbsp;向 `Cargo.toml` manifest 文件添加依赖。

[cargo-generate-lockfile(1)](cargo-generate-lockfile.html)\
&nbsp;&nbsp;&nbsp;&nbsp;为项目生成 `Cargo.lock`。

[cargo-info(1)](cargo-info.html)\
&nbsp;&nbsp;&nbsp;&nbsp;显示 registry 中 package 的信息。默认 registry 为 crates.io。

[cargo-locate-project(1)](cargo-locate-project.html)\
&nbsp;&nbsp;&nbsp;&nbsp;以 JSON 形式输出 `Cargo.toml` 文件位置。

[cargo-metadata(1)](cargo-metadata.html)\
&nbsp;&nbsp;&nbsp;&nbsp;以机器可读格式输出 package 的解析后依赖信息。

[cargo-pkgid(1)](cargo-pkgid.html)\
&nbsp;&nbsp;&nbsp;&nbsp;打印完整限定的 package 规格。

[cargo-remove(1)](cargo-remove.html)\
&nbsp;&nbsp;&nbsp;&nbsp;从 `Cargo.toml` manifest 文件移除依赖。

[cargo-tree(1)](cargo-tree.html)\
&nbsp;&nbsp;&nbsp;&nbsp;以树形方式展示依赖图。

[cargo-update(1)](cargo-update.html)\
&nbsp;&nbsp;&nbsp;&nbsp;更新本地锁文件中记录的依赖。

[cargo-vendor(1)](cargo-vendor.html)\
&nbsp;&nbsp;&nbsp;&nbsp;将所有依赖本地 Vendor 化。

### 包管理命令

[cargo-init(1)](cargo-init.html)\
&nbsp;&nbsp;&nbsp;&nbsp;在已有目录中创建新的 Cargo package。

[cargo-install(1)](cargo-install.html)\
&nbsp;&nbsp;&nbsp;&nbsp;构建并安装 Rust 二进制。

[cargo-new(1)](cargo-new.html)\
&nbsp;&nbsp;&nbsp;&nbsp;创建新的 Cargo package。

[cargo-search(1)](cargo-search.html)\
&nbsp;&nbsp;&nbsp;&nbsp;在 crates.io 中搜索 package。

[cargo-uninstall(1)](cargo-uninstall.html)\
&nbsp;&nbsp;&nbsp;&nbsp;删除 Rust 二进制。

### 发布命令

[cargo-login(1)](cargo-login.html)\
&nbsp;&nbsp;&nbsp;&nbsp;将 registry 的 API token 保存到本地。

[cargo-logout(1)](cargo-logout.html)\
&nbsp;&nbsp;&nbsp;&nbsp;从本地移除 registry 的 API token。

[cargo-owner(1)](cargo-owner.html)\
&nbsp;&nbsp;&nbsp;&nbsp;管理 registry 上 crate 的所有者。

[cargo-package(1)](cargo-package.html)\
&nbsp;&nbsp;&nbsp;&nbsp;将本地 package 组装为可分发 tarball。

[cargo-publish(1)](cargo-publish.html)\
&nbsp;&nbsp;&nbsp;&nbsp;将 package 上传到 registry。

[cargo-yank(1)](cargo-yank.html)\
&nbsp;&nbsp;&nbsp;&nbsp;从索引中移除已推送 crate。

### 报告命令

[cargo-report(1)](cargo-report.html)\
&nbsp;&nbsp;&nbsp;&nbsp;生成并显示多种报告。

[cargo-report-future-incompatibilities(1)](cargo-report-future-incompatibilities.html)\
&nbsp;&nbsp;&nbsp;&nbsp;报告未来将停止编译的 crate。

### 通用命令

[cargo-help(1)](cargo-help.html)\
&nbsp;&nbsp;&nbsp;&nbsp;显示 Cargo 帮助信息。

[cargo-version(1)](cargo-version.html)\
&nbsp;&nbsp;&nbsp;&nbsp;显示版本信息。

## 选项

### 特殊选项

<dl>

<dt class="option-term" id="option-cargo--V"><a class="option-anchor" href="#option-cargo--V"><code>-V</code></a></dt>
<dt class="option-term" id="option-cargo---version"><a class="option-anchor" href="#option-cargo---version"><code>--version</code></a></dt>
<dd class="option-desc"><p>打印版本信息并退出。若与 <code>--verbose</code> 一起使用，会打印额外信息。</p>
</dd>


<dt class="option-term" id="option-cargo---list"><a class="option-anchor" href="#option-cargo---list"><code>--list</code></a></dt>
<dd class="option-desc"><p>列出所有已安装的 Cargo 子命令。若与 <code>--verbose</code> 一起使用，会打印额外信息。</p>
</dd>


<dt class="option-term" id="option-cargo---explain"><a class="option-anchor" href="#option-cargo---explain"><code>--explain</code> <em>code</em></a></dt>
<dd class="option-desc"><p>运行 <code>rustc --explain CODE</code>，输出错误消息的详细说明
（例如 <code>E0004</code>）。</p>
</dd>


</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo--v"><a class="option-anchor" href="#option-cargo--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo---verbose"><a class="option-anchor" href="#option-cargo---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>启用详细输出。可指定两次得到“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo--q"><a class="option-anchor" href="#option-cargo--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo---quiet"><a class="option-anchor" href="#option-cargo---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo---color"><a class="option-anchor" href="#option-cargo---color"><code>--color</code> <em>when</em></a></dt>
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
<dt class="option-term" id="option-cargo---locked"><a class="option-anchor" href="#option-cargo---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言必须使用与现有 <code>Cargo.lock</code> 初次生成时完全一致的依赖与版本。
出现以下任一情况时 Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>因依赖解析不同，Cargo 尝试修改锁文件。</li>
</ul>
<p>可用于追求确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo---offline"><a class="option-anchor" href="#option-cargo---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>阻止 Cargo 以任何理由访问网络。未设置该标志时，如果 Cargo 需要访问网络但网络
不可用，会直接报错。设置后，Cargo 会在可能情况下尝试离线继续。</p>
<p>注意，这可能导致与在线模式不同的依赖解析结果。Cargo 会限制为仅使用本地已下载
crate，即便本地索引副本显示有更新版本。
可先用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖，再转入离线。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置项</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo---frozen"><a class="option-anchor" href="#option-cargo---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 与 <code>--offline</code>。</p>
</dd>

</dl>

### 通用选项

<dl>

<dt class="option-term" id="option-cargo-+toolchain"><a class="option-anchor" href="#option-cargo-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>如果 Cargo 通过 rustup 安装，且 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
则会被解释为 rustup 工具链名称（如 <code>+stable</code> 或 <code>+nightly</code>）。
更多覆盖规则见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>。</p>
</dd>


<dt class="option-term" id="option-cargo---config"><a class="option-anchor" href="#option-cargo---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数应为 TOML 语法的 <code>KEY=VALUE</code>，
或额外配置文件路径。此标志可多次指定。
更多信息见<a href="../reference/config.html#command-line-overrides">命令行覆盖</a>章节。</p>
</dd>


<dt class="option-term" id="option-cargo--C"><a class="option-anchor" href="#option-cargo--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何指定操作前先切换当前工作目录。这会影响 cargo 默认查找项目
manifest（<code>Cargo.toml</code>）的位置，以及发现 <code>.cargo/config.toml</code> 时搜索的目录等。
该选项必须出现在命令名之前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly channel</a>
可用，并需要 `-Z unstable-options` 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo--h"><a class="option-anchor" href="#option-cargo--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo---help"><a class="option-anchor" href="#option-cargo---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo--Z"><a class="option-anchor" href="#option-cargo--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>传递给 Cargo 的不稳定（仅 nightly）标志。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 执行成功。
* `101`：Cargo 未能完成。

## 文件

`~/.cargo/`\
&nbsp;&nbsp;&nbsp;&nbsp;Cargo“主目录”的默认位置，会在此存储各类文件。可通过 `CARGO_HOME`
环境变量修改该位置。

`$CARGO_HOME/bin/`\
&nbsp;&nbsp;&nbsp;&nbsp;由 [cargo-install(1)](cargo-install.html) 安装的二进制位于此处。
若使用 [rustup]，Rust 随附可执行文件也在此目录。

`$CARGO_HOME/config.toml`\
&nbsp;&nbsp;&nbsp;&nbsp;全局配置文件。关于配置文件的更多信息见
[参考文档](../reference/config.html)。

`.cargo/config.toml`\
&nbsp;&nbsp;&nbsp;&nbsp;Cargo 会自动在当前目录及所有父目录中搜索名为 `.cargo/config.toml`
的文件。这些配置文件会与全局配置文件合并。

`$CARGO_HOME/credentials.toml`\
&nbsp;&nbsp;&nbsp;&nbsp;用于登录 registry 的私有认证信息。

`$CARGO_HOME/registry/`\
&nbsp;&nbsp;&nbsp;&nbsp;该目录包含 registry 索引缓存下载内容及已下载依赖。

`$CARGO_HOME/git/`\
&nbsp;&nbsp;&nbsp;&nbsp;该目录包含 git 依赖的缓存下载内容。

请注意：`$CARGO_HOME` 目录的内部结构尚未稳定，未来可能变化。

[rustup]: https://rust-lang.github.io/rustup/

## 示例

1. 构建本地 package 及其全部依赖：

       cargo build

2. 以优化模式构建 package：

       cargo build --release

3. 为交叉编译目标运行测试：

       cargo test --target i686-unknown-linux-gnu

4. 创建一个可构建可执行文件的新 package：

       cargo new foobar

5. 在当前目录创建 package：

       mkdir foo && cd foo
       cargo init .

6. 查看某个命令的选项与用法：

       cargo help clean

## BUGS

问题请见 <https://github.com/rust-lang/cargo/issues>。

## 另请参阅

[rustc(1)](https://doc.rust-lang.org/rustc/index.html), [rustdoc(1)](https://doc.rust-lang.org/rustdoc/index.html)
