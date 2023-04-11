---
title: "Polars v.s. Pandas：既生瑜何生亮！"
date: 2023-03-27T21:03:55+08:00
draft: true
tags: ["data", "preprocess"]
cover:
    image: "img/polars_github_logo_rect_dark_name.svg"
    alt: "polars github logo"
    caption: "Polars Logo (source: [github](https://github.com/pola-rs/polars))"
---

> TL；DR：假如目前的專案中有使用到 `pandas` 來處理**大量**表格化資料的話，請一定要嘗試看看使用 `polars` 替代 `pandas`，使用得當可以大幅降低程式碼執行時間喔！

### 引言

因為公司專案需求的關係，經常需要操作大規模的表格化資料，這時`pandas`相關的操作(I/O、前處理)通常需要大量的時間。

在同事蔡博的推薦下，我嘗試將專案的中的`pandas`使用`polars`取代，結果發現程式執行時間顯著下降，因此希望向有同樣需求的人們推廣這個套件。

在進入正文前，希望先給讀者一些概念，因此我隨機生成了`1,000,000`筆資料，每筆資料有五十個數值型欄位，及五十個類別型欄位。

這時使用 `pandas` 讀取所需的時間為 7.59 秒，然而使用 `polars` 讀取只需要 1.16 秒，讀取速度有六倍的提升。

|               	| 1M data (50 num, 50 cat) 	|
|---------------	|:------------------------:	|
| Polars(sec)   	|          1.1653          	|
| Pandas(sec)   	|          9.4884          	|
| Pandas 2(sec) 	|          7.3792          	|

Table 1: Speed Comparision between Polars and Pandas.

底下我會介紹一些 `polars` 優於 `pandas` 的地方，然後提供一些將 `polars` 導入機器學習專案的建議。

### Why Polars?

在我使用 `Polars` 的過程中，以下三點是我在使用上覺得 `Polars` 最具優勢的地方：

1. Very fast IO
2. Copy-on-write
3. Query optimizations
