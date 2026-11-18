# GitHub Pages への公開手順

このリポジトリは、Passkey デモアプリのコールバック先として使う静的サイトです。
GitHub Pages に公開すると、次のような HTTPS URL でアクセスできます。

```text
https://<GitHubユーザー名>.github.io/auth/callback.html
```

このサイトはコールバック用の HTML と、Android App Links / iOS Universal Links の
ドメイン確認ファイルを配信します。認証処理やコールバック値のサーバー側検証は行いません。

## 1. 自分の GitHub アカウントにリポジトリを用意する

1. GitHub にログインし、このリポジトリを自分のアカウントに Fork します。
2. Fork したリポジトリの **Settings > General > Repository name** を開き、
   リポジトリ名を `<自分のGitHubユーザー名>.github.io` に変更します。
   例: ユーザー名が `octocat` の場合は `octocat.github.io`。
3. リポジトリが **Public** であることを確認します。

GitHub Pages のユーザーサイトでは、リポジトリ名が GitHub ユーザー名と一致している
必要があります。通常のプロジェクト名のリポジトリを使うと URL にリポジトリ名が入り、
ここに記載したアプリ設定とは異なる URL になります。

## 2. GitHub Pages を有効にする

1. リポジトリの **Settings > Pages** を開きます。
2. **Build and deployment** の **Source** で `Deploy from a branch` を選びます。
3. Branch を `main`、フォルダーを `/(root)` にして **Save** します。
4. Pages の公開が完了するまで待ちます。公開先は
   `https://<自分のGitHubユーザー名>.github.io/` です。

公開後、次の URL が開けることを確認してください。

```text
https://<自分のGitHubユーザー名>.github.io/
https://<自分のGitHubユーザー名>.github.io/auth/callback.html
https://<自分のGitHubユーザー名>.github.io/.well-known/assetlinks.json
https://<自分のGitHubユーザー名>.github.io/.well-known/apple-app-site-association
```

リポジトリ直下の `.nojekyll` は削除しないでください。`.well-known` のようなドットで
始まるディレクトリをそのまま公開するために必要です。

## 3. Android / iOS 用の関連付けファイルを更新する

現在の `assetlinks.json` と `apple-app-site-association` には、このサンプルアプリ用の
識別情報が設定されています。別のアプリや署名鍵で使う場合は、公開前に自身の値へ
置き換えてください。

### Android App Links

`.well-known/assetlinks.json` の以下を、対象アプリの値にします。

- `package_name`: Android アプリの application ID
- `sha256_cert_fingerprints`: アプリの APK に使う署名証明書の SHA-256 フィンガープリント

デバッグ用とリリース用で署名鍵が違う場合は、それぞれのフィンガープリントを設定します。
既存の値をそのまま使えるのは、同じ application ID と署名鍵のアプリに限ります。

### iOS Universal Links

`.well-known/apple-app-site-association` の `appIDs` を、`<Apple Developer Team ID>.<Bundle ID>`
形式で設定します。`components` のパスはコールバックのパス
(`/auth/callback.html`) に合わせてください。AASA ファイル名に拡張子は付けません。

Xcode 側でも対象アプリの **Signing & Capabilities > Associated Domains** に
`applinks:<自分のGitHubユーザー名>.github.io` を設定します。

## 4. デモアプリのコールバック URL を合わせる

GitHub Pages の URL を変えた場合、アプリ側が古いドメインを参照し続けないよう、
デモアプリの設定も更新します。

- Android: `app/src/main/AndroidManifest.xml` の host と
  `MainActivity.kt` のコールバック URL
- iOS: `PasskeyRedirectDemo.entitlements` の Associated Domains と
  `PasskeyRedirectViewModel.swift` のコールバック URL

ログイン開始先の設定や各 README / セットアップ手順に古いドメインが記載されている場合も、
必要に応じて新しい URL に置き換えてください。Android/iOS の関連付けを使わない場合でも、
認証先へ渡す `redirect_uri` は実際に公開した URL と完全に一致させます。

## 5. 公開結果を確認する

ブラウザーで各ファイルを開き、404 ではなくファイルの内容が表示されることを確認します。
コマンドで確認する場合:

```bash
curl -i https://<自分のGitHubユーザー名>.github.io/.well-known/assetlinks.json
curl -i https://<自分のGitHubユーザー名>.github.io/.well-known/apple-app-site-association
```

どちらも `200` で取得でき、HTML のエラーページではなく正しい JSON が返る必要があります。
App Links / Universal Links の動作確認には、OS側のアプリ設定と署名情報も一致している必要が
あります。公開 URL が表示できることだけではアプリへのリンク起動は保証されません。
