---
title: "Bot Builder SDK v4 お試し"
date: 2018-05-10T18:56:48+09:00
thumbnail: "img/bot-builder-sdk-v4/bot-test-in-web-chat-b.png"
draft: false
tags: [ "bot", "Node.js" ]
---

Bot Builder SDK v4
===================

背景
----

2018/5/7 (現地時間) に Microsoft Build 2018 にて Bot Builder SDK の v4 のプレビュー版が発表されました。

なので、Node.js で試しにボットを作成してみました。

<!--more-->

環境
----

- macOS High Sierra
- Node.js 8.11.1 LTS
- generator-botbuilder 4.0.0
- botbuilder 4.0.0-preview1.1
- botbuilder-azure 4.0.0-preview1.2

ボットの作成
-----------

### 準備

プロジェクト用のディレクトリを作成し、移動します。

```bash
mkdir bot-builder-4-example
cd bot-builder-4-example
```

Yeoman と Yeoman の Bot Builder SDK 用のジェネレーターのプレビュー版をインストールします。

```bash
npm install --save-dev yo
npm install --save-dev generator-botbuilder@preview
```

`./node_modules/.bin` をパスに設定します。

```bash
export PATH="$PATH:./node_modules/.bin"
```

### プロジェクト作成

プロジェクトを生成します。

```
yo botbuilder
```

What 's the name of your bot? で、現在のディレクトリと同じ名前を設定すると、
現在のディレクトリ内にプロジェクトが作成されます。
そうでない場合は、指定した bot 名でサブディレクトリが作成されます。

言語として JavaScript を選びました。

`package.json` に次を追記します。

```json
{
  "scripts": {
    "start": "node app.js"
  },
}
```

### ESLint

JavaScript の構文チェックのため ESLint の設定をしておきます。

インストールします。

```bash
npm install --save-dev eslint eslint-plugin-node
```

`package.json` に Node.js のバージョンを設定します。

```json
{
  "engines": {
    "node": ">=6.0.0"
  }
}
```

`.eslintrc.json` を作成します。

```json
{
  "extends": ["eslint:recommended", "plugin:node/recommended"],
  "plugins": ["node"],
  "env": {
    "es6": true,
    "node": true
  },
  "rules": {
    "node/exports-style": "error",
    "no-console": "warn"
  }
}
```

### Bot Framework Emulator インストール

次の場所から Bot Framework Emulator v4 をダウンロードしインストールします。

<https://github.com/Microsoft/BotFramework-Emulator/releases>

### ローカル実行

サーバーを起動します。

```bash
npm start
```

Bot Framework Emulator を起動します。

Open Bot をクリックします。そして、作成したプロジェクトの `bot-builder-4-example.bot` を
選択します。

ローカルサーバーと Bot Framework Emulator で動作を確認することができます。
テンプレートで Echo を選択しているので、こちらがポストした内容を表示してくれます。

Bot Service 作成
------------------

