---
description: コードの品質、セキュリティ、パフォーマンスをレビューします。
allowed-tools: Read, Grep, Glob
---

# /review - コードレビュー

コードの品質、セキュリティ、パフォーマンスをレビューします。

## 使用方法

```
/review [ファイルパスまたはディレクトリ]
```

引数なしの場合、変更されたファイル（`git diff`）をレビューします。

## 実行手順

1. 対象ファイルを特定
   - 引数が指定された場合: そのファイル/ディレクトリを対象
   - 引数なしの場合: `git diff --name-only` で変更ファイルを取得

2. 対象ファイルを読み込み、以下の観点でレビュー

## レビュー観点

### コントローラー (Controllers/)

1. **コーディング規約**
   - 命名規則（PascalCase/camelCase）
   - PHPDocコメント
   - 型宣言

2. **アーキテクチャ**
   - レイヤー分離（Controller → Service → Repository）
   - 依存性の注入（コンストラクタインジェクション）
   - FormRequestの使用

3. **セキュリティ**
   - 認可チェック（Policy/Gate）
   - Mass Assignment対策
   - 入力値バリデーション

4. **パフォーマンス**
   - N+1問題
   - 不要なクエリ

### モデル (Models/)

1. **Eloquent**
   - リレーション定義
   - fillable/guarded設定
   - キャスト設定

2. **クエリ最適化**
   - スコープの使用
   - Eager Loading

### サービス層 (Services/)

1. **ビジネスロジック**
   - 単一責任の原則
   - テスタビリティ

2. **例外処理**
   - 適切な例外の投げ方
   - ログ出力

### Bladeテンプレート (views/)

1. **セキュリティ**
   - XSS対策（エスケープ）
   - CSRF対策

2. **コンポーネント化**
   - 再利用可能なコンポーネント
   - レイアウトの継承

3. **パフォーマンス**
   - N+1回避（@eachの使用注意）
   - 不要なクエリ

### フォームリクエスト (Requests/)

1. **バリデーション**
   - 適切なルール設定
   - カスタムメッセージ（日本語）

2. **認可**
   - authorize()メソッド

## 出力フォーマット

```markdown
## レビュー結果: [ファイル名]

### 問題点

#### [重要度: 高/中/低] 問題タイトル
- **場所**: ファイル:行番号
- **問題**: 問題の説明
- **修正案**:
\`\`\`php
修正後のコード
\`\`\`

### 改善提案

- 改善点1
- 改善点2

### 良い点

- 良い点1
- 良い点2
```

## レビューチェックリスト

### 必須チェック項目

- [ ] エラーメッセージは日本語か
- [ ] FormRequestでバリデーションしているか
- [ ] 認可チェック（Policy/Gate）があるか
- [ ] CSRFトークンが設定されているか
- [ ] SQLインジェクション対策されているか
- [ ] XSS対策（{{ }}でエスケープ）されているか
- [ ] Mass Assignment対策（fillable）されているか

### 推奨チェック項目

- [ ] テストは書かれているか
- [ ] N+1問題がないか（Eager Loading）
- [ ] ログ出力は適切か
- [ ] コードの重複はないか
- [ ] 型宣言が記述されているか

## セキュリティチェック項目

### 認証・認可

```php
// ❌ 悪い例
public function edit(User $user)
{
    return view('users.edit', compact('user'));
}

// ✅ 良い例
public function edit(User $user)
{
    $this->authorize('update', $user);
    return view('users.edit', compact('user'));
}
```

### SQLインジェクション

```php
// ❌ 悪い例
DB::select("SELECT * FROM users WHERE email = '$email'");

// ✅ 良い例
User::where('email', $email)->first();
// または
DB::select("SELECT * FROM users WHERE email = ?", [$email]);
```

### XSS

```blade
{{-- ❌ 悪い例 --}}
{!! $userInput !!}

{{-- ✅ 良い例 --}}
{{ $userInput }}
```

### CSRF

```blade
{{-- ❌ 悪い例 --}}
<form method="POST">
    ...
</form>

{{-- ✅ 良い例 --}}
<form method="POST">
    @csrf
    ...
</form>
```
