---
name: troubleshooting
description: よくあるエラーと対処法、FAQ。エラーが発生した時、問題解決のヒントが欲しい時に使用。
---

# トラブルシューティング

## 環境構築時のエラー

### Docker関連

**エラー: `docker-compose up` でコンテナが起動しない**
```
Error: Cannot start service api: ...
```

対処法:
```bash
# コンテナとボリュームを完全に削除して再構築
docker-compose down -v
docker-compose up -d --build
```

---

**エラー: ポートが既に使用中**
```
Error: Bind for 0.0.0.0:8000 failed: port is already allocated
```

対処法:
```bash
# 使用中のプロセスを確認
lsof -i :8000
# または
netstat -an | grep 8000

# プロセスを終了するか、docker-compose.yml でポートを変更
```

---

**エラー: DBコンテナが起動しない**

対処法:
```bash
# データディレクトリをクリア（データが消えるので注意）
rm -rf ./db/data/*
docker-compose up -d --build
```

---

### npm関連

**エラー: `npm install` で依存関係エラー**
```
npm ERR! peer dep missing: ...
```

対処法:
```bash
# node_modulesを削除して再インストール
rm -rf node_modules package-lock.json
npm install
```

---

**エラー: Next.jsビルドエラー**
```
Type error: Cannot find module '...'
```

対処法:
```bash
# TypeScriptキャッシュをクリア
rm -rf .next
npm run dev
```

---

## 実行時エラー

### API関連

**エラー: CORSエラー**
```
Access to fetch at 'http://localhost:8000/...' from origin 'http://localhost:3000' has been blocked by CORS policy
```

対処法:
1. `main.py` のCORS設定を確認
2. `.env` の `APP_FRONTEND_URL` が正しいか確認
```python
# main.py
app.add_middleware(
    CORSMiddleware,
    allow_origins=[Settings.APP_FRONTEND_URL],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

**エラー: 500 Internal Server Error**

対処法:
```bash
# ログを確認
docker-compose logs -f api

# よくある原因
# 1. 環境変数が未設定 → .envファイルを確認
# 2. DBに接続できない → DBコンテナが起動しているか確認
# 3. 必要なライブラリが不足 → requirements.txtを確認
```

---

**エラー: 認証エラー（401 Unauthorized）**

原因: トークンが無効または期限切れ

対処法:
1. ブラウザのlocalStorageからトークンをクリア
2. 再度ログイン

```javascript
// 開発者ツールのConsoleで実行
localStorage.removeItem("auth_token");
location.reload();
```

---

### データベース関連

**エラー: テーブルが存在しない**
```
sqlalchemy.exc.ProgrammingError: Table 'xxx' doesn't exist
```

対処法:
```bash
# DBコンテナに入ってマイグレーション実行
docker exec -it <db-container> /bin/bash

# または、Alembicを使用している場合
alembic upgrade head
```

---

**エラー: 文字化け**
```
UnicodeDecodeError: 'utf-8' codec can't decode ...
```

対処法:
1. DBの文字コード設定を確認
```sql
SHOW VARIABLES LIKE 'character%';
```
2. `docker-compose.yml` でDBの文字コードを設定
```yaml
command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

---

## フロントエンドエラー

**エラー: Hydration mismatch**
```
Warning: Text content did not match. Server: "..." Client: "..."
```

対処法:
1. `"use client"` ディレクティブが付いているか確認
2. `useEffect` 内でクライアント専用の処理を行う

```tsx
"use client";
import { useState, useEffect } from "react";

export default function Page() {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) return null;
  // ...
}
```

---

**エラー: localStorage is not defined**

原因: サーバーサイドレンダリング時にlocalStorageにアクセス

対処法:
```tsx
useEffect(() => {
  // useEffect内でlocalStorageにアクセス
  const token = localStorage.getItem("auth_token");
}, []);
```

---

## よくある質問

**Q: セッションをリセットしたい**

A: ブラウザのlocalStorageをクリアしてリロード

---

**Q: 環境変数を変更したが反映されない**

A: Dockerコンテナを再起動する
```bash
docker-compose down
docker-compose up -d --build
```

---

**Q: 開発サーバーが重い・遅い**

A:
1. node_modulesを再インストール
2. .nextフォルダを削除
3. 不要なブラウザタブを閉じる

---

**Q: Git commitがpre-commitフックで失敗する**

A: lint/formatエラーを修正する
```bash
npm run lint
npm run format
```