[Microsoft Azure](https://portal.azure.com/) にアクセスします。

*すべてのサービス* をクリックし、*Bot Service* クリックします。

*追加* > *Web App Bot* を選択し、作成をクリックします。

次の設定をします。

- ボット名: 任意の名前
- リソースグループ: 新規 任意の名前
- 場所: 任意の場所
- 価格レベル: F0
- アプリ名: 任意の名前
- ボットテンプレート: Basic (NodeJS)

作成をクリックします。

### 環境変数名を変更

ジェネレーターで生成したソースコード上の環境変数名が、Bot Service のアプリケーション環境変数名と異なっているので、ソースコードを修正します。

```javascript
const adapter = new BotFrameworkAdapter({
  appId: process.env.MICROSOFT_APP_ID,
  appPassword: process.env.MICROSOFT_APP_PASSWORD
});
```

```javascript
const adapter = new BotFrameworkAdapter({
  appId: process.env.MicrosoftAppId,
  appPassword: process.env.MicrosoftAppPassword
});
```

### Azure Web App へローカルの git からデプロイ

[Microsoft Azure](https://portal.azure.com/) にアクセスします。

Bot Service を選択し、作成した Bot Service を選択します。

*App Service 設定* > *すべての App Service 設定* をクリックします。

*デプロイメント* > *デプロイメント オプション* > *セットアップ* をクリックします。

ローカル git を選択します。

Azure Web App の概要から *デプロイ先の Git クローン URL* をコピーします。

プロジェクトのルートディレクトリで git の初期化を行います。

```bash
git init
```

コピーした Git クローン URL を、リモートリポジトリに azure という名前で登録します。

*{git-clone-url}* を上記 *デプロイ先の Git クローン URL* に置き換えます。

```bash
git remote add azure {git-clone-url}
```

コミットします。

```bash
git add -A
git commit -m "Initial commit"
```

azure リポジトリにプッシュします。

```bash
git push azure master
```

### Web チャットでテスト

Azure Portal の Bot Service に用意されている Web チャットでテスト機能を使って
動作確認をします。

[Microsoft Azure](https://portal.azure.com/) にアクセスします。

Bot Service を選択し、作成した Bot Service を選択します。

*ボット管理* > *Web チャットでテスト* クリックします。

デプロイしたボットと会話できることを確認します。

{{% img src="img/bot-builder-sdk-v4/bot-test-in-web-chat.png" h="400" %}}

### ログの確認

デプロイしたボットと会話した際に、エラーとなった場合は、ログを確認します。

次の URL の DebugConsole にてログファイルを確認することができます。

<https://{your-app-name}.scm.azurewebsites.net/DebugConsole>

アプリケーションログは、 `/LogFiles/Application/index.html` にあります。

<https://{your-app-name}.scm.azurewebsites.net/api/vfs/LogFiles/Application/index.html>

### 状態を Azure Table Storage に保存

ジェネレーターで生成したコードは、状態を `MemoryStorage` に保存しています。
そのため、アプリケーションを再起動すると状態がリセットされてしまいます。

そこで状態を Azure Table Storage に保存するようにします。

まずは、botbuilder-azure をインストールします。

```bash
npm install --save botbuilder-azure@preview
```

`app.js` のストレージの設定を修正します。

require を追記します。

```javascript
const { TableStorage } = require('botbuilder-azure');
```

conversationState 生成のコードも、次のものから、

```javascript
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

次の通り修正します。

ローカル環境では、引き続き MemoryStorage を利用するようにしています。

```javascript
const development = (process.env.NODE_ENV === 'development');
let conversationState = null;
if (development) {
  conversationState = new ConversationState(new MemoryStorage());
  adapter.use(conversationState);
} else {
  const tableName = 'botdata';
  const azureStorage = new TableStorage({
    tableName,
    storageAccountOrConnectionString: process.env.AzureWebJobsStorage
  });
  conversationState = new ConversationState(azureStorage);
  adapter.use(conversationState);
}
```

ローカル環境で開発環境として動かす場合は次のよう実行します。

```bash
NODE_ENV=development npm start
```

Azure Web App にプッシュします。

```bash
git add -A
git commit -m "Use Azure Table Storage"
git push azure master
```

Bot Service を選択し、作成した Bot Service を選択します。

*App Service 設定* > すべての *App Service 設定* をクリックします。

*概要* > *再起動* をクリックし、アプリを再起動します。

再起動しても `state.count` がリセットされずインクリメントされることが確認できます。

コード
-----

今回実装したコードは次の場所に置いてあります。

<https://github.com/rysh25/bot-builder-4-example>

参考
----

[Create a bot using Bot Builder SDK for JavaScript - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)

次の場所に bot のサンプルコードがあります。

[botbuilder-js/samples at master · Microsoft/botbuilder-js · GitHub](https://github.com/Microsoft/botbuilder-js/tree/master/samples)
