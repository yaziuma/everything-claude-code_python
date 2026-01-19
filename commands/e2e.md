---
description: Playwrightでエンドツーエンドテストを生成・実行。テストジャーニーを作成し、テストを実行し、スクリーンショット/動画/トレースをキャプチャし、アーティファクトをアップロード。
---

# E2Eコマンド

このコマンドは**e2e-runner**エージェントを呼び出して、Playwrightを使用してエンドツーエンドテストを生成、維持、実行します。

## このコマンドが行うこと

1. **テストジャーニー生成** - ユーザーフロー用のPlaywrightテストを作成
2. **E2Eテスト実行** - ブラウザ間でテストを実行
3. **アーティファクトキャプチャ** - 失敗時のスクリーンショット、動画、トレース
4. **結果アップロード** - HTMLレポートとJUnit XML
5. **不安定テスト特定** - 不安定なテストを隔離

## 使用タイミング

以下の場合に`/e2e`を使用:
- 重要なユーザージャーニーのテスト（ログイン、取引、支払い）
- マルチステップフローのエンドツーエンド検証
- UIインタラクションとナビゲーションのテスト
- フロントエンドとバックエンド間の統合検証
- 本番デプロイメントの準備

## 動作方法

e2e-runnerエージェントは以下を行います:

1. **ユーザーフローを分析**してテストシナリオを特定
2. **Playwrightテストを生成**（Page Object Modelパターン使用）
3. **複数ブラウザでテストを実行**（Chrome、Firefox、Safari）
4. **失敗をキャプチャ**（スクリーンショット、動画、トレース付き）
5. **結果とアーティファクト付きレポートを生成**
6. **不安定テストを特定**して修正を推奨

## 使用例

```
ユーザー: /e2e マーケット検索と表示フローをテスト

エージェント（e2e-runner）:
# E2Eテスト生成: マーケット検索と表示フロー

## 特定されたテストシナリオ

**ユーザージャーニー:** マーケット検索 → 結果表示 → マーケットクリック → 詳細表示

**テストカバレッジ:**
1. マーケットページに移動
2. セマンティック検索を実行
3. 検索結果を確認
4. 最初の結果をクリック
5. マーケット詳細ページの読み込みを確認
6. チャートのレンダリングを確認

## 生成されたテストコード

```typescript
// tests/e2e/markets/search-and-view.spec.ts
import { test, expect } from '@playwright/test'
import { MarketsPage } from '../../pages/MarketsPage'
import { MarketDetailsPage } from '../../pages/MarketDetailsPage'

