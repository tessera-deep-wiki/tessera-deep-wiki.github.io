# 开发工作流

<details>
<summary><strong>相关源文件</strong></summary>

* [cargo-tessera/src/commands/dev.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/dev.rs)
* [cargo-tessera/src/commands/android.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/android.rs)
* [cargo-tessera/README.md](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/README.md)
</details>

## 目的与范围

本文档涵盖了由 `cargo-tessera` 提供的迭代式开发工作流，重点介绍通过自动重建和重启实现快速开发的 `dev` 和 `android dev` 命令。关于初始项目设置，请参阅 [快速开始](Getting-Started.md)。关于平台特定的构建配置，请参阅 [平台特定开发](Platform-Specific-Development.md)。

开发工作流的核心是文件监听、智能构建触发以及进程生命周期管理，旨在最小化 UI 迭代过程中的阻力。

---

## 桌面开发服务器

`cargo tessera dev` 命令提供了一个文件监听开发服务器，当源文件发生变化时，它会自动重建并重启你的应用。

### 命令调用

```bash
# 基础用法 - 监听当前包
cargo tessera dev

# 显示详细的 Cargo 输出
cargo tessera dev --verbose

# 针对特定的工作区包
cargo tessera dev --package my-package

# 开启发布模式优化（构建较慢，运行较快）
cargo tessera dev --release
```

### 文件监听架构

开发服务器使用 `notify` 包监控文件系统变更。监听范围包括包目录下的 `src/`（递归）、`Cargo.toml` 以及 `build.rs`。

**防抖与触发逻辑：**
系统使用 300ms 的防抖窗口。当检测到新的文件事件时，如果当前正在进行构建，开发服务器会立即终止旧的构建进程，以确保只构建最新的代码状态。

### 生命周期与终止

开发服务器会持续监控构建进程和运行中的应用。
*   **应用退出处理**：当应用进程退出时，开发服务器会停止运行，而不是自动重启。这允许开发者在修改配置或参数时不会陷入自动重启的循环。
*   **异常诊断**：如果应用崩溃或以异常状态码退出，开发服务器会显示退出码以帮助诊断。

---

## Android 开发模式

`cargo tessera android dev` 命令提供类似的文件监听功能，但会将应用部署到 Android 设备而非本地运行。

### 设备要求

Android 开发需要连接物理设备或模拟器，并确保 ADB 访问权限。你可以通过 `adb devices` 获取设备 ID 并在启动时通过 `--device` 标志指定。

### Android 部署流程

1.  **同步项目**：更新 Gradle 构建文件。
2.  **构建 Rust 库**：针对 Android 目标（如 `aarch64-linux-android`）进行交叉编译。
3.  **打包 APK**：使用 Gradle 将编译好的 Rust 库打包进 APK。
4.  **安装并运行**：通过 ADB 安装 APK 到设备并启动 Activity。
5.  **流式日志**：通过 Logcat 显示过滤后的应用输出。

当检测到文件变更时，进行中的 Android 部署会被取消并重新触发。

---

## 调试技术

### 详细模式

使用 `--verbose` 标志可以启用完整的 `cargo` 命令输出，有助于诊断构建问题。它会显示完整的编译器输出、依赖解析详情以及链接信息。

### 构建失败恢复

如果构建失败，开发服务器会保持监听模式，显示警告信息并等待下一次文件修改，而不会终止。只有在触发新的成功构建后，旧的应用实例才会被替换。

### 运行时错误诊断

*   **退出状态报告**：应用崩溃时，控制台会显示类似 `application exited with status 101` 的提示。
*   **Android 日志过滤**：Android 开发模式会根据噪声级别（如 `Polite`, `Debug`, `Verbose`）自动过滤 Logcat 输出，使开发者专注于应用相关的日志。
