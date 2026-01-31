# tessera-shard

<details>
<summary><strong>相关源文件</strong></summary>

* [Cargo.lock](https://github.com/tessera-ui/tessera/blob/821ebad7/Cargo.lock)
* [example/src/app.rs](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs)
* [tessera-ui/Cargo.toml](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-ui/Cargo.toml)
</details>

**tessera-shard** 是 Tessera UI 框架的路由与导航包。它提供了声明式的客户端路由，具有类型安全的导航、编译时路由验证，并与组件状态管理无缝集成。

关于更广泛的状态管理系统（包括 `remember` 和 `retain`），请参阅 [状态管理](State-Management.md)。关于 `#[shard]` 过程宏的实现，请参阅 [宏库](tessera-macros.md)。

---

## 概览

`tessera-shard` 包允许在 Tessera 应用的不同视图（称为“分片”）之间进行导航。每个分片都是一个被 `#[shard]` 标注的组件函数，它会自动生成一个实现 `RouterDestination` trait 的对应 `*Destination` 结构体。`Router` 单例管理导航栈，并提供推入（push）、弹出（pop）和替换路由的方法。

**来源：** [tessera-ui/Cargo.toml L13-L14](https://github.com/tessera-ui/tessera/blob/821ebad7/tessera-ui/Cargo.toml#L13-L14)

---

## 核心组件

### 路由单例

`Router` 是管理导航栈并提供线程安全路由操作访问的全局单例。

**来源：** [example/src/app.rs L149-L160](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs#L149-L160)

### 路由目标 Trait (RouterDestination Trait)

`RouterDestination` trait 定义了可导航目标的接口。每个目标负责渲染其关联的组件并管理任何所需的状态。

**来源：** [example/src/app.rs L377-L383](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs#L377-L383)

---

## 导航操作

### 推入路由

向导航栈添加新路由：

```rust
Router::with_mut(|router| {
    router.push(TextInputShowcaseDestination {});
});
```

### 弹出路由

移除当前路由并返回上一个：

```rust
Router::with_mut(|router| {
    router.pop();
});
```

### 重置栈

清空导航栈并设置新的根目标：

```rust
Router::with_mut(|router| {
    router.reset_with(HomeDestination { ... });
});
```

---

## 路由根节点

`router_root` 函数在应用入口点初始化路由系统并渲染当前目标。

**用法：**

```rust
scaffold(
    ScaffoldArgs::default().top_bar(top_app_bar),
    move || {
        router_root(HomeDestination { ... });
    },
);
```

**来源：** [example/src/app.rs L227-L232](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs#L227-L232)

---

## 分片中的状态管理

分片组件可以接受通过目标结构体传递的参数。这实现了有状态导航，使路由状态能够在整个导航栈中得到保留。

### 参数类型

| 参数类型 | 用途 | 示例 |
| --- | --- | --- |
| `State<T>` | 共享的可变状态 | `State<BottomSheetController>` |
| 原始类型 | 简单数据 | `usize`, `String` |
| 自定义结构体 | 复杂数据 | 目标特定的数据 |

**来源：** [example/src/app.rs L377-L383](https://github.com/tessera-ui/tessera/blob/821ebad7/example/src/app.rs#L377-L383)
