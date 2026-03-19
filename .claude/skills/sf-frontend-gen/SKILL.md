---
name: sf-frontend-gen
description: >
  SpecFlow 前端模块生成 Skill。当用户想为某个业务模块生成前端代码时触发，包括：页面、组件、状态管理、API 调用层、数据模型的完整结构。
  触发场景：用户说 /sf-frontend-gen、"帮我生成 XX 模块"、"根据 Tech Pack 生成前端代码"、"新建一个 feature"、"生成订单/用户/商品模块"等。
  即使用户没有明确说"生成"，只要他们提供了 Tech Pack 或需求描述并希望产出前端代码文件，也应使用此 Skill。
---

# sf-frontend-gen — 前端模块生成

根据 Tech Pack（或需求描述），按 SpecFlow 分层规则生成业务模块的完整代码结构。

## 前置条件

执行前确认：
1. 项目根目录有 `CLAUDE.md`，其中"当前选型"已声明（flutter / react）
2. 用户已提供模块的 Tech Pack 或足够的需求描述

## 工作流程

### Step 1 — 读取选型

读取项目根目录 `CLAUDE.md`，提取"当前选型"字段：

```
当前选型：flutter   →  Step 4 时读取 references/flutter-templates.md
当前选型：react     →  Step 4 时读取 references/react-templates.md
```

同时读取 `references/shared-layer-rules.md` 了解分层规则。

### Step 2 — 理解需求

从用户提供的 Tech Pack 或描述中提取：
- **模块名**（snake_case，如 `order`、`user_profile`）
- **页面列表**及对应路由
- **每个页面的 UI 组件**
- **数据模型字段**（名称、类型、是否必填）
- **后端 API**（endpoint、请求参数、响应字段）
- **状态需求**（列表 / 分页 / 表单 / 加载态 / 错误态）

有不明确的地方时，停止并向用户提问，不自行假设。

### Step 3 — 规划目录结构

按 CLAUDE.md 分层规则，确定将生成的文件结构：

```
src/
├── pages/{module}/
│   └── {PageName}Page.{ext}        # 路由入口，只做组合
└── features/{module}/
    ├── api/
    │   └── {module}_api.{ext}      # HTTP 请求封装
    ├── model/
    │   └── {module}_model.{ext}    # 数据类型定义
    ├── components/
    │   └── {ComponentName}.{ext}   # 模块私有组件
    └── state/
        └── {module}_state.{ext}    # 状态管理
```

### Step 4 — 生成代码

读取对应选型的模板文件（`references/flutter-templates.md` 或 `references/react-templates.md`），以模板为基础逐文件生成代码。

生成时遵守：
- **不跨层调用**：`components/` 不直接调 API，通过 `state/` 间接触发
- **状态集中**：业务状态只在 `state/` 中维护，不散落在组件内
- **错误处理下沉**：HTTP 异常由 `services/http/` 拦截器统一处理，业务层只处理业务逻辑
- **命名遵循** CLAUDE.md 中的命名约定

### Step 5 — 输出确认清单，等待用户确认

列出所有将要写入的文件路径，**不自动写入文件**，等待用户确认后再创建。

## 输出格式

```
## sf-frontend-gen 生成计划

**模块**：{module}
**选型**：{flutter | react}

### 将生成的文件

- [ ] src/pages/{module}/{PageName}Page.{ext}
- [ ] src/features/{module}/api/{module}_api.{ext}
- [ ] src/features/{module}/model/{module}_model.{ext}
- [ ] src/features/{module}/components/{ComponentName}.{ext}
- [ ] src/features/{module}/state/{module}_state.{ext}

### 待确认事项
（列出任何设计上的不确定点，请用户决策）

---
确认后我将逐文件写入代码。
```

## 参考文件

| 文件 | 何时读取 |
|------|---------|
| `references/shared-layer-rules.md` | Step 1，了解分层规则和模块边界 |
| `references/react-templates.md` | Step 4，当前选型为 react 时 |
| `references/flutter-templates.md` | Step 4，当前选型为 flutter 时 |
