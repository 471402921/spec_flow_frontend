# React 代码模板

> 选型为 React 时，sf-frontend-gen 使用此模板生成代码。
> 技术栈：TypeScript + Vite + Zustand + React Router + Axios + shadcn/ui

## 文件扩展名

| 类型 | 扩展名 |
|------|--------|
| 组件 | `.tsx` |
| 逻辑 / 工具 | `.ts` |
| 样式 | `.css`（或 Tailwind class，无独立样式文件） |

---

## model 模板

```typescript
// src/features/{module}/model/{module}.model.ts

export interface {Model} {
  id: string;
  // ... 字段
  createdAt: string;
  updatedAt: string;
}

export interface {Module}ListParams {
  page: number;
  pageSize: number;
  // ... 筛选字段
}
```

---

## api 模板

```typescript
// src/features/{module}/api/{module}.api.ts
import { http } from '@/services/http';
import type { {Model}, {Module}ListParams } from '../model/{module}.model';
import type { PageResult } from '@/models/common';

export const {module}Api = {
  list: (params: {Module}ListParams) =>
    http.get<PageResult<{Model}>>('/{module}', { params }),

  getById: (id: string) =>
    http.get<{Model}>(`/{module}/${id}`),

  create: (data: Omit<{Model}, 'id' | 'createdAt' | 'updatedAt'>) =>
    http.post<{Model}>('/{module}', data),

  update: (id: string, data: Partial<{Model}>) =>
    http.put<{Model}>(`/{module}/${id}`, data),

  delete: (id: string) =>
    http.delete<void>(`/{module}/${id}`),
};
```

---

## state 模板（Zustand）

```typescript
// src/features/{module}/state/{module}.store.ts
import { create } from 'zustand';
import { {module}Api } from '../api/{module}.api';
import type { {Model}, {Module}ListParams } from '../model/{module}.model';

interface {Module}State {
  list: {Model}[];
  total: number;
  loading: boolean;
  error: string | null;
  params: {Module}ListParams;
}

interface {Module}Actions {
  fetchList: (params?: Partial<{Module}ListParams>) => Promise<void>;
  reset: () => void;
}

const initialState: {Module}State = {
  list: [],
  total: 0,
  loading: false,
  error: null,
  params: { page: 1, pageSize: 20 },
};

export const use{Module}Store = create<{Module}State & {Module}Actions>((set, get) => ({
  ...initialState,

  fetchList: async (params) => {
    const merged = { ...get().params, ...params };
    set({ loading: true, error: null, params: merged });
    try {
      const res = await {module}Api.list(merged);
      set({ list: res.data.items, total: res.data.total, loading: false });
    } catch (e: any) {
      set({ error: e.message, loading: false });
    }
  },

  reset: () => set(initialState),
}));
```

---

## component 模板

```tsx
// src/features/{module}/components/{Component}.tsx
import type { FC } from 'react';

interface {Component}Props {
  // props
}

export const {Component}: FC<{Component}Props> = ({ /* props */ }) => {
  return (
    <div>
      {/* UI */}
    </div>
  );
};
```

---

## page 模板

```tsx
// src/pages/{module}/{Module}ListPage.tsx
import { useEffect } from 'react';
import { use{Module}Store } from '@/features/{module}/state/{module}.store';
import { {Component} } from '@/features/{module}/components/{Component}';

export default function {Module}ListPage() {
  const { list, loading, fetchList } = use{Module}Store();

  useEffect(() => {
    fetchList();
  }, []);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      {list.map((item) => (
        <{Component} key={item.id} {/* props */} />
      ))}
    </div>
  );
}
```

---

## services/http 模板

```typescript
// src/services/http/index.ts
import axios from 'axios';

export const http = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
});

// 请求拦截：注入 Token
http.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// 响应拦截：解包 Result<T>，处理错误
http.interceptors.response.use(
  (response) => {
    const { code, message, data } = response.data;
    if (code !== 0) return Promise.reject(new Error(message));
    return { ...response, data };
  },
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```
