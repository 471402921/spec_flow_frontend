# 通用分层规则

> 框架无关，适用于 Flutter 和 React。

## 目录职责

| 目录 | 职责 | 禁止 |
|------|------|------|
| `pages/` | 路由入口，组合 features 组件 | 直接调用 API、包含业务逻辑 |
| `features/{module}/api/` | HTTP 请求封装 | 直接操作 UI 状态 |
| `features/{module}/model/` | 数据类型定义 | 包含业务逻辑 |
| `features/{module}/components/` | 该模块私有 UI 组件 | 跨模块引用其他 feature 内容 |
| `features/{module}/state/` | 模块状态管理 | 直接发起 HTTP 请求 |
| `services/` | 通用服务（HTTP client、存储、认证） | 包含业务逻辑 |
| `shared/` | 跨模块复用的组件和工具函数 | 依赖特定业务模块 |
| `models/` | 全局共享数据类型 | 包含业务逻辑 |

## 依赖方向

```
pages → features/{module}/components → features/{module}/state → features/{module}/api
                                                                ↓
                                                          services/http
```

- 依赖只能向右（向下）流动
- `state/` 调用 `api/`，`api/` 调用 `services/http/`
- 组件通过状态管理间接触发 API，不直接调用

## 模块边界

- 一个 feature 模块对应一个业务领域（如 `auth`、`order`、`product`）
- 模块内可以自由引用本模块内的任何层
- 跨模块共享的内容必须提升到 `shared/` 或 `models/`
- 禁止 `features/A` 直接引用 `features/B` 的内部文件

## 新增模块检查清单

- [ ] 模块名使用 `snake_case`（Flutter）或 `kebab-case`（React）
- [ ] 至少包含 `api/`、`model/`、`state/` 三层
- [ ] `pages/` 中有对应的路由入口页面
- [ ] 没有跨模块的直接依赖
