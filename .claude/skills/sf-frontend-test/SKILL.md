---
name: sf-frontend-test
description: >
  SpecFlow 前端测试生成 Skill。当用户想为前端模块生成测试代码时触发，支持 Flutter（widget test + unit test）和 React（vitest + React Testing Library）。
  触发场景：用户说 /sf-frontend-test、"帮我写测试"、"生成单元测试"、"给这个组件写 test"、"补充测试覆盖"等。
  即使用户只是说"这个模块还没有测试"，也应主动使用此 Skill 生成测试计划。
---

# sf-frontend-test — 前端测试生成

根据当前选型，为指定模块或文件生成单元测试和组件测试。

## 工作流程

### Step 1 — 读取选型

读取 `CLAUDE.md`，提取"当前选型"字段，决定生成 Flutter 还是 React 测试。

同时读取 `references/test-patterns.md`，作为本次生成的测试模板参考。

### Step 2 — 确定测试范围

如果用户触发时已指定模块或文件，直接进入 Step 3。
如果范围不明确，询问：
1. 测试哪个模块或文件？
2. 测试类型偏好：单元测试 / 组件测试 / 全部？（默认：全部）

### Step 3 — 分析被测代码

读取目标文件，提取：
- 公开函数 / 方法签名
- 组件 props / 输入参数
- 状态变化路径（action → state 变化）
- API 调用点（需要 mock）
- 边界条件与异常路径（空列表、loading、error）

### Step 4 — 规划测试文件

测试文件放置位置：

```
# Flutter
test/
└── features/{module}/
    ├── {module}_api_test.dart
    ├── {module}_state_test.dart
    └── widgets/{ComponentName}_test.dart

# React（与源码同目录）
src/features/{module}/
    ├── api/{module}.test.ts
    ├── state/{module}.test.ts
    └── components/{ComponentName}.test.tsx
```

每层测试覆盖重点：

| 层 | 覆盖内容 |
|----|---------|
| `api/` | mock HTTP，验证请求参数和响应解析 |
| `state/` | 初始状态、action 触发后变化、错误态 |
| `components/` | 渲染正常、交互回调、loading / empty / error 状态 |
| `model/` | 序列化 / 反序列化（如有复杂逻辑） |

### Step 5 — 输出测试计划，等待确认

列出将生成的测试文件和关键测试用例，**不自动写入**，等待用户确认后再创建。

## 输出格式

```
## sf-frontend-test 生成计划

**模块**：{module}
**选型**：{flutter | react}
**测试类型**：{单元测试 | 组件测试 | 全部}

### 将生成的测试文件

- [ ] {test_file_path} — 覆盖：{函数/组件列表}

### Mock 依赖
- {需要 mock 的模块或服务}

### 关键测试用例
（供确认覆盖是否完整）
- {描述关键 case，每层 2-4 个}

---
确认后我将逐文件写入测试代码。
```

## 参考文件

| 文件 | 何时读取 |
|------|---------|
| `references/test-patterns.md` | Step 1，读取对应选型的测试模板 |
