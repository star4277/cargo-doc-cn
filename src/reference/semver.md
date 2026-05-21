# 语义版本兼容性

本章详细介绍了传统上被认为是
软件包新版本的兼容或破坏性 SemVer 更改。请参阅
[SemVer 兼容性] 部分详细了解 SemVer 是什么以及 Cargo 如何使用
使用它来确保库的兼容性。

这些只是*指导方针*，并不一定是所有人都必须遵循的硬性规则。
项目会服从。 [更改类别] 部分详细介绍了本指南如何
对变更的级别和严重性进行分类。本指南的大部分内容侧重于
会导致“cargo”和“rustc”无法构建某些内容的更改
以前工作过。几乎每一次改变都会带来一定的风险
对运行时行为产生负面影响，对于这些情况，它通常是
项目维护者的判断是否是一个
SemVer 不兼容的更改。

[Change categories]: #change-categories
[SemVer compatibility]: resolver.md#semver-compatibility

## 更改类别

下面列出的所有政策均按变化程度分类：

* **重大变更**：需要重大 SemVer 提升的变更。
* **较小的更改**：仅需要较小的 SemVer 提升的更改。
* **可能发生重大变化**：一些项目可能认为重大的变化
  和其他人认为次要的。

“可能造成破坏”类别涵盖有“潜力”的变化
更新期间中断，但不一定会导致损坏。影响
应仔细考虑这些变化。确切的性质将取决于
关于项目维护者的变更和原则。

有些项目可能会选择仅在较小的更改上增加补丁号。它
鼓励遵循 SemVer 规范，并且仅在补丁中应用错误修复
发布。但是，错误修复可能需要 API 更改，该更改被标记为
“微小的改变”，并且不应该影响兼容性。本指南不需
应该如何对待每个人的“微小变化”的立场
次要更改和补丁更改之间的区别是取决于
变化的性质。

有些更改被标记为“次要”，即使它们具有潜在风险
破坏构建。这是针对潜力极其巨大的情况
低，并且潜在的破坏代码不太可能用惯用语编写
Rust 或特别不鼓励使用。

本指南使用术语“主要”和“次要”，假设这与
“1.0.0”版本或更高版本。以“0.y.z”开头的初始开发版本
可以将“y”中的更改视为主要版本，将“z”中的更改视为次要版本。
“0.0.z”版本始终是重大更改。这是因为 Cargo 使用
约定仅最左边的非零分量发生变化
被认为不兼容。

