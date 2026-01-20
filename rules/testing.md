# テスト要件

## 最小テストカバレッジ：80%

テストタイプ（すべて必須）：
1. **単体テスト** - 個別の関数、ユーティリティ、サービス（pytest）
2. **統合テスト** - APIエンドポイント、データベース操作（pytest + TestClient）
3. **E2Eテスト** - 重要なユーザーフロー（pytest-playwright）

## テスト駆動開発

必須ワークフロー：
1. 最初にテストを書く（RED）
2. テストを実行 - 失敗するはず
3. 最小限の実装を書く（GREEN）
4. テストを実行 - 通るはず
5. リファクタリング（IMPROVE）
6. カバレッジを確認（80%以上）

```bash
# テスト実行
uv run pytest

# カバレッジ付き
uv run pytest --cov=app --cov-report=html

# 特定のテスト
uv run pytest tests/test_users.py -v

# 失敗で停止
uv run pytest -x

# 並列実行
uv run pytest -n auto
```

## テスト構造

```python
# tests/test_users.py
import pytest
from httpx import AsyncClient
from app.main import app

# AsyncClientの使用を推奨 (FastAPIのテスト)
@pytest.mark.asyncio
async def test_create_user_success():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.post(
            "/api/users",
            json={"email": "test@example.com", "name": "Test"}
        )
    assert response.status_code == 201
    assert response.json()["email"] == "test@example.com"
```

## フィクスチャ

```python
# tests/conftest.py
import pytest
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

@pytest.fixture
async def db_session():
    """非同期テスト用DBセッション"""
    # aiosqliteドライバを使用 (メモリ内DB)
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    
    async with engine.begin() as conn:
        # ここで create_all などを呼ぶ（Base.metadataが必要）
        # from app.models import Base
        # await conn.run_sync(Base.metadata.create_all)
        pass

    async_session = async_sessionmaker(engine, expire_on_commit=False)
    
    async with async_session() as session:
        yield session

@pytest.fixture
async def test_user(db_session):
    """テスト用ユーザー"""
    # Userモデルのインポートが必要
    # from app.models import User
    user = User(email="test@example.com", name="Test")
    db_session.add(user)
    await db_session.commit()
    return user
```

## テスト失敗のトラブルシューティング

1. **tdd-guide**エージェントを使用
2. テストの分離をチェック（フィクスチャのスコープ確認）
3. モックが正しいことを確認（`unittest.mock`または`pytest-mock`）
4. テストではなく実装を修正（テストが間違っている場合を除く）

```python
# モックの例
from unittest.mock import patch, MagicMock

@pytest.mark.asyncio
async def test_external_api_call():
    with patch("app.services.external_api.call") as mock_call:
        mock_call.return_value = {"status": "ok"}
        result = await service.process()
        assert result["status"] == "ok"
        mock_call.assert_called_once()
```

## エージェントサポート

- **tdd-guide** - 新機能にPROACTIVEに使用、テスト優先を強制
- **e2e-runner** - pytest-playwright E2Eテスト専門家