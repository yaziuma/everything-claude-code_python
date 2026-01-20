# 共通パターン

## APIレスポンス形式

```python
from typing import Generic, TypeVar, Optional
from pydantic import BaseModel

T = TypeVar("T")

class PaginationMeta(BaseModel):
    total: int
    page: int
    limit: int
    pages: int

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None
    meta: Optional[PaginationMeta] = None

# 使用例
@router.get("/users", response_model=ApiResponse[list[UserOut]])
async def list_users(page: int = 1, limit: int = 10):
    users, total = await user_service.list_paginated(page, limit)
    return ApiResponse(
        success=True,
        data=users,
        meta=PaginationMeta(
            total=total,
            page=page,
            limit=limit,
            pages=(total + limit - 1) // limit
        )
    )
```

## 依存性注入パターン

```python
from fastapi import Depends
from sqlalchemy.orm import Session

def get_db():
    """データベースセッション依存性"""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

def get_user_service(db: Session = Depends(get_db)) -> UserService:
    """ユーザーサービス依存性"""
    return UserService(UserRepository(db))

@router.get("/users/{user_id}")
async def get_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
):
    return await service.get_by_id(user_id)
```

## リポジトリパターン

```python
from typing import Generic, TypeVar, Optional
from sqlalchemy.orm import Session
from pydantic import BaseModel

ModelType = TypeVar("ModelType")
CreateSchemaType = TypeVar("CreateSchemaType", bound=BaseModel)
UpdateSchemaType = TypeVar("UpdateSchemaType", bound=BaseModel)

class BaseRepository(Generic[ModelType, CreateSchemaType, UpdateSchemaType]):
    def __init__(self, model: type[ModelType], db: Session):
        self.model = model
        self.db = db

    def find_all(
        self,
        skip: int = 0,
        limit: int = 100
    ) -> list[ModelType]:
        return self.db.query(self.model).offset(skip).limit(limit).all()

    def find_by_id(self, id: int) -> Optional[ModelType]:
        return self.db.query(self.model).filter(self.model.id == id).first()

    def create(self, data: CreateSchemaType) -> ModelType:
        db_obj = self.model(**data.model_dump())
        self.db.add(db_obj)
        self.db.commit()
        self.db.refresh(db_obj)
        return db_obj

    def update(self, id: int, data: UpdateSchemaType) -> Optional[ModelType]:
        db_obj = self.find_by_id(id)
        if db_obj:
            for key, value in data.model_dump(exclude_unset=True).items():
                setattr(db_obj, key, value)
            self.db.commit()
            self.db.refresh(db_obj)
        return db_obj

    def delete(self, id: int) -> bool:
        db_obj = self.find_by_id(id)
        if db_obj:
            self.db.delete(db_obj)
            self.db.commit()
            return True
        return False
```

## サービス層パターン

```python
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    async def create_user(self, data: UserCreate) -> User:
        # ビジネスロジック
        if await self.repository.exists_by_email(data.email):
            raise ValueError("メールアドレスは既に使用されています")

        # パスワードハッシュ化
        hashed = hash_password(data.password)

        return self.repository.create(
            UserCreate(
                **data.model_dump(exclude={"password"}),
                password_hash=hashed
            )
        )
```

## htmxパターン

```python
# app/api/users.py
from fastapi import APIRouter, Request
from fastapi.responses import HTMLResponse
from app.templates import templates

router = APIRouter()

@router.get("/users", response_class=HTMLResponse)
async def list_users(request: Request):
    users = await user_service.list_all()
    return templates.TemplateResponse(
        "pages/users.html",
        {"request": request, "users": users}
    )

@router.get("/users/{user_id}", response_class=HTMLResponse)
async def get_user_partial(request: Request, user_id: int):
    """htmx部分更新用"""
    user = await user_service.get_by_id(user_id)
    return templates.TemplateResponse(
        "partials/user_card.html",
        {"request": request, "user": user}
    )
```

```html
<!-- templates/pages/users.html -->
<div id="user-list">
  {% for user in users %}
    <div hx-get="/users/{{ user.id }}"
         hx-trigger="click"
         hx-target="#user-detail"
         hx-swap="innerHTML">
      {{ user.name }}
    </div>
  {% endfor %}
</div>
<div id="user-detail"></div>
```

## スケルトンプロジェクト

新しい機能を実装する際:
1. 実戦でテストされたスケルトンプロジェクトを検索
2. 並列エージェントを使用してオプションを評価:
   - セキュリティ評価
   - 拡張性分析
   - 関連性スコアリング
   - 実装計画
3. 最適なマッチを基盤としてクローン
4. 実証済みの構造内で反復
