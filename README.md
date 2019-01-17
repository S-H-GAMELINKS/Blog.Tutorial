# Rails初心者のためのBlog作成チュートリアル
## 概要

Railsにこれから初めて触れる方を対象にしたチュートリアルです

## チュートリアル
### Railsのひな型を作る

まず、`rails new`を実行し、Railsアプリのひな型を作成します。

```shell
rails new blog
```

次に、作成したRailsアプリのディレクトリへと移動します。

```shell
cd blog
```

### ScaffoldでCRUDを作成

`rails g scaffold` コマンドを使い、ブログに必要な一覧ページや記事作成ページなどを作ります

```shell
rails g scaffold post title:string content:text date:date
```

その後、`rails db:migrate`でマイグレーションを行います

```shell
rails db:migrate
```

あとは`rails s`を実行して、`localhost:3000/posts`にアクセスします
