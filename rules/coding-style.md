# コーディングスタイル

## 状態管理と不変性

共有される状態（State）やドメインモデルは不変に保つことを推奨しますが、関数内部の一時変数やリスト操作などはPythonicな方法（ミュータブル）で構いません。

### 推奨：ドメインモデルの不変性

Pydanticモデルを使用し、`frozen=True`を設定することを推奨します：

```python
from pydantic import BaseModel

class User(BaseModel):
    model_config = {"frozen": True}  # イミュータブル

    id: int
    name: str
    email: str

# 更新は新しいインスタンスを作成
# updated_user = user.model_copy(update={"name": new_name})
```

### 許容：ローカル変数のミューテーション

関数スコープ内でのリスト構築などは、パフォーマンスと可読性のためにミュータブルな操作を行っても構いません。

```python
# 許容：リストへの追加
def process_items(items: list[str]) -> list[str]:
    result = []
    for item in items:
        if validate(item):
            result.append(item.lower())  # appendはOK
    return result
```

## ファイル構成

少数の大きなファイルより多数の小さなファイル：
- 高凝集、低結合
- 200-400行が典型、最大800行
- 大きなモジュールからユーティリティを抽出

### ディレクトリ構造

実用的なレイヤーアーキテクチャ（Pragmatic Layered Architecture）を採用：
```
app/
├── main.py              # アプリ起動・設定
├── core/                # 全体設定・インフラ基盤
│   ├── config.py        # 環境変数設定
│   └── db.py            # DB接続・セッション管理
├── api/                 # Presentation層 (Web I/F)
│   ├── dependencies.py  # 共通の依存性注入
│   ├── routers/         # URLルーティング
│   │   ├── users.py
│   │   └── items.py
│   └── schemas/         # Pydanticモデル (API入出力DTO)
│       ├── user.py
│       └── item.py
├── services/            # Application層 (ビジネスロジック)
│   ├── user_service.py
│   └── item_service.py
├── models/              # Domain & Infrastructure (SQLAlchemyモデル)
│   ├── user.py          # DBテーブル定義 兼 ドメインエンティティ
│   └── item.py
├── repositories/        # Infrastructure層 (データアクセス)
│   ├── base.py          # 共通CRUD操作
│   └── user_repo.py     # 具体的なクエリ操作
└── templates/           # Jinja2テンプレート
    ├── base.html
    └── pages/
```

## エラーハンドリング

常に包括的にエラーを処理：

```python
import logging
from fastapi import HTTPException

logger = logging.getLogger(__name__)

async def risky_operation():
    try:
        result = await some_async_task()
        return result
    except ValueError as e:
        logger.error(f"バリデーションエラー: {e}")
        raise HTTPException(
            status_code=400,
            detail="詳細なユーザーフレンドリーメッセージ"
        )
    except Exception as e:
        logger.exception("予期しないエラー")
        raise HTTPException(
            status_code=500,
            detail="内部サーバーエラー"
        )
```

## 入力検証

常にユーザー入力をPydanticで検証：

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    email: EmailStr
    age: int = Field(ge=0, le=150)
    name: str = Field(min_length=1, max_length=100)

# FastAPIエンドポイントで自動検証
@router.post("/users")
async def create_user(user: UserCreate):
    # user は既に検証済み
    return await user_service.create(user)
```

## 型注釈

すべての関数に型注釈を付ける：

```python
from typing import Optional

def get_user_by_id(user_id: int) -> Optional[User]:
    """ユーザーをIDで取得"""
    ...

async def list_users(
    skip: int = 0,
    limit: int = 100
) -> list[User]:
    """ユーザー一覧を取得"""
    ...
```

## コード品質チェックリスト

作業完了をマークする前に：
- [ ] コードが読みやすく適切に命名されている
- [ ] 関数が小さい（50行未満）
- [ ] ファイルが集中している（800行未満）
- [ ] 深いネストなし（4レベル超）
- [ ] 適切なエラーハンドリング
- [ ] print文なし（本番コード）
- [ ] ハードコードされた値なし
- [ ] 共有状態の不変性が守られている
- [ ] 型注釈が完全
- [ ] PEP 8準拠（Ruffでチェック）