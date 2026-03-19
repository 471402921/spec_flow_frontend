# 测试模式参考

## Flutter

### 单元测试（state / api / model）

```dart
// test/features/{module}/state/{module}_notifier_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:riverpod/riverpod.dart';

class Mock{Module}Api extends Mock implements {Module}Api {}

void main() {
  late ProviderContainer container;
  late Mock{Module}Api mockApi;

  setUp(() {
    mockApi = Mock{Module}Api();
    container = ProviderContainer(
      overrides: [
        {module}ApiProvider.overrideWithValue(mockApi),
      ],
    );
  });

  tearDown(() => container.dispose());

  group('{Module}Notifier', () {
    test('初始状态正确', () {
      final state = container.read({module}NotifierProvider);
      expect(state.list, isEmpty);
      expect(state.loading, false);
      expect(state.error, isNull);
    });

    test('fetchList 成功后更新列表', () async {
      final mockData = [{Model}(id: '1', /* ... */)];
      when(() => mockApi.list()).thenAnswer((_) async => mockData);

      await container.read({module}NotifierProvider.notifier).fetchList();
      final state = container.read({module}NotifierProvider);

      expect(state.list, equals(mockData));
      expect(state.loading, false);
    });

    test('fetchList 失败后设置 error', () async {
      when(() => mockApi.list()).thenThrow(Exception('网络错误'));

      await container.read({module}NotifierProvider.notifier).fetchList();
      final state = container.read({module}NotifierProvider);

      expect(state.error, isNotNull);
      expect(state.loading, false);
    });
  });
}
```

### Widget 测试

```dart
// test/features/{module}/widgets/{component_name}_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  testWidgets('{ComponentName} 渲染正常', (tester) async {
    await tester.pumpWidget(
      const ProviderScope(
        child: MaterialApp(
          home: Scaffold(
            body: {ComponentName}(/* props */),
          ),
        ),
      ),
    );

    expect(find.byType({ComponentName}), findsOneWidget);
    // expect(find.text('期望文案'), findsOneWidget);
  });

  testWidgets('点击触发回调', (tester) async {
    var tapped = false;
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: {ComponentName}(onTap: () => tapped = true),
        ),
      ),
    );

    await tester.tap(find.byType({ComponentName}));
    expect(tapped, isTrue);
  });
}
```

### Model 序列化测试

```dart
test('{Model} fromJson/toJson 正确', () {
  final json = {
    'id': '123',
    'createdAt': '2026-01-01T00:00:00Z',
    'updatedAt': '2026-01-01T00:00:00Z',
  };
  final model = {Model}.fromJson(json);
  expect(model.id, '123');
  expect(model.toJson(), json);
});
```

---

## React

### Store 单元测试（Zustand + vitest）

```typescript
// src/features/{module}/state/{module}.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { use{Module}Store } from './{module}.store';
import { {module}Api } from '../api/{module}.api';

vi.mock('../api/{module}.api');

describe('use{Module}Store', () => {
  beforeEach(() => {
    use{Module}Store.setState({ list: [], loading: false, error: null });
  });

  it('初始状态正确', () => {
    const { list, loading, error } = use{Module}Store.getState();
    expect(list).toEqual([]);
    expect(loading).toBe(false);
    expect(error).toBeNull();
  });

  it('fetchList 成功后更新列表', async () => {
    const mockData = { items: [{ id: '1' }], total: 1 };
    vi.mocked({module}Api.list).mockResolvedValue({ data: mockData } as any);

    await use{Module}Store.getState().fetchList();
    const { list, total, loading } = use{Module}Store.getState();

    expect(list).toEqual(mockData.items);
    expect(total).toBe(1);
    expect(loading).toBe(false);
  });

  it('fetchList 失败后设置 error', async () => {
    vi.mocked({module}Api.list).mockRejectedValue(new Error('网络错误'));

    await use{Module}Store.getState().fetchList();
    const { error, loading } = use{Module}Store.getState();

    expect(error).toBe('网络错误');
    expect(loading).toBe(false);
  });
});
```

### 组件测试（React Testing Library）

```tsx
// src/features/{module}/components/{Component}.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { {Component} } from './{Component}';

describe('{Component}', () => {
  it('渲染正常', () => {
    render(<{Component} {/* props */} />);
    expect(screen.getByRole('...')).toBeInTheDocument();
  });

  it('点击触发 onAction', () => {
    const onAction = vi.fn();
    render(<{Component} onAction={onAction} />);
    fireEvent.click(screen.getByRole('button'));
    expect(onAction).toHaveBeenCalledOnce();
  });

  it('loading 状态展示 skeleton', () => {
    render(<{Component} loading />);
    expect(screen.getByTestId('skeleton')).toBeInTheDocument();
  });
});
```

### API 测试（mock axios）

```typescript
// src/features/{module}/api/{module}.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { {module}Api } from './{module}.api';
import { http } from '@/services/http';

vi.mock('@/services/http', () => ({
  http: { get: vi.fn(), post: vi.fn(), put: vi.fn(), delete: vi.fn() },
}));

describe('{module}Api', () => {
  it('list 发送正确的请求参数', async () => {
    vi.mocked(http.get).mockResolvedValue({ data: { items: [], total: 0 } });
    await {module}Api.list({ page: 2, pageSize: 10 });
    expect(http.get).toHaveBeenCalledWith('/{module}', {
      params: { page: 2, pageSize: 10 },
    });
  });
});
```
