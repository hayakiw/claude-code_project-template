---
name: coding-standards
description: プロジェクトのコーディング規約。バックエンド(Python/FastAPI)とフロントエンド(Next.js/TypeScript)のディレクトリ構成、命名規則、コーディングパターンを定義。
---

# コーディング規約

## バックエンド (Python/FastAPI)

### ディレクトリ構成
```
backend/
├── routers/       # APIエンドポイント定義
├── services/      # ビジネスロジック
├── repositories/  # データアクセス層
├── models/        # データベースモデル定義
├── schemas/       # リクエスト/レスポンスのバリデーション
├── enums/         # 列挙型定義
└── utils/         # ユーティリティ
```

### 命名規則

| 対象 | 規則 | 例 |
|------|------|-----|
| クラス | PascalCase | `UserService`, `UserRepository` |
| 関数/メソッド | snake_case | `get_user_by_id`, `create_user` |
| 変数 | snake_case | `user_data`, `ip_address` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| テーブルクラス | `〇〇Table` | `UserTable`, `OrderTable` |
| 例外クラス | `〇〇Exception` | `UserNotFoundException`, `ValidationException` |
| Enumクラス | PascalCase | `UserStatus`, `OrderType` |

### コーディングパターン

**ルーター（routers/）:**
```python
@router.post("/users")
async def create_user(
    request: Request,
    user_request: UserCreateRequest,
    db: Session = Depends(get_db)
):
    """ユーザーを作成する

    Args:
        request: FastAPIのリクエストオブジェクト
        user_request: リクエストボディ
        db: データベースセッション（DI経由）

    Returns:
        dict: レスポンスオブジェクト
    """
    repository = UserRepository(db)
    service = UserService(repository, db)

    try:
        result = await service.create_user(...)
        return {"status": "ok", "data": result}
    except ValidationException as e:
        return {"status": "error", "type": "validation", "message": str(e)}
```

**サービス層（services/）:**
```python
class UserService:
    def __init__(self, repository: UserRepository, db):
        self.repository = repository
        self.db = db

    async def create_user(self, name: str, email: str):
        """ユーザーを作成する

        Args:
            name: ユーザー名
            email: メールアドレス

        Returns:
            作成されたユーザー情報

        Raises:
            ValidationException: バリデーションエラーの場合
        """
```

**リポジトリ層（repositories/）:**
```python
class UserRepository:
    def __init__(self, db: Session):
        self.db = db

    def get_user_by_id(self, user_id: int):
        return self.db.query(UserTable).filter(
            UserTable.id == user_id
        ).first()
```

**Pydanticスキーマ（schemas/）:**
```python
class UserCreateRequest(BaseModel):
    """ユーザー作成リクエスト"""
    name: str = Field(..., description="ユーザー名")
    email: str = Field(..., description="メールアドレス")

    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if not v or '@' not in v:
            raise ValueError('有効なメールアドレスを入力してください')
        return v.strip()
```

**データベースモデル（models/）:**
```python
class UserTable(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(255), nullable=False)
    email = Column(String(255), nullable=False, unique=True)
    created_at = Column(DateTime, nullable=False, default=datetime.utcnow)

class User(BaseModel):
    id: int
    name: str
    email: str
    created_at: datetime
```

### ログ出力

```python
import logging
logger = logging.getLogger(__name__)

logger.info("処理を開始します")
logger.warning(f"ユーザーが見つかりません: {user_id}")
logger.error(f"予期せぬエラー: {str(e)}", exc_info=True)
```

---

## フロントエンド (Next.js/TypeScript)

### ディレクトリ構成
```
frontend/
├── app/              # App Routerページ
│   ├── (auth)/       # 認証関連ページ
│   └── (main)/       # メインページ
├── components/       # 再利用可能なコンポーネント
│   ├── common/       # 汎用コンポーネント
│   └── features/     # 機能別コンポーネント
├── hooks/            # カスタムフック
├── lib/              # ユーティリティ関数
├── types/            # 型定義
└── public/           # 静的ファイル
```

### 命名規則

| 対象 | 規則 | 例 |
|------|------|-----|
| コンポーネント | PascalCase | `UserCard`, `LoginForm` |
| ファイル名 | コンポーネント名.tsx | `UserCard.tsx` |
| 型定義 | PascalCase | `type Props`, `type User` |
| 変数/関数 | camelCase | `fetchUser`, `isLoading` |
| イベントハンドラ | on〇〇 / handle〇〇 | `onClick`, `handleSubmit` |

### コーディングパターン

**ページコンポーネント:**
```tsx
"use client";

import { useState, useEffect } from "react";
import UserCard from "@/components/UserCard";

export default function UsersPage() {
  const [isLoading, setIsLoading] = useState(false);
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    // 初期化処理
  }, []);

  return (
    <div className="flex flex-col">
      {/* JSX */}
    </div>
  );
}
```

**コンポーネント:**
```tsx
export type User = {
  id: number;
  name: string;
  email: string;
};

type Props = {
  user: User;
  onSelect: (user: User) => void;
};

export default function UserCard({ user, onSelect }: Props) {
  return (
    <div className="p-4 border rounded" onClick={() => onSelect(user)}>
      {/* JSX */}
    </div>
  );
}
```

**API呼び出し:**
```tsx
const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/users`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(formData),
});

if (!res.ok) {
  throw new Error(`HTTP error! status: ${res.status}`);
}

const data = await res.json();
if (data.status === "ok") {
  // 成功処理
}
```

### スタイリング (Tailwind CSS)

```tsx
// レスポンシブ
<div className="flex flex-col sm:flex-row">

// ダークモード対応
<div className="bg-white dark:bg-zinc-700 text-gray-900 dark:text-white">
```

---

## 共通ルール

### APIレスポンス形式
```json
// 成功
{ "status": "ok", "data": { ... } }

// エラー
{ "status": "error", "type": "validation", "message": "エラーメッセージ" }
```

### エラーメッセージ
- ユーザー向けメッセージは**日本語**で記述

### コメント・ドキュメント
- コメントは日本語可
- docstringはGoogle形式（Args, Returns, Raises）

### Git
- コミットメッセージ: `[種別] 内容`
- 種別: `modify`, `fixed`, `add`, `remove` など
