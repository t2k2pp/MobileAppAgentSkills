---
name: flutter-performance
description: Flutterパフォーマンス最適化スキル。Impellerレンダリング、起動時間最適化、メモリプロファイリング、フレームドロップ対策を支援。パフォーマンスチューニング、ボトルネック解消時に使用。
---

# Flutter パフォーマンス最適化スキル

## パフォーマンス目標

| 指標 | 目標値 |
|------|--------|
| 起動時間（Cold Start） | < 2秒 |
| フレームレート | 60fps維持 |
| ジャンクフレーム | < 1% |
| メモリ使用量 | 機能に応じて最小化 |
| APKサイズ | 可能な限り小さく |

---

## レンダリング最適化

### Impeller（Flutter 3.10+）
```bash
# iOS: デフォルト有効
# Android: 明示的に有効化
flutter run --enable-impeller
```

### Widgetリビルド最小化
```dart
// ❌ 毎回リビルド
Widget build(BuildContext context) {
  return Column(
    children: [
      Text(DateTime.now().toString()), // 毎フレーム変化
      const ExpensiveWidget(),         // 巻き込まれる
    ],
  );
}

// ✅ 分離してリビルド範囲限定
Widget build(BuildContext context) {
  return Column(
    children: [
      const TimerWidget(),     // 時間表示を分離
      const ExpensiveWidget(), // リビルドされない
    ],
  );
}
```

### RepaintBoundary
```dart
// 再描画範囲を限定
RepaintBoundary(
  child: CustomPaint(
    painter: ExpensivePainter(),
  ),
)
```

### const活用
```dart
// ✅ 全てconst
const Column(
  children: [
    Text('Static'),
    Icon(Icons.star),
    SizedBox(height: 16),
  ],
)
```

---

## ListView最適化

### 基本原則
```dart
// ❌ 全件生成
ListView(
  children: items.map((i) => ItemWidget(i)).toList(),
)

// ✅ 遅延生成
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemWidget(items[index]),
)

// ✅ 固定高さで最適化
ListView.builder(
  itemExtent: 72.0, // 固定高さ
  itemBuilder: (context, index) => ItemWidget(items[index]),
)
```

### Sliver使用
```dart
CustomScrollView(
  slivers: [
    const SliverAppBar(title: Text('Title')),
    SliverList.builder(
      itemBuilder: (context, index) => ItemWidget(items[index]),
    ),
  ],
)
```

### 画像遅延読み込み
```dart
CachedNetworkImage(
  imageUrl: url,
  placeholder: (_, __) => const Shimmer(),
  fadeInDuration: const Duration(milliseconds: 200),
  memCacheWidth: 200, // メモリキャッシュサイズ制限
)
```

---

## 重い処理のIsolate化

### compute関数
```dart
// 重いJSON解析
final result = await compute(parseJson, jsonString);

List<Item> parseJson(String json) {
  final data = jsonDecode(json) as List;
  return data.map((e) => Item.fromJson(e)).toList();
}
```

### Isolate.run
```dart
final result = await Isolate.run(() {
  // 重い画像処理
  return processImage(data);
});
```

### 継続的バックグラウンド処理
```dart
// ReceivePortでIsolate通信
final receivePort = ReceivePort();
await Isolate.spawn(heavyTask, receivePort.sendPort);

receivePort.listen((message) {
  // 結果を受信
});
```

---

## 起動時間最適化

### 遅延初期化
```dart
// ❌ 起動時に全て初期化
void main() {
  initializeHeavyService1();
  initializeHeavyService2();
  runApp(const MyApp());
}

// ✅ 必要時に初期化
late final HeavyService _service;

Future<HeavyService> getService() async {
  return _service ??= await HeavyService.initialize();
}
```

### Deferred Loading
```dart
// 遅延ロード（Web向け）
import 'package:heavy_feature/heavy_feature.dart' deferred as heavy;

Future<void> loadFeature() async {
  await heavy.loadLibrary();
  heavy.showFeature();
}
```

### Splash最適化
```xml
<!-- android/app/src/main/res/drawable/launch_background.xml -->
<!-- ネイティブスプラッシュで体感速度向上 -->
```

---

## メモリ最適化

### 画像メモリ管理
```dart
// メモリキャッシュクリア
PaintingBinding.instance.imageCache.clear();
PaintingBinding.instance.imageCache.clearLiveImages();

// 適切なサイズで読み込み
Image.network(
  url,
  cacheWidth: 200,  // デバイスピクセル考慮
  cacheHeight: 200,
)
```

### dispose徹底
```dart
@override
void dispose() {
  _controller.dispose();
  _subscription.cancel();
  _animationController.dispose();
  super.dispose();
}
```

### メモリリーク検出
```dart
// DevTools Memory Viewで確認
// 1. Snapshot取得
// 2. 操作を繰り返す
// 3. 再度Snapshot
// 4. 増加オブジェクトを確認
```

---

## ビルドサイズ最適化

### Tree Shaking
```bash
# 未使用コード除去（デフォルト有効）
flutter build apk --release
```

### 難読化
```bash
flutter build apk --release --obfuscate --split-debug-info=build/debug-info
```

### アイコン・画像最適化
- WebP形式使用
- 適切な解像度のみ含める
- flutter_launcher_icons使用

### APK分割
```bash
flutter build apk --release --split-per-abi
# arm64-v8a, armeabi-v7a, x86_64 別々に生成
```

---

## プロファイリング

### Performance Overlay
```dart
MaterialApp(
  showPerformanceOverlay: true, // デバッグ時
)
```

### DevTools使用
```bash
flutter pub global activate devtools
flutter pub global run devtools

# Profile modeで実行
flutter run --profile
```

### Timeline記録
```dart
import 'dart:developer';

Timeline.startSync('expensive_operation');
// 処理
Timeline.finishSync();
```

---

## チェックリスト

パフォーマンス確認:
- [ ] ListView.builderで遅延構築
- [ ] const可能な箇所でconst使用
- [ ] 重い処理をIsolateで実行
- [ ] 画像にキャッシュとサイズ制限
- [ ] dispose漏れなし
- [ ] Profile modeで60fps維持
- [ ] ジャンクフレーム1%未満
- [ ] APKサイズ最適化
