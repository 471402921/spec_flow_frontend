# SpecFlow Frontend — 前端开发 SOP

本文档是前端开发的标准操作流程（SOP），面向 AI（Claude Code）和开发者。

---

## 开发流程总览

```
需求/PRD → Tech Pack（前端部分）→ /sf-frontend-gen → /sf-frontend-review → /sf-frontend-test → 提交
```

每个阶段由对应 Skill 驱动，**阶段间不自动推进，需用户明确确认**。

---

## Phase 0 — 需求准备

### PRD
- 复用后端 `/specflow-doc-prd` 生成的 PRD，或由用户直接提供需求描述
- 前端无独立 PRD Skill，需求文档放入 `doc/requirements/`

### Tech Pack（前端部分）
- 用户基于后端 Tech Pack 补充前端内容，保存到 `doc/design/`
- 前端 Tech Pack 应包含：
  - 页面列表与路由设计
  - 每个页面的 UI 布局描述（或 Figma 链接）
  - 组件拆分建议
  - 状态字段（哪些需要全局状态，哪些是局部状态）
  - 与后端 API 的对应关系

---

## Phase 1 — 模块生成（sf-frontend-gen）

**触发**：`/sf-frontend-gen`

**输入**：Tech Pack 或需求描述 + 已声明的当前选型

**产出**：
- `src/pages/{module}/` — 页面入口
- `src/features/{module}/` — 业务模块（api / model / components / state）

**注意事项**：
- 生成前 Skill 会输出文件清单，**确认后才写入**
- 生成的代码是初稿，需经 Phase 2 审核
- 如果模块间有共享类型，先提升到 `src/models/` 再生成

---

## Phase 2 — 代码审核（sf-frontend-review）

**触发**：`/sf-frontend-review`

**输入**：Phase 1 生成的代码（或任意已有代码）

**产出**：带 🔴/🟡/🟢 分级的审核报告

**修复优先级**：
1. 🔴 阻断问题：分层违规、运行时崩溃风险 — 必须修复后才能进入 Phase 3
2. 🟡 建议问题：性能、可读性 — 视情况修复
3. 🟢 符合规范 — 无需操作

---

## Phase 3 — 测试生成（sf-frontend-test）

**触发**：`/sf-frontend-test`

**输入**：Phase 1 生成的源码（Phase 2 审核通过后）

**产出**：
- Flutter：`test/features/{module}/` 下的 `*_test.dart`
- React：`src/features/{module}/` 下的 `*.test.ts(x)`

**覆盖目标**：
- `api/` 层：mock HTTP，验证请求参数和响应解析
- `state/` 层：初始态、成功态、错误态、状态转换
- `components/`：渲染正常、交互触发、loading/empty/error 状态

---

## 分层规则速查

```
src/
├── pages/          路由入口，只做组合
├── features/
│   └── {module}/
│       ├── api/    HTTP 请求，只在这里发起
│       ├── model/  数据类型定义
│       ├── components/  模块私有组件
│       └── state/  状态管理，只在这里维护业务状态
├── services/       通用服务（http client、auth、storage）
├── shared/         跨模块共享组件和工具
└── models/         全局共享数据类型
```

**黄金法则**：
- 组件不直接调 API → 通过 state 间接触发
- 状态不散落在组件内 → 集中在 state/
- HTTP 异常不在业务层处理 → 由 services/http 拦截器统一处理

---

## 选型切换

1. 修改 `CLAUDE.md` 中"当前选型"一行
2. 重新运行 `/sf-frontend-gen` 生成对应框架的代码
3. 选型切换不影响分层规则，只影响模板和命名约定

---

## 常用命令速查

| 操作 | Flutter | React |
|------|---------|-------|
| 安装依赖 | `flutter pub get` | `npm install` |
| 启动开发 | `flutter run` | `npm run dev` |
| 运行测试 | `flutter test` | `npm run test` |
| 构建产物 | `flutter build apk/ios/web` | `npm run build` |
| 代码生成 | `dart run build_runner build` | — |

---

## 文档索引

| 文件/目录 | 说明 |
|-----------|------|
| `CLAUDE.md` | 框架契约，包含选型声明、分层规则、API 约定 |
| `doc/requirements/` | PRD 和需求文档 |
| `doc/design/` | Tech Pack（前端部分）|
| `.claude/skills/sf-frontend-gen/` | 模块生成 Skill |
| `.claude/skills/sf-frontend-review/` | 代码审核 Skill |
| `.claude/skills/sf-frontend-test/` | 测试生成 Skill |
| `src/` | 实际源码（由 Skill 生成） |
