---
name: flutter-ci-cd
description: Flutter CI/CDスキル。GitHub Actions/Codemagicによる自動ビルド、テスト、Play Store/App Store自動デプロイ、Fastlane統合を支援。リリース自動化、パイプライン構築時に使用。
---

# Flutter CI/CD スキル

## CI/CDパイプライン概要

```
Push/PR → Lint → Test → Build → Deploy
    ↓
 ┌─────────────────────────────────────┐
 │  CI: GitHub Actions / Codemagic     │
 │  CD: Fastlane + Store API           │
 └─────────────────────────────────────┘
```

---

## GitHub Actions設定

### 基本ワークフロー
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
          channel: 'stable'
          cache: true
      
      - name: Install dependencies
        run: flutter pub get
      
      - name: Generate code
        run: dart run build_runner build --delete-conflicting-outputs
      
      - name: Analyze
        run: flutter analyze --fatal-infos
      
      - name: Format check
        run: dart format --set-exit-if-changed .

  test:
    runs-on: ubuntu-latest
    needs: analyze
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
          cache: true
      
      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      
      - name: Run tests
        run: flutter test --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          file: coverage/lcov.info

  build-android:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
          cache: true
      
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      
      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      
      - name: Build APK
        run: flutter build apk --release
      
      - name: Build App Bundle
        run: flutter build appbundle --release
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: android-release
          path: |
            build/app/outputs/flutter-apk/app-release.apk
            build/app/outputs/bundle/release/app-release.aab
```

### iOS ビルド
```yaml
  build-ios:
    runs-on: macos-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
          cache: true
      
      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      
      - name: Build iOS
        run: flutter build ipa --release --no-codesign
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: ios-release
          path: build/ios/ipa/*.ipa
```

---

## Fastlane統合

### セットアップ
```bash
# Android
cd android
fastlane init

# iOS
cd ios
fastlane init
```

### Android Fastfile
```ruby
# android/fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Deploy to Play Store internal track"
  lane :internal do
    upload_to_play_store(
      track: 'internal',
      aab: '../build/app/outputs/bundle/release/app-release.aab',
      json_key_data: ENV['PLAY_STORE_JSON_KEY']
    )
  end

  desc "Promote internal to production"
  lane :promote_to_production do
    upload_to_play_store(
      track: 'internal',
      track_promote_to: 'production',
      json_key_data: ENV['PLAY_STORE_JSON_KEY']
    )
  end
end
```

### iOS Fastfile
```ruby
# ios/fastlane/Fastfile
default_platform(:ios)

platform :ios do
  desc "Deploy to TestFlight"
  lane :beta do
    setup_ci if ENV['CI']
    
    match(type: "appstore", readonly: true)
    
    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store"
    )
    
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
  end
end
```

---

## 署名設定

### Android署名
```properties
# android/key.properties (gitignore対象)
storePassword=<password>
keyPassword=<password>
keyAlias=<alias>
storeFile=<path-to-keystore>
```

```groovy
// android/app/build.gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

### iOS Match
```bash
# 証明書管理（Match）
fastlane match init
fastlane match appstore
```

---

## 環境変数管理

### GitHub Secrets
```
PLAY_STORE_JSON_KEY    # Google Play Service Account JSON
KEYSTORE_BASE64        # Android keystore (base64)
KEY_PROPERTIES         # key.properties内容
MATCH_PASSWORD         # iOS Match暗号化パスワード
APP_STORE_CONNECT_API_KEY  # App Store Connect API Key
```

### Secrets復元 (Android)
```yaml
- name: Decode keystore
  run: |
    echo ${{ secrets.KEYSTORE_BASE64 }} | base64 -d > android/app/keystore.jks
    echo "${{ secrets.KEY_PROPERTIES }}" > android/key.properties
```

---

## バージョン管理

### 自動バージョンインクリメント
```yaml
- name: Get version
  id: version
  run: |
    VERSION=$(grep 'version:' pubspec.yaml | sed 's/version: //')
    echo "version=$VERSION" >> $GITHUB_OUTPUT

- name: Increment build number
  run: |
    BUILD_NUMBER=${{ github.run_number }}
    sed -i "s/version: .*/version: ${{ steps.version.outputs.version }}+$BUILD_NUMBER/" pubspec.yaml
```

---

## チェックリスト

CI/CD構築時に確認:
- [ ] GitHub Actions/Codemagicワークフロー設定
- [ ] Android署名キー設定
- [ ] iOS証明書・プロビジョニング設定
- [ ] Secretsに機密情報を保存
- [ ] Fastlane設定
- [ ] 内部テスト→本番の段階的リリース設定