test.describe('マーケット検索と表示フロー', () => {
  test('ユーザーはマーケットを検索して詳細を表示できる', async ({ page }) => {
    // 1. マーケットページに移動
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // ページ読み込みを確認
    await expect(page).toHaveTitle(/Markets/)
    await expect(page.locator('h1')).toContainText('Markets')

    // 2. セマンティック検索を実行
    await marketsPage.searchMarkets('election')

    // APIレスポンスを待機
    await page.waitForResponse(resp =>
      resp.url().includes('/api/markets/search') && resp.status() === 200
    )

    // 3. 検索結果を確認
    const marketCards = marketsPage.marketCards
    await expect(marketCards.first()).toBeVisible()
    const resultCount = await marketCards.count()
    expect(resultCount).toBeGreaterThan(0)

    // 検索結果のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/search-results.png' })

    // 4. 最初の結果をクリック
    const firstMarketTitle = await marketCards.first().textContent()
    await marketCards.first().click()

    // 5. マーケット詳細ページの読み込みを確認
    await expect(page).toHaveURL(/\/markets\/[a-z0-9-]+/)

    const detailsPage = new MarketDetailsPage(page)
    await expect(detailsPage.marketName).toBeVisible()
    await expect(detailsPage.marketDescription).toBeVisible()

    // 6. チャートのレンダリングを確認
    await expect(detailsPage.priceChart).toBeVisible()

    // マーケット名の一致を確認
    const detailsTitle = await detailsPage.marketName.textContent()
    expect(detailsTitle?.toLowerCase()).toContain(
      firstMarketTitle?.toLowerCase().substring(0, 20) || ''
    )

    // マーケット詳細のスクリーンショットを撮影
    await page.screenshot({ path: 'artifacts/market-details.png' })
  })

  test('結果なしの検索で空状態を表示', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 存在しないマーケットを検索
    await marketsPage.searchMarkets('xyznonexistentmarket123456')

    // 空状態を確認
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
    await expect(page.locator('[data-testid="no-results"]')).toContainText(
      /no.*results|no.*markets/i
    )

    const marketCount = await marketsPage.marketCards.count()
    expect(marketCount).toBe(0)
  })

  test('検索をクリアして再度すべてのマーケットを表示', async ({ page }) => {
    const marketsPage = new MarketsPage(page)
    await marketsPage.goto()

    // 初期マーケット数
    const initialCount = await marketsPage.marketCards.count()

    // 検索を実行
    await marketsPage.searchMarkets('trump')
    await page.waitForLoadState('networkidle')

    // フィルタされた結果を確認
    const filteredCount = await marketsPage.marketCards.count()
    expect(filteredCount).toBeLessThan(initialCount)

    // 検索をクリア
    await marketsPage.searchInput.clear()
    await page.waitForLoadState('networkidle')

    // すべてのマーケットが再度表示されることを確認
    const finalCount = await marketsPage.marketCards.count()
    expect(finalCount).toBe(initialCount)
  })
})
```

## テスト実行

```bash
# 生成されたテストを実行
npx playwright test tests/e2e/markets/search-and-view.spec.ts

3ワーカーを使用して3テストを実行中

  ✓  [chromium] › search-and-view.spec.ts:5:3 › ユーザーはマーケットを検索して詳細を表示できる (4.2s)
  ✓  [chromium] › search-and-view.spec.ts:52:3 › 結果なしの検索で空状態を表示 (1.8s)
  ✓  [chromium] › search-and-view.spec.ts:67:3 › 検索をクリアして再度すべてのマーケットを表示 (2.9s)

  3通過 (9.1s)

生成されたアーティファクト:
- artifacts/search-results.png
- artifacts/market-details.png
- playwright-report/index.html
```

## テストレポート

```
╔══════════════════════════════════════════════════════════════╗
║                    E2Eテスト結果                             ║
╠══════════════════════════════════════════════════════════════╣
║ ステータス:  ✅ すべてのテスト通過                            ║
║ 合計:       3テスト                                          ║
║ 通過:       3 (100%)                                         ║
║ 失敗:       0                                                ║
║ 不安定:     0                                                ║
║ 実行時間:   9.1s                                             ║
╚══════════════════════════════════════════════════════════════╝

アーティファクト:
📸 スクリーンショット: 2ファイル
📹 動画: 0ファイル（失敗時のみ）
🔍 トレース: 0ファイル（失敗時のみ）
📊 HTMLレポート: playwright-report/index.html

レポート表示: npx playwright show-report
```

✅ E2EテストスイートがCI/CD統合準備完了！
```

## テストアーティファクト

テスト実行時に以下のアーティファクトがキャプチャされます:

**すべてのテストで:**
- タイムラインと結果付きHTMLレポート
- CI統合用JUnit XML

**失敗時のみ:**
- 失敗状態のスクリーンショット
- テストの動画録画
- デバッグ用トレースファイル（ステップバイステップ再生）
- ネットワークログ
- コンソールログ

## アーティファクト表示

```bash
# ブラウザでHTMLレポートを表示
npx playwright show-report

# 特定のトレースファイルを表示
npx playwright show-trace artifacts/trace-abc123.zip

# スクリーンショットはartifacts/ディレクトリに保存
open artifacts/search-results.png
```

## 不安定テスト検出

