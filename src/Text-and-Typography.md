# 文本与排版

<details>
<summary><strong>相关源文件</strong></summary>

* [tessera-components/src/pipelines/text/pipeline.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/src/pipelines/text/pipeline.rs)
* [tessera-components/assets/NotoColorEmoji_COLRv0.LICENSE](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/assets/NotoColorEmoji_COLRv0.LICENSE)
* [tessera-components/assets/NotoColorEmoji_COLRv0.ttf](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-components/assets/NotoColorEmoji_COLRv0.ttf)
</details>

本文档描述了 Tessera 中的文本渲染系统，包括基于 glyphon 的渲染管线、字体管理、文本布局与测量、缓存策略，以及通过 Material Design 主题系统进行的排版配置。

关于交互式文本输入组件，请参阅 [交互组件](Interactive-Components.md)。关于通用布局约束和尺寸，请参阅 [布局系统](Layout-System.md)。

---

## 概览

Tessera 的文本渲染系统构建在 [glyphon](https://crates.io/crates/glyphon) 库之上，该库提供了字体管理、文本成型（shaping）和 GPU 加速的光栅化。系统架构由三个主要子系统组成：

1.  **字体系统**：在所有文本组件之间共享的全局静态字体数据库。
2.  **文本管线**：通过 GPU 渲染文本的渲染管线实现。
3.  **文本数据与缓存**：具有测量 API 且经过 LRU 缓存的文本布局。

该系统处理复杂的排版特性，包括字体回退（fallback）、多行布局、Unicode 成型以及平台特定的表情符号渲染。

---

## 字体系统

### 全局字体数据库

字体系统使用静态单例跨所有渲染操作共享字体数据。这避免了每个组件重复进行昂贵的初始化。

### 平台特定字体加载

*   **桌面端**：自动使用系统的字体发现机制。
*   **Android**：由于底层的 `swash` 光栅化器无法渲染系统自带的 COLRv1 表情符号字体，初始化时会移除这些面（faces），并嵌入捆绑的 COLRv0 变体（Noto Color Emoji）。同时配置默认字体族（如 Roboto）。

---

## 文本测量与缓存

### 测量 API

文本测量是一个两阶段过程，旨在优化性能：
1.  **布局阶段**：执行文本成型和布局，计算尺寸与基线信息，并将结果存入 LRU 缓存。
2.  **渲染阶段**：从缓存中检索数据；如果缓存条目已被逐出，则自动重新计算。

### LRU 缓存系统

创建 `TextData`（文本成型、布局计算等）开销巨大。LRU 缓存存储了最近使用的文本布局结果。缓存键由文本内容、颜色、字体大小、行高和计算出的边界共同组成。

---

## 文本组件

### text 组件

主要文本渲染组件接受 `TextArgs` 配置。如果未显式指定，它将使用当前主题提供的 `TextStyle`。

**用法示例：**

```rust
// 基础文本
text(TextArgs::default()
    .text("你好，世界")
    .size(Dp(16.0))
);

// 结合主题排版的文本
provide_text_style(theme.typography.body_medium, || {
    text(TextArgs::default().text("正文内容"));
});
```

---

## 排版系统

### Material Design 排版

`MaterialTheme` 提供了基于 Material Design 3 的完整排版比例，涵盖从 `display_large`（展示大）到 `label_small`（标签小）的 15 种预定义样式。每种样式包含字体大小和行高。

### 应用排版样式

使用 `provide_text_style()` 为后代文本组件设置当前样式。这种模式允许组件继承排版样式，而无需显式传递参数。

---

## 性能特性

*   **字体系统初始化**：高开销，但仅在首次访问时执行一次。
*   **文本成型**：每帧开销较高，通过 1024 容量的 LRU 缓存进行缓解。
*   **字形光栅化**：通过 glyphon 的 `SwashCache` 管理。
*   **图集更新**：仅在出现新字形时进行增量更新。
