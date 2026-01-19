---
name: tdd-workflow
description: 新機能の記述、バグ修正、コードリファクタリング時にこのスキルを使用。ユニット、統合、E2Eテストを含む80%以上のカバレッジでテスト駆動開発を強制。
---

# テスト駆動開発ワークフロー

このスキルは、すべてのコード開発が包括的なテストカバレッジを伴うTDD原則に従うことを確保します。

## 有効化タイミング

- 新機能や機能の記述
- バグや問題の修正
- 既存コードのリファクタリング
- APIエンドポイントの追加
- 新しいコンポーネントの作成

## 基本原則

### 1. コードの前にテスト
常にテストを最初に書き、次にテストを通すためのコードを実装します。

### 2. カバレッジ要件
- 最低80%カバレッジ（ユニット + 統合 + E2E）
- すべてのエッジケースをカバー
- エラーシナリオをテスト
- 境界条件を検証

### 3. テストタイプ

#### ユニットテスト
- 個別の関数とユーティリティ
- コンポーネントロジック
- 純粋関数
- ヘルパーとユーティリティ

#### 統合テスト
- APIエンドポイント
- データベース操作
- サービス間相互作用
- 外部API呼び出し

#### E2Eテスト（Playwright）
- 重要なユーザーフロー
- 完全なワークフロー
- ブラウザ自動化
- UIインタラクション

## TDDワークフローステップ

### ステップ1: ユーザージャーニーを記述
```
[役割]として、[アクション]したい、[利益]のために

例:
ユーザーとして、マーケットをセマンティックに検索したい、
正確なキーワードなしでも関連マーケットを見つけられるように。
```

### ステップ2: テストケースを生成
各ユーザージャーニーについて、包括的なテストケースを作成:

```typescript
describe('セマンティック検索', () => {
  it('クエリに関連するマーケットを返す', async () => {
    // テスト実装
  })

  it('空のクエリを適切に処理する', async () => {
    // エッジケースをテスト
  })

  it('Redis利用不可時に部分文字列検索にフォールバック', async () => {
    // フォールバック動作をテスト
  })

  it('類似度スコアで結果をソート', async () => {
    // ソートロジックをテスト
  })
})
```

### ステップ3: テストを実行（失敗するはず）
```bash
npm test
# テストは失敗するはず - まだ実装していない
```

### ステップ4: コードを実装
テストを通すための最小限のコードを記述:

```typescript
// テストに導かれた実装
export async function searchMarkets(query: string) {
  // ここに実装
}
```

### ステップ5: テストを再実行
```bash
npm test
# テストは今度は通るはず
```

### ステップ6: リファクタリング
テストを緑に保ちながらコード品質を改善:
- 重複を削除
- 命名を改善
- パフォーマンスを最適化
- 可読性を向上

### ステップ7: カバレッジを確認
```bash
npm run test:coverage
# 80%以上のカバレッジが達成されたことを確認
```

## テストパターン

### ユニットテストパターン（Jest/Vitest）
```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from './Button'

describe('Buttonコンポーネント', () => {
  it('正しいテキストでレンダリングされる', () => {
    render(<Button>クリックしてください</Button>)
    expect(screen.getByText('クリックしてください')).toBeInTheDocument()
  })

  it('クリック時にonClickを呼び出す', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>クリック</Button>)

    fireEvent.click(screen.getByRole('button'))

    expect(handleClick).toHaveBeenCalledTimes(1)
  })

  it('disabledプロップがtrueの時に無効化される', () => {
    render(<Button disabled>クリック</Button>)
    expect(screen.getByRole('button')).toBeDisabled()
  })
})
```

### API統合テストパターン
```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

describe('GET /api/markets', () => {
  it('マーケットを正常に返す', async () => {
    const request = new NextRequest('http://localhost/api/markets')
    const response = await GET(request)
    const data = await response.json()

    expect(response.status).toBe(200)
    expect(data.success).toBe(true)
    expect(Array.isArray(data.data)).toBe(true)
  })

  it('クエリパラメータを検証する', async () => {
    const request = new NextRequest('http://localhost/api/markets?limit=invalid')
    const response = await GET(request)

    expect(response.status).toBe(400)
  })

  it('データベースエラーを適切に処理する', async () => {
    // データベース障害をモック
    const request = new NextRequest('http://localhost/api/markets')
    // エラーハンドリングをテスト
  })
})
```

