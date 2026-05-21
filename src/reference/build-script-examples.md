# Build Script Examples

下面通过一些示例说明如何编写构建脚本。

在 [crates.io] 上有许多 crate 提供常见构建脚本能力。
可以查看 [`build-dependencies`
keyword](https://crates.io/keywords/build-dependencies) 了解可用项。
下面列出一些常见 crate[^1]：

* [`bindgen`](https://crates.io/crates/bindgen) --- 自动生成 C 库的 Rust FFI 绑定。
* [`cc`](https://crates.io/crates/cc) --- 编译 C/C++/汇编代码。
* [`pkg-config`](https://crates.io/crates/pkg-config) --- 使用 `pkg-config` 工具探测系统库。
* [`cmake`](https://crates.io/crates/cmake) --- 调用 `cmake` 构建本地库。
* [`autocfg`](https://crates.io/crates/autocfg)、
  [`rustc_version`](https://crates.io/crates/rustc_version)、
  [`version_check`](https://crates.io/crates/version_check) ---
  用于基于当前 `rustc`（如编译器版本）实现条件编译。

[^1]: 该列表不代表官方背书。请自行评估依赖是否适合你的项目。

## Code generation

有些 Cargo package 在编译前需要先生成代码。
下面通过一个简单示例演示：在构建脚本里生成库调用代码。

先看目录结构：

```text
.
├── Cargo.toml
├── build.rs
└── src
    └── main.rs

1 directory, 3 files
```

这里有一个 `build.rs` 构建脚本，以及 `main.rs` 二进制入口。
manifest 如下：

```toml
# Cargo.toml

[package]
name = "hello-from-generated-code"
version = "0.1.0"
edition = "2024"
```

再看构建脚本内容：

```rust,no_run
// build.rs

use std::env;
use std::fs;
use std::path::Path;

fn main() {
    let out_dir = env::var_os("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("hello.rs");
    fs::write(
        &dest_path,
        "pub fn message() -> &'static str {
            \"Hello, World!\"
        }
        "
    ).unwrap();
    println!("cargo::rerun-if-changed=build.rs");
}
```

这里有几点要注意：

* 脚本通过 `OUT_DIR` 环境变量确定输出文件位置。
  输入文件一般可通过进程当前工作目录定位；本例没有输入文件。
* 一般来说，构建脚本不应修改 `OUT_DIR` 之外的文件。
  这在你自己项目里看似可行，但作为依赖被他人使用时会出问题：
  `.cargo/registry` 中源码默认应视为不可变。
  `cargo` 在打包时也不会允许这类脚本。
  * 有些项目希望把生成文件提交进仓库并当源码使用。
    这种情况下，不应由 `build.rs` 直接生成。
    更合适做法是写测试或类似检查，确保仓库内文件与“生成结果”完全一致，
    一旦不一致就失败，并在 CI 中执行该检查。
    （测试可生成临时文件做比对；若要更新生成文件，可用临时文件覆盖仓库版本。）
* 本脚本较简单，只生成一个小文件。
  更复杂场景可能是从 C 头文件或其他语言定义生成 Rust 模块。
* [`rerun-if-changed` 指令](build-scripts.md#rerun-if-changed)
  告诉 Cargo：仅当构建脚本本身变化时才重跑脚本。
  如果没有这行，Cargo 默认会在 package 任一文件变化时重跑构建脚本。
  若代码生成依赖输入文件，应在这里输出这些输入文件路径列表。

再看库代码：

```rust,ignore
// src/main.rs

include!(concat!(env!("OUT_DIR"), "/hello.rs"));

fn main() {
    println!("{}", message());
}
```

关键点在这里：
它使用 rustc 提供的 [`include!` macro][include-macro]，
配合 [`concat!`][concat-macro] 和 [`env!`][env-macro]，
把生成文件（`hello.rs`）直接纳入 crate 编译。

按这种结构，构建脚本可以生成并引入任意数量的文件。

[include-macro]: ../../std/macro.include.html
[concat-macro]: ../../std/macro.concat.html
[env-macro]: ../../std/macro.env.html

## Building a native library

有时 package 需要构建一些本地 C/C++ 代码。
这也是构建脚本的典型用途：先构建本地库，再构建 Rust crate。
下面示例将创建一个调用 C 输出 `Hello, World!` 的 Rust 库。

先看目录布局：

```text
.
├── Cargo.toml
├── build.rs
└── src
    ├── hello.c
    └── main.rs

1 directory, 4 files
```

和上一个例子很像。再看 manifest：

```toml
# Cargo.toml

[package]
name = "hello-world-from-c"
version = "0.1.0"
edition = "2024"
```

先不引入 build 依赖，直接看构建脚本：

```rust,no_run
// build.rs

use std::process::Command;
use std::env;
use std::path::Path;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();

    // Note that there are a number of downsides to this approach, the comments
    // below detail how to improve the portability of these commands.
    Command::new("gcc").args(&["src/hello.c", "-c", "-fPIC", "-o"])
                       .arg(&format!("{}/hello.o", out_dir))
                       .status().unwrap();
    Command::new("ar").args(&["crus", "libhello.a", "hello.o"])
                      .current_dir(&Path::new(&out_dir))
                      .status().unwrap();

    println!("cargo::rustc-link-search=native={}", out_dir);
    println!("cargo::rustc-link-lib=static=hello");
    println!("cargo::rerun-if-changed=src/hello.c");
}
```

该脚本先调用 `gcc` 把 C 文件编译为目标文件，再调用 `ar` 打包为静态库。
最后通过输出指令告诉 Cargo：
产物在 `out_dir`，并让编译器通过 `-l static=hello` 静态链接 `libhello.a`。

这种硬编码写法有几个问题：

* `gcc` 命令不具备跨平台可移植性。
  例如 Windows 常没有 `gcc`，并且并非所有 Unix 平台都一定有。
  `ar` 也有类似问题。
* 未考虑交叉编译。
  例如目标是 Android 时，本机 `gcc` 大概率不会产出 ARM 可执行代码。

这时就该使用 `build-dependencies` 了。
Cargo 生态有不少 crate 能把这类工作做得更易用、更可移植、更标准化。
我们用 [crates.io] 上的 [`cc`
crate](https://crates.io/crates/cc) 试一下。
先在 `Cargo.toml` 里加入：

```toml
[build-dependencies]
cc = "1.0"
```

并把构建脚本改成：

```rust,ignore
// build.rs

fn main() {
    cc::Build::new()
        .file("src/hello.c")
        .compile("hello");
    println!("cargo::rerun-if-changed=src/hello.c");
}
```

[`cc` crate] 对 C 代码构建做了大量抽象：

* 自动选择合适编译器（Windows 的 MSVC、MinGW 的 `gcc`、Unix 的 `cc` 等）。
* 读取 `TARGET` 并传递正确编译参数。
* 自动处理 `OPT_LEVEL`、`DEBUG` 等环境变量。
* 自动处理 stdout 输出与 `OUT_DIR` 路径。

这也体现了一个重要收益：
把通用能力尽量下沉到公共 build dependency，
比在每个构建脚本里重复造轮子更可靠。

回到例子，看下 `src` 目录内容：

```c
// src/hello.c

#include <stdio.h>

void hello() {
    printf("Hello, World!\n");
}
```

```rust,ignore
// src/main.rs

// Note the lack of the `#[link]` attribute. We’re delegating the responsibility
// of selecting what to link over to the build script rather than hard-coding
// it in the source file.
unsafe extern { fn hello(); }

fn main() {
    unsafe { hello(); }
}
```

这样就完成了：在 Cargo package 中通过构建脚本构建 C 代码。
也可以看到，在很多场景里 build dependency 不仅关键，而且代码更简洁。

你也看到了一个典型模式：
某些 crate 只在构建阶段作为依赖，并不会进入运行时依赖。

[`cc` crate]: https://crates.io/crates/cc

## Linking to system libraries

这个例子演示如何链接系统库，以及构建脚本在其中的作用。

Rust crate 经常需要链接系统提供的本地库：
要么做功能绑定，要么作为实现细节。
要把它做成平台无关实现，细节很多。
最佳实践通常是：尽可能把复杂性封装掉，降低使用方负担。

这里我们将为系统 `zlib` 库创建绑定。
`zlib` 在多数类 Unix 系统上常见，提供数据压缩能力。
真实项目里通常直接用 [`libz-sys`
crate]，本例只做极简版。
完整实现请看[源码][libz-source]。

为便于定位库，我们使用 [`pkg-config` crate]。
它调用系统 `pkg-config` 工具获取库信息，
并自动告诉 Cargo 如何链接该库。
该方式通常适用于安装了 `pkg-config` 的类 Unix 系统。
先写 manifest：

```toml
# Cargo.toml

[package]
name = "libz-sys"
version = "0.1.0"
edition = "2024"
links = "z"

[build-dependencies]
pkg-config = "0.3.16"
```

注意 `[package]` 里有 `links` 键。
这告诉 Cargo：当前 crate 会链接 `libz`。
后文 ["Using another sys crate"](#using-another-sys-crate)
会利用这一点。

构建脚本很简单：

```rust,ignore
// build.rs

fn main() {
    pkg_config::Config::new().probe("zlib").unwrap();
    println!("cargo::rerun-if-changed=build.rs");
}
```

再配一个基础 FFI 绑定：

```rust,ignore
// src/lib.rs

use std::os::raw::{c_uint, c_ulong};

unsafe extern "C" {
    pub fn crc32(crc: c_ulong, buf: *const u8, len: c_uint) -> c_ulong;
}

#[test]
fn test_crc32() {
    let s = "hello";
    unsafe {
        assert_eq!(crc32(0, s.as_ptr(), s.len() as c_uint), 0x3610a686);
    }
}
```

执行 `cargo build -vv` 可查看构建脚本输出。
在已安装 `libz` 的系统上，可能类似：

```text
[libz-sys 0.1.0] cargo::rustc-link-search=native=/usr/lib
[libz-sys 0.1.0] cargo::rustc-link-lib=z
[libz-sys 0.1.0] cargo::rerun-if-changed=build.rs
```

很好，`pkg-config` 已完成“查找库并告知 Cargo 链接方式”的工作。

很多 package 还会内置库源码：
若系统找不到该库，或某 feature/环境变量开启，就改为静态构建。
例如真实 [`libz-sys` crate] 会检查环境变量 `LIBZ_SYS_STATIC`
或 `static` feature，来决定是否从源码构建而非用系统库。
更完整例子见[源码][libz-source]。

[`libz-sys` crate]: https://crates.io/crates/libz-sys
[`pkg-config` crate]: https://crates.io/crates/pkg-config
[libz-source]: https://github.com/rust-lang/libz-sys

## Using another `sys` crate

使用 `links` 键时，crate 可以设置元数据供其依赖方读取。
这提供了 crate 间传递信息的机制。
这个例子里，我们将创建一个 C 库，并复用真实的 [`libz-sys` crate] 中的 zlib。

如果你的 C 库依赖 zlib，
可借助 [`libz-sys` crate] 自动查找或构建它。
这对跨平台支持很有帮助，例如 Windows 通常不预装 zlib。
`libz-sys` 会[设置 `include`
元数据](https://github.com/rust-lang/libz-sys/blob/3c594e677c79584500da673f918c4d2101ac97a1/build.rs#L156)，
告诉其他 package zlib 头文件位置。
我们的构建脚本可通过 `DEP_Z_INCLUDE` 环境变量读取。
示例：

```toml
# Cargo.toml

[package]
name = "z_user"
version = "0.1.0"
edition = "2024"

[dependencies]
libz-sys = "1.0.25"

[build-dependencies]
cc = "1.0.46"
```

这里引入 `libz-sys` 后，
最终库只会链接一个 `libz`，
并允许我们在构建脚本中获取它的信息：

```rust,ignore
// build.rs

fn main() {
    let mut cfg = cc::Build::new();
    cfg.file("src/z_user.c");
    if let Some(include) = std::env::var_os("DEP_Z_INCLUDE") {
        cfg.include(include);
    }
    cfg.compile("z_user");
    println!("cargo::rerun-if-changed=src/z_user.c");
}
```

在 `libz-sys` 处理完复杂工作后，
C 源码可直接 `#include` zlib 头文件，
即便系统原本未安装也能找到头文件。

```c
// src/z_user.c

#include "zlib.h"

// ... rest of code that makes use of zlib.
```

## Reading target configuration

当构建脚本需要依据目标平台做决策时，
应读取 `CARGO_CFG_*` 环境变量，
而不是使用 `cfg!` 或 `#[cfg]`。
原因是：构建脚本本身是在 *host* 机器上编译并运行的，
而 `CARGO_CFG_*` 反映的是 *target* 平台。
交叉编译时这一区别尤为关键。

```rust,ignore
// build.rs

fn main() {
    // reads the TARGET configuration
    let target_os = std::env::var("CARGO_CFG_TARGET_OS").unwrap();

    if target_os == "windows" {
        println!("cargo::rustc-link-lib=userenv");
    } else if target_os == "linux" {
        println!("cargo::rustc-link-lib=pthread");
    }
}
```

注意某些配置值可能包含逗号分隔的多个值
（例如 `CARGO_CFG_TARGET_FAMILY` 可能是 `unix,wasm`）。
检查这些值时要按多值情况处理。

若想用更易用的类型化 API，可考虑 [`build-rs`] crate，
它会帮你处理这些细节。

[`build-rs`]: https://crates.io/crates/build-rs

## Conditional compilation

构建脚本可输出 [`rustc-cfg` instructions]，
从而启用可在编译期检测的条件。
这个例子看看 [`openssl` crate] 如何借此支持多个 OpenSSL 版本。

[`openssl-sys` crate] 负责构建与链接 OpenSSL。
它支持多种实现（如 LibreSSL）和多个版本。
它也使用 `links` 键向其他构建脚本传递信息。
其中一个信息是 `version_number`，即探测到的 OpenSSL 版本。
相关代码大致[如下](https://github.com/sfackler/rust-openssl/blob/dc72a8e2c429e46c275e528b61a733a66e7877fc/openssl-sys/build/main.rs#L216)：

```rust,ignore
println!("cargo::metadata=version_number={openssl_version:x}");
```

这会让所有直接依赖 `openssl-sys` 的 crate
获得 `DEP_OPENSSL_VERSION_NUMBER` 环境变量。

提供上层接口的 `openssl` crate 依赖 `openssl-sys`。
它的构建脚本可通过 `DEP_OPENSSL_VERSION_NUMBER`
读取版本信息，并生成一些 [`cfg`
值](https://github.com/sfackler/rust-openssl/blob/dc72a8e2c429e46c275e528b61a733a66e7877fc/openssl/build.rs#L18-L36)：

```rust,ignore
// (portion of build.rs)

println!("cargo::rustc-check-cfg=cfg(ossl101,ossl102)");
println!("cargo::rustc-check-cfg=cfg(ossl110,ossl110g,ossl111)");

if let Ok(version) = env::var("DEP_OPENSSL_VERSION_NUMBER") {
    let version = u64::from_str_radix(&version, 16).unwrap();

    if version >= 0x1_00_01_00_0 {
        println!("cargo::rustc-cfg=ossl101");
    }
    if version >= 0x1_00_02_00_0 {
        println!("cargo::rustc-cfg=ossl102");
    }
    if version >= 0x1_01_00_00_0 {
        println!("cargo::rustc-cfg=ossl110");
    }
    if version >= 0x1_01_00_07_0 {
        println!("cargo::rustc-cfg=ossl110g");
    }
    if version >= 0x1_01_01_00_0 {
        println!("cargo::rustc-cfg=ossl111");
    }
}
```

这些 `cfg` 值随后可配合 [`cfg` attribute] 或 [`cfg`
macro] 做条件编译。
例如 SHA3 支持在 OpenSSL 1.1.1 才加入，
所以在更老版本里会被[条件排除](https://github.com/sfackler/rust-openssl/blob/dc72a8e2c429e46c275e528b61a733a66e7877fc/openssl/src/hash.rs#L67-L85)：

```rust,ignore
// (portion of openssl crate)

#[cfg(ossl111)]
pub fn sha3_224() -> MessageDigest {
    unsafe { MessageDigest(ffi::EVP_sha3_224()) }
}
```

当然，这种做法要谨慎。
它会让产物对构建环境更敏感。
例如二进制发布到另一台机器后，
那台机器未必有完全一致的共享库版本，从而可能引发问题。

[`cfg` attribute]: ../../reference/conditional-compilation.md#the-cfg-attribute
[`cfg` macro]: ../../std/macro.cfg.html
[`rustc-cfg` instructions]: build-scripts.md#rustc-cfg
[`openssl` crate]: https://crates.io/crates/openssl
[`openssl-sys` crate]: https://crates.io/crates/openssl-sys

[crates.io]: https://crates.io/
