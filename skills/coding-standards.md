---
name: coding-standards
description: TypeScript、JavaScript、React、Node.js開発のための汎用コーディング標準、ベストプラクティス、パターン。
---

# コーディング標準・ベストプラクティス

すべてのプロジェクトに適用可能な汎用コーディング標準。

## コード品質原則

### 1. 可読性第一
- コードは書くより読まれることが多い
- 明確な変数と関数名
- コメントよりも自己文書化コードを優先
- 一貫したフォーマット

### 2. KISS（Keep It Simple, Stupid）
- 動作する最もシンプルなソリューション
- 過度なエンジニアリングを避ける
- 早すぎる最適化はしない
- 巧妙なコードより理解しやすいコード

### 3. DRY（Don't Repeat Yourself）
- 共通ロジックを関数に抽出
- 再利用可能なコンポーネントを作成
- モジュール間でユーティリティを共有
- コピー&ペーストプログラミングを避ける

### 4. YAGNI（You Aren't Gonna Need It）
- 必要になる前に機能を構築しない
- 投機的汎用性を避ける
- 必要な時のみ複雑性を追加
- シンプルに始めて、必要時にリファクタリング

## TypeScript/JavaScript標準

### 変数命名

```typescript
// ✅ 良い：説明的な名前
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 悪い：不明確な名前
const q = 'election'
const flag = true
const x = 1000
```

### 関数命名

```typescript
// ✅ 良い：動詞-名詞パターン
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 悪い：不明確または名詞のみ
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### 不変性パターン（重要）

```typescript
// ✅ 常にスプレッド演算子を使用
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// ❌ 決して直接ミューテートしない
user.name = 'New Name'  // 悪い
items.push(newItem)     // 悪い
```

### エラーハンドリング

```typescript
// ✅ 良い：包括的なエラーハンドリング
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('フェッチが失敗しました:', error)
    throw new Error('データの取得に失敗しました')
  }
}

// ❌ 悪い：エラーハンドリングなし
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

### Async/Awaitベストプラクティス

```typescript
// ✅ 良い：可能な場合は並列実行
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats()
])

// ❌ 悪い：不要な逐次実行
const users = await fetchUsers()
const markets = await fetchMarkets()
const stats = await fetchStats()
```

### 型安全性

```typescript
// ✅ 良い：適切な型
interface Market {
  id: string
  name: string
  status: 'active' | 'resolved' | 'closed'
  created_at: Date
}

function getMarket(id: string): Promise<Market> {
  // 実装
}

// ❌ 悪い：'any'を使用
function getMarket(id: any): Promise<any> {
  // 実装
}
```

## Reactベストプラクティス

### コンポーネント構造

```typescript
// ✅ 良い：型付き関数コンポーネント
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ 悪い：型なし、不明確な構造
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### カスタムフック

```typescript
// ✅ 良い：再利用可能なカスタムフック
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用法
const debouncedQuery = useDebounce(searchQuery, 500)
```

### 状態管理

```typescript
// ✅ 良い：適切な状態更新
const [count, setCount] = useState(0)

// 前の状態に基づく関数的更新
setCount(prev => prev + 1)

// ❌ 悪い：直接状態参照
setCount(count + 1)  // 非同期シナリオで古くなる可能性
```

### 条件レンダリング

```typescript
// ✅ 良い：明確な条件レンダリング
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ 悪い：三項演算子地獄
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API設計標準

### REST API規則

```
GET    /api/markets              # すべてのマーケットをリスト
GET    /api/markets/:id          # 特定のマーケットを取得
POST   /api/markets              # 新しいマーケットを作成
PUT    /api/markets/:id          # マーケットを更新（完全）
PATCH  /api/markets/:id          # マーケットを更新（部分）
DELETE /api/markets/:id          # マーケットを削除

# フィルタリング用クエリパラメータ
GET /api/markets?status=active&limit=10&offset=0
```

### レスポンス形式

```typescript
// ✅ 良い：一貫したレスポンス構造
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
  meta?: {
    total: number
    page: number
    limit: number
  }
}

// 成功レスポンス
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 }
})

// エラーレスポンス
return NextResponse.json({
  success: false,
  error: '無効なリクエスト'
}, { status: 400 })
```

### 入力検証

```typescript
import { z } from 'zod'

// ✅ 良い：スキーマ検証
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1)
})

export async function POST(request: Request) {
  const body = await request.json()

  try {
    const validated = CreateMarketSchema.parse(body)
    // 検証されたデータで続行
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({
        success: false,
        error: '検証に失敗しました',
        details: error.errors
      }, { status: 400 })
    }
  }
}
```

