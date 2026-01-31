# 毛玻璃效果

<details>
<summary><strong>相关源文件</strong></summary>

* [example/src/example_components/fluid_glass.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/fluid_glass.rs)
* [example/src/example_components/glass_button.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/glass_button.rs)
* [example/src/example_components/glass_slider.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/glass_slider.rs)
* [example/src/example_components/glass_switch.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/example_components/glass_switch.rs)
</details>

本页面记录了 Tessera UI 中的毛玻璃（Glassmorphism）视觉效果系统，包括核心 `fluid_glass` 组件以及专门的毛玻璃样式 UI 组件（如滑块和开关）。毛玻璃效果通过模糊、色调颜色和边框提供了现代的半透明背景。

关于不带毛玻璃效果的基础交互组件，请参阅 [交互组件](Interactive-Components.md)。关于毛玻璃组件的布局与定位，请参阅 [布局组件](Layout-Components.md)。

## 概览

Tessera 的毛玻璃系统通过自定义 WGSL 计算和片元着色器实现。它提供背景模糊、色散（chromatic dispersion）、光折射、过程噪声以及交互式波纹。

## 核心毛玻璃效果系统

### fluid_glass 组件

`fluid_glass` 函数创建具有多种视觉属性的可配置毛玻璃效果。它接受 `FluidGlassArgs` 参数，并在其表面渲染子内容。

**视觉效果属性：**
*   **色调颜色** (`tint_color`)：基础色调，通过 Alpha 控制强度。
*   **模糊半径** (`blur_radius`)：背景模糊强度。
*   **色散高度** (`dispersion_height`)：色差效果范围。
*   **折射强度** (`refraction_amount`)：光线扭曲强度。
*   **噪声量** (`noise_amount`)：表面纹理强度。
*   **边框** (`border`)：带有通过着色器计算的模拟光照高亮效果的边框。

**渲染管线：**
渲染过程包含多个阶段：通过计算着色器处理背景内容（模糊、均值颜色、对比度），随后应用最终的片元着色器以实现折射、色散和边框渲染。

---

## 毛玻璃 UI 组件

这些组件是基础交互组件的毛玻璃版本，提供了统一的视觉风格。

### 毛玻璃按钮

包装了 `fluid_glass` 并管理点击交互与波纹动画。提供标准 `glass_button` 和优化过的 `glass_icon_button`。

### 毛玻璃滑块

提供带有毛玻璃样式的交互式滑块。它渲染一条玻璃轨道和一个可拖动的玻璃滑块。

### 毛玻璃开关

提供具有毛玻璃风格的切换开关。它具有玻璃轨道和在开启/关闭位置之间滑动的动画玻璃滑块。

---

## 渲染实现详情

`FluidGlassPipeline` 使用 GPU 计算管线实时处理背景内容：
1.  **背景捕获**：抓取当前组件后方的屏幕内容。
2.  **模糊计算**：执行双通道高斯模糊。
3.  **色调混合**：与用户指定的色调颜色进行混合。
4.  **SDF 形状求值**：使用有向距离场（SDF）定义组件形状。
5.  **物理模拟**：计算表面法线以实现折射和真实的边框高亮。

这种实时处理方式确保了当毛玻璃组件下方的内容移动或改变时，效果能实时更新。
