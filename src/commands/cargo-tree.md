# cargo-tree(1)
## 名称

cargo-tree --- 以树形方式展示依赖图

## 概要

`cargo tree` [_options_]

## 描述

该命令会在终端展示依赖树。下面是一个依赖 `rand` 的简单项目示例：

```text
myproject v0.1.0 (/myproject)
├── rand v0.7.3
│   ├── getrandom v0.1.14
│   │   ├── cfg-if v0.1.10
│   │   └── libc v0.2.68
│   ├── libc v0.2.68 (*)
│   ├── rand_chacha v0.2.2
│   │   ├── ppv-lite86 v0.2.6
│   │   └── rand_core v0.5.1
│   │       └── getrandom v0.1.14 (*)
│   └── rand_core v0.5.1 (*)
[build-dependencies]
└── cc v1.0.50
```

标记为 `(*)` 的 package 表示已“去重显示”：
该 package 的依赖已在图中其他位置展示，因此此处不再重复。
可使用 `--no-dedupe` 关闭去重并重复显示。

可通过 `-e` 选择要展示的依赖边类型。
其中 `features` 类型会切换为展示“每个依赖启用了哪些 feature”。
例如 `cargo tree -e features`：

```text
myproject v0.1.0 (/myproject)
└── log feature "serde"
    └── log v0.4.8
        ├── serde v1.0.106
        └── cfg-if feature "default"
            └── cfg-if v0.1.10
```

在这棵树里，`myproject` 依赖 `log` 并启用了 `serde` feature。
`log` 又依赖 `cfg-if` 并启用了 `default` feature。
使用 `-e features` 时，通常配合 `-i` 查看 feature 如何流入某个 package。
详见后面的示例。

### Feature 统一（Feature Unification）

该命令展示的图更接近 Cargo 最终构建时的“feature 已统一”结果，
而不是 `Cargo.toml` 中原始逐项声明。
例如同一依赖同时出现在 `[dependencies]` 和 `[dev-dependencies]`，
且启用 feature 不同，输出里可能会合并这些 feature，
并在某个重复位置标记 `(*)`。

因此，如果你想看接近 `cargo build` 的视图，
`cargo tree -e normal,build` 通常比较接近；
想看接近 `cargo test` 的视图，`cargo tree` 通常比较接近。
但它并不保证与 Cargo 实际构建完全等价，
因为真实编译受很多因素影响。

