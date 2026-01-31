# tessera-components

<details>
<summary><strong>相关源文件</strong></summary>

* [TODO.md](https://github.com/tessera-ui/tessera/blob/821ebad7/TODO.md)
* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [example/src/example_components.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components.rs)
* [tessera-components/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/Cargo.toml)
* [tessera-components/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs)
* [tessera-components/src/pipelines/text/pipeline.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/pipelines/text/pipeline.rs)
</details>

本页面记录了 `tessera-components` 包，它是一个构建在 tessera-ui 框架之上的全面 Material Design 组件库。它包含遵循 Material Design 3 规范的布局原语、交互式控件、文本渲染、视觉效果和导航组件。

关于驱动这些组件的核心框架信息，请参阅 [tessera-ui 核心库](tessera-ui-Core.md)。关于在应用中使用这些组件的指南，请参阅 [组件库文档](Component-Library.md)。

---

## 概览

`tessera-components` 包作为 Tessera 应用的主要 UI 工具包，提供了实现 Material Design 3 (MD3) 模式的生产级组件。该包围绕三个核心职责构建：

1.  **组件实现**：按功能（布局、交互、导航等）组织的组合式 UI 组件集合。
2.  **渲染管线**：用于形状、文本、图像及高级视觉效果的 GPU 加速渲染系统。
3.  **主题系统**：Material Design 配色方案、排版比例和形状族。

该包旨在与 `ComponentsPackage` 注册系统配合使用，该系统会自动初始化所需的渲染管线和平台依赖。

**来源**：[tessera-components/src/lib.rs L1-L156](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs#L1-L156)

---

## 包架构

### 包注册系统

`ComponentsPackage` 实现了 `TesseraPackage` trait，为组件初始化提供统一的入口点。当通过 `tessera_ui::entry!` 注册时，它会：

1.  注册用于平台特定服务的 `PlatformPackage`。
2.  添加 `TesseraComponents` 渲染模块。
3.  通过 `register_pipelines()` 触发管线注册。

**来源**：[tessera-components/src/lib.rs L122-L156](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs#L122-L156)

### 渲染模块实现

`TesseraComponents` 结构体实现了 `RenderModule`，委托给 `pipelines::register_pipelines()` 来配置 GPU 渲染系统：

| 管线 | 类型 | 用途 |
| --- | --- | --- |
| `ShapePipeline` | 可绘制 | 带有抗锯齿的基于 SDF 的形状渲染 |
| `GlyphonTextRender` | 可绘制 | 通过 glyphon 字体系统的文本渲染 |
| `ImagePipeline` | 可绘制 | 纹理采样与展示 |
| `FluidGlassPipeline` | 可绘制 | 模糊与折射效果 |
| `BlurPipeline` | 可计算 | 双通道高斯模糊 |

每个管线都通过类型擦除分发注册到 `PipelineRegistry` 中。

---

## 内置渲染管线

### 文本渲染管线

文本渲染管线使用全局 `FONT_SYSTEM` 单例，以避免在每个组件上进行昂贵的字体数据库初始化。`GlyphonTextRender` 管线管理：

*   **字体图集**：用于字形光栅化的 GPU 纹理图集。
*   **Swash 缓存**：用于已成型字形的 CPU 端缓存。
*   **LRU 缓存**：存储由文本内容和样式参数索引的预成型 `TextData` 实例。

在 Android 上，管线包含特殊处理，将 COLRv1 表情符号字体替换为与 swash 光栅化兼容的捆绑 COLRv0 变体。

**来源**：[tessera-components/src/pipelines/text/pipeline.rs L17-L590](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/pipelines/text/pipeline.rs#L17-L590)

---

## Material Design 主题系统

### 主题上下文传播

主题系统使用框架的上下文机制在整个组件树中提供 Material Design 令牌。组件通过 `use_context::<MaterialTheme>()` 检索主题数据，并应用基于 Material Design 3 规范的颜色、排版和形状。

### 配色方案结构

Material Design 配色方案定义了语义化的颜色角色，如 `primary`（主色）、`secondary`（次色）、`surface`（表面色）等。组件根据其语义角色选择颜色，而不是硬编码特定的颜色值，从而确保应用各处的一致性。

---

## 组件状态管理模式

### 控制器模式

许多组件使用控制器模式管理复杂状态：

1.  **创建**：使用 `remember(Controller::new)`。
2.  **提供**：传递给 `*_with_controller()` 变体函数。
3.  **修改**：用户交互调用 `controller.with_mut(|c| c.method())`。
4.  **响应性**：当控制器状态改变时，组件自动重渲染。

**示例：Snackbar 状态管理**

`SnackbarHostState` 管理挂起的 snackbar 队列、计时和结果跟踪。宿主每帧轮询状态，以确定显示哪个 snackbar 以及何时根据超时设置将其关闭。

**来源**：[tessera-components/src/snackbar.rs L211-L307](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/snackbar.rs#L211-L307)
