---
title: 用 React 開發具有完整 CI/CD 流程的 Chrome Extension
date: 2017-05-21 23:32
categories: 教學
tags:
  - Chrome Extension
  - React
  - CI/CD
  - travis
  - JavaScript
---

![](https://source.unsplash.com/ObweQkF5w30/700x393)

## 前言

身為一個前端工程師，在工作上遇到一些需要反覆操作的流程時，總會想要用一些方法來優化它。像我自己也在公司開發了一些 [CLI 及 Chrome Extension] 來加速日常的開發。然而在開發了這些工具的同時，我卻也不經意的為自己引入了新的*反覆操作*，如果可以把測試、過版、壓檔、上傳、發佈……等步驟全部自動化，我就可以更專注的在新功能上的開發上。

底下將以開發一個新的 Chrome Extension 為例，利用 [Travis CI] 及 [Chrome Web Store Publish API] 做到完整的 CI/CD 整合。

[CLI 及 Chrome Extension]: https://kaddopur.github.io/projects
[Travis CI]: https://travis-ci.org/
[Chrome Web Store Publish API]: https://developer.chrome.com/webstore/using_webstore_api

## 開發環境
```
$ node -v
v6.3.1
$ yarn -v
yarn install v0.24.5
```

## 流程
> 1. 建一個新的 React 專案
> 2. Chrome Extension 設定
> 3. 基本 CI 設定
> 4. 自動化打包及發布 CD 設定
> 5. 發布 Chrome Extension

<!-- more -->

### 建一個新的 React 專案

這個步驟很簡單，只有一行
```
$ yarn create react-app <EXTENSION_NAME>
```
本篇教學所用的`EXTENSION_NAME`是`my-ext`，所以我用
```
$ yarn create react-app my-ext
```
它會幫你開一個新的 React 專案`my-ext`。

### Chrome Extension 設定

這個新的 React 專案現在還不是一個合法的 Chrome Extension，我們需要補上正確的設定。

```json
// public/manifest.json
{
  "short_name": "React App",
  "name": "Create React App Sample",
  "icons": [
    {
      "src": "favicon.ico",
      "sizes": "192x192",
      "type": "image/png"
    }
  ],
  "start_url": "./index.html",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff"
}
```

目前的`manifest.json`是 PWA 的設定，而不是我們所想要的 Chrome Extension 設定，所以將其改寫為

```json
// public/manifest.json
{
  "manifest_version": 2,
  "name": "my-ext",
  "description": "...",
  "version": "0.1.0",
  "browser_action": {
    "default_popup": "index.html"
  }
}
```

再來，`manifest.json`中的 version 應該要與`package.json`中的 version 互相對應。為了不要每次過版都要手動改兩邊的版本，我用`version-everything`來幫我處理版號問題

```
$ yarn add -D version-everything
```

首先安裝它，再更改`package.json`

```json
// package.json
{
  ...
  "scripts" : {
    ...
    "version": "version-everything && git add -u"
  },
  "version_files": [
    "public/manifest.json"
  ]
}
```

如此一來，我們在用`yarn version`的指令跳版號時，會觸發 version 這個 hook 以達到版號同步的效果。

```
$ yarn build
```

接下來我們在 Chrome 上看看`my-ext`能不能正確地被載入。`yarn build`這個指令在打包時會把在`./public`底下的檔案也一併複製到`./build`底下，如此一來，我們即可以在 chrome 中用[載入未封裝擴充功能][load_extension]，裝載`./build`

載入後可以看到瀏覽器右上角出現一個按鈕，按下去之後會彈現一個視窗如下圖
![http://i.imgur.com/viruNna.png](http://i.imgur.com/viruNna.png)

恭喜，我們的 Chrome Extension 看起來設定對了。

[load_extension]: https://developer.chrome.com/extensions/getstarted#unpacked

### 基本 CI 設定

這步驟主要是設定好 [Travis CI] 的串接，首先我們必須要 push 我們的專案到 [github repo]，再來在 Travis 中打開此 repo 的開關 (在這裡是 kaddopur/my-ext)，最後設置`.travis.yml`如下
```yml
# .travis.yml
language: node_js
node_js:
  - "6"
cache: yarn
```
這邊我們用的是 `node@6`，再來我們也打開了 yarn caching。將`.travis.yml` commit 之後 push 到 github，你會看到 Travis 開始跑起來了。


[github repo]: https://github.com/kaddopur/my-ext

### 自動化打包及發布 CD 設定

我們將會利用 Travis 的 `after_success` hook 來自動觸發上傳及發布，但在上傳之前，我們需要準備好 Extension 的 zip 檔。在`yarn build`之後，我用`zip-folder`來幫我產生壓縮檔

```
$ yarn add -D zip-folder
```

```js
// scripts/zip.js
const zipFolder = require('zip-folder');

zipFolder(`${process.cwd()}/build`, `${process.cwd()}/bundle.zip`, err => {
  if (err) {
    console.log('oh no!', err);
  } else {
    console.log('EXCELLENT');
  }
});
```

它會把`/build`資料夾壓成`bundle.zip`

再來我會使用`webstore-upload`來上傳壓縮檔與發布

```
$ yarn add -D webstore-upload
```

```js
// scripts/upload.js
const webstore_upload = require('webstore-upload');
const uploadOptions = {
    accounts: {
        default: {
            publish: true,
            client_id: process.env.CLIENT_ID,
            client_secret: process.env.CLIENT_SECRET,
            refresh_token: process.env.REFRESH_TOKEN
        }
    },
    extensions: {
        my_ext: {
            appID: process.env.APP_ID,
            zip: `${process.cwd()}/bundle.zip`
        }
    },
    uploadExtensions: ['my_ext']
};

webstore_upload(uploadOptions, 'default')
    .then(function(result) {
        console.log(result);
        // do somethings nice
        return 'yay';
    })
    .catch(function(err) {
        console.error(err);
    });
```

我們要提供`client_id`,`client_secret`,`refresh_token`及 Extension 的`appID`方可使用。特別提醒一下，我把這些值設為 Travis 的環境變數並在執行時取得，而非 commit 在程式碼中。

這些值怎麼取得的
- 前三項怎麼產生請看這 [How to generate Google API keys]
- `appID`請先手動上傳你的 Extension 一次後，在[開發人員資訊主頁]的右邊每個 Extension 的**更多資訊**中，可以找到**商品ID**即為你的`appID`。

最後，我們把`after_success` hook 設定好
```json
// package.json
{
  ...
  "scripts": {
    ...
    "zip": "node scripts/zip.js",
    "upload": "node scripts/upload.js"
  }
}
```

```yml
# .travis.yml
after_success:
  - yarn build
  - yarn zip
  - yarn upload
```

就大功告成啦！

[How to generate Google API keys]: https://github.com/DrewML/chrome-webstore-upload/blob/master/How%20to%20generate%20Google%20API%20keys.md 

[開發人員資訊主頁]: https://chrome.google.com/webstore/developer/dashboard?hl=zh-TW

### 發布 Chrome Extension

只要 repo 上的 master branch 有新的 commit，Travis 都會幫我們跑 CI/CD 流程。需要特別注意的是，Chrome Web Store 無法上傳和同版本的 Extension，所以 merge 時請記得跳個版。

```
$ yarn version
yarn version v0.24.5
info Current version: 0.1.0
question New version: 0.1.1 // 輸入你的新版號
info New version: 0.1.1
$ version-everything && git add -u // 更新 manifest.json 中的 version
Loading package.json
Current version is "0.1.1"
Updated public/manifest.json from 0.1.0 to 0.1.1
✨  Done in 5.21s.
```

直接`git push`或是你要先發 PR 再 merge 也行。之後在你的[開發人員資訊主頁]就可以看到版本更新了！

## 後記
程式碼放在 https://github.com/kaddopur/my-ext ，如果過程中有任何問題或討論也歡迎在底下留言。

## 修訂
2017-06-03 - 改用`zip-folder`產生壓縮檔
