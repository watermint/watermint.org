---
layout: post
title: 'Mathematicaの練習: Asanaタスクの集計'
tags:
 - mathematica
 - asana
---

大学で使ってからもう十数年ぶりになりますがMathematicaをさわっています。数学とあとは機械学習関連を勉強するためMathematicaのHome版を購入してみました。まずはデータを分析したりするところから手始めにやってみようと思っていますが、	十数年ぶりということもありますし、大学時代もさほど深く使い込んでいなかったのでWolfram Languageの知識はほぼゼロの状態です。

慣れたプログラミング言語で書けばすぐできることですが、手始めにAsanaで管理しているタスクを集計したり分析する流れをやってみました。AsanaのタスクはJSON形式でエクスポートができますのでこれを読み込んで集計してみます。

![週ごとのタスク数](/images/2017-03-31-mathematica.jpg)

JSONを読み込むにはImportを使いますがこのとき、"JSON"ではなく、"RawJSON"をつかうとすべてがAssociation(連想配列のようなもの)として読み込むことができます。ここまでくるだけで結構つまづきました。Asanaから得られるJSONは"data"というキーが最初にあるのでいろいろなプログラミング言語の連想配列と同じように`["data"]`のように書いて値を取得します。値が取り出せたので続いて集計です。

AsanaのJSON書式は[API reference](https://asana.com/developers/api-reference/tasks)あたりを参照しながら処理を進めましょう。たとえばタスクが作成された日時は、"created_at"というキーに対する値として設定されています。時刻の書式はISO 8601形式です。

ISO8601形式をMathematicaで読み込むには `DateObject["2017-03-31T09:00:00Z"]`のようにDateObjectへ渡してやればよいようです。

手元のAsanaデータは1月中旬に整理整頓してそれ以前はあまり正確ではありませんでしたから、1月中旬以降のものだけを集計対象としています。

```mathematica
asanaTasks = 
  Select[asanaJSON, 
   DateObject[#["created_at"]] > DateObject[{2017, 1, 15}] &];
```

集計対象としたいタスクだけを取り出すには`Select`関数を使えば良いようです。関数型言語でプログラミングした経験があれば比較的すんなり理解しやすいかと思いますが、`&`のような簡略書式はMathematica独特なのでこれはマニュアルや例をみながら見様見まねで覚えるしかなさそうですね。

集計をするときも`CountsBy`のような関数で簡単に集計できます。関数がたくさんあるので、プロトタイピングするにはもってこいです。

週ごとに集計するために日付を丸めたかったのですがこれがなかなかわからず苦労しました。

```mathematica
taskCountPerWeek = CountsBy[
   asanaTasks,
   CurrentDate[
     DateObject[#["created_at"]],
     "Week"
     ] &
   ];
```

最近出た11.1というリリースで追加された`CurrentDate`を使うと簡単にできるようです。`CurrentDate[Now, "Week"]`のように日付の粒度を指定すればその粒度で値がかえってきます。語感としてややこしいのは、CurrentDateとありますがDateObjectなどを渡してやれば与えた日時を基準に変換してくれます。

最後に`DateListPlot`でグラフを書けば週ごとにどれぐらいタスクが作成されているかわかりました。
