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
pytest

# カバレッジ付き
pytest --cov=app --cov-report=html

# 特定のテスト
pytest tests/test_users.py -v

# 失敗で停止
pytest -x

# 並列実行
pytest -n auto
```

## テスト構造

```python
# tests/test_users.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

class TestUserCreate:
    """ユーザー作成のテスト"""

    def test_create_user_success(self):
        """正常なユーザー作成"""
        response = client.post(
            "/api/users",
            json={"email": "test@example.com", "name": "Test"}
        )
        assert response.status_code == 201
        assert response.json()["email"] == "test@example.com"

    def test_create_user_invalid_email(self):
        """無効なメールアドレス"""
        response = client.post(
            "/api/users",
            json={"email": "invalid", "name": "Test"}
        )
        assert response.status_code == 422  # Validation Error
```

## フィクスチャ

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

@pytest.fixture
def db_session():
    """テスト用DBセッション"""
    engine = create_engine("sqlite:///:memory:")
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.close()

@pytest.fixture
def test_user(db_session):
    """テスト用ユーザー"""
    user = User(email="test@example.com", name="Test")
    db_session.add(user)
    db_session.commit()
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

def test_external_api_call():
    with patch("app.services.external_api.call") as mock_call:
        mock_call.return_value = {"status": "ok"}
        result = service.process()
        assert result["status"] == "ok"
        mock_call.assert_called_once()
```

## エージェントサポート

- **tdd-guide** - 新機能にPROACTIVEに使用、テスト優先を強制
- **e2e-runner** - pytest-playwright E2Eテスト専門家
