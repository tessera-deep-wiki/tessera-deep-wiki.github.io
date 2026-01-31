# 平台特定开发

<details>
<summary><strong>相关源文件</strong></summary>

* [tessera-ui/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-ui/Cargo.toml)
* [cargo-tessera/src/commands/android.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/android.rs)
* [cargo-tessera/src/commands/build.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/build.rs)
* [flake.nix](https://github.com/tessera-ui/tessera/blob/821ebad7/flake.nix)
</details>

本页面涵盖了 Tessera 应用的平台特定开发工作流、构建配置和部署。主要关注为桌面平台（Windows, macOS, Linux）和 Android 构建应用的实践方面。

关于通用开发工作流和热重载，请参阅 [开发工作流](Development-Workflow.md)。关于创建自定义组件，请参阅 [自定义组件与管线](Custom-Components-and-Pipelines.md)。

---

## 桌面平台支持

Tessera 通过平台特定的依赖配置和条件编译来支持主流桌面操作系统。

### 平台特定依赖

核心 `tessera-ui` 根据目标系统包含不同的依赖项：
*   **Unix (Linux/macOS)**：使用 `libc` 处理线程命名。
*   **Windows**：使用 `windows` 包处理线程命名。
*   **标准桌面**：使用不带 `game-activity` 的常规 `winit`。

### 桌面构建配置

构建桌面目标使用标准的 Cargo 工具。`cargo tessera build` 命令封装了 `cargo build`，并提供状态报告和产物路径输出。

---

## Android 平台支持

Tessera 通过集成 `cargo-mobile2`、Gradle 构建系统和 Android NDK 提供全面的 Android 支持。

### 前置条件

Android 开发需要：
1.  **Android SDK**：包含平台工具和构建工具（推荐版本 34）。
2.  **Android NDK**：用于交叉编译的原生开发包。
3.  **ADB**：已加入系统路径的 Android 调试桥。
4.  **Rust 目标**：安装对应的 Android 交叉编译 target。

### 项目初始化

使用 `cargo tessera android init` 生成 `gen/android/` 目录下的 Gradle 项目结构。此过程会：
*   安装所有必要的 Rust 编译目标（ABIs）。
*   从依赖项中收集插件元数据。
*   从 Handlebars 模板生成 Gradle 配置文件。
*   创建包含 Android 清单文件和资源文件的项目目录。

### Android 配置

在 `Cargo.toml` 中，你可以通过 `[package.metadata.tessera.android]` 配置包名、最低 SDK 版本和权限。Tessera 的权限（如 `INTERNET`, `CAMERA`）会自动映射到 Android 对应的清单权限。

### Android 构建过程

`cargo tessera android build` 会：
1.  同步 Gradle 模板。
2.  为选定的 ABI 执行 `cargo build` 交叉编译。
3.  将生成的 `.so` 库文件拷贝到 JNI 目录。
4.  调用 Gradle 打包 APK 或 AAB。

---

## 平台特定代码模式

### 条件编译

Tessera 在代码中广泛使用 `cfg` 属性进行平台检测，例如：
*   `#[cfg(target_os = "windows")]`
*   `#[cfg(target_os = "android")]`

### 剪贴板支持

*   **桌面端**：使用 `arboard` 实现跨平台访问。
*   **Android 端**：通过 JNI 调用 Android 的 `ClipboardManager`。
这些差异被封装在统一的 `Clipboard` API 之下。

---

## Nix 开发环境

提供的 `flake.nix` 文件包含了预配置的桌面和 Android 开发环境：
*   **桌面 Shell**：提供 Rust 工具链、Vulkan 库、X11/Wayland 开发库以及代码格式化工具。
*   **Android Shell**：提供已安装 Android 编译目标的 `rustup`、Android SDK/NDK 以及 `cargo-ndk` 工具。