テストが断続的に失敗する場合:

```
⚠️  不安定テスト検出: tests/e2e/markets/trade.spec.ts

テストは10回中7回通過（70%通過率）

一般的な失敗:
"要素'[data-testid="confirm-btn"]'の待機タイムアウト"

推奨修正:
1. 明示的待機を追加: await page.waitForSelector('[data-testid="confirm-btn"]')
2. タイムアウトを増加: { timeout: 10000 }
3. コンポーネントの競合状態をチェック
4. 要素がアニメーションで隠されていないか確認

隔離推奨: 修正まではtest.fixme()でマーク
```

## ブラウザ設定

テストはデフォルトで複数ブラウザで実行:
- ✅ Chromium（デスクトップChrome）
- ✅ Firefox（デスクトップ）
- ✅ WebKit（デスクトップSafari）
- ✅ Mobile Chrome（オプション）

ブラウザを調整するには`playwright.config.ts`で設定。

## CI/CD統合

CIパイプラインに追加:

```yaml
# .github/workflows/e2e.yml
- name: Playwrightをインストール
  run: npx playwright install --with-deps

- name: E2Eテストを実行
  run: npx playwright test

- name: アーティファクトをアップロード
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

## PMX固有の重要フロー

PMXでは以下のE2Eテストを優先:

**🔴 重要（常に通過必須）:**
1. ユーザーはウォレットを接続できる
2. ユーザーはマーケットを閲覧できる
3. ユーザーはマーケットを検索できる（セマンティック検索）
4. ユーザーはマーケット詳細を表示できる
5. ユーザーは取引を行える（テスト資金で）
6. マーケットは正しく解決される
7. ユーザーは資金を引き出せる

**🟡 重要:**
1. マーケット作成フロー
2. ユーザープロフィール更新
3. リアルタイム価格更新
4. チャートレンダリング
5. マーケットのフィルタとソート
6. モバイルレスポンシブレイアウト

## ベストプラクティス

**すべきこと:**
- ✅ 保守性のためPage Object Modelを使用
- ✅ セレクタにdata-testid属性を使用
- ✅ 任意のタイムアウトではなくAPIレスポンスを待機
- ✅ 重要なユーザージャーニーをエンドツーエンドでテスト
- ✅ mainにマージ前にテストを実行
- ✅ テスト失敗時にアーティファクトをレビュー

**してはいけないこと:**
- ❌ 脆弱なセレクタを使用（CSSクラスは変更される可能性）
- ❌ 実装詳細をテスト
- ❌ 本番環境でテストを実行
- ❌ 不安定テストを無視
- ❌ 失敗時のアーティファクトレビューをスキップ
- ❌ すべてのエッジケースをE2Eでテスト（ユニットテストを使用）

## 重要な注意事項

**PMXにとって重要:**
- 実際のお金を含むE2Eテストはtestnet/stagingでのみ実行必須
- 本番環境で取引テストを実行しない
- 金融テストには`test.skip(process.env.NODE_ENV === 'production')`を設定
- 少額のテスト資金のみでテストウォレットを使用

## 他のコマンドとの統合

- テストする重要なジャーニーを特定するために`/plan`を使用
- ユニットテスト（より高速、より詳細）には`/tdd`を使用
- 統合とユーザージャーニーテストには`/e2e`を使用
- テスト品質を確認するために`/code-review`を使用

## 関連エージェント

このコマンドは以下にある`e2e-runner`エージェントを呼び出します:
`~/.claude/agents/e2e-runner.md`

## クイックコマンド

```bash
# すべてのE2Eテストを実行
npx playwright test

# 特定のテストファイルを実行
npx playwright test tests/e2e/markets/search.spec.ts

# ヘッドモードで実行（ブラウザを表示）
npx playwright test --headed

# テストをデバッグ
npx playwright test --debug

# テストコードを生成
npx playwright codegen http://localhost:3000

# レポートを表示
npx playwright show-report
```