* API兼容性
    * 项目
        * [主要：重命名/移动/删除任何公共项目](#item-remove)
        * [次要：添加新的公共项目](#item-new)
    * 类型
        * [主要：更改定义良好的类型的对齐方式、布局或大小](#type-layout)
    * 结构体
        * [主要：当当前所有字段都是公共字段时添加私有结构字段](#struct-add-private-field-when-public)
        * [主要：当不存在私有字段时添加公共字段](#struct-add-public-field-when-no-private)
        * [次要：当至少一个已经存在时添加或删除私有字段](#struct-private-fields-with-private)
        * [次要：从具有所有私有字段（至少一个字段）的元组结构到普通结构，反之亦然](#struct-tuple-normal-with-private)
    * 枚举
        * [主要：添加新的枚举变体（没有 `non_exhaustive`）](#enum-variant-new)
        * [主要：向枚举变体添加新字段](#enum-fields-new)
    * 特质
        * [主要：添加非默认特征项](#trait-new-item-no-default)
        * [主要：对特质项目签名的任何更改](#trait-item-signature)
        * [可能破坏：添加默认特征项](#trait-new-default-item)
        * [主要：添加一个特征项，使特征成为非对象安全](#trait-object-safety)
        * [主要：添加没有默认值的类型参数](#trait-new-parameter-no-default)
        * [次要：添加默认的特征类型参数](#trait-new-parameter-default)
    * 实施
        * [可能的重大更改：添加任何固有项目](#impl-item-new)
    * 泛型
        * [主要：收紧通用边界](#generic-bounds-tighten)
        * [次要：放宽通用边界](#generic-bounds-loose)
        * [次要：添加默认类型参数](#generic-new-default)
        * [次要：泛化类型以使用泛型（具有相同类型）](#generic-generalize-identical)
        * [主要：泛化类型以使用泛型（可能具有不同的类型）](#generic-generalize- different)
        * [次要：将泛型类型更改为更泛型的类型](#generic-more-generic)
        * [主要：在RPIT中捕获更多通用参数](#generic-rpit-capture)
    * 功能
        * [主要：添加/删除函数参数](#fn-change-arity)
        * [可能破坏：引入新的函数类型参数](#fn-generic-new)
        * [次要：泛化函数以使用泛型（支持原始类型）](#fn-generalize-兼容)
        * [主要：泛化一个函数以使用类型不匹配的泛型](#fn-generalize-mismatch)
        * [次要：使“不安全”函数变得安全](#fn-unsafe-safe)
    * 属性
        * [主要：从`no_std`支持切换到需要`std`](#attr-no-std-to-std)
        * [主要：将 `non_exhaustive` 添加到现有的没有私有字段的枚举、变体或结构中](#attr-adding-non-exhaustive)
* Tooling and environment compatibility
    * [可能破坏：更改所需的 Rust 最低版本](#env-new-rust)
    * [可能破坏：改变平台和环境要求](#env-change-requirements)
    * [次要：引入新的 l​​ints](#new-lints)
    * Cargo
        * [次要：添加新的 Cargo 功能](#cargo-feature-add)
        * [主要：删除 Cargo 功能](#cargo-feature-remove)
        * [主要：如果更改功能或公共项目，则从功能列表中删除功能](#cargo-feature-remove-another)
        * [可能破坏：删除可选依赖项](#cargo-remove-opt-dep)
        * [次要：改变依赖特性](#cargo-change-dep-feature)
        * [次要：添加依赖项](#cargo-dep-add)
* [应用程序兼容性](#application-compatibility)

## API 兼容性

下面所有的例子都包含三部分：原始代码、代码
修改后，以及可能出现的代码示例用法
在另一个项目中。经过一个小改动，示例用法应该成功
使用之前和之后的版本进行构建。

### 主要：重命名/移动/删除任何公共项目{#item-remove}

缺乏公开暴露的[物品][物品]将导致对该物品的任何使用
编译失败。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
// ... item has been removed

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    updated_crate::foo(); // Error: cannot find function `foo`
}
```

这包括添加任何类型的 [`cfg` 属性]，它可以更改
项目或行为基于[条件编译]可用。

缓解策略：
* 将要删除的项目标记为[已弃用]，然后稍后删除它们
  破坏 SemVer 的版本中的日期。
* 将重命名的项目标记为[已弃用]，并使用[`pub use`]项目重新导出
  到旧名字。

### 次要：添加新的公共项目 {#item-new}

添加新的公共[项目]是一个微小的变化。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
// ... absence of item

///////////////////////////////////////////////////////////
// After
pub fn foo() {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
// `foo` is not used since it didn't previously exist.
```

请注意，在极少数情况下，这可能是由于 glob 导致的 **重大更改**
进口。例如，如果您添加一个新特征，并且项目已使用 glob
import 将该特征带入范围，并且新特征引入了
与它所实现的任何类型冲突的关联项，这可以
由于歧义而导致编译时错误。例子：

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
// ... absence of trait

///////////////////////////////////////////////////////////
// After
pub trait NewTrait {
    fn foo(&self) {}
}

impl NewTrait for i32 {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::*;

pub trait LocalTrait {
    fn foo(&self) {}
}

impl LocalTrait for i32 {}

fn main() {
    123i32.foo(); // Error:  multiple applicable items in scope
}
```

这不被认为是重大变化，因为传统上全局导入是
已知的向前兼容性危险。从外部全局导入项目
应避免使用 Crate。

### 主要：更改明确定义的类型 {#type-layout} 的对齐方式、布局或大小

更改以前定义良好的类型的对齐方式、布局或大小是一项重大更改。

一般来说，使用[默认表示]的类型没有明确定义的对齐方式、布局或大小。
编译器可以自由更改对齐方式、布局或大小，因此代码不应对此做出任何假设。

> **注意**：如果外部包对类型的对齐、布局或大小做出假设，即使没有明确定义，外部包也可能会损坏。
> 这不被视为 SemVer 重大更改，因为不应做出这些假设。

一些不属于重大更改的更改示例如下（假设不违反本指南中的其他规则）：

* 添加、删除、重新排序或更改默认表示结构、联合或枚举的字段，使更改遵循本指南中的其他规则（例如，使用“non_exhaustive”允许这些更改，或对已经私有的私有字段进行更改）。
  请参阅[struct-add-private-field-when-public](#struct-add-private-field-when-public)、[struct-add-public-field-when-no-private](#struct-add-public-field-when-no-private)、[struct-private-fields-with-private](#struct-private-fields-with-private)、[enum-fields-new](#enum-fields-new)。
* 如果枚举使用“non_exhaustive”，则将变体添加到默认表示枚举中。
  这可能会改变枚举的对齐方式或大小，但这些并没有明确定义。
  请参阅[enum-variant-new](#enum-variant-new)。
* 添加、删除、重新排序或更改 `repr(C)` 结构、联合或枚举的私有字段，遵循本指南中的其他规则（例如，使用 `non_exhaustive`，或在其他私有字段已存在时添加私有字段）。
  请参阅[repr-c-private-change](#repr-c-private-change)。
* 如果枚举使用“non_exhaustive”，则向“repr(C)”枚举添加变体。
  请参阅[repr-c-enum-variant-new](#repr-c-enum-variant-new)。
* 将 `repr(C)` 添加到默认表示结构、联合或枚举。
  请参阅[repr-c-add](#repr-c-add)。
* 将 `repr(<int>)` [原始表示] 添加到枚举中。
  请参阅[repr-int-enum-add](#repr-int-enum-add)。
* 将 `repr(transparent)` 添加到默认表示结构或枚举。
  请参阅[repr-transparent-add](#repr-transparent-add)。

使用 [`repr` 属性] 的类型可以说具有以某种方式定义的对齐和布局，代码可能会做出一些假设，因为更改该类型可能会破坏这种假设。

在某些情况下，具有“repr”属性的类型可能没有明确定义的对齐方式、布局或大小。
在这些情况下，更改类型可能是安全的，但应小心谨慎。
例如，具有私有字段且未以其他方式记录其对齐、布局或大小保证的类型不能被外部包所依赖，因为公共 API 没有完全定义类型的对齐、布局或大小。

具有*私有*字段的类型被明确定义的一个常见示例是具有通用类型的单个私有字段的类型，使用“repr(transparent)”，
文档的散文讨论了它对于泛型类型是透明的。
例如，请参阅[`UnsafeCell`]。

重大变更的一些示例包括：

* 将 `repr(packed)` 添加到结构或联合。
  请参阅[repr-packed-add](#repr-packed-add)。
* 将 `repr(align)` 添加到结构、联合或枚举。
  请参阅[repr-align-add](#repr-align-add)。
* 从结构或联合中删除 `repr(packed)`。
  请参阅[repr-packed-remove](#repr-packed-remove)。
* 如果改变对齐方式或布局，则更改 `repr(packed(N))` 的值 N。
  请参阅[repr-packed-n-change](#repr-packed-n-change)。
* 如果改变了对齐方式，则更改 `repr(align(N))` 的值 N。
  请参阅[repr-align-n-change](#repr-align-n-change)。
* 从结构、联合或枚举中删除 `repr(align)`。
  请参阅[repr-align-remove](#repr-align-remove)。
* 更改 `repr(C)` 类型的公共字段的顺序。
  请参阅[repr-c-shuffle](#repr-c-shuffle)。
* 从结构、联合或枚举中删除 `repr(C)`。
  请参阅[repr-c-remove](#repr-c-remove)。
* 从枚举中删除 `repr(<int>)`。
  请参阅[repr-int-enum-remove](#repr-int-enum-remove)。
* 更改 `repr(<int>)` 枚举的原始表示。
  请参阅[repr-int-enum-change](#repr-int-enum-change)。
* 从结构或枚举中删除 `repr(transparent)`。
  请参阅[repr-transparent-remove](#repr-transparent-remove)。

[the default representation]: ../../reference/type-layout.html#the-default-representation
[primitive representation]: ../../reference/type-layout.html#primitive-representations
[`repr` attribute]: ../../reference/type-layout.html#representations
[`std::mem::transmute`]: ../../std/mem/fn.transmute.html
[`UnsafeCell`]: ../../std/cell/struct.UnsafeCell.html#memory-layout

#### 次要： `repr(C)` 添加、删除或更改私有字段 {#repr-c-private-change}

添加、删除或更改 `repr(C)` 结构、联合或枚举的私有字段通常是安全的，假设它遵循本指南中的其他准则（请参阅 [struct-add-private-field-when-public](#struct-add-private-field-when-public)、[struct-add-public-field-when-no-private](#struct-add-public-field-when-no-private)， [struct-private-fields-with-private](#struct-private-fields-with-private)、[enum-fields-new](#enum-fields-new))。

例如，只有在已经存在其他私有字段或者是“non_exhaustive”的情况下才可以添加私有字段。
如果有私有字段，则可以添加公共字段，或者它是“non_exhaustive”，并且添加不会改变其他字段的布局。

但是，这可能会改变类型的大小和对齐方式。
如果尺寸或对齐方式发生变化，则应小心。
代码不应假设具有私有字段或“non_exhaustive”的类型的大小或对齐方式，除非它有记录的大小或对齐方式。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
#[repr(C)]
pub struct Example {
    pub f1: i32,
    f2: i32, // a private field
}

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
#[repr(C)]
pub struct Example {
    pub f1: i32,
    f2: i32,
    f3: i32, // a new field
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    // NOTE: Users should not make assumptions about the size or alignment
    // since they are not documented.
    let f = updated_crate::Example::default();
}
```

#### 次要： `repr(C)` 添加枚举变量 {#repr-c-enum-variant-new}

如果枚举使用“non_exhaustive”，则向“repr(C)”枚举添加变体通常是安全的。
有关更多讨论，请参阅 [enum-variant-new](#enum-variant-new)。

请注意，这可能是一个重大更改，因为它改变了类型的大小和对齐方式。
有关类似问题，请参阅 [repr-c-private-change](#repr-c-private-change)。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(C)]
#[non_exhaustive]
pub enum Example {
    Variant1 { f1: i16 },
    Variant2 { f1: i32 },
}

///////////////////////////////////////////////////////////
// After
#[repr(C)]
#[non_exhaustive]
pub enum Example {
    Variant1 { f1: i16 },
    Variant2 { f1: i32 },
    Variant3 { f1: i64 }, // added
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    // NOTE: Users should not make assumptions about the size or alignment
    // since they are not specified. For example, this raised the size from 8
    // to 16 bytes.
    let f = updated_crate::Example::Variant2 { f1: 123 };
}
```

#### 次要：将 `repr(C)` 添加到默认表示 {#repr-c-add}

使用 [默认表示] 将 `repr(C)` 添加到结构、联合或枚举是安全的。
这是安全的，因为用户不应该对具有默认表示的类型的对齐、布局或大小做出假设。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Example {
    pub f1: i32,
    pub f2: i16,
}

///////////////////////////////////////////////////////////
// After
#[repr(C)] // added
pub struct Example {
    pub f1: i32,
    pub f2: i16,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    let f = updated_crate::Example { f1: 123, f2: 456 };
}
```

#### 次要：将 `repr(<int>)` 添加到枚举 {#repr-int-enum-add}

将 `repr(<int>)` [原始表示] 添加到具有 [默认表示] 的枚举是安全的。
这是安全的，因为用户不应该对具有默认表示形式的枚举的对齐、布局或大小做出假设。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub enum E {
    Variant1,
    Variant2(i32),
    Variant3 { f1: f64 },
}

///////////////////////////////////////////////////////////
// After
#[repr(i32)] // added
pub enum E {
    Variant1,
    Variant2(i32),
    Variant3 { f1: f64 },
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    let x = updated_crate::E::Variant3 { f1: 1.23 };
}
```

#### 次要：将 `repr(transparent)` 添加到默认表示结构或枚举 {#repr-transparent-add}

使用 [默认表示] 将 `repr(transparent)` 添加到结构或枚举是安全的。
这是安全的，因为用户不应该使用默认表示来假设结构体或枚举的对齐、布局或大小。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Example<T>(T);

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
#[repr(transparent)] // added
pub struct Example<T>(T);

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    let x = updated_crate::Example::<i32>::default();
}
```

#### 主要：将 `repr(packed)` 添加到结构或联合 {#repr-packed-add}

将“repr(packed)”添加到结构或联合是一项重大更改。
对类型“repr(packed)”进行的更改可能会破坏代码，例如无法获取对字段的引用，或者导致不相交闭包捕获的截断。

<!-- TODO：如果所有字段都是私有的，这样做应该安全吗？ -->

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Example {
    pub f1: u8,
    pub f2: u16,
}

///////////////////////////////////////////////////////////
// After
#[repr(packed)] // added
pub struct Example {
    pub f1: u8,
    pub f2: u16,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let f = updated_crate::Example { f1: 1, f2: 2 };
    let x = &f.f2; // Error: error[E0793]: reference to field of packed struct is unaligned
}
```

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Example(pub i32, pub i32);

///////////////////////////////////////////////////////////
// After
#[repr(packed)]
pub struct Example(pub i32, pub i32);

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let mut f = updated_crate::Example(123, 456);
    let c = || {
        // Without repr(packed), the closure precisely captures `&f.0`.
        // With repr(packed), the closure captures `&f` to avoid undefined behavior.
        let a = f.0;
    };
    f.1 = 789; // Error: cannot assign to `f.1` because it is borrowed
    c();
}
```

#### 主要：将 `repr(align)` 添加到结构、联合或枚举 {#repr-align-add}

将“repr(align)”添加到结构、联合或枚举是一项重大更改。
创建类型“repr(align)”会破坏在“repr(packed)”类型中对该类型的任何使用，因为不允许该组合。

<!-- TODO: 这看起来应该是非常罕见的。是否应该为此制定任何例外？ -->

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Aligned {
    pub a: i32,
}

///////////////////////////////////////////////////////////
// After
#[repr(align(8))] // added
pub struct Aligned {
    pub a: i32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Aligned;

#[repr(packed)]
pub struct Packed { // Error: packed type cannot transitively contain a `#[repr(align)]` type
    f1: Aligned,
}

fn main() {
    let p = Packed {
        f1: Aligned { a: 123 },
    };
}
```

#### 主要：从结构或联合中删除 `repr(packed)` {#repr-packed-remove}

从结构或联合中删除“repr(packed)”是一项重大更改。
这可能会改变外部 Crate 所依赖的对齐或布局。

如果任何字段是公共的，那么删除“repr(packed)”可能会改变不相交闭包捕获工作的方式。
在某些情况下，这可能会导致代码损坏，类似于[版本指南][版本闭包]中概述的内容。

[edition-closures]: ../../edition-guide/rust-2021/disjoint-capture-in-closures.html

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(C, packed)]
pub struct Packed {
    pub a: u8,
    pub b: u16,
}

///////////////////////////////////////////////////////////
// After
#[repr(C)] // removed packed
pub struct Packed {
    pub a: u8,
    pub b: u16,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Packed;

fn main() {
    let p = Packed { a: 1, b: 2 };
    // Some assumption about the size of the type.
    // Without `packed`, this fails since the size is 4.
    const _: () = assert!(std::mem::size_of::<Packed>() == 3); // Error: assertion failed
}
```

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(C, packed)]
pub struct Packed {
    pub a: *mut i32,
    pub b: i32,
}
unsafe impl Send for Packed {}

///////////////////////////////////////////////////////////
// After
#[repr(C)] // removed packed
pub struct Packed {
    pub a: *mut i32,
    pub b: i32,
}
unsafe impl Send for Packed {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Packed;

fn main() {
    let mut x = 123;

    let p = Packed {
        a: &mut x as *mut i32,
        b: 456,
    };

    // When the structure was packed, the closure captures `p` which is Send.
    // When `packed` is removed, this ends up capturing `p.a` which is not Send.
    std::thread::spawn(move || unsafe {
        *(p.a) += 1; // Error: cannot be sent between threads safely
    });
}
```

#### 主要：如果改变对齐或布局，则更改`repr(packed(N))`的值N {#repr-packed-n-change}

如果改变对齐或布局，则更改“repr(packed(N))”的 N 值是一项重大更改。
这可能会改变外部 Crate 所依赖的对齐或布局。

如果值“N”低于公共字段的对齐方式，那么这将破坏任何尝试引用该字段的代码。

请注意，对“N”的某些更改可能不会更改对齐方式或布局，例如，当当前值已经等于类型的自然对齐方式时增加它。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(packed(4))]
pub struct Packed {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// After
#[repr(packed(2))] // changed to 2
pub struct Packed {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Packed;

fn main() {
    let p = Packed { a: 1, b: 2 };
    let x = &p.b; // Error: error[E0793]: reference to field of packed struct is unaligned
}
```

#### 主要：如果改变对齐方式，则更改`repr(align(N))`的值N {#repr-align-n-change}

如果改变了对齐方式，则更改 repr(align(N)) 的值 N 是一项重大更改。
这可能会改变外部 Crate 所依赖的对齐方式。

如果类型没有像[类型布局](#type-layout)中讨论的那样明确定义（例如具有任何私有字段以及具有未记录的对齐或布局），则进行此更改应该是安全的。

请注意，对“N”的某些更改可能不会更改对齐方式或布局，例如，当当前值已经等于或小于类型的自然对齐方式时减少它。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(align(8))]
pub struct Packed {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// After
#[repr(align(4))] // changed to 4
pub struct Packed {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Packed;

fn main() {
    let p = Packed { a: 1, b: 2 };
    // Some assumption about the size of the type.
    // The alignment has changed from 8 to 4.
    const _: () = assert!(std::mem::align_of::<Packed>() == 8); // Error: assertion failed
}
```

#### 主要：从结构、联合或枚举中删除 `repr(align)` {#repr-align-remove}

如果结构、联合或枚举的布局定义良好，则从结构、联合或枚举中删除“repr(align)”是一项重大更改。
这可能会改变外部 Crate 所依赖的对齐或布局。

如果类型没有像[类型布局](#type-layout)中讨论的那样明确定义（例如具有任何私有字段和具有未记录的对齐方式），则进行此更改应该是安全的。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(C, align(8))]
pub struct Packed {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// After
#[repr(C)] // removed align
pub struct Packed {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Packed;

fn main() {
    let p = Packed { a: 1, b: 2 };
    // Some assumption about the size of the type.
    // The alignment has changed from 8 to 4.
    const _: () = assert!(std::mem::align_of::<Packed>() == 8); // Error: assertion failed
}
```

#### 主要：更改 `repr(C)` 类型的公共字段的顺序 {#repr-c-shuffle}

更改“repr(C)”类型的公共字段的顺序是一项重大更改。
外部 Crate 可能依赖于字段的特定顺序。

```rust,ignore,run-fail
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(C)]
pub struct SpecificLayout {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// After
#[repr(C)]
pub struct SpecificLayout {
    pub b: u32, // changed order
    pub a: u8,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::SpecificLayout;

unsafe extern "C" {
    // This C function is assuming a specific layout defined in a C header.
    fn c_fn_get_b(x: &SpecificLayout) -> u32;
}

fn main() {
    let p = SpecificLayout { a: 1, b: 2 };
    unsafe { assert_eq!(c_fn_get_b(&p), 2) } // Error: value not equal to 2
}

# mod cdep {
#     // This simulates what would normally be something included from a build script.
#     // This definition would be in a C header.
#     #[repr(C)]
#     pub struct SpecificLayout {
#         pub a: u8,
#         pub b: u32,
#     }
#
#     #[no_mangle]
#     pub fn c_fn_get_b(x: &SpecificLayout) -> u32 {
#         x.b
#     }
# }
```

#### 主要：从结构、联合或枚举中删除 `repr(C)` {#repr-c-remove}

从结构、联合或枚举中删除“repr(C)”是一项重大更改。
外部 Crate 可能依赖于类型的具体布局。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(C)]
pub struct SpecificLayout {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// After
// removed repr(C)
pub struct SpecificLayout {
    pub a: u8,
    pub b: u32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::SpecificLayout;

unsafe extern "C" {
    // This C function is assuming a specific layout defined in a C header.
    fn c_fn_get_b(x: &SpecificLayout) -> u32; // Error: is not FFI-safe
}

fn main() {
    let p = SpecificLayout { a: 1, b: 2 };
    unsafe { assert_eq!(c_fn_get_b(&p), 2) }
}

# mod cdep {
#     // This simulates what would normally be something included from a build script.
#     // This definition would be in a C header.
#     #[repr(C)]
#     pub struct SpecificLayout {
#         pub a: u8,
#         pub b: u32,
#     }
#
#     #[no_mangle]
#     pub fn c_fn_get_b(x: &SpecificLayout) -> u32 {
#         x.b
#     }
# }
```

#### 主要：从枚举 {#repr-int-enum-remove} 中删除 `repr(<int>)`

从枚举中删除 `repr(<int>)` 是一项重大更改。
外部 Crate 可能假设判别式是特定大小。
例如，枚举的 [`std::mem::transmute`] 可能会失败。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(u16)]
pub enum Example {
    Variant1,
    Variant2,
    Variant3,
}

///////////////////////////////////////////////////////////
// After
// removed repr(u16)
pub enum Example {
    Variant1,
    Variant2,
    Variant3,
}

///////////////////////////////////////////////////////////
// Example usage that will break.

fn main() {
    let e = updated_crate::Example::Variant2;
    let i: u16 = unsafe { std::mem::transmute(e) }; // Error: cannot transmute between types of different sizes
}
```

#### 主要：更改 `repr(<int>)` 枚举的原始表示形式 {#repr-int-enum-change}

更改 `repr(<int>)` 枚举的原始表示是一个重大更改。
外部 Crate 可能假设判别式是特定大小。
例如，枚举的 [`std::mem::transmute`] 可能会失败。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(u16)]
pub enum Example {
    Variant1,
    Variant2,
    Variant3,
}

///////////////////////////////////////////////////////////
// After
#[repr(u8)] // changed repr size
pub enum Example {
    Variant1,
    Variant2,
    Variant3,
}

///////////////////////////////////////////////////////////
// Example usage that will break.

fn main() {
    let e = updated_crate::Example::Variant2;
    let i: u16 = unsafe { std::mem::transmute(e) }; // Error: cannot transmute between types of different sizes
}
```

#### 主要：从结构或枚举中删除 `repr(transparent)` {#repr-transparent-remove}

从结构或枚举中删除“repr(transparent)”是一项重大更改。
外部 Crate 可能依赖于具有透明字段的对齐、布局或大小的类型。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[repr(transparent)]
pub struct Transparent<T>(T);

///////////////////////////////////////////////////////////
// After
// removed repr
pub struct Transparent<T>(T);

///////////////////////////////////////////////////////////
// Example usage that will break.
#![deny(improper_ctypes)]
use updated_crate::Transparent;

unsafe extern "C" {
    fn c_fn() -> Transparent<f64>; // Error: is not FFI-safe
}

fn main() {}
```

### 主要：当所有当前字段都是公共字段时添加私有结构字段 {#struct-add-private-field-when-public}

当将私有字段添加到先前具有所有公共字段的结构时，
这将破坏任何尝试使用[结构文字]构造它的代码。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo {
    pub f1: i32,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo {
    pub f1: i32,
    f2: i32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let x = updated_crate::Foo { f1: 123 }; // Error: cannot construct `Foo`
}
```

缓解策略：
* 不要向全公共字段结构添加新字段。
* 首次引入时将结构体标记为 [`#[non_exhaustive]`][non_exhaustive]
  一个结构体，以防止用户使用结构体文字语法，而是
  提供构造函数方法和/或[默认]实现。

### 主要：当不存在私有字段时添加公共字段 {#struct-add-public-field-when-no-private}

当公共字段添加到具有所有公共字段的结构体时，这将
破坏任何尝试使用[结构文字]构造它的代码。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo {
    pub f1: i32,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo {
    pub f1: i32,
    pub f2: i32,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let x = updated_crate::Foo { f1: 123 }; // Error: missing field `f2`
}
```

缓解策略：
* 不要向全公共字段结构添加新字段。
* 首次引入时将结构体标记为 [`#[non_exhaustive]`][non_exhaustive]
  一个结构体，以防止用户使用结构体文字语法，而是
  提供构造函数方法和/或[默认]实现。

### 次要：当至少一个已存在时添加或删除私有字段 {#struct-private-fields-with-private}

当结构体中添加或删除私有字段时，这是安全的
已经拥有至少一个私有字段。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo {
    f1: i32,
}

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo {
    f2: f64,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    // Cannot access private fields.
    let x = updated_crate::Foo::default();
}
```

这是安全的，因为现有代码无法使用 [structliteral] 来构造
它，也没有详尽地匹配其内容。

请注意，对于元组结构，如果元组包含，则这是一个**重大更改**
公共字段，并且添加或删除私有字段会更改
任何公共字段的索引。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo(pub i32, i32);

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo(f64, pub i32, i32);

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let x = updated_crate::Foo::default();
    let y = x.0; // Error: is private
}
```

### 次要：从包含所有私有字段（至少有一个字段）的元组结构到普通结构，反之亦然 {#struct-tuple-normal-with-private}

如果满足以下条件，将元组结构更改为普通结构（反之亦然）是安全的
字段是私有的。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo(i32);

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo {
    f1: i32,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
fn main() {
    // Cannot access private fields.
    let x = updated_crate::Foo::default();
}
```

这是安全的，因为现有代码无法使用 [structliteral] 来构造
它，也不匹配其内容。

### 主要：添加新的枚举变体（不带 `non_exhaustive`）{#enum-variant-new}

如果枚举不使用
[`#[non_exhaustive]`][non_exhaustive] 属性。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub enum E {
    Variant1,
}

///////////////////////////////////////////////////////////
// After
pub enum E {
    Variant1,
    Variant2,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    use updated_crate::E;
    let x = E::Variant1;
    match x { // Error: `E::Variant2` not covered
        E::Variant1 => {}
    }
}
```

缓解策略：
* 引入enum时，标记为[`#[non_exhaustive]`][non_exhaustive]
  强制用户使用[通配符模式]来捕获新变体。

### 主要：向枚举变量添加新字段 {#enum-fields-new}

将新字段添加到枚举变体是一个重大更改，因为所有
字段是公共的，构造函数和匹配将无法编译。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub enum E {
    Variant1 { f1: i32 },
}

///////////////////////////////////////////////////////////
// After
pub enum E {
    Variant1 { f1: i32, f2: i32 },
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    use updated_crate::E;
    let x = E::Variant1 { f1: 1 }; // Error: missing f2
    match x {
        E::Variant1 { f1 } => {} // Error: missing f2
    }
}
```

缓解策略：
* 引入枚举时，将变体标记为[`non_exhaustive`][non_exhaustive]
  以便在没有通配符的情况下无法构造或匹配它。
  ```rust,ignore,skip
  pub enum E {
      #[non_exhaustive]
      Variant1{f1: i32}
  }
  ```
* 引入enum时，使用显式结构体作为值，在这里你可以
  控制现场可见性。
  ```rust,ignore,skip
  pub struct Foo {
     f1: i32,
     f2: i32,
  }
  pub enum E {
      Variant1(Foo)
  }
  ```

### 主要：添加非默认特征项 {#trait-new-item-no-default}

将非默认项添加到特征中是一项重大更改。这将
破坏该特征的任何实现者。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    fn foo(&self);
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}  // Error: not all trait items implemented
```

缓解策略：
* 始终为新的关联特征提供默认实现或值
  项目。
* 引入特质时，使用【密封特质】技巧来防止
  箱外的用户无法实现该特征。

### 主要：对特征项签名的任何更改{#trait-item-signature}

对特征项签名进行任何更改都是重大更改。这个可以
打破该特征的外部实现者。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {
    fn f(&self, x: i32) {}
}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    // For sealed traits or normal functions, this would be a minor change
    // because generalizing with generics strictly expands the possible uses.
    // But in this case, trait implementations must use the same signature.
    fn f<V>(&self, x: V) {}
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {
    fn f(&self, x: i32) {}  // Error: trait declaration has 1 type parameter
}
```

缓解策略：
* 引入具有默认实现的新项目以覆盖新的
  功能而不是修改现有项目。
* 引入特质时，使用【密封特质】技巧来防止
  箱外的用户无法实现该特征。

### 可能破坏：添加默认特征项 {#trait-new-default-item}

添加默认特征项通常是安全的。然而，这有时可以
导致编译错误。例如，如果
另一个特征中存在同名的方法。

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    fn foo(&self) {}
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

trait LocalTrait {
    fn foo(&self) {}
}

impl Trait for Foo {}
impl LocalTrait for Foo {}

fn main() {
    let x = Foo;
    x.foo(); // Error: multiple applicable items in scope
}
```

请注意，对于 [固有的] 上的名称冲突，*不*存在这种歧义。
实现]，因为它们优先于特征项。

请参阅 [trait-object-safety](#trait-object-safety) 了解要考虑的特殊情况
添加特质物品时。

缓解策略：
* 一些项目可能认为这种破损是可以接受的，特别是如果新的
  项目名称不太可能与任何现有代码冲突。选择名字
  小心以帮助避免这些碰撞。另外，也许可以接受
  要求下游用户添加[消歧语法]来选择
  更新依赖项时的正确功能。

### 主要：添加一个特征项，使特征非对象安全{#trait-object-safety}

添加一个特质项目是一项重大更改，该项目将特质更改为不存在
[对象安全]。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {
    // An associated const makes the trait not object-safe.
    const CONST: i32 = 123;
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}

fn main() {
    let obj: Box<dyn Trait> = Box::new(Foo); // Error: the trait `updated_crate::Trait` is not dyn compatible
}
```

做相反的事情是安全的（将非对象安全特征变成安全特征）
一）。

### 主要：添加没有默认值的类型参数 {#trait-new-parameter-no-default}

在特征中添加没有默认值的类型参数是一项重大更改。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait<T> {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}  // Error: missing generics
```

缓解策略：
* 请参阅[添加默认特征类型参数](#trait-new-parameter-default)。

### 次要：添加默认特征类型参数 {#trait-new-parameter-default}

只要特征有默认值，就可以安全地将类型参数添加到特征中。
外部实现者将使用默认值，无需指定
范围。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}

///////////////////////////////////////////////////////////
// After
pub trait Trait<T = i32> {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Trait;
struct Foo;

impl Trait for Foo {}
```

### 可能的重大更改：添加任何固有项目 {#impl-item-new}

通常向实现添加固有项目应该是安全的，因为
固有项目优先于特质项目。然而，在某些情况下
如果名称与已实现的特征相同，则冲突可能会导致问题
具有不同签名的项目。

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub struct Foo;

///////////////////////////////////////////////////////////
// After
pub struct Foo;

impl Foo {
    pub fn foo(&self) {}
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Foo;

trait Trait {
    fn foo(&self, x: i32) {}
}

impl Trait for Foo {}

fn main() {
    let x = Foo;
    x.foo(1); // Error: this method takes 0 arguments but 1 argument was supplied
}
```

请注意，如果签名匹配，则不会出现编译时错误，
但运行时行为可能会发生无声变化（因为它现在正在执行
不同的功能）。

缓解策略：
* 一些项目可能认为这种破损是可以接受的，特别是如果新的
  项目名称不太可能与任何现有代码冲突。选择名字
  小心以帮助避免这些碰撞。另外，也许可以接受
  要求下游用户添加[消歧语法]来选择
  更新依赖项时的正确功能。

### 主要：收紧通用边界{#generic-bounds-tighten}

收紧类型的通用界限是一项重大更改，因为这可以
打破用户对更宽松界限的期待。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<A> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo<A: Eq> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Foo;

fn main() {
    let s = Foo { f1: 1.23 }; // Error: the trait bound `{float}: Eq` is not satisfied
}
```

### 次要：放宽通用边界{#generic-bounds-loose}

放宽类型的通用界限是安全的，因为它只扩展了类型的范围
允许。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<A: Clone> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// After
pub struct Foo<A> {
    pub f1: A,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s = Foo { f1: 123 };
}
```

### 次要：添加默认类型参数 {#generic-new-default}

只要类型有默认值，向类型添加类型参数就是安全的。全部
现有引用将使用默认值，无需指定
范围。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
#[derive(Default)]
pub struct Foo {}

///////////////////////////////////////////////////////////
// After
#[derive(Default)]
pub struct Foo<A = i32> {
    f1: A,
}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s: Foo = Default::default();
}
```

### 次要：泛化类型以使用泛型（具有相同类型）{#generic-generalize-identical}

结构体或枚举字段可以从具体类型更改为泛型类型
参数，前提是更改会导致所有参数的类型相同
现有用例。例如，允许进行以下更改：

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo(pub u8);

///////////////////////////////////////////////////////////
// After
pub struct Foo<T = u8>(pub T);

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s: Foo = Foo(123);
}
```

因为 `Foo` 的现有用法是 `Foo<u8>` 的简写，它会产生
相同的字段类型。

### 主要：泛化类型以使用泛型（可能具有不同的类型）{#generic-generalize- Different}

将结构体或枚举字段从具体类型更改为泛型类型
如果类型可以更改，则参数可能会中断。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<T = u8>(pub T, pub u8);

///////////////////////////////////////////////////////////
// After
pub struct Foo<T = u8>(pub T, pub T);

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::Foo;

fn main() {
    let s: Foo<f32> = Foo(3.14, 123); // Error: mismatched types
}
```

### 次要：将泛型类型更改为更泛型类型 {#generic-more-generic}

将泛型类型更改为更泛型的类型是安全的。例如，
下面添加了一个默认为原始类型的泛型参数，即
是安全的，因为所有现有用户都将使用相同的类型
字段，默认参数不需要指定。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo<T>(pub T, pub T);

///////////////////////////////////////////////////////////
// After
pub struct Foo<T, U = T>(pub T, pub U);

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::Foo;

fn main() {
    let s: Foo<f32> = Foo(1.0, 2.0);
}
```

### 主要：在 RPIT 中捕获更多通用参数 {#generic-rpit-capture}

在 [RPIT]（返回位置 impl 特征）中捕获其他通用参数是一项重大更改。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn f<'a, 'b>(x: &'a str, y: &'b str) -> impl Iterator<Item = char> + use<'a> {
    x.chars()
}

///////////////////////////////////////////////////////////
// After
pub fn f<'a, 'b>(x: &'a str, y: &'b str) -> impl Iterator<Item = char> + use<'a, 'b> {
    x.chars().chain(y.chars())
}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    let a = String::new();
    let b = String::new();
    let iter = updated_crate::f(&a, &b);
    drop(b); // Error: cannot move out of `b` because it is borrowed
}
```

将通用参数添加到 RPIT 对如何使用结果类型施加了额外的限制。

请注意，当未指定“use<>”语法时，会存在隐式捕获。在 Rust 2021 及更早版本中，仅当生命周期参数在语法上出现在 RPIT 类型签名的边界内时才会被捕获。从 Rust 2024 开始，所有生命周期参数都会被无条件捕获。这意味着从 Rust 2024 开始，默认是最大兼容的，要求你在想要捕获 less 时明确，这是 SemVer 的承诺。

有关 RPIT 捕获的更多信息，请参阅[版本指南][rpit-capture-guide]和[参考][rpit-reference]。

这是一个小改动，可以在 RPIT 中捕获更少的通用参数。

> 注意：所有作用域内类型和 const 泛型参数必须隐式捕获（未指定 `+ use<...>`）或显式捕获（必须在 `+ use<...>` 中列出），因此目前不允许更改此类泛型的捕获内容。

[RPIT]: ../../reference/types/impl-trait.md#abstract-return-types
[rpit-capture-guide]: ../../edition-guide/rust-2024/rpit-lifetime-capture.html
[rpit-reference]: ../../reference/types/impl-trait.md#capturing

### 主要：添加/删除函数参数 {#fn-change-arity}

改变函数的数量是一个重大改变。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
pub fn foo(x: i32) {}

///////////////////////////////////////////////////////////
// Example usage that will break.
fn main() {
    updated_crate::foo(); // Error: this function takes 1 argument
}
```

缓解策略：
* 引入具有新签名的新函数，并且可能
  [弃用][弃用]旧的。
* 引入采用结构体参数的函数，结构体是在其中构建的
  与构建器模式。这允许将新字段添加到结构中
  将来。

### 可能具有破坏性：引入新的函数类型参数 {#fn-generic-new}

通常，添加非默认类型参数是安全的，但在某些情况下
在某些情况下，这可能是一个重大变化：

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub fn foo<T>() {}

///////////////////////////////////////////////////////////
// After
pub fn foo<T, U>() {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::foo;

fn main() {
    foo::<u8>(); // Error: function takes 2 generic arguments but 1 generic argument was supplied
}
```

然而，这种显式调用非常罕见（通常可以写成
其他方式）这种破损通常是可以接受的。一个人应该考虑到
考虑有问题的函数被调用的可能性有多大
显式类型参数。

### 次要：泛化函数以使用泛型（支持原始类型）{#fn-generalize-兼容}

函数参数的类型或其返回值可以是
*通用化*使用泛型，包括引入新的类型参数，
只要可以实例化为原始类型即可。例如，
允许进行以下更改：

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo(x: u8) -> u8 {
    x
}
pub fn bar<T: Iterator<Item = u8>>(t: T) {}

///////////////////////////////////////////////////////////
// After
use std::ops::Add;
pub fn foo<T: Add>(x: T) -> T {
    x
}
pub fn bar<T: IntoIterator<Item = u8>>(t: T) {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::{bar, foo};

fn main() {
    foo(1);
    bar(vec![1, 2, 3].into_iter());
}
```

因为所有现有的用途都是新签名的实例。

也许有些令人惊讶的是，泛化适用于特征对象，如
好吧，考虑到每个特质都会实现自身：

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub trait Trait {}
pub fn foo(t: &dyn Trait) {}

///////////////////////////////////////////////////////////
// After
pub trait Trait {}
pub fn foo<T: Trait + ?Sized>(t: &T) {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.
use updated_crate::{foo, Trait};

struct Foo;
impl Trait for Foo {}

fn main() {
    let obj = Foo;
    foo(&obj);
}
```

（使用“？Sized”是必不可少的；否则你无法恢复原始的
签名。）

以这种方式引入泛型可能会创建类型推断
失败。这些通常很少见，对于某些人来说可能是可以接受的破损
项目，因为这可以通过附加类型注释来修复。

```rust,ignore
// Breaking change example

///////////////////////////////////////////////////////////
// Before
pub fn foo() -> i32 {
    0
}

///////////////////////////////////////////////////////////
// After
pub fn foo<T: Default>() -> T {
    Default::default()
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::foo;

fn main() {
    let x = foo(); // Error: type annotations needed
}
```

### 主要：泛化函数以使用类型不匹配的泛型 {#fn-generalize-mismatch}

更改函数参数或返回类型是一个重大更改，如果
泛型类型限制或更改以前允许的类型。例如，
下面添加了一个现有的可能无法满足的通用约束
代码：

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo(x: Vec<u8>) {}

///////////////////////////////////////////////////////////
// After
pub fn foo<T: Copy + IntoIterator<Item = u8>>(x: T) {}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::foo;

fn main() {
    foo(vec![1, 2, 3]); // Error: `Copy` is not implemented for `Vec<u8>`
}
```

### 次要：使“不安全”函数安全 {#fn-unsafe-safe}

以前“不安全”的函数可以在不破坏代码的情况下变得安全。

但请注意，它可能会导致 [`unused_unsafe`][unused_unsafe] lint
如下例所示触发，这将导致本地 crates
指定 `#![deny(warnings)]` 来停止编译。每[介绍新的
lints](#new-lints)，允许更新引入新的警告。

反其道而行之（使安全函数变得“不安全”）是一个重大改变。

```rust,ignore
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub unsafe fn foo() {}

///////////////////////////////////////////////////////////
// After
pub fn foo() {}

///////////////////////////////////////////////////////////
// Example use of the library that will trigger a lint.
use updated_crate::foo;

unsafe fn bar(f: unsafe fn()) {
    f()
}

fn main() {
    unsafe { foo() }; // The `unused_unsafe` lint will trigger here
    unsafe { bar(foo) };
}
```

在结构/枚举上创建以前“不安全”的关联函数或方法
safe 也是一个小改动，而关联的情况则不然
特征上的函数（请参阅[特征项签名的任何更改](#trait-item-signature)）。

### 主要：从 `no_std` 支持切换到需要 `std` {#attr-no-std-to-std}

如果您的库特别支持 [`no_std`] 环境，那么它是一个
进行重大更改以制作需要“std”的新版本。

```rust,ignore,skip
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
#![no_std]
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
pub fn foo() {
    std::time::SystemTime::now();
}

///////////////////////////////////////////////////////////
// Example usage that will break.
// This will fail to link for no_std targets because they don't have a `std` crate.
#![no_std]
use updated_crate::foo;

fn example() {
    foo();
}
```

缓解策略：
* 避免这种情况的常见习惯用法是包含一个 `std` [Cargo feature]
  可选择启用“std”支持，当该功能关闭时，库
  可以在“no_std”环境中使用。

### 主要：将 `non_exhaustive` 添加到没有私有字段的现有枚举、变体或结构中 {#attr-adding-non-exhaustive}

制作物品[`#[non_exhaustive]`][non_exhaustive]会改变它们的方式
在定义它们的 Crate 外部使用：

- 无法构造非详尽的结构和枚举变体
  使用[结构文字]语法，包括[功能更新语法]。
- 非详尽结构上的模式匹配需要 `..` 和
  枚举上的匹配并不意味着详尽。
- 不允许使用“as”将枚举变体转换为其判别式。

无法使用 [structliteral] 语法构造具有私有字段的结构
无论是否使用 [`#[non_exhaustive]`][non_exhaustive]。
将 [`#[non_exhaustive]`][non_exhaustive] 添加到这样的结构中不是
一个突破性的改变。

```rust,ignore
// MAJOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub struct Foo {
    pub bar: usize,
}

pub enum Bar {
    X,
    Y(usize),
    Z { a: usize },
}

pub enum Quux {
    Var,
}

///////////////////////////////////////////////////////////
// After
#[non_exhaustive]
pub struct Foo {
    pub bar: usize,
}

pub enum Bar {
    #[non_exhaustive]
    X,

    #[non_exhaustive]
    Y(usize),

    #[non_exhaustive]
    Z { a: usize },
}

#[non_exhaustive]
pub enum Quux {
    Var,
}

///////////////////////////////////////////////////////////
// Example usage that will break.
use updated_crate::{Bar, Foo, Quux};

fn main() {
    let foo = Foo { bar: 0 }; // Error: cannot create non-exhaustive struct using struct expression

    let bar_x = Bar::X; // Error: unit variant `X` is private
    let bar_y = Bar::Y(0); // Error: tuple variant `Y` is private
    let bar_z = Bar::Z { a: 0 }; // Error: cannot create non-exhaustive variant using struct expression

    let q = Quux::Var;
    match q {
        Quux::Var => 0,
        // Error: non-exhaustive patterns: `_` not covered
    };
}
```

缓解策略：
* 将结构体、枚举和枚举变体标记为
  [`#[non_exhaustive]`][non_exhaustive] 第一次介绍它们时，
  而不是稍后添加 [`#[non_exhaustive]`][non_exhaustive]。

## 工具和环境兼容性

### 可能破坏：更改所需的 Rust 最低版本 {#env-new-rust}

在 Rust 新版本中引入新功能的使用可能会破坏
使用旧版本 Rust 的项目。这还包括使用新的
Cargo 新版本中的功能，并且需要使用仅限夜间的
功能位于以前在稳定版上运行的 Crate 中。

通常建议将此视为较小的更改，而不是视为
由于[各种原因][msrv-is-minor]，这是一个重大变化。它
通常相对容易更新到较新版本的 Rust。铁锈也有
6周的快速发布周期，并且一些项目将提供兼容性
在发布窗口内（例如当前的稳定版本加上 N
以前的版本）。请记住，一些大型项目可能无法
快速更新他们的 Rust 工具链。

缓解策略：
* 通过设置记录你的包的最低支持的 Rust 版本
  [`package.rust-version`]，允许 Cargo 的依赖解析
  需要时尝试[选择软件包的旧版本]。
  这样做时请务必考虑[支持期望]。
* 使用[Cargo features] 来选择加入新功能。
* 为旧版本提供广泛的支持。
* 如果可能的话，复制新标准库项目的源代码，以便您
  可以继续使用旧版本但利用新功能。
* 提供可以接收向后移植的较旧次要版本的单独分支
  重要的错误修复。
* 留意 [`[cfg(version(..))]`][cfg-version] 和
  [`#[cfg(accessible(..))]`][cfg-accessible] 提供选择加入的功能
  新功能的机制。这些目前不稳定，仅可用
  在夜间频道中。

[select older versions of your package]: resolver.md#rust-version
[support expectations]: rust-version.md#support-expectations

### 可能会造成破坏：更改平台和环境要求 {#env-change-requirements}

图书馆对图书馆做出了各种各样的假设
运行环境，例如主机平台、操作系统
版本、可用服务、文件系统支持等。它可能是一个突破性的
如果您发布的新版本限制了以前支持的内容，则进行更改，
例如需要更新版本的操作系统。这些变化
可能很难跟踪，因为您可能并不总是知道更改是否会发生
未自动测试的环境。

有些项目可能认为这种破损是可以接受的，特别是如果破损
对于大多数用户来说不太可能，或者项目没有资源
支持所有环境。另一个值得注意的情况是当供应商
停止对某些硬件或操作系统的支持，该项目可能会认为
合理地也停止支持。

缓解策略：
* 记录您特别支持的平台和环境。
* 在 CI 的各种环境中测试您的代码。

### 次要：引入新的 l​​ints {#new-lints}

对库的某些更改可能会导致该库的用户触发新的 lint。
这通常应被视为兼容的更改。

```rust,ignore,dont-deny
// MINOR CHANGE

///////////////////////////////////////////////////////////
// Before
pub fn foo() {}

///////////////////////////////////////////////////////////
// After
#[deprecated]
pub fn foo() {}

///////////////////////////////////////////////////////////
// Example use of the library that will safely work.

fn main() {
    updated_crate::foo(); // Warning: use of deprecated function
}
```

请注意，如果他们明确拒绝警告，并且更新的 Crate 是直接依赖项，那么从技术上讲，这可能会导致项目失败。
拒绝警告时应小心谨慎，并了解随着时间的推移可能会引入新的 l​​int。
但是，库作者应谨慎引入新警告，并可能需要考虑对其用户的潜在影响。

以下 lint 是更新依赖项时可能引入的示例：

* [`deprecated`][deprecated-lint] --- 当依赖项将 [`#[deprecated]` 属性][deprecated] 添加到您正在使用的项目时引入。
* [`unused_must_use`] --- 当依赖项将 [`#[must_use]` 属性][must-use-attr] 添加到您不使用结果的项目时引入。
* [`unused_unsafe`] --- 当依赖项从函数中*删除*`unsafe` 限定符时引入，并且这是在不安全块中调用的唯一不安全函数。

此外，将 rustc 更新到新版本可能会引入新的 l​​int。

引入新 lint 的传递依赖项通常不会导致失败，因为 Cargo 使用 [`--cap-lints`](../../rustc/lints/levels.html#capping-lints) 来抑制依赖项中的所有 lints。

缓解策略：
* 如果您构建时警告被拒绝，请了解您可能需要在更新依赖项时解决新警告。
  如果使用 RUSTFLAGS 传递 `-Dwarnings`，还要添加 `-A` 标志以允许可能导致问题的 lint，例如 `-Adeprecated`。
* 在[功能][Cargo features]背后引入弃用。
  例如 `#[cfg_attr(feature = "deprecated", deprecated="use bar instead")]`。
  然后，当您计划在未来的 SemVer 重大更改中删除某个项目时，您可以与用户沟通，告知他们应该在更新之前启用“已弃用”功能，以删除已弃用项目的使用。
  这允许用户选择何时响应弃用，而无需立即响应。
  缺点是，可能很难向用户传达他们需要采取这些手动步骤来准备重大更新的信息。

[`unused_must_use`]: ../../rustc/lints/listing/warn-by-default.html#unused-must-use
[deprecated-lint]: ../../rustc/lints/listing/warn-by-default.html#deprecated
[must-use-attr]: ../../reference/attributes/diagnostics.html#the-must_use-attribute
[`unused_unsafe`]: ../../rustc/lints/listing/warn-by-default.html#unused-unsafe

### Cargo

#### 次要：添加新的 Cargo 功能 {#cargo-feature-add}

添加新的[Cargo features]通常是安全的。如果该功能引入新的
导致重大变更的变更，这可能会给项目带来困难
具有更严格的向后兼容性需求。在这种情况下，请避免
将功能添加到“默认”列表中，并可能记录
启用该功能的后果。

```toml
# MINOR CHANGE

###########################################################
# Before
[features]
# ..empty

###########################################################
# After
[features]
std = []
```

#### 主要：删除 Cargo 功能 {#cargo-feature-remove}

删除 [Cargo features] 通常是一个重大更改。这会导致
任何启用该功能的项目都会出现错误。

```toml
# MAJOR CHANGE

###########################################################
# Before
[features]
logging = []

###########################################################
# After
[dependencies]
# ..logging removed
```

缓解策略：
* 清楚地记录您的功能。如果有内部或实验性的
  功能，对其进行标记，以便用户了解该功能的状态。
* 将旧功能保留在“Cargo.toml”中，但删除它
  功能。记录该功能已弃用，并将其删除
  未来主要 SemVer 版本。

#### 主要：从功能列表中删除某个功能（如果该功能更改了功能或公共项目）{#cargo-feature-remove-another}

如果从另一个功能中删除一个功能，这可能会破坏现有用户，如果
他们期望通过该功能来提供该功能。

```toml
# Breaking change example

###########################################################
# Before
[features]
default = ["std"]
std = []

###########################################################
# After
[features]
default = []  # This may cause packages to fail if they are expecting std to be enabled.
std = []
```

#### 可能破坏：删除可选依赖项 {#cargo-remove-opt-dep}

删除 [可选依赖项][opt-dep] 可能会破坏使用您的库的项目，因为
另一个项目可能通过 [Cargo features] 启用该依赖关系。

当存在可选依赖项时，cargo 隐式定义了以下功能
相同的名称提供一种机制来启用依赖关系并检查
当它被启用时。这个问题可以通过使用 `dep:` 语法来避免
`[features]` 表，它禁用此隐式功能。使用“dep:”
使得可以在 more 下隐藏可选依赖项的存在
可以更安全地修改语义相关的名称。

```toml
# Breaking change example

###########################################################
# Before
[dependencies]
curl = { version = "0.4.31", optional = true }

###########################################################
# After
[dependencies]
# ..curl removed
```

```toml
# MINOR CHANGE
#
# This example shows how to avoid breaking changes with optional dependencies.

###########################################################
# Before
[dependencies]
curl = { version = "0.4.31", optional = true }

[features]
networking = ["dep:curl"]

###########################################################
# After
[dependencies]
# Here, one optional dependency was replaced with another.
hyper = { version = "0.14.27", optional = true }

[features]
networking = ["dep:hyper"]
```

缓解策略：
* 在`[features]`表中使用`dep:`语法以避免暴露可选
  首先，依赖关系。请参阅[可选依赖项][opt-dep]
  更多信息。
* 清楚地记录您的功能。如果不包含可选依赖项
  在记录的功能列表中，那么您可以决定认为它是安全的
  更改未记录的条目。
* 保留可选的依赖项，并且不要在您的库中使用它。
* 将可选依赖项替换为不执行任何操作的 [Cargo feature]，
  并记录它已被弃用。
* 使用启用可选依赖项的高级功能，并记录
  这些是启用扩展功能的首选方式。为了
  例如，如果您的库有类似的可选支持
  “网络”，创建一个通用功能名称“网络”，使
  实现“网络”所需的可选依赖项。然后记录
  “网络”功能。

[opt-dep]: features.md#optional-dependencies

#### 次要：更改依赖项功能 {#cargo-change-dep-feature}

更改依赖项的功能通常是安全的，只要
功能不会引入重大更改。

```toml
# MINOR CHANGE

###########################################################
# Before
[dependencies]
rand = { version = "0.7.3", features = ["small_rng"] }


###########################################################
# After
[dependencies]
rand = "0.7.3"
```

#### 次要：添加依赖项 {#cargo-dep-add}

添加新的依赖项通常是安全的，只要新的依赖项
不会引入导致重大变更的新要求。
例如，在项目中添加需要 nightly 的新依赖项
以前在稳定版上运行的功能是一个重大变化。

```toml
# MINOR CHANGE

###########################################################
# Before
[dependencies]
# ..empty

###########################################################
# After
[dependencies]
log = "0.4.11"
```

## 应用程序兼容性

Cargo 项目还可能包括可执行的二进制文件，它们有自己的
接口（例如 CLI 接口、操作系统级交互等）。由于这些
是 Cargo 包的一部分，它们经常使用和共享相同的版本
包裹。您需要决定是否以及如何使用 SemVer
在对应用程序进行更改时与您的用户签订合同。这
对应用程序的潜在破坏和兼容更改太多
列出，因此鼓励您使用 [SemVer] 规范的精神来指导
您关于如何将版本控制应用于您的应用程序的决定，或者至少
记录您的承诺。

[`cfg` attribute]: ../../reference/conditional-compilation.md#the-cfg-attribute
[`no_std`]: ../../reference/names/preludes.html#the-no_std-attribute
[`package.rust-version`]: rust-version.md
[`pub use`]: ../../reference/items/use-declarations.html
[Cargo feature]: features.md
[Cargo features]: features.md
[cfg-accessible]: https://github.com/rust-lang/rust/issues/64797
[cfg-version]: https://github.com/rust-lang/rust/issues/64796
[conditional compilation]: ../../reference/conditional-compilation.md
[Default]: ../../std/default/trait.Default.html
[deprecated]: ../../reference/attributes/diagnostics.html#the-deprecated-attribute
[disambiguation syntax]: ../../reference/expressions/call-expr.html#disambiguating-function-calls
[functional update syntax]: ../../reference/expressions/struct-expr.html#functional-update-syntax
[inherent implementations]: ../../reference/items/implementations.html#inherent-implementations
[items]: ../../reference/items.html
[non_exhaustive]: ../../reference/attributes/type_system.html#the-non_exhaustive-attribute
[object safe]: ../../reference/items/traits.html#object-safety
[rust-feature]: https://doc.rust-lang.org/nightly/unstable-book/
[sealed trait]: https://rust-lang.github.io/api-guidelines/future-proofing.html#sealed-traits-protect-against-downstream-implementations-c-sealed
[SemVer]: https://semver.org/
[struct literal]: ../../reference/expressions/struct-expr.html
[wildcard patterns]: ../../reference/patterns.html#wildcard-pattern
[unused_unsafe]: ../../rustc/lints/listing/warn-by-default.html#unused-unsafe
[msrv-is-minor]: https://github.com/rust-lang/api-guidelines/discussions/231
