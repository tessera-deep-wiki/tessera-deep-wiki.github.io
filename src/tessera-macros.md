# tessera-macros

<details>
<summary><strong>相关源文件</strong></summary>

* [tessera-macros/CHANGELOG.md](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/CHANGELOG.md)
* [tessera-macros/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/Cargo.toml)
* [tessera-macros/README.md](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/README.md)
* [tessera-macros/src/lib.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs)
* [tessera-ui/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-ui/Cargo.toml)
</details>

`tessera-macros` 包提供了将常规 Rust 函数转换为 Tessera UI 组件的过程宏。本文档涵盖了宏的实现、代码生成模式、控制流插桩以及运行时集成。

关于使用这些宏的路由系统，请参阅 [底层接口](tessera-shard.md)。关于组件生命周期和运行时的详情，请参阅 [组件模型](Component-Model.md) 和 [状态管理](State-Management.md)。

---

## 目的与范围

`tessera-macros` 包实现了两个属性宏：

1.  **`#[tessera]`** — 通过注入运行时集成代码、在组件树中注册组件并提供布局与事件处理的辅助函数，将函数转换为无状态 UI 组件。
2.  **`#[shard]`** — 扩展了 `#[tessera]` 的导航能力，生成一个实现 `RouterDestination` 的 `*Destination` 结构体，并可选地管理具有可配置生命周期的分片状态。

这两个宏都在编译时使用 `syn`、`quote` 和 `proc-macro2` 包执行 AST 转换。

**来源：** [tessera-macros/src/lib.rs L1-L11](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs#L1-L11)

---

## #[tessera] 宏

### 转换步骤

`#[tessera]` 属性宏执行以下转换：

*   **解析**：使用 `syn::ItemFn` 解析函数签名和主体。
*   **种子生成**：从函数名生成稳定的哈希种子，以防止 ID 冲突。
*   **控制流插桩**：使用 `GroupGuard` 包装条件语句、循环和 match 分支。
*   **Logic ID 生成**：通过 `module_path!()` + 函数名计算稳定的 `logic_id`。
*   **辅助函数注入**：将 `layout` 和 `input_handler` 函数注入到作用域中。
*   **树注册**：调用 `ComponentTree::add_node()` 注册组件。
*   **清理守卫**：注入 RAII 句柄，以便在作用域退出时自动进行树清理。

**来源：** [tessera-macros/src/lib.rs L220-L305](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs#L220-L305)

---

## 控制流插桩

### 目的

为了在条件分支（如 `if`、`match`、`for`、`while`）中维持稳定的组件身份，宏使用 `GroupGuard` 包装每个分支。这确保了不同分支中的 `remember()` 调用能获得独特的槽位键（slot keys）。

### ControlFlowInstrumenter

`ControlFlowInstrumenter` 结构体实现了 `syn::visit_mut::VisitMut` 来遍历并修改 AST。它会为每个分支生成类似 `let _group_guard = GroupGuard::new(group_id);` 的代码。

**来源：** [tessera-macros/src/lib.rs L98-L198](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs#L98-L198)

---

## #[shard] 宏

### 目的

`#[shard]` 宏扩展了 `#[tessera]` 以支持导航和每个目标的渲染状态管理。它会生成：

1.  一个实现 `RouterDestination` 的 `*Destination` 结构体。
2.  绑定到目标生命周期的可选延迟初始化状态。
3.  与 `Router` 和 `ShardRegistry` 的集成。

**来源：** [tessera-macros/src/lib.rs L307-L368](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs#L307-L368)

### 状态参数处理

#### 生命周期模式

| 标注 | 生命周期 | 行为 |
| --- | --- | --- |
| `#[state]` | 分片级 | 当目标被弹出 (`pop`) 时移除状态 |
| `#[state(app)]` | 应用级 | 状态在整个应用生命周期内持久化 |

**来源：** [tessera-macros/src/lib.rs L386-L414](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-macros/src/lib.rs#L386-L414)
