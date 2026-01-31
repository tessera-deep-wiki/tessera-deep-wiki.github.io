# 布局组件

<details>
<summary><strong>相关源文件</strong></summary>

* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [example/src/example_components.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components.rs)
* [tessera-components/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/lib.rs)
</details>

本文档涵盖了由 `tessera-components` 包提供的基础布局组件。这些组件作为 UI 层次结构中排列子元素的构建块，处理空间定位、对齐、弹性空间分配、滚动以及虚拟化。

关于交互式 UI 元素（如按钮和滑块），请参阅 [交互组件](Interactive-Components.md)。关于文本显示与排版，请参阅 [文本与排版](Text-and-Typography.md)。关于毛玻璃视觉效果，请参阅 [毛玻璃效果](Glass-Effects.md)。关于导航与模态框组件，请参阅 [导航与模态框](Navigation-and-Modals.md)。

## 布局组件概览

`tessera-components` 中的布局系统提供了一套全面的组件，分为以下几类：

### 组件类别

*   **基础布局**：`row`（水平排列）、`column`（垂直堆叠）、`boxed`（层叠堆叠）、`flow_row` / `flow_column`（流式折行）。
*   **容器组件**：`surface`（视觉容器）、`scrollable`（溢出滚动）。
*   **虚拟化布局**：`lazy_list` (`lazy_column` / `lazy_row`)、`lazy_grid`（瓦片网格）、`lazy_staggered_grid`（瀑布流布局）。
*   **特殊用途**：`spacer`（空白间距）、`pager`（滑动页面）。

**基础布局** 提供排列子元素的根本模式。**容器组件** 管理视觉样式、滚动行为和裁剪。**虚拟化布局** 通过仅渲染可见项目来优化大数据集的性能。

---

## 基于约束的布局系统

布局组件使用核心 `tessera-ui` 框架中定义的基于约束的测量模型。父容器通过 `DimensionValue` 和 `Constraint` 类型向子元素传递尺寸约束，这些约束在测量阶段被解析。

### 尺寸值类型

每个维度可以指定为以下三种策略之一：
*   **Fixed**：精确的像素大小。
*   **Fill**：扩展以填充可用空间。
*   **Wrap**：根据内容调整大小。

### 对齐系统

布局组件使用两个正交的对齐系统：
*   **主轴对齐** (`MainAxisAlignment`)：控制主布局方向上的排列（如 `Start`, `Center`, `SpaceBetween` 等）。
*   **交叉轴对齐** (`CrossAxisAlignment`)：控制垂直于主轴方向的排列（如 `Start`, `Center`, `Stretch` 等）。

---

## 基础布局组件

### row 和 column

`row` 和 `column` 组件分别沿水平和垂直轴提供线性排列。两者都支持通过权重（weight）为子元素分配比例空间，并提供全面的对齐控制。

**用法示例：**

```rust
// 带有权重子元素的垂直列
column(ColumnArgs::default(), |scope| {
    scope.child(|| text("页眉"));
    scope.child_weighted(|| scrollable_content(), 1.0);
    scope.child(|| text("页脚"));
});

// 带对齐方式的水平行
row(
    RowArgs::default()
        .main_axis_alignment(MainAxisAlignment::SpaceBetween)
        .cross_axis_alignment(CrossAxisAlignment::Center),
    |scope| {
        scope.child(|| button("取消"));
        scope.child(|| button("确定"));
    }
);
```

### boxed

`boxed` 组件将子元素堆叠在同一个空间内。它会测量所有子元素，并将自身大小调整为最大尺寸。常用于分层背景、覆盖层或实现 Z 轴堆叠。

### flow_row 和 flow_column

当子元素超出主轴可用空间时，这些组件会自动将其折行或折列。适用于标签、纸片（chips）或任何需要自然换行的动态项目集合。

---

## 容器组件

### surface

`surface` 组件提供了一个视觉容器，具有可配置的背景、形状、海拔（高度感）、边框和交互状态。它是 Material Design 表面系统的基础。

### scrollable

`scrollable` 为大于其容器的内容提供滚动功能。它支持垂直和水平滚动，捕获滚轮和触控事件，并根据需要渲染滚动条。

---

## 工具组件

### spacer

`spacer` 组件创建一个空的、非交互的空间。它接受 `Modifier` 来定义尺寸。常用于按钮之间的间距、填补工具栏中未使用的空间或在布局中创建固定大小的缺口。
