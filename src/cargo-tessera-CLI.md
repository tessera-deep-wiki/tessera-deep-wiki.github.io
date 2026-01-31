# cargo-tessera CLI

<details>
<summary><strong>相关源文件</strong></summary>

* [cargo-tessera/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/Cargo.toml)
* [cargo-tessera/README.md](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/README.md)
* [cargo-tessera/src/commands/android.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/android.rs)
* [cargo-tessera/src/commands/dev.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/dev.rs)
* [cargo-tessera/src/commands/new.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/new.rs)
</details>

本文档涵盖了 `cargo-tessera` CLI 工具，它为 Tessera UI 应用提供项目脚手架生成、开发工作流自动化以及构建支持。该 CLI 处理项目创建、热重载开发服务器、桌面构建以及 Android 部署工作流。

关于框架核心信息，请参阅 [tessera-ui 核心库](tessera-ui-Core.md)。关于组件库文档，请参阅 [组件库文档](Component-Library.md)。

**来源：** [cargo-tessera/README.md L1-L68](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/README.md#L1-L68)

---

## 安装与分发

`cargo-tessera` 二进制文件作为 Cargo 扩展分发，允许作为 `cargo tessera <command>` 调用。安装：

```bash
cargo install cargo-tessera
```

该包通过 `include_dir` 包含了内嵌模板，确保模板可用而无需单独下载。

---

## 命令结构

CLI 使用分层命令结构，包含根命令 `tessera` 和多个子命令：

*   **new**：项目脚手架生成。
*   **dev**：热重载开发服务器。
*   **build**：桌面端构建。
*   **android**：Android 工作流（init, build, dev, rust-build）。
*   **plugin**：插件脚手架生成。

**来源：** [cargo-tessera/src/main.rs L15-L153](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/main.rs#L15-L153)

---

## 项目脚手架 (new 命令)

`new` 命令使用内嵌模板和 Handlebars 渲染来创建项目。

### 命令调用

```bash
cargo tessera new my-app [--template <template>]
```

如果省略项目名称或模板，交互式提示将引导用户进行选择。

---

## 开发服务器 (dev 命令)

`dev` 命令实现了一个文件监听开发服务器，它会在源文件更改时自动重建并重启应用。

### 监听循环架构

实现维护了三个进程状态：
*   `build_child`：活跃的编译进程。
*   `child`：正在运行的应用进程。
*   `pending_change`：指示文件已修改的标志。

**来源：** [cargo-tessera/src/commands/dev.rs L15-L192](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/dev.rs#L15-L192)

---

## Android 开发工作流

`android` 子命令提供了一个完整的 Android 开发管线，集成了 Gradle 和 Android NDK。

### Android 初始化工作流

`init` 命令为 Android Studio 集成生成 Gradle 项目结构。过程包括：
1.  安装所有 Android ABI 的 Rust 目标。
2.  收集插件。
3.  准备模板数据（应用元数据、权限、依赖项）。
4.  渲染 Gradle 文件。
5.  同步插件模块。

**来源：** [cargo-tessera/src/commands/android.rs L309-L378](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/android.rs#L309-L378)

---

## 错误处理

CLI 使用 `anyhow::Result` 进行错误传播，并通过 `Context` trait 格式化带有上下文信息的错误。`main` 函数打印错误链，显示完整的错误上下文以便调试。

**来源：** [cargo-tessera/src/main.rs L155-L247](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/main.rs#L155-L247)
