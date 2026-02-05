---
name: common-tasks
description: よくある作業手順（環境構築、機能追加、デプロイ）。開発作業の進め方を確認したい時に使用。
---

# よくある作業手順

## 環境構築

### 初回セットアップ

```bash
# 1. 環境変数ファイルをコピー
cp -p backend/.env.example backend/.env
cp -p frontend/.env.example frontend/.env

# 2. 環境変数を設定
# 各.envファイルに必要な値を設定

# 3. バックエンド起動
cd backend
# Docker使用時
docker-compose up -d --build
# または直接起動
pip install -r requirements.txt
uvicorn app.main:app --reload

# 4. フロントエンド起動
cd frontend
npm install
npm run dev
```

### 動作確認URL
- フロントエンド: http://localhost:3000
- APIドキュメント: http://localhost:8000/docs
- ヘルスチェック: http://localhost:8000/health

---

## 開発作業

### バックエンド開発

**Dockerコンテナの操作:**
```bash
cd backend

# 起動
docker-compose up -d --build

# 停止
docker-compose down

# ログ確認
docker-compose logs -f api

# コンテナに入る
docker exec -it <container-name> /bin/bash
```

**コード変更後の反映:**
```bash
docker-compose up -d --build
```

### フロントエンド開発

```bash
cd frontend

# 開発サーバー起動（ホットリロード有効）
npm run dev

# ビルド
npm run build

# Lint実行
npm run lint
```

---

## 新機能追加

### 新しいAPIエンドポイントを追加

1. **スキーマ定義** (`schemas/`に追加)
```python
# schemas/new_feature.py
from pydantic import BaseModel, Field

class NewFeatureRequest(BaseModel):
    """リクエストスキーマ"""
    param: str = Field(..., description="パラメータ説明")
```

2. **サービス層** (`services/`に追加)
```python
# services/new_feature_service.py
class NewFeatureService:
    def __init__(self, repository, db):
        self.repository = repository
        self.db = db

    def process(self, param: str):
        """処理を実装"""
        pass
```

3. **ルーター** (`routers/`に追加)
```python
# routers/new_feature.py
from fastapi import APIRouter, Depends
router = APIRouter()

@router.post("/new-feature")
async def new_feature(request: NewFeatureRequest, db: Session = Depends(get_db)):
    service = NewFeatureService(repository, db)
    return {"status": "ok", "data": service.process(request.param)}
```

4. **ルーター登録** (`main.py`を編集)
```python
from routers import new_feature
app.include_router(new_feature.router)
```

### 新しいフロントエンドページを追加

1. **ページ作成** (`app/`にディレクトリ作成)
```tsx
// app/new-page/page.tsx
"use client";

export default function NewPage() {
  return <div>新しいページ</div>;
}
```

2. **コンポーネント作成** (必要に応じて`components/`に追加)
```tsx
// components/NewComponent.tsx
type Props = {
  title: string;
};

export default function NewComponent({ title }: Props) {
  return <h1>{title}</h1>;
}
```

---

## データ管理

### データベース操作

```bash
# DBコンテナに入る
docker exec -it <db-container-name> /bin/bash

# DBにログイン（MySQLの場合）
mysql -u root -p

# DBにログイン（PostgreSQLの場合）
psql -U postgres

# テーブル確認
SHOW TABLES;  # MySQL
\dt           # PostgreSQL
```

---

## テスト・デバッグ

### API接続テスト

```bash
# ヘルスチェック
curl http://localhost:8000/health

# 認証付きリクエスト
curl -X POST http://localhost:8000/api/endpoint \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"key": "value"}'
```

### ログ確認

```bash
# バックエンドログ
docker-compose logs -f api

# エラーログのみ
docker-compose logs -f api 2>&1 | grep -i error
```

---

## デプロイ

### 本番ビルド

```bash
# フロントエンド
cd frontend
npm run build

# バックエンド（本番用Docker設定が必要）
cd backend
docker-compose -f docker-compose.prod.yml up -d --build
```

### 環境変数の本番設定

```bash
# backend/.env
APP_DEBUG=false
APP_FRONTEND_URL=https://your-domain.com

# frontend/.env
NEXT_PUBLIC_API_URL=https://api.your-domain.com
```
