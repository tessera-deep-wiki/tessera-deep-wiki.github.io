# 导航与模态框

<details>
<summary><strong>相关源文件</strong></summary>

* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [example/src/example_components/tabs.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/tabs.rs)
* [example/src/example_components/menus.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/menus.rs)
* [example/src/example_components/snackbar.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/snackbar.rs)
</details>

本页面记录了 `tessera-components` 库提供的导航和模态叠加组件。这些组件实现了遵循 Material Design 3 规范的视图切换、上下文信息显示以及即时反馈功能。

关于使用路由系统的全局页面导航，请参阅 [底层接口](tessera-shard.md)。关于交互式表单组件，请参阅 [交互组件](Interactive-Components.md)。

---

## 组件类别

*   **导航组件**：`tabs`（标签页）、`navigation_bar`（底部导航栏）、`navigation_rail`（侧边导航轨）。用于在同一路由内的不同视图或章节间切换。
*   **模态叠加层**：`dialog`（对话框）、`bottom_sheet`（底部工作表）、`side_sheet`（侧边工作表）。用于显示需要用户响应或确认的聚焦内容。
*   **即时反馈**：`snackbar`（消息条）、`menus`（菜单）。用于显示简短消息、状态更新或上下文操作。

---

## 导航组件

### 标签页

提供页面内视图切换模式。支持“主要”和“次要”两种视觉变体，并管理标签之间的平滑指示器动画。

### 导航栏与导航轨

*   **底部导航栏** (`navigation_bar`)：为 3-5 个顶级目的地提供底部导航，通常用于移动端窄屏。
*   **侧边导航轨** (`navigation_rail`)：为 3-7 个目的地提供侧边导航，支持可选的展开状态，适用于宽屏。应用通常通过断点（breakpoint）在两者之间自动切换。

---

## 模态叠加系统

Tessera 中的模态叠加层采用 **提供者-控制器模式** (Provider-Controller Pattern)：
1.  **提供者组件** (Provider)：包装应用内容并管理叠加层的渲染层级。
2.  **控制器对象** (Controller)：提供 API（如 `open`, `close`）来控制显示状态。
3.  两者通过 `remember()` 连接，并通过组件树传递。

### 对话框

用于显示需要用户确认或输入的聚焦内容。支持预定义的 `basic_dialog`，包含标题、正文文本和操作按钮。

### 底部工作表

从屏幕底部滑出以显示补充内容。支持标准 Material 风格或带有背景遮罩的模态风格。

### 侧边工作表

从屏幕边缘（左侧或右侧）滑入。常用于设置面板、过滤器或不占据全屏的辅助导航。

---

## 消息条系统

`snackbar` 在屏幕底部显示简短、易逝的消息，可包含可选的操作按钮。与对话框不同，它们不会中断用户工作流。

**队列管理**：`SnackbarHostState` 维护一个先进先出 队列，当当前消息被关闭或超时（通常为 4-10 秒）后，会自动显示下一条消息。

---

## 菜单组件

菜单提供锚定在特定 UI 元素上的上下文操作。
*   **控制器 API**：通过 `open_at(anchor)` 在指定位置打开。
*   **布局**：支持多种放置方式（如 `BelowStart`, `AboveEnd` 等），并可设置偏移量。

---

## 响应式导航示例

在示例应用中，系统会测量父容器宽度：
*   当宽度小于 600dp 时，显示底部 `navigation_bar`。
*   当宽度大于 600dp 时，显示侧边 `navigation_rail`。
两者共享相同的导航操作逻辑，实现了无缝的跨设备体验。
