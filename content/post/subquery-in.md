+++
categories = ["IT", "book"]
date = "2016-02-27T18:46:44+09:00"
draft = false
slug = "sql-subquery"
tags = ["sql"]
title = "SQLのサブクエリ"

+++

前回読んだ[SQLゼロから始めるデータベース操作](http://www.amazon.co.jp/CD%E4%BB%98-SQL-%E3%82%BC%E3%83%AD%E3%81%8B%E3%82%89%E3%81%AF%E3%81%98%E3%82%81%E3%82%8B%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9%E6%93%8D%E4%BD%9C-%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E5%AD%A6%E7%BF%92%E3%82%B7%E3%83%AA%E3%83%BC%E3%82%BA-%E3%83%9F%E3%83%83%E3%82%AF/dp/4798118818/ref=sr_1_1?ie=UTF8&qid=1456564713&sr=8-1&keywords=sql%E3%82%BC%E3%83%AD%E3%81%8B%E3%82%89%E5%A7%8B%E3%82%81%E3%82%8B)の内容でサブクエリとINのあたりがややこしかったのでまとめてみます。

### サブクエリとは
sub(下位の)query(問い合わせ)
つまり上の問い合わせとしたの問い合わせという階層ができるんですね。

```sql
SELECT shohin_bunrui, cnt_shohin
  FROM (SELECT shohin_bunrui, COUNT(*) AS cnt_shohin
    FROM Shohin GROUP BY shohin_bunrui) AS ShohinSum;
```

FROMの中にまたSELECT文が入っていて内側と外側に分かれています。
実行は内側からされるので、Shohinテーブルからshohin_bunruiとCOUNT(*)をshohin_bunruiごとにとってきたものにさらに外側のSELECT文を実行しています。

と、これが普通のサブクエリですが、サブクエリには

- **スカラ・サブクエリ**
- **相関・サブクエリ**

というものがあります。

#### スカラ・サブクエリ
スカラ・サブクエリは簡単でスカラ(単一)な値を返すサブクエリです。
例えば単価の平均をAVG()で求めるとすると結果は1つの値が返ってくるので、それをWHERE句などで利用します。
本に載ってた例を挙げると

```sql
SELECT shohin_id, shohin_mei, hanbai_tanka
  FROM Shohin
WHERE hanbai_tanka > (SELECT AVG(hanbai_tanka) FROM Shohin);
-- AVG()内の結果は絶対に複数行を返さないようにする
```

とかができます。
この場合は販売単価が全商品の平均販売単価よりも高いものだけ選択しています。
別にスカラ・サブクエリはWHERE句以外にも使えます。

#### 相関サブクエリ
もう一つの相関サブクエリは小分けにしたグループ内での比較に使います。
さっきの全商品の平均より高い商品の選択に条件を加えて「商品分類ごとに平均販売単価より高い商品」を選択する場合などに有効です。
この場合スカラ・サブクエリでは

```sql
SELECT shohin_id, shohin_mei, hanbai_tanka
  FROM Shohin
WHERE hanbai_tanka > (SELECT AVG(hanbai_tanka) FROM Shohin GROUP BY shohin_bunrui);
-- AVG()内の結果が複数行
```

これではエラーになってしまいます。

この問題は相関サブクエリを使うことで解消できます。

```sql
SELECT shohin_bunrui, shohin_mei, hanbai_tanka
  FROM Shohin AS S1
WHERE hanbai_tanka > (SELECT AVG(hanbai_tanak)
                      FROM Shohin AS S2
                      WHERE S1.shohin_bunrui = S2.shohin_bunrui
                      -- このWHEREの条件が重要
                      GROUP BY shohin_bunrui);
```

WHERE句でテーブル.列名 としてshohin_bunruiで縛ることによって実質的に1行しか返していないと見なされるようにしています。
ざっくり言うとスカラ・サブクエリをgroupごとに行っている感じですかね。


### 締め
本では図を使ってもっとわかりやすく説明してくれているのでこの駄文を読んで興味のある方は読んでみてもいいかもしれないです。

正直、ブログに書いてても、
相関クエリはやっぱりちょっとしっくりきていないですね。

まあ使ううちに慣れていく部分もあると思うので今日はこれくらいで。
