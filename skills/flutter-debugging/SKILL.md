---
name: flutter-debugging
description: Flutterバグ修正スキル。DevToolsによるデバッグ、よくあるエラーパターンと解決策、パフォーマンス問題特定、クラッシュ分析を支援。バグ修正、パフォーマンス改善、障害対応時に使用。
---

# Flutterバグ修正スキル

## デバッグワークフロー

```
1. 問題の再現確認
2. エラーログ・スタックトレース収集
3. 根本原因の特定
4. 修正の実装
5. テストで検証
6. リグレッションテスト
```

## Flutter DevTools

### 起動方法
```bash
# アプリ実行中に
flutter pub global activate devtools
flutter pub global run devtools

# または VS Code/Android Studio のDevToolsパネル
```

### 主要機能

#### Widget Inspector
- Widget ツリー表示
- プロパティ確認
- レイアウト問題の特定

#### Performance View
- フレームレート監視
- ジャンク検出
- ビルド時間分析

#### Memory View
- メモリ使用量監視
- リーク検出
- ガベージコレクション分析

## よくあるエラーと解決策

### 1. RenderFlex overflowed
```dart
// エラー: A RenderFlex overflowed by X pixels

// ❌ 問題コード
Row(
  children: [
    Text('Very long text that does not fit'),
    Icon(Icons.star),
  ],
)

// ✅ 解決策1: Expanded/Flexible使用
Row(
  children: [
    Expanded(child: Text('Long text', overflow: TextOverflow.ellipsis)),
    Icon(Icons.star),
  ],
)

// ✅ 解決策2: SingleChildScrollView
SingleChildScrollView(
  scrollDirection: Axis.horizontal,
  child: Row(...),
)
```

### 2. setState() called after dispose()
```dart
// ❌ 問題コード
Future<void> loadData() async {
  final data = await api.fetch();
  setState(() => _data = data); // dispose後に呼ばれる可能性
}

// ✅ 解決策: mountedチェック
Future<void> loadData() async {
  final data = await api.fetch();
  if (mounted) {
    setState(() => _data = data);
  }
}

// ✅ 推奨: Riverpodで状態管理（Widgetのライフサイクルに依存しない）
```

### 3. Null check operator on null value
```dart
// ❌ 問題コード
final name = user!.name; // userがnullの可能性

// ✅ 解決策1: nullチェック
final name = user?.name ?? 'Unknown';

// ✅ 解決策2: 早期リターン
if (user == null) return const EmptyState();
final name = user.name;
```

### 4. Looking up deactivated widget's ancestor
```dart
// ❌ 問題コード（async後にcontext使用）
onPressed: () async {
  await doSomething();
  Navigator.of(context).pop(); // contextが無効の可能性
}

// ✅ 解決策: context使用前にチェック
onPressed: () async {
  await doSomething();
  if (context.mounted) {
    Navigator.of(context).pop();
  }
}
```

### 5. Invalid constant value
```dart
// ❌ 問題コード
const myWidget = Text(variable); // 変数はconstにできない

// ✅ 解決策
final myWidget = Text(variable);
// または定数のみ使用
const myWidget = Text('Static text');
```

## パフォーマンス問題の特定

### フレームドロップ検出
```dart
// Timeline記録
import 'dart:developer';
Timeline.startSync('expensive_operation');
// 処理
Timeline.finishSync();

// プロファイルモードで実行
// flutter run --profile
```

### 不要な再ビルド特定
```dart
// デバッグ用ウィジェット
class RebuildCounter extends StatefulWidget {
  @override
  _RebuildCounterState createState() => _RebuildCounterState();
}

class _RebuildCounterState extends State<RebuildCounter> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    _count++;
    debugPrint('Rebuild count: $_count');
    return child;
  }
}
```

### メモリリーク検出
```dart
// DevToolsのMemory Viewで:
// 1. Snapshotを取得
// 2. 操作を繰り返す
// 3. 再度Snapshotを取得
// 4. 増加しているオブジェクトを確認

// よくある原因:
// - StreamSubscription.cancel()忘れ
// - AnimationController.dispose()忘れ
// - Timer.cancel()忘れ
```

## クラッシュ分析

### Firebase Crashlytics統合
```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();

  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(const MyApp());
}
```

### カスタムログ
```dart
// 重要な操作をログ
FirebaseCrashlytics.instance.log('User started checkout');

// ユーザー情報（デバッグ用）
FirebaseCrashlytics.instance.setUserIdentifier(userId);
FirebaseCrashlytics.instance.setCustomKey('cart_items', itemCount);
```

## デバッグコマンド

```bash
# ログ確認
flutter logs

# 詳細ログ
flutter run -v

# リリースビルドのデバッグ
flutter run --release --enable-software-rendering

# 特定デバイス
flutter run -d <device_id>

# クリーンビルド
flutter clean && flutter pub get && flutter run
```

## デバッグチェックリスト

問題調査時に確認:
- [ ] エラーメッセージとスタックトレースを確認
- [ ] 最近の変更をgit diffで確認
- [ ] 再現手順を明確化
- [ ] DevToolsでWidget/Performance/Memory確認
- [ ] 関連するProviderの状態を確認
- [ ] ネットワークリクエスト/レスポンスを確認
- [ ] プラットフォーム固有の問題か確認

## 修正後の検証

```bash
# 全テスト実行
flutter test

# 静的解析
flutter analyze

# 該当機能のintegration test
flutter test integration_test/feature_test.dart

# リグレッションテスト
flutter test test/regression/
```
