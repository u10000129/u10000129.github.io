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

> TL；DR：假如目前的專案中有使用 [Pandas](https://pandas.pydata.org/) 來處理**大量**表格化資料的話，請一定要嘗試看看使用 [Polars](https://www.pola.rs/) 替代 Pandas，使用得當將可以大幅降低程式碼執行時間！

### Links

- [Polars](https://www.pola.rs/)
- Polars - [User Guide](https://pola-rs.github.io/polars-book/user-guide/)
- Polars - [Migrate from Pandas](https://pola-rs.github.io/polars-book/user-guide/migration/pandas/)
- Polars - [Lazy API](https://pola-rs.github.io/polars-book/user-guide/lazy/using/)

### Why Polars?

平時在處理大量表格化資料的時候，I/O 和前處理經常會需要大量的時間，在同事蔡博的推薦下，嘗試使用了 Polars，結果發現速度相比於 Pandas 而言真的快上許多。

基於 Polars 的官方文件，Polars 的優勢來自於：

1. 使用 [Rust](https://www.rust-lang.org/) 實作
2. 遵循 [Apache Arrow](https://arrow.apache.org/) 規範


### How Polars?

這篇文章不會介紹如何使用 Polars，因為 Polars 官方的 [User Guide](https://pola-rs.github.io/polars-book/user-guide/) 已經寫得非常清楚！

底下節錄兩個我認為特別重要的章節：

1. [Migrate from Pandas to Polars](https://pola-rs.github.io/polars-book/user-guide/migration/pandas/)
2. [Lazy API](https://pola-rs.github.io/polars-book/user-guide/lazy/using/)

Polars 和 Pandas 有許多相同的 function，但[語法上也有許多需要轉換的地方](https://pola-rs.github.io/polars-book/user-guide/migration/pandas/#key-syntax-differences)，使用 Pandas 的語法去寫 Polars 有些情況可行，但是會失去 Polars 的速度優勢，所以需要特別注意。

另外，使用 [Lazy API](https://pola-rs.github.io/polars-book/user-guide/lazy/using/) 去寫 Polars，Polars 會自行執行 [Query Plan](https://pola-rs.github.io/polars-book/user-guide/lazy/query_plan/#graphviz-visualization)，就像是 [DBMS](https://en.wikipedia.org/wiki/Database#Database_management_system) 在處理SQL！所以，請盡可能地使用 Lazy API。