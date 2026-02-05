---
description: テストの実行と結果の確認を行います。バックエンド、フロントエンド、E2Eテストに対応。
allowed-tools: Bash, Read
---

# /test - テスト実行

テストの実行と結果の確認を行います。

## 使用方法

```
/test [オプション]
```

## オプション

| オプション | 説明 |
|-----------|------|
| `api` | バックエンドAPIのテスト |
| `frontend` | フロントエンドのテスト |
| `e2e` | E2Eテスト |
| `all` | 全テスト実行 |
| （なし） | 自動判定（変更ファイルに応じて実行） |

## 実行手順

1. オプションに応じてテスト対象を決定
2. 対応するテストコマンドを実行
3. 結果を報告

## バックエンドテスト（api）

### pytestによる自動テスト

```bash
cd backend

# 仮想環境を有効化
source venv/bin/activate  # Windows: venv\Scripts\activate

# テスト実行
pytest tests/ -v

# 特定のテストのみ
pytest tests/test_auth.py -v

# カバレッジ付き
pytest tests/ --cov=app --cov-report=html
```

### 手動テスト

**ヘルスチェック:**
```bash
curl http://localhost:8000/health
```

**認証テスト:**
```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password"}'
```

**認証付きリクエスト:**
```bash
curl -X GET http://localhost:8000/users/me \
  -H "Authorization: Bearer <token>"
```

## フロントエンドテスト（frontend）

### ビルドテスト

```bash
cd frontend

# ビルドが通るか確認
npm run build

# Lint
npm run lint

# 型チェック
npm run type-check
```

### ユニットテスト（Jest/Vitest）

```bash
cd frontend

# テスト実行
npm run test

# カバレッジ付き
npm run test:coverage

# ウォッチモード
npm run test:watch
```

### 手動テスト

1. **初期表示**
   - http://localhost:3000 にアクセス
   - 正しく表示されることを確認

2. **認証フロー**
   - ログイン画面でフォーム入力
   - ログイン成功後、ダッシュボードに遷移

3. **主要機能**
   - 各機能が正常に動作することを確認
   - エラー時に適切なメッセージが表示されること

## E2Eテスト（e2e）

### テストシナリオ

1. **正常系: 認証フロー**
   ```
   1. /login にアクセス
   2. メールアドレスとパスワードを入力
   3. ログインボタンをクリック
   4. ダッシュボードに遷移する
   ```

2. **異常系: バリデーションエラー**
   ```
   1. /login にアクセス
   2. 不正な形式のメールアドレスを入力
   3. エラーメッセージが表示される
   ```

3. **異常系: 認証エラー**
   ```
   1. 保護されたページに未認証でアクセス
   2. ログインページにリダイレクトされる
   ```

## テスト結果の確認

### 期待される結果

**認証成功:**
```json
{
  "status": "ok",
  "data": {
    "token": "jwt-token-string",
    "user": {
      "id": 1,
      "email": "test@example.com"
    }
  }
}
```

**認証失敗:**
```json
{
  "status": "error",
  "type": "authentication",
  "message": "メールアドレスまたはパスワードが正しくありません"
}
```

**バリデーションエラー:**
```json
{
  "status": "error",
  "type": "validation",
  "message": "入力内容を確認してください",
  "details": [
    {"field": "email", "message": "有効なメールアドレスを入力してください"}
  ]
}
```

## トラブルシューティング

テスト失敗時は troubleshooting スキルを参照。
