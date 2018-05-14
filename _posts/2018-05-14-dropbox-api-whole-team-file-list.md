---
layout: post
title: 'Dropbox API: チーム全体のフォルダ・ファイル一覧を取得する'
tags:
 - dropbox
 - dropbox api
 - api
---

Dropbox BusinessではAPIを通じてチーム内のユーザーについてファイル・フォルダ一覧を取得したり、更新することができます。これは、マルウェア対策システムがファイルを検査・隔離するために利用したり、ワークフローや生産性向上ツールが自動的にファイルを仕分けするといった用途を想定していると考えられます。なお便利な一方、非常に強力な権限となるため取り扱い上の注意は管理者パスワードなどと同様厳重に管理する必要があることは言うまでもありません。

このようにチーム全体のファイルを扱う場合には[Dropbox APIの概要のBusiness Endpoints](/2017/12/08/dropbox-api-overview/#business-endpoints)でご紹介したTeam member file access権限を用います。この権限を使うアプリケーションを作成する手順などは[該当記事](/2017/12/08/dropbox-api-overview/#%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E7%99%BB%E9%8C%B2)をご参照ください。

チームメンバーのファイルやフォルダを操作するには大きく分けて二種類の方法があります。

1. チームメンバーの代理として実行
2. 管理者として実行

チームメンバーの代理として実行する方法では、指定したユーザーが所有しているファイルやフォルダ、参加している共有フォルダ・チームフォルダなどへアクセスすることができます。
管理者として実行する場合にはチーム内すべてのファイル・フォルダへアクセスすることができます。

使い分けとして、たとえばワークフローアプリケーションなどで承認されたドキュメントを所定フォルダに移動するといった用途であれば、承認者の権限で承認者が所有フォルダに移動するといった場合には代理での実行が適切でしょう。マルウェアのスキャンや監査といったチーム全体を対象としたい場合には管理者としての実行が適切でしょう。

# チームメンバーの代理として実行

チームメンバーの代理として実行する場合にはまず[teams/members/list](https://www.dropbox.com/developers/documentation/http/teams#team-members-list)でアカウント一覧を取得します。User Endpointに対してAPIコールする際にHTTPヘッダ`Dropbox-API-Select-User: チームメンバーID`を追加すると、その該当アカウントとして処理が行われます。

たとえば[files/list_folder](https://www.dropbox.com/developers/documentation/http/documentation#files-list_folder)を代理として実行する場合には次のように呼び出します。

```sh
curl -X POST https://api.dropboxapi.com/2/files/list_folder \
  --header 'Authorization: Bearer 認証トークン' \
  --header 'Content-Type: application/json' \
  --header 'Dropbox-Api-Select-User: チームメンバーID' \
  --data '{"path":""}'
```

# チーム管理者として実行

チーム管理者として実行する場合には`Dropbox-API-Select-User`ヘッダの代わりに`Dropbox-API-Select-Admin`ヘッダを用います。また、チームメンバーIDにはチーム管理者のメンバーIDを指定します。
`Dropbox-API-Select-Admin`ヘッダを用いた場合、チーム内のネームスペースすべてにアクセス可能となりますがAPIによって二つのモードがあり可視範囲が変わります。

一つ目はWhole Teamと呼ばれるモードです。主に参照系APIが該当します。該当APIはAPIドキュメントにてAUTHENTICATIONに`Dropbox-API-Select-Admin (Whole Team)`と記載されています。Whole Teamモードではメンバーのプライベートファイルを含むすべてのファイルにアクセスすることができます。

もう一つはTeam Adminと呼ばれるモードです。主に更新系APIが該当します。該当APIはAPIドキュメントにてAUTHENTICATIONに`Dropbox-API-Select-Admin (Team Admin)`と記載されています。Team Adminモードではメンバーのプライベートファイルにはアクセスできません。

チーム管理者として実行する際の流れは大まかに次のようになると思います。

1. 実行するチーム管理者のメンバーIDを取得
2. チームフォルダ一覧やネームスペース一覧を取得
3. 各ネームスペースについて処理

まずチームメンバーの代理として実行する場合と同様に[teams/members/list](https://www.dropbox.com/developers/documentation/http/teams#team-members-list)でアカウント一覧を取得しチーム管理者のメンバーIDを取得します。アプリケーション用に専用のアカウントが用意できるようであればアプリケーションにあらかじめメンバーIDを設定しておくといった運用も可能でしょう。

次にチームフォルダ一覧やネームスペース一覧を取得します。チームフォルダ一覧は[team/team_folder/list](https://www.dropbox.com/developers/documentation/http/teams#team-team_folder-list)、ネームスペース一覧は[team/namespaces/list](https://www.dropbox.com/developers/documentation/http/teams#team-namespaces-list)で取得できます。ワークフロー処理などチームフォルダ内で処理が完結する場合は[team/team_folder/list](https://www.dropbox.com/developers/documentation/http/teams#team-team_folder-list)、チーム全体のファイルを監査するといった処理の場合はネームスペース一覧から処理を始めるとよいでしょう。

処理したいチームフォルダやネームスペースのネームスペースIDを取得します。なお、チームフォルダIDはそのままネームスペースIDとして利用できます。たとえば、[team/team_folder/list](https://www.dropbox.com/developers/documentation/http/teams#team-team_folder-list)のレスポンスが次のような場合、ネームスペースIDは123456789となります。

```json
{
    "team_folders": [
        {
            "team_folder_id": "123456789",
            "name": "Marketing",
            "status": {
                ".tag": "active"
            },
            "is_team_shared_dropbox": false
        }
    ],
    "cursor": "ZtkX9_EHj3x7PMkVuFIhwKYXEpwpLwyxp9vMKomUhllil9q7eWiAu",
    "has_more": false
}
```

# ネームスペースを使ってファイル一覧を取得する

パスとネームスペースの扱いについては以前の記事[Dropbox API: ネームスペースとパスの表現](/2018/01/22/dropbox-api-path/)にてご紹介いたしました。こちらでご紹介したとおり、`ns:ネームスペースID/パス`といった形式でパス指定すれば通常のファイル一覧取得と同じようにファイル一覧を取得することができます。

たとえばチームフォルダ(ネームスペースID = `123456789`)のトップフォルダからファイル一覧を取得するには次のように実行します。

```sh
curl -X POST https://api.dropboxapi.com/2/files/list_folder \
  --header 'Authorization: Bearer 認証トークン' \
  --header 'Content-Type: application/json' \
  --header 'Dropbox-Api-Select-Admin: チーム管理者のメンバーID' \
  --data '{"path":"ns:123456789"}'
```


