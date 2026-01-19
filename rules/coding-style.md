# コーディングスタイル

## 不変性（重要）

常に新しいオブジェクトを作成し、決してミューテートしない：

```javascript
// 間違い：ミューテーション
function updateUser(user, name) {
  user.name = name  // ミューテーション！
  return user
}

// 正しい：不変性
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## ファイル構成

少数の大きなファイルより多数の小さなファイル：
- 高凝集、低結合
- 200-400行が典型、最大800行
- 大きなコンポーネントからユーティリティを抽出
- 型ではなく機能/ドメインで整理

## エラーハンドリング

常に包括的にエラーを処理：

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('操作が失敗しました:', error)
  throw new Error('詳細なユーザーフレンドリーメッセージ')
}
```

## 入力検証

常にユーザー入力を検証：

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## コード品質チェックリスト

作業完了をマークする前に：
- [ ] コードが読みやすく適切に命名されている
- [ ] 関数が小さい（50行未満）
- [ ] ファイルが集中している（800行未満）
- [ ] 深いネストなし（4レベル超）
- [ ] 適切なエラーハンドリング
- [ ] console.log文なし
- [ ] ハードコードされた値なし
- [ ] ミューテーションなし（不変パターンを使用）