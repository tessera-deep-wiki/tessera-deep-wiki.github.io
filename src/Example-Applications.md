# 示例应用

<details>
<summary><strong>相关源文件</strong></summary>

* [example/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/example/Cargo.toml)
* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [example/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/lib.rs)
</details>

本文档描述了演示 Tessera UI 框架用法的示例应用包 (`example`)。该应用通过一个支持跨平台的交互式、可导航界面展示了所有可用组件。

关于单个组件的信息，请参阅 [组件库文档](Component-Library.md)。关于工作区组织，请参阅 [工作区结构](Workspace-Structure.md)。

## 库结构

`example` 包被组织为库和二进制文件，并针对平台特定代码使用了条件编译。

### Cargo 配置

该包使用双重构建目标配置：

| 类型 | 用途 | 路径 |
| --- | --- | --- |
| `rlib`, `cdylib` | 用于 Android 构建的库 | `src/lib.rs` |
| 二进制 | 桌面端可执行文件 | `src/main.rs` |

**来源：** [example/Cargo.toml L1-L30](https://github.com/tessera-ui/tessera/blob/821ebad7/example/Cargo.toml#L1-L30)

---

## 平台特定入口点

示例应用使用条件编译为桌面和 Android 平台提供不同的入口点。

### Android 平台

`android_main` 函数是 Android 构建的入口点：
*   使用 `#[unsafe(no_mangle)]` 修饰以确保 JNI 兼容性。
*   接收来自 Android activity 的 `AndroidApp` 参数。
*   初始化 `android_logger` 以集成系统日志。

### 桌面平台

`desktop_main` 函数是桌面平台（Windows, macOS, Linux）的入口点：
*   使用 `tracing_subscriber` 进行带有美化格式和跨度事件的日志记录。
*   调用不带平台特定参数的 `Renderer::run()`。

**来源：** [example/src/lib.rs L10-L51](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/lib.rs#L10-L51)

---

## 应用架构

应用使用基于路由的导航系统和分片状态管理，提供了一个多页面的组件展示。

### 根应用组件

应用组件层次结构包括：
*   **提供者栈**：侧边栏、底部工作表和对话框提供者。
*   **布局**：顶部应用栏、路由根节点和底部导航栏。
*   **导航目标**：`HomeDestination`（带有组件卡片的延迟列）和 `AboutDestination`。

**来源：** [example/src/app.rs L60-L181](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs#L60-L181)

---

## 状态管理模式

示例应用演示了使用分片系统的几种状态管理模式。

### 应用级状态

`app` 函数使用分片管理的 `AppState`，该状态持有提供者的状态（侧边栏、对话框等）。

### 首页状态

`home` 函数维护组件列表的状态以及单个卡片的波纹效果（ripple effects）状态。

### 顶部应用栏状态

`top_app_bar` 函数维护返回按钮的波纹状态。

**来源：** [example/src/app.rs L52-L58](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs#L52-L58)
