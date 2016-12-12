---
layout: post
title: Akka + JavaFX Proof of Concept
date: '2015-02-24T12:17:00+00:00'
tags:
- akka
- javafx
- scala
tumblr_url: http://watermint.org/post/111919630702/akka-javafx-proof-of-concept
---

Since GUI framework like JavaFX requires always run on main thread for handle events/manipurating UI properties. This usually cause unexpected behaviors. Fextile trying to separate layout specific codes, handling events, manipurating UI properties, and business logic on top of Akka async messaging framework.

Defining layout is very similar to JavaFX/ScalaFX applications.

```scala
object MyApp extends FextileApp {
  // Do layout just like JavaFX/ScalaFX application
  stage = new PrimaryStage {
    title = "My Application"
    width = 800
    height = 600
    scene = new Scene {
      fill = Color.BLUE
    }
  }

  override def props: Option[Props] = Some(Props[MyApp])
}
```

For all events from scene graph are transfered to actors on Akka.

```scala
class MyApp extends Actor {
  override def receive: Receive = {
    case (s: Scene, e: MouseClicked) =>
      s.fill = Color.RED

    case (_, _: WindowHidden) =>
      Fextile.shutdown()
  }
}
```

code on github: [semester/semester-foundation-fextile](https://github.com/semester/semester-foundation-fextile)
