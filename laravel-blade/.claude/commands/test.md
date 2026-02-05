---
description: テストの実行と結果の確認を行います。ユニットテスト、機能テスト、ブラウザテストに対応。
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
| `unit` | ユニットテストのみ実行 |
| `feature` | 機能テストのみ実行 |
| `browser` | ブラウザテスト（Dusk）実行 |
| `all` | 全テスト実行 |
| （なし） | 自動判定（変更ファイルに応じて実行） |

## 実行手順

1. オプションに応じてテスト対象を決定
2. 対応するテストコマンドを実行
3. 結果を報告

## PHPUnit テスト

### 全テスト実行

```bash
# 標準実行
php artisan test

# 詳細出力
php artisan test -v

# 並列実行（高速化）
php artisan test --parallel

# カバレッジ付き
php artisan test --coverage
```

### 特定のテスト実行

```bash
# ファイル指定
php artisan test tests/Feature/UserTest.php

# メソッド指定
php artisan test --filter=test_user_can_login

# クラス指定
php artisan test --filter=UserTest

# ディレクトリ指定
php artisan test tests/Unit/
```

### テストグループ実行

```bash
# グループ指定
php artisan test --group=auth

# グループ除外
php artisan test --exclude-group=slow
```

## ブラウザテスト (Laravel Dusk)

### セットアップ

```bash
# Duskインストール
composer require --dev laravel/dusk
php artisan dusk:install

# ChromeDriver更新
php artisan dusk:chrome-driver
```

### テスト実行

```bash
# 全ブラウザテスト
php artisan dusk

# 特定のテスト
php artisan dusk tests/Browser/LoginTest.php

# ヘッドレスモード
php artisan dusk --headless
```

## 手動テスト

### 認証テスト

**1. ログインテスト:**
```
1. /login にアクセス
2. メールアドレスとパスワードを入力
3. ログインボタンをクリック
4. ダッシュボードに遷移することを確認
```

**2. 登録テスト:**
```
1. /register にアクセス
2. 必要事項を入力
3. 登録ボタンをクリック
4. アカウントが作成されることを確認
```

### APIテスト

```bash
# ヘルスチェック
curl http://localhost:8000/api/health

# 認証付きリクエスト
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password"}'

# トークン付きリクエスト
curl -X GET http://localhost:8000/api/user \
  -H "Authorization: Bearer <token>"
```

## テストコード例

### 機能テスト

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_view_dashboard(): void
    {
        // Arrange
        $user = User::factory()->create();

        // Act
        $response = $this->actingAs($user)->get('/dashboard');

        // Assert
        $response->assertStatus(200);
        $response->assertViewIs('dashboard');
    }

    public function test_guest_cannot_view_dashboard(): void
    {
        $response = $this->get('/dashboard');

        $response->assertRedirect('/login');
    }
}
```

### ユニットテスト

```php
<?php

namespace Tests\Unit\Services;

use App\Services\UserService;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserServiceTest extends TestCase
{
    use RefreshDatabase;

    private UserService $userService;

    protected function setUp(): void
    {
        parent::setUp();
        $this->userService = new UserService();
    }

    public function test_create_user_success(): void
    {
        // Arrange
        $data = [
            'name' => 'テスト太郎',
            'email' => 'test@example.com',
            'password' => 'password123',
        ];

        // Act
        $user = $this->userService->createUser($data);

        // Assert
        $this->assertInstanceOf(User::class, $user);
        $this->assertEquals('テスト太郎', $user->name);
        $this->assertDatabaseHas('users', ['email' => 'test@example.com']);
    }
}
```

## テスト結果の確認

### 期待される結果

**成功時:**
```
PASS  Tests\Feature\UserTest
✓ user can view dashboard
✓ guest cannot view dashboard

Tests:    2 passed (5 assertions)
Duration: 0.50s
```

**失敗時:**
```
FAIL  Tests\Feature\UserTest
✕ user can view dashboard

Failed asserting that 404 is identical to 200.
at tests/Feature/UserTest.php:20
```

## トラブルシューティング

テスト失敗時は troubleshooting スキルを参照。

### よくある問題

**1. データベースエラー**
```bash
# テスト用DBを作成
php artisan migrate --env=testing
```

**2. キャッシュの問題**
```bash
php artisan config:clear
php artisan cache:clear
```

**3. ファクトリの問題**
```bash
# ファクトリを確認
php artisan tinker
>>> User::factory()->make()
```
