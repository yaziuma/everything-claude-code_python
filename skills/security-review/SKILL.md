---
name: security-review
description: 認証の追加、ユーザー入力の処理、シークレットの取り扱い、APIエンドポイントの作成、支払い/機密機能の実装時にこのスキルを使用。包括的なセキュリティチェックリストとパターンを提供。
---

# セキュリティレビュースキル

このスキルは、すべてのコードがセキュリティベストプラクティスに従い、潜在的な脆弱性を特定することを確保します。

## 有効化タイミング

- 認証または認可の実装
- ユーザー入力またはファイルアップロードの処理
- 新しいAPIエンドポイントの作成
- シークレットまたは認証情報の取り扱い
- 支払い機能の実装
- 機密データの保存または送信
- サードパーティAPIの統合

## セキュリティチェックリスト

### 1. シークレット管理

#### ❌ 絶対にしてはいけないこと
```typescript
const apiKey = "sk-proj-xxxxx"  // ハードコードされたシークレット
const dbPassword = "password123" // ソースコード内
```

#### ✅ 常にすべきこと
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// シークレットが存在することを確認
if (!apiKey) {
  throw new Error('OPENAI_API_KEYが設定されていません')
}
```

#### 確認ステップ
- [ ] ハードコードされたAPIキー、トークン、パスワードなし
- [ ] すべてのシークレットが環境変数に
- [ ] `.env.local`が.gitignoreに
- [ ] git履歴にシークレットなし
- [ ] 本番シークレットがホスティングプラットフォーム（Vercel、Railway）に

### 2. 入力検証

#### 常にユーザー入力を検証
```typescript
import { z } from 'zod'

// 検証スキーマを定義
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 処理前に検証
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### ファイルアップロード検証
```typescript
function validateFileUpload(file: File) {
  // サイズチェック（最大5MB）
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('ファイルが大きすぎます（最大5MB）')
  }

  // タイプチェック
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('無効なファイルタイプです')
  }

  // 拡張子チェック
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('無効なファイル拡張子です')
  }

  return true
}
```

#### 確認ステップ
- [ ] すべてのユーザー入力がスキーマで検証済み
- [ ] ファイルアップロードが制限済み（サイズ、タイプ、拡張子）
- [ ] クエリでユーザー入力を直接使用していない
- [ ] ホワイトリスト検証（ブラックリストではない）
- [ ] エラーメッセージが機密情報を漏洩しない

### 3. SQLインジェクション防止

#### ❌ 絶対にSQL連結しない
```typescript
// 危険 - SQLインジェクション脆弱性
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 常にパラメータ化クエリを使用
```typescript
// 安全 - パラメータ化クエリ
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// または生SQLで
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 確認ステップ
- [ ] すべてのデータベースクエリがパラメータ化クエリを使用
- [ ] SQLで文字列連結なし
- [ ] ORM/クエリビルダーが正しく使用されている
- [ ] Supabaseクエリが適切にサニタイズされている

### 4. 認証・認可

#### JWTトークン処理
```typescript
// ❌ 間違い: localStorage（XSSに脆弱）
localStorage.setItem('token', token)

// ✅ 正しい: httpOnlyクッキー
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 認可チェック
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 常に最初に認可を確認
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: '権限がありません' },
      { status: 403 }
    )
  }

  // 削除を実行
  await db.users.delete({ where: { id: userId } })
}
```

#### 行レベルセキュリティ（Supabase）
```sql
-- すべてのテーブルでRLSを有効化
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- ユーザーは自分のデータのみ表示可能
CREATE POLICY "ユーザーは自分のデータを表示"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- ユーザーは自分のデータのみ更新可能
CREATE POLICY "ユーザーは自分のデータを更新"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 確認ステップ
- [ ] トークンがhttpOnlyクッキーに保存（localStorageではない）
- [ ] 機密操作前の認可チェック
- [ ] SupabaseでRow Level Securityが有効
- [ ] ロールベースアクセス制御が実装済み
- [ ] セッション管理が安全

### 5. XSS防止

#### HTMLをサニタイズ
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 常にユーザー提供のHTMLをサニタイズ
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### コンテンツセキュリティポリシー
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      script-src 'self' 'unsafe-eval' 'unsafe-inline';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  }
]
```

