+++
categories = ["IT", "book"]
date = "2016-03-01T23:55:34+09:00"
draft = true
slug = "rails-kiso"
tags = ["ruby", "rails"]
title = "改定3版　基礎Ruby on Railsをやってみる"
description = "Railsしっかり身につけたい"
+++

Rails Tutorial を2周ほどやって基礎的なことはだいたいわかってるつもりで復習としてこの本をやってみる。
文章は流し見で、写経中心。

# Chapter1
開発準備とRailsの説明。ここは殆ど飛ばしたが、Vagrantの環境構築もっと理解しないとな。

# Chapter2
Rubyの説明、継承とかクラスメソッドとかHashとか例外処理とかシンボルとかetc...
結構短い間にぎゅっと説明しているので、Rubyさっぱりで読む人は別に1冊Rubyの本見るか([たのしいRuby](amazon.co.jp/dp/4797386290)とか)、[ドットインストール](dotinstall.com)のRubyをやるかしたほうがいいかも。

# Chapter3
Railsの基礎的な部文。View, Controller, HTTP, ルーティング, css

flash.nowでそのアクション内で使用できる。  
アクションコールバックにはbefore_action, after_action以外にaround_actionもある。

「コントローラとテンプレート(view)はオブジェクトは別」など、以外と背景の色が違うトピック的な部分で大事なこと言ってたりする。
確かに、paramsやflashみたいにどっちでも使えるメソッドがあるからややこしかった。


# Chapter4
基礎にさらにModel, Databaseの説明  
時間の設定って`config/application.rb`の`class Application < Rails::Application`の中に`config.time_zone = "Tokyo"`ってやるんだ。

## Rakeとは
rubyで書かれたビルドツール。ファイルの依存関係を調べて自動的にコマンドを実行するソフトウェア。Railsでは、データベースの操作やテストの実行に使われる。それらはタスクと呼ばれ``rake routes``とか``rake db:migrate``、``rake test``などがある。  
``rake -T``でタスク一覧を見れる。これらタスクはGemパッケージの中に書かれているが、自分でタスクを作ることもできる。lib/tasksフォルダの下に~.rakeのファイルを作って``task: task_name do ~ end ``の中にタスクを書けばOK。
例:

```rake
desc "List all members"
task member_list: :environment do
  Member.all.each do |member|
    puts "#{member.number}\t#{member.name}"
  end
end
```



## 設定より規約
Railsでは命名規則が重要でModelの作成は
``rails g model 単数``とし、作成されるファイルはスネークケースで、クラス名はキャメルケースとなっている。Memberの場合、member.rbが作成されMemberというクラスができる。Membersとしてはいけない。

## マイグレーション
モデル作成時にdb/migrateフォルダ以下に201508082305_create_members.rbのようなファイルが作成される。これをマイグレーションスクリプトという。これによってカラムを定義したり変更したりできる。これにはsqlの知識が少し必要だが[SQLゼロから始めるデータベース操作](http://www.amazon.co.jp/CD%E4%BB%98-SQL-%E3%82%BC%E3%83%AD%E3%81%8B%E3%82%89%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E6%93%8D%E4%BD%9C-%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E5%AD%A6%E7%BF%92%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA-%E3%83%9F%E3%83%83%E3%82%AF/dp/4798118818/ref=sr_1_1?ie=UTF8&qid=1456564713&sr=8-1&keywords=sql%E3%82%BC%E3%83%AD%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B)は初心者におすすめ。またこの本自体でも説明してくれているので前提知識がなくてもちゃんと読めば問題ない。不安なら[ドットインストール](dotinstall.com)でsqliteかmysqlあたりをやればいいだろう。
マイグレーションの話はRails Tutorialよりも結構詳しく書いてある。マイグレーションは結構重要で、リリース後のマイグレーションは慎重にしないといけない。

## クエリーメソッドとリレーションオブジェクト
Member.whre("number < 20")などはクエリーメソッドと呼ばれていて検索条件を保持するだけで検索は実行されない。返した値は配列のように扱えるが実際は配列ではなくリレーションオブジェクトが返されている。これによってデータが必要なときにだけ検索を実行してくれ、余計な検索の実行を省くことができる(Lazy Loading)。即座に実行したい場合はloadメソッドを使用する。


# Chapter5
RESTに関して、実際にroutes.rbにリソースを書いてコントローラーでindex, show, new, edit, create, update, destroyを追加していく。

## collectionとmember
```ruby
resources :members do
    collection { get "search" }
end
```
resourcesにはブロックを渡せて`collection`と`member`というメソッドを使える。collectionは全てのデータを対象とし、memberは特定のデータを対象とする。上の例では、members(すべてのデータ)に対して検索を行いたいのでcollectionを利用している。

## present? blank?
blank?メソッドはオブジェクトがnil, false, 空文字列, 空白文字(改行、タブ含む), 空の配列, 空のハッシュの場合trueを返し、そうでない場合はfalseを返す。present?メソッドはこの逆。

## form_tag, form_for
検索フォームを作るときにform_tagを利用していてform_forとの使い分けに疑問を持ったので、
[【Rails】formヘルパーを徹底的に理解する](http://qiita.com/shunsuke227ono/items/7accec12eef6d89b0aa9)がわかりやすかった。  
> form_for: 任意のmodelに基づいたformを作るときに使う
> form_tag: modelに基づかないformを作るときに使う
