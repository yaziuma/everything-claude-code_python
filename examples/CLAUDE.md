# プロジェクトCLAUDE.mdの例

これはプロジェクトレベルのCLAUDE.mdファイルの例です。プロジェクトルートに配置してください。

## プロジェクト概要

[プロジェクトの簡潔な説明 - 何をするか、技術スタック]

## 重要なルール

### 1. コード構成

- 少数の大きなファイルより多数の小さなファイル
- 高凝集、低結合
- 200-400行が典型、ファイルあたり最大800行
- 型ではなく機能/ドメインで整理

### 2. コードスタイル

- コード、コメント、ドキュメントに絵文字なし
- 常に不変性 - オブジェクトや配列を決してミューテートしない
- 本番コードにconsole.logなし
- try/catchによる適切なエラーハンドリング
- Zodまたは類似による入力検証

### 3. テスト

- TDD：テストを最初に書く
- 最小80%カバレッジ
- ユーティリティの単体テスト
- APIの統合テスト
- 重要なフローのE2Eテスト

### 4. セキュリティ

- ハードコードされた秘密情報なし
- 機密データは環境変数
- すべてのユーザー入力を検証
- パラメータ化クエリのみ
- CSRF保護を有効化

## ファイル構造

```
src/
|-- app/              # Next.js app router
|-- components/       # 再利用可能なUIコンポーネント
|-- hooks/            # カスタムReactフック
|-- lib/              # ユーティリティライブラリ
|-- types/            # TypeScript定義
```

## 主要パターン

### APIレスポンス形式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### エラーハンドリング

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('操作が失敗しました:', error)
  return { success: false, error: 'ユーザーフレンドリーメッセージ' }
}
```

## 環境変数

```bash
# 必須
DATABASE_URL=
API_KEY=

# オプション
DEBUG=false
```

## 利用可能なコマンド

- `/tdd` - テスト駆動開発ワークフロー
- `/plan` - 実装計画を作成
- `/code-review` - コード品質をレビュー
- `/build-fix` - ビルドエラーを修正

## Gitワークフロー

- 従来のコミット：`feat:`、`fix:`、`refactor:`、`docs:`、`test:`
- mainに直接コミットしない
- PRにはレビューが必要
- マージ前にすべてのテストが通る必要がある