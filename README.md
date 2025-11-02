# 問題2：以下のデータのテーブル構造を定義してください。

## 概要

SNS機能の一部を想定したRDB設計。  
言語：`PHP 8.2` / FW：`Laravel 12` / DB：`MariaDB 11.4`  
  
※ 選考用に一時的に Public としています。選考終了後は Private に戻します。  

---

## ディレクトリ構成

```txt
sns-docs/
├── er.drawio.png                   # ER図
├── table_definitions.md            # 各テーブル定義
└── README.md
```

---

## 要件の満たし方

- **会員登録とステータス管理**  
  - 会員情報：`members(name, account_name, member_status_id, last_login_at)` に保持。  
  - ステータス：`member_status` に3区分（仮登録・本登録・退会）を初期データとして保持。  
  - 「退会」は論理削除（`member_status.label = 'withdrawn'`）として扱い、物理削除は行わない。

- **メールアドレス管理**  
  - 一会員につき複数アドレスを保持可能（`member_emails`）。  
  - 主メールは `is_primary=1` のみ1件を許可（生成列 `primary_member_id` + UNIQUE 制約で担保）。  
  - メールアドレスは全体で一意（`email UNIQUE`）。

- **投稿と返信**  
  - 投稿：`posts(member_id, content, posted_at)` に保持。  
  - 返信：`replies(post_id, member_id, content, posted_at)` に保持し、**投稿にのみ紐づく構造**（返信への返信は不可）。  
  - 投稿削除時は関連する返信を自動削除（`ON DELETE CASCADE`）。  
  - 投稿・返信とも本人のみ削除・編集可能（アプリ層で制御）。

- **スタンプ機能**  
  - 種別：`stamps(name, image_path)` で管理。  
  - 投稿用：`post_stamps(post_id, member_id, stamp_id)`  
  - 返信用：`reply_stamps(reply_id, member_id, stamp_id)`  
  - 投稿・返信削除時は関連スタンプも `ON DELETE CASCADE` により自動削除。  
  - 同一ユーザー×同一投稿・返信は1件制約（複合 UNIQUE）。

- **タグ付け**  
  - タグ：`tags(name)`  
  - 中間：`post_tags(post_id, tag_id)` により多対多を実現。  
  - 組み合わせ重複は禁止（複合PK）。  
  - 投稿削除時は中間行も `ON DELETE CASCADE` で自動削除。

- **検索対応**  
  - 会員名検索：`members.account_name`（UKで検索代替）。  
  - タグ検索：`post_tags.tag_id`＋`tags.name` に索引。  
  - 投稿日検索：`posts.posted_at`、`replies.posted_at` に索引。

- **削除ポリシー**  
  - `members` 起点の FK は `NO ACTION`（退会 = 論理削除）。  
    ※ 要件に「退会した会員は退会ステータス」とあるため。  
  - 投稿削除に伴う関連データ（`replies`, `post_stamps`, `reply_stamps`）は `ON DELETE CASCADE` で自動削除。  
