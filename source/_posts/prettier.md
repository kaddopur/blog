---
title: 使用 prettier 自動調整 JavaScript 樣式
tags:
  - JavaScript
  - prettier
  - Facebook
  - tool
categories: 新知
date: 2017-05-10 17:41:00
---


## 前言

`prettier` 是一款 JavaScript 的樣式處理工具，它類似 golang 的 `gofmt` 可以自動排版你的程式。除了 Vanilla JS 之外，尚支援了 ES6, jsx, Flow 以及正在開發中的 TypeScript。它是由 [vjeux (Christopher Chedeau)][vjeux]  開發，在 [github][prettier_github] 上面約有 10,000 個 stars。

`prettier` 從今年一月的第一個 commit 開始到現在快半年，不過到它真正引起我的興趣，是從上週 (5/3) 釋出的 `v1.3.0` 開始。至於這個版本有什麼特別的功能嗎？非也。主要是因為這個版本有公開它在 Facebook 內部各專案使用 `prettier` 的情形。

<!-- more -->
> The first projects to adopt prettier were Jest, React and immutable-js. Those are small codebases in the order of hundreds of files that have their own infrastructure. There are a handful of people working on them full time.<br />
> Then, Oculus and Nuclide converted their codebase over. The scale is bigger with a few thousands of files and tens of full time contributors but looks pretty similar to the first projects. The conversions went in one big codemod and that's it.

可以看到 Facebook 的專案一個接一個開始用 `prettier` 來規範及管理他們的 JavaScript 樣式，而 `prettier` 和我們平常在用的 `eslint` 或是其他的 lint 工具相比之下孰優孰劣呢？

## 優點

1. **一致性 Consistency**
同一種表達式對於不同的人可能會有不同的寫法，使用 `prettier` 可以輕易保持代碼中樣式的一致式，避免個人習慣樣式的不同。

2. **可教性 Teachability**
經過 `prettier` 調整格式之後，你可以知道 JavaScript 引擎是怎麼解析你所寫的代碼，進而幫助你養成良好的撰寫習慣來避免一些語意上的錯誤。例如：
  ```js
  const x = value1 && value2 || value3 && value4;
  ```
  會幫你加上括號
  ```js
  const x = (value1 && value2) || (value3 && value4);
  ```
3. **自由度 Freedom**
只需要專心在寫代碼這件事就好了，不用刻意去手動調整樣式。在抽函數的時候也蠻有用的。

## 如何使用

`prettier` 提供了 CLI 方便你轉換檔案。你也可以將 `prettier` 設定在專案的工作流程中，在 commit 時自動排版並簽入至你的 repo。或是你也可以將 `prettier` 與你最愛的編輯器綁定，讓你在存檔時可以自動排版並看到結果。以下會分別介紹。

可以局部安裝到你的專案
```
$ yarn add prettier --dev
```
或是全局安裝
```
$ yarn global add prettier
```
### CLI

使用方式如下，可以選擇想要轉換的檔案。
```
$ prettier [opts] [filename ...]
```
實際上使用的情形可能會像這樣
```
$ prettier --single-quote --write "{app,__{tests,mocks}__}/**/*.js"
```
想要知道有什麼參數可以下，可以輸入
```
$ prettier
Usage: prettier [opts] [filename ...]

Available options:
  --write                  Edit the file in-place. (Beware!)
  --list-different or -l   Print filenames of files that are different from Prettier formatting.
  --stdin                  Read input from stdin.
  --print-width <int>      Specify the length of line th
  ...
```
會列出所有可以用的選項

### Node

將 `husky` 和 `lint-staged` 加到專案的 devDependencies
```
$ yarn add lint-staged husky --dev
```
在 `package.json` 中填加以下 config
```json
{
  "scripts": {
    "precommit": "lint-staged"
  },
  "lint-staged": {
    "*.js": [
      "prettier --write",
      "git add"
    ]
  }
}
```
`husky` 會讓專案在 git commit 時執行 "precommit" 這個命令。而在 `lint-staged` 中，會檢查所有被 stage 的檔案，如果有符合特定檔名，就會再運行指定的命令。

在上面的例子中，我們會在 git commit 之前，對所有被 stage 的 JavaScript 檔案運行 `prettier` 後，再重新 stage 更新後的檔案，以確保所有進入 repo 的代碼的樣式正確。

### Editor

`prettier` 對大部分的編輯器都有對應的工具，這邊以我的編輯器 VS Code 為例。

去 MartketPlace 下載 [Prettier - JavaScript formatter][vscode_ext]。安裝完成後，可以用 `CMD + Shift + P` 來重新排版整個檔案，或是用 `CMD + Shift + P` 重排所選擇的區域。

想要調整 `prettier` 所使用的排版選項，可以在 `User Settings` 來全局更改所想要的 `prettier` 參數，或是也可以用 `Workspace Settings` 來按照不同專案設定不同選項。

## 結論

在專案中，可以無痛導入工作流程而不用特別針對樣式錯誤去手動調整是 `prettier` 的一大優勢，可以讓 coding style 一次到位，也沒有撰寫代碼時多餘的打字或心理成本。

對於已經導入 `eslint` 並有團隊自訂 rule set 的專案，`prettier` 雖可以用參數調整所產出的代碼，但可以調整的地方有限，所產出的代碼也不一定會完全相符於之前的 rule set，這時必定要與團隊成員有良好溝通再行導入，畢竟**"團隊的樣式就是最好的樣式"**。

[vjeux]: https://twitter.com/vjeux
[prettier_web]: https://prettier.github.io/prettier/
[prettier_github]: https://github.com/prettier/prettier
[vscode_ext]: https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode
