# テーブル定義書

本ドキュメントは、SNS機能のデータ構造を `MariaDB 11.4` 向けに定義したものです。  
言語： `PHP 8.2.27` / FW： `Laravel 12` を想定。  
※「Eloquent管理」： `timestamps()` により アプリ側で自動セット（DBデフォルトは付けない／NULL許可）  

---

## 1. member_status（会員ステータス）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                  |
|-------------------|-------------------------|-----------|-------------------|--------|-------------------------------------------------------|
| id                | SMALLINT UNSIGNED       | NO        |                   | PK     | 1=仮登録, 2=本登録, 3=退会                               |
| code              | VARCHAR(32)             | NO        |                   | UK     | `provisional/active/withdrawn`                        |
| label             | VARCHAR(32)             | NO        |                   |        | 表示名                                                 |

---

## 2. members（会員）
| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| name              | VARCHAR(100)            | NO        |                   |        | 名前                                                    |
| account_name      | VARCHAR(64)             | NO        |                   | UK     | ユーザーID（例：#akira, #user001 など）                   |
| member_status_id  | SMALLINT UNSIGNED       | NO        |                   | FK     | member_status.id                                       |
| last_login_at     | DATETIME                | YES       |                   |        | 最終ログイン                                             |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
  
**検索INDEX**：`members(account_name)`（UKで代替）  

---

## 3. member_emails（会員の複数メール）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| member_id         | BIGINT UNSIGNED         | NO        |                   | FK     | members.id                                             |
| email             | VARCHAR(255)            | NO        |                   | UK     |                                                        |
| is_primary        | TINYINT(1)              | NO        | 0                 |        | 主メール（1=true, 0=false）                              |
| primary_member_id | BIGINT UNSIGNED（生成列） | YES       | 仮想列             | UK     | `CASE WHEN is_primary=1 THEN member_id ELSE NULL END` |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
  
**主メール1件の担保**  
- 生成列 `primary_member_id`（VIRTUAL）＋ `UNIQUE (primary_member_id)` で、**is_primary=1 の行のみ会員ごと一意**。
※ マイグレーションでは DB::statement() で ALTER TABLE（仮想列） と CREATE UNIQUE INDEX（UNIQUE設定） を追加。

---

## 4. posts（投稿）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| member_id         | BIGINT UNSIGNED         | NO        |                   | FK     | members.id                                             |
| content           | TEXT                    | NO        |                   |        | 本文                                                    |
| posted_at         | DATETIME                | NO        | CURRENT_TIMESTAMP |        | 投稿日時                                                |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
  
**検索INDEX**：`posts(posted_at)`  

---

## 5. replies（返信：投稿にのみ紐づく）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| post_id           | BIGINT UNSIGNED         | NO        |                   | FK     | posts.id（ON DELETE CASCADE）                           |
| member_id         | BIGINT UNSIGNED         | NO        |                   | FK     | members.id                                             |
| content           | TEXT                    | NO        |                   |        | 本文                                                    |
| posted_at         | DATETIME                | NO        | CURRENT_TIMESTAMP |        | 返信日時                                                |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
  
**検索INDEX**：`replies(posted_at)`  

---

## 6. stamps（スタンプ種別）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | SMALLINT UNSIGNED       | NO        | AUTO_INCREMENT    | PK     |                                                        |
| name              | VARCHAR(64)             | NO        |                   | UK     | スタンプ名                                               |
| image_path        | VARCHAR(255)            | NO        |                   |        | 画像パス/URL                                             |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |

---

## 7. post_stamps（投稿へのスタンプ）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| post_id           | BIGINT UNSIGNED         | NO        |                   | FK     | posts.id（ON DELETE CASCADE）                           |
| member_id         | BIGINT UNSIGNED         | NO        |                   | FK     | members.id                                             |
| stamp_id          | SMALLINT UNSIGNED       | NO        |                   | FK     | stamps.id                                              |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| （複合一意）        | —                       | —         | —                 | UK     | member_id, post_id ： 同一ユーザー×同一投稿＝1件           |

---

## 8. reply_stamps（返信へのスタンプ）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| reply_id          | BIGINT UNSIGNED         | NO        |                   | FK     | replies.id（ON DELETE CASCADE）                         |
| member_id         | BIGINT UNSIGNED         | NO        |                   | FK     | members.id                                             |
| stamp_id          | SMALLINT UNSIGNED       | NO        |                   | FK     | stamps.id                                              |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| （複合一意）        | —                       | —         | —                 | UK     | member_id, reply_id ： 同一ユーザー×同一返信＝1件          |

---

## 9. tags（タグ）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| id                | BIGINT UNSIGNED         | NO        | AUTO_INCREMENT    | PK     |                                                        |
| name              | VARCHAR(64)             | NO        |                   | UK     | タグ名                                                  |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
  
**検索INDEX**：`tags(name)`  

---

## 10. post_tags（投稿×タグ：中間）

| カラム名            | 型                      | NULL許可   | デフォルト         | キー種別 | 備考                                                   |
|-------------------|-------------------------|-----------|-------------------|--------|--------------------------------------------------------|
| post_id           | BIGINT UNSIGNED         | NO        |                   | FK     | posts.id（ON DELETE CASCADE）                           |
| tag_id            | BIGINT UNSIGNED         | NO        |                   | FK     | tags.id（タグは運用上削除しない前提）                       |
| （複合主キー）       | —                       | —         | —                 | PK     | post_id, tag_id ： 重複付与禁止（組み合わせで一意）         |
| created_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
| updated_at        | TIMESTAMP               | YES       |                   |        | Eloquent管理                                           |
  
**検索INDEX**：`post_tags(tag_id)`  

---

## 🔍 検索用 INDEX

- `members(account_name)`（UKで代替）  
- `posts(posted_at)`  
- `replies(posted_at)`  
- `tags(name)`
- `post_tags(tag_id)`

---

## 🗒️ 初期データ

- `member_status`:  
  `(1,'provisional','仮登録'), (2,'active','本登録'), (3,'withdrawn','退会')`  
※ 退会 = 論理削除想定（理由：要件に「退会した会員は退会ステータス」とあるため。）  