## ファイル構成

### プロジェクト構造

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # APIルート
│   ├── markets/           # マーケットページ
│   └── (auth)/           # 認証ページ（ルートグループ）
├── components/            # Reactコンポーネント
│   ├── ui/               # 汎用UIコンポーネント
│   ├── forms/            # フォームコンポーネント
│   └── layouts/          # レイアウトコンポーネント
├── hooks/                # カスタムReactフック
├── lib/                  # ユーティリティと設定
│   ├── api/             # APIクライアント
│   ├── utils/           # ヘルパー関数
│   └── constants/       # 定数
├── types/                # TypeScript型
└── styles/              # グローバルスタイル
```

### ファイル命名

```
components/Button.tsx          # コンポーネントはPascalCase
hooks/useAuth.ts              # 'use'プレフィックス付きcamelCase
lib/formatDate.ts             # ユーティリティはcamelCase
types/market.types.ts         # .types接尾辞付きcamelCase
```

## コメント・ドキュメント

### コメントするタイミング

```typescript
// ✅ 良い：なぜを説明、何をではない
// 停止中のAPIを圧迫しないよう指数バックオフを使用
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000)

// 大きな配列でのパフォーマンスのため意図的にミューテーションを使用
items.push(newItem)

// ❌ 悪い：明らかなことを述べる
// カウンターを1増加
count++

// 名前をユーザーの名前に設定
name = user.name
```

### パブリックAPIのJSDoc

```typescript
/**
 * セマンティック類似性を使用してマーケットを検索。
 *
 * @param query - 自然言語検索クエリ
 * @param limit - 結果の最大数（デフォルト：10）
 * @returns 類似度スコアでソートされたマーケット配列
 * @throws {Error} OpenAI APIが失敗またはRedis利用不可の場合
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10
): Promise<Market[]> {
  // 実装
}
```

## パフォーマンスベストプラクティス

### メモ化

```typescript
import { useMemo, useCallback } from 'react'

// ✅ 良い：高コストな計算をメモ化
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 良い：コールバックをメモ化
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])
```

### 遅延読み込み

```typescript
import { lazy, Suspense } from 'react'

// ✅ 良い：重いコンポーネントを遅延読み込み
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### データベースクエリ

```typescript
// ✅ 良い：必要な列のみ選択
const { data } = await supabase
  .from('markets')
  .select('id, name, status')
  .limit(10)

// ❌ 悪い：すべてを選択
const { data } = await supabase
  .from('markets')
  .select('*')
```

## テスト標準

### テスト構造（AAAパターン）

```typescript
test('類似度を正しく計算する', () => {
  // Arrange（準備）
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act（実行）
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert（検証）
  expect(similarity).toBe(0)
})
```

### テスト命名

```typescript
// ✅ 良い：説明的なテスト名
test('クエリにマッチするマーケットがない場合は空配列を返す', () => { })
test('OpenAI APIキーが不足している場合はエラーを投げる', () => { })
test('Redis利用不可時は部分文字列検索にフォールバック', () => { })

// ❌ 悪い：曖昧なテスト名
test('動作する', () => { })
test('検索をテスト', () => { })
```

## コードの臭い検出

以下のアンチパターンに注意：

### 1. 長い関数
```typescript
// ❌ 悪い：50行超の関数
function processMarketData() {
  // 100行のコード
}

// ✅ 良い：小さな関数に分割
function processMarketData() {
  const validated = validateData()
  const transformed = transformData(validated)
  return saveData(transformed)
}
```

### 2. 深いネスト
```typescript
// ❌ 悪い：5レベル以上のネスト
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // 何かを実行
        }
      }
    }
  }
}

// ✅ 良い：早期リターン
if (!user) return
if (!user.isAdmin) return
if (!market) return
if (!market.isActive) return
if (!hasPermission) return

// 何かを実行
```

### 3. マジックナンバー
```typescript
// ❌ 悪い：説明のない数値
if (retryCount > 3) { }
setTimeout(callback, 500)

// ✅ 良い：名前付き定数
const MAX_RETRIES = 3
const DEBOUNCE_DELAY_MS = 500

if (retryCount > MAX_RETRIES) { }
setTimeout(callback, DEBOUNCE_DELAY_MS)
```

**覚えておいてください**：コード品質は交渉の余地がありません。明確で保守可能なコードは迅速な開発と自信を持ったリファクタリングを可能にします。