---
description: テスト駆動開発ワークフローを強制。インターフェースを足場化し、最初にテストを生成し、次に通すための最小限のコードを実装。80%以上のカバレッジを確保。
---

# TDDコマンド

このコマンドは**tdd-guide**エージェントを呼び出して、テスト駆動開発方法論を強制します。

## このコマンドが行うこと

1. **インターフェースの足場化** - 最初に型/インターフェースを定義
2. **最初にテストを生成** - 失敗するテストを書く（RED）
3. **最小限のコードを実装** - 通すのに十分なだけ書く（GREEN）
4. **リファクタリング** - テストを緑に保ちながらコードを改善（REFACTOR）
5. **カバレッジを確認** - 80%以上のテストカバレッジを確保

## 使用タイミング

以下の場合に`/tdd`を使用：
- 新機能の実装
- 新しい関数/コンポーネントの追加
- バグの修正（最初にバグを再現するテストを書く）
- 既存コードのリファクタリング
- 重要なビジネスロジックの構築

## 動作方法

tdd-guideエージェントは以下を行います：

1. **インターフェースを定義**入力/出力用
2. **失敗するテストを書く**（コードがまだ存在しないため）
3. **テストを実行**し、正しい理由で失敗することを確認
4. **最小限の実装を書く**テストを通すため
5. **テストを実行**し、通ることを確認
6. **コードをリファクタリング**テストを緑に保ちながら
7. **カバレッジをチェック**し、80%未満の場合はテストを追加

## TDDサイクル

```
RED → GREEN → REFACTOR → 繰り返し

RED:      失敗するテストを書く
GREEN:    通すための最小限のコードを書く
REFACTOR: テストを通したままコードを改善
繰り返し:   次の機能/シナリオ
```

## 使用例

```
ユーザー：/tdd マーケット流動性スコアを計算する関数が必要

エージェント（tdd-guide）：
# TDDセッション：マーケット流動性スコア計算機

## ステップ1：インターフェースを定義（足場化）

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: 実装
  throw new Error('Not implemented')
}
```

## ステップ2：失敗するテストを書く（RED）

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('流動性の高いマーケットに高いスコアを返すべき', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('流動性の低いマーケットに低いスコアを返すべき', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1日前
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('エッジケースを処理すべき：ゼロボリューム', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## ステップ3：テストを実行 - 失敗を確認

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ 流動性の高いマーケットに高いスコアを返すべき (2 ms)
    Error: Not implemented

1 test failed, 0 passed
```

✅ テストが期待通りに失敗。実装準備完了。

## ステップ4：最小限のコードを実装（GREEN）

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // ゼロボリュームのエッジケースを処理
  if (market.totalVolume === 0) {
    return 0
  }

  // コンポーネントスコアを計算（0-100スケール）
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // 最近のアクティビティボーナス
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // 加重平均
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // 0-100にクランプ
}
```

## ステップ5：テストを実行 - 通過を確認

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 流動性の高いマーケットに高いスコアを返すべき (3 ms)
  ✓ 流動性の低いマーケットに低いスコアを返すべき (2 ms)
  ✓ エッジケースを処理すべき：ゼロボリューム (1 ms)

3 tests passed
```

✅ すべてのテストが通過！

## ステップ6：リファクタリング（改善）

```typescript
// lib/liquidity.ts - 定数とより良い可読性でリファクタリング
const WEIGHTS = {
  VOLUME: 0.4,
  SPREAD: 0.3,
  TRADERS: 0.2,
  RECENCY: 0.1,
} as const

const SCALE_FACTORS = {
  VOLUME: 1000,
  SPREAD: 1000,
  TRADERS: 10,
  RECENCY_PENALTY: 10,
} as const

function clamp(value: number, min: number, max: number): number {
  return Math.min(Math.max(value, min), max)
}

export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0

  const volumeScore = Math.min(market.totalVolume / SCALE_FACTORS.VOLUME, 100)
  const spreadScore = clamp(100 - (market.bidAskSpread * SCALE_FACTORS.SPREAD), 0, 100)
  const traderScore = Math.min(market.activeTraders / SCALE_FACTORS.TRADERS, 100)

  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = clamp(100 - (hoursSinceLastTrade * SCALE_FACTORS.RECENCY_PENALTY), 0, 100)

  const weightedScore =
    volumeScore * WEIGHTS.VOLUME +
    spreadScore * WEIGHTS.SPREAD +
    traderScore * WEIGHTS.TRADERS +
    recencyScore * WEIGHTS.RECENCY

  return clamp(weightedScore, 0, 100)
}
```

## ステップ7：テストがまだ通ることを確認

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 流動性の高いマーケットに高いスコアを返すべき (3 ms)
  ✓ 流動性の低いマーケットに低いスコアを返すべき (2 ms)
  ✓ エッジケースを処理すべき：ゼロボリューム (1 ms)

3 tests passed
```

✅ リファクタリング完了、テストはまだ通過！

## ステップ8：カバレッジをチェック

```bash
npm test -- --coverage lib/liquidity.test.ts

File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

カバレッジ：100% ✅（目標：80%）
```

✅ TDDセッション完了！
```

## TDDベストプラクティス

**すべきこと：**
- ✅ 実装前に最初にテストを書く
- ✅ テストを実行し、実装前に失敗することを確認
- ✅ テストを通すための最小限のコードを書く
- ✅ テストが緑になった後のみリファクタリング
- ✅ エッジケースとエラーシナリオを追加
- ✅ 80%以上のカバレッジを目指す（重要なコードは100%）

**すべきでないこと：**
- ❌ テスト前に実装を書く
- ❌ 各変更後のテスト実行をスキップ
- ❌ 一度に多すぎるコードを書く
- ❌ 失敗するテストを無視
- ❌ 実装詳細をテスト（動作をテスト）
- ❌ すべてをモック（統合テストを優先）

## 含めるべきテストタイプ

**単体テスト**（関数レベル）：
- ハッピーパスシナリオ
- エッジケース（空、null、最大値）
- エラー条件
- 境界値

**統合テスト**（コンポーネントレベル）：
- APIエンドポイント
- データベース操作
- 外部サービス呼び出し
- フック付きReactコンポーネント

**E2Eテスト**（`/e2e`コマンドを使用）：
- 重要なユーザーフロー
- 複数ステップのプロセス
- フルスタック統合

## カバレッジ要件

- **80%最小**すべてのコード用
- **100%必須**以下用：
  - 財務計算
  - 認証ロジック
  - セキュリティ重要コード
  - コアビジネスロジック

## 重要な注意事項

**必須**：テストは実装前に書かれなければなりません。TDDサイクルは：

1. **RED** - 失敗するテストを書く
2. **GREEN** - 通すために実装
3. **REFACTOR** - コードを改善

REDフェーズをスキップしてはいけません。テスト前にコードを書いてはいけません。

## 他のコマンドとの統合

- 最初に`/plan`を使用して何を構築するかを理解
- `/tdd`を使用してテスト付きで実装
- ビルドエラーが発生した場合は`/build-and-fix`を使用
- 実装をレビューするために`/code-review`を使用
- カバレッジを確認するために`/test-coverage`を使用

## 関連エージェント

このコマンドは以下にあるtdd-guideエージェントを呼び出します：
`~/.claude/agents/tdd-guide.md`

また、以下のtdd-workflowスキルを参照できます：
`~/.claude/skills/tdd-workflow/`