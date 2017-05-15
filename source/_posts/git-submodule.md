---
title: git submodule 教學
categories: 筆記
date: 2017-05-14 12:00:00
tags: [git, CLI]
---

## 前言
Submodules (子模組) 其實是個很常見的概念，在專案中想要引入其他專案的代碼，亦或是第三方的函式庫，被引入的庫就可以被看為是一個子模組。

在 Node.js 的專案中，子模組是由 [yarn] / [npm] 來幫我們管理其中的依賴。其他的語言也有相對應的套件管理系統，像是 Ruby 有 [RubyGems]，Elixir 有 [hex] ……等，這些套件管理系統可以利用 [semver] 格式的版本號來拉取想要的子模組版本。

但如果想引入的專案並沒有放在套件管理系統中，或是這個語言根本沒有對應的套件管理系統的時候，要想管理子模組就開始會變得有點麻煩。我們需要有一個比較好的方式來靈活管理我們的子模組。

<!-- more -->
`git submodule` 指令剛好可以在這邊派上用場，它可以用來管理巢狀的 git 專案。你可以在專案中引用其他的專案，並以該專案的提交 Hash 當做所依賴的版本號 (有點像是用 git 當做你的套件管理系統XD )，但使用的步驟稍微有點複雜，下面會說明。

[yarn]: https://yarnpkg.com/zh-Hans/
[npm]: https://www.npmjs.com/
[RubyGems]: https://rubygems.org/?locale=zh-TW
[hex]: https://hex.pm/
[semver]: http://semver.org/lang/zh-TW/

## 常用指令

### 新增子模組
想要把一個專案當做是子模組加到你想要的路徑之下，可以用
```
$ git submodule add <repository> [<path>]
```
舉個實際的例子
```
$ git submodule add https://github.com/kaddopur/hexo-theme-bubuzou.git themes/bubuzou
```
意思就是把 `hexo-theme-bubuzou` 這個專案加到 `theme/bubuzou` 這個路徑之下。你會發現有三個檔案被改動/新增了，其中兩個你可以看得到。
```
# .gitmodules
[submodule "themes/bubuzou"]
	path = themes/bubuzou
	url = git@github.com:kaddopur/hexo-theme-bubuzou.git
```
```
# themes/bubuzou
Subproject commit 51c576971e7f8f3693bd16ea21075e45758e7432
```
一個是 git 幫我們產生的 `.gitmodules` 設定檔，其中包含了儲存庫與路徑的對應關係。而另一個是 `themes/bubuzou`，由外層的 git 來看，它是一個子模組當前所使用的提交 Hash。而另一個看不到的改動是在 `.git/config` 當中，當你要刪除子模組時要記得去清掉。

我們可以將變動存為另一個提交，如此一來即完成了子模組的新增。

### 提交子模組更新

> 子模組有變動要怎麼提交？可否 resursive 提交？

### 移除子模組

要移除子模組有點麻煩，一共 6 個步驟

1. 刪掉 `.git/config` 中相關的段落
2. 刪掉 `.gitmodules` 中相關的段落
3. 刪掉 `.git/modules/<submodule>` 
4. 用 `git rm --cached` 刪掉子模組，並停止 tracking 子模組
5. 提交更新
6. 真正把子模組刪除

實際的指令可能像是
```
```



Delete the relevant section from the .gitmodules file.
Stage the .gitmodules changes git add .gitmodules
Delete the relevant section from .git/config.
Run git rm --cached path_to_submodule (no trailing slash).
Run rm -rf .git/modules/path_to_submodule
Commit git commit -m "Removed submodule <name>"
Delete the now untracked submodule files
rm -rf path_to_submodule


### 複製包含子模組的專案
直接 `git clone` 上層專案，你會發現子模組的資料夾存在但卻是空的。此時你要先運行
```
$ git submodule init
```
這個指令會將你在 `.gitmodules` 中的子模組設定寫入 `.git/config` 中，接下來再運行

```
$ git submodule update
```
這個時候子模組才會真正被下載下來

## 結論
如果你的專案常常被其他專案引用，這時就是你拆分專案成為一個子模組的好時機。但如果你是寫 Node.js 或是 Ruby，請儘量使用該語言的套件管理系統來公布你的專案以方便日後引用。但


## 後記
這次會寫 `git submodule` 的文章是因為這個部落格用了一個新主題 [bubuzou]，而我想要對這個主題做一些修改並加入版本控制，所以我先 fork 這個主題後，將其以[子模組]的方式加入我的部落格中。而文內所提到的一些指令，主要是剛開始設定部落格主題時實際用到的，我想將之記錄下來以便日後查詢，或是有人要問的時候可以少費一些唇舌直接貼這篇。在此特別感謝強者我同事[湯姆餡]熱心教學。

[bubuzou]: https://github.com/Bulandent/hexo-theme-bubuzou
[湯姆餡]: https://tom76kimo.github.io/blog/
[子模組]: https://git-scm.com/book/zh-tw/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E7%B5%84-Submodules
