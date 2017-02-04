---
layout: post
title: Play 2.2/ScalaでLogbackの設定をLTSVにする
date: '2013-10-09T06:36:00+00:00'
tags:
- play framework
- scala
- logger
- logback
- ltsv
tumblr_url: http://watermint.org/post/63375566413/play-scala-logback-ltsv
---

趣味のプログラムを[Play Framework 2.2](http://www.playframework.com/documentation/2.2.x/Home)とScalaで書いています。趣味のプログラムなので、ログを監視したりするような事はまず無いのですが、あとからプログラムで読みやすいように[LTSV (Labeled Tab-separated Values)](http://ltsv.org/)形式で出力する事にしてみました。

ログの設定は、`conf/application-logger.xml`というファイルを作っておけば読み込んで適用してくれます。いろいろ試行錯誤した結果次のような設定で落ち着きました。

```xml
<configuration>
    <conversionRule conversionWord="coloredLevel" converterClass="play.api.Logger$ColoredLevel" />
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${application.home}/logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${application.home}/logs/application.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>14</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>UTF-8</charset>
            <pattern>time:%date{ISO8601}&#x9;level:%level&#x9;logger:%logger&#x9;thread:%thread&#x9;msg:%replace(%replace(%message){'\n','\\n'}){'\t',' '}&#x9;exception:%replace(%replace(%xException{5}){'\n','\\n'}){'\t',' '}%n%nopex</pattern>
            <immediateFlush>true</immediateFlush>
        </encoder>
    </appender>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%coloredLevel %logger{15} - %message%n%xException{5}</pattern>
        </encoder>
    </appender>
    <logger name="play" level="INFO" />
    <logger name="application" level="INFO" />
    <root level="ERROR">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

STDOUTへの出力はデフォルトのままとしています。ファイル`${application.home}/logs/application.log`への書き込みのみLTSVとしています。

最初からログローテーションまでは設定しなくても良いのですが、ちょっと作ってみた。というレベルのプログラムでも意外と長期間動かしっぱなしになってしまった。という事もあるので、あらかじめ設定しておく事にしています。

ログに含める事ができる情報はLogbackのマニュアルによると[Conversion Word](http://logback.qos.ch/manual/layouts.html#conversionWord)で定義されているものが使えるようです。呼び出しもとメソッド名や行番号もとれるようですが、動作速度が犠牲になるようなのでここでは有効にしませんでした。

Play framework側のプロパティでapplication.home以外に使えるものはないのか情報を探してみましたが、ソースを見たところ([Logger.scala](https://github.com/playframework/playframework/blob/2.2.x/framework/src/play/src/main/scala/play/api/Logger.scala#L171)) application.home以外は特に定義されていませんでした。
