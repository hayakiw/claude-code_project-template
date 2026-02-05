---
name: coding-standards
description: プロジェクトのコーディング規約。Laravel/Bladeのディレクトリ構成、命名規則、コーディングパターンを定義。
---

# コーディング規約

## バックエンド (Laravel/PHP)

### ディレクトリ構成
```
app/
├── Http/
│   ├── Controllers/    # コントローラー
│   ├── Middleware/     # ミドルウェア
│   ├── Requests/       # フォームリクエスト（バリデーション）
│   └── Resources/      # APIリソース
├── Models/             # Eloquentモデル
├── Services/           # ビジネスロジック
├── Repositories/       # データアクセス層（オプション）
├── Enums/              # 列挙型（PHP 8.1+）
└── Exceptions/         # カスタム例外
```

### 命名規則

| 対象 | 規則 | 例 |
|------|------|-----|
| クラス | PascalCase | `UserController`, `UserService` |
| メソッド | camelCase | `getUserById`, `createUser` |
| 変数 | camelCase | `userData`, `ipAddress` |
| 定数 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| テーブル名 | snake_case（複数形） | `users`, `order_items` |
| モデル | PascalCase（単数形） | `User`, `OrderItem` |
| マイグレーション | snake_case | `create_users_table` |
| Bladeビュー | kebab-case | `user-profile.blade.php` |

### コーディングパターン

**コントローラー（Controllers/）:**
```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\UserStoreRequest;
use App\Services\UserService;
use Illuminate\Http\RedirectResponse;
use Illuminate\View\View;

class UserController extends Controller
{
    public function __construct(
        private UserService $userService
    ) {}

    /**
     * ユーザー一覧を表示
     */
    public function index(): View
    {
        $users = $this->userService->getAllUsers();

        return view('users.index', compact('users'));
    }

    /**
     * ユーザーを作成
     */
    public function store(UserStoreRequest $request): RedirectResponse
    {
        try {
            $this->userService->createUser($request->validated());

            return redirect()
                ->route('users.index')
                ->with('success', 'ユーザーを作成しました');
        } catch (\Exception $e) {
            return redirect()
                ->back()
                ->withInput()
                ->with('error', 'ユーザーの作成に失敗しました');
        }
    }
}
```

**サービス層（Services/）:**
```php
<?php

namespace App\Services;

use App\Models\User;
use Illuminate\Support\Collection;

class UserService
{
    /**
     * 全ユーザーを取得
     */
    public function getAllUsers(): Collection
    {
        return User::orderBy('created_at', 'desc')->get();
    }

    /**
     * ユーザーを作成
     *
     * @param array $data ユーザーデータ
     * @return User 作成されたユーザー
     */
    public function createUser(array $data): User
    {
        return User::create([
            'name' => $data['name'],
            'email' => $data['email'],
            'password' => bcrypt($data['password']),
        ]);
    }
}
```

**フォームリクエスト（Requests/）:**
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UserStoreRequest extends FormRequest
{
    /**
     * バリデーションルール
     */
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8', 'confirmed'],
        ];
    }

    /**
     * バリデーションメッセージ
     */
    public function messages(): array
    {
        return [
            'name.required' => '名前を入力してください',
            'email.required' => 'メールアドレスを入力してください',
            'email.email' => '有効なメールアドレスを入力してください',
            'email.unique' => 'このメールアドレスは既に登録されています',
            'password.required' => 'パスワードを入力してください',
            'password.min' => 'パスワードは8文字以上で入力してください',
            'password.confirmed' => 'パスワードが一致しません',
        ];
    }
}
```

**Eloquentモデル（Models/）:**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

    /**
     * ユーザーの投稿を取得
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

**Enum（Enums/）:**
```php
<?php

namespace App\Enums;

enum UserStatus: string
{
    case Active = 'active';
    case Inactive = 'inactive';
    case Suspended = 'suspended';

    public function label(): string
    {
        return match($this) {
            self::Active => '有効',
            self::Inactive => '無効',
            self::Suspended => '停止中',
        };
    }
}
```

### ログ出力

```php
use Illuminate\Support\Facades\Log;

