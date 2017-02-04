---
layout: post
title: 'Product/Service GA/EOL/EOSL dates'
tags:
 - ga
 - eol
 - eosl
 - ruby
 - php
 - java
 - mysql
---

# Glossary

* GA ... General Availability
* EOL ... End of Life. End of support for enhancement and/or bug fix.
* EOSL ... End of Service Life. End of support for both bug fix and security fix.
* TBD ... To be determined.

# Programming Languages

## Ruby (MRI)

| Version | GA         | EOL            | EOSL           |
|---------|------------|----------------|----------------|
| 1.9.3   | 2011-10-31 | 2014-02-23     | 2015-02-25     |
| 2.0.0   | 2013-02-24 | TBD (2015-02?) | TBD (2016-02?) |
| 2.1     | 2013-12-25 | TBD (2015-12?) | TBD (2016-12?) |
| 2.2     | 2014-12-25 | TBD (2016-12?) | TBD (2017-12?) |

* After 2.0, expected EOL date is +2 years from GA. Guessing EOSL date is +1 year from EOL.
* [https://twitter.com/yukihiro_matz/status/421152831844806656](https://twitter.com/yukihiro_matz/status/421152831844806656)
* [ReleaseEngineering](https://bugs.ruby-lang.org/projects/ruby/wiki/ReleaseEngineering)

## PHP

| Version | GA         | EOL            | EOSL           |
|---------|------------|----------------|----------------|
| 5.4     | 2012-03-01 | 2014-09-14     | 2015-09-14     |
| 5.5     | 2013-06-20 | 2015-06-20     | 2016-06-20     |
| 5.6     | 2014-08-28 | 2016-08-28     | 2017-08-28     |

* [Supported Versions](http://php.net/supported-versions.php)

## Java SE

| Version | GA         | Public EOL     |
|---------|------------|----------------|
| 5.0     | 2004-05    | 2009-10        |
| 6       | 2006-12    | 2013-02        |
| 7       | 2011-06    | 2015-04        |
| 8       | 2014-04    | TBD            |

* Oracle offers premier/extended/sustaining support plans.
* [http://www.oracle.com/technetwork/jp/java/eol-135779.html](http://www.oracle.com/technetwork/jp/java/eol-135779.html)

# Database

## MySQL

| Version | GA         | Public EOL     |
|---------|------------|----------------|
| 5.0     | 2005-10-15 | 2012-01-09     |
| 5.1     | 2008-11-27 | 2013-12-31     |
| 5.5     | 2010-12-03 | 
| 5.6     | 2013-02-05 | 

* [http://www.mysql.com/support/eol-notice.html](http://www.mysql.com/support/eol-notice.html)
* [http://en.wikipedia.org/wiki/MySQL](http://en.wikipedia.org/wiki/MySQL)

# KVS

## Redis

| Version | GA         | EOL/EOSL       |
|---------|------------|----------------|
| 2.4     | 2011-10-18 | 2012-11-29(?)  |
| 2.6     | 2012-10-24 | 2013-12-11(?)  |
| 2.8     | 2013-11-22 | |
| 3.0     | TBD        | |

* No explicit EOL/EOSL policy found. Guessing EOL/EOSL dates from latest update of each version (2.4.18, 2.6.17).
* [https://github.com/antirez/redis](https://github.com/antirez/redis)

# Server OS

## Reference

* [各OSのリリース日とサポート終了日を表にまとめてみた](http://qiita.com/yunano/items/4757f86f9e92bb4f503f)

## Debian (Server)

| Version           | GA         | EOL/EOSL       |
|-------------------|------------|----------------|
| 6.0 (Squeeze) LTS | 2011-02-06 | 2016-02-06     |
| 7.0 (Wheezy)      | 2013-05-04 | TBD            |

* [https://wiki.debian.org/DebianReleases](https://wiki.debian.org/DebianReleases)
* [https://wiki.debian.org/DebianSqueeze#Debian.2FSqueeze_Life_cycle](https://wiki.debian.org/DebianSqueeze#Debian.2FSqueeze_Life_cycle)

## Ubuntu (Server)

| Version            | GA         | EOL/EOSL       |
|--------------------|------------|----------------|
| 10.04 LTS          | 2010-04-29 | 2015-04        |
| 12.04 LTS (Precise)| 2012-04-26 | 2017-04        |
| 14.04 LTS (Trusty) | 2014-04-17 | 2019-04        |

* [https://wiki.ubuntu.com/Releases](https://wiki.ubuntu.com/Releases)

## CentOS

| Version | GA         | EOL            | EOSL           |
|---------|------------|----------------|----------------|
| 3       | 2004-03-19 | 2006-06-20     | 2010-10-31     |
| 4       | 2005-03-09 | 2009-03-31     | 2012-02-29     |
| 5       | 2007-04-12 | 2014-Q1        | 2017-03-31     |
| 6       | 2011-07-11 | 2017-Q2        | 2020-11-30     |
| 7       | 2014-07-07 | 2020-Q4        | 2024-06-30     |

* [http://en.wikipedia.org/wiki/CentOS](http://en.wikipedia.org/wiki/CentOS)
* [http://wiki.centos.org/About/Product](http://wiki.centos.org/About/Product)

# Mobile OS

## Reference

* [Android Support vs iOS Support](http://www.fidlee.com/android-support-vs-ios-support/)

## iOS

| Version | GA         | EOL/EOSL      |
|---------|------------|---------------|
| 6       | 2012-09-19 | 2013-02-21(?) |
| 7       | 2013-09-18 | 2014-06-30(?) |
| 8       | 2014-09-17 | TBD           |

* No explicit EOL/EOSL policy found. Guessing EOL/EOSL dates from latest update of each version.
* iOS 6.x latest update: [iOS 6.1.6](http://support.apple.com/en-us/HT202920)
* iOS 7.x latest update: [iOS 7.1.2](http://support.apple.com/en-us/HT203014)

## Android

| Version               | GA         | EOL/EOSL      |
|-----------------------|------------|---------------|
| 3.2 (API level 13)    | 2011-06-15 | 2012-02(?)    |
| 4.0 (API level 14/15) | 2011-10-18 | 2012-03-29(?) |
| 4.1 (API level 16)    | 2012-07-09 | 2012-10-09(?) |
| 4.2 (API level 17)    | 2012-11-13 | 2013-02-11(?) |
| 4.3 (API level 19)    | 2013-07-24 | 2013-10-03(?) |
| 4.4 (API level 20)    | 2013-10-31 | 2014-06-19(?) |
| 5.0 (API level 21)    | 2014-11-12 | TBD           |

* No explicit EOL/EOSL policy found. Guessing EOL/EOSL dates from latest update of each version.
* [http://en.wikipedia.org/wiki/Android_version_history](http://en.wikipedia.org/wiki/Android_version_history)
