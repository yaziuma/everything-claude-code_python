---
paths:
  - "app/api/**/*.py"
  - "app/services/**/*.py"
  - "app/repositories/**/*.py"
---
# Backend Patterns

## 推奨パターン
- APIレスポンスは共通ラッパーを使う。
- DIは `Depends` とファクトリ関数で統一。
- リポジトリはCRUDを共通化し、個別クエリを分離。
