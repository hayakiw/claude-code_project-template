# CLAUDE.md

このファイルは、このリポジトリ内のコードを操作する際に Claude Code (claude.ai/code) にガイダンスを提供します。

## プロジェクト概要

<!-- プロジェクトの概要を記載してください -->
<!-- 例: 顧客管理システム。ユーザー登録、認証、データ管理機能を提供。 -->

## 技術スタック

**フロントエンド**
- Next.js (TypeScript)
- React
- Tailwind CSS

**バックエンド**
- Python 3.12+
- FastAPI

**データベース**
- MySQL 8.0（Docker経由、ポート3306）

**インフラ**
- Docker（バックエンドは `backend/docker-compose.yml` で起動）

## ドキュメント構成
```
docs/
├── 01_requirements.md      # 要件定義書
├── 02_design.md            # 設計書
├── 03_api_specification.md # API仕様書（オプション）
└── 04_database_schema.md   # DB設計書（オプション）
```
