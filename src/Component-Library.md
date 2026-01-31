# 组件库

<details>
<summary><strong>相关源文件</strong></summary>

* [TODO.md](https://github.com/tessera-ui/tessera/blob/821ebad7/TODO.md)
* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [example/src/example_components.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components.rs)
* [tessera-components/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/Cargo.toml)
* [tessera-components/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs)
</details>

本页面提供了由 `tessera-components` 包提供的内置组件库的参考指南。它涵盖了包结构、主题系统、组件组织以及渲染管线。关于特定组件类型的详细文档，请参阅子页面：

*   布局组件（行、列、可滚动区域等）：[布局组件](Layout-Components.md)
*   交互组件（按钮、复选框、滑块等）：[交互组件](Interactive-Components.md)
*   文本渲染与排版：[文本与排版](Text-and-Typography.md)
*   毛玻璃效果与模糊：[毛玻璃效果](Glass-Effects.md)
*   导航与模态框组件：[导航与模态框](Navigation-and-Modals.md)

关于创建自定义组件的信息，请参阅 [自定义组件与管线](Custom-Components-and-Pipelines.md)。

---

## 概览

`tessera-components` 包提供了一个基于 `tessera-ui` 框架构建的全面 Material Design 组件库。它实现了 Material Design 3 (M3) 规范，并包含 40 多个用于构建桌面和移动应用的生产级组件。

组件库作为 Cargo 工作区成员组织在 [tessera-components/](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/) 下，并导出一个 `ComponentsPackage` 类型，用于注册渲染管线和平台依赖。组件以被 `#[tessera]` 标注的函数形式暴露，可以从其他 `#[tessera]` 函数中调用。

**关键特性：**

*   符合 Material Design 3 标准。
*   集成具有动态配色方案的主题系统。
*   GPU 加速的渲染管线（文本、形状、图像、模糊效果）。
*   通过 `accesskit` 集成提供无障碍支持。
*   跨平台表情符号渲染（包含 Android 特定处理）。

---

## 包注册

要使用组件库，应用必须在入口点注册 `ComponentsPackage`。该包会初始化渲染管线和平台依赖。

**注册模式：**

```rust
use tessera_components::ComponentsPackage;

tessera_ui::entry!(
    app,
    packages = [ComponentsPackage::default()],
);
```

`ComponentsPackage` 执行两项关键注册：

1.  **平台包**：通过 `PlatformPackage` 注册平台特定工具。
2.  **渲染管线**：通过 `TesseraComponents` 模块注册所有组件管线。

**来源：** [tessera-components/src/lib.rs L138-L155](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs#L138-L155)

---

## Material 主题系统

`tessera-components` 中的所有组件都依赖于 `MaterialTheme` 上下文，该上下文必须在组件树的根部提供。主题系统包含符合 Material Design 3 的配色方案、排版和形状定义。

**主题提供：**

```rust
use tessera_components::theme::{MaterialTheme, material_theme};

material_theme(MaterialTheme::default, || {
    // 应用组件在此处
});
```

组件使用 `use_context::<MaterialTheme>()` 检索主题值。

---

## 组件组织

`tessera-components` 包将组件组织到逻辑模块中。每个模块导出组件函数、参数类型和默认常量。

**模块结构：**

| 类别 | 模块 | 关键组件 |
| --- | --- | --- |
| **布局** | `row`, `column`, `boxed`, `flow_row`, `scrollable` | `row()`, `column()`, `scrollable()` |
| **延迟布局** | `lazy_list`, `lazy_grid` | `lazy_column()`, `lazy_row()`, `lazy_grid()` |
| **交互** | `button`, `checkbox`, `switch`, `slider` | `button()`, `checkbox()`, `slider()` |
| **文本与输入** | `text`, `text_field`, `text_input` | `text()`, `text_input()` |
| **导航** | `navigation_bar`, `navigation_rail`, `tabs`, `scaffold` | `navigation_bar()`, `tabs()` |
| **反馈** | `snackbar`, `dialog`, `bottom_sheet`, `progress` | `snackbar_host()`, `dialog_provider()` |
| **展示** | `surface`, `icon`, `image`, `card`, `chip` | `surface()`, `icon()`, `card()` |
| **效果** | `fluid_glass`, `glass_button`, `glass_slider` | `fluid_glass()`, `glass_button()` |

**组件函数模式：**
所有组件都遵循一致的模式：

1.  **参数结构体**：类型化配置（如 `ButtonArgs`）。
2.  **组件函数**：接收参数和可选子闭包的 `#[tessera]` 函数。
3.  **默认值结构体**：用于 Material Design 规范的常量（如 `ButtonDefaults`）。
4.  **作用域 API**：适用于带有子节点的组件（如 `RowScope`）。

---

## 渲染管线系统

组件库注册了多个处理 GPU 加速绘制的渲染管线。

**内置管线：**

| 管线 | 绘制命令类型 | 用途 | 关键特性 |
| --- | --- | --- | --- |
| **ShapePipeline** | `ShapeCommand` | 基于 SDF 的形状渲染 | 圆角矩形、圆形、边框、阴影 |
| **GlyphonTextRender** | `TextCommand` | 文本渲染 | 字体管理、成型、表情支持 |
| **ImagePipeline** | `ImageCommand` | 位图图像显示 | 纹理采样、着色 |
| **FluidGlassPipeline** | `GlassCommand` | 模糊与折射效果 | 背景采样、双通道模糊 |
| **BlurPipeline** | `BlurCommand` | 基于计算的模糊 | 通过计算着色器实现高斯模糊 |

**来源：** [tessera-components/src/pipelines/text/pipeline.rs L1-L590](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/pipelines/text/pipeline.rs#L1-L590)