### E2Eテストパターン（Playwright）
```typescript
import { test, expect } from '@playwright/test'

test('ユーザーはマーケットを検索・フィルタできる', async ({ page }) => {
  // マーケットページに移動
  await page.goto('/')
  await page.click('a[href="/markets"]')

  // ページが読み込まれたことを確認
  await expect(page.locator('h1')).toContainText('マーケット')

  // マーケットを検索
  await page.fill('input[placeholder="マーケットを検索"]', 'election')

  // デバウンスと結果を待機
  await page.waitForTimeout(600)

  // 検索結果が表示されることを確認
  const results = page.locator('[data-testid="market-card"]')
  await expect(results).toHaveCount(5, { timeout: 5000 })

  // 結果に検索語が含まれることを確認
  const firstResult = results.first()
  await expect(firstResult).toContainText('election', { ignoreCase: true })

  // ステータスでフィルタ
  await page.click('button:has-text("アクティブ")')

  // フィルタされた結果を確認
  await expect(results).toHaveCount(3)
})

test('ユーザーは新しいマーケットを作成できる', async ({ page }) => {
  // まずログイン
  await page.goto('/creator-dashboard')

  // マーケット作成フォームを入力
  await page.fill('input[name="name"]', 'テストマーケット')
  await page.fill('textarea[name="description"]', 'テスト説明')
  await page.fill('input[name="endDate"]', '2025-12-31')

  // フォームを送信
  await page.click('button[type="submit"]')

  // 成功メッセージを確認
  await expect(page.locator('text=マーケットが正常に作成されました')).toBeVisible()

  // マーケットページへのリダイレクトを確認
  await expect(page).toHaveURL(/\/markets\/test-market/)
})
```

## テストファイル構成

```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx          # ユニットテスト
│   │   └── Button.stories.tsx       # Storybook
│   └── MarketCard/
│       ├── MarketCard.tsx
│       └── MarketCard.test.tsx
├── app/
│   └── api/
│       └── markets/
│           ├── route.ts
│           └── route.test.ts         # 統合テスト
└── e2e/
    ├── markets.spec.ts               # E2Eテスト
    ├── trading.spec.ts
    └── auth.spec.ts
```

## 外部サービスのモック

### Supabaseモック
```typescript
jest.mock('@/lib/supabase', () => ({
  supabase: {
    from: jest.fn(() => ({
      select: jest.fn(() => ({
        eq: jest.fn(() => Promise.resolve({
          data: [{ id: 1, name: 'テストマーケット' }],
          error: null
        }))
      }))
    }))
  }
}))
```

### Redisモック
```typescript
jest.mock('@/lib/redis', () => ({
  searchMarketsByVector: jest.fn(() => Promise.resolve([
    { slug: 'test-market', similarity_score: 0.95 }
  ])),
  checkRedisHealth: jest.fn(() => Promise.resolve({ connected: true }))
}))
```

### OpenAIモック
```typescript
jest.mock('@/lib/openai', () => ({
  generateEmbedding: jest.fn(() => Promise.resolve(
    new Array(1536).fill(0.1) // 1536次元埋め込みをモック
  ))
}))
```

## テストカバレッジ検証

### カバレッジレポートを実行
```bash
npm run test:coverage
```

### カバレッジ閾値
```json
{
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

## 避けるべき一般的なテストミス

### ❌ 間違い: 実装詳細をテスト
```typescript
// 内部状態をテストしない
expect(component.state.count).toBe(5)
```

### ✅ 正しい: ユーザーに見える動作をテスト
```typescript
// ユーザーが見るものをテスト
expect(screen.getByText('カウント: 5')).toBeInTheDocument()
```

### ❌ 間違い: 脆弱なセレクタ
```typescript
// 簡単に壊れる
await page.click('.css-class-xyz')
```

### ✅ 正しい: セマンティックセレクタ
```typescript
// 変更に強い
await page.click('button:has-text("送信")')
await page.click('[data-testid="submit-button"]')
```

### ❌ 間違い: テスト分離なし
```typescript
// テストが相互依存
test('ユーザーを作成', () => { /* ... */ })
test('同じユーザーを更新', () => { /* 前のテストに依存 */ })
```

### ✅ 正しい: 独立したテスト
```typescript
// 各テストが独自のデータを設定
test('ユーザーを作成', () => {
  const user = createTestUser()
  // テストロジック
})

test('ユーザーを更新', () => {
  const user = createTestUser()
  // 更新ロジック
})
```

## 継続的テスト

### 開発中のウォッチモード
```bash
npm test -- --watch
# ファイル変更時にテストが自動実行
```

### プリコミットフック
```bash
# 各コミット前に実行
npm test && npm run lint
```

### CI/CD統合
```yaml
# GitHub Actions
- name: テストを実行
  run: npm test -- --coverage
- name: カバレッジをアップロード
  uses: codecov/codecov-action@v3
```

## ベストプラクティス

1. **テストを最初に書く** - 常にTDD
2. **テストごとに一つのアサート** - 単一の動作に焦点
3. **説明的なテスト名** - テスト内容を説明
4. **Arrange-Act-Assert** - 明確なテスト構造
5. **外部依存関係をモック** - ユニットテストを分離
6. **エッジケースをテスト** - null、undefined、空、大きな値
7. **エラーパスをテスト** - ハッピーパスだけでなく
8. **テストを高速に保つ** - ユニットテストは各50ms未満
9. **テスト後のクリーンアップ** - 副作用なし
10. **カバレッジレポートをレビュー** - ギャップを特定

## 成功指標

- 80%以上のコードカバレッジを達成
- すべてのテストが通過（緑）
- スキップまたは無効化されたテストなし
- 高速なテスト実行（ユニットテストは30秒未満）
- E2Eテストが重要なユーザーフローをカバー
- テストが本番前にバグをキャッチ

---

**覚えておくこと**: テストはオプションではありません。自信を持ったリファクタリング、迅速な開発、本番の信頼性を可能にするセーフティネットです。