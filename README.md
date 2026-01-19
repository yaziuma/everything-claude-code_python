これはForkされたリポジトリです。

私が標準で利用している、Python + htmxに最適化されたリポジトリを作る予定です。

下記にオリジナルのREADMEを示します。

# Everything Claude Code

**Anthropicハッカソン優勝者による、Claude Codeの完全なコンフィグ集**

このリポジトリには、実際のプロダクト開発で10ヶ月以上使い込んで進化させた、本番環境対応のエージェント、スキル、フック、コマンド、ルール、MCPコンフィグが含まれています。

---

## まず完全ガイドをお読みください

**これらのコンフィグに入る前に、Xの完全ガイドをお読みください：**


<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />


**[Everything Claude Code 完全ガイド](https://x.com/affaanmustafa/status/2012378465664745795)**



ガイドでは以下を説明しています：
- 各コンフィグタイプの役割と使用タイミング
- Claude Codeセットアップの構造化方法
- コンテキストウィンドウ管理（パフォーマンスに重要）
- 並列ワークフローと高度なテクニック
- これらのコンフィグの背景にある哲学

**このリポジトリはコンフィグのみです！ヒント、トリック、その他の例は私のX記事と動画にあります（このreadmeの進化に合わせてリンクを追加予定）。**

---

## 内容

```
everything-claude-code/
|-- agents/           # 委任用の専門サブエージェント
|   |-- planner.md           # 機能実装計画
|   |-- architect.md         # システム設計決定
|   |-- tdd-guide.md         # テスト駆動開発
|   |-- code-reviewer.md     # 品質・セキュリティレビュー
|   |-- security-reviewer.md # 脆弱性分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2Eテスト
|   |-- refactor-cleaner.md  # デッドコード削除
|   |-- doc-updater.md       # ドキュメント同期
|
|-- skills/           # ワークフロー定義とドメイン知識
|   |-- coding-standards.md         # 言語ベストプラクティス
|   |-- backend-patterns.md         # API、データベース、キャッシュパターン
|   |-- frontend-patterns.md        # React、Next.jsパターン
|   |-- project-guidelines-example.md # プロジェクト固有スキル例
|   |-- tdd-workflow/               # TDD方法論
|   |-- security-review/            # セキュリティチェックリスト
|   |-- clickhouse-io.md            # ClickHouse分析
|
|-- commands/         # クイック実行用スラッシュコマンド
|   |-- tdd.md              # /tdd - テスト駆動開発
|   |-- plan.md             # /plan - 実装計画
|   |-- e2e.md              # /e2e - E2Eテスト生成
|   |-- code-review.md      # /code-review - 品質レビュー
|   |-- build-fix.md        # /build-fix - ビルドエラー修正
|   |-- refactor-clean.md   # /refactor-clean - デッドコード削除
|   |-- test-coverage.md    # /test-coverage - カバレッジ分析
|   |-- update-codemaps.md  # /update-codemaps - ドキュメント更新
|   |-- update-docs.md      # /update-docs - ドキュメント同期
|
|-- rules/            # 常に従うべきガイドライン
|   |-- security.md         # 必須セキュリティチェック
|   |-- coding-style.md     # 不変性、ファイル構成
|   |-- testing.md          # TDD、80%カバレッジ要件
|   |-- git-workflow.md     # コミット形式、PRプロセス
|   |-- agents.md           # サブエージェントへの委任タイミング
|   |-- performance.md      # モデル選択、コンテキスト管理
|   |-- patterns.md         # APIレスポンス形式、フック
|   |-- hooks.md            # フックドキュメント
|
|-- hooks/            # トリガーベース自動化
|   |-- hooks.json          # PreToolUse、PostToolUse、Stopフック
|
|-- mcp-configs/      # MCPサーバー設定
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railwayなど
|
|-- plugins/          # プラグインエコシステムドキュメント
|   |-- README.md           # プラグイン、マーケットプレイス、スキルガイド
|
|-- examples/         # 設定例
    |-- CLAUDE.md           # プロジェクトレベル設定例
    |-- user-CLAUDE.md      # ユーザーレベル設定例（~/.claude/CLAUDE.md）
    |-- statusline.json     # カスタムステータスライン設定
```

---

## クイックスタート

### 1. 必要なものをコピー

```bash
# リポジトリをクローン
git clone https://github.com/affaan-m/everything-claude-code.git

# エージェントをClaude設定にコピー
cp everything-claude-code/agents/*.md ~/.claude/agents/

# ルールをコピー
cp everything-claude-code/rules/*.md ~/.claude/rules/

# コマンドをコピー
cp everything-claude-code/commands/*.md ~/.claude/commands/

# スキルをコピー
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. settings.jsonにフックを追加

`hooks/hooks.json`のフックを`~/.claude/settings.json`にコピーします。

### 3. MCPを設定

`mcp-configs/mcp-servers.json`から必要なMCPサーバーを`~/.claude.json`にコピーします。

**重要：** `YOUR_*_HERE`プレースホルダーを実際のAPIキーに置き換えてください。

### 4. ガイドを読む

真剣に、[ガイドを読んでください](https://x.com/affaanmustafa/status/2012378465664745795)。これらのコンフィグはコンテキストがあると10倍理解しやすくなります。

---

## 主要概念

### エージェント

サブエージェントは限定されたスコープで委任されたタスクを処理します。例：

```markdown
---
name: code-reviewer
description: 品質、セキュリティ、保守性のためのコードレビュー
tools: Read, Grep, Glob, Bash
model: opus
---

あなたはシニアコードレビュアーです...
```

### スキル

スキルはコマンドやエージェントによって呼び出されるワークフロー定義です：

```markdown
# TDDワークフロー

1. まずインターフェースを定義
2. 失敗するテストを書く（RED）
3. 最小限のコードを実装（GREEN）
4. リファクタリング（IMPROVE）
5. 80%以上のカバレッジを確認
```

### フック

フックはツールイベントで発火します。例 - console.logについて警告：

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] console.logを削除してください' >&2"
  }]
}
```

### ルール

ルールは常に従うべきガイドラインです。モジュラーに保ちます：

```
~/.claude/rules/
  security.md      # ハードコードされた秘密情報禁止
  coding-style.md  # 不変性、ファイル制限
  testing.md       # TDD、カバレッジ要件
```

---

## 貢献

**貢献を歓迎し、推奨します。**

このリポジトリはコミュニティリソースとなることを意図しています。以下をお持ちの場合：
- 有用なエージェントやスキル
- 巧妙なフック
- より良いMCP設定
- 改良されたルール

ぜひ貢献してください！ガイドラインについては[CONTRIBUTING.md](CONTRIBUTING.md)をご覧ください。

### 貢献のアイデア

- 言語固有スキル（Python、Go、Rustパターン）
- フレームワーク固有設定（Django、Rails、Laravel）
- DevOpsエージェント（Kubernetes、Terraform、AWS）
- テスト戦略（異なるフレームワーク）
- ドメイン固有知識（ML、データエンジニアリング、モバイル）

---

## 背景

私は実験的ロールアウトからClaude Codeを使用しています。2025年9月のAnthropic x Forum Venturesハッカソンで[@DRodriguezFX](https://x.com/DRodriguezFX)と[zenith.chat](https://zenith.chat)を構築して優勝しました - 完全にClaude Codeを使用して。

これらのコンフィグは複数の本番アプリケーションで実戦テスト済みです。

---

## 重要な注意事項

### コンテキストウィンドウ管理

**重要：** すべてのMCPを一度に有効にしないでください。ツールが多すぎると200kのコンテキストウィンドウが70kに縮小される可能性があります。

経験則：
- 20-30のMCPを設定
- プロジェクトごとに10個未満を有効に保つ
- アクティブなツールを80個未満に

使用しないものを無効にするには、プロジェクト設定で`disabledMcpServers`を使用します。

### カスタマイゼーション

これらのコンフィグは私のワークフローに合わせて作られています。あなたは以下を行うべきです：
1. 共感できるものから始める
2. あなたのスタックに合わせて修正
3. 使わないものを削除
4. 独自のパターンを追加

---

## リンク

- **完全ガイド：** [Everything Claude Code 完全ガイド](https://x.com/affaanmustafa/status/2012378465664745795)
- **フォロー：** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat：** [zenith.chat](https://zenith.chat)

---

## ライセンス

MIT - 自由に使用し、必要に応じて修正し、可能であれば貢献してください。

---

**このリポジトリが役立つ場合はスターをつけてください。ガイドを読んでください。素晴らしいものを作ってください。**