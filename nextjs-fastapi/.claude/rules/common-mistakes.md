---
name: common-mistakes
description: よくある間違いとその修正方法
---

# よくある間違いと修正

## データベースクエリ

### ❌ 間違い
```python
# N+1クエリが発生
users = User.query.all()
for user in users:
    print(user.posts)  # 毎回クエリが発生
```

### ✅ 正しい
```python
# eager loadingを使用
users = User.query.options(joinedload(User.posts)).all()
for user in users:
    print(user.posts)
```
