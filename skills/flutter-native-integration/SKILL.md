---
name: flutter-native-integration
description: Flutterネイティブ連携スキル。Platform Channel、Method Channel、EventChannel、Dart FFI、Pigeon自動生成を支援。カメラ、センサー等のネイティブ機能連携時に使用。
---

# Flutter ネイティブ連携スキル

## 連携方式の選択

| 方式 | 用途 | 推奨度 |
|------|------|--------|
| **Pigeon** | 型安全なメソッド呼び出し | ★★★ 推奨 |
| **Method Channel** | シンプルなメソッド呼び出し | ★★ |
| **Event Channel** | ネイティブからのストリーム | ★★ |
| **Dart FFI** | C/C++ライブラリ直接呼び出し | 特殊用途 |

---

## Pigeon（推奨）

### セットアップ
```yaml
dev_dependencies:
  pigeon: ^17.0.0
```

### Pigeon定義
```dart
// pigeons/messages.dart
import 'package:pigeon/pigeon.dart';

@ConfigurePigeon(PigeonOptions(
  dartOut: 'lib/src/messages.g.dart',
  kotlinOut: 'android/app/src/main/kotlin/com/example/Messages.g.kt',
  swiftOut: 'ios/Runner/Messages.g.swift',
))

class DeviceInfo {
  String? model;
  String? osVersion;
  int? batteryLevel;
}

@HostApi()
abstract class DeviceApi {
  DeviceInfo getDeviceInfo();
  @async
  bool requestPermission(String permission);
}

@FlutterApi()
abstract class DeviceEventApi {
  void onBatteryLevelChanged(int level);
}
```

### コード生成
```bash
dart run pigeon --input pigeons/messages.dart
```

### Kotlin実装
```kotlin
// android/app/src/main/kotlin/.../Messages.g.kt から実装
class DeviceApiImpl(private val context: Context) : DeviceApi {
    override fun getDeviceInfo(): DeviceInfo {
        return DeviceInfo(
            model = Build.MODEL,
            osVersion = Build.VERSION.RELEASE,
            batteryLevel = getBatteryLevel()
        )
    }

    override fun requestPermission(permission: String, callback: (Result<Boolean>) -> Unit) {
        // 非同期処理
        callback(Result.success(true))
    }
}
```

### Swift実装
```swift
// ios/Runner/Messages.g.swift から実装
class DeviceApiImpl: DeviceApi {
    func getDeviceInfo() throws -> DeviceInfo {
        return DeviceInfo(
            model: UIDevice.current.model,
            osVersion: UIDevice.current.systemVersion,
            batteryLevel: Int64(UIDevice.current.batteryLevel * 100)
        )
    }
    
    func requestPermission(permission: String, completion: @escaping (Result<Bool, Error>) -> Void) {
        completion(.success(true))
    }
}
```

### Dart使用
```dart
final deviceApi = DeviceApi();
final info = await deviceApi.getDeviceInfo();
print('Model: ${info.model}');
```

---

## Method Channel

### Dart側
```dart
class NativeBridge {
  static const _channel = MethodChannel('com.example.app/native');

  Future<String> getPlatformVersion() async {
    final version = await _channel.invokeMethod<String>('getPlatformVersion');
    return version ?? 'Unknown';
  }

  Future<Map<String, dynamic>> getDeviceInfo() async {
    final result = await _channel.invokeMapMethod<String, dynamic>('getDeviceInfo');
    return result ?? {};
  }
}
```

### Kotlin側
```kotlin
class MainActivity : FlutterActivity() {
    private val CHANNEL = "com.example.app/native"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)
        
        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getPlatformVersion" -> result.success("Android ${android.os.Build.VERSION.RELEASE}")
                    "getDeviceInfo" -> result.success(mapOf(
                        "model" to Build.MODEL,
                        "brand" to Build.BRAND
                    ))
                    else -> result.notImplemented()
                }
            }
    }
}
```

### Swift側
```swift
@UIApplicationMain
class AppDelegate: FlutterAppDelegate {
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        let controller = window?.rootViewController as! FlutterViewController
        let channel = FlutterMethodChannel(
            name: "com.example.app/native",
            binaryMessenger: controller.binaryMessenger
        )
        
        channel.setMethodCallHandler { call, result in
            switch call.method {
            case "getPlatformVersion":
                result("iOS \(UIDevice.current.systemVersion)")
            case "getDeviceInfo":
                result([
                    "model": UIDevice.current.model,
                    "name": UIDevice.current.name
                ])
            default:
                result(FlutterMethodNotImplemented)
            }
        }
        
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
}
```

---

## Event Channel

### Dart側
```dart
class BatteryMonitor {
  static const _eventChannel = EventChannel('com.example.app/battery');

  Stream<int> get batteryLevel {
    return _eventChannel.receiveBroadcastStream().map((event) => event as int);
  }
}
```

### Kotlin側
```kotlin
class BatteryStreamHandler(private val context: Context) : EventChannel.StreamHandler {
    private var receiver: BroadcastReceiver? = null

    override fun onListen(arguments: Any?, events: EventChannel.EventSink?) {
        receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context?, intent: Intent?) {
                val level = intent?.getIntExtra(BatteryManager.EXTRA_LEVEL, -1) ?: -1
                events?.success(level)
            }
        }
        context.registerReceiver(receiver, IntentFilter(Intent.ACTION_BATTERY_CHANGED))
    }

    override fun onCancel(arguments: Any?) {
        receiver?.let { context.unregisterReceiver(it) }
        receiver = null
    }
}
```

---

## Dart FFI

### セットアップ
```yaml
dependencies:
  ffi: ^2.1.0
  
# native_add.cではなくDynamic Libraryをロード
```

### C関数呼び出し
```dart
import 'dart:ffi';

typedef NativeAdd = Int32 Function(Int32, Int32);
typedef DartAdd = int Function(int, int);

void main() {
  final dylib = DynamicLibrary.open('libnative.so');
  final add = dylib.lookupFunction<NativeAdd, DartAdd>('add');
  print(add(1, 2)); // 3
}
```

---

## ベストプラクティス

### 1. Pigeon優先
- 型安全
- ボイラープレート削減
- ドキュメント自動生成

### 2. エラーハンドリング
```dart
try {
  final result = await channel.invokeMethod('method');
} on PlatformException catch (e) {
  // ネイティブ側エラー
  print('Error: ${e.code} - ${e.message}');
} on MissingPluginException {
  // チャンネル未登録
  print('Plugin not available');
}
```

### 3. テスト
```dart
// モック
TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
    .setMockMethodCallHandler(channel, (call) async {
  if (call.method == 'getPlatformVersion') {
    return 'Mock Platform';
  }
  return null;
});
```

---

## チェックリスト

ネイティブ連携実装時:
- [ ] Pigeon使用を優先検討
- [ ] 両プラットフォーム（iOS/Android）実装
- [ ] エラーハンドリング実装
- [ ] null安全性考慮
- [ ] バックグラウンド処理の考慮
- [ ] テスト（モック）実装
