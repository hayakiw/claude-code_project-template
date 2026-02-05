---
name: project-structure
description: プロジェクトのディレクトリ構成と主要ファイルの役割。新しいファイルを追加する場所、既存コードの場所を知りたい時に使用。
---

# プロジェクト構成

## 全体構成

```
project-root/
├── app/               # アプリケーションコード
├── bootstrap/         # ブートストラップ
├── config/            # 設定ファイル
├── database/          # マイグレーション、シーダー
├── public/            # 公開ディレクトリ
├── resources/         # ビュー、CSS、JS
├── routes/            # ルート定義
├── storage/           # ログ、キャッシュ
├── tests/             # テスト
├── docs/              # ドキュメント
└── .claude/           # Claude Code設定
```

---

## アプリケーション (`app/`)

```
app/
├── Console/
│   └── Commands/             # Artisanコマンド
│
├── Exceptions/
│   └── Handler.php           # 例外ハンドラ
│
├── Http/
│   ├── Controllers/          # コントローラー
│   │   ├── Auth/
│   │   │   ├── LoginController.php
│   │   │   └── RegisterController.php
│   │   ├── UserController.php
│   │   └── DashboardController.php
│   │
│   ├── Middleware/           # ミドルウェア
│   │   ├── Authenticate.php
│   │   └── VerifyCsrfToken.php
│   │
│   └── Requests/             # フォームリクエスト
│       ├── Auth/
│       │   └── LoginRequest.php
│       └── UserStoreRequest.php
│
├── Models/                   # Eloquentモデル
│   ├── User.php
│   └── Post.php
│
├── Services/                 # ビジネスロジック
│   ├── UserService.php
│   └── AuthService.php

├── Enums/                    # 列挙型
│   └── UserStatus.php
│
└── Providers/                # サービスプロバイダ
    ├── AppServiceProvider.php
    └── AuthServiceProvider.php
```

### 主要ファイルの役割

| ファイル | 役割 |
|---------|------|
| `Http/Controllers/` | リクエスト処理、レスポンス返却 |
| `Http/Requests/` | バリデーションルール定義 |
| `Models/` | データベーステーブルのモデル定義 |
| `Services/` | ビジネスロジック実装 |
| `Providers/` | サービス登録、初期化処理 |

---

## ビュー・アセット (`resources/`)

```
resources/
├── views/
│   ├── layouts/              # レイアウト
│   │   └── app.blade.php     # メインレイアウト
│   │
│   ├── components/           # Bladeコンポーネント
│   │   ├── alert.blade.php
│   │   ├── button.blade.php
│   │   └── input.blade.php
│   │
│   ├── partials/             # 部分テンプレート
│   │   ├── header.blade.php
│   │   ├── footer.blade.php
│   │   └── sidebar.blade.php
│   │
│   ├── auth/                 # 認証関連
│   │   ├── login.blade.php
│   │   └── register.blade.php
│   │
│   ├── users/                # ユーザー関連
│   │   ├── index.blade.php   # 一覧
│   │   ├── show.blade.php    # 詳細
│   │   ├── create.blade.php  # 作成フォーム
│   │   └── edit.blade.php    # 編集フォーム
│   │
│   └── dashboard.blade.php   # ダッシュボード
│
├── css/
│   └── app.css               # Tailwind CSS
│
├── js/
│   └── app.js                # JavaScript
│
└── lang/                     # 言語ファイル
    └── ja/
        ├── auth.php
        ├── pagination.php
        └── validation.php
```

### 主要ファイルの役割

| ファイル | 役割 |
|---------|------|
| `views/layouts/` | ページ共通のレイアウト |
| `views/components/` | 再利用可能なUIコンポーネント |
| `views/partials/` | 部分テンプレート（ヘッダー等） |
| `css/app.css` | Tailwind CSSのエントリーポイント |
| `lang/ja/` | 日本語翻訳ファイル |

---

## ルーティング (`routes/`)

```
routes/
├── web.php        # Webルート（セッション、CSRF対応）
├── api.php        # APIルート
├── console.php    # Artisanコマンド
└── channels.php   # ブロードキャストチャンネル
```

---

## データベース (`database/`)

```
database/
├── migrations/           # マイグレーション
│   ├── 2024_01_01_000000_create_users_table.php
│   └── 2024_01_01_000001_create_posts_table.php
│
├── factories/            # ファクトリ
│   └── UserFactory.php
│
└── seeders/              # シーダー
    ├── DatabaseSeeder.php
    └── UserSeeder.php
```

---

## 設定 (`config/`)

```
config/
├── app.php          # アプリケーション設定
├── auth.php         # 認証設定
├── database.php     # データベース設定
├── mail.php         # メール設定
├── session.php      # セッション設定
└── services.php     # 外部サービス設定
```

---

## テスト (`tests/`)

```
tests/
├── Feature/              # 機能テスト
│   ├── Auth/
│   │   └── LoginTest.php
│   └── UserTest.php
│
├── Unit/                 # ユニットテスト
│   └── Services/
│       └── UserServiceTest.php
│
└── TestCase.php          # 基底テストクラス
```

---

## 処理フロー

### 認証フロー（例）
```
[ユーザー] → ログイン画面 → POST /login
    → LoginController@store
    → AuthService.login()
    → セッション作成 → ダッシュボードへリダイレクト
```

### データ取得フロー（例）
```
[ユーザー] → ダッシュボード → GET /users
    → UserController@index
    → UserService.getAllUsers()
        → User::with('posts')->get()
    → Bladeビューでレンダリング
```
