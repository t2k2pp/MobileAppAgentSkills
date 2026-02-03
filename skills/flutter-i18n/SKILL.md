---
name: flutter-i18n
description: Flutter国際化スキル。flutter_localizations設定、ARBファイル管理、動的言語切り替え、RTL対応を支援。多言語アプリ開発時に使用。
---

# Flutter 国際化（i18n）スキル

## セットアップ

### 依存関係
```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.19.0

flutter:
  generate: true
```

### l10n設定
```yaml
# l10n.yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-class: AppLocalizations
```

---

## ARBファイル

### 英語（テンプレート）
```json
// lib/l10n/app_en.arb
{
  "@@locale": "en",
  "appTitle": "My App",
  "@appTitle": {
    "description": "The title of the application"
  },
  "hello": "Hello {name}!",
  "@hello": {
    "description": "A greeting with a name parameter",
    "placeholders": {
      "name": {
        "type": "String",
        "example": "John"
      }
    }
  },
  "itemCount": "{count, plural, =0{No items} =1{1 item} other{{count} items}}",
  "@itemCount": {
    "description": "A plural message",
    "placeholders": {
      "count": {
        "type": "int"
      }
    }
  },
  "price": "Price: {amount}",
  "@price": {
    "placeholders": {
      "amount": {
        "type": "double",
        "format": "currency",
        "optionalParameters": {
          "symbol": "$"
        }
      }
    }
  }
}
```

### 日本語
```json
// lib/l10n/app_ja.arb
{
  "@@locale": "ja",
  "appTitle": "マイアプリ",
  "hello": "こんにちは、{name}さん！",
  "itemCount": "{count, plural, =0{アイテムなし} other{{count}件のアイテム}}",
  "price": "価格: {amount}"
}
```

---

## アプリ設定

### MaterialApp設定
```dart
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

MaterialApp(
  localizationsDelegates: AppLocalizations.localizationsDelegates,
  supportedLocales: AppLocalizations.supportedLocales,
  locale: const Locale('ja'), // デフォルト言語
  home: const MyHomePage(),
)
```

### Riverpodで言語管理
```dart
@riverpod
class LocaleNotifier extends _$LocaleNotifier {
  @override
  Locale build() {
    // SharedPreferencesから読み込み
    return const Locale('ja');
  }

  void setLocale(Locale locale) {
    state = locale;
    // SharedPreferencesに保存
  }
}

// MaterialAppで使用
Consumer(
  builder: (context, ref, _) {
    final locale = ref.watch(localeNotifierProvider);
    return MaterialApp(
      locale: locale,
      // ...
    );
  },
)
```

---

## 使用方法

### 基本
```dart
// コンテキストから取得
final l10n = AppLocalizations.of(context)!;

Text(l10n.appTitle)
Text(l10n.hello('Flutter'))
Text(l10n.itemCount(5))
```

### コンテキストなしで使用
```dart
// Extension
extension AppLocalizationsX on BuildContext {
  AppLocalizations get l10n => AppLocalizations.of(this)!;
}

// 使用
Text(context.l10n.appTitle)
```

---

## RTL（右から左）対応

### 自動対応
```dart
// Directionality自動設定
MaterialApp(
  localizationsDelegates: [...],
  supportedLocales: [
    Locale('en'),
    Locale('ar'), // アラビア語
    Locale('he'), // ヘブライ語
  ],
)
```

### 手動制御
```dart
Directionality(
  textDirection: TextDirection.rtl,
  child: MyWidget(),
)
```

### RTL対応レイアウト
```dart
// ❌ 固定方向
Padding(padding: EdgeInsets.only(left: 16))

// ✅ 論理方向（RTL対応）
Padding(padding: EdgeInsetsDirectional.only(start: 16))

// ❌ 固定位置
Positioned(left: 0, child: ...)

// ✅ 論理位置
PositionedDirectional(start: 0, child: ...)
```

---

## 日付・数値フォーマット

### 日付
```dart
import 'package:intl/intl.dart';

// ロケール対応日付
final dateFormat = DateFormat.yMMMd(locale);
Text(dateFormat.format(DateTime.now()))
// en: Jan 15, 2026
// ja: 2026年1月15日
```

### 数値・通貨
```dart
// 数値
final numberFormat = NumberFormat.decimalPattern(locale);
Text(numberFormat.format(1234567.89))
// en: 1,234,567.89
// ja: 1,234,567.89

// 通貨
final currencyFormat = NumberFormat.currency(locale: locale, symbol: '¥');
Text(currencyFormat.format(1000))
// ¥1,000
```

---

## 翻訳ワークフロー

### 1. 英語ARBをテンプレートとして作成
### 2. 翻訳者に他言語ARBを依頼
### 3. ARBファイルを配置
### 4. コード生成
```bash
flutter gen-l10n
```

### 翻訳管理ツール
- Lokalise
- Crowdin
- POEditor
- Google Sheets（シンプルな場合）

---

## チェックリスト

国際化対応時に確認:
- [ ] flutter_localizations設定済み
- [ ] l10n.yaml作成済み
- [ ] 全文字列がARBファイルに定義
- [ ] 複数形（plural）対応
- [ ] 日付・数値フォーマット対応
- [ ] RTL対応（必要な場合）
- [ ] 言語切り替え機能
- [ ] テストで各言語確認
