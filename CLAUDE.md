# SpecFlow Frontend — 框架契约

## 项目说明

SpecFlow Frontend 是 SpecFlow Spec-Driven / Agentic Engineering 框架的前端部分。支持 **Flutter** 和 **React** 双选型，通过下方一行声明切换，Skills 据此自动分发对应模板。

## 当前选型

```
当前选型：react
```

---

## 通用分层规则（框架无关）

```
src/
├── pages/                    # 页面 / 路由入口，只做组合，不含业务逻辑
├── features/{module}/        # 业务模块（按领域划分）
│   ├── api/                  # 该模块的 HTTP 调用
│   ├── model/                # 该模块的数据类型
│   ├── components/           # 该模块私有组件
│   └── state/                # 该模块状态管理
├── services/                 # 通用服务（HTTP client、存储、认证）
├── shared/                   # 跨模块共享组件、工具函数
└── models/                   # 全局数据类型（跨模块共用）
```

**分层约束：**
- `pages/` 只组合 features 的组件，不直接调用 API
- `features/{module}/components/` 不跨模块引用其他 feature 的组件
- API 调用只在 `features/{module}/api/` 或 `services/` 中发起
- 状态管理只在 `features/{module}/state/` 中维护，不散落在组件内

---

## 构建与测试命令

### Flutter

```bash
# 依赖安装
flutter pub get

# 运行（开发）
flutter run

# 构建
flutter build apk          # Android
flutter build ios          # iOS
flutter build web          # Web

# 测试
flutter test               # 单元 + Widget 测试
flutter test --coverage    # 含覆盖率
```

### React

```bash
# 依赖安装
npm install

# 运行（开发）
npm run dev

# 构建
npm run build

# 测试
npm run test               # vitest
npm run test:coverage      # 含覆盖率
```

---

## 与后端 API 对接约定

### 统一响应格式

后端所有接口返回 `Result<T>`：

```json
{
  "code": 0,
  "message": "success",
  "data": {},
  "traceId": "abc-123"
}
```

`code == 0` 表示成功，其余为业务错误。

### 认证

所有需鉴权的请求携带 Bearer Token：

```
Authorization: Bearer <token>
```

### 错误处理映射

| HTTP 状态码 | 处理方式 |
|-------------|----------|
| 401 | 清除 Token，跳转登录页 |
| 403 | 提示"无权限" |
| 422 / 400 | 展示 `message` 字段内容给用户 |
| 500 | 提示"服务异常，请稍后重试"，记录 `traceId` |
| 网络超时 | 提示"网络异常"，支持重试 |

HTTP client 统一封装在 `services/http/`，拦截器处理 Token 注入与错误映射，业务层不做 HTTP 异常处理。

---

## 代码规范

### 命名约定

| 场景 | Flutter | React |
|------|---------|-------|
| 文件名 | `snake_case.dart` | `kebab-case.tsx` |
| 类 / 组件 | `PascalCase` | `PascalCase` |
| 变量 / 函数 | `camelCase` | `camelCase` |
| 常量 | `UPPER_SNAKE_CASE` | `UPPER_SNAKE_CASE` |
| 状态 Provider / Store | `{Module}Provider` / `{Module}Store` | `use{Module}Store` |

### 文件组织原则

- 每个文件只导出一个主要类 / 组件
- 文件名与导出名保持一致
- `index` 文件仅用于聚合导出，不含业务逻辑

### 状态管理原则

- Flutter：使用 Riverpod，State 类用 Freezed 定义为不可变
- React：使用 Zustand，Store 按模块拆分，禁止在组件内用 `useState` 管理业务状态
- 表单状态除外，可在组件内用本地状态

---

## Skills 调用约束

- **不跨阶段**：当前阶段未完成前，不调用下一阶段的 Skill
- **不自动推进**：每个 Skill 执行完毕后，停止并等待用户确认
- **稳定优先**：有疑问时询问，不自行假设需求细节
- **选型感知**：所有 Skill 在执行前读取"当前选型"声明，据此选择模板

### 可用 Skills

| Skill | 触发词 | 说明 |
|-------|--------|------|
| `sf-frontend-gen` | `/sf-frontend-gen` | 根据 Tech Pack 生成业务模块完整代码 |
| `sf-frontend-review` | `/sf-frontend-review` | 代码审核（分层、API 对接、最佳实践） |
| `sf-frontend-test` | `/sf-frontend-test` | 生成单元测试 / Widget 测试 |
