# 快速开始

<details>
<summary><strong>相关源文件</strong></summary>

* [README.md](https://github.com/tessera-ui/tessera/blob/821ebad7/README.md)
* [cargo-tessera/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/Cargo.toml)
* [cargo-tessera/src/commands/new.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/new.rs)
</details>

本快速入门指南将引导你完成 Tessera 的安装、创建首个项目、编写基础组件并运行开发服务器。在 10 分钟内，你就能在桌面上运行一个 Tessera 应用。

关于开发工作流和针对不同平台的构建详情，请参阅 [开发工作流](Development-Workflow.md) 和 [平台特定开发](Platform-Specific-Development.md)。

## 前置要求

在开始之前，请确保已安装 Rust 工具链。Tessera 需要 Rust 1.75 或更高版本。

**验证安装：**

```bash
rustc --version
cargo --version
```

**平台特定依赖项：**

| 平台 | 必需包 |
| --- | --- |
| Linux (X11) | `libx11-dev libxrandr-dev libxcursor-dev` |
| Linux (Wayland) | `libwayland-dev libxkbcommon-dev` |
| macOS | Xcode 命令行工具 |
| Windows | 无 (所有依赖项由 Rust 工具链提供) |

---

## 安装 cargo-tessera

`cargo-tessera` CLI 工具提供了项目脚手架生成、热重载开发和构建自动化功能。

```bash
cargo install cargo-tessera
```

---

## 创建首个项目

使用 `cargo tessera new` 命令生成新项目。CLI 会提供交互式提示：

```bash
cargo tessera new my-app
cd my-app
```

**模板选择：**

| 模板 | 描述 | 用途 |
| --- | --- | --- |
| `basic` | Hello World 入门示例 | 学习框架 |
| `blank` | 空白画布 | 从零开始构建 |

---

## 编写首个组件

Tessera 组件是被 `#[tessera]` 宏标注的函数。组件通过调用其他组件来构建 UI 树。

### 基础组件示例

`basic` 模板会创建一个简单的 Hello World 应用：

```rust
use tessera_components::{
    surface::{SurfaceArgs, surface},
    text::text,
    theme::{MaterialTheme, material_theme},
};
use tessera_ui::{Modifier, tessera};

#[tessera]
pub fn app() {
    material_theme(MaterialTheme::default, || {
        surface(
            SurfaceArgs::default().modifier(Modifier::new().fill_max_size()),
            || {
                text("你好，Tessera！");
            },
        );
    });
}
```

### 带有状态的组件

要创建一个交互式计数器应用，使用 `remember()` 函数来维持状态：

```rust
#[tessera]
fn app() {
    let count = remember(|| 0);
    column(ColumnArgs::default(), move |scope| {
        scope.child(move || {
            button(
                ButtonArgs::filled(move || count.with_mut(|c| *c += 1)),
                || text("+"),
            )
        });
        scope.child(move || text(format!("计数: {}", count.get())));
    });
}
```

---

## 运行开发服务器

`cargo tessera dev` 命令启动一个热重载开发服务器，它会监听文件变更并自动重建并重启应用。

```bash
cargo tessera dev
```

开发服务器会监听：
*   `src/` 目录（所有源文件）。
*   `Cargo.toml`（依赖项变更）。
*   `build.rs`（构建脚本变更）。

使用 `Ctrl+C` 停止开发服务器。

---

## 构建生产版本

准备好创建生产版本时，使用 `cargo tessera build`：

```bash
cargo tessera build --release
```

生成的二进制文件位于 `target/release/` 目录下。针对 Windows 的跨平台编译示例：

```bash
cargo tessera build --release --target x86_64-pc-windows-msvc
```
