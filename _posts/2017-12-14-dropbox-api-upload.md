---
layout: post
title: 'Dropbox API: ファイルのアップロード'
tags:
 - dropbox
 - dropbox api
 - go
---

今回はDropbox APIを使ってファイルをアップロードをする流れをご紹介します。ファイルアップロード処理は[/files/upload](https://www.dropbox.com/developers/documentation/http/documentation#files-upload)を使えば良いのですが、ファイルサイズが大きい場合には一つのリクエストで処理するのではなく分割してアップロードすることが求められます。

分割アップロードする場合には、次のように3つの手順をたどります。

1. 分割した最初のチャンクをアップロード (戻り値としてセッションIDが得られます) - [/files/upload_session/start](https://www.dropbox.com/developers/documentation/http/documentation#files-upload_session-start)
2. セッションIDをもとに最後の一つ手前チャンクまでを追記アップロードします - [/files/upload_session/append_v2](https://www.dropbox.com/developers/documentation/http/documentation#files-upload_session-append_v2)
3. セッションIDと最後のチャンクをアップロードします - [/files/upload_session/finish](https://www.dropbox.com/developers/documentation/http/documentation#files-upload_session-finish)

# 分割の閾値

分割するかどうかの閾値はAPIドキュメントに下記のように記載があります。

> A single request should not upload more than 150 MB.

150MB以上であれば分割してアップロードすべし。とのことなのですが、気になるのはこの1MBが 1,000,000バイトなのか、1,048,576バイトなのか。です。
気になるので試してみたところ次の通りでした。

* 150,000,000バイト ... アップロード成功
* 157,286,399バイト (150 * 1,048,576 - 1)... アップロード成功
* 157,286,400バイト (150 * 1,048,576) ... アップロード成功
* 160,000,000バイト ... アップロード成功

予想外のことですがとりあえず150MBを軽く超えてもエラーなど出ず、アップロードができるようです。
ただ、あまり一度にアップロードするサイズが大きくなりすぎると、途中で通信エラーが出た場合に再送するならばロスが大きくなるのでそこそこに分割した方が良いでしょう。

# Goでの実装例

前回と同様に[dropbox-sdk-go-unofficial](https://github.com/dropbox/dropbox-sdk-go-unofficial)を使った実装例です。まずはソースをご覧ください。

```go
uploadSrc := "data.zip"
// 分割してアップロードするかどうかの閾値
var chunkedUploadThreshold, chunkSize int64
chunkedUploadThreshold = 150 * 1048576
chunkSize = chunkedUploadThreshold
config := dropbox.Config{
  Token: "トークン文字列",
}
client := files.New(config)
// ローカルファイルの情報取得
info, err := os.Lstat(uploadSrc)
if err != nil {
  return err
}
f, err := os.Open(uploadSrc)
if err != nil {
  return err
}
defer f.Close()
ci := files.NewCommitInfo("/アップロード先/data.zip")
// ファイルの日付、UTCで秒未満の単位は丸める
ci.ClientModified = info.ModTime().UTC().Round(time.Second)
if info.Size() < chunkedUploadThreshold {
  _, err = client.Upload(ci, f)
  return err
} else {
  // セッションを開始、最初のチャンクサイズ分だけアップロード
  r := io.LimitReader(f, chunkSize)
  s, err := client.UploadSessionStart(files.NewUploadSessionStartArg(), r)
  if err != nil {
    return err
  }
  // 最後から一つ手前のチャンクまで分割アップロード
  var uploaded int64 // アップロード済みサイズ
  uploaded = chunkSize
  for (info.Size() - uploaded) > chunkSize {
    cursor := files.NewUploadSessionCursor(s.SessionId, uint64(uploaded))
    arg := files.NewUploadSessionAppendArg(cursor)
    r = io.LimitReader(f, chunkSize)
    err = client.UploadSessionAppendV2(arg, r)
    if err != nil {
      return err
    }
    uploaded += chunkSize
  }
  // 最後のチャンクをアップロード
  cursor := files.NewUploadSessionCursor(s.SessionId, uint64(uploaded))
  arg := files.NewUploadSessionFinishArg(cursor, ci)
  _, err = client.UploadSessionFinish(arg, f)
  return err
}
```

## アップロード先

`CommitInfo`構造体にパスやファイルの日付などを設定します。アップロード先パスですが、アップロード先ディレクトリ名だけでなく、アップロード先ファイル名も含めたパスを指定します。

`autorename`というパラメータがありますが、これを`true`にすると同じパスにすでにファイルがある場合は新しくアップロードするファイルが`data (1).zip`のように重複しないファイル名でアップロードされます。

## ファイル更新日付

ファイル更新日付を指定する場合には`client_modified`パラメータを指定します。指定しない場合にはDropboxへファイルが保存された際のサーバ日時になります。指定はISO 8601フォーマットで、UTCとします。

Goの例ではUTCに変換した上で、下記のように秒以下を丸めています。
```go
ci.ClientModified = info.ModTime().UTC().Round(time.Second)
```

## 分割アップロード

Goの場合には分割アップロードに便利な`io.LimitReader`というクラスがありますのでこれを利用するといいでしょう。分割したいチャンクサイズを指定するとそこでEOFになってくれます。

[/files/upload_session/start](https://www.dropbox.com/developers/documentation/http/documentation#files-upload_session-start)で最初のチャンクをアップロードしたら、セッションIDが得られます。

これを使って、以降は[/files/upload_session/append_v2](https://www.dropbox.com/developers/documentation/http/documentation#files-upload_session-append_v2)を呼び出してチャンクをアップロードしていきます。なお、ファイルは0バイト目から連続している必要があり、cursorで飛び飛びのオフセットを指定した場合には`incorrect_offset`エラーとなります。

このoffsetパラメータの目的はAPIドキュメントにあるとおり、通信エラーなどで重複送信や部分的なロスが発生した場合にデータの整合性を検出するためです。

> We use this to make sure upload data isn't lost or duplicated in the event of a network error.

