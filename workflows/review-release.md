---
description: レビューからリリースへの移行フロー。PRレビュー、修正、マージ、リリース準備のガイド。
---

# レビュー→リリースフロー

## フロー概要

```
Pull Request作成
    ↓
1. 自動チェック実行
    ↓
2. コードレビュー
    ↓
3. 修正対応
    ↓
4. マージ
    ↓
5. リリース準備
    ↓
6. リリース
```

---

## Step 1: Pull Request作成

### PR作成前チェック
```bash
# フォーマット
dart format .

# 静的解析
flutter analyze

# テスト実行
flutter test

# カバレッジ確認
flutter test --coverage
```

### PRテンプレート
```markdown
## 概要
[変更の概要]

## 変更内容
- [変更1]
- [変更2]

## テスト
- [ ] ユニットテスト追加
- [ ] ウィジェットテスト追加
- [ ] 手動テスト実施

## スクリーンショット（UI変更時）
[Before/After]

## チェックリスト
- [ ] flutter analyze 通過
- [ ] flutter test 通過
- [ ] ドキュメント更新
```

---

## Step 2: 自動チェック

### GitHub Actions
```yaml
name: PR Check
on: [pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test --coverage
```

### 必須チェック
- [ ] `flutter analyze` 通過
- [ ] `flutter test` 通過
- [ ] カバレッジ閾値達成

---

## Step 3: コードレビュー

### レビュアー
- `flutter-reviewer`（自動）
- チームメンバー（手動）

### レビュー観点
1. **CRITICAL**: セキュリティ、メモリリーク
2. **HIGH**: パフォーマンス、型安全性
3. **MEDIUM**: コード品質、命名
4. **LOW**: ドキュメント、スタイル

### 承認条件
- CRITICAL問題なし
- HIGH問題なし
- 1名以上の承認

---

## Step 4: 修正対応

### 指摘対応
```bash
# 修正コミット
git add .
git commit -m "fix: レビュー指摘対応 - [内容]"
git push

# または fixup commit
git commit --fixup HEAD~1
git rebase -i --autosquash HEAD~2
```

### 再レビュー依頼
- 修正内容をコメントで説明
- Re-review をリクエスト

---

## Step 5: マージ

### マージ前最終チェック
- [ ] 全レビュー承認済み
- [ ] 全CIチェック通過
- [ ] コンフリクトなし
- [ ] ベースブランチと同期済み

### マージ方法
```bash
# Squash Merge（推奨）
gh pr merge --squash

# または通常マージ
gh pr merge --merge
```

---

## Step 6: リリース準備

### バージョン更新
```yaml
# pubspec.yaml
version: 1.2.3+45  # major.minor.patch+buildNumber
```

### CHANGELOG更新
```markdown
## [1.2.3] - 2026-02-04

### Added
- [新機能]

### Changed
- [変更]

### Fixed
- [バグ修正]
```

### リリースビルド
```bash
# Android
flutter build apk --release
flutter build appbundle --release

# iOS
flutter build ipa --release
```

---

## Step 7: リリース

### ストア申請チェックリスト
- [ ] バージョン更新済み
- [ ] リリースノート作成済み
- [ ] スクリーンショット更新（必要時）
- [ ] ビルド署名済み
- [ ] 内部テスト完了

### リリースコマンド
```bash
# Git Tag
git tag -a v1.2.3 -m "Release 1.2.3"
git push origin v1.2.3

# GitHub Release
gh release create v1.2.3 --generate-notes
```

---

## リリース後

### 監視
- クラッシュレポート（Firebase Crashlytics）
- アプリレビュー
- ユーザーフィードバック

### ホットフィックス対応
```bash
git checkout -b hotfix/v1.2.4 v1.2.3
# 修正
git commit -m "hotfix: [内容]"
git tag -a v1.2.4 -m "Hotfix 1.2.4"
```
