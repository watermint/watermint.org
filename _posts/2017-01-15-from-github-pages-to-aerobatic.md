---
layout: post
title: 'Github Pages + CloudFlareからAerobaticへ'
tags:
 - github pages
 - aerobatic
 - jekyll
---

つい先月、Tumblrから[Jekyll + Github Pages](/2016/12/12/tumblr-to-jekyll/)に移行したところですが、今度はJekyll + [Aerobatic](https://www.aerobatic.com/)に移行しました。

これでこのブログのサーバお引っ越しは、次のように4回目になりました。

1. WordPress + さくらインターネット (2005〜2013)
2. Tumblr (2013〜2016)
3. Jekyll + Github Pages + CloudFlare (2016〜2017)
4. Jekyll + Aerobatic (2017〜)

WordPressからTumblrに移行したり、TumblrからJekyllに移行するのはコンテンツの調整などでさまざま面倒な点があります。たとえば、既存コンテンツに対するURLを引き継ぎたいといった要望に対して対応するのはコンテンツ個別に調整と動作確認が必要でした。

今回、コンテンツはそのままJekyllで運用するのでインフラ部分の移行のみです。

## Github Pagesから移行した理由

基本的な使い方としてはGithub Pagesで充分満足が行くものでした。ただ、気になる点としては下記2点がありました。

* HTTPSに対応できない
* Minifierが使えない

Github Pagesでカスタムドメインとして登録した場合、現状ではHTTPSでコンテンツを提供することができません。ブログコンテンツの内容として、HTTPS通信の必要性はさほどありません。ただ、Appleによる[App Transport Securityなどでは2016年末までという期限こそ延長された](https://developer.apple.com/news/?id=12212016b)もののデフォルトでHTTPSを提供するようにとした方針に変更はありません。こういった状況を考えれば、HTTPSでのコンテンツ提供準備は早くできているに越したことはありません。

次にMinifierです。[jekyll-minifier](https://github.com/digitalsparky/jekyll-minifier)を使おうとしましたがGithub Pagesではうまく動作しませんでした。

## Github Pages + CloudFlare

次にGithub Pagesだけではうまく2つの課題に対応できなかったので、CloudFlareを組み合わせることにしました。CloudFlareを経由して通信を行えば２つの課題は解消できます。もともと、watermint.orgドメインのDNS管理はCloudFlareで行っていたのでCloudFlareを経由するかどうかのオプションを変えるだけですみました。

```
[Client] ---<HTTP/HTTPS>--- [CloudFlare] ---<HTTP>--- [Github Pages]
```

CloudFlareを経由すると、HTTPSで接続要求があった場合CloudFlare側でHTTPSで処理してくれます。元のコンテンツであるGithub PagesとCloudFlare間は従来通りHTTPでの通信となります。CloudFlareから証明書も無償で発行してもらえます。

CloudFlareではCDNとしてコンテンツキャッシュをするだけでなく、Minifyのような処理も追加オプションとして用意されています。これも使い方は単純にオプションを有効化すればよいだけで、自動的にHTMLやCSSなどのコンテンツを最適化してくれます。

## Aerobaticへの移行

上記のように課題は解決されたので移行の必要性は高くありませんでしたが、機能面でGithub Pagesよりも多彩であることと、Aerobaticサーバ側から直接HTTPS通信をサポートすることができるのでこちらに移行することにしました。AerobaticでもCloudFlareと同様に証明書は無償で発行してもらえます。

```
[Client] ---<HTTP/HTTPS>--- [Aerobatic]
```

AerobaticはGithubとGithub Pagesの関係と同様に、BitbucketとAerobaticというようにソースコードレポジトリと対にしてコンテンツを管理します。Aerobaticでは複数のブランチに対してサービスの設定が可能で、ステージング用のブランチとステージング用サブドメインを設定するといった運用ができます。

コンテンツは公開順ごとにバージョンが振られていきロールバックなどもgit操作なしで簡単に行えます。

他には、Githubでプライベートレポジトリは有償ですが、Bitbucketであればプライベートレポジトリも無償で利用できます。Jekyllのサイトデータであれば、公開してあっても特に困りませんが、あまりレポジトリ内のファイルが散らかっていると恥ずかしいので、、といった理由でプライベートにできたほうがうれしいですね。

![Websiteバージョン](/images/2017-01-15-website-versions.png)

実はTumblrからJekyllに移行する際の本命がこちらのAerobaticだったのですが後述の問題があったので取り急ぎGithub Pagesに移行することにしていました。

## Aerobaticでドメイン設定ができない

AerobaticではCDNにAWSのCloudFrontを使っています。証明書はCloudFront側で発行されるものを使うのですが、この処理でエラーとなってしまい、ホスティングの設定ができませんでした。

検索してもなかなか同様の報告が見当たらず解決の糸口がなかったのですが、今回とりあえずAerobaticのサポート宛てに問い合わせをしてみたところ、翌日には解決となりました。

Aerobaticでカスタムドメインの設定をする際、まずCloudFront側からドメイン所有者であることを確認するためのメールを受け取り、承認処理を行います。この操作のあと、Aerobatic上でverify処理を進めれば次はDNSの設定。となるはずなのですがAerobatic上のverify処理で「CNAME already registered with CloudFront」というエラーがでて一向に進みません。

このことをサポートに問い合わせたところ、すぐにエラーを解消してもらえたのですが、おそらく、ドメインwatermint.orgでCloudFrontを使ったことはなかったので承認処理手順の際に何かの拍子で2回処理が実行されてしまったのかもしれません。

## jekyll-minifier + Aerobatic

jekyll-minifierとAerobaticの組み合わせは先月試したときにはうまく動作したのですが、移行したタイミングでは動作しなくなってしまっていました。

jekyll-minifierのリリース履歴を見ると2週間前に課題を解決するために、[0.1.0](https://github.com/digitalsparky/jekyll-minifier/releases/tag/v0.1.0)がリリースされておりこの影響によるものでした。jekyll-minifierの参照については `_config.yml` に下記のように依存を書いておいただけだったのでバージョン指定はしていませんでした。

```yaml
gems:
 - jekyll-paginate
 - jekyll-minifier
```

このように依存するライブラリのバージョンアップなどで急に動かなくなってしまっても不都合なので、`Gemfile`と`Gemfile.lock`でバージョンを固定しておくことにしました。Aerobaticの[Automated Builds](https://www.aerobatic.com/docs/automated-builds)で説明がある通り、(1) `_plugins`にある`*.rb`ファイル、(2) `Gemfile`あるいは`Gemfile.lock`があれば`bundler`によるインストール、(3) `_config.yml`の`gems`配列。といった順でライブラリが参照されます。

## Aerobaticでの運用

Aerobaticでは無償プランにて2つのドメインに対してHTTPSを含むコンテンツをホスティングすることができます。一方で無償版の制約としては1日のデプロイ回数が5回までに制限されていることです。

一日に何度も記事を公開するようなブログであれば有償プランを選択する必要があります。また、上述のminifierのような処理を入れようとするタイミングではいろいろ試行錯誤するので5回という制約はかなり作業に影響します。Aerobaticは際立って高価なサービスではないので有償版にアップグレードしてもよいのですが、複数のドメインでサービスを提供したり、大量のコンテンツを持っているということもないので現状は見送って1日5回という制約の中で運用することにしています。
