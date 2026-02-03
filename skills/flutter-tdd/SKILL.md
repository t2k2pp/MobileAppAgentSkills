---
name: flutter-tdd
description: Flutter TDDスキル。テスト駆動開発のRed-Green-Refactorサイクル、unit/widget/integration testの実装、Mockito/Mocktailによるモック、CI/CD統合を支援。テスト作成、テスト設計、品質保証作業時に使用。
---

# Flutter TDDスキル

## TDD基本サイクル

### Red-Green-Refactor
```
1. RED:    失敗するテストを書く
2. GREEN: テストが通る最小限のコードを書く
3. REFACTOR: コードを改善する（テストは維持）
```

## テスト種別と使い分け

| 種別 | 対象 | 実行速度 | 信頼性 | カバー率目標 |
|------|------|----------|--------|-------------|
| Unit Test | 関数、クラス | 超高速 | 高 | 80%+ |
| Widget Test | UI コンポーネント | 高速 | 高 | 70%+ |
| Integration Test | 機能全体 | 低速 | 最高 | 主要フロー |

## Unit Test

### 基本構造
```dart
// test/unit/user_notifier_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:mockito/annotations.dart';

@GenerateNiceMocks([MockSpec<AuthRepository>()])
import 'user_notifier_test.mocks.dart';

void main() {
  late MockAuthRepository mockAuthRepository;
  late ProviderContainer container;

  setUp(() {
    mockAuthRepository = MockAuthRepository();
    container = ProviderContainer(
      overrides: [
        authRepositoryProvider.overrideWithValue(mockAuthRepository),
      ],
    );
  });

  tearDown(() => container.dispose());

  group('UserNotifier', () {
    test('初期状態はloading', () {
      final state = container.read(userProvider);
      expect(state, const AsyncValue<User?>.loading());
    });

    test('ログイン成功時にユーザーを返す', () async {
      // Arrange
      final expectedUser = User(id: '1', name: 'Test');
      when(mockAuthRepository.getCurrentUser())
          .thenAnswer((_) async => expectedUser);

      // Act
      await container.read(userProvider.future);

      // Assert
      final state = container.read(userProvider);
      expect(state.value, expectedUser);
    });

    test('エラー時にAsyncValue.errorを返す', () async {
      // Arrange
      when(mockAuthRepository.getCurrentUser())
          .thenThrow(Exception('Network error'));

      // Act & Assert
      await expectLater(
        container.read(userProvider.future),
        throwsException,
      );
    });
  });
}
```

### Mocktail（Mockitoの代替）
```dart
import 'package:mocktail/mocktail.dart';

class MockAuthRepository extends Mock implements AuthRepository {}

// 使用方法は同様
when(() => mockRepo.login(any(), any())).thenAnswer((_) async => user);
```

## Widget Test

### 基本構造
```dart
// test/widget/login_screen_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() {
  group('LoginScreen', () {
    testWidgets('メールアドレスとパスワードフィールドが表示される', (tester) async {
      await tester.pumpWidget(
        const ProviderScope(
          child: MaterialApp(home: LoginScreen()),
        ),
      );

      expect(find.byType(TextField), findsNWidgets(2));
      expect(find.text('メールアドレス'), findsOneWidget);
      expect(find.text('パスワード'), findsOneWidget);
    });

    testWidgets('空のフォーム送信でエラーメッセージが表示される', (tester) async {
      await tester.pumpWidget(
        const ProviderScope(
          child: MaterialApp(home: LoginScreen()),
        ),
      );

      await tester.tap(find.byType(ElevatedButton));
      await tester.pump();

      expect(find.text('メールアドレスを入力してください'), findsOneWidget);
    });

    testWidgets('有効な入力で送信ボタンが有効になる', (tester) async {
      await tester.pumpWidget(
        const ProviderScope(
          child: MaterialApp(home: LoginScreen()),
        ),
      );

      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );
      await tester.pump();

      final button = tester.widget<ElevatedButton>(find.byType(ElevatedButton));
      expect(button.onPressed, isNotNull);
    });
  });
}
```

### Provider Override
```dart
testWidgets('ログイン成功でホーム画面に遷移', (tester) async {
  final mockAuthNotifier = MockAuthNotifier();

  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        authProvider.overrideWith(() => mockAuthNotifier),
      ],
      child: const MaterialApp(home: LoginScreen()),
    ),
  );

  // テスト実行...
});
```

## Integration Test

### セットアップ
```yaml
# pubspec.yaml
dev_dependencies:
  integration_test:
    sdk: flutter
```

### テスト実装
```dart
// integration_test/app_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('ログインフロー', () {
    testWidgets('正常ログイン→ホーム画面表示', (tester) async {
      app.main();
      await tester.pumpAndSettle();

      // ログイン画面が表示される
      expect(find.text('ログイン'), findsOneWidget);

      // 認証情報を入力
      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );

      // ログインボタンをタップ
      await tester.tap(find.text('ログイン'));
      await tester.pumpAndSettle();

      // ホーム画面に遷移
      expect(find.text('ホーム'), findsOneWidget);
    });
  });
}
```

### 実行コマンド
```bash
# Androidエミュレータで実行
flutter test integration_test/app_test.dart

# 特定デバイスで実行
flutter test integration_test --device-id=<device_id>
```

## Golden Test（スクリーンショット比較）

```dart
testWidgets('UIのゴールデンテスト', (tester) async {
  await tester.pumpWidget(
    const ProviderScope(
      child: MaterialApp(home: ProfileCard()),
    ),
  );

  await expectLater(
    find.byType(ProfileCard),
    matchesGoldenFile('goldens/profile_card.png'),
  );
});

// ゴールデン更新
// flutter test --update-goldens
```

## テストカバレッジ

```bash
# カバレッジレポート生成
flutter test --coverage

# HTMLレポート生成
genhtml coverage/lcov.info -o coverage/html

# レポート表示
open coverage/html/index.html
```

## CI/CD統合（GitHub Actions）

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'

      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info
```

## チェックリスト

テスト完了前に確認:
- [ ] 公開メソッドにunit testがある
- [ ] 主要画面にwidget testがある
- [ ] 主要ユーザーフローにintegration testがある
- [ ] エッジケース（null、空、エラー）がテストされている
- [ ] モックが外部依存性に使用されている
- [ ] テストが独立している（順序依存なし）
- [ ] カバレッジが80%以上
