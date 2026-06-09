# my-first-public-repository

# Git 強制同期系コマンド集

## origin/main の状態へ完全リセット

ローカル変更をすべて破棄し、GitHub 上の `origin/main` と完全一致させる。

```bash
git fetch origin
git reset --hard origin/main
git clean -fd
```

- staged (`Changes to be committed`) を破棄
- unstaged (`Changes not staged for commit`) を破棄
- untracked file も削除
- ローカルを `origin/main` と完全一致

---

## 作業ブランチを local main に強制追従

現在の作業ブランチへ `main` を取り込み、コンフリクト時は `main` 側を優先する。

```bash
git merge main -X theirs
```

- コンフリクト時に `main` の内容を採用
- 作業ブランチを `main` 寄りに同期

---

## GitHub に push した commit を消す（履歴改変）

直前の commit を消し、その状態を GitHub に強制反映する。

```bash
git reset --hard HEAD~1
git push --force origin main
```

- `HEAD~1`
  = 1 commit 前へ戻る
- GitHub 側の履歴も上書きされる
- 共同開発では危険

### 安全版（履歴を壊さない）

```bash
git revert HEAD
git push origin main
```

- 打ち消し commit を追加
- 履歴は保持される
- チーム開発向け





# `@AuthenticationPrincipal Jwt jwt` の使い方

## 概要

```java
@AuthenticationPrincipal Jwt jwt
```

と書くと、Spring Security が現在ログイン中ユーザーの JWT を controller 引数に自動注入する。

内部的には `SecurityContextHolder` から取得している。

---

# 使用例

```java
@RestController
public class PostController {

    @PostMapping("/posts")
    public void createPost(
            @AuthenticationPrincipal Jwt jwt,
            @RequestBody CreatePostRequest request
    ) {

        String userId = jwt.getSubject();

        System.out.println(userId);

        System.out.println(request.text());
    }
}
```

---

# 何が起きているか

クライアント:

```http
POST /posts
Authorization: Bearer eyJ...
```

↓

Spring Security が access token(JWT) を検証

↓

`Authentication` を生成

↓

`SecurityContextHolder` に保存

↓

`@AuthenticationPrincipal` が JWT を controller 引数へ注入

---

# よく使うメソッド

## ユーザーID(sub)

```java
jwt.getSubject()
```

---

## claim取得

```java
jwt.getClaim("email")
```

```java
jwt.getClaim("cognito:username")
```

---

# 内部的にやっていること

以下を Spring が自動化している。

```java
Authentication auth =
    SecurityContextHolder.getContext()
        .getAuthentication();

Jwt jwt = (Jwt) auth.getPrincipal();
```

---

# 利点

- 毎回 `SecurityContextHolder` を書かなくてよい
- 現在ユーザーを簡単に取得できる
- userId を request body に含める必要がない
- JWT認証との相性が良い

---

# 注意

通常、ユーザー識別は JWT から行う。

危険:

```json
{
  "userId": "admin"
}
```

安全:

```java
String userId = jwt.getSubject();
```

# search

```java
@Repository
@RequiredArgsConstructor
public class SearchRepository {

    private final NamedParameterJdbcTemplate jdbcTemplate;

    public List<Post> getTargetPosts(
            String key,
            List<Integer> tagIdList
    ) {

        StringBuilder sql = new StringBuilder("""
            SELECT DISTINCT p.*
            FROM post p
            LEFT JOIN post_tag pt
                ON p.id = pt.post_id
            WHERE 1 = 1
        """);

        MapSqlParameterSource params = new MapSqlParameterSource();

        // key の部分一致
        if (key != null && !key.isBlank()) {

            sql.append("""
                AND p.content ILIKE :key
            """);

            params.addValue("key", "%" + key + "%");
        }

        // tag の OR 検索
        if (tagIdList != null && !tagIdList.isEmpty()) {

            sql.append("""
                AND pt.tag_id IN (:tagIds)
            """);

            params.addValue("tagIds", tagIdList);
        }

        return jdbcTemplate.query(
                sql.toString(),
                params,
                (rs, rowNum) -> new Post(
                        rs.getInt("id"),
                        rs.getString("content")
                )
        );
    }
}
```

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api")
public class SearchController {

    private final SearchRepository searchRepository;

    @GetMapping("/search")
    public List<Post> getTargetPosts(
            @RequestParam(required = false) String key,

            @RequestParam(
                    name = "tag",
                    required = false
            )
            List<Integer> tagIdList
    ) {

        return searchRepository.getTargetPosts(
                key,
                tagIdList
        );
    }
}
```

# メモ

- `/api/search?key=Java&tag=1&tag=2`
  のような URL は自然

- `tag=1&tag=2`
  は Spring が `List<Integer>` に自動変換

- `key=` は空文字 `""`
  `key` 自体が無い場合は `null`

- `WHERE 1 = 1`
  は後ろに `AND` を機械的に追加


