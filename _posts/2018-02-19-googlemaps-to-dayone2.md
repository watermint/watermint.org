---
layout: post
title: 'Googleマップのスターが消える問題と、Day One 2に移行した話'
tags:
 - googlemaps
 - dayone
---

Google Mapsのスターとは、お気に入りのレストランなどにスターをつけてメモしておけるというものです。旅行先で行こうと考えている観光スポット、知人から教わったおすすめレストランなど今まで記録はすべてGoogle Mapsでスターをつけることでメモしていました。
ただこの便利な機能にも不満が2点。一つはスターが勝手に消えてしまうことで。二つ目があまり追加情報が書き込めないことです。

![Google Mapsでのスター](/images/2018-02-19-googlemaps.jpg)

今回はGoogle Mapsでスターをつけた位置をDay One 2の記事として移行した際のお話です。なお各種仕様は変わるかもしれませんので試される際には自己責任で。

# Google Mapsでの2つの課題

## 課題1: スターが消える問題

まず一つ目のスターが勝手に消えてしまう問題について。少し検索してみると下記のように詳細に調査されている記事が見つかりました。

* [Googleマップの「スター」(お気に入りの場所)が、古いものから消えて表示されなくなる問題](http://www.bousaid.com/entry/2017/06/05/081606)

いろいろ丁寧に検証されているので大変参考になりました。結局、保存はされているが、どうやら表示数に制限があり、古いものから表示されなくなるのではとのこと。データが保存されていても簡単に取り出せないとするとあまり保存している意味もありません。

## 課題2: 追加情報が書き込めない

この中華料理屋さんの担々麺がおいしいとか、この焼き鳥屋さんは塩よりタレがおすすめとか些細ではあるものの、教えてもらった情報は書き留めておきたいものです。Google Mapsでは最近スターの種類が増えたのでカテゴリごとにスターを分けたりできますが、それでも追加情報を書き込むという段階まではたどり着きません。

Google+などSNSを使って管理する方法もあるのですが、各所に情報が分散しても使いづらいですし、自分で投稿したものにもかかわらず検索が難しいので数年前からこの目的のためにSNSは利用していません。

# Day One 2へ移行

[Day One 2](http://dayoneapp.com/)はもともと日記などライフログをつけるためのアプリです。写真やメモに対して位置情報やタグなどを追加して情報を管理することができます。

Day Oneは2013年頃から使い始めていて、ようやく最近Day One 2へ移行を完了しました。Day One 2では基本的な使い勝手はそのままにかなり機能が追加されています。よく見ると、Day One 2ではJSON Zip Fileという形式でデータをエクスポートしたり、インポートすることができます。これを使えば、Google Mapsから一括してデータを読み込みできそうです。

## Day One 2のJSON Zip File形式

JSON Zip File形式のドキュメントは見つかりませんでしたが、データ形式としてはごく単純なものです。ZIPファイルの中にJournalごとのJSONファイルと、写真データが格納されています。Day One 2にデータをインポートするにはこのJSONファイルをインポートすれば良さそうです。

JSONファイルは次のようなフォーマットです。Day One 2 version 2.5.10にて幾つかテストをしてみたところ、インポートの際には記事ごとのuuidは振らなくても自動的に発番されるので問題ありません。locationデータについても緯度(longitude)・経度(latitude)のみあればよく他は省略してもインポートは成功します。

```json
{
"metadata" : {
  "version" : "1.0"
},
"entries" : [
{
  "starred" : false,
  "location" : {
    "region" : {
      "center" : {
        "longitude" : 120.1973037719727,
        "latitude" : 22.99810981750488
      },
      "radius" : 75
    },
    "longitude" : 120.1973,
    "placeName" : "有方公寓 Your Fun Apartment",
    "latitude" : 22.99811
  },
  "creationDate" : "2015-03-14T11:35:05Z",
  "text" : "[有方公寓 Your Fun Apartment](http:\/\/maps.google.com\/?cid=1337703935590192274)\n\nNo. 9, Lane 269, Section 2, Hai'an Road, West Central District, Tainan City, Taiwan 700",
  "timeZone" : "Asia\/Tokyo",
  "uuid" : "345585B421AF47DEA98B497C66CFBF8F",
  "duration" : 0
}
]}
```

## Google Takeoutでスター一覧を取得

Google TakeoutでスターをつけたデータをGeoJSONとして取り出します。手順については先ほど掲載させていただいた[Googleマップの「スター」(お気に入りの場所)が、古いものから消えて表示されなくなる問題](http://www.bousaid.com/entry/2017/06/05/081606)に詳しく書かれています。

GeoJSONの中身をみてみると以下のように緯度、経度、タイトル、Google Maps上でのURLなどが得られることがわかりました。

```json
{
  "type" : "FeatureCollection",
  "features" : [ {
    "geometry" : {
      "coordinates" : [ 120.1972925, 22.9981111 ],
      "type" : "Point"
    },
    "properties" : {
      "Google Maps URL" : "http://maps.google.com/?cid=1337703935590192274",
      "Location" : {
        "Address" : "No. 9, Lane 269, Section 2, Hai'an Road, West Central District, Tainan City, Taiwan 700",
        "Business Name" : "有方公寓 Your Fun Apartment",
        "Country Code" : "TW",
        "Geo Coordinates" : {
          "Latitude" : "22.9981111",
          "Longitude" : "120.1972925"
        }
      },
      "Published" : "2015-03-14T11:35:05Z",
      "Title" : "有方公寓 Your Fun Apartment",
      "Updated" : "2017-02-22T12:03:22Z"
    },
    "type" : "Feature"
  },
]}
```

## Day One 2のJSON Zip File形式に変換

あとは形式を変更するだけですが、JSON同士の変換となるので簡単です。今回はRubyで書きました。ライセンスはMITライセンスとしておきます。

```ruby
# MIT License
# 
# Copyright (c) 2018 Takayuki Okazaki
# 
# Permission is hereby granted, free of charge, to any person obtaining
# a copy of this software and associated documentation files (the
# "Software"), to deal in the Software without restriction, including
# without limitation the rights to use, copy, modify, merge, publish,
# distribute, sublicense, and/or sell copies of the Software, and to
# permit persons to whom the Software is furnished to do so, subject to
# the following conditions:
# 
# The above copyright notice and this permission notice shall be
# included in all copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
# EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
# CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
# TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
# SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

require 'json'

def feature2entry(feature)
  def geo(f, l)
    if f.key?('Geo Coordinates')
      f['Geo Coordinates'][l]
    elsif f.key?(l)
      f[l]
    else
      p feature
      exit
    end
  end
  if feature['Location'].key?('Address')
    address = "\n\n#{feature['Location']['Address']}"
  else
    address = ''
  end
  if feature['Location'].key?('Business Name')
    place_name = feature['Location']['Business Name']
  else
    place_name = feature['Title']
  end

  {
    :location => {
      :placeName => place_name,
      :longitude => geo(feature['Location'], 'Longitude').to_f,
      :latitude  => geo(feature['Location'], 'Latitude').to_f,
    },
    :text => "[#{feature['Title']}](#{feature['Google Maps URL']})#{address}",
    :creationDate => feature['Published'],
  }
end

def dayone(entries)
  journal = {
    :metadata => {
      :version => "1.0"
    },
    :entries => entries
  }
  JSON.generate(journal)
end

geojson = JSON.load(open('Saved Places.json'))
entries = Hash[geojson['features'].map { |f| [f['properties']['Google Maps URL'], f] }]
journal = dayone(entries.values.map { |f| feature2entry(f['properties']) })

puts journal
```

あとはこの出力を Location.json など新しいジャーナル名として保存し、zipファイルにしてインポートします。これで無事にDay One 2に読み込み完了です。あとは、ゴミデータを削除したり、中華料理とか和食とかタグをつけて整理すれば完了です。

![記事として読み込まれた位置情報](/images/2018-02-19-dayone1.jpg)

地図表示に切り替えればGoogle Mapsと同じように周辺の地図といままでメモした場所を一覧して確認することができます。

![地図表示](/images/2018-02-19-dayone2.jpg)

