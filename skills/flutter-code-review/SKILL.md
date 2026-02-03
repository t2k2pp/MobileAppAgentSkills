---
name: flutter-code-review
description: Flutterコードレビュースキル。コード品質、パフォーマンス、セキュリティ、アーキテクチャ準拠の観点からレビュー。プルリクエストレビュー、コード品質向上時に使用。
---

# Flutterコードレビュースキル

## レビュー実行手順

1. `flutter analyze` で静的解析エラー確認
2. `git diff` で変更内容確認
3. 以下のカテゴリ別にレビュー実行
4. フィードバックを優先度順に整理

## レビュー観点

### 1. 重大（CRITICAL）- 即時修正必須

#### セキュリティ問題
```dart
// ❌ ハードコードされた認証情報
const apiKey = 'sk-xxxxxxxxxxxxx';

// ✅ 環境変数から取得
final apiKey = const String.fromEnvironment('API_KEY');
```

```dart
// ❌ 平文でトークン保存
SharedPreferences.setString('token', token);

// ✅ Secure Storage使用
FlutterSecureStorage().write(key: 'token', value: token);
```

#### メモリリーク
```dart
// ❌ dispose漏れ
class _MyState extends State<MyWidget> {
  final controller = TextEditingController();
  // dispose未実装
}

// ✅ 適切なdispose
@override
void dispose() {
  controller.dispose();
  super.dispose();
}
```

### 2. 高（HIGH）- リリース前に修正

#### パフォーマンス問題
```dart
// ❌ build内で重い処理
Widget build(BuildContext context) {
  final items = sortAndFilter(rawData); // 毎回実行
  return ListView.builder(...);
}

// ✅ 状態管理で事前計算
final itemsProvider = Provider((ref) {
  final rawData = ref.watch(rawDataProvider);
  return sortAndFilter(rawData);
});
```

```dart
// ❌ 無駄な再ビルド
return Column(
  children: [
    Text(DateTime.now().toString()), // 毎フレーム更新
  ],
);

// ✅ 必要な場合のみ更新
return Column(
  children: [
    const StaticHeader(),
    DynamicContent(),
  ],
);
```

#### 型安全性
```dart
// ❌ dynamic使用
dynamic parseData(Map json) => json['data'];

// ✅ 型指定
T? parseData<T>(Map<String, dynamic> json, String key) => json[key] as T?;
```

### 3. 中（MEDIUM）- 改善推奨

#### コード品質
```dart
// ❌ 長いメソッド（50行以上）
void processUser() {
  // 100行のコード...
}

// ✅ 責務分割
void processUser() {
  validateInput();
  transformData();
  saveToDatabase();
}
```

```dart
// ❌ マジックナンバー
if (items.length > 10) { ... }

// ✅ 定数化
const int maxVisibleItems = 10;
if (items.length > maxVisibleItems) { ... }
```

#### 命名規則
```dart
// ❌ 不明確な命名
final d = getData();
final tmp = process(d);

// ✅ 意図が明確な命名
final userProfile = fetchUserProfile();
final formattedProfile = formatForDisplay(userProfile);
```

### 4. 低（LOW）- 検討事項

#### ドキュメント
```dart
// ❌ 複雑なロジックにコメントなし
int calculate(int a, int b, int c) => (a * b + c) ~/ (a - b);

// ✅ ドキュメントコメント
/// 加重平均を計算する
/// [a] 重み, [b] 値, [c] オフセット
int calculateWeightedAverage(int weight, int value, int offset) =>
    (weight * value + offset) ~/ (weight - value);
```

## レビューチェックリスト

### アーキテクチャ
- [ ] Clean Architectureの層分離が守られている
- [ ] 依存関係の方向が正しい（外→内）
- [ ] 適切なディレクトリ構造に配置されている

### 状態管理
- [ ] Providerが適切に定義されている
- [ ] 不要な再ビルドが発生しない
- [ ] AsyncValueが正しくハンドリングされている

### UI
- [ ] constコンストラクタが使用されている
- [ ] Widget分割が適切
- [ ] アクセシビリティ（Semantics）が設定されている

### テスト
- [ ] 新規コードにテストがある
- [ ] エッジケースがテストされている
- [ ] モックが適切に使用されている

### セキュリティ
- [ ] ハードコードされた秘密情報がない
- [ ] 入力値バリデーションがある
- [ ] 機密データはSecure Storageに保存

## レビューコメントフォーマット

```markdown
## コードレビュー結果

**ファイル:** `lib/features/auth/login_screen.dart`
**レビュアー:** flutter-reviewer
**判定:** ⚠️ 修正後承認

### CRITICAL（即時修正）
なし

### HIGH（リリース前修正）
- **L42:** パフォーマンス - build内でのリスト処理を状態管理に移動
  ```dart
  // 現状
  final sorted = items..sort();
  // 推奨
  final sorted = ref.watch(sortedItemsProvider);
  ```

### MEDIUM（改善推奨）
- **L15-30:** 複雑なWidget - 分割を検討
- **L55:** マジックナンバー `100` → 定数化

### LOW（検討）
- **L8:** publicメソッドにdocコメント追加

---
レビュー完了: 2026-02-04
```

## 自動レビューツール

### analysis_options.yaml
```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    - prefer_const_constructors
    - prefer_const_declarations
    - avoid_print
    - avoid_dynamic_calls
    - prefer_final_locals
    - sort_constructors_first
    - unawaited_futures

analyzer:
  errors:
    missing_return: error
    dead_code: warning
```

### 実行コマンド
```bash
# 静的解析
flutter analyze

# フォーマット確認
dart format --set-exit-if-changed .

# メトリクス（dart_code_metrics使用時）
dart run dart_code_metrics:metrics analyze lib
```
