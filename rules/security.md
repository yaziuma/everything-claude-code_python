# セキュリティガイドライン

## 必須セキュリティチェック

すべてのコミット前に：
- [ ] ハードコードされた秘密情報なし（APIキー、パスワード、トークン）
- [ ] すべてのユーザー入力が検証済み（Pydantic使用）
- [ ] SQLインジェクション防止（SQLAlchemy ORM使用）
- [ ] XSS防止（Jinja2自動エスケープ有効）
- [ ] CSRF保護が有効（FastAPIでCSRFトークン使用）
- [ ] 認証/認可が確認済み（JWT/セッション）
- [ ] すべてのエンドポイントでレート制限（slowapi等）
- [ ] エラーメッセージが機密データを漏洩しない

## 秘密情報管理

```python
import os
from pydantic_settings import BaseSettings

# 絶対にダメ：ハードコードされた秘密情報
api_key = "sk-proj-xxxxx"

# 常に：環境変数（pydantic-settings推奨）
class Settings(BaseSettings):
    openai_api_key: str
    database_url: str
    secret_key: str

    model_config = {"env_file": ".env"}

settings = Settings()

# 直接os.environを使う場合
api_key = os.environ.get("OPENAI_API_KEY")
if not api_key:
    raise ValueError("OPENAI_API_KEYが設定されていません")
```

## SQLインジェクション防止

```python
# 絶対にダメ：文字列連結
query = f"SELECT * FROM users WHERE id = {user_id}"

# 正しい：SQLAlchemy ORM
user = session.query(User).filter(User.id == user_id).first()

# またはパラメータ化クエリ
from sqlalchemy import text
result = session.execute(
    text("SELECT * FROM users WHERE id = :id"),
    {"id": user_id}
)
```

## XSS防止

```python
# Jinja2は自動エスケープが有効（デフォルト）
# templates/user.html
# {{ user.name }}  ← 自動エスケープされる

# 明示的にエスケープ無効化（危険！）
# {{ user.bio | safe }}  ← 信頼できるデータのみ

# Pythonでのサニタイズ
from markupsafe import escape
safe_input = escape(user_input)
```

## パスワード管理

```python
# passlib使用（推奨）
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)
```

## 認証・認可

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

async def get_current_user(
    token: str = Depends(oauth2_scheme)
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="認証情報を検証できません",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = await get_user(user_id)
    if user is None:
        raise credentials_exception
    return user
```

## セキュリティ対応プロトコル

セキュリティ問題が発見された場合：
1. 即座に停止
2. **security-reviewer**エージェントを使用
3. 続行前に重要な問題を修正
4. 露出した秘密情報をローテーション
5. 類似問題についてコードベース全体をレビュー

## セキュリティツール

```bash
# 依存関係の脆弱性チェック
pip-audit

# 静的セキュリティ分析
bandit -r app/

# 秘密情報スキャン
gitleaks detect

# 依存関係の安全性チェック
safety check
```
