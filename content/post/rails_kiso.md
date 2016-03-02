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

# 1章
開発準備とRailsの説明。ここは殆ど飛ばしたが、Vagrantの環境構築もっと理解しないとな。

# 2章
Rubyの説明、継承とかクラスメソッドとかHashとか例外処理とかシンボルとかetc...
結構短い間にぎゅっと説明しているので、Rubyさっぱりで読む人は別に1冊Rubyの本見るか(たのしいRubyとか)、ドットインストールのRubyをやるかしたほうがいいかも。

# 3章
Railsの基礎的な部文。
flash.nowでそのアクション内で使用できる。  
アクションコールバックにはbefore_action, after_action以外にaround_actionもある。

「コントローラとテンプレート(view)はオブジェクトは別」など、以外と背景の色が違うトピック的な部分で大事なこと言ってたりする。
確かに、paramsやflashみたいにどっちでも使えるメソッドがあるからややこしかった。
