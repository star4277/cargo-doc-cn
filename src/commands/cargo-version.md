# cargo-version(1)

## 名称

cargo-version --- 显示版本信息

## 概要

`cargo version` [_options_]

## 描述

显示 Cargo 的版本。

## 选项

<dl>

<dt class="option-term" id="option-cargo-version--v"><a class="option-anchor" href="#option-cargo-version--v"><code>-v</code></a></dt>
<dt class="option-term" id="option-cargo-version---verbose"><a class="option-anchor" href="#option-cargo-version---verbose"><code>--verbose</code></a></dt>
<dd class="option-desc"><p>显示附加的版本信息。</p>
</dd>


</dl>

## 示例

1. 显示版本：

       cargo version

2. 也可通过参数查看版本：

       cargo --version
       cargo -V

3. 显示额外版本信息：

       cargo -Vv

## 另请参见
[cargo(1)](cargo.html)
