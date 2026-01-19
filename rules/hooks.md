# フックシステム

## フックタイプ

- **PreToolUse**: ツール実行前（検証、パラメータ変更）
- **PostToolUse**: ツール実行後（自動フォーマット、チェック）
- **Stop**: セッション終了時（最終検証）

## 現在のフック（~/.claude/settings.json内）

### PreToolUse
- **tmuxリマインダー**: 長時間実行コマンド（pip、poetry、uvicorn、gunicornなど）にtmuxを提案
- **git pushレビュー**: プッシュ前にレビューのためエディタを開く
- **docブロッカー**: 不要な.md/.txtファイルの作成をブロック

### PostToolUse
- **PR作成**: PR URLとGitHub Actionsステータスをログ
- **Ruff**: 編集後にPythonファイルを自動フォーマット（`ruff format`）
- **mypyチェック**: .pyファイル編集後にmypyを実行
- **print文警告**: 編集されたファイル内のprint()について警告

### Stop
- **print監査**: セッション終了前にすべての変更されたファイルでprint()をチェック

## 自動承認権限

注意して使用:
- 信頼できる、明確に定義された計画に対して有効化
- 探索的作業では無効化
- dangerously-skip-permissionsフラグは絶対に使用しない
- 代わりに`~/.claude.json`で`allowedTools`を設定

## TodoWriteベストプラクティス

TodoWriteツールを使用して:
- マルチステップタスクの進捗を追跡
- 指示の理解を確認
- リアルタイムでの方向修正を可能にする
- 詳細な実装ステップを表示

Todoリストが明らかにするもの:
- 順序が間違ったステップ
- 欠落している項目
- 余分な不要な項目
- 間違った粒度
- 誤解された要件

## フック設定例

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'pip install や uvicorn は tmux で実行を推奨'"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          {
            "type": "command",
            "command": "ruff format $CLAUDE_FILE_PATHS && ruff check --fix $CLAUDE_FILE_PATHS"
          }
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo $CLAUDE_FILE_PATHS | grep -q '\\.py$'; then mypy $CLAUDE_FILE_PATHS --ignore-missing-imports; fi"
          }
        ]
      }
    ]
  }
}
```
