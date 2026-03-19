# 代码审核检查清单

## 通用（框架无关）

### 分层合规
- [ ] `pages/` 中没有直接的 API 调用
- [ ] `pages/` 中没有业务逻辑（数据处理、条件判断等）
- [ ] `features/{module}/components/` 没有跨模块的 feature 依赖
- [ ] `features/{module}/api/` 没有直接操作 UI 状态
- [ ] `features/{module}/state/` 没有直接发起 HTTP 请求

### API 对接
- [ ] Bearer Token 由 HTTP client 拦截器统一注入，业务层无手动拼接
- [ ] 响应按 `Result<T>` 格式解包（`code == 0` 为成功）
- [ ] 401 跳转登录逻辑在拦截器中，业务层无重复处理
- [ ] 403 有"无权限"提示
- [ ] 500 错误有兜底提示，记录或展示 `traceId`
- [ ] 网络超时有重试或提示逻辑

### 命名
- [ ] 文件名符合选型命名约定（flutter: snake_case, react: kebab-case）
- [ ] 类/组件名 PascalCase
- [ ] 变量/函数名 camelCase
- [ ] 常量 UPPER_SNAKE_CASE

### 代码质量
- [ ] 无注释掉的死代码
- [ ] 无 `console.log` / `print` 调试语句遗留
- [ ] 无硬编码的 API URL（应通过环境变量）
- [ ] 无硬编码的 Token 或密钥

---

## Flutter 专项

### Riverpod
- [ ] Provider 在正确的 scope 注册（避免全局 Provider 持有页面级状态）
- [ ] `ref.watch` 用于读取状态（响应式），`ref.read` 用于事件触发
- [ ] Provider 依赖链清晰，无循环依赖

### Freezed
- [ ] State 类使用 Freezed 定义（`@freezed`）
- [ ] 状态修改通过 `copyWith`，不直接修改字段

### Widget
- [ ] Widget 嵌套不超过 5 层，超过则抽取为独立 Widget
- [ ] `BuildContext` 不跨异步使用（`if (!mounted) return`）
- [ ] `StatefulWidget` 的 `dispose` 方法释放了 Controller / Subscription

### 性能
- [ ] 列表使用 `ListView.builder`，不用 `Column` 渲染长列表
- [ ] 图片使用 `cached_network_image` 或等效缓存方案

---

## React 专项

### Zustand
- [ ] Store 按模块拆分，单个 Store 不超过 10 个字段
- [ ] 组件内无业务相关的 `useState`（表单除外）
- [ ] Store action 不直接操作 DOM

### 组件
- [ ] `useEffect` 有完整的依赖数组（无遗漏依赖）
- [ ] `useEffect` 有必要的 cleanup（取消订阅、清除定时器）
- [ ] 频繁渲染的组件考虑了 `React.memo`
- [ ] 昂贵计算使用了 `useMemo`
- [ ] 传给子组件的回调使用了 `useCallback`

### TypeScript
- [ ] 无 `any` 类型（除非有明确注释说明原因）
- [ ] API 响应类型有完整定义，不用 `any` 接收
- [ ] Props 类型使用 `interface` 定义，不用 `type` 的 inline 对象

### 性能
- [ ] 长列表使用虚拟化（`react-virtual` 或等效方案）
- [ ] 图片有懒加载