#### 確認ステップ
- [ ] ユーザー提供HTMLがサニタイズ済み
- [ ] CSPヘッダーが設定済み
- [ ] 未検証の動的コンテンツレンダリングなし
- [ ] ReactのXSS保護機能を使用

### 6. CSRF保護

#### CSRFトークン
```typescript
import { csrf } from '@/lib/csrf'

export async function POST(request: Request) {
  const token = request.headers.get('X-CSRF-Token')

  if (!csrf.verify(token)) {
    return NextResponse.json(
      { error: '無効なCSRFトークンです' },
      { status: 403 }
    )
  }

  // リクエストを処理
}
```

#### SameSiteクッキー
```typescript
res.setHeader('Set-Cookie',
  `session=${sessionId}; HttpOnly; Secure; SameSite=Strict`)
```

#### 確認ステップ
- [ ] 状態変更操作でCSRFトークン
- [ ] すべてのクッキーでSameSite=Strict
- [ ] ダブルサブミットクッキーパターンが実装済み

### 7. レート制限

#### APIレート制限
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // ウィンドウあたり100リクエスト
  message: 'リクエストが多すぎます'
})

// ルートに適用
app.use('/api/', limiter)
```

#### 高コスト操作
```typescript
// 検索の積極的レート制限
const searchLimiter = rateLimit({
  windowMs: 60 * 1000, // 1分
  max: 10, // 1分あたり10リクエスト
  message: '検索リクエストが多すぎます'
})

app.use('/api/search', searchLimiter)
```

#### 確認ステップ
- [ ] すべてのAPIエンドポイントでレート制限
- [ ] 高コスト操作でより厳しい制限
- [ ] IPベースレート制限
- [ ] ユーザーベースレート制限（認証済み）

### 8. 機密データ露出

#### ログ
```typescript
// ❌ 間違い: 機密データをログ
console.log('ユーザーログイン:', { email, password })
console.log('支払い:', { cardNumber, cvv })

// ✅ 正しい: 機密データを編集
console.log('ユーザーログイン:', { email, userId })
console.log('支払い:', { last4: card.last4, userId })
```

#### エラーメッセージ
```typescript
// ❌ 間違い: 内部詳細を露出
catch (error) {
  return NextResponse.json(
    { error: error.message, stack: error.stack },
    { status: 500 }
  )
}

// ✅ 正しい: 一般的なエラーメッセージ
catch (error) {
  console.error('内部エラー:', error)
  return NextResponse.json(
    { error: 'エラーが発生しました。再試行してください。' },
    { status: 500 }
  )
}
```

#### 確認ステップ
- [ ] ログにパスワード、トークン、シークレットなし
- [ ] ユーザー向けエラーメッセージは一般的
- [ ] 詳細エラーはサーバーログのみ
- [ ] ユーザーにスタックトレースを露出しない

### 9. ブロックチェーンセキュリティ（Solana）

#### ウォレット検証
```typescript
import { verify } from '@solana/web3.js'

