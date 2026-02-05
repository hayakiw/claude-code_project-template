---
name: project-structure
description: プロジェクトのディレクトリ構成と主要ファイルの役割。新しいファイルを追加する場所、既存コードの場所を知りたい時に使用。
---

# プロジェクト構成

## 全体構成

```
project-root/
├── frontend/          # フロントエンド
├── backend/           # バックエンド
├── docs/              # ドキュメント
└── .claude/           # Claude Code設定
```

---

## フロントエンド (`frontend/`)

```
frontend/
├── app/                      # Next.js App Router
│   ├── layout.tsx            # ルートレイアウト
│   ├── page.tsx              # トップページ
│   ├── (auth)/               # 認証関連ページ
│   │   ├── login/
│   │   │   └── page.tsx      # ログイン画面
│   │   └── register/
│   │       └── page.tsx      # 登録画面
│   └── (main)/               # メインページ
│       └── dashboard/
│           └── page.tsx      # ダッシュボード
│
├── components/               # 再利用コンポーネント
│   ├── common/               # 汎用コンポーネント
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Loading.tsx
│   └── features/             # 機能別コンポーネント
│
├── hooks/                    # カスタムフック
│   └── useAuth.ts
│
├── lib/                      # ユーティリティ
│   └── api.ts
│
├── types/                    # 型定義
│   └── index.ts
│
├── public/                   # 静的ファイル
├── .env                      # 環境変数
├── .env.example              # 環境変数テンプレート
├── package.json
├── tailwind.config.ts
└── tsconfig.json
```

### 主要ファイルの役割

| ファイル | 役割 |
|---------|------|
| `app/layout.tsx` | 全ページ共通のレイアウト定義 |
| `app/page.tsx` | トップページ |
| `components/common/` | 汎用的なUIコンポーネント |
| `hooks/` | 状態管理やAPIアクセスのカスタムフック |
| `lib/api.ts` | API呼び出しのユーティリティ |

---

## バックエンド (`backend/`)

```
backend/
├── app/
│   ├── main.py              # エントリーポイント
│   ├── config.py            # 環境設定
│   ├── logging_config.py    # ログ設定
│   │
│   ├── routers/             # APIエンドポイント
│   │   ├── root.py          # ヘルスチェック
│   │   ├── auth.py          # 認証API
│   │   └── users.py         # ユーザーAPI
│   │
│   ├── services/            # ビジネスロジック
│   │   ├── auth_service.py
│   │   └── user_service.py
│   │
│   ├── repositories/        # データアクセス層
│   │   └── user_repository.py
│   │
│   ├── models/              # DBモデル
│   │   └── user.py
│   │
│   ├── schemas/             # リクエスト/レスポンススキーマ
│   │   ├── auth.py
│   │   └── user.py
│   │
│   ├── enums/               # 列挙型
│   │   └── user_status.py
│   │
│   ├── db/
│   │   └── database.py      # データベース設定
│   │
│   ├── utils/               # ユーティリティ
│   │   └── helpers.py
│   │
│   ├── .env                 # 環境変数
│   └── .env.example         # 環境変数テンプレート
│
├── tests/                   # テスト
├── Dockerfile
├── requirements.txt
└── docker-compose.yml
```

### 主要ファイルの役割

| ファイル | 役割 |
|---------|------|
| `main.py` | FastAPIアプリ初期化、ミドルウェア、ルーター登録 |
| `config.py` | 環境変数読み込み（DB接続、APIキー等） |
| `routers/` | APIエンドポイント定義 |
| `services/` | ビジネスロジック実装 |
| `repositories/` | データベース操作 |
| `models/` | データベースモデル定義 |
| `schemas/` | リクエスト/レスポンスのバリデーション |

---

## 処理フロー

### 認証フロー（例）
```
[ユーザー] → ログイン画面 → POST /auth/login
    → AuthService.login()
    → UserRepository.get_user_by_email()
    → トークン発行 → localStorage保存
```

### データ取得フロー（例）
```
[ユーザー] → ダッシュボード → GET /users
    → UserService.get_users()
        → ユーザー認証確認
        → UserRepository.get_all()
    → データ表示
```
