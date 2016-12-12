---
layout: post
title: Twitter Bootstrap like UI framework for JavaFX/ScalaFX
date: '2013-12-13T11:55:00+00:00'
tags:
- ui
- ux
- design
- java
- javafx
- scala
- scalafx
- bootstrap
- twitter bootstrap
- framework
tumblr_url: http://watermint.org/post/69845345116/bootstrap-like-ui-framework-for-javafx-and-scalafx
---

## Feeling of Applications

Applications which running on iOS, Android, Windows, and OS X have nearly identical user experience. There are guidelines of fonts, color, margins, etc for labels, buttons, etc including effects or animation for action. User experience guidelines are publicly available for developers like below.

* [iOS Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/userexperience/conceptual/mobilehig/)
* [Design Android Developers](http://developer.android.com/design/index.html)
* [Design library for Windows Phone](http://msdn.microsoft.com/en-us/library/windowsphone/design/hh202915(v=vs.105).aspx)

Application developers and designers refer these guidelines, then design UI/UX for their applications. For iOS and Windows Phones have official review for publishing on application store. Applications may reject when their applications are not compatible with design guidelines.


## Twitter Bootstrap

However, there are no reviews for cross platform applications like web applications. Web applications may have different feeling compared to their native applications. Of course, we could find guidelines or best practices for web application UI/UX design. For example, [Bootstrap](http://getbootstrap.com/) is well known UI framework for designing web applications. Bootstrap is collection of CSS/JavaScript to define UI component and their categories. Defined UI components are like

* Red button for destructive operation like delete file, delete repository, etc.
* Yellow text region for attention to the user to describe warnings.

![Bootstrap UI components](/images/2013-12-13-fextile-bootstrapcomponents.png)

Bootstrap also have customization feature, called theme, to define own color scheme, fonts, sizes, etc.

## JavaFX and UI framework

JavaFX is a new UI framework running on Java. JavaFX have varieties of UI components, like buttons, tab pane, tables, lists, sliders etc.

![JavaFX UI components](/images/2013-12-13-fextile-javafxcomponents.jpg)

However, at least I know, there is no UI framework like Twitter Bootstrap. There are no definitions like button for destructive operation, warning text, approval operation, etc. That&rsquo;s why I started development of UI framework for JavaFX/ScalaFX.

## Fextile; UI framework for JavaFX/ScalaFX

[Fextile](https://github.com/watermint/Fextile/blob/0.0.14) is my project name which coined from JavaFX and textile. On the begining of the project, I just thought it's easy to port Bootstrap's CSS into JavaFX because JavaFX has similar feature called JavaFX CSS. But there are no compatibility except syntax. So I needed to create both JavaFX CSS and class libraries.

Current implementation of Fextile built based upon ScalaFX. [ScalaFX](http://code.google.com/p/scalafx/) is a DSL/wrapper of JavaFX on Scala.

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

Sample1; simple header and labels. First label uses default fonts, size and color. Second label uses "text-primary" class on Bootstrap. Fextile also defines "text-muted", "text-warning", etc.

![Sample 2 - Wider screen](/images/2013-12-13-fextile-sample2-wide.png)

Sample 2; responsive layout. This sample is responsive to screen width. On wider screen, texts are aligned as two columns.

![Sample 2 - Narrow screen](/images/2013-12-13-fextile-sample2-narrow.png)

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

For narrow screen, text are aligned as one column.

![Sample3](/images/2013-12-13-fextile-sample3.png)

* [Sample3.scala](https://github.com/watermint/Fextile/blob/0.0.14/src/main/scala/fextile/sample/Sample3.scala)

Sample 3; screen to screen navigation. This sample uses screen to screen navigation component like `UINavigationController` in iOS. Screen scroll left on "Next" button pressed.

[Fextile](https://github.com/watermint/Fextile) is on early starge of development. If you'd like to have more components. Please send me pull-requests on github. Thanks.

