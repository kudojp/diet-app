# README

## 概要

- これは、Ruby on Railsで実装された、日々の食事記録を投稿できるSNSアプリである。
- このレポジトリではPR運用をしており、各機能の詳細な設計や開発のログはそのPRページに記入してある。
- CSS, JavaScriptのアセット管理はwebpackを用いている。

## 本番環境

https://diet-app-2020.herokuapp.com

## ローカル環境での機動方法

```
pg_ctl -D /usr/local/var/postgres start
bundle exec rails s
```

## ローカル環境でのテスト走らせ方

```
bundle exec rspec
```


## 技術選定

### 仕様言語とミドルウェア
- Ruby on Rails (Ruby=2.7.0, Rails=6.0.2)
- HTML(erb), SCSS, JavaScript
- PostgreSQL
- Heroku

### 使用ライブラリ

#### Ruby

|ライブラリ名|使用用途|
|----|----|
|devise|認証機能のために使用|
|omniauth|Facebook認証のためにdeviseとともに使用|
|cocoon|cocoon-js(JSライブラリ)とともに、ネストされたモデルのフォーム作成のために使用|
|counter_culture|あるレコードに関連するレコードの個数や値の集計(カウンターキャッシュ)機能のために使用|
|rubocop|ローカル環境でのコードの静的解析のために使用|
|rspec|モデル層のテストのために使用|
|factory_bot_rails|rspecでダミーのモデルのインスタンスを作成するために使用|

#### CSS

|ライブラリ名|使用用途|
|----|----|
|Bootstrap|デザインテンプレートとして使用|

#### JavaScript

|ライブラリ名|使用用途|
|----|----|
|jquery|Ajax通信のためなどに使用|
|flatpickr|カレンダーから日時を選択するUIのために使用|
|cocoon-js|cocoon(gem)とともに、ネストされたモデルのフォーム作成のために使用|

## 機能一覧

- パスワード認証/Facebook認証の2種類のサインアップ/ログイン機能
- 食事ポスト(最低1つ以上の食品目で構成される)の投稿機能/投稿削除機能
- 他人の食事ポストへの投票(いいね/悪いね)機能
- ユーザー間のフォロー機能
- 自分がフォローしているユーザの投稿を表示するフィード機能

## DB設計

#### usersテーブル


|         Column         |              Type             | Nullable |              Default   |           
|-----|----|----|----|
| id                     | bigint                         | not null | nextval('users_id_seq'::regclass)|
| email                  | character varying              | not null | |
| account_id             | character varying              | not null | |
| name                   | character varying              | not null | |
| encrypted_password     | character varying              |          | ''::character varying|
| is_male                | boolean                        |          | |
| height                 | double precision               |          | |
| weight                 | double precision               |          | |
| comment                | text                           |          | |
| reset_password_token   | character varying              |          | |
| reset_password_sent_at | timestamp without time zone    |          | |
| remember_created_at    | timestamp without time zone    |          | |
| created_at             | timestamp(6) without time zone | not null | |
| updated_at             | timestamp(6) without time zone | not null | |
| provider               | character varying              |          | |
| uid                    | character varying              |          | |

```
Indexes:
    "users_pkey" PRIMARY KEY, btree (id)
    "index_users_on_account_id" UNIQUE, btree (account_id)
    "index_users_on_email" UNIQUE, btree (email)
    "index_users_on_reset_password_token" UNIQUE, btree (reset_password_token)
Referenced by:
    TABLE "meal_posts" CONSTRAINT "fk_rails_07c05f4a8d" FOREIGN KEY (user_id) REFERENCES users(id)
    TABLE "votes" CONSTRAINT "fk_rails_c9b3bef597" FOREIGN KEY (user_id) REFERENCES users(id)
```



#### meal_postsテーブル

|         Column         |              Type             | Nullable |              Default   |           
|-----|----|----|----|
| id                             | bigint                         | not null | nextval('meal_posts_id_seq'::regclass)|
| content                        | text                           |          | |
| time                           | timestamp without time zone    |          | |
| user_id                        | bigint                         |          | |
| created_at                     | timestamp(6) without time zone | not null | |
| updated_at                     | timestamp(6) without time zone | not null | |
| total_calories                 | integer                        |          | |
| food_items_count               | integer                        | not null | 0 |
| food_items_with_calories_count | integer                        | not null | 0 |
 
