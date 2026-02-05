---
name: common-mistakes
description: よくある間違いとその修正方法
---

# よくある間違いと修正

## Eloquent クエリ

### N+1問題

**❌ 間違い:**
```php
// N+1クエリが発生
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count();  // 毎回クエリが発生
}
```

**✅ 正しい:**
```php
// Eager Loadingを使用
$users = User::with('posts')->get();
foreach ($users as $user) {
    echo $user->posts->count();
}
```

---

### 存在確認の非効率なクエリ

**❌ 間違い:**
```php
// 全件取得してからカウント
if (User::all()->count() > 0) {
    // ...
}
```

**✅ 正しい:**
```php
// existsを使用
if (User::exists()) {
    // ...
}
```

---

## バリデーション

### コントローラーでの直接バリデーション

**❌ 間違い:**
```php
public function store(Request $request)
{
    $request->validate([
        'name' => 'required',
        'email' => 'required|email',
    ]);
    // 再利用できない、テストしづらい
}
```

**✅ 正しい:**
```php
// FormRequestを使用
public function store(UserStoreRequest $request)
{
    // バリデーションは自動実行
    $user = User::create($request->validated());
}
```

---

## Mass Assignment

### fillableの設定漏れ

**❌ 間違い:**
```php
class User extends Model
{
    // fillableが未設定
}

// Mass Assignment例外が発生
User::create($request->all());
```

**✅ 正しい:**
```php
class User extends Model
{
    protected $fillable = ['name', 'email', 'password'];
}

// または validated() を使用
User::create($request->validated());
```

---

## Blade テンプレート

### XSS脆弱性

**❌ 間違い:**
```blade
{{-- エスケープなしで出力 --}}
{!! $user->bio !!}
```

**✅ 正しい:**
```blade
{{-- 自動エスケープ --}}
{{ $user->bio }}

{{-- HTMLを許可する場合は明示的にサニタイズ --}}
{!! clean($user->bio) !!}
```

---

### CSRFトークンの欠如

**❌ 間違い:**
```blade
<form method="POST" action="{{ route('users.store') }}">
    {{-- CSRFトークンがない --}}
    <button type="submit">送信</button>
</form>
```

**✅ 正しい:**
```blade
<form method="POST" action="{{ route('users.store') }}">
    @csrf
    <button type="submit">送信</button>
</form>
```

---

## ルーティング

### ルート名の未設定

**❌ 間違い:**
```php
Route::get('/users', [UserController::class, 'index']);

// URLのハードコード
<a href="/users">
```

**✅ 正しい:**
```php
Route::get('/users', [UserController::class, 'index'])->name('users.index');

// ルート名を使用
<a href="{{ route('users.index') }}">
```

---

## 認証・認可

### 認可チェックの漏れ

**❌ 間違い:**
```php
public function edit(User $user)
{
    // 誰でも編集可能
    return view('users.edit', compact('user'));
}
```

**✅ 正しい:**
```php
public function edit(User $user)
{
    $this->authorize('update', $user);
    return view('users.edit', compact('user'));
}
```

---

## 環境変数

### envの直接使用

**❌ 間違い:**
```php
// キャッシュ後に取得できない
$apiKey = env('API_KEY');
```

**✅ 正しい:**
```php
// config経由で取得
$apiKey = config('services.api.key');

// config/services.php
'api' => [
    'key' => env('API_KEY'),
],
```

---

## ファイルアップロード

### バリデーション不足

**❌ 間違い:**
```php
public function upload(Request $request)
{
    // ファイルタイプ検証なし
    $request->file('image')->store('images');
}
```

**✅ 正しい:**
```php
public function upload(Request $request)
{
    $request->validate([
        'image' => ['required', 'image', 'mimes:jpg,png,gif', 'max:2048'],
    ]);

    $path = $request->file('image')->store('images', 'public');
}
```
