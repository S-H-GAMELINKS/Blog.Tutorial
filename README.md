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

### コメント機能の作成

折角ですので、コメントを投稿できるようにしたいですよね？

まず、コメントを取り扱う`Comment`モデルを作成します。

```shell
rails g model comment content:text post:references
```

マイグレーションを行います

```shell
rails db:migrate
```

`app/models/post.rb`に`Comment`との関連付けを記述します

```ruby:app/models/post.rb
class Post < ApplicationRecord
    has_many :comments
end
```

そして、`config/routes.rb` にてルーティングを設定します

```ruby:config/routes.rb
Rails.application.routes.draw do
  resources :posts do
    resources :comments, :only => [:create, :destroy]
  end
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

次に、各記事へのコメントフォームを作成していきます。

まずは下記のように`app/views/posts/show.html.erb`を変更します

```erb:app/views/posts/show.html.erb
<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @post.title %>
</p>

<p>
  <strong>Content:</strong>
  <%= @post.content %>
</p>

<p>
  <strong>Date:</strong>
  <%= @post.date %>
</p>

<h2>Comments</h2>
  <div id="comments">
    <%= render @post.comments %>
  </div>

<%= render 'comments/new', post: @post %> 

<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Back', posts_path %>
```

変更箇所としてはこの部分になります

```erb
<h2>Comments</h2>
  <div id="comments">
    <%= render @post.comments %>
  </div>

<%= render 'comments/new', post: @post %> 
```

`<%= render @post.comments %>` の部分でコメントを一覧できるviewファイルをパーシャルとして呼び出しています

また、`<%= render 'comments/new', post: @post %> `の部分では新規作成するコメントのviewファイルをパーシャルとして呼び出しています

パーシャルとして呼び出している部分をそれぞれ作っていきます

まず、`app/views/comments/_comment.html.erb` を作成し、下記のようにします。

```erb:app/views/comments/_comment.html.erb
<p><%= comment.content %></p>
<p><%= link_to "Delete", "#{comment.post_id}/comments/#{comment.id}", method: :delete, data: { confirm: 'Are you sure?' } %> 
```

次に、`app/views/comments/_new.html.erb`を作成します

```erb:app/views/comments/_new.html.erb
<%= form_for([ @post, Comment.new ], remote: true) do |form| %>
    Your Comment:<br>
    <%= form.text_area :content, size: '50x20' %><br>
    <%= form.submit %>
<% end %> 
```

これで、コメントの入力フォームが表示されるようになっているはずです

次に、コメントの作成と削除を行うアクションを作成します

`app/controllers/comments_controller.rb`を作成します

```ruby:app/controllers/comments_controller.rb
class CommentsController < ApplicationController
    before_action :set_post

    def create
        @post.comments.create! comments_params
        redirect_to @post
    end

    def destroy
        @post.comments.destroy params[:id]
        redirect_to @post
    end

     private
        def set_post
            @post = Post.find(params[:post_id])
        end

         def comments_params
            params.required(:comment).permit(:content)
        end
end
```

あとは、`rails s` でローカルサーバを建てて実際にコメントが作成&削除できていればOKです。

### Heroku へのデプロイ

[こちら](https://devcenter.heroku.com/articles/heroku-cli) を参考にしてHeroku CLI をインストールします。

次に、Herokuへデプロイするように`Gemfile`を修正します。

まず、`sqlite3` を本番環境で使用せず、開発環境で使用するように変更します。

```ruby:Gemfile
group :development do
  # Use sqlite3 as the database for Active Record
  gem 'sqlite3'
end
```

Herokuでは`Postgres`を本番環境で使用するため、以下のように本番環境用に`pg`を使用します

```ruby:Gemfile
group :production do
  gem 'pg'
end
```

変更後、`bundle install` を実行します

```shell
bundle install
```

`Heroku CLI`を使用してデプロイ先のアプリを作成します

```shell
heroku login
heroku apps:create -a blog-sample
```

デプロイするために、`heroku`というリモートリポジトリを追加します

```shell
heroku git:remote -a blog-sample
```

ここまでの変更を`commit`して`push`します。

```shell
git commit -am "deploy to Heroku"
git push heroku master
```

その後、`rails db:migrate`を実行します。

```shell
heroku run rails db:migrate -a blog-sample
```

最後に、`https://blog-sample.herokuapp.com`にアクセスしてブログが表示されていればOKです
