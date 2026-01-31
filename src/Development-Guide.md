# 开发指南

<details>
<summary><strong>相关源文件</strong></summary>

* [.github/workflows/ci.yml](https://github.com/tessera-ui/tessera/blob/821ebad7/.github/workflows/ci.yml)
* [.github/workflows/docs.yml](https://github.com/tessera-ui/tessera/blob/821ebad7/.github/workflows/docs.yml)
* [.github/workflows/release.yml](https://github.com/tessera-ui/tessera/blob/821ebad7/.github/workflows/release.yml)
* [Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/Cargo.toml)
* [cargo-tessera/CHANGELOG.md](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/CHANGELOG.md)
* [cargo-tessera/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/Cargo.toml)
* [cargo-tessera/src/commands/new.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/new.rs)
* [cargo-tessera/src/commands/plugin.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/plugin.rs)
* [cargo-tessera/src/output.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/output.rs)
* [cargo-tessera/src/template.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/template.rs)
* [scripts/release-package.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/scripts/release-package.rs)
* [tessera-macros/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs)
</details>

本页面为使用 Tessera UI 框架开发应用程序提供了全面指南。它涵盖了在桌面和移动平台上构建、测试和部署 Tessera 应用的实践方面。

**范围**：本指南侧重于开发工作流、工具生态以及构建 Tessera 应用的最佳实践。关于特定主题：

*   初始项目设置与首个组件：参见 [快速开始](Getting-Started.md)
*   热重载工作流与调试：参见 [开发工作流](Development-Workflow.md)
*   Android 与平台特定构建：参见 [平台特定开发](Platform-Specific-Development.md)
*   创建自定义渲染管线：参见 [自定义组件与管线](Custom-Components-and-Pipelines.md)
*   为 Tessera 贡献代码：参见 [贡献与代码质量](Contributing-and-Code-Quality.md)

## 开发生命周期概览

Tessera 的开发遵循标准的 Rust 工作流，并结合了专门用于 UI 开发和跨平台部署的工具。

1.  **项目创建**：使用 `cargo tessera new` 并选择模板。
2.  **组件编写**：使用 `#[tessera]` 宏编写函数式组件。
3.  **开发与测试**：
    *   桌面端：使用 `cargo tessera dev` 实现实时预览。
    *   移动端：使用 `cargo tessera android dev` 实现设备部署。
4.  **构建发布**：生成优化的二进制文件或 APK/AAB 产物。
5.  **版本发布**：遵循约定式提交（Conventional Commits），通过自动化脚本发布。

**来源**：架构概览图 6, [cargo-tessera/src/commands/new.rs L1-L146](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/src/commands/new.rs#L1-L146)

---

## 工具生态

`cargo-tessera` CLI 提供了所有必要的开发命令。它抽象了各平台的构建系统，提供统一接口。

### CLI 命令参考

| 命令 | 用途 | 关键特性 |
| --- | --- | --- |
| `cargo tessera new <name>` | 创建新项目 | 交互式模板选择，名称校验，Handlebars 渲染 |
| `cargo tessera dev` | 开发服务器 | 文件监听，自动重建，进程重启，支持工作区 |
| `cargo tessera build` | 桌面端生产构建 | Release 模式，优化的二进制文件 |
| `cargo tessera android init` | Android 环境搭建 | 生成 Gradle 项目，配置 NDK |
| `cargo tessera android build` | Android 生产构建 | 构建 APK/AAB，签署产物 |
| `cargo tessera android dev` | Android 热重载 | 监听文件，自动重建并部署至设备 |
| `cargo tessera plugin new` | 创建插件 | 提供管线模板和自定义着色器脚手架 |

**来源**：[cargo-tessera/CHANGELOG.md L1-L69](https://github.com/tessera-ui/tessera/blob/821ebad7/cargo-tessera/CHANGELOG.md#L1-L69)

---

## 项目结构

一个通过 `cargo tessera new` 创建的典型项目结构如下：

*   `Cargo.toml`：包清单文件。
*   `src/main.rs`：使用 `tessera_ui::entry!` 的桌面入口点。
*   `gen/android/`：生成的 Android 项目（如果启用了 Android 支持）。
*   `.cargo/config.toml`：Android 编译目标配置。

---

## 编译配置

### 工作区配置

根目录的 `Cargo.toml` 定义了工作区设置，包括 2024 版本解析器、Link-time 优化（LTO）、符号剥离（strip）以及开发模式下的增量编译优化。

### 发布构建

```bash
# 桌面端
cargo tessera build --release

# Android 端
cargo tessera android build

# Android 端 (AAB 商店包)
cargo tessera android build --release --aab
```

**来源**：[Cargo.toml L1-L30](https://github.com/tessera-ui/tessera/blob/821ebad7/Cargo.toml#L1-L30)

---

## 下一步

*   **快速入门**：通过 [快速开始](Getting-Started.md) 创建你的首个项目。
*   **高效开发**：通过 [开发工作流](Development-Workflow.md) 学习迭代式开发。
*   **移动开发**：通过 [平台特定开发](Platform-Specific-Development.md) 构建 Android 应用。
*   **扩展框架**：通过 [自定义组件与管线](Custom-Components-and-Pipelines.md) 扩展渲染能力。
*   **开源贡献**：参考 [贡献与代码质量](Contributing-and-Code-Quality.md) 回馈社区。
