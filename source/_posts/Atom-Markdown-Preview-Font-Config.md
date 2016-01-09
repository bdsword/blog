title: Atom Markdown Preview 套件字體樣式設定
date: 2016-01-09 21:20:37
tags:
- Atom
- Markdown
---

Atom Markdown Preview 套件字體樣式設定
====================================

在使用Atom的時候，會將字體設定成支援中文的字體，這可以透過下列方式來設定。

> Atom選單 -> Edit -> Preferences -> Editor Settings -> Font Family

我使用的設定為

> Monaco,Sans,"Adobe Heiti Std",Arial

但是，在使用Markdown preview套件的時候，卻發現中文依然為方塊狀，沒有正確的顯示出來，Google之後發現其實可以打開Atom的使用者style sheet自行設定樣式，開啟方式如下:

> Atom選單 -> Edit -> Open Your Stylesheet

設定為類似下方的樣子即可:

```css
* {
  font-family: Monaco, Sans, Adobe Heiti Std;
}

/* 以下設定markdown-preview套件預覽頁面的字體 */
.markdown-preview.markdown-preview {
  font-family: Monaco, Sans, Adobe Heiti Std;
}
```
