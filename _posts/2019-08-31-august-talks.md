---
layout: post
title: "八月雜談匯總"
description: "有點想試一下在校園網環境下 ARP 釣取校園網帳戶密碼，拿來做字典也是極好的。"
date: 2019-08-31
tags: [Life, Information Security, Math]
comments: true
---

從現在開始，一些不成文的想法/知識/紀錄將以這種形式記載，保留特定條目擴寫成文的可能。

在此之前出現大量英文標題的情況，是字符集導致的歷史遺留問題。雖然字符集早已被修復，但使用英文標題的習慣延續了一段時間。我昨天晚上（8.30）翻看了一下編譯原理筆記，發現網站的中文排版實在太漂亮了（）。所以，從現在開始，更多的中文標題將被使用。

## 目錄

+ 製作令牌
+ 微分方程特徵值
+ 在新標籤頁跳轉後修改父網頁
+ DNS 放大攻擊
+ 民生經濟

## 製作令牌

本質上是一個**隨機數生成器**，安裝時在客戶端和服務器生成並保存種子，在可行的時候進行時間同步，定時生成新的隨機數。

## 微分方程和特徵值

微分方程描述了**一類函數**。對於一階微分方程 $\frac{dy}{dt}=\lambda y$，可以將 $\frac{d}{dt}$ 視為一個變換。此時 $\lambda$ 描述了這個變換的特徵值，並且我們確定這個變換的「特徵向量」是 $e^{\lambda t}$。對於二階微分方程 $\frac{d^2}{dx^2}y+\lambda y=0$，則是 $Ae^{\beta x}+Be^{-\beta x}$（$\lambda <0$）或者 $A\cos \beta x + B\sin \beta x$（$\lambda \ge 0$）或者 $Ax+b$（$\lambda = 0$）。

## 在新標籤頁跳轉後修改父網頁

HTTP 提供了一個 `opener` 對象。它可以被子網頁操作，使用 `location.replace` 來替換父網頁，即使是在跨域的情況下。這意味著可以在一個網站中植入（自動打開新標籤頁的）鏈接，誘導用戶點擊，然後替換父網頁為釣魚網站。

對於現代瀏覽器而言，`rel="noopener"` 可以防護這一行為。

## DNS 放大攻擊

DNS 查詢請求具有這樣的特徵，返回數據流量遠大於查詢流量。因此可以通過偽造 DNS 查詢請求，令 DNS 服務器向目標服務器返回查詢結果來放大 DoS 效果。

## 民生經濟

我覺得民生方面的經濟有好多糊塗帳。比如說 2020 年收入翻一番，實際上 2020 年人均購買力可能只翻零點幾番，也可能不翻或者往回翻，易証番數小於一。因為很多人的收入是和物價直接掛鉤的，直觀的例子是販賣農副產品的，如果他們的收入翻一番，物價也要漲高難以置信的幅度。而這是不具備這方面知識和信息的人難以計算的。

人類真的在越來越富有嗎？