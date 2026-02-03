# Flutter プロジェクト構造テンプレート

## Feature-First アーキテクチャ

```
lib/
├── core/                          # 共通モジュール
│   ├── constants/                 # 定数
│   │   ├── app_constants.dart
│   │   └── api_endpoints.dart
│   ├── errors/                    # エラー定義
│   │   ├── exceptions.dart
│   │   └── failures.dart
│   ├── network/                   # ネットワーク設定
│   │   ├── dio_client.dart
│   │   └── interceptors/
│   ├── theme/                     # テーマ設定
│   │   ├── app_theme.dart
│   │   ├── colors.dart
│   │   └── typography.dart
│   └── utils/                     # ユーティリティ
│       ├── extensions/
│       └── validators.dart
│
├── features/                      # 機能モジュール
│   ├── auth/                      # 認証機能
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_local_datasource.dart
│   │   │   │   └── auth_remote_datasource.dart
│   │   │   ├── models/
│   │   │   │   └── user_model.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── user.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── usecases/
│   │   │       ├── login_usecase.dart
│   │   │       └── logout_usecase.dart
│   │   └── presentation/
│   │       ├── providers/
│   │       │   └── auth_provider.dart
│   │       ├── screens/
│   │       │   ├── login_screen.dart
│   │       │   └── register_screen.dart
│   │       └── widgets/
│   │           ├── login_form.dart
│   │           └── social_login_buttons.dart
│   │
│   └── [other_features]/          # 他の機能も同構造
│
├── shared/                        # 共有コンポーネント
│   ├── providers/                 # 共有Provider
│   │   └── app_providers.dart
│   └── widgets/                   # 共有Widget
│       ├── buttons/
│       ├── dialogs/
│       ├── inputs/
│       └── loading/
│
└── main.dart                      # エントリーポイント

test/
├── unit/                          # ユニットテスト
│   └── features/
│       └── auth/
├── widget/                        # ウィジェットテスト
│   └── features/
│       └── auth/
└── mocks/                         # テスト用モック
    └── mock_repositories.dart

integration_test/                  # 統合テスト
└── app_test.dart
```

## ファイルテンプレート

### Entity
```dart
// lib/features/{feature}/domain/entities/{entity}.dart
import 'package:freezed_annotation/freezed_annotation.dart';

part '{entity}.freezed.dart';
part '{entity}.g.dart';

@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required String email,
    @Default(false) bool isVerified,
    DateTime? createdAt,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```

### Repository Interface
```dart
// lib/features/{feature}/domain/repositories/{feature}_repository.dart
abstract class AuthRepository {
  Future<User> login(String email, String password);
  Future<void> logout();
  Future<User?> getCurrentUser();
  Stream<User?> watchAuthState();
}
```

### Provider
```dart
// lib/features/{feature}/presentation/providers/{feature}_provider.dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part '{feature}_provider.g.dart';

@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  FutureOr<User?> build() async {
    return ref.watch(authRepositoryProvider).getCurrentUser();
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      return ref.read(authRepositoryProvider).login(email, password);
    });
  }
}
```

### Screen
```dart
// lib/features/{feature}/presentation/screens/{screen}_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class LoginScreen extends ConsumerWidget {
  const LoginScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authState = ref.watch(authProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('ログイン')),
      body: authState.when(
        data: (user) => const LoginForm(),
        loading: () => const Center(child: CircularProgressIndicator()),
        error: (e, s) => Center(child: Text('エラー: $e')),
      ),
    );
  }
}
```
