# t2k2pp Tool Store

AI駆動開発のためのスキル、サブエージェント、MCP、ワークフロー集。

## 📦 概要

このリポジトリは、AI（Claude等）による開発を支援するツールストアです。
プロジェクトに必要なツールを動的に取得して使用できます。

## 🏗️ 構造

```
ToolStore/
├── catalog.yaml              # ツールカタログ（インデックス）
├── fetcher/                  # skill-fetcher（ツール取得スキル）
│   └── SKILL.md
│
├── domains/                  # ドメイン別ツール
│   ├── mobile/
│   │   ├── flutter/          # Flutter/Dart (15スキル, 11エージェント)
│   │   ├── swift/            # iOS ネイティブ（計画中）
│   │   └── kotlin/           # Android ネイティブ（計画中）
│   └── common/               # 共通ツール
│       └── skills/
│           └── ai-flutter-guidelines/
│
└── mcps/                     # Model Context Protocols
    ├── database/             # Supabase, Firebase
    ├── search/               # Web検索
    ├── storage/              # S3
    └── ai/                   # OpenAI
```

## 🚀 使用方法

### 1. fetcher スキルをプロジェクトに配置

```bash
# fetcherをコピー
curl -sL https://raw.githubusercontent.com/t2k2pp/MobileAppAgentSkills/main/fetcher/SKILL.md \
  -o .agent/skills/skill-fetcher/SKILL.md --create-dirs
```

### 2. やりたいことを伝える

```
「Flutterでヘルスケアアプリを作りたい」
```

### 3. 推奨ツールが提案される

| カテゴリ | ツール | 説明 |
|---------|--------|------|
| skill | flutter-development | Riverpod 3.0実装ガイド |
| skill | flutter-native-integration | HealthKit連携 |
| skill | ai-flutter-guidelines | AI禁止事項【必須】 |
| subagent | flutter-developer | 実装担当エージェント |

### 4. 承認後、自動取得・開発開始

## 📂 ツール一覧

### スキル

| ドメイン | ID | 説明 |
|---------|-----|------|
| flutter | flutter-development | Riverpod 3.0、Widget実装 |
| flutter | flutter-tdd | TDD（Red-Green-Refactor） |
| flutter | flutter-code-review | コードレビュー |
| flutter | flutter-debugging | デバッグ・DevTools |
| flutter | flutter-ci-cd | CI/CD・ストアデプロイ |
| flutter | flutter-performance | パフォーマンス最適化 |
| flutter | flutter-i18n | 多言語対応 |
| flutter | flutter-native-integration | ネイティブ連携・iOS固有API |
| flutter | flutter-analytics | Firebase Analytics |
| flutter | flutter-migration | マイグレーション |
| flutter | ios-platform-setup | Xcode・証明書設定 |
| flutter | ios-store-guidelines | App Store審査対策 |
| flutter | mobile-app-design | Clean Architecture設計 |
| flutter | mobile-app-uiux | Material Design 3・HIG |
| common | ai-flutter-guidelines | **AI開発禁止事項【必須】** |

### サブエージェント

| ID | 役割 | モデル |
|----|------|--------|
| flutter-developer | 製造実装 | Sonnet |
| mobile-architect | 設計・技術選定 | Opus |
| uiux-designer | UI/UX設計 | Sonnet |
| security-expert | セキュリティ | Opus |
| flutter-tdd-runner | TDD実行 | Sonnet |
| flutter-reviewer | コードレビュー | Opus |
| flutter-debugger | デバッグ | Sonnet |
| devops-engineer | CI/CD | Sonnet |
| performance-specialist | パフォーマンス | Opus |
| documentation-writer | ドキュメント | Sonnet |
| localization-expert | 多言語 | Sonnet |

### MCP（準備中）

| カテゴリ | ID | 説明 |
|---------|----|------|
| database | supabase-mcp | Supabase連携 |
| database | firebase-mcp | Firebase連携 |
| search | web-search-mcp | Web検索 |
| storage | s3-mcp | AWS S3 |
| ai | openai-mcp | OpenAI API |

## 📋 カタログ

詳細は [catalog.yaml](./catalog.yaml) を参照。

## 🔮 ロードマップ

- [ ] Swift スキル・サブエージェント
- [ ] Kotlin スキル・サブエージェント
- [ ] Web（React/Next.js）ドメイン
- [ ] Backend（Python/Go）ドメイン
- [ ] MCP 実装

## 📝 ライセンス

Private - t2k2pp
