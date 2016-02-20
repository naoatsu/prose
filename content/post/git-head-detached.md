+++
categories = ["git"]
date = "2016-02-20T16:24:33+09:00"
slug = "git-detached-head"
tags = ["git"]
title = "gitでHEAD detached状態になった"

+++

Gitまだまだ基本的なことしか知らない人(自分)は何かあった時にすぐ検索すると思うんですが、どうもエラーが出た時がこわい。
この前もコミット後の変更を取り消そうと色々してたらHEAD detachedって注意が出てきて戸惑ってしまった。
[このサイト](http://devlights.hatenablog.com/entry/20130417/p1)を参考にしたらいけたけど、よくよく見たらちゃんとエラーメッセージが出ててそれ見たらmasterにcheckoutしてdetachedしてたコミットのブランチを作るように言ってくれてた。

まずは焦らずにgitさんはいい人だと理解してエラーを見ることが大事。
