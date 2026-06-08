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

# できたこと1
## 要素
Bizコミュニケーション、やり遂げる

### 事象
開発タスク演習やAWSへのデプロイ作業の期間中、チームメンバーの多くが実装や検証などの個人作業に集中していた。その中で、自分はスケジュール作成や進捗管理、メンターとの連絡・調整など、開発以外のチーム運営に関する役割を主に担当した。開発作業そのものだけでなく、プロジェクトを円滑に進めるために必要な業務にも主体的に取り組んだ。

### そのとき考えたこと
当時は、各メンバーがそれぞれの担当タスクに集中していたため、チーム全体の進行管理や外部とのコミュニケーションを担う人が必要だと考えた。誰かが対応するのを待つのではなく、自分が担当することでチーム全体の生産性向上につながると判断した。また、開発だけでなく、関係者との連携や情報共有もプロジェクト成功に必要な要素であると考えながら行動した。

### 教訓
プロジェクトでは実装作業だけでなく、スケジュール管理や関係者とのコミュニケーションといった周辺業務も重要であることを学んだ。チーム全体を見渡し、不足している役割を自ら補うことで、より大きな成果につながると実感した。今後も自分の担当範囲にとらわれず、チームに必要な行動を主体的に取っていきたい。

# できたこと2
## 要素
やり遂げる

### 事象
沖縄合宿でのプロダクト開発において、より良い成果物を作るために、機能開発だけでなくテスト工程についても計画に組み込んだ。当初は限られた期間内で完了できるよう一部機能や工程を削減する方向で検討していたが、チーム内で「可能な限り良いプロダクトを作る」という方針を再確認した。その結果、必要な作業をやり切るために作業時間の延長を申請し、計画した機能の大部分を完成させることができた。

### そのとき考えたこと
当初はスケジュール内に収めることを優先していたが、開発の目的は単に期限内に終わらせることではなく、価値のあるプロダクトを作ることだと考え直した。もちろん納期意識は重要だが、目的達成のために必要な努力を惜しまないことも同様に重要だと考え、最後まで妥協せず取り組むことを選択した。

### 教訓
目の前の制約だけにとらわれるのではなく、本来の目的から逆算して判断することの重要性を学んだ。また、高い目標を設定した場合でも、必要な工数やリソースを確保しながら粘り強く取り組むことで成果につなげられることを実感した。今後も困難な状況でも安易に妥協せず、目的達成に向けて最後までやり遂げる姿勢を持ち続けたい。

# できたこと3
## 要素
図々しさ

### 事象
全体ディナーの際に砂川さんへ積極的に話しかけ、自分の考えやキャリア観について意見を伺った。また、村上さんとの朝食会にも参加し、普段の業務ではなかなか接点のない方々と交流する機会を作った。自分から会話を広げることを意識し、さまざまなテーマについて質問や意見交換を行った。

### そのとき考えたこと
こうした場は普段聞けない話を聞ける貴重な機会であり、遠慮しているともったいないと考えた。そのため、多少図々しく感じられたとしても、自分から積極的に話しかけてみようと思った。また、自分の考えを率直に伝えることで、相手からより具体的なフィードバックをもらえると考え、積極的に意見交換を行った。

### 教訓
自分から機会を取りに行かなければ得られない学びがあることを実感した。実際に話を聞く中で、自分では気付いていなかった視点や考え方を知ることができ、一部は自分の価値観や判断基準にも影響を与えた。今後も遠慮しすぎず、学びの機会には積極的に飛び込んでいきたい。

# できたこと4
## 要素
やり遂げる

### 事象
リリース判定会が近づく中で、チームにはまだ複数の未完了タスクが残っていた。限られた時間の中で残作業を確認し、自分が対応可能なタスクを洗い出して着手した。実装後は動作確認やレビュー観点でのセルフチェックを行い、期限までに担当した作業を完了させることができた。

### そのとき考えたこと
リリース直前の実装は不具合を生むリスクが高いため、単純にスピードを優先するのではなく、品質との両立が必要だと考えた。そのため、実装を細かい単位に分けて進め、各段階で動作確認を行いながら作業を進めた。また、チーム全体としてリリース判定会に間に合わせることを優先し、自分にできる貢献を最大限行うことを意識した。

### 教訓
期限が迫った状況でも、冷静に優先順位を整理し、自分が貢献できる領域を見つけて行動することが重要だと学んだ。また、短納期の状況ほど品質管理が重要であり、適切な確認工程を挟むことでスピードと品質を両立できることを実感した。今後も厳しいスケジュールの中であっても、最後まで責任を持って成果につなげていきたい。

# できなかったこと1
## 要素
Bizコミュニケーション

### 事象
開発期間中、一部のタスクについて責任者が明確になっておらず、進捗を確認したりアラートを上げたりする担当者が不在となっていた。その結果、チーム内で「誰かが対応しているだろう」という認識が生まれ、完了していないタスクが後になって発覚した。

### そのとき考えたこと
当時は各メンバーがそれぞれの作業を進めており、大きな問題は起きていないと考えていた。しかし振り返ると、タスクの担当者や進捗状況を明確に管理する仕組みが不足していた。誰かが管理している前提で進めてしまい、自ら確認や声掛けを行わなかった。

### 教訓
チーム開発では、担当者が曖昧なタスクは高い確率で抜け漏れにつながることを学んだ。進捗確認やアラート発信は誰かがやるだろうではなく、明確な責任者を決める必要がある。今後はタスク管理の段階で責任者を明確化し、状況に応じて自分から確認や働きかけを行いたい。

# できなかったこと2
## 要素
Bizコミュニケーション

### 事象
研修期間中は毎日進捗報告を行う予定だったが、合計4回報告を失念した。特に最後の1回は、沖縄合宿で深夜まで作業を延長する申請を行った日であり、普段とは異なるスケジュールで行動していたこともあって報告が漏れてしまった。

### そのとき考えたこと
通常のスケジュールであれば問題なく報告できていたため、自分では運用できていると考えていた。しかし、予定変更や突発対応が発生した際に報告が抜けてしまったことから、個人の記憶や習慣に依存した運用になっていたことに気付いた。

### 教訓
業務で必要な報告は、余裕がある状況だけで回る仕組みでは不十分であると学んだ。忙しい状況やイレギュラーな状況でも確実に実施できるよう、リマインダーやチェックリストなどの仕組みを活用する必要がある。今後は個人の記憶に頼らず、再現性のある運用を構築したい。

# できなかったこと3
## 要素
気が利く

### 事象
GitHubのプルリクエストレビューにおいて、承認基準が十分に厳しくなかった。その結果、リリース直前になって複数の不具合が発見され、チーム全体で修正対応に追われる状況となった。最終的には対応できたものの、品質面での課題が残った。

### そのとき考えたこと
当時は開発スピードを意識するあまり、実装内容の確認やテスト観点のチェックが不十分になっていた。また、自分自身も「大きな問題はないだろう」と考え、潜在的なリスクを深く掘り下げて確認できていなかった。

### 教訓
レビューは単なる形式的な承認作業ではなく、品質を担保する重要な工程であることを改めて認識した。目の前の実装だけでなく、その変更がどのような不具合を引き起こし得るかまで想像して確認する必要がある。今後はレビュー観点を明確にし、品質とスピードのバランスを意識しながら、より精度の高いレビューを行いたい。