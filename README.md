# Flutter モバイルアプリ開発支援ツール

Flutter 2025-2026ベストプラクティスに基づく、AI駆動開発支援ツールセットです。

## 概要

このリポジトリには以下が含まれています：

| カテゴリ | 内容 | 数 |
|---------|------|-----|
| スキル | 開発フェーズ別の専門スキル | 13 |
| サブエージェント | 役割別のAIエージェント | 11 |
| ワークフロー | 開発フロー定義 | 3 |
| テンプレート | プロジェクト/設計/テスト | 3 |
| チェックリスト | 品質ゲート | 4 |
| 設定サンプル | Flutter推奨設定 | 2 |

---

## クイックスタート

### 1. スキルの使用

スキルは特定の開発タスクを支援するナレッジベースです。

```
skills/
├── ai-flutter-guidelines/     # AI開発ガイドライン（禁止事項・対策）★必読
├── mobile-app-design/     # アプリ設計ガイド
├── mobile-app-uiux/       # UI/UX設計ガイド
├── flutter-development/   # 実装ガイド（Riverpod 3.0）
├── flutter-tdd/           # TDDガイド
├── flutter-code-review/   # コードレビューガイド
├── flutter-debugging/     # デバッグガイド
├── flutter-ci-cd/         # CI/CD・デプロイ自動化
├── flutter-performance/   # パフォーマンス最適化
├── flutter-i18n/          # 多言語対応（国際化）
├── flutter-native-integration/ # ネイティブ連携
├── flutter-analytics/     # 分析・監視（Firebase）
└── flutter-migration/     # マイグレーション・リファクタリング
```

**使用例:**
> 「新機能の設計を始めたい」→ `skills/mobile-app-design/SKILL.md` を参照

### 2. サブエージェントの使用

サブエージェントは特定の役割を持つAIアシスタントです。

```
subagents/
├── mobile-architect.md    # 設計判断・アーキテクチャ
├── uiux-designer.md       # UI/UX設計
├── security-expert.md     # セキュリティレビュー
├── flutter-developer.md   # 実装
├── flutter-tdd-runner.md  # テスト作成
├── flutter-reviewer.md    # コードレビュー
├── flutter-debugger.md    # バグ修正
├── devops-engineer.md     # CI/CD・デプロイ自動化
├── performance-specialist.md # パフォーマンスチューニング
├── documentation-writer.md # 技術ドキュメント作成
└── localization-expert.md # 多言語対応・ローカライゼーション
```

**使用例:**
> 「この設計をレビューして」→ `mobile-architect` を呼び出し

### 3. ワークフローに従う

```
workflows/
├── development-flow.md   # 開発フロー全体
├── design-to-impl.md     # 設計→実装フロー
└── review-release.md     # レビュー→リリースフロー
```

**標準フロー:**
```
DESIGN → SECURITY_REVIEW → IMPLEMENT → TEST → REVIEW → RELEASE
```

---

## スキル詳細

### mobile-app-design
- Clean Architecture設計
- Riverpod 3.0状態管理パターン
- データモデリング（freezed）
- ADR（意思決定記録）

### mobile-app-uiux
- Material Design 3準拠
- レスポンシブレイアウト（Compact/Medium/Expanded）
- アクセシビリティ（WCAG 2.1 AA）
- デザイントークン定義

### flutter-development
- Riverpod 3.0 + コード生成
- constコンストラクタ最適化
- パフォーマンス（ListView.builder, Isolate）
- エラーハンドリング

### flutter-tdd
- Red-Green-Refactorサイクル
- unit/widget/integrationテスト
- Mocktailによるモック
- カバレッジ目標（80%+）

### flutter-code-review
- CRITICAL/HIGH/MEDIUM/LOW分類
- セキュリティチェック
- パフォーマンスチェック
- アーキテクチャ準拠確認

### flutter-debugging
- DevTools活用
- よくあるエラーパターンと解決策
- メモリリーク検出
- クラッシュ分析

---

## テンプレート

### プロジェクト構造
Feature-Firstアーキテクチャのディレクトリ構成

### 設計書テンプレート
機能設計書の標準フォーマット

### テストケーステンプレート
unit/widget/integrationテストの記述例

---

## チェックリスト

| チェックリスト | 使用タイミング |
|---------------|---------------|
| design-checklist.md | 設計完了時 |
| implementation-checklist.md | 実装完了時 |
| security-checklist.md | セキュリティレビュー時 |
| release-checklist.md | リリース前 |

---

## 設定サンプル

### analysis_options.yaml
Flutter推奨のlint設定（strict mode）

### pubspec_template.yaml
2026年2月時点の推奨パッケージ構成
- Riverpod 3.0
- Dio + Retrofit
- freezed
- flutter_secure_storage

---

## 技術スタック（推奨）

| カテゴリ | 推奨 |
|---------|------|
| 状態管理 | Riverpod 3.0 |
| アーキテクチャ | Clean Architecture (Feature-First) |
| ネットワーク | Dio + Retrofit |
| モデル | freezed + json_serializable |
| セキュアストレージ | flutter_secure_storage |
| テスト | Mocktail |
| 静的解析 | flutter_lints + カスタム |

---

## 使い方のヒント

1. **新規プロジェクト開始時**
   - `templates/project-structure/` を参考に構造作成
   - `config-samples/` の設定を適用

2. **機能開発時**
   - `workflows/development-flow.md` に従う
   - 各フェーズで対応するスキル/サブエージェントを使用

3. **品質確保**
   - 各フェーズで対応するチェックリストを確認

---

## ライセンス

MIT License
