---
layout: post
title: 'TumblrからJekyll + Github Pagesへ'
date: '2016-12-12T18:30:00+09:00'
tags:
- tumblr
- migration
- jekyll
- github
---

[3年ほど前にWordPressからTumblrへ移行しました](/2013/08/24/wordpress-to-tumblr)。特に大きな問題があったわけではないのですが、最近ソースコードを挿入した記事をいくつか書こうとしたところTumblrのMarkdownではうまく表現され切れない点にすこしストレスを感じていました。表示をあれこれいじっている時間ももったいなくなってきたのでいっそのこと、[GFM (GitHub Flavored Markdown)](https://guides.github.com/features/mastering-markdown/#GitHub-flavored-markdown)で書けるようなサービスか、システムに移行しようと考え始めました。

あれこれ調べているうちに、どうやら[Jekyll](https://jekyllrb.com/)をつかって[Github Pages](https://pages.github.com/)へ移行するのがスムーズそうだなと調べ始めて評価し、次の点で問題なさそうだったので移行することにしました。

* GFMで書ける
* 既存のURL構成を引き継げる
* Tumblrからの記事取り込みもできる
* シンプルなテーマがある

逆に移行に当たってできなくなる機能もあります。たとえばソーシャルメディア連携です。Tumblrでホストしてもらえば、Tumblr上のソーシャルメディアでリブログなど拡散手段があります。ほかには予約投稿やTwitter連携といった機能も使えなくなります。このあたりはトレードオフですが、ソーシャルメディアで拡散していただくよりも、そもそも記事が書きづらくて筆無精になるのであれば優先順位は明らかです。

ソーシャルメディア連携もあとからIFTTTなど何か使えば実現できるでしょうし、予約投稿も必要になれば決まった時間に`git push`するよう何か構成を考えればいいだけです。

# 移行の効果

移行中のあれこれは後述することにしてまずは、移行によってどういったメリットがあったかみてみます。

WordPressから移行した際にはサイトの速度が劇的に改善しましたが、今回は比較的軽微な差に収まっています。

## TumblrでのPageSpeed結果

まずはTumblrでの[PageSpeed](https://developers.google.com/speed/pagespeed/)計測結果から。

![Tumblr / モバイル](/images/2016-12-12-tumblr-mobile.png)

モバイル用のスコアは67点。画像サイズが大きいと警告がでているのと、キャッシュ設定などのところで最適化の余地があるとの判定です。

![Tumblr / デスクトップ](/images/2016-12-12-tumblr-desktop.png)

デスクトップ用のスコアは77点。モバイルと同様に画像が大きいと警告が出ています。いずれにしても体感的にはもっさりした感覚は全くなく、表示速度などの不満は特にありませんでした。

## Jekyll + Github PagesでPageSpeed結果

![Jekyll + Github Pages / モバイル](/images/2016-12-12-jekyll-mobile.png)

モバイル用のスコアは72点と少しだけTumblrでの表示より改善しています。これはTumblrのインフラスピードとの差というよりは、採用しているテーマのCSSやJavaScriptの差がほとんどだと考えられます。

![Jekyll + Github Pages / デスクトップ](/images/2016-12-12-jekyll-desktop.png)

デスクトップ用のスコアは77点とTumblrでの結果と同じです。

## その他の違い

Tumblrではモバイル対応していないテーマを使っていたので、TumblrのUIが優先して表示されています。一方、Jekyllで今回利用した[Lanyon](http://lanyon.getpoole.com/)はモバイルフレンドリーなテーマで、今回どちらでも同じLook and Feelでサイトを表示できるようになりました。

# 移行

移行作業はJekyllの使い方を覚えるところから全部でおおよそ5〜6時間かかったかと思います。Jekyllはかなり活発に開発されているようなのであまりここで細々と手順を書いてもすぐ使いもにならなくなってしまう気がするのでおおまかな流れだけご紹介しておきます。

## Jekyll環境の設定

最近はこの手のツールを使うときはすべてDockerを使って独立した環境で実行するようにしています。再現性もありますし、手元の環境もシンプルに保てます。Jekyllを使うには既存のDockerイメージを使えば充分ですが、いくつかテーマを試したりしているうちに依存関係のあるライブラリをまとめてイメージとして持っていたほうが便利だったので次のような内容の`Dockerfile`を作っています。

```
FROM jekyll/jekyll:builder

RUN apk add --no-cache --virtual build-dependencies build-base
RUN apk add --no-cache libxml2-dev libxslt-dev
RUN apk add --no-cache ruby-dev curl-dev zlib-dev yaml-dev
RUN gem install nokogiri
RUN gem install minima
RUN gem install jekyll-import
```

`jekyll-import`は後述のTumblrから記事を取り込んだときに使ったものです。この`Dockerfile`をビルドしておきます。たとえば`site`ディレクトリ以下につくっていくならこんな感じにコンテナを実行して、テンプレートを作ったり`jekyll-import`を実行したりできます。

```sh
$ docker run --rm -i $(pwd)/site:/srv/jekyll -p 4000:4000 -t (ビルドしたときのタグ名) bash
```

今回はあれこれテーマをダウンロードし、`jekyll-import`でTumblrから記事をざっくり取り込んで表示確認をしながらレイアウトを決定しました。

## Tumblrからのインポート

[jekyll-importのTumblrについての記事](http://import.jekyllrb.com/docs/tumblr/)にコマンドが書かれていますのでこれを実行します。
移行といっても今回はTumblrで書いた記事が12記事しかなかったので、ある程度の部分は詳しく調べずに手作業でファイルを修正していくというスタイルで解決しました。

今回実行した際は次のように実行しました。

```sh
ruby -rubygems -e 'require "jekyll-import";
    JekyllImport::Importers::Tumblr.run({
      "url"            => "http://(TumblrのID).tumblr.com",
      "format"         => "html",
      "grab_images"    => true,
      "add_highlights" => true,
      "rewrite_urls"   => true
    })'
```

最初formatはMarkdown(md)を指定していましたが、うまく記事が取りこめなかったのでhtmlで取り込み、あとで手動でMarkdownに変換しました。

Tumblrでは記事のURLが `/post/(記事ID)/(記事タイトル)`というフォーマットでしたが、Jekyllでは`/(年)/(月)/(日)/(記事タイトル)`となるので変換が必要です。`rewrite_urls`で相当するアドレスにリダイレクトするファイルを作ってくれるので、リンクはそのまま維持できます。

ただ困ったことに生々されたリダイレクト用の`index.html`では`/(年)-(月)-(日)-(記事タイトル)`へのリダイレクトになっていてうまく動きません。あれこれ探すのも面倒だったのでここは手動で直してしまいました。

`grab_images`という設定があり画像をもってきてくれそうな印象がありましたが、これもうまく動かなかったので手作業で画像をもってきたり、手元に保存してあったものを使ったりして再構成しました。

## テーマ選び

いくつかサンプルを見ながら選びましたがシンプルなものがよかったのでLanyonにしました。ほかにもシンプルなデザインのものはありましたが、Lanyonは欲しい機能がおおむね入っていることと、CSSやJavaScriptを含めても全体の構成がシンプルで全体が見渡しやすいことが気に入りました。

# 切り替え

できあがったJekyllのページ一式をGithub Pagesへプッシュし、あとはCloudFlareでDNSの切り替えをすれば終了です。

Jekyllも開発スピードが速そうなので、開発についていくとしたらかなり大変そうです。ただでき上がったサイトはかなりシンプルなHTML、CSS、JavaScriptと画像ファイルだけで構成されているのでセキュリティー上の理由などで更新しなければいけないケースはかなり少ないと思います。

そういった意味では、一度環境ができ上がってしまえばTumblrでプレビューにあれこれ悩みながら記事を書くよりも手元の環境で確実なプレビューを見てから公開できるワークフローをつくったほうがより生産性が高そうだということで今は満足しています。

画像を含めたオーサリングのワークフローをつくったり、プレビューから予定投稿などまでいろいろ作り込みたいところですがそのあたりはまた今度じっくり作り込んでみることにします。


