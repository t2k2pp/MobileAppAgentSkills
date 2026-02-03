---
name: flutter-analytics
description: Flutter分析・監視スキル。Firebase Analytics、Crashlytics、パフォーマンス監視、カスタムイベント設計を支援。アプリ分析基盤構築、クラッシュ監視設定時に使用。
---

# Flutter 分析・監視スキル

## Firebase セットアップ

### 依存関係
```yaml
dependencies:
  firebase_core: ^2.27.0
  firebase_analytics: ^10.8.0
  firebase_crashlytics: ^3.4.0
  firebase_performance: ^0.9.3
```

### 初期化
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  
  // Crashlytics有効化
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };
  
  runApp(const MyApp());
}
```

---

## Firebase Analytics

### 基本イベント
```dart
class AnalyticsService {
  final FirebaseAnalytics _analytics = FirebaseAnalytics.instance;

  // 画面遷移
  Future<void> logScreenView(String screenName) async {
    await _analytics.logScreenView(screenName: screenName);
  }

  // ログイン
  Future<void> logLogin(String method) async {
    await _analytics.logLogin(loginMethod: method);
  }

  // 購入
  Future<void> logPurchase({
    required String itemId,
    required double price,
    required String currency,
  }) async {
    await _analytics.logPurchase(
      currency: currency,
      value: price,
      items: [
        AnalyticsEventItem(
          itemId: itemId,
          price: price,
        ),
      ],
    );
  }
}
```

### カスタムイベント
```dart
// カスタムイベント設計
Future<void> logFeatureUsed(String featureName, {Map<String, Object>? params}) async {
  await _analytics.logEvent(
    name: 'feature_used',
    parameters: {
      'feature_name': featureName,
      ...?params,
    },
  );
}

// 使用例
analyticsService.logFeatureUsed('dark_mode_toggle', params: {'enabled': true});
```

### ユーザープロパティ
```dart
// ユーザー属性設定
Future<void> setUserProperties({
  required String userId,
  String? userType,
  String? subscriptionPlan,
}) async {
  await _analytics.setUserId(id: userId);
  
  if (userType != null) {
    await _analytics.setUserProperty(name: 'user_type', value: userType);
  }
  if (subscriptionPlan != null) {
    await _analytics.setUserProperty(name: 'subscription_plan', value: subscriptionPlan);
  }
}
```

---

## Firebase Crashlytics

### 非致命的エラー記録
```dart
class CrashlyticsService {
  final FirebaseCrashlytics _crashlytics = FirebaseCrashlytics.instance;

  Future<void> recordError(
    dynamic exception,
    StackTrace? stack, {
    String? reason,
    bool fatal = false,
  }) async {
    await _crashlytics.recordError(
      exception,
      stack,
      reason: reason,
      fatal: fatal,
    );
  }

  // カスタムキー設定
  Future<void> setCustomKey(String key, dynamic value) async {
    await _crashlytics.setCustomKey(key, value);
  }

  // ユーザーID設定
  Future<void> setUserId(String userId) async {
    await _crashlytics.setUserIdentifier(userId);
  }

  // ログ追加
  Future<void> log(String message) async {
    await _crashlytics.log(message);
  }
}
```

### エラーハンドリング統合
```dart
// Riverpodプロバイダーでエラー処理
@riverpod
class UserNotifier extends _$UserNotifier {
  @override
  FutureOr<User?> build() async {
    return null;
  }

  Future<void> login(String email, String password) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      try {
        return await ref.read(authRepositoryProvider).login(email, password);
      } catch (e, stack) {
        // Crashlyticsに記録
        await ref.read(crashlyticsServiceProvider).recordError(
          e,
          stack,
          reason: 'Login failed',
        );
        rethrow;
      }
    });
  }
}
```

---

## Performance Monitoring

### 自動トレース
```dart
// Firebase Performance自動有効化
// - アプリ起動時間
// - 画面レンダリング
// - HTTPリクエスト（要Dio Interceptor）
```

### カスタムトレース
```dart
import 'package:firebase_performance/firebase_performance.dart';

class PerformanceService {
  Future<T> trace<T>(String name, Future<T> Function() action) async {
    final trace = FirebasePerformance.instance.newTrace(name);
    await trace.start();
    
    try {
      final result = await action();
      trace.putAttribute('status', 'success');
      return result;
    } catch (e) {
      trace.putAttribute('status', 'error');
      trace.putAttribute('error', e.toString().substring(0, 100));
      rethrow;
    } finally {
      await trace.stop();
    }
  }
}

// 使用例
final data = await performanceService.trace('fetch_user_data', () async {
  return await repository.fetchUserData();
});
```

### HTTPメトリクス
```dart
// Dio Interceptor
class PerformanceInterceptor extends Interceptor {
  final Map<RequestOptions, HttpMetric> _metrics = {};

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final metric = FirebasePerformance.instance.newHttpMetric(
      options.uri.toString(),
      HttpMethod.values.firstWhere(
        (m) => m.name.toUpperCase() == options.method.toUpperCase(),
        orElse: () => HttpMethod.Get,
      ),
    );
    metric.start();
    _metrics[options] = metric;
    handler.next(options);
  }

  @override
  void onResponse(Response response, ResponseInterceptorHandler handler) {
    final metric = _metrics.remove(response.requestOptions);
    metric?.httpResponseCode = response.statusCode;
    metric?.responsePayloadSize = response.data.toString().length;
    metric?.stop();
    handler.next(response);
  }
}
```

---

## イベント設計ガイドライン

### 命名規則
```
# スネークケース使用
user_signed_up
item_purchased
feature_used

# 動詞_名詞 形式
clicked_button ❌
button_clicked ✅
```

### 推奨イベント
| カテゴリ | イベント名 | パラメータ |
|---------|-----------|-----------|
| 認証 | login | method |
| 認証 | sign_up | method |
| 機能 | feature_used | feature_name |
| エラー | error_occurred | error_type, screen |
| 購入 | purchase | item_id, value, currency |

### パラメータ制限
- 名前: 40文字以内
- 値: 100文字以内
- イベントあたり25パラメータまで

---

## デバッグ

### デバッグビュー有効化
```bash
# Android
adb shell setprop debug.firebase.analytics.app com.example.app

# iOS
# Schemeに -FIRDebugEnabled 追加
```

### Crashlytics強制クラッシュ
```dart
// テスト用
FirebaseCrashlytics.instance.crash();
```

---

## チェックリスト

分析設定時に確認:
- [ ] Firebase初期化設定
- [ ] Crashlytics Flutter/Platformエラー捕捉
- [ ] 画面遷移イベント設定
- [ ] ユーザーID設定
- [ ] カスタムイベント設計
- [ ] パフォーマンストレース設定
- [ ] デバッグビューで確認
- [ ] プライバシーポリシー更新
