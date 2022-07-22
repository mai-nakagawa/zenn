---
title: "DeployGateにiOSアプリをデプロイできるようにする" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["ios", "deploygate"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これは何？

`dg` コマンド (DeployGat CLI) で iOS アプリをデプロイしようとしたら、以下のようなエラーが出てデプロイできない。
```
no applicable iphone distribution or apple distribute certificate found on your mac
```

かなり解決に時間かかったのと、DeployGate や Apple の公式ドキュメントでピンポイントの情報が見つけられなかったので、誰かの役に立つかと思い、手順を残します。

なお M1 Mac における `dg` コマンドのインストールで困ったときは [DeployGate がサクッとM1に入らなかった話](https://zenn.dev/misato/articles/1267007983f139) が参考になるかも知れません。

# 手順

## 1. Apple Developer Programに登録する

[Apple Developer Programに登録する](https://developer.apple.com/jp/support/app-account/)

## 2. Apple ID を Xcode に登録する

Xcode の `Preferences` から `Accounts` タブを開き、[Apple Developer Programに登録](https://developer.apple.com/jp/support/app-account/)した Apple ID を登録する

## 3. Certificate Signing Request

1. `キーチェーンアクセス`アプリで`認証局に証明書を要求...`を選択する:
   ![Keychain Access Request](/images/keychain_access_request.png)
1. `証明書情報`の画面が開くので以下のように入力して、Certificate Signing Request(証明書署名要求)ファイルをローカルに保存する
   - `メールアドレス`: [Apple Developer Programに登録](https://developer.apple.com/jp/support/app-account/)した Apple ID
   - `通称`: 証明書に利用したFQDN (例: `example.com`)
   - `CAのメールアドレス`: 空欄
   - `要求の処理`: `ディスクの保存`
   ![Certificate Information](/images/certificate_information.png)

## 4. Distribution Certificate

1. [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list) を開く。
   - このページへのアクセス権がない場合は、管理者に依頼して付与してもらう。[Roles for the Apple Developer Program](https://developer.apple.com/support/roles/) によると、`Account Holder` と `Admin` には権限があるが、`App Manager` にはデフォルトでは権限がない。
   ![Roles for the Apple Developer Program](/images/roles_for_apple_developer_program.png)
1. `Certificates`を選択して`+`アイコンをクリックする。
   ![New Certificate](/images/new_certificate.png)
1. `Software`で`Apple Distribution`を選択して`Continue`で次に進む。
   ![Apple Distribution](/images/apple_distribution.png)
1. 前セクションで作成したCertificate Signing Requestファイルをアップロードして`Continue`で次に進む
   ![Upload Certificate Signing Request](/images/upload_certificate_signing_request.png)
1. 次の画面で Apple Distribution 証明書がダウンロードできるので、ローカルにダウンロードして、ダブルクリックして`キーチェーンアクセス`アプリにインストールする

## 5. Identifiers

1. [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list) を開く。
1. `Identifiers`を選択して`+`アイコンをクリックする。
1. `App IDs`を選択して、`Continue`をクリックする。
1. `App`または`App Clip`を選択する画面が出るので、`App`を選んで`Continue`。
1. `Description`と`Bundle ID`を入力する。`Bundle ID`には Xcode の`Bundle Identifier`で設定した値を入力する。

## 6. Devices

1. [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list) を開く。
1. `Devices`を選択して`+`アイコンをクリックする。
1. DeployGate にデプロイした iOS アプリをインストールしたい iOS デバイスの UDID を登録する。

## 7. Profiles

1. [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/certificates/list) を開く。
1. `Profiles`を選択して`+`アイコンをクリックする。
1. `Ad Hoc`を選択して`Continue`。
1. 上の`Identifiers`セクションで作成した`App ID`を選択して`Continue`。
1. 上の`Certificates`セクションで作成した`Certificates`を選択して`Continue`。
1. Provisioning Profile が作成されるので、ダウンロードして、ダブルクリックして`キーチェーンアクセス`アプリにインストールする

## 8. dg deploy を実行する

`dg deploy`を試してみる。今度はうまく行きました！

# 今後の展望

次は[FlutterアプリをGitHub ActionsでDeployGateのデプロイを自動化！](https://zenn.dev/ss_shiraki/articles/53fb6b5d3a2d69)を参考に、GitHubリポジトリからの自動デプロイに挑戦したいと思います。
