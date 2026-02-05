---
name: common-tasks
description: よくある作業手順（環境構築、機能追加、デプロイ）。開発作業の進め方を確認したい時に使用。
---

# よくある作業手順

## 環境構築

### 初回セットアップ

```bash
# 1. リポジトリをクローン
git clone <repository-url>
cd <project-name>

# 2. 依存パッケージをインストール
composer install
npm install

# 3. 環境変数ファイルをコピー
cp .env.example .env

# 4. アプリケーションキーを生成
php artisan key:generate

# 5. データベース設定
# .env ファイルを編集してDB接続情報を設定

# 6. マイグレーション実行
php artisan migrate

# 7. シーダー実行（テストデータ投入）
php artisan db:seed

# 8. アセットビルド
npm run build

# 9. 開発サーバー起動
php artisan serve
```

### 動作確認URL
- アプリケーション: http://localhost:8000
- ログイン: http://localhost:8000/login

---

## 開発作業

### 開発サーバーの起動

```bash
# Laravel開発サーバー
php artisan serve

# Vite開発サーバー（ホットリロード）
npm run dev

# 両方同時に起動（別ターミナルで）
php artisan serve & npm run dev
```

### Artisanコマンド

**コントローラー作成:**
```bash
# 通常のコントローラー
php artisan make:controller UserController

# リソースコントローラー（CRUD）
php artisan make:controller UserController --resource

# モデルバインディング付き
php artisan make:controller UserController --resource --model=User
```

**モデル作成:**
```bash
# モデルのみ
php artisan make:model Post

# マイグレーション、ファクトリ、シーダー付き
php artisan make:model Post -mfs
```

**フォームリクエスト作成:**
```bash
php artisan make:request UserStoreRequest
```

**マイグレーション作成:**
```bash
php artisan make:migration create_posts_table
php artisan make:migration add_status_to_users_table
```

**サービスクラス作成:**
```bash
# 手動で作成
mkdir -p app/Services
# app/Services/UserService.php を作成
```

### キャッシュ管理

```bash
# 全キャッシュクリア
php artisan optimize:clear

# 個別クリア
php artisan cache:clear    # アプリキャッシュ
php artisan config:clear   # 設定キャッシュ
php artisan route:clear    # ルートキャッシュ
php artisan view:clear     # ビューキャッシュ

# 本番用キャッシュ生成
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## 新機能追加

### 新しいCRUD機能を追加

1. **マイグレーション作成**
```bash
php artisan make:migration create_posts_table
```

```php
// database/migrations/xxxx_create_posts_table.php
public function up(): void
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->string('title');
        $table->text('content');
        $table->timestamps();
    });
}
```

2. **モデル作成**
```bash
php artisan make:model Post
```

```php
// app/Models/Post.php
class Post extends Model
{
    protected $fillable = ['user_id', 'title', 'content'];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

3. **サービス層作成**
```php
// app/Services/PostService.php
class PostService
{
    public function getAllPosts()
    {
        return Post::with('user')->latest()->paginate(10);
    }

    public function createPost(array $data): Post
    {
        return Post::create($data);
    }
}
```

4. **フォームリクエスト作成**
```bash
php artisan make:request PostStoreRequest
```

```php
// app/Http/Requests/PostStoreRequest.php
public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:255'],
        'content' => ['required', 'string'],
    ];
}
```

5. **コントローラー作成**
```bash
php artisan make:controller PostController --resource
```

```php
// app/Http/Controllers/PostController.php
class PostController extends Controller
{
    public function __construct(
        private PostService $postService
    ) {}

    public function index()
    {
        $posts = $this->postService->getAllPosts();
        return view('posts.index', compact('posts'));
    }

    public function store(PostStoreRequest $request)
    {
        $this->postService->createPost([
            'user_id' => auth()->id(),
            ...$request->validated(),
        ]);

        return redirect()->route('posts.index')
            ->with('success', '投稿を作成しました');
    }
}
```

6. **ルート追加**
```php
// routes/web.php
Route::middleware('auth')->group(function () {
    Route::resource('posts', PostController::class);
});
```

7. **ビュー作成**
```bash
mkdir -p resources/views/posts
# index.blade.php, create.blade.php, show.blade.php, edit.blade.php を作成
```

8. **マイグレーション実行**
```bash
php artisan migrate
```

---

## テスト・デバッグ

### テスト実行

```bash
# 全テスト実行
php artisan test

# 特定のテストファイル
php artisan test --filter=UserTest

# カバレッジ付き
php artisan test --coverage

# 並列実行
php artisan test --parallel
```

### デバッグ

```php
// dd() - Dump and Die
dd($variable);

// dump() - 処理を続行
dump($variable);

// Log出力
Log::info('Debug', ['data' => $variable]);
```

### Tinker（対話式シェル）

```bash
php artisan tinker

# Tinker内で
>>> User::all()
>>> User::factory()->create()
>>> Post::where('user_id', 1)->get()
```

---

## デプロイ

### 本番ビルド

```bash
# 依存パッケージインストール（開発用パッケージ除外）
composer install --optimize-autoloader --no-dev

# アセットビルド
npm run build

# キャッシュ最適化
php artisan config:cache
php artisan route:cache
php artisan view:cache

# マイグレーション
php artisan migrate --force
```

### 環境変数の本番設定

```bash
# .env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://your-domain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=your_database
DB_USERNAME=your_username
DB_PASSWORD=your_password

CACHE_DRIVER=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
```

### メンテナンスモード

```bash
# メンテナンスモード有効化
php artisan down

# メンテナンスモード解除
php artisan up

# 特定IPを許可
php artisan down --allow=127.0.0.1
```
