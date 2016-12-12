---
layout: post
title: Dropbox Business utilities written in Golang
date: '2016-10-27T08:30:29+00:00'
tags:
- golang
- go
- dropbox
- dropboxbusiness
- api
tumblr_url: http://watermint.org/post/152355740498/dropbox-business-utilities-written-in-golang-ja
---

今夏ごろからDropbox Business向けのツールをちょこちょこと開発しています。[Dropbox Business API](https://www.dropbox.com/developers)は一般的なOAuth2を使った認証と、HTTPでのAPIのためたいていのプログラミング言語で簡単に扱えます。また、DropboxからPython、Ruby、Java、JavaScript、.NET、Objective-C、SwiftなどSDKもリリースされているのでこれらを使えばより簡単です。

ちなみにこれほど多くのSDKをメンテナンスするのはかなり手間ですが、それを解消するために[Stone](https://github.com/dropbox/stone)というオープンソースが公開されています。StoneはAPI仕様記述からSDKを生成するツールで一度この仕様記述をメンテナンスしていけばSDKは自動生成できます。Dropbox APIの仕様についても同様に[Stoneでの仕様記述](https://github.com/dropbox/dropbox-api-spec)があり、公開されているSDKは自動生成されたものをベースにしています。

# つくったツール

作成したのは下記二つのツールです。

* [dcfg](http://github.com/watermint/dcfg) … G Suiteのグループで管理されている組織、グループ構成をDropbox Businessチーム用に同期する
* [dreport](https://github.com/watermint/dreport) … Dropbox Businessチームに関するメンバー一覧などをレポートとしてCSVで出力する

いずれもCLIツールでシステム管理者が利用する想定のものです。ただ、昨今はシステム管理といってもWindowsだけでなくMacなど様々なシステム環境があります。このように利用用途からかんがえて複数プラットホームで利用できたほうが便利なので、個人的には初めてとなるGolangを採用してみました。

# はじめてのGolang

一番最近よく使っていたプログラミング言語はScalaだったので、カルチャーギャップというか、プログラミング言語がもつ信念のようなところに大きな差があって少し戸惑いました。

Scalaのようなオブジェクト指向的にも関数型的にもかけるプログラミング言語の直後ということもあって、Golangの書き方はややまどろっこしく感じます。ただこのまどろっこしさも次第に慣れていくことで、また考え方がわかってきたことでさほど気にならなくなってきました。

どこに書いてあったかソースは失念しましたがたとえば、「関数のなかで、計算量のオーダーが O(1) → O(n)のように変わるような書き方はあえてさせないことで、関数を使う側がどういうオーダーの計算量を常に意識させるようにしている」というような説明がありました。Golangのテストフレームワークがassertをあえて実装していない、といったこともそうですが、プログラミング言語やフレームワークで様々なヒューマンエラーを減らしていこうとする考え方とはまた違う思想を感じます。

この考え方を真に受けてコードを書いていったので、でき上がったプログラムはにたようなforループやif分岐がたくさんでき上がりました。今回、コードをかくに当たってあまり他の方のコードを読んだりしなかったので本流とはだいぶ外れているかもしれません。

# パッケージ管理

Golangで戸惑ったのがパッケージ管理です。Goのプログラムは`GOPATH`環境変数で指定したディレクトリ以下にすべての依存ファイル、そして自分のプロジェクトファイルも置いていくという方式です。

自分のプロジェクトをたとえば、github.com/watermint/dcfg で公開する予定ならば下図のように `GOPATH`以下に置いていきます。外部ライブラリたとえば、github.com/dropbox/dropbox-sdk-go-unofficialを使うのであればこれらも`go get`コマンドを経由して同様に`GOPATH`以下に配置します。

`$GOPATH / src / github.com / watermint / dcfg`

ひょっとすると、Goの考え方でAPIは一度公開したならば同じ名前で同じように使えるべき。という信念があるのかもしれません。
この方式で困るのは複数プロジェクトを進行しているときに、異なるバージョンのライブラリを利用したいや、ライブラリのバージョンをしばらく固定しておきたいときにはやっかいです。

また別の環境でビルドする際にも依存するライブラリの管理が煩雑になります。こういった問題に対しRubyであればgem、Javaなどではmaven、gradle、sbtなどパッケージ管理は様々あり、これらで管理するのが当たり前という風なイメージを持っています。Golangでも同じようなものを探してみましたが、いまひとつこれといったものがどれなのかわかりませんでした。

これも今回はあまり細かく調べずに、[glide](https://github.com/Masterminds/glide)というツールを利用しました。glideをつかうと、依存するライブラリ郡を自分のプロジェクトフォルダ以下の`vendor`というフォルダにまとめてくれます。バージョン指定も可能ですし、一括してライブラリ郡をダウンロードするなど細々面倒を見てくれます。

また今回つかったdropbox-go-sdk-unofficialは名前の通りアンオフィシャルSDKで、まだ活発に大規模なリファクタリングがおこなわれていてバージョンを固定しないとまともに開発できない。。という現状もありglideによるパッケージ管理はもっとはやく導入しておけばと悔やまれました。

おそらく他のツールでも同様によしなに準備してくれるのだと思います。

# ビルド

本格的にコーディングを始める前に、ビルド環境だけは念入りに準備することにしました。ビルド環境のOSのバージョンが違うことで何か問題がでたりするとかなりの時間を浪費してしまいますし、いろんなところで新しくビルドしようとおもったときにも手間がかかります。

今回はご多分に漏れず[Docker](https://www.docker.com/)を使っています。Dockerfileを用意しておいて、Dockerコンテナ上でビルドをしてそのバイナリを手元環境でテストするというワークフローを確立しました。

ついでにTravis CIでもテストが自動的に実行されるようにしておきました。テストカバレッジなども集計するという流れを自動化しておくこともできますが、これもなかなか情報が分散していて地味に手間のかかる作業でした。

これから環境を用意されようと思われる方は下記の.travis.ymlをご参考にしていただければと思います。

{% raw %}
```yaml
language: go
go:
 - 1.7
 - tip

before_install:
 - go get golang.org/x/tools/cmd/cover
 - go get github.com/modocache/gover
 - go get github.com/mattn/goveralls
 - go get github.com/Masterminds/glide

install:
 - glide install

script:
 - go list -f '{{if len .TestGoFiles}}"go test -coverprofile={{.Dir}}/.coverprofile {{.ImportPath}}"{{end}}' $(glide novendor) | xargs -L 1 sh -c
 - gover

after_success:
 - goveralls -coverprofile=gover.coverprofile -service=travis-ci -repotoken $COVERALLS_TOKEN
```
{% endraw %}

* [https://github.com/watermint/dcfg/blob/v1.3.9/.travis.yml](https://github.com/watermint/dcfg/blob/v1.3.9/.travis.yml)

ここで登場するcover、gover、goverallsはカバレッジを取得、集約するためのものです。よく見るとgoverは前職の同僚が作ったものですね。こんなところでお目にかかるとは誇らしいものです。

ところで`script:`のセクションはややトリッキーな書き方をしています。これもあれこれ情報を集めながら試行錯誤したものでこれでいいのか微妙なところですがひとまず動作しているのでこれで。。

{% raw %}
```yaml
- go list -f '{{if len .TestGoFiles}}"go test -coverprofile={{.Dir}}/.coverprofile {{.ImportPath}}"{{end}}' $(glide novendor) | xargs -L 1 sh -c
```
{% endraw %}

`go test`というのでカバレッジがとれますが残念ながら`go test`では1つのソースファイルについてのみテストとカバレッジ測定となってプロジェクト全体ではとれません。そこで、ソースコードひとつずつについて`go test`を実行してカバレッジデータのファイルを個別につくってあとで結合するという力技です。

# Retrospective

今回はCLIツールを作りましたが、利用シーンを考えるとGUIツールもぜひ作ってみたいところです。
GolangでのGUIライブラリはまだこれといった決定打がないようで、ひょっとしたらJavaでつくるか、SwiftをつかってiOSアプリにしてしまうほうがいいのかもしれません。

今回CLIとは別に[Gin](https://github.com/gin-gonic/gin)というライブラリを使ってWebサーバを立ち上げるパターンも試作してみました。Play framework等と比べればまだまだツールサポートなども弱いので生産性に課題を感じましたがしっかり作り込めばGUIでつくるよりも将来性がありそうです。

このあたりはまた次回の研究課題です。
