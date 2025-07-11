# Totonoi Compass
## 概要
優柔不断なあなたへ、提案形式でランダムにサウナを提案するアプリケーションです。Let the saunas suggest decide where we go today!

## 目的/背景
優柔不断な人が今日いくサウナをランダムに提案してあげたい！

## 画面、機能要件
- ログイン画面
  - メールアドレス、パスワードを入力して認証OKだったら、提案画面へ遷移する
- 提案画面(ログイン後)
  - 提案ボタンを押下すると結果画面へ遷移する
  - いい感じのサウナ画像を埋め込んで、おしゃれなレイアウトにしたい
- 結果画面
  - ランダムにサウナを表示してあげる(サウナ名、ホームページのURL、行ったことある/行ったことない のフラグ的なもの)
- 新規登録画面(ログイン不要)
  - ユーザー名、メールアドレス、パスワードを入力し新規登録し、問題なければログイン画面へ遷移する
  - ログイン画面へ戻るボタン
- 登録済みサウナ一覧画面
  - 登録済みサウナ一覧
  - 登録済みサウナ削除機能
  - 新規サウナ登録/編集する機能
- サウナ登録/編集画面
  - サウナ名、ホームページURL、行ったことある/なしラジオボタン をフォーム入力して保存できる

### 技術スタック
- **バックエンド**: Spring Boot
- **データベース**: PostgreSQL
- **フロントエンド**: thymeleaf
- **コンテナ化**: Docker
- **デプロイ方法**: Render（予定）


## テーブル設計
### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  family_name TEXT NOT NULL,
  family_name_ruby TEXT NOT NULL,
  first_name TEXT NOT NULL,
  first_name_ruby TEXT NOT NULL,
  mail TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL,
  created TIMESTAMP DEFAULT NOW(),
  modified TIMESTAMP DEFAULT NOW()
);
```

### saunas
```sql
CREATE TABLE saunas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_mail TEXT NOT NULL,
  name TEXT NOT NULL,
  url TEXT,
  visited BOOLEAN DEFAULT false,
  created TIMESTAMP DEFAULT NOW(),
  modified TIMESTAMP DEFAULT NOW(),
  CONSTRAINT saunas_users_mail_fkey
    FOREIGN KEY (user_mail)
    REFERENCES users(mail)
    ON DELETE CASCADE
);
```

## 画面設計とコントローラー
Thymeleafをビューとして使用するクラシカルなWebアプリケーションの構成。

### 認証不要
| 画面名     | URL (GET) | URL (POST)                 | Controllerメソッド（例）                               | Thymeleafテンプレート | 概要                                     |
| ---------- | --------- | -------------------------- | ------------------------------------------------------ | ------------------- | ---------------------------------------- |
| 新規登録画面 | `/users/signup` | `/users/signup`                  | `UserController#readSignUpPage`, `UserController#createUser`     | `signup.html`       | 新規ユーザー登録を行う                     |
| ログイン画面 | `/login`  | `/login` (Spring Security) | `LoginController#readLoginPage`                                 | `login.html`        | ログインフォームを提供（処理はSpring Security） |

### 要認証
| 画面名         | URL (GET)                      | URL (POST/PUT/DELETE)                         | Controller#メソッド                                     | Thymeleafテンプレート        | 概要                                     |
| -------------- | ------------------------------ | ---------------------------------- | ------------------------------------------------------------ | ---------------------------- | ---------------------------------------- |
| 提案画面       | `/suggest`                     | `/suggest/result`                  | `SuggestController#readSuggestPage`, `SuggestController#suggest`      | `suggest.html`, `result.html`| 提案機能を提供する（ログイン後のトップページ） |
| サウナ一覧（管理） | `/saunas`                | -                                  | `SaunasController#readSaunasPage`                                       | `list.html`            | 登録済みサウナの一覧表示と管理           |
| サウナ登録画面   | `/saunas/new`            | `/saunas/create`             | `SaunasController#readCreateSaunaPage`, `SaunasController#createSauna`    | `form.html`            | 新規サウナを登録する                     |
| サウナ編集画面   | `/saunas/{id}/edit`      | `/saunas/{id}/update`        | `SaunasController#readEditSaunaPage`, `SaunasController#updateSauna`   | `form.html`            | 既存のサウナ情報を更新する               |
| サウナ削除     | -                              | `/saunas/{id}/delete`        | `SaunaController#deleteSauna`                                | -                            | サウナを削除し、一覧画面へリダイレクト   |

## Thymeleafテンプレート構成
`src/main/resources/templates/`
- `signup.html`: 新規登録ページ
- `login.html`: ログインページ
- `suggest.html`: 提案ページ
- `result.html`: 提案結果ページ
- `list.html`: サウナ一覧（管理）ページ
- `form.html`: サウナ登録・編集共通フォームページ
- `layout.html`: 全ページ共通のレイアウト（ヘッダー、フッターなど）

## セキュリティ
- Spring Securityのフォームベース認証を利用する。
- 認証エントリーポイントは `/login` とする。
- ログイン成功後のデフォルト遷移先は `/suggest` とする。
- `/signup`, `/login`のリソースは認証不要でアクセスを許可する。
- 上記以外のすべてのパスへのアクセスは認証を必須とする。
---

## セットアップ方法
1. リポジトリをクローンします。
2. `.env`ファイルを作成し、データベースの接続情報を設定します。
3. Docker Composeを使用してアプリケーションを起動します。

```bash
docker-compose up --build
```
4. ブラウザで`http://localhost:8080`にアクセスします。

5. 起動停止は以下のコマンドで行います。

```bash
# コンテナを停止削除する。バックグラウンド起動時でも止められる
docker-compose down
```
もしくは、ctrl + Cで停止できます(フォアグラウンド実行のみ停止可能)。
