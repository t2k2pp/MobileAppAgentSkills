# テストケーステンプレート

## 機能: [機能名]

---

## 1. ユニットテスト

### 1.1 [対象クラス/関数名]

#### テストケース一覧
| ID | テスト内容 | 入力 | 期待結果 |
|----|-----------|------|----------|
| UT-001 | 正常系 - 成功ケース | 有効な値 | 期待値を返す |
| UT-002 | 異常系 - null入力 | null | 例外をスロー |
| UT-003 | 境界値 - 最小値 | 0 | 正常処理 |
| UT-004 | 境界値 - 最大値 | MAX_INT | 正常処理 |

#### テストコード例
```dart
group('AuthNotifier', () {
  late MockAuthRepository mockRepo;
  late ProviderContainer container;

  setUp(() {
    mockRepo = MockAuthRepository();
    container = ProviderContainer(
      overrides: [authRepositoryProvider.overrideWithValue(mockRepo)],
    );
  });

  tearDown(() => container.dispose());

  test('UT-001: ログイン成功時にユーザーを返す', () async {
    // Arrange
    final expectedUser = User(id: '1', name: 'Test');
    when(() => mockRepo.login(any(), any()))
        .thenAnswer((_) async => expectedUser);

    // Act
    final notifier = container.read(authProvider.notifier);
    await notifier.login('test@example.com', 'password');

    // Assert
    expect(container.read(authProvider).value, expectedUser);
  });

  test('UT-002: ログイン失敗時にエラー状態', () async {
    // Arrange
    when(() => mockRepo.login(any(), any()))
        .thenThrow(AuthException('Invalid credentials'));

    // Act
    final notifier = container.read(authProvider.notifier);
    await notifier.login('invalid@example.com', 'wrong');

    // Assert
    expect(container.read(authProvider).hasError, true);
  });
});
```

---

## 2. ウィジェットテスト

### 2.1 [画面/コンポーネント名]

#### テストケース一覧
| ID | テスト内容 | 操作 | 期待結果 |
|----|-----------|------|----------|
| WT-001 | 初期表示 | - | フォームが表示される |
| WT-002 | バリデーション | 空フォーム送信 | エラーメッセージ表示 |
| WT-003 | ボタン状態 | 有効入力 | ボタン有効化 |
| WT-004 | ローディング | 送信中 | インジケータ表示 |

#### テストコード例
```dart
group('LoginScreen', () {
  testWidgets('WT-001: フォームが正しく表示される', (tester) async {
    await tester.pumpWidget(
      const ProviderScope(
        child: MaterialApp(home: LoginScreen()),
      ),
    );

    expect(find.byType(TextField), findsNWidgets(2));
    expect(find.text('メールアドレス'), findsOneWidget);
    expect(find.text('パスワード'), findsOneWidget);
    expect(find.byType(ElevatedButton), findsOneWidget);
  });

  testWidgets('WT-002: 空フォームでエラー表示', (tester) async {
    await tester.pumpWidget(
      const ProviderScope(
        child: MaterialApp(home: LoginScreen()),
      ),
    );

    await tester.tap(find.byType(ElevatedButton));
    await tester.pump();

    expect(find.text('メールアドレスを入力してください'), findsOneWidget);
  });
});
```

---

## 3. 統合テスト

### 3.1 [フロー名]

#### テストケース一覧
| ID | テスト内容 | 前提条件 | 操作手順 | 期待結果 |
|----|-----------|----------|----------|----------|
| IT-001 | ログインフロー | 未ログイン | 1. メール入力→2. パスワード入力→3. ログインタップ | ホーム画面表示 |
| IT-002 | ログアウトフロー | ログイン済み | 1. メニュー開く→2. ログアウトタップ | ログイン画面表示 |

#### テストコード例
```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('ログインフロー', () {
    testWidgets('IT-001: 正常ログイン→ホーム画面表示', (tester) async {
      app.main();
      await tester.pumpAndSettle();

      // ログイン画面確認
      expect(find.text('ログイン'), findsOneWidget);

      // 認証情報入力
      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );

      // ログイン実行
      await tester.tap(find.text('ログイン'));
      await tester.pumpAndSettle();

      // ホーム画面確認
      expect(find.text('ホーム'), findsOneWidget);
    });
  });
}
```

---

## 4. テスト実行コマンド

```bash
# 全テスト実行
flutter test

# カバレッジ付き
flutter test --coverage

# 特定ファイル
flutter test test/unit/auth_notifier_test.dart

# 統合テスト
flutter test integration_test/app_test.dart
```

---

## 5. カバレッジ目標

| 種別 | 目標 | 現在 |
|------|------|------|
| ユニットテスト | 80% | -% |
| ウィジェットテスト | 70% | -% |
| 統合テスト | 主要フロー | -件 |