关于 feature 统一的更多信息可参考
[专门章节](../reference/features.html#feature-unification)。

## 选项

### Tree 选项

<dl>

<dt class="option-term" id="option-cargo-tree--i"><a class="option-anchor" href="#option-cargo-tree--i"><code>-i</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-tree---invert"><a class="option-anchor" href="#option-cargo-tree---invert"><code>--invert</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>显示给定 package 的“反向依赖”。
该参数会反转树形方向，展示依赖该 package 的上游包。</p>
<p>注意：在 workspace 中，默认只会显示当前目录对应 workspace 成员子树内的反向依赖。
可用 <code>--workspace</code> 扩展为整个 workspace 的反向依赖。
可用 <code>-p</code> 仅展示给定 <code>-p</code> package 子树内的反向依赖。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---prune"><a class="option-anchor" href="#option-cargo-tree---prune"><code>--prune</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>从依赖树显示中裁剪掉给定 package。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---depth"><a class="option-anchor" href="#option-cargo-tree---depth"><code>--depth</code> <em>depth</em></a></dt>
<dd class="option-desc"><p>依赖树最大展示深度。
例如深度为 1 时，仅显示直接依赖。</p>
<p>若值为 <code>workspace</code>，则仅显示当前 workspace 成员依赖。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---no-dedupe"><a class="option-anchor" href="#option-cargo-tree---no-dedupe"><code>--no-dedupe</code></a></dt>
<dd class="option-desc"><p>不对重复依赖去重。
通常某 package 的依赖一旦已经展示，后续再次出现时不会重复展开，
仅以 <code>(*)</code> 表示“此前已出现”。
该参数会让这些重复项也完整展开。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--d"><a class="option-anchor" href="#option-cargo-tree--d"><code>-d</code></a></dt>
<dt class="option-term" id="option-cargo-tree---duplicates"><a class="option-anchor" href="#option-cargo-tree---duplicates"><code>--duplicates</code></a></dt>
<dd class="option-desc"><p>仅显示存在多版本并存的依赖（隐含 <code>--invert</code>）。
与 <code>-p</code> 一起使用时，仅显示该 package 子树中的重复项。</p>
<p>避免同一 package 多次构建通常有利于构建时间和可执行文件体积。
该参数可帮助定位问题 package。
你可以进一步检查：依赖旧版本的包能否升级到新版本，
从而只构建一个版本实例。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--e"><a class="option-anchor" href="#option-cargo-tree--e"><code>-e</code> <em>kinds</em></a></dt>
<dt class="option-term" id="option-cargo-tree---edges"><a class="option-anchor" href="#option-cargo-tree---edges"><code>--edges</code> <em>kinds</em></a></dt>
<dd class="option-desc"><p>要显示的依赖边类型，逗号分隔：</p>
<ul>
<li><code>all</code>：显示所有边类型。</li>
<li><code>normal</code>：显示普通依赖。</li>
<li><code>build</code>：显示构建依赖。</li>
<li><code>dev</code>：显示开发依赖。</li>
<li><code>features</code>：显示每个依赖启用的 feature。
若仅指定该类型，会自动包含其它依赖类型。</li>
<li><code>no-normal</code>：不包含普通依赖。</li>
<li><code>no-build</code>：不包含构建依赖。</li>
<li><code>no-dev</code>：不包含开发依赖。</li>
<li><code>no-proc-macro</code>：不包含过程宏依赖。</li>
</ul>
<p><code>normal</code>、<code>build</code>、<code>dev</code>、<code>all</code>
不能与 <code>no-normal</code>、<code>no-build</code>、<code>no-dev</code> 混用。</p>
<p>默认值为 <code>normal,build,dev</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---target"><a class="option-anchor" href="#option-cargo-tree---target"><code>--target</code> <em>triple</em></a></dt>
<dd class="option-desc"><p>按给定 <a href="../appendix/glossary.html#target">target triple</a> 过滤依赖。
默认是主机平台。使用 <code>all</code> 可包含所有目标。</p>
</dd>


</dl>

### Tree 格式选项

<dl>

<dt class="option-term" id="option-cargo-tree---charset"><a class="option-anchor" href="#option-cargo-tree---charset"><code>--charset</code> <em>charset</em></a></dt>
<dd class="option-desc"><p>选择树形字符集。可选值为 <code>utf8</code> 或 <code>ascii</code>。
未指定时 Cargo 会自动选择。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--f"><a class="option-anchor" href="#option-cargo-tree--f"><code>-f</code> <em>format</em></a></dt>
<dt class="option-term" id="option-cargo-tree---format"><a class="option-anchor" href="#option-cargo-tree---format"><code>--format</code> <em>format</em></a></dt>
<dd class="option-desc"><p>设置每个 package 的格式字符串。默认是 <code>{p}</code>。</p>
<p>这是用于展示每个 package 的模板字符串，可替换占位符：</p>
<ul>
<li><code>{p}</code>、<code>{package}</code>：package 名称。</li>
<li><code>{l}</code>、<code>{license}</code>：package 许可证。</li>
<li><code>{r}</code>、<code>{repository}</code>：package 仓库 URL。</li>
<li><code>{f}</code>、<code>{features}</code>：已启用 feature 的逗号分隔列表。</li>
<li><code>{lib}</code>：该 package 库在 <code>use</code> 语句中使用的名称。</li>
</ul>
</dd>


<dt class="option-term" id="option-cargo-tree---prefix"><a class="option-anchor" href="#option-cargo-tree---prefix"><code>--prefix</code> <em>prefix</em></a></dt>
<dd class="option-desc"><p>设置每行显示方式。<em>prefix</em> 可选值：</p>
<ul>
<li><code>indent</code>（默认）：按树形缩进显示。</li>
<li><code>depth</code>：列表显示，并在每行前打印数字深度。</li>
<li><code>none</code>：平铺列表显示。</li>
</ul>
</dd>


</dl>

### 包选择

默认情况下（未给出包选择选项时），选中哪些 package 取决于所选 manifest
（若未指定 `--manifest-path`，则基于当前工作目录推断）。
如果 manifest 是 workspace 根，则选择 workspace 的默认成员；
否则仅选择该 manifest 定义的 package。

workspace 默认成员可通过根 manifest 的
`workspace.default-members` 显式设置。
若未设置：
虚拟 workspace 会包含所有成员（等价于 `--workspace`），
非虚拟 workspace 仅包含根 crate 本身。

<dl>

<dt class="option-term" id="option-cargo-tree--p"><a class="option-anchor" href="#option-cargo-tree--p"><code>-p</code> <em>spec</em></a></dt>
<dt class="option-term" id="option-cargo-tree---package"><a class="option-anchor" href="#option-cargo-tree---package"><code>--package</code> <em>spec</em></a></dt>
<dd class="option-desc"><p>仅显示指定 package。SPEC 格式见 <a href="cargo-pkgid.html">cargo-pkgid(1)</a>。
该参数可重复指定，支持常见 Unix glob（如 <code>*</code>、<code>?</code>、<code>[]</code>）。
为避免 shell 先行展开 glob，必须将每个模式用单引号或双引号包裹。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---workspace"><a class="option-anchor" href="#option-cargo-tree---workspace"><code>--workspace</code></a></dt>
<dd class="option-desc"><p>显示 workspace 的所有成员。</p>
</dd>



<dt class="option-term" id="option-cargo-tree---exclude"><a class="option-anchor" href="#option-cargo-tree---exclude"><code>--exclude</code> <em>SPEC</em></a></dt>
<dd class="option-desc"><p>排除指定 package。必须与 <code>--workspace</code> 一起使用。
该参数可重复指定，支持常见 Unix glob（如 <code>*</code>、<code>?</code>、<code>[]</code>）。
为避免 shell 先行展开 glob，必须将每个模式用单引号或双引号包裹。</p>
</dd>


</dl>

### Manifest 选项

<dl>

<dt class="option-term" id="option-cargo-tree---manifest-path"><a class="option-anchor" href="#option-cargo-tree---manifest-path"><code>--manifest-path</code> <em>path</em></a></dt>
<dd class="option-desc"><p><code>Cargo.toml</code> 文件路径。默认情况下，Cargo 会在当前目录及其父目录中搜索
<code>Cargo.toml</code>。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---locked"><a class="option-anchor" href="#option-cargo-tree---locked"><code>--locked</code></a></dt>
<dd class="option-desc"><p>断言使用与现有 <code>Cargo.lock</code> 初次生成时完全相同的依赖和版本。
若出现以下任一情况，Cargo 会报错退出：</p>
<ul>
<li>锁文件缺失。</li>
<li>由于依赖解析变化，Cargo 试图修改锁文件。</li>
</ul>
<p>适用于需要确定性构建的环境，例如 CI 流水线。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---offline"><a class="option-anchor" href="#option-cargo-tree---offline"><code>--offline</code></a></dt>
<dd class="option-desc"><p>禁止 Cargo 以任何理由访问网络。
若不加此参数，在需要访问网络但网络不可用时，Cargo 会报错退出；
加上此参数后，Cargo 会在可能情况下尝试离线继续执行。</p>
<p>注意这可能导致与在线模式不同的依赖解析。
Cargo 只会使用本地已下载的 crate，
即便本地索引显示存在更新版本也不会获取。
可在离线前先使用 <a href="cargo-fetch.html">cargo-fetch(1)</a> 下载依赖。</p>
<p>也可通过 <code>net.offline</code> <a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---frozen"><a class="option-anchor" href="#option-cargo-tree---frozen"><code>--frozen</code></a></dt>
<dd class="option-desc"><p>等价于同时指定 <code>--locked</code> 和 <code>--offline</code>。</p>
</dd>


</dl>

### Feature 选择

Feature 参数可用于控制启用哪些特性。
当未给出 feature 选项时，所有被选 package 都会启用 `default` feature。

更多细节见
[features 文档](../reference/features.html#command-line-feature-options)。

<dl>

<dt class="option-term" id="option-cargo-tree--F"><a class="option-anchor" href="#option-cargo-tree--F"><code>-F</code> <em>features</em></a></dt>
<dt class="option-term" id="option-cargo-tree---features"><a class="option-anchor" href="#option-cargo-tree---features"><code>--features</code> <em>features</em></a></dt>
<dd class="option-desc"><p>要启用的 feature 列表（空格或逗号分隔）。
可通过 <code>package-name/feature-name</code> 语法启用 workspace 成员的 feature。
该参数可重复指定，最终会启用所有给出的 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---all-features"><a class="option-anchor" href="#option-cargo-tree---all-features"><code>--all-features</code></a></dt>
<dd class="option-desc"><p>启用所有已选 package 的全部可用 feature。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---no-default-features"><a class="option-anchor" href="#option-cargo-tree---no-default-features"><code>--no-default-features</code></a></dt>
<dd class="option-desc"><p>不启用已选 package 的 <code>default</code> feature。</p>
</dd>


</dl>

### 显示选项

<dl>

<dt class="option-term" id="option-cargo-tree--v"><a class="option-anchor" href="#option-cargo-tree--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-tree---verbose"><a class="option-anchor" href="#option-cargo-tree---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>使用详细输出。可重复指定两次以获得“非常详细”输出，
其中包含依赖警告、构建脚本输出等额外信息。
也可通过 <code>term.verbose</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--q"><a class="option-anchor" href="#option-cargo-tree--q"><code>-q</code></a></dt>
<dt class="option-term" id="option-cargo-tree---quiet"><a class="option-anchor" href="#option-cargo-tree---quiet"><code>--quiet</code></a></dt>
<dd class="option-desc"><p>不打印 cargo 日志消息。
也可通过 <code>term.quiet</code>
<a href="../reference/config.html">配置值</a> 指定。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---color"><a class="option-anchor" href="#option-cargo-tree---color"><code>--color</code> <em>when</em></a></dt>
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

<dt class="option-term" id="option-cargo-tree-+toolchain"><a class="option-anchor" href="#option-cargo-tree-+toolchain"><code>+</code><em>toolchain</em></a></dt>
<dd class="option-desc"><p>若 Cargo 通过 rustup 安装，并且传给 <code>cargo</code> 的第一个参数以 <code>+</code> 开头，
它会被解释为 rustup 工具链名称（例如 <code>+stable</code> 或 <code>+nightly</code>）。
详情见 <a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档</a>
中关于工具链覆盖的说明。</p>
</dd>


<dt class="option-term" id="option-cargo-tree---config"><a class="option-anchor" href="#option-cargo-tree---config"><code>--config</code> <em>KEY=VALUE</em> or <em>PATH</em></a></dt>
<dd class="option-desc"><p>覆盖 Cargo 配置值。参数可为 TOML 语法的 <code>KEY=VALUE</code>，
也可为额外配置文件路径。此参数可多次指定。
详情见 <a href="../reference/config.html#command-line-overrides">命令行覆盖</a> 章节。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--C"><a class="option-anchor" href="#option-cargo-tree--C"><code>-C</code> <em>PATH</em></a></dt>
<dd class="option-desc"><p>在执行任何操作前切换当前工作目录。
这会影响 Cargo 默认查找项目清单（<code>Cargo.toml</code>）的位置，
以及发现 <code>.cargo/config.toml</code> 时的搜索目录等。
该选项必须出现在命令名前，例如 <code>cargo -C path/to/my-project build</code>。</p>
<p>该选项仅在 <a href="https://doc.rust-lang.org/book/appendix-07-nightly-rust.html">nightly
channel</a> 可用，
并且需要通过 <code>-Z unstable-options</code> 启用（见
<a href="https://github.com/rust-lang/cargo/issues/10098">#10098</a>）。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--h"><a class="option-anchor" href="#option-cargo-tree--h"><code>-h</code></a></dt>
<dt class="option-term" id="option-cargo-tree---help"><a class="option-anchor" href="#option-cargo-tree---help"><code>--help</code></a></dt>
<dd class="option-desc"><p>打印帮助信息。</p>
</dd>


<dt class="option-term" id="option-cargo-tree--Z"><a class="option-anchor" href="#option-cargo-tree--Z"><code>-Z</code> <em>flag</em></a></dt>
<dd class="option-desc"><p>Cargo 的不稳定（仅 nightly）参数。运行 <code>cargo -Z help</code> 查看详情。</p>
</dd>


</dl>

## 环境

Cargo 读取的环境变量详情，请参见[参考文档](../reference/environment-variables.html)。

## 退出状态

* `0`：Cargo 成功。
* `101`：Cargo 未能完成。

## 示例

1. 显示当前目录 package 的依赖树：

       cargo tree

2. 显示所有依赖 `syn` 的 package：

       cargo tree -i syn

3. 显示每个 package 启用的 feature：

       cargo tree --format "{p} {f}"

4. 仅显示被构建多次的 package。
   当树中出现多个 SemVer 不兼容版本（如 1.0.0 和 2.0.0）时可能发生。

       cargo tree -d

5. 解释 `syn` 的 feature 为什么会被启用：

       cargo tree -e features -i syn

   `-e features` 用于显示 feature，`-i` 用于反转图，
   以展示哪些 package 依赖 `syn`。
   输出示例：

   ```text
   syn v1.0.17
   ├── syn feature "clone-impls"
   │   └── syn feature "default"
   │       └── rustversion v1.0.2
   │           └── rustversion feature "default"
   │               └── myproject v0.1.0 (/myproject)
   │                   └── myproject feature "default" (command-line)
   ├── syn feature "default" (*)
   ├── syn feature "derive"
   │   └── syn feature "default" (*)
   ├── syn feature "full"
   │   └── rustversion v1.0.2 (*)
   ├── syn feature "parsing"
   │   └── syn feature "default" (*)
   ├── syn feature "printing"
   │   └── syn feature "default" (*)
   ├── syn feature "proc-macro"
   │   └── syn feature "default" (*)
   └── syn feature "quote"
       ├── syn feature "printing" (*)
       └── syn feature "proc-macro" (*)
   ```

   阅读该图时，可从根到叶按链路追踪每个 feature 的来源。
   例如，“full” feature 是由 `rustversion` 引入，
   `rustversion` 来自 `myproject`（启用默认 feature），
   而 `myproject` 是命令行选中的 package。
   其它 `syn` feature 多数来自 `default` feature
   （其中 “quote” 来自 “printing” 和 “proc-macro”，这两者都属于默认 feature）。

   若你在去重条目 `(*)` 间交叉定位困难，可加 `--no-dedupe` 查看完整展开输出。

## 另请参见
[cargo(1)](cargo.html), [cargo-metadata(1)](cargo-metadata.html)
