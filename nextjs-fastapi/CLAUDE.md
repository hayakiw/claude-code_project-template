# CLAUDE.md

このファイルは、このリポジトリ内のコードを操作する際に Claude Code (claude.ai/code) にガイダンスを提供します。

## プロジェクト概要

<!-- プロジェクトの概要を記載してください -->
<!-- 例: 顧客管理システム。ユーザー登録、認証、データ管理機能を提供。 -->

## 技術スタック

- **フロントエンド**: Next.js (TypeScript), React, Tailwind CSS
- **バックエンド**: Python 3.12+, FastAPI
- **データベース**: <!-- 例: PostgreSQL, MySQL, MongoDB -->
- **インフラ**: <!-- 例: Docker, AWS, Vercel -->

## 開発コマンド

### バックエンド
```bash
cd backend

# 仮想環境作成・有効化
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 依存関係インストール
pip install -r requirements.txt

# 開発サーバー起動
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# テスト実行
pytest tests/ -v

# リント実行
ruff check .
```
APIドキュメント: http://localhost:8000/docs

### フロントエンド
```bash
cd frontend

# 依存関係インストール
npm install

# 開発サーバー起動 (http://localhost:3000)
npm run dev

# 本番ビルド
npm run build

# リント実行
npm run lint
```

### 環境構築
```bash
# 環境変数ファイルをコピー
cp -p backend/.env.example backend/.env
cp -p frontend/.env.example frontend/.env

# 各.envファイルに必要な値を設定
```

## アーキテクチャ

### バックエンド (`backend/`)

**エントリーポイント:**
- `main.py` - FastAPIアプリ（CORSミドルウェア設定）
- `config.py` - 環境設定（DB接続、APIキー）

**APIルート:**
<!-- 主要なAPIエンドポイントを記載 -->

**サービス層:**
<!-- サービス層の構成を記載 -->

**データ層:**
<!-- データアクセス層の構成を記載 -->

### フロントエンド (`frontend/`)

**ページ:**
<!-- 主要なページを記載 -->

**コンポーネント:**
<!-- 主要なコンポーネントを記載 -->

## データベーススキーマ

<!-- 主要なテーブル/コレクションを記載 -->

## 開発ドキュメント（.claude/）

| ドキュメント | 内容 |
|-------------|------|
| [coding-standards.md](.claude/coding-standards.md) | コーディング規約・命名規則・コードパターン |
| [project-structure.md](.claude/project-structure.md) | ディレクトリ構成・主要ファイルの役割・処理フロー |
| [common-tasks.md](.claude/common-tasks.md) | よくある作業手順（環境構築、機能追加、デプロイ） |
| [troubleshooting.md](.claude/troubleshooting.md) | よくあるエラーと対処法・FAQ |

### カスタムコマンド（.claude/commands/）

| コマンド | 内容 |
|---------|------|
| [/review](.claude/commands/review.md) | コードレビュー（品質・セキュリティ・パフォーマンス） |
| [/test](.claude/commands/test.md) | テスト実行・動作確認手順 |

## コーディング規約（要点）

- エラーメッセージ・コメントは日本語
- APIレスポンスは `{ status: "ok" | "error", ... }` 形式
- バックエンド: PascalCaseクラス、snake_case関数、Google形式docstring
- フロントエンド: PascalCaseコンポーネント、Tailwind CSS、`"use client"`
- コミットメッセージ: `[種別] 内容`（例: `[modify] ログイン画面修正`）
