+++
categories = ["it"]
date = "2016-03-02T11:14:04+09:00"
draft = false
slug = "atom-emmet-autocomplete-tab"
tags = ["atom"]
title = "atomでemmetとautocompleteでtabが被ってしまう"
description = "autocompleteをtabで利用している方向け"
+++

テキストエディターは[Atom](https://atom.io/)

autocompleteによって予測が出ている時に``enter``でなく``tab``で決定できるようにしている。

そのせいで、拡張子が.htmlとかの時に`tab`がemmetのトリガーにもなっているので`tab`を押すとemmetの方が発動してしまう。

対処法としてOpenConfigFolderからkeymap.csonをいじればいいが、わざわざコマンドを変えたくなかったのでautocompleteの予測変換が出ている時だけ`tab`がautocompleteのトリガーになるようにしたかった。

[Stack Over Flow ](https://github.com/atom/autocomplete-plus/issues/86)に同様のissueを発見

```cson
# Stop emmet from hijacking tab from snippets and autocomplete
'atom-text-editor.autocomplete-active:not([mini])':
  'tab': 'autocomplete-plus:confirm'

# Stop emmet from hijacking tab from snippet tab stops
'atom-text-editor[data-grammar="text html basic"]:not([mini]), atom-text-editor[data-grammar~="jade"]:not([mini]), atom-text-editor[data-grammar~="css"]:not([mini]), atom-text-editor[data-grammar~="sass"]:not([mini])':
  'tab': 'snippets:next-tab-stop'
```

これを追加するだけでできた。
下の'tab': 'snippets:next-tab-stop'
ってのは標準のatomのsnippetでemmetを上書きしている感じか