```
Indexes:
    "meal_posts_pkey" PRIMARY KEY, btree (id)
    "index_meal_posts_on_user_id" btree (user_id)
Foreign-key constraints:
    "fk_rails_07c05f4a8d" FOREIGN KEY (user_id) REFERENCES users(id)
Referenced by:
    TABLE "food_items" CONSTRAINT "fk_rails_333bcce849" FOREIGN KEY (meal_post_id) REFERENCES meal_posts(id)
    TABLE "votes" CONSTRAINT "fk_rails_bbb5af58df" FOREIGN KEY (meal_post_id) REFERENCES meal_posts(id)
```


#### food_itemsテーブル

|         Column         |              Type             | Nullable |              Default   |           
|-----|----|----|----|
|id           | bigint            | not null | nextval('food_items_id_seq'::regclass)|
|name         | character varying | not null | |
|amount       | character varying |          | |
|calory       | bigint            |          | |
|meal_post_id | bigint            | not null | |

```
Indexes:
    "food_items_pkey" PRIMARY KEY, btree (id)
    "index_food_items_on_meal_post_id" btree (meal_post_id)
Foreign-key constraints:
    "fk_rails_333bcce849" FOREIGN KEY (meal_post_id) REFERENCES meal_posts(id)
```

#### relationshipsテーブル

|         Column         |              Type             | Nullable |              Default   |           
|-----|----|----|----|
| id          | bigint                        | not null | nextval('relationships_id_seq'::regclass)|
| follower_id | integer                       |          | |
| followed_id | integer                       |          | |
| created_at  | timestamp(6) without time zone| not null | |
| updated_at  | timestamp(6) without time zone| not null | |

```
Indexes:
    "relationships_pkey" PRIMARY KEY, btree (id)
    "index_relationships_on_follower_id_and_followed_id" UNIQUE, btree (follower_id, followed_id)
    "index_relationships_on_followed_id" btree (followed_id)
    "index_relationships_on_follower_id" btree (follower_id)
```

#### votesテーブル

|         Column         |              Type             | Nullable |              Default   |           
|-----|----|----|----|
| id           | bigint                        | not null | nextval('votes_id_seq'::regclass)|
| user_id      | bigint                        | not null | |
| meal_post_id | bigint                        | not null | |
| is_upvote    | boolean                       | not null | |
| created_at   | timestamp(6) without time zone| not null | |
| updated_at   | timestamp(6) without time zone| not null | |

```
Indexes:
    "votes_pkey" PRIMARY KEY, btree (id)
    "index_votes_on_user_id_and_meal_post_id" UNIQUE, btree (user_id, meal_post_id)
    "index_votes_on_meal_post_id" btree (meal_post_id)
    "index_votes_on_user_id" btree (user_id)
Foreign-key constraints:
    "fk_rails_bbb5af58df" FOREIGN KEY (meal_post_id) REFERENCES meal_posts(id)
    "fk_rails_c9b3bef597" FOREIGN KEY (user_id) REFERENCES users(id)
```

## 苦労した点、開発の軌跡

### [# ログイン機構、User 間の Follow 機能、MealPost の投稿機能の追加 #1](https://github.com/kudojp/diet-app/pull/1)

- 最初の実装。この PR 時点では、機能的にはほぼ Rails チュートリアルに近かった。
- 認証には devise を用いた。メールアドレスとパスワードでログイン。
- asset pipeline を使わず、webpack を導入した。
- datetimepicker でカレンダーを表示するのに苦戦した

### [# Voting(投票機能)の追加 #2](https://github.com/kudojp/diet-app/pull/2)

