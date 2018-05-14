---
layout: post
title: 'Dropbox API: 認証・認可について'
tags:
 - dropbox
 - dropbox api
 - api
 - go
 - java
 - oauth
---

今回はDropbox APIの認証・認可についてかいつまんでご紹介します。Dropbox APIの認証・認可はOAuth2を利用しています。公式SDKの実装サンプルなどを参考にJavaとGoでの実装例をもとに紹介していきます。

[Dropbox APIの概要](/2017/12/08/dropbox-api-overview/)でもご紹介したとおり、実装前にはアプリケーションの登録が登録が必要です。登録がすんだらApp keyとApp Secretが得られますのでこれらを使いましょう。

# Java

SDKに[authorize](https://github.com/dropbox/dropbox-sdk-java/tree/master/examples/authorize)というサンプルがありますのでこれを参考にすると良いでしょう。
一般的なOAuthの流れと比べて変わったところは特にありません。[Dropbox SDK JavaのJavadoc](https://dropbox.github.io/dropbox-sdk-java/api-docs/v2.1.x/)に簡単な流れがのっているのでこちらを引用してご紹介します。

```java
// No Redirect Example
DbxRequestConfig requestConfig = new DbxRequestConfig("text-edit/0.1");
DbxAppInfo appInfo = DbxAppInfo.Reader.readFromFile("api.app");
DbxWebAuth auth = new DbxWebAuth(requestConfig, appInfo);
DbxWebAuth.Request authRequest = DbxWebAuth.newRequestBuilder()
    .withNoRedirect()
    .build();
String authorizeUrl = auth.authorize(authRequest);
System.out.println("1. Go to " + authorizeUrl);
System.out.println("2. Click \"Allow\" (you might have to log in first).");
System.out.println("3. Copy the authorization code.");
System.out.print("Enter the authorization code here: ");
String code = System.console().readLine();
if (code != null) {
    code = code.trim();
    DbxAuthFinish authFinish = webAuth.finishFromCode(code);
    DbxClientV2 client = new DbxClientV2(requestConfig, authFinish.getAccessToken());
    // APIを使った処理
}
```

この例ではApp Key・App Secretをファイルから読み込んでいます。api.appという名前で次のようなJSON形式ファイルを準備しておいてください。

```json
{
  "key"    : "App keyと置き換え",
  "secret" : "App secretと置き換え"
}
```
 
大まかな流れとしては次の通りです。

1. 設定情報などを準備 (`DbxRequestConfig`, `DbxAppInfo`)
2. リクエストを準備
3. 認証URLを生成し、ユーザーに認可をもとめる
4. コードを受け取りトークンを生成
5. `DbxClientV2`インスタンスを作成して各種APIコール

設定できるオプションとしては、`DbxRequestConfig`にはエラー時の再試行回数などがあります。

# Go

個人的にGoで実装する際には非公式SDKの[dropbox-sdk-go-unofficial](https://github.com/dropbox/dropbox-sdk-go-unofficial)を使っています。処理の流れは基本的にJavaでの実装と同じです。OAuth認証・認可処理自体はSDKとは関係なく[golang.org/x/oauth2](https://godoc.org/golang.org/x/oauth2)を利用しています。

```go
authCfg := &oauth2.Config{
	ClientID:     "App keyと置き換え",
	ClientSecret: "App Secretと置き換え",
	Scopes:       []string{},
	Endpoint: oauth2.Endpoint{
		AuthURL:  "https://www.dropbox.com/oauth2/authorize",
		TokenURL: "https://api.dropboxapi.com/oauth2/token",
	},
}
authorizeUrl := authCfg.AuthCodeURL(
	"csrf-stateの文字列",
	oauth2.SetAuthURLParam("response_type", "code"),
)
fmt.Println("1. Go to " + authorizeUrl);
fmt.Println("2. Click \"Allow\" (you might have to log in first).");
fmt.Println("3. Copy the authorization code.");
fmt.Print("Enter the authorization code here: ");
var code string
fmt.Scan(&code)
token, err := authCfg.Exchange(context.Background(), code)
if err == nil {
	dropboxCfg := dropbox.Config{
		Token: token.AccessToken,
	}
	client := files.New(dropboxCfg)
	// ...
}
```

# 認証時のロール

ちょっと変わったところとしてはリクエストを準備する際のロールについて紹介しておこうと思います。OAuthではscopesというパラメータを使ってアプリケーションに対し、認可する範囲を設定します。Dropboxでは[Dropbox APIの概要](/2017/12/08/dropbox-api-overview/)にてご紹介したとおり、アプリケーション登録時にアクセス権限を設定します。

このため、scopesパラメータを使わない場合も多いのですが一つだけ指定できるものがあります。

Dropboxでは個人向けDropbox (Dropbox Basic、Plus、Professional)アカウントと、Dropbox Businessアカウントを[リンクすることでパソコン、スマートフォン、タブレットなどのアプリケーションで同時に二つのアカウントを利用できる機能](https://www.dropbox.com/help/business/connect-personal-work-account)があります。

このときに、アプリケーションの接続を会社のアカウントのみに限定したいとか、逆に個人向けのみに限定したいといった指定にscopesパラメータを利用します。

Javaの場合は`DbxWebAuth.Request.Builder`で `withRequireRole()`メソッドへ[ROLE_PERSONAL](https://dropbox.github.io/dropbox-sdk-java/api-docs/v2.1.x/com/dropbox/core/DbxWebAuth.html#ROLE_PERSONAL)または[ROLE_WORK](https://dropbox.github.io/dropbox-sdk-java/api-docs/v2.1.x/com/dropbox/core/DbxWebAuth.html#ROLE_WORK)を指定します。

# トークンの無効化

一定の処理が終わって、ログアウトに相当するトークンの無効化処理をしたい場合の処理です。これは、User EndpointsとBusiness Endpointsで若干変わります。

## User Endpointsの場合

User Endpointsの場合は[auth/token/revoke](https://www.dropbox.com/developers/documentation/http/documentation#auth-token-revoke)にAccess Tokenを渡すだけです。

## Business Endpointsの場合

Business Endpointsでは`auth/token/revoke`に直接相当するAPIがありません。このためチーム管理者が[管理者コンソールよりチーム向けアプリケーションを無効化する必要があります](https://www.dropbox.com/help/business/business-api#manage)。

