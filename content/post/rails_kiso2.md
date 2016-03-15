+++
categories = ["it", "book"]
date = "2016-03-10T20:30:56+09:00"
description = "Railsしっかり使いたい"
draft = false
slug = "rails-kiso-part2"
tags = ["ruby", "rails"]
title = "改定3版　基礎Ruby on Railsをやってみる後半"
+++

[基礎Ruby on Rails](http://www.amazon.co.jp/dp/4844338153)をやってみる後半。
[前回]({{< relref "rails_kiso.md" >}})

# Chapter7
主にテストを扱っている。

## FactoryGirls
テスト用データを簡単に用意できる。

## モデルのスコープ
レコードの検索をシンプルに書き表すための機能。
`scope : スコープ名, -> { クエリーメソッド }`

```ruby
class Article
  scope :member_only, -> { where(member_only: true) }
  # 省略
end
```
->{ }はProcオブジェクトを作成する記法。Procとは「コードのかたまり」を表すオブジェクト。(無名関数という)

```ruby
p = -> (n) { Math.sqrt rand(n) }
puts p.call
```
Procを呼び出すには上記のようにcallメソッドを使う。また、(n)は引数
`-> {}`の代わりにlambdaメソッドを使うこともできる。
```ruby
p = lambda { |n| Math.sqrt rand(n) }
puts p.call(100)
```

スコープの話に戻って、
```ruby
class Article
  scope :member_only, -> { where(member_only: true) }
  #~~~
end
```
のmember_onlyはArticleクラスのクラスメソッドになり、リレーションオブジェクトを返すようになる。
`Article.member_only.order(released_at: :desc)`は`Article.where(member_only: true).oreder(released_at: :desc)`と同じ結果になる。

## テスト
GemパッケージのMinitestを利用。
下のAssertionメソッドはMinitest::Assertionsモジュールに用意されている

| Assertionメソッド | 用途     |
| :------------- | :------------- |
|assert a, "aが存在しない"|引数aがtrueなら成功。どのテストも最後に文字列でメッセージを指定できる|
|assert_equal 式1, 式2|式1 == 式2なら成功|
|assert_nil 式|式がnilなら成功|
|assert_empty 式|式が空の文字列、配列、ハッシュなら成功|
|assert_kind_of クラス, 式|式がそのクラスのインスタンスなら成功|
|assert_match パターン, 文字列|文字列が正規表現のパターンにマッチすれば成功|
|assert_includes 配列, 変数|変数が配列に含まれれば成功|
|refute|assetの代わりに使うとfalseの時に成功する(assertの反対)|
|assert_raises(例外) { メソッド }|メソッドが指定の例外を発生させたら成功|


## 統合テスト
例えばログインしてデータを書き換えてログアウトする。など流れのテストの場合、複数のコントローラが関係するため統合テストを用いる
`rails g integration_test integration_file_name`で作成できる。
<!--
| 統合テストで使うメソッドの一例 | 用途     |
| :------------- | :------------- |
| follow_redirect! | リダイレクト先のページを読み込ませる       |
||| -->


# Chapter8
実践的なアプリケーション

## アセットパイプライン
SassやCoffeeScriptをコンパイルしてくれたり、圧縮してくれたりする機能

CSSなら
```css
/*
 *= require_tree .
 *= require_self
 */
 ```

JSなら
```js
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require_tree .
```

とコメントしておく。
require_treeがあるとそのフォルダにあるファイルをすべてこのファイルにまとめてくれる。
またcssのrequire_selfはこのcss自体を最終的なスタイルシートに含めることを指定している。  
require jqueryなどはそれぞれ機能をgemから取り入れてる。(ざっくり)


#### Turbolinks
Rails4.0以降に追加された機能
ページ内のリンクをクリックした時に同じサイトの場合にCSSとJavascriptを書き換えずにそのままにしておく機能。その分高速化に繋がる。URLを直接読み込む場合やページ再読み込みの場合は通常のページ読み込みと変わらない。
注意点として、JavaScriptではradyイベントだけでなくpage:loadイベントにも対応する必要がある。

## ログイン機能
ログイン機能を実装するためにはブラウザのcookieにセッションデータを保存してユーザーを識別する必要がある。  

#### セッション
複数のページにわたってブラウザーとサーバーの間で維持される接続状態のこと。

セッションにデータを保存するためにはsession[:データ名]に値を入れてあげれば良い

```ruby
member = Member.authenticate(name, password)
session[:member_id] = member.id
```
反対に取り出すには

```ruby
id = session[:member_id]
member = Member.find(id)
```
また削除するには

```ruby
session.delete(:member_id)
```
とすれば良い。  
Railsはセッションデータを符号化しているが暗号化はしていないので解読されるため注意が必要。ただしデータが改ざんされた場合はエラーになる仕組み。セッションデータにはサイズが上限がある。

#### cookie
cookies[:データ名]に保存することでcookieにもデータを保存することができる。
もし会員のidをクッキーに保存したい時は、cookiesの代わりにcookies.signedを使うことによって符号化できる。

```ruby
cookies.signed[:member_id] = member.id
```

#### password
bcrypt-rubyというgemを使う

```ruby
def password=(val)
  if val.present?
    self.hashed_password = BCrypt::Password.create(val)
  end
  @password = val
end
```
とすることによってbcryptによってつくられたパスワードがhashed_password属性に入る。bcrypt-rubyのようにパスワードは非可逆の暗号を用いないといけない。

精製したパスワードを確認したい時は
`BCrypt::Password.new(member.hashed_password) == password`とする。

ログインのためにauthenticateメソッドを作る
```ruby
def authenticate(name, password)
      member = find_by(name: name)
      if member && member.hashed_password.present? && BCrypt::Password.new(member.hashed_password) == password
        member
      else
        nil
      end
    end
```

#### 遅延初期化
現在のユーザを返すcurrent_memberを作る機会は多いと思うが何度もデータベースにアクセスさせると無駄が多いので
```ruby
def current_member
  if session[:member_id]
    @current_member ||= Member.find_by(id: session[:member_id])
  end
end
```
のように行う。これを遅延初期化と呼ぶ。なお`||=`は`+=`や`-=`と同じように

```ruby
@current_member = @current_member || Member.find_by(id: session[:member_id])
```
の省略版。実際にはRailsが勝手にSQLの検索結果をキャッシュに保存してくれるためアクセス回数が減るわけではないらしい。

#### setup
テストで`setup`メソッドを作成すると各テストを実行する直前にこのメソッドが実行される。

## ストロング・パラメータ
Rails4.0で導入された新たなセキュリティ強化策。
フォームからスクリプトなどを使って予期しないパラメータを送ってこられる(マスアサイメント脆弱性)ことから防ぐ。

使い方はActionController::Parametersオブジェクトを返すプライベートメソッドをコントローラに加える
```ruby
private
  def member_params
    params.require(:member).permit(:number, :name)
  end
```
これによって`permit`によって許可されたパラメータ以外がフォームから送られると例外(ActiveModel::ForbiddenAttributesError)が発生する。
`require`メソッドは`:member`キーがないときに例外NoMethodErrorが発生してしまうのを防ぎ、ActionController::ParameterMissingで発生させる。

## エラーページ
エラー表示をカスタマイズできる
rescue_fromメソッドをApplicationControllerに記述する。(詳細は省略)
注意点として例外クラスは親を先に指定しないと上書きされる。
## ページネーション
Gemパッケージのwill_paginateを使用する。
`gem 'will_paginate'`
will_paginateを導入すると、モデルのクエリーメソッドにpaginateメソッドが追加される。

```ruby
@members = Member.paginate(page: 2, per_page: 15)
```
この場合現在のページは2ページで、1ページあたり15件取り出す。
テンプレートで

```erb
<%= will_paginate @members %>
```
とすればページネーションのリンクが使える。
またリンクにはpageパラメータがつくのでコントローラで`page: params[:page]`とページ数を指定できる。
先ほどの例を変えると

```ruby
@members = Member.paginate(page: params[:page], per_page: 15)
```
となる。

他にもよく使われるGemパッケジとして`kaminari`がある。(カスタマイズ性はこちらの方が良さそう)

# Chapter9
モデル間の関連付け

### 外部キー
別のテーブルの主キーを参照するカラムのこと
マイグレーションスクリプトで`t.references`で指定すると作れる。(`add_index`すべし)
外部キーのカラム名はcar_idのように「参照先のテーブル名(モデル名)を単数形にしたもの」＋「_id」とする。  
例えば車テーブルのidに対する車輪テーブルのcar_idカラム
この時Carクラスに`has_many :wheels`をWheelクラスに`belongs_to :car`を置いた場合。`@car.wheels`で`Wheel.where(car_id: @car.id)`と同様の機能を持つようにRailsが用意してくれる。

外部キーのカラム名がルールと異なる時は、`foreign_key`オプションでカラム名を指定できる。

### 関連付け
上の例は1対多の関連付けである。
1対1の場合は`has_one`と`belongs_to`を
多対多の場合は双方に`has_many`を作りつつ、`has_many through`も使用する。このためには両方を`belongs_to`する中間テーブルが必要。

`has_many`などで指定するメソッド名を変えたい場合は、`class_name`オプションを使う。

```ruby
class Car < ActiveRecord::Base
  has_many :engines, class_name: "Motor" # メソッド名をmotorsでなくenginesにしたい場合
end
```

#### dependent
参照先のレコードを削除した時に参照元のレコードも自動的に削除したい時に`dependent`オプションを`:destroy`とする。

```ruby
class Car < ActiveRecord::Base
  has_many :engines, dependent: :destroy
end
```


### ネストされたリソース

```ruby
resources :members do
  resources :entries
end
```
とルーティングすると下表のようにルーティングされる。

| アクション | パス     | HTTPメソッド |
| :------------- | :------------- | :-------------- |
| index       | members/123/entries      | GET |
| shown       | members/123/entries/456      | GET |
| new       | members/123/entries/new      | GET |
| edit       | members/123/entries/456/edit      | GET |
| create       | members/123/entries      | POST |
| update       | members/123/entries/456      | PATCH |
| destroy       | members/123/entries/456      | DELETE |
※123はmember_id(params[:member_id]), 456はentriesのid(params[:id])


### ネストされたフォーム
フォームの中に関連付けたモデルの登録も行いたい時。
例えばメンバー情報の登録フォームで関連づいてる画像データの登録も行いたい時は、

```Member.rb
has_one :image, class_name: "MemberImage", dependent: :destroy
accepts_nested_attributes_for :image, allow_destroy: true

```
とMemberモデルに記述することによって

```ruby
<%= form_for @member do |f| %>
  <%= f.fields_for :image do |imgf| %>
    <%= imgf.file_field :uploaded_image %>
  <% end %>
<% end %>
```
のように記述できる。`accepts_nested_attributes_for`や`fields_for`の第一引数にはhas_oneやhas_manyで指定した名前(上記では:image)を渡す。


## 画像アップロード
ActionDispatch::Http::UploadedFileクラスのオブジェクトからデータを取り出す。

## 管理者ページ

```ruby
namespace :admin do
  root to: "top#index"
  resources :members do
    collection { get "search" }
  end
  resources :articles
end
```
で名前空間付きのリソースが出来る。
コントローラでもフォルダ分けして(この場合ではadminフォルダを作る)名前空間付きのコントローラを作成する。


# まとめ
後半になるにつれて何度が上がっていき、時にChapter9はなかなかしんどかった。
Chapter9に関してはもう一度復習した方が良さそうだ。
また、テストに関しては簡易的に抑えていたので、スピーディだったが、実際はもう少し書いた方が良さそうな印象。
なんにせよ終わってひと段落。このまとめもざっと描いた感じだからおいおい修正していこうと思う。
