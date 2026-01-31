# 贡献与代码质量

<details>
<summary><strong>相关源文件</strong></summary>

* [.github/workflows/ci.yml](https://github.com/tessera-ui/tessera/blob/821ebad7/.github/workflows/ci.yml)
* [AGENTS.md](https://github.com/tessera-ui/tessera/blob/821ebad7/AGENTS.md)
* [docs/RELEASE_RULE.md](https://github.com/tessera-ui/tessera/blob/821ebad7/docs/RELEASE_RULE.md)
* [scripts/release-package.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/scripts/release-package.rs)
</details>

本页面描述了 Tessera 项目的贡献指南、代码质量标准以及发布流程。涵盖了代码风格规则、提交信息规范、测试要求、CI/CD 工作流以及自动化发布。

关于设置开发环境，请参阅 [开发工作流](Development-Workflow.md)。

---

## 代码风格指南

Tessera 执行严格的代码风格规则以确保一致性。

### 格式化

项目使用 **rustfmt 2024 版本**。所有代码在提交前必须通过 `cargo fmt --all -- --check` 检查。

### 导入组织

导入必须组织为 **四个独立部分**，由空行分隔，并按字母顺序排序：
1.  **标准库** 导入 (`std`, `core`, `alloc`)。
2.  **第三方包** 导入。
3.  **Crate 根路径** 导入 (`crate::*`)。
4.  **子模块** 导入 (`super::*`)。

### 模块组织

所有模块必须使用 `src/module_name.rs` 模式。**禁止** 使用 `src/module_name/mod.rs` 模式。

### 注释政策

*   **文档注释** (`///`, `//!`)：面向最终用户，描述用途和用法。**禁止** 包含实现详情或内部架构笔记。
*   **普通注释** (`//`)：仅用于解释“为什么”这么写，解释非直观的权衡或约束。**禁止** 重述代码逻辑。

---

## 提交指南

### 约定式提交

所有提交必须遵循约定式提交规范，这是自动化发布流程的基础。

*   `feat:`：新功能（触发 **次版本** 升级）。
*   `fix:`：错误修复（触发 **修订号** 升级）。
*   `refactor:` / `chore:` / `docs:`：重构/常规任务/文档（触发 **修订号** 升级）。

### 破坏性变更

破坏性变更必须在正文或页脚包含 `BREAKING CHANGE:` 标记，或在类型后缀加 `!`（如 `feat(api)!:`）。这会触发 **主版本** 升级。

---

## 测试与质量保证

所有代码变更必须：
1.  通过 `cargo check --workspace --all-targets` 编译。
2.  通过 `cargo test --workspace` 所有现有测试。
3.  为新功能包含新的测试用例。
4.  通过 Android 交叉编译验证（在 CI 中自动执行）。

---

## CI/CD 工作流

项目使用 GitHub Actions 自动化质量检查：
*   **CI 工作流**：在每次推送和 PR 时运行，执行编译检查、测试、格式检查以及导入组织验证。
*   **发布工作流**：推送版本标签（如 `tessera-ui-v0.5.0`）时触发，自动生成 GitHub Release 并附加变更日志。
*   **文档工作流**：将 API 文档自动部署到 GitHub Pages。

---

## 发布流程

### 版本规则

Tessera 遵循 [语义化版本](https://semver.org/) (Semantic Versioning)。版本升级由 `scripts/release-package.rs` 脚本自动执行，该脚本会：
1.  分析 Git 提交历史。
2.  确定各包的升级类型。
3.  **依赖传播**：如果一个基础包升级，其所有依赖包都会自动获得至少一个修订号升级。
4.  生成变更日志并更新 `Cargo.toml`。
5.  执行发布并推送标签。

**注意**：该脚本是发布包的 **唯一** 获准方法。
