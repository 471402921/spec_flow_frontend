# Flutter 代码模板

> 选型为 Flutter 时，sf-frontend-gen 使用此模板生成代码。
> 技术栈：Dart + Riverpod + GoRouter + Dio + Freezed

## 文件扩展名

所有文件使用 `.dart`，文件名 `snake_case`。

---

## model 模板（Freezed）

```dart
// lib/features/{module}/model/{module}_model.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part '{module}_model.freezed.dart';
part '{module}_model.g.dart';

@freezed
class {Model} with _${Model} {
  const factory {Model}({
    required String id,
    // ... 字段
    required String createdAt,
    required String updatedAt,
  }) = _{Model};

  factory {Model}.fromJson(Map<String, dynamic> json) =>
      _${Model}FromJson(json);
}
```

---

## api 模板

```dart
// lib/features/{module}/api/{module}_api.dart
import 'package:dio/dio.dart';
import '../model/{module}_model.dart';

class {Module}Api {
  final Dio _dio;
  {Module}Api(this._dio);

  Future<List<{Model}>> list({int page = 1, int pageSize = 20}) async {
    final response = await _dio.get('/{module}', queryParameters: {
      'page': page,
      'pageSize': pageSize,
    });
    return (response.data as List)
        .map((e) => {Model}.fromJson(e as Map<String, dynamic>))
        .toList();
  }

  Future<{Model}> getById(String id) async {
    final response = await _dio.get('/{module}/$id');
    return {Model}.fromJson(response.data as Map<String, dynamic>);
  }

  Future<{Model}> create(Map<String, dynamic> data) async {
    final response = await _dio.post('/{module}', data: data);
    return {Model}.fromJson(response.data as Map<String, dynamic>);
  }

  Future<void> delete(String id) async {
    await _dio.delete('/{module}/$id');
  }
}
```

---

## state 模板（Riverpod）

```dart
// lib/features/{module}/state/{module}_state.dart
import 'package:freezed_annotation/freezed_annotation.dart';
import '../model/{module}_model.dart';

part '{module}_state.freezed.dart';

@freezed
class {Module}State with _${Module}State {
  const factory {Module}State({
    @Default([]) List<{Model}> list,
    @Default(0) int total,
    @Default(false) bool loading,
    String? error,
  }) = _{Module}State;
}
```

```dart
// lib/features/{module}/state/{module}_notifier.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../api/{module}_api.dart';
import '{module}_state.dart';

part '{module}_notifier.g.dart';

@riverpod
class {Module}Notifier extends _${Module}Notifier {
  @override
  {Module}State build() => const {Module}State();

  Future<void> fetchList({int page = 1}) async {
    state = state.copyWith(loading: true, error: null);
    try {
      final items = await ref.read({module}ApiProvider).list(page: page);
      state = state.copyWith(list: items, loading: false);
    } catch (e) {
      state = state.copyWith(error: e.toString(), loading: false);
    }
  }
}

@riverpod
{Module}Api {module}Api({Module}ApiRef ref) {
  return {Module}Api(ref.read(dioProvider));
}
```

---

## widget 模板

```dart
// lib/features/{module}/components/{component_name}.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class {ComponentName} extends ConsumerWidget {
  const {ComponentName}({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return Container(
      // UI
    );
  }
}
```

---

## page 模板

```dart
// lib/pages/{module}/{module}_list_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../features/{module}/state/{module}_notifier.dart';
import '../../features/{module}/components/{component_name}.dart';

class {Module}ListPage extends ConsumerStatefulWidget {
  const {Module}ListPage({super.key});

  @override
  ConsumerState<{Module}ListPage> createState() => _{Module}ListPageState();
}

class _{Module}ListPageState extends ConsumerState<{Module}ListPage> {
  @override
  void initState() {
    super.initState();
    Future.microtask(() =>
        ref.read({module}NotifierProvider.notifier).fetchList());
  }

  @override
  Widget build(BuildContext context) {
    final state = ref.watch({module}NotifierProvider);
    if (state.loading) return const Center(child: CircularProgressIndicator());
    return ListView.builder(
      itemCount: state.list.length,
      itemBuilder: (context, index) => {ComponentName}(/* props */),
    );
  }
}
```

---

## services/http（Dio 封装）

```dart
// lib/services/http/dio_client.dart
import 'package:dio/dio.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'dio_client.g.dart';

@riverpod
Dio dio(DioRef ref) {
  final dio = Dio(BaseOptions(
    baseUrl: const String.fromEnvironment('API_BASE_URL'),
    connectTimeout: const Duration(seconds: 10),
    receiveTimeout: const Duration(seconds: 10),
  ));

  // 请求拦截：注入 Token
  dio.interceptors.add(InterceptorsWrapper(
    onRequest: (options, handler) {
      // TODO: 从 secure storage 读取 token
      // options.headers['Authorization'] = 'Bearer $token';
      handler.next(options);
    },
    onResponse: (response, handler) {
      // 解包 Result<T>
      final body = response.data as Map<String, dynamic>;
      if (body['code'] != 0) {
        handler.reject(DioException(
          requestOptions: response.requestOptions,
          message: body['message'] as String,
        ));
        return;
      }
      response.data = body['data'];
      handler.next(response);
    },
    onError: (error, handler) {
      if (error.response?.statusCode == 401) {
        // 跳转登录
      }
      handler.next(error);
    },
  ));

  return dio;
}
```
