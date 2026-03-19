---
name: sf-frontend-review
description: >
  SpecFlow 前端代码审核 Skill。当用户想检查前端代码质量时触发，包括：分层是否正确、与后端 API 对接是否规范、框架最佳实践是否遵守。
  触发场景：用户说 /sf-frontend-review、"帮我 review 一下代码"、"检查分层是否正确"、"看看有没有问题"、"审核 XX 模块"等。
  即使用户只是问"这样写有没有问题"并附上代码，也应使用此 Skill 进行系统性审核。
---

# sf-frontend-review — 前端代码审核

对 `src/` 目录或指定模块进行系统性代码审核，输出带优先级的问题列表。

## 工作流程

### Step 1 — 读取选型与规范

读取 `CLAUDE.md`，提取当前选型、分层规则、命名约定、API 对接约定。

同时读取 `references/review-checklist.md`，作为本次审核的检查依据。

### Step 2 — 确定审核范围

如果用户在触发时已指定模块或文件，直接开始审核，无需追问。
如果范围不明确，才询问：审核整个 `src/` 还是指定模块？

### Step 3 — 分层合规检查

读取目标文件，逐一检查：

| 检查项 | 规则 |
|--------|------|
| API 调用位置 | 只允许在 `features/{module}/api/` 或 `services/` 中 |
| 状态管理位置 | 业务状态只在 `features/{module}/state/`，不散落在组件内 |
| 跨模块引用 | `features/A/components/` 不得直接引用 `features/B/` 内容 |
| pages 层职责 | 只做组合，不含业务逻辑和直接 API 调用 |
| 错误处理位置 | 业务层不做 HTTP 异常 try/catch，由拦截器统一处理 |

### Step 4 — API 对接规范检查

- Bearer Token 是否由拦截器统一注入（业务层无手动拼接）
- 响应是否按 `Result<T>` 格式解析（`code == 0` 为成功）
- 401 / 403 / 500 是否有对应的统一处理

### Step 5 — 框架最佳实践检查

**Flutter：**
- Riverpod Provider 是否在正确 scope 注册
- State 类是否用 Freezed 定义（不可变）
- Widget 嵌套是否超过 5 层（建议拆分）
- `BuildContext` 是否跨异步使用

**React：**
- Zustand Store 是否按模块拆分，无混用
- `useEffect` 是否有完整依赖数组
- 频繁渲染路径是否考虑了 `memo` / `useMemo` / `useCallback`
- 是否有 `any` 类型遗留

### Step 6 — 输出审核报告

按格式输出完整报告，**不自动修改代码**，等待用户指示。

## 输出格式

```
## sf-frontend-review 审核报告

**审核范围**：{src/ | features/{module}}
**选型**：{flutter | react}
**审核时间**：{date}

### 问题列表

#### 🔴 必须修复（阻断）
- [ ] `文件路径:行号` — 问题描述

#### 🟡 建议修复（非阻断）
- [ ] `文件路径:行号` — 问题描述

#### 🟢 符合规范
- 分层结构：✅
- API 对接：✅
- 命名约定：✅

### 总结
（整体评价，1-3 句话）

---
如需修复，请告知是否由我来处理。
```

## 参考文件

| 文件 | 何时读取 |
|------|---------|
| `references/review-checklist.md` | Step 1，作为检查依据全文读取 |
