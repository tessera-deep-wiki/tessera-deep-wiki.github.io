# 自定义组件与管线

<details>
<summary><strong>相关源文件</strong></summary>

* [tessera-ui/src/renderer/drawer/pipeline.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-ui/src/renderer/drawer/pipeline.rs)
* [tessera-ui/src/renderer/compute/pipeline.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-ui/src/renderer/compute/pipeline.rs)
* [tessera-macros/README.md](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/README.md)
</details>

本页面解释了如何创建自定义组件，以及如何通过自定义管线（Pipeline）扩展 Tessera 的渲染系统。涵盖了组件抽象层、可插拔管线架构，以及如何实现可绘制管线（用于几何渲染）和可计算管线（用于 GPU 加速特效）。

关于使用内置组件的信息，请参阅 [组件库文档](Component-Library.md)。关于通用开发工作流，请参阅 [开发工作流](Development-Workflow.md)。

---

## 渲染架构概览

Tessera 的渲染系统建立在 **可插拔着色器架构** 之上。组件生成抽象命令，这些命令由注册的管线处理，从而实现对渲染行为的完全定制。

### 命令抽象

Tessera 使用类型擦除（type-erased）的命令系统：
*   **组件层**：定义自定义组件并生成命令。
*   **类型擦除层**：命令被包装为 `dyn DrawCommand` 或 `dyn ComputeCommand`。
*   **管线层**：通过 `TypeId` 将命令分发到对应的渲染管线。

---

## 创建自定义组件

组件使用 `#[tessera]` 过程宏定义，该宏会自动生成组件树注册、生命周期管理以及辅助函数（如 `layout`, `input_handler`）注入所需的代码。

**组件定义模式：**
1.  **函数签名**：定义组件的 API。
2.  **宏标注**：处理节点创建和上下文注入。
3.  **`layout()` 调用**：定义自定义布局行为。
4.  **`input_handler()` 调用**：注册事件处理器。

---

## 自定义绘制管线

绘制管线使用顶点和片元着色器将几何图形渲染到屏幕。它们为特定的 `DrawCommand` 类型实现 `DrawablePipeline` trait。

### 绘制管线生命周期

`DrawablePipeline` 包含五个阶段：
*   `begin_frame`：每帧初始化，更新缓冲区。
*   `begin_pass`：每个通道设置，绑定共享资源。
*   `draw`：**必需阶段**，为批处理执行渲染通道。
*   `end_pass` / `end_frame`：清理工作。

### WGSL 着色器

绘制着色器通常包含：
*   **Uniforms**：视图投影矩阵、不透明度、时间等。
*   **Vertex Input**：从组件传递的顶点或实例数据。
*   **SDF 求值**：在片元着色器中通过数学公式计算形状和边框。

---

## 自定义计算管线

计算管线执行 GPU 计算着色器，用于实现模糊、调色或物理模拟。它们实现 `ComputablePipeline` trait。

### 乒乓渲染

对于需要多个步骤的特效（如高斯模糊），框架会自动处理纹理交换。它在 `ComputeContext` 中提供 `input_view`（源）和 `output_view`（目标），每一轮计算后两者互换。

---

## 管线注册

所有自定义管线必须在第一帧之前注册。
*   **绘制管线注册**：`PipelineRegistry::register::<T>(pipeline)`。
*   **计算管线注册**：`ComputePipelineRegistry::register::<T>(pipeline)`。
渲染器会根据命令的 `TypeId` 自动查找并分发。

---

## 性能考量

1.  **命令批处理**：具有相同 `TypeId` 的命令会被自动分组。在实现 `draw()` 方法时，应通过实例化渲染（Instancing）在单个调用中处理所有传入的命令。
2.  **屏障优化**：尽量减少 `sample_region()` 的使用。声明背景采样需求会导致渲染通道断开并触发昂贵的纹理拷贝。
3.  **资源重用**：在 `new()` 中分配缓冲区，在渲染循环中使用 `queue.write_buffer()` 更新，避免频繁创建和销毁 GPU 资源。
