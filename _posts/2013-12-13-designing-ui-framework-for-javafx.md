---
layout: post
title: JavaFXアプリケーションの手触り感をととのえる、Fextile
date: '2013-12-13T08:02:14+00:00'
tags:
- javafx
- java
- scala
- scalafx
- fextile
- user interface
- user experience
- ui
- ux
- design
tumblr_url: http://watermint.org/post/69825099011/designing-ui-framework-for-javafx
---

この記事は[JavaFX Advent Calendar 2013](http://www.adventar.org/calendars/146)の13日目、昨日は[orekyuu](http://www1221uj.sakura.ne.jp/blog/)さんの[JavaFXでTwitterクライアント作ってみた。](http://www1221uj.sakura.ne.jp/blog/2013/12/javafx%E3%81%A7twitter%E3%82%AF%E3%83%A9%E3%82%A4%E3%82%A2%E3%83%B3%E3%83%88%E4%BD%9C%E3%81%A3%E3%81%A6%E3%81%BF%E3%81%9F%E3%80%82/)でした。

## 手触り感

iOSや、Android、Windows、OS Xのアプリケーションは、アプリケーションの製作者が違っていてもだいたい同じような手触り感があります。これはラベルの配置や色、スペースの取り方、フォントのような見栄えはもちろんのこと、ボタンをおしたときのフィードバック、画面遷移の時のエフェクトなどさまざまな振る舞いがOSやUIフレームワークによってルールが規定されているためです。規定はたとえば次のようにガイドラインとして公開されています。

* [iOS Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/mobilehig/)
* [Design Android Developers](http://developer.android.com/design/index.html)
* [Design library for Windows Phone](http://msdn.microsoft.com/en-us/library/windowsphone/design/hh202915(v=vs.105).aspx)


アプリケーション開発者はこれを参照し、アプリケーションに必要なUI/UXを作り上げていきます。iOSやWindows Phoneアプリケーションのように、アプリケーション公開までに審査があるようなケースではガイドラインに沿っていないアプリケーションは排除されるなどOSやデバイス全体のエコシステムとして手触り感の統一をしています。

## クロスプラットホームの場合<

ではWebアプリケーションの場合はどうでしょうか。WebアプリケーションはデバイスやOS、Webブラウザの種別やプラグインなどによって左右されることもありますが、モダンなデバイス／OSからであればたいていの環境で同じように動作します。この場合、iOSならではの手触り感、Androidならではの手触り感といったOSごとの手触り感とは違う別の手触り感が表現されることになります。Webアプリケーションでもいくつかルールやマナーはありますが、審査等はありませんので厳格かつ広く守られているガイドラインのようなモノは存在しません。

自由にデザインできるという反面、自由度が高すぎて制作コストが高くなったり、制作期間が長引いてしまうというデメリットもあります。このような課題に対して、近年では[Twitter Bootstrap](http://getbootstrap.com)のようにWebのデザインルールを決めたCSSやJavaScriptライブラリを使い、アプリケーションのテイストに合わせてテーマなどをカスタマイズする手法がとられることが多くなりました。

![Bootstrap UI components](/images/2013-12-13-fextile-bootstrapcomponents.png)

ヘッダやボタン、ラベルがある程度のカテゴリにわけられているため、Webアプリケーション開発者・デザイナーはこれらのカテゴリを利用してデザインを決定します。カテゴリと色やサイズなどはCSSとして分離されているため、プロトタイプ開発時にはデフォルトのデザインで作成しておき、リリース時には適切なブランドカラーを配置するなどデザイン作業に余裕が生まれます。

## JavaFX

クロスプラットホームのUIを作る手段としてJavaはすでによく用いられています。一昔前はJavaアプレットがあまりにも重いという印象で敬遠されることもありましたが、近年では高速化も進み、10年ほど前のユーザエクスペリエンスが低い状態ではなくなりました。ただJavaのUIフレームワークはAWTとSwingという10年ほど前の状態からあまり進歩していませんでした。その間にはMicrosoftのSilverlightやAdobe AirなどデスクトップアプリケーションのためのUIライブラリが続々と登場し差は広がる一方でした。

2008年にはJavaFX Scriptとして、新しく既存のAWT・Swingに変わるUIフレームワークが発表されました。当初、JavaFX ScriptとしてJava言語とは別の新しいプログラミング言語を使ってUIを構成するようにプロジェクトは進んでいましたが、紆余曲折の末、AWTやSwingと同様にJava言語から利用できるUIフレームワークと位置づけられてJavaFX 2がリリースされました。JavaFX 2ではWebKitエンジンを組み込んだモダンなWebView等を始めとして強力なコンポーネントが追加されました。クロスプラットホームで利用可能なUIフレームワークとして新たな選択肢が登場したといえます。

## JavaFXと手触り感

JavaFXには、ボタン、タブ、リストやテーブルなど多くのコントロールが提供されています。これらのデザインはプログラムから変更したり、JavaFX CSSをつかって外部ファイルにて定義することができます。

![JavaFX UIのコントロール](/images/2013-12-13-fextile-javafxcomponents.jpg)

JavaFXのコントロールは、一定のデザインテイストで制作されているためボタンやタブ、スクロールバーなどの手触り感は共通しています。ただ、ファイル削除など「破壊的な操作をするためのボタン」とか、決済をするための「同意をするためのボタン」といったユーザに対する意図を伝えるためのカテゴリ分けまでは定義されていません。このため開発者はそれぞれアプリケーションの都合に合わせて色やサイズを変えることになるのですが、自由度が高すぎて制作コストが高くなったりします。

さて、この話何か見覚えがあります。これを解決するためには、Twitter BootstrapのようなUIフレームワークがJavaFX向けにあればいいのではないか。早速、そういったライブラリがないか探してみました。ただ、ある程度探してみた限りでは求めているようなUIフレームワークは見当たりませんでした。無ければ作るしかありません。

## JavaFX向けのTwitter Bootstrapを作る

さて、JavaFXには前述の通りJavaFX CSSというCSSライクな機能が提供されています。ではTwitter BootstrapのCSSをそのまま移植すればよいのではないかというのが最初の考えでした。ただ、残念ながらJavaFX CSSとCSSは直接的な互換性はなく、またJavaFX CSSからは指定できない項目があるなど簡単な移植ではすまないということが分かり、JavaFX CSSと合わせてクラスライブラリも作成することにしました。

Java言語で作ってもよかったのですが、今回は最近勉強中のScala言語を使って実装することにしました。Scalaには[ScalaFX](http://code.google.com/p/scalafx/)というScalaからJavaFXを利用するためのライブラリがありますので、ScalaFXをベースに作ることにしました。

## Fextile<

ライブラリの名前はFextileにしました。テキスタイル(Textile; 織物、布地)とJavaFXを何となく合わせた造語です。まずはデザインテイストを全体的にTwitter Bootstrapに合わせてデザインし、実際のアプリケーションをデザインしながら優先度の高いコンポーネントを製作しました。でき上がったのが次のサンプルのようなコンポーネントです。

![Sample 1](/images/2013-12-13-fextile-sample1.png)

```scala
package fextile.sample

import fextile._
import scalafx.application.JFXApp
import scalafx.scene.control.Label

object Sample1 extends JFXApp {
  lazy val sample = new VContainer {
    content = Seq(
      new H1 {
        text = "Fextile Sample"
      },
      new Label {
        text = "Lorem ipsum dolor sit amet, ex sit tempor ceteros interesset. Per ei epicuri complectitur, has quodsi accumsan consetetur eu. Vis at meis quando, ne nec ullum instructior. Vel dolore alienum explicari et, hinc inermis corpora vel an, nominati inciderint eam ad. Cum ex dolor malorum vituperata, cu possit equidem accusata cum."
        wrapText = true
      },
      new Label {
        text = "Lorem ipsum dolor sit amet, ex sit tempor ceteros interesset. Per ei epicuri complectitur, has quodsi accumsan consetetur eu. Vis at meis quando, ne nec ullum instructior. Vel dolore alienum explicari et, hinc inermis corpora vel an, nominati inciderint eam ad. Cum ex dolor malorum vituperata, cu possit equidem accusata cum."
        wrapText = true
        styleClass = Seq("text-primary")
      }
    )
  }

  stage = new JFXApp.PrimaryStage {
    title = "Fextile"
    width = 800
    height = 600

    // use fextile.Scene instead of scalafx.scene.Scene
    scene = new Scene {
      root = sample
    }
  }
}
```

* [Sample1.scala](https://github.com/watermint/Fextile/blob/0.0.14/src/main/scala/fextile/sample/Sample1.scala)

ヘッダとテキスト。サンプルではテキストの色はデフォルトのものと、Twitter Bootstrapでのtext-primary相当のものを使っています。JavaFX CSS側にはtext-muted、text-warningなど他のクラスを定義されていますので、利用シーンごとに指定を変えて使います。

次のサンプルではデバイスや画面の広さによってコンポーネントの配置が適切に変更されるような、いわゆるレスポンシブデザインをサポートしたコンポーネントを利用しています。

![Sample 2 - 画面が広い場合](/images/2013-12-13-fextile-sample2-wide.png)

画面が狭くなると、2列に表示されていたコンポーネントは1列に整列し直されます。

![Sample 2 - 画面が狭い場合](/images/2013-12-13-fextile-sample2-narrow.png)

```scala
package fextile.sample

import fextile._
import scalafx.scene.control.Label
import scalafx.application.JFXApp

object Sample2 extends JFXApp {
  lazy val sample2 = new GridRow {
    add(
      new H1 {
        text = "Responsive"
      },
      xs12, sm12, md12, lg12
    )
    add(
      new Label {
        text = "Lorem ipsum dolor sit amet, ex sit tempor ceteros interesset. Per ei epicuri complectitur, has quodsi accumsan consetetur eu. Vis at meis quando, ne nec ullum instructior. Vel dolore alienum explicari et, hinc inermis corpora vel an, nominati inciderint eam ad. Cum ex dolor malorum vituperata, cu possit equidem accusata cum."
        wrapText = true
      },
      xs12, sm6, md6, lg6
    )
    add(
      new Label {
        text = "Lorem ipsum dolor sit amet, ex sit tempor ceteros interesset. Per ei epicuri complectitur, has quodsi accumsan consetetur eu. Vis at meis quando, ne nec ullum instructior. Vel dolore alienum explicari et, hinc inermis corpora vel an, nominati inciderint eam ad. Cum ex dolor malorum vituperata, cu possit equidem accusata cum."
        wrapText = true
        styleClass = Seq("text-primary")
      },
      xs12, sm6, md6, lg6
    )
  }

  stage = new JFXApp.PrimaryStage {
    title = "Fextile"
    width = 800
    height = 600

    // use fextile.Scene instead of scalafx.scene.Scene
    scene = new Scene {
      root = sample2
    }
  }
}
```

* [Sample2.scala](https://github.com/watermint/Fextile/blob/0.0.14/src/main/scala/fextile/sample/Sample2.scala)

次のサンプルではiOSのUINavigationControllerのように、1つの画面から左から右へとより詳細な情報を表示するためのコンポーネントを使っています。ボタンを押すと、画面全体が左へスライドし新しい画面が表示されます。

![Sample3](/images/2013-12-13-fextile-sample3.png)

* [Sample3.scala](https://github.com/watermint/Fextile/blob/0.0.14/src/main/scala/fextile/sample/Sample3.scala)

いまのところこれだけですが、比較的実用的になってきました。これから実際のアプリケーションを実装する過程では、まだ多くのリファクタリングやコンポーネント追加が必要になりますが、簡単なユースケース向けにはひとまず形としてでき上がったかなと思っています。

[Fextile](https://github.com/watermint/Fextile/blob/0.0.14)はMITライセンスにてオープンソースとして公開しておりますので、同じような問題で悩まれている方はぜひお使いいただき、新しい提案もいただければ幸いです。
