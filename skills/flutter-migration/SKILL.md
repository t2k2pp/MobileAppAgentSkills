---
name: flutter-migration
description: Flutterマイグレーションスキル。Flutterバージョンアップグレード、非推奨API対応、Provider→Riverpod移行、レガシーコードリファクタリングを支援。
---

# Flutter マイグレーションスキル

## バージョンアップグレード

### アップグレード手順
```bash
# 1. 現在のバージョン確認
flutter --version
flutter doctor

# 2. Flutter更新
flutter upgrade

# 3. 依存関係更新
flutter pub upgrade --major-versions

# 4. 非推奨警告確認
flutter analyze

# 5. テスト実行
flutter test
```

### 破壊的変更確認
```bash
# リリースノート確認
https://docs.flutter.dev/release/breaking-changes
```

---

## よくあるマイグレーション

### Provider → Riverpod 3.0

#### 依存関係変更
```yaml
# Before
dependencies:
  provider: ^6.0.0

# After
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0

dev_dependencies:
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
```

#### ChangeNotifier → Notifier
```dart
// Before: Provider
class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  
  void increment() {
    _count++;
    notifyListeners();
  }
}

// After: Riverpod 3.0
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;
  
  void increment() {
    state++;
  }
}
```

#### Consumer変更
```dart
// Before: Provider
Consumer<CounterModel>(
  builder: (context, model, child) {
    return Text('${model.count}');
  },
)

// After: Riverpod
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  },
)
```

#### FutureBuilder → FutureProvider
```dart
// Before
FutureBuilder<User>(
  future: fetchUser(),
  builder: (context, snapshot) {
    if (snapshot.connectionState == ConnectionState.waiting) {
      return CircularProgressIndicator();
    }
    if (snapshot.hasError) {
      return Text('Error: ${snapshot.error}');
    }
    return Text(snapshot.data!.name);
  },
)

// After
@riverpod
Future<User> user(UserRef ref) async {
  return await fetchUser();
}

// 使用
ref.watch(userProvider).when(
  data: (user) => Text(user.name),
  loading: () => CircularProgressIndicator(),
  error: (e, s) => Text('Error: $e'),
)
```

---

### Navigator 1.0 → go_router

#### セットアップ
```yaml
dependencies:
  go_router: ^13.0.0
```

#### ルート定義
```dart
// Before: Navigator 1.0
Navigator.push(
  context,
  MaterialPageRoute(builder: (_) => DetailScreen(id: '123')),
);

// After: go_router
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const HomeScreen(),
    ),
    GoRoute(
      path: '/detail/:id',
      builder: (context, state) {
        final id = state.pathParameters['id']!;
        return DetailScreen(id: id);
      },
    ),
  ],
);

// 使用
context.go('/detail/123');
// または
context.push('/detail/123');
```

---

### Null Safety移行

#### 段階的移行
```bash
# 1. 依存関係のnull safety確認
dart pub outdated --mode=null-safety

# 2. 移行ツール実行
dart migrate

# 3. 手動修正
# - ! の使用を最小限に
# - late の使用を最小限に
```

#### よくあるパターン
```dart
// Before
String name;

// After - オプショナル
String? name;

// After - 必須（late使用は最終手段）
late final String name;

// After - デフォルト値
String name = '';
```

---

### 非推奨API対応

#### MaterialButton → ElevatedButton等
```dart
// Before (deprecated)
RaisedButton(onPressed: () {}, child: Text('Click'))
FlatButton(onPressed: () {}, child: Text('Click'))

// After
ElevatedButton(onPressed: () {}, child: Text('Click'))
TextButton(onPressed: () {}, child: Text('Click'))
OutlinedButton(onPressed: () {}, child: Text('Click'))
```

#### WillPopScope → PopScope
```dart
// Before (deprecated in 3.12)
WillPopScope(
  onWillPop: () async => true,
  child: Scaffold(...),
)

// After
PopScope(
  canPop: true,
  onPopInvokedWithResult: (didPop, result) {
    if (didPop) return;
    // カスタム処理
  },
  child: Scaffold(...),
)
```

---

## リファクタリング戦略

### 段階的アプローチ
1. **テスト追加**: 既存機能にテスト追加
2. **依存関係更新**: 1つずつ更新
3. **警告解消**: deprecated警告を順次対応
4. **パターン統一**: 新パターンに統一

### 大規模移行
```
Week 1: 調査・計画
  - 現状分析
  - 影響範囲特定
  - マイグレーション計画作成

Week 2-3: インフラ整備
  - 新パッケージ追加
  - 共通コンポーネント移行

Week 4+: 機能別移行
  - 機能単位で移行
  - 各機能完了後にテスト
```

---

## チェックリスト

マイグレーション時に確認:
- [ ] 現状のFlutter/Dartバージョン確認
- [ ] 破壊的変更リスト確認
- [ ] テストカバレッジ確認
- [ ] 依存パッケージの互換性確認
- [ ] 段階的移行計画作成
- [ ] 各ステップでテスト実行
- [ ] 本番環境へ段階的リリース

### 禁止事項
- ❌ テストなしでの大規模移行
- ❌ 複数の大きな変更を同時実施
- ❌ 非推奨警告を無視し続ける
