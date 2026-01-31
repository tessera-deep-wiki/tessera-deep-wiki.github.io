# 交互组件

<details>
<summary><strong>相关源文件</strong></summary>

* [TODO.md](https://github.com/tessera-ui/tessera/blob/821ebad7/TODO.md)
* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [example/src/example_components.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components.rs)
* [tessera-components/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs)
</details>

本文档涵盖了由 `tessera-components` 包提供的交互式 UI 组件，这些组件用于处理用户输入并响应事件。这些组件包括按钮、复选框、单选按钮、开关和滑块。

关于布局和定位组件，请参阅 [布局组件](Layout-Components.md)。关于文本输入和编辑，请参阅 [文本与排版](Text-and-Typography.md)。关于交互组件的高级视觉效果，请参阅 [毛玻璃效果](Glass-Effects.md)。关于导航组件（如标签页），请参阅 [导航与模态框](Navigation-and-Modals.md)。

## 组件架构概览

Tessera UI 中的交互组件遵循围绕状态管理、事件处理和通过 `surface` 组件进行渲染构建的一致架构模式。每个组件都将视觉呈现、交互逻辑和状态同步进行分离。

交互组件使用 `remember()` 进行内部状态管理，并提供回调函数（如 `on_click`, `on_change`）用于将状态变更传达给父应用。大多数交互组件都构建在 `surface` 组件之上，该组件提供了波纹效果、形状样式和事件处理。

---

## 按钮组件

`tessera-components` 包为不同的交互模式提供了多种按钮变体。

### 标准按钮

`button` 组件提供具有波纹效果和 Material Design 样式的可点击表面。它是大多数用户触发操作的基础。

`ButtonArgs` 通过构建器模式支持全面自定义，包括：`filled`（填充型）、`tonal`（色调型）、`outlined`（轮廓型）、`elevated`（海拔型）和 `text`（文本型），以遵循 Material Design 3 的不同强调级别指南。

### 按钮组

`button_groups` 组件组织相关的按钮，并协调其选择状态和样式。支持“标准”样式（带间距）和“连接”样式（相邻）。

### 分段按钮

`segmented_button` 提供紧凑的单选或多选控件。它包含处理重叠和等宽约束的自定义布局逻辑。

### 拆分按钮

`split_button_layout` 将主要操作与次要操作（或菜单诱导）配对。主要操作和次要操作按钮具有互补的圆角形状。

---

## 复选框组件

`checkbox` 组件提供用于二元或三元选择的复选框。它包含平滑的勾选标志出现/消失动画，并支持通过 `on_change` 回调更新状态。

---

## 单选按钮组件

`radio_button` 提供用于组内单选的单选按钮。应用通过共享状态（如选择索引）来协调单选按钮组。

---

## 开关组件

`switch` 组件渲染一个用于布尔状态的切换控件，具有动画形式的滑块移动和轨道颜色转换。提供标准 `switch` 和带有模糊效果的 `glass_switch`。

---

## 滑块组件

`slider` 组件允许在归一化范围 [0.0, 1.0] 内进行连续值选择。它支持拖动交互、焦点管理，并提供实时数值反馈。

---

## 状态管理模式

交互组件遵循一致的模式来使用 Tessera 的 `remember()` 和 `State<T>` 系统进行状态管理。

### 与外部状态集成

交互组件通过回调函数与应用状态集成。应用通常使用 `remember()` 管理本地状态，或使用外部 `State<T>` 句柄管理共享状态。组件在内部维护自己的交互状态（如动画、焦点、拖动过程），同时通过回调更新外部状态。

---

## 事件处理架构

所有交互组件都使用 `input_handler()` 系统处理用户事件，遵循标准化的模式：
1.  **光标检查**：判断光标是否在组件边界内。
2.  **事件循环**：处理按下、释放和拖动逻辑。
3.  **数值更新**：对于滑块等组件计算并更新数值。
4.  **触发回调**：执行用户提供的回调函数。
5.  **状态更新**：更新组件内部的交互视觉状态。
