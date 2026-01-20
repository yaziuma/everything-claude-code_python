# Everything Claude Code (Python版)

## プロジェクト概要

Anthropicハッカソン優勝者が10ヶ月以上かけて作成したClaude Code設定テンプレート集のフォーク。
Python + htmx向けに最適化予定。

## リポジトリ構成

| ディレクトリ | 内容 | ファイル数 |
|-------------|------|-----------|
| `agents/` | 専門サブエージェント定義 | 9個 |
| `commands/` | スラッシュコマンド | 9個 |
| `rules/` | 常時適用ガイドライン | 8個 |
| `skills/` | ワークフロー定義 | 7個 |
| `hooks/` | 自動化フック | 1個 |
| `mcp-configs/` | MCPサーバー設定 | 1個 |
| `examples/` | CLAUDE.md例 | 3個 |

## エージェント一覧

| エージェント | 用途 | モデル |
|-------------|------|--------|
| planner | 実装計画作成 | Opus |
| architect | システム設計・アーキテクチャ決定 | Opus |
| code-reviewer | コード品質・セキュリティレビュー | Opus |
| security-reviewer | セキュリティ脆弱性検出（OWASP Top 10対応） | Opus |
| tdd-guide | テスト駆動開発支援 | Opus |
| build-error-resolver | ビルド・TypeScriptエラー修正 | Opus |
| e2e-runner | Playwright E2Eテスト | Opus |
| refactor-cleaner | デッドコード削除 | Opus |
| doc-updater | ドキュメント更新 | Opus |

## ルール一覧

| ルール | 内容 |
|--------|------|
| security.md | 必須セキュリティチェック（秘密情報、SQLインジェクション、XSS等） |
| coding-style.md | 不変性、ファイル構成（200-800行）、エラーハンドリング |
| testing.md | TDD必須、80%カバレッジ要件 |
| git-workflow.md | コミット形式、PRプロセス |
| agents.md | エージェント委任ガイド |
| performance.md | モデル選択、コンテキスト管理 |
| patterns.md | APIレスポンス形式 |
| hooks.md | フックドキュメント |

## フック機能

### PreToolUse
- 開発サーバーはtmux経由を強制
- 長時間コマンドでtmux推奨リマインダー
- git push前に確認ダイアログ
- 不要な.mdファイル作成をブロック

### PostToolUse
- PR作成後にURL表示
- Pythonファイル編集後にRuff実行
- mypy型チェック
- print文警告

### Stop
- セッション終了時のprint文最終監査

## MCP設定

13のMCPサーバー設定が用意：
- GitHub、Supabase、Vercel、Railway
- Cloudflare（docs/workers/observability）
- ClickHouse、Firecrawl、Memory、Context7など

**注意**: 一度に10個未満のMCPを有効化すること（コンテキストウィンドウ節約）

## 対象技術スタック

- **フレームワーク**: FastAPI
- **ORM**: SQLAlchemy
- **テンプレート**: Jinja2
- **バリデーション**: Pydantic
- **テスト**: pytest
- **フロントエンド**: htmx

## タスク

- [ ] `uv sync` を実行して環境をセットアップ
- [ ] `pyproject.toml` に依存関係を追加
- [ ] エージェント定義をPython向けに修正（agents/）
- [ ] ルール定義をPython向けに修正（rules/）
- [ ] コマンド定義をPython向けに修正（commands/）
- [ ] スキル定義をPython向けに修正（skills/）
- [ ] フック設定をPython向けに修正（hooks/）
- [ ] MCP設定の整理（mcp-configs/）

## 利用可能なコマンド

- `/setup` - `uv sync` (初期セットアップ)
- `/tdd` - テスト駆動開発ワークフロー
- `/plan` - 実装計画を作成
- `/code-review` - コード品質をレビュー
- `/build-fix` - ビルドエラーを修正
- `/e2e` - E2Eテスト生成
- `/refactor-clean` - デッドコード削除
- `/test-coverage` - カバレッジ分析
- `/update-codemaps` - コードマップ更新
- `/update-docs` - ドキュメント同期

## 重要なルール

### コード構成
- 少数の大きなファイルより多数の小さなファイル
- 高凝集、低結合
- 200-400行が典型、最大800行

### コードスタイル
- 状態管理は慎重に - 共有される状態は不変にする
- 本番コードにprint文なし
- 適切なエラーハンドリング

### テスト
- TDD：テストを最初に書く
- 最小80%カバレッジ
- ユニット・統合・E2Eテスト必須

### セキュリティ
- ハードコードされた秘密情報なし
- 機密データは環境変数
- すべてのユーザー入力を検証
- パラメータ化クエリのみ