- 仕様設計は[ここ](https://github.com/kudojp/diet-app/pull/2#issue-392782081)
- Voting はいいね(+1)と悪いね(-1)の２種類があり、自分の投稿に投票できない
- 各 MealPost の合計得点に関して時間がかかりすぎるので counter_culture を使うべきだった(後の FoodItem の実装以降導入)

### [# MealPost から[Up|Down]Voters 一覧を、User から[|Un]favorite MealPosts の一覧を見れるようにする #11](https://github.com/kudojp/diet-app/pull/11)

- あるユーザーが[いいね|悪いね]した投稿一覧、ある投稿を[いいね|悪いね]したユーザ一覧をかえすエンドポイントを実装した。肝は[ここ](https://github.com/kudojp/diet-app/pull/11/files#diff-0185a9df92260be0d1b3fc746cb6264b)と[ここ](https://github.com/kudojp/diet-app/pull/11/files#diff-4676c008b11a5480d73d4a6de01e45b9)。

  ```
  class MealPost < ApplicationRecord
    has_many :votes, dependent: :destroy	  has_many :votes, dependent: :destroy
    has_many :upvotes, -> { where(is_upvote: true) }, class_name: 'Vote'
    has_many :upvoters, through: :upvotes, source: :user
    ......
  end
  ```

### [# facebook 認証によるサインインとログイン機能の追加 #4](https://github.com/kudojp/diet-app/pull/4)

- 仕様設計は[ここ](https://github.com/kudojp/diet-app/pull/4#issue-405645108)
- 従来の devise を用いたメールアドレスとパスワードでのログイン形式に加え、ominiauth による Facebook 認証を加えた。
- 本システムでは以下の認証システムをとる

  1.  `FacebookでLogin`ボタンを押下
  2.  Facebook にリダイレクトされ、そこでログインかつ許可ボタンを押下。
  3.  アプリにリダレクトされる。この時点で、認可されて得られる(provider, uid)に対応する User が存在すれば自動的にその User としてログインされ、ホームへ飛ばされる。User が存在しない場合は Login されずにホームへ飛ばされる。

- このために
  1. `OmniauthCallbacksController`と`FacebookUsersController`の 2 つのコントローラを実装した
  2. User テーブルに`provider`と`uid`の２カラムを付け足した(`provider`には`facebook`という文字列が収納される。これは後に Google などの他の外部認証を加えることを想定して作ったカラムである)
- この PR 時点では、 Facecbook 認証では Facebook の認可サーバから取ってきた`provider`と`uid`に合致する列が users テーブルにが存在すれば認証完了、としていた。
- この PR 時点では Facecbook 認証したユーザは password 設定ができない。故に profile や password 更新ができない。これらは以下の PR で直された。

### [# Facebook 認証で登録した User が後からパスワードを設定できるようにする #8](https://github.com/kudojp/diet-app/pull/8)

- 上の PR の続き。仕様設計は[ここ](https://github.com/kudojp/diet-app/pull/8#issue-407352113)
- やったことは主に 3 つ
  1. facebook 認証で登録したユーザーが後から password を設定することができるようにする
  2. 現在の profile 更新ページを 2 つに分ける。これは、身長体重などは password なしで変更できるようにし、password に関しては現在の password 入力が求めるため。
  3. UI の再構成をした。

### [# MealPost のサブ項目として FoodItem を追加する #13](https://github.com/kudojp/diet-app/pull/13)

- 仕様設計は[ここ](https://github.com/kudojp/diet-app/pull/13#issue-409025615)、時間かかったのは[ここ](https://github.com/kudojp/diet-app/pull/13#issuecomment-629778398)と[ここ](https://github.com/kudojp/diet-app/pull/13#issuecomment-629842322)
- MealPost の has_nested_attributes_for で FoodItem モデルを実装した
- cocoon gem を使用することで、FoodItems のフォームの数を柔軟に加減できる UI を構築した。最初にページをロードした時は FoodItem のフォームは 3 つ表示されているが、、(➕) ボタンを押すことでフォームの数を増やしたり、(✖︎) ボタンを押すことでフォームの数を減らしたりできるようにした
- FoodItem はそれぞれカロリー値を入力できる(必須ではない)。MealPost ではそれに属する FoodItems のカロリーの総和を`15kcal+`といった形式で表示する。(`+`はその MealPost に属する全ての FoodItem にカロリーが入力されている場合のみ省略される)これを実現するため、counter_culture gem を導入し、 meal_posts テーブルに`total_calories` `food_items_count` `food_items_with_calories_count`という 3 つのカラムを付け加えた。

## その他

<details><summary>TODOs</summary><div>

### 機能の TODOs

- Google 認証でのサインイン、ログイン機能
- フォロワー/フォロイング数の表示
- プロフィールに画像を追加
- ポストへのコメント機能の追加
- DM チャットやビデオ通話機能の追加

### 環境開発向上のための TODOs

- Github Action による CI/CD の導入
- Docker の導入
- AWS 上にデプロイ
- UI の洗練

</div></details>
