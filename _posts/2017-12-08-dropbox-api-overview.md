---
layout: post
title: 'Dropbox APIの概要'
tags:
 - dropbox
 - dropbox api
 - api
---

DropboxのAPIについて説明している日本語情報が少ないようなので、個人的にメンテナンスしている[toolbox](https://github.com/watermint/toolbox)というプロジェクトの経験をもとに説明していこうと思います。APIを網羅的に解説するよりは、勘所をつかむための解説にしようと思いますので詳細は英語版とはなりますが[公式ドキュメント](https://www.dropbox.com/developers)をご参照ください。またOAuthやJSON、curlコマンドなどWeb APIにて一般的な概念やツールなどについては説明を省略します。

なお、本記事は個人の見解であり、執筆時点での情報を元としています。記事の正確性などについて保証するものではない点ご容赦ください。最新情報については[公式ドキュメント](https://www.dropbox.com/developers)をご参照ください。

前置きが長くなりましたが今回は概要です。

---

# Dropbox APIの種類

Dropboxより提供されているAPIには大きく分けて下記二種類があります。

* Dropbox API または User Endpoints ... ユーザー毎のファイル管理など
* Dropbox Business API または Business Endpoints ... Dropbox Businessチームの管理

なおドキュメントによって文脈によって単に「Dropbox API」とだけいったときに両方を含む場合もあれば、ユーザー毎のファイル管理だけを指す場合もあります。一部厳密性に欠ける部分もありますが、本記事では両方を指す場合にDropbox API、個別に種類を言い分けるためにはUser EndpointsとBusiness Endpointsという呼称を使うことにします。

# API Explorer

ドキュメントを読み込むなどある程度準備してから取りかかるのもいいですが、まずは簡単に試してみるのが手っ取り早いでしょう。[API Explorer](https://dropbox.github.io/dropbox-api-v2-explorer/)というツールが公開されていますのでこちらを利用します。

![API Explorer](/images/2017-12-08-api-explorer.jpg)

左側にAPI一覧があり、APIを選択すると指定できるパラメータなどがフォームとして表示されます。わかりづらいですが、右上に「Switch to Business Endpoints」というリンクがありますのでBusiness Endpointsについても試すことができます。

ためしにフォルダ内のファイル一覧を表示してみましょう。APIはカテゴリ毎にパスが区切られていて、ファイル操作であれば `https://api.dropboxapi.com/2/files/` 以下のエンドポイントが該当します。
この中の、[list_folder](https://www.dropbox.com/developers/documentation/http/documentation#files-list_folder)というAPIがファイル一覧取得のためのAPIです。API Explorerではこのカテゴリ毎に整理されていますのでfiles以下からlist_folderを探してみてください。

![list_folder](/images/2017-12-08-list_folder.jpg)

まずは認証から。`Get Token`というボタンがあるのでこれを押してトークンを取得します。Dropboxへのログインや認可の確認などが求められますがそれらを確認のうえトークン文字列を取得してください。

早速実行してみましょう。Submit Callボタンを押します。パラメータ`path`に何も入れなければDropboxフォルダのルートからの一覧になります。逆に、ルートフォルダを一覧するために `/` は使えません。Show Codeボタンを押せば`curl`での実行例も表示されます。戻り値については[list_folder](https://www.dropbox.com/developers/documentation/http/documentation#files-list_folder) APIドキュメントに記載がありますが、正常ケース・異常ケースともに結果がJSON形式で返されます。

# アプリケーションの登録

Dropbox APIを使って開発を進めるにはまずDropboxへアプリケーションの登録が必要となります。登録のためにはDropboxアカウントが必要となります。Dropbox開発者ページの[My apps](https://www.dropbox.com/developers/apps)から登録します。

![Create App](/images/2017-12-08-newapp.jpg)

APIの種別、アクセス範囲などを選択した上でアプリケーション名を登録します。アプリケーション名はDropboxサービス全体で一意になる必要があり重複した名称をつけることはできません。開発したいアプリケーションで複数のアクセス範囲を使いたい場合にはそれらアクセス範囲ごとにアプリケーションの登録が必要となるので、命名規則なども準備したうえで登録するとスムーズです。

アクセス範囲についての詳細は次の通りです。

## User Endpoints

### App Folder

アクセス範囲をアプリケーションのみが使用するフォルダに限定します。Dropboxフォルダ以下に「アプリ」またはユーザーの利用設定言語が英語などであれば「App」などのフォルダが作成され、このアプリフォルダ以下に`/アプリ/アプリ名`のようなフォルダが作成されます。App Folderを選択するとこのフォルダ以下しか操作することができません。

### Full Dropbox

該当アカウントのすべてのファイルを操作できます。

## Business Endpoints

Dropbox Business向けのAPIはチーム内のアカウント管理や監査ログを取得するなど目的に応じて範囲が分けられています。

### Team information

チームに関する情報と利用統計情報などにアクセスできます。情報を取得するだけ管理など変更処理はできません。

### Team auditing

Team informationの範囲に加え、チーム監査のために利用します。[アクティビティ監査ログ](https://www.dropbox.com/help/business/activity-audit)のような情報にアクセスすることができます。

### Team member file access

Team informationならびにTeam auditingに加え、任意のチーム内アカウントへのファイル操作などアクションを実行することができます。たとえば、チーム内のアカウントすべての共有フォルダの共有先一覧を取得したいといった場合にはこのアクセス範囲を利用します。

### Team member management

Team informationに加え、チーム管理者としての操作をするためのアクセス範囲です。チームメンバーを招待したり、アカウントの情報編集、削除などといった管理操作が可能です。たとえば、社内ワークフローと組み合わせて社内ワークフローで承認された人だけにDropbox Businessアカウントを招待するという用途にはこのアクセス範囲を利用します。

# アプリケーションの設定

アプリケーションを登録すると設定画面が表示されます。OAuth2のGenerated access tokenという欄にGenerateというボタンがあります。これを押すと、既存のログイン済みアカウントで認可済みのトークンが生成されます。テストなどで利用すると良いでしょう。ちゃんとしたOAuth2シーケンスでトークンを取得する場合に必要となるApp key/secretについてもこの画面から参照することができます。

![App settings](/images/2017-12-08-app-config.jpg)

なおBusiness Endpoints向けのアプリケーションではこの操作を行うのは、そのチームのチーム管理者である必要があります。

# Dropbox SDK

API Explorerで勘所がつかめ、アプリケーション設定が終わったらプログラミング開始です。REST APIなのでパラメータや戻り値を加工すれば何とでも呼び出しはできますが、やはりあらかじめ構造体/クラスなどが定義されたSDKを使った方が手軽です。

[Choose your platform](https://www.dropbox.com/developers/documentation)ドキュメントを参照すると代表的なプログラミング言語用の公式SDKが一覧されています。執筆時点での対応言語は下記の通り。

* [Swift](https://www.dropbox.com/developers/documentation/swift)
* [Objective-C](https://www.dropbox.com/developers/documentation/objective-c)
* [Python](https://www.dropbox.com/developers/documentation/python)
* [.NET](https://www.dropbox.com/developers/documentation/dotnet)
* [Java](https://www.dropbox.com/developers/documentation/java)
* [JavaScript](https://www.dropbox.com/developers/documentation/javascript)

GoやRuby向けなどこのなかにない場合には[コミュニティにてメンテナンスされているSDK](https://www.dropbox.com/developers/documentation/communitysdks)も利用できます。

さらにコミュニティSDKもない場合、たとえばPowerShellで簡単な管理ツールを作りたいというようにこの中で対応言語がない場合には[HTTP](https://www.dropbox.com/developers/documentation/http/overview)のAPIドキュメントを参照しながら作成していくことになります。

## SDKの選び方

APIはバージョン毎に互換性が保たれるよう提供されていますが、比較的頻繁に新しいAPIやパラメータ、戻り値が追加されるのでこれらに追従することを考えると公式SDKを採用した方がやや有利です。

Dropbox公式SDKは[stone](https://github.com/dropbox/stone)というツールを使って自動生成されるようになっていますから、こういった変化への追従に強い点が特徴です。

一方で、お作法が統一されていて言語毎に特化した書き方ではないのでややなじみにくく感じる部分もあるかもしれません。

ファイルをアップロードするだけ、とかダウンロードするだけといった用途が一定または特化している場合にはコミュニティSDKのほうがなじみやすいかもしれません。

# Dropbox APIの使い処

Dropbox上でできる主要操作はだいたいDropbox APIを利用すれば実現できます。また逆に、APIでなければ実現できないものもあります。少し実例を元に紹介していきましょう。

## 共有リンクのパスワード/有効期限を一括設定する

Dropbox ProfessionalやDropbox Business契約があるばあいには、[共有リンクへパスワードや有効期限を設定することができます](https://www.dropbox.com/help/files-folders/link-expiration)。ただ、共有リンク一つ一つに画面から設定するのは面倒なので一括設定したいと考えることもあります。こういったときにDropbox APIを利用すると実現できます。

たとえば拙作[toolbox](https://github.com/watermint/toolbox/releases/tag/22.20171004.2)ではこの操作を実装しました。

Business Endpointsのアクセス権「Team member file access」を使い、すべてのユーザーについて共有リンク一覧を取得します。

すべてのユーザーに対して処理を行うにはまず[teams/members/list](https://www.dropbox.com/developers/documentation/http/teams#team-members-list)でアカウント一覧を取得します。各ユーザーについて[sharing/list_shared_links](https://www.dropbox.com/developers/documentation/http/documentation#sharing-list_shared_links) APIでリンクを取得するのですがこのときに、[Member file access](https://www.dropbox.com/developers/documentation/http/teams#teams-member-file-access)にあるとおり、HTTPヘッダに`Dropbox-API-Select-User: チームメンバーID`を追加すると、その該当アカウントとして処理が行われます。

共有リンクが取得できたら、同様に`Dropbox-API-Select-User: チームメンバーID`を使って、各ユーザーそれぞれの共有リンクについて[sharing/modify_shared_link_settings](https://www.dropbox.com/developers/documentation/http/documentation#sharing-modify_shared_link_settings)を使って有効期限を今日から7日後などに上書きします。

## 共有フォルダへの参加者一覧

監査などの目的でDropbox Businessチーム内の各利用者が参加している共有フォルダの参加者一覧を取得したい場合があるとします。
これも以前Pythonで実装したことがあります。

* [ListSharedFolderMembers.py](https://github.com/dropbox/DropboxBusinessScripts/blob/master/Sharing/ListSharedFolderMembers.py)

Business Endpointsのアクセス権「Team member file access」を使い、すべてのユーザーについて処理を行います。
共有フォルダの場合にはアクセス権限がメンバー単位だけでなくグループ単位の場合もあるのでグループ情報を取得するために[teams/groups/list](https://www.dropbox.com/developers/documentation/http/teams#team-groups-list)なども使っています。

## 大量ファイルのコピー/移動/削除など

Web画面から操作する際にファイルが一定数を超えていると処理ができない旨が表示されます。Dropboxアプリケーション実行すれば大量ファイルでも処理はできるのですが、パソコン上に同期していない場合などでは処理ができないのと、パソコンにやや負荷もかかります。

このためそれぞれのファイルを逐次コピーないし、移動するようにUser EndpointsのAPIを使えば着実に処理を進めることができます。コピーであれば[files/copy_v2](https://www.dropbox.com/developers/documentation/http/documentation#files-copy_v2)、移動であれば[files/move_v2](https://www.dropbox.com/developers/documentation/http/documentation#files-move_v2)といったAPIを使います。

なおAPIを使った場合でもAPIドキュメントにあるとおり10,000ファイル以上を同時に処理することはできません。

> too_many_files
> The operation would involve more than 10,000 files and folders.

小分けにして処理する必要があります。

## ログイン済みデバイス一覧や強制ログアウト

Business Endpointsの[team/devices/list_member_devices](https://www.dropbox.com/developers/documentation/http/teams#team-devices-list_members_devices) APIを使うとDropbox Businessチーム内のデスクトップ、モバイル、Webとそれぞれのセッション一覧を取得することができます。セッション情報には最終接続IPアドレスやUser Agentなども含まれるので、一定のルールを元に強制ログアウトを実施すると行った使い方もできます。たとえば、社内IP以外が検出されたら強制ログアウトなど。

強制ログアウトのためには[team/devices/revoke_device_session](https://www.dropbox.com/developers/documentation/http/teams#team-devices-revoke_device_session) APIを使います。

# まとめ

Dropbox APIを使うと社内ワークフローと連携して自動処理を行ったり、運用ポリシーなどに合わせて管理する、あるいはDevOpsのように開発ワークフローと組み合わせて、Dropboxの特定フォルダにファイルがアップロードされたらビルド開始。といったことも実現可能です。

お作法はあるていどあるものの、一度覚えてしまえばやりたいことが簡潔に書けるようなAPIがそろっているかなと思います。