async function verifyWalletOwnership(
  publicKey: string,
  signature: string,
  message: string
) {
  try {
    const isValid = verify(
      Buffer.from(message),
      Buffer.from(signature, 'base64'),
      Buffer.from(publicKey, 'base64')
    )
    return isValid
  } catch (error) {
    return false
  }
}
```

#### トランザクション検証
```typescript
async function verifyTransaction(transaction: Transaction) {
  // 受信者を確認
  if (transaction.to !== expectedRecipient) {
    throw new Error('無効な受信者です')
  }

  // 金額を確認
  if (transaction.amount > maxAmount) {
    throw new Error('金額が制限を超えています')
  }

  // ユーザーが十分な残高を持っているか確認
  const balance = await getBalance(transaction.from)
  if (balance < transaction.amount) {
    throw new Error('残高不足です')
  }

  return true
}
```

#### 確認ステップ
- [ ] ウォレット署名が検証済み
- [ ] トランザクション詳細が検証済み
- [ ] トランザクション前の残高チェック
- [ ] ブラインドトランザクション署名なし

### 10. 依存関係セキュリティ

#### 定期更新
```bash
# 脆弱性をチェック
npm audit

# 自動修正可能な問題を修正
npm audit fix

# 依存関係を更新
npm update

# 古いパッケージをチェック
npm outdated
```

#### ロックファイル
```bash
# 常にロックファイルをコミット
git add package-lock.json

# 再現可能なビルドのためCI/CDで使用
npm ci  # npm installの代わり
```

#### 確認ステップ
- [ ] 依存関係が最新
- [ ] 既知の脆弱性なし（npm audit clean）
- [ ] ロックファイルがコミット済み
- [ ] GitHubでDependabotが有効
- [ ] 定期的なセキュリティ更新

## セキュリティテスト

### 自動セキュリティテスト
```typescript
// 認証をテスト
test('認証が必要', async () => {
  const response = await fetch('/api/protected')
  expect(response.status).toBe(401)
})

// 認可をテスト
test('管理者ロールが必要', async () => {
  const response = await fetch('/api/admin', {
    headers: { Authorization: `Bearer ${userToken}` }
  })
  expect(response.status).toBe(403)
})

// 入力検証をテスト
test('無効な入力を拒否', async () => {
  const response = await fetch('/api/users', {
    method: 'POST',
    body: JSON.stringify({ email: 'not-an-email' })
  })
  expect(response.status).toBe(400)
})

// レート制限をテスト
test('レート制限を強制', async () => {
  const requests = Array(101).fill(null).map(() =>
    fetch('/api/endpoint')
  )

  const responses = await Promise.all(requests)
  const tooManyRequests = responses.filter(r => r.status === 429)

  expect(tooManyRequests.length).toBeGreaterThan(0)
})
```

## デプロイ前セキュリティチェックリスト

本番デプロイメント前に必須:

- [ ] **シークレット**: ハードコードされたシークレットなし、すべて環境変数に
- [ ] **入力検証**: すべてのユーザー入力が検証済み
- [ ] **SQLインジェクション**: すべてのクエリがパラメータ化済み
- [ ] **XSS**: ユーザーコンテンツがサニタイズ済み
- [ ] **CSRF**: 保護が有効
- [ ] **認証**: 適切なトークン処理
- [ ] **認可**: ロールチェックが実装済み
- [ ] **レート制限**: すべてのエンドポイントで有効
- [ ] **HTTPS**: 本番で強制
- [ ] **セキュリティヘッダー**: CSP、X-Frame-Optionsが設定済み
- [ ] **エラーハンドリング**: エラーに機密データなし
- [ ] **ログ**: 機密データがログされていない
- [ ] **依存関係**: 最新、脆弱性なし
- [ ] **Row Level Security**: Supabaseで有効
- [ ] **CORS**: 適切に設定済み
- [ ] **ファイルアップロード**: 検証済み（サイズ、タイプ）
- [ ] **ウォレット署名**: 検証済み（ブロックチェーンの場合）

## リソース

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.jsセキュリティ](https://nextjs.org/docs/security)
- [Supabaseセキュリティ](https://supabase.com/docs/guides/auth)
- [Webセキュリティアカデミー](https://portswigger.net/web-security)

---

**覚えておくこと**: セキュリティはオプションではありません。一つの脆弱性がプラットフォーム全体を危険にさらす可能性があります。疑わしい場合は、慎重な側に立ってください。