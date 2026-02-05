---
name: troubleshooting
description: よくあるエラーと対処法、FAQ。エラーが発生した時、問題解決のヒントが欲しい時に使用。
---

# トラブルシューティング

## 環境構築時のエラー

### Composer関連

**エラー: `composer install` で依存関係エラー**
```
Your requirements could not be resolved to an installable set of packages.
```

対処法:
```bash
# Composerキャッシュをクリア
composer clear-cache

# lock ファイルを削除して再インストール
rm composer.lock
composer install
```

---

**エラー: PHPバージョンが合わない**
```
This package requires php ^8.1 but your PHP version (7.4) does not satisfy that requirement.
```

対処法:
```bash
# PHPバージョンを確認
php -v

# PHP 8.1以上をインストール・切り替え
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

### データベース関連

**エラー: マイグレーション失敗**
```
SQLSTATE[HY000] [2002] Connection refused
```

対処法:
1. `.env` のDB接続情報を確認
2. DBサーバーが起動しているか確認
```bash
# MySQLの場合
sudo systemctl status mysql

# DBコンテナの場合
docker-compose ps
```

---

**エラー: テーブルが既に存在する**
```
SQLSTATE[42S01]: Base table or view already exists
```

対処法:
```bash
# マイグレーションをリフレッシュ（データが消えるので注意）
php artisan migrate:fresh

# または特定のマイグレーションをロールバック
php artisan migrate:rollback --step=1
```

---

## 実行時エラー

### Laravel関連

**エラー: アプリケーションキーが設定されていない**
```
No application encryption key has been specified.
```

対処法:
```bash
php artisan key:generate
```

---

**エラー: 設定キャッシュ後に環境変数が反映されない**

対処法:
```bash
# 設定キャッシュをクリア
php artisan config:clear

# または全キャッシュをクリア
php artisan optimize:clear
```

---

**エラー: 500 Internal Server Error**

対処法:
```bash
# ログを確認
tail -f storage/logs/laravel.log

# よくある原因
# 1. ストレージの権限問題
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache

# 2. .envファイルが存在しない
cp .env.example .env
php artisan key:generate

# 3. Composerのオートロードが古い
composer dump-autoload
```

---

**エラー: Class not found**
```
Class 'App\Services\UserService' not found
```

対処法:
```bash
# オートロードを再生成
composer dump-autoload

# 名前空間とファイルパスが一致しているか確認
```

---

### 認証関連

**エラー: 419 Page Expired（CSRF）**

対処法:
1. フォームに `@csrf` があるか確認
2. セッションが切れていないか確認
3. CSRFトークンが正しく送信されているか確認
```blade
<form method="POST" action="{{ route('login') }}">
    @csrf
    ...
</form>
```

---

**エラー: 401 Unauthorized / 認証が効かない**

対処法:
1. ミドルウェアが適用されているか確認
```php
// routes/web.php
Route::middleware('auth')->group(function () {
    // 認証が必要なルート
});
```

2. セッションドライバーの設定を確認
```bash
# .env
SESSION_DRIVER=file  # または database, redis
```

---

### Blade関連

**エラー: View not found**
```
View [users.index] not found.
```

対処法:
1. ビューファイルが正しい場所にあるか確認
   - `resources/views/users/index.blade.php`
2. ビューキャッシュをクリア
```bash
php artisan view:clear
```

---

**エラー: Undefined variable**
```
Undefined variable $users
```

対処法:
```php
// コントローラーから変数を渡しているか確認
return view('users.index', compact('users'));

// または
return view('users.index', ['users' => $users]);
```

---

## Vite関連

**エラー: Vite manifest not found**
```
Vite manifest not found at: /path/to/public/build/manifest.json
```

対処法:
```bash
# 開発時
npm run dev

# 本番時
npm run build
```

---

**エラー: HMR（ホットリロード）が効かない**

対処法:
1. Vite設定を確認
```javascript
// vite.config.js
export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

2. 両方のサーバーを起動
```bash
php artisan serve  # ターミナル1
npm run dev        # ターミナル2
```

---

## よくある質問

**Q: セッションをリセットしたい**

A: ブラウザのCookieを削除、または
```bash
php artisan session:flush  # Redisの場合
```

---

**Q: 環境変数を変更したが反映されない**

A:
```bash
php artisan config:clear
# 本番環境の場合
php artisan config:cache
```

---

**Q: 開発サーバーが重い・遅い**

A:
1. `APP_DEBUG=true` になっているか確認
2. 不要なサービスプロバイダを無効化
3. Xdebugを無効化（開発時以外）
4. `php artisan optimize:clear`

---

**Q: マイグレーションを間違えた**

A:
```bash
# 最後のマイグレーションをロールバック
php artisan migrate:rollback --step=1

# マイグレーションファイルを修正後、再実行
php artisan migrate
```

---

**Q: テストデータを再投入したい**

A:
```bash
# マイグレーションリフレッシュ + シーダー
php artisan migrate:fresh --seed
```

---

**Q: Git commitがpre-commitフックで失敗する**

A: lint/formatエラーを修正
```bash
# PHP CS Fixer
./vendor/bin/php-cs-fixer fix

# Laravel Pint
./vendor/bin/pint

# ESLint
npm run lint:fix
```