Log::info('処理を開始します');
Log::warning('ユーザーが見つかりません', ['user_id' => $userId]);
Log::error('予期せぬエラー', ['exception' => $e->getMessage()]);
```

---

## フロントエンド (Blade/Tailwind CSS)

### ディレクトリ構成
```
resources/
├── views/
│   ├── layouts/          # レイアウト
│   │   └── app.blade.php
│   ├── components/       # Bladeコンポーネント
│   │   ├── button.blade.php
│   │   └── input.blade.php
│   ├── partials/         # 部分テンプレート
│   │   ├── header.blade.php
│   │   └── footer.blade.php
│   ├── auth/             # 認証関連
│   │   ├── login.blade.php
│   │   └── register.blade.php
│   └── users/            # ユーザー関連
│       ├── index.blade.php
│       └── show.blade.php
├── css/
│   └── app.css
└── js/
    └── app.js
```

### Bladeテンプレートパターン

**レイアウト（layouts/app.blade.php）:**
```blade
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', config('app.name'))</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body class="bg-gray-100">
    @include('partials.header')

    <main class="container mx-auto px-4 py-8">
        @if(session('success'))
            <x-alert type="success">{{ session('success') }}</x-alert>
        @endif

        @if(session('error'))
            <x-alert type="error">{{ session('error') }}</x-alert>
        @endif

        @yield('content')
    </main>

    @include('partials.footer')
    @stack('scripts')
</body>
</html>
```

**ページ（users/index.blade.php）:**
```blade
@extends('layouts.app')

@section('title', 'ユーザー一覧')

@section('content')
<div class="bg-white rounded-lg shadow p-6">
    <h1 class="text-2xl font-bold mb-4">ユーザー一覧</h1>

    <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
                <tr>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">名前</th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">メール</th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">操作</th>
                </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-200">
                @forelse($users as $user)
                    <tr>
                        <td class="px-6 py-4 whitespace-nowrap">{{ $user->name }}</td>
                        <td class="px-6 py-4 whitespace-nowrap">{{ $user->email }}</td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <a href="{{ route('users.show', $user) }}" class="text-blue-600 hover:text-blue-900">
                                詳細
                            </a>
                        </td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="3" class="px-6 py-4 text-center text-gray-500">
                            ユーザーが登録されていません
                        </td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>

    {{ $users->links() }}
</div>
@endsection
```

**Bladeコンポーネント（components/button.blade.php）:**
```blade
@props([
    'type' => 'button',
    'variant' => 'primary'
])

@php
$classes = match($variant) {
    'primary' => 'bg-blue-600 hover:bg-blue-700 text-white',
    'secondary' => 'bg-gray-600 hover:bg-gray-700 text-white',
    'danger' => 'bg-red-600 hover:bg-red-700 text-white',
    default => 'bg-blue-600 hover:bg-blue-700 text-white',
};
@endphp

<button
    type="{{ $type }}"
    {{ $attributes->merge(['class' => "px-4 py-2 rounded-md font-medium transition-colors {$classes}"]) }}
>
    {{ $slot }}
</button>
```

### スタイリング (Tailwind CSS)

```blade
{{-- レスポンシブ --}}
<div class="flex flex-col sm:flex-row">

{{-- ダークモード対応 --}}
<div class="bg-white dark:bg-gray-800 text-gray-900 dark:text-white">
```

---

## 共通ルール

### ルーティング
```php
// routes/web.php
Route::middleware('auth')->group(function () {
    Route::resource('users', UserController::class);
});
```

### エラーメッセージ
- ユーザー向けメッセージは**日本語**で記述
- `resources/lang/ja/` に言語ファイルを配置

### コメント・ドキュメント
- コメントは日本語可
- PHPDocコメントを記述（@param, @return, @throws）

### Git
- コミットメッセージ: `[種別] 内容`
- 種別: `modify`, `fixed`, `add`, `remove` など
