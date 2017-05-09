---
title: 使用 prettier 自動調整 JavaScript 樣式
categories: 週記
date: 2017-05-09
tags: [JavaScript, prettier, Facebook, tool]
---

## 前言

`prettier` 是一款 JavaScript 的樣式處理工具，它類似 golang 的 `gofmt` 可以自動排版你的程式。除了 Vanilla JS 之外，尚支援了 ES6, jsx, Flow 以及正在開發中的 TypeScript。它是由 [vjeux (Christopher Chedeau)][vjeux]  開發，在 [github][prettier_github] 上面約有 10,000 個 stars。

`prettier` 從今年一月的第一個 commit 開始到現在快半年，不過到它真正引起我的興趣，是從上週 (5/3) 釋出的 `v1.3.0` 開始。至於這個版本有什麼特別的功能嗎？非也。主要是因為這個版本有公開它在 Facebook 內部各專案使用 `prettier` 的情形。

> The first projects to adopt prettier were Jest, React and immutable-js. Those are small codebases in the order of hundreds of files that have their own infrastructure. There are a handful of people working on them full time.<br>
> Then, Oculus and Nuclide converted their codebase over. The scale is bigger with a few thousands of files and tens of full time contributors but looks pretty similar to the first projects. The conversions went in one big codemod and that's it.

可以看到 Facebook 的專案一個接一個開始用 `prettier` 來規範及管理他們的 JavaScript 樣式，而 `prettier` 和我們平常在用的 `eslint` 或是其他的 lint 工具相比之下孰優孰劣呢？

## 特點

## 如何使用

`prettier` 提供了 CLI 方便你轉換檔案。你也可以將 `prettier` 設定在專案的工作流程中，在 commit 時自動排版並簽入至你的 repo。或是你也可以將 `prettier` 與你最愛的編輯器綁定，讓你在存檔時可以自動排版並看到結果。以下會分別介紹。

### CLI

### Node

### Editor (VS Code)

## 結論

在專案中，可以無痛導入工作流程而不用特別針對樣式錯誤去手動調整是 `prettier` 的一大優勢，可以讓 coding style 一次到位，也沒有撰寫代碼時的多餘打字或心理成本。

對於已經導入 `eslint` 並有團隊自訂 rule set 的專案，`prettier` 雖可以用參數調整所產出的代碼，但可以調整的地方有限，所產出的代碼也不一定會完全相符於之前的 rule set，這時必定要與團隊成員有良好溝通再行導入，畢竟**"團隊的樣式就是最好的樣式"**。

[vjeux]: https://twitter.com/vjeux
[prettier_web]: https://prettier.github.io/prettier/
[prettier_github]: https://github.com/prettier/prettier
