---
title: "Polars v.s. Pandas：既生瑜何生亮！"
date: 2023-05-09T23:20:38+08:00
draft: false
tags: ["data", "preprocess"]
cover:
    image: "img/polars_github_logo_rect_dark_name.svg"
    alt: "polars github logo"
    caption: "Polars Logo (source: [github](https://github.com/pola-rs/polars))"
---

> TL；DR：假如目前的專案中有使用 [Pandas](https://pandas.pydata.org/) 來處理*大量*表格化資料的話，請一定要嘗試看看使用 [Polars](https://www.pola.rs/) 替代 Pandas，使用得當將可以大幅降低程式碼執行時間！

## Why Polars?

平時在處理大量表格化資料的時候，I/O 和前處理經常會需要大量的時間，在同事蔡博的推薦下，嘗試使用了 Polars，結果發現速度相比於 Pandas 而言真的快上許多。

基於 Polars 的官方文件，Polars 的優勢包含：

1. 使用 [Rust](https://www.rust-lang.org/) 實作，比起 Pandas 使用 Python 實作，更加底層
2. 遵循 [Apache Arrow](https://arrow.apache.org/) 規範，使得在資源運用上更有效率
3. etc.


## How Polars?

這篇文章不會介紹如何使用 Polars，因為 Polars 官方的 [User Guide](https://pola-rs.github.io/polars-book/user-guide/) 已經寫得非常清楚！

底下節錄兩個我認為特別重要的章節：

1. [Migrate from Pandas to Polars](https://pola-rs.github.io/polars-book/user-guide/migration/pandas/)
2. [Lazy API](https://pola-rs.github.io/polars-book/user-guide/lazy/using/)

Polars 和 Pandas 有許多相同的 function，但語法上也有許多需要轉換的地方，請參考 [key-syntax-differences](https://pola-rs.github.io/polars-book/user-guide/migration/pandas/#key-syntax-differences)，使用 Pandas 的語法去寫 Polars 有些情況可行，但是會無法發揮 Polars 的優勢，需要特別注意。

另外，使用 [Lazy API](https://pola-rs.github.io/polars-book/user-guide/lazy/using/) 去寫 Polars，Polars 會自行執行 [Query Plan](https://pola-rs.github.io/polars-book/user-guide/lazy/query_plan/#graphviz-visualization)，就像是 [DBMS](https://en.wikipedia.org/wiki/Database#Database_management_system) 在處理SQL！所以，請盡可能地使用 Lazy API。

## Polars v.s. Pandas!

在這個章節我使用了一些常見的 function，來比較 Polars 和 Pandas 的速度，包括：

- [Read CSV](#read-csv)
- [Join](#join)
- [Filter](#filter)
- [Aggregate](#aggregate)
- [GroupBy](#groupby)

在比較速度以前，我生成了總共有 101 個欄位的假資料，其中 1 欄是 id，然後各有 50 欄的數值型及類別型欄位，並且生成了五種不同筆數的資料，分別為 10 萬筆、20 萬筆、50 萬筆、80 萬筆及 100 萬筆。

每種不同的 function，都會重複執行 20 次，並將每次執行所需的時間記錄下來，底下所展示的摺線圖，實線的部分代表這 20 次測試計算出來的平均值，而影子的部分則代表 95% 的信賴區間。

在實驗的過程裡，我使用了 v2.0.1 的 Pandas 及 v0.17.11 的 Polars。

除了 Pandas 既有的 numpy backend 以外，我同時比較了在 Pandas v2.0.0 以後推出的 pyarrow backend，並且在 pip install 的時候遵循 Pandas 的[官方建議](https://pandas.pydata.org/docs/dev/whatsnew/v2.0.0.html)，下載了`performance`及`aws`的 dependencies，來確保 Pandas 能展現最佳效能。

下圖中的標示意義如下：

- Polars - Polars
- Pandas2 - Pandas using pyarrow backend
- Pandas - Pandas using numpy backend


### Read CSV

可以看到 Pandas 在使用 pyarrow backend 的情況下，速度相有明顯的提升，但仍然是 Polars 最具優勢。

{{< figure src="/polars-vs-pandas/read_csv.png" caption="Fig. 1. Read CSV file." align=center >}}

### Join

注意一下，這邊使用到的 Join 是根據 DataFrame 的 ID 欄位進行 5 次的 Join，並且 Polars 在執行時有使用到 Lazy Operation。

可以看到仍是 Polars 較具優勢。

{{< figure src="/polars-vs-pandas/quintuple_join.png" caption="Fig. 2. Join." align=center >}}

### Filter

Filter 的部分，我針對數值型欄位及類別型欄位分別進行了兩種 Filter。

數值型的部分，我透過中位數進行 Filter，實驗結果在 [Filter by Number](#filter-by-number)。

類別行的部分，我透過特定字元 ("a") 進行 Filter，實驗結果在 [Filter by Character](#filter-by-character)。

#### Filter by Number

可以看到 Polars 仍是最快，**並且 Pandas 使用 pyarrow backend 會比使用 numpy backend 還慢！**

{{< figure src="/polars-vs-pandas/filter_by_num.png" caption="Fig. 3. Filter by Number." align=center >}}

#### Filter by Character

同樣地，Polars 的速度最快，並且 Pandas 使用 pyarrow backend，在進行 Filter 時的速度較慢。

{{< figure src="/polars-vs-pandas/filter_by_cat.png" caption="Fig. 4. Filter by Character." align=center >}}

### Aggregate

在做 Aggregate 的實驗時，我使用了數值型欄位和類別型欄位，總共做了三種 Aggregate。

首先，使用數值型欄位，透過 Aggregate 成 Median 及 Mean，進行了兩種實驗，分別放在 [Aggregate by Median](#aggregate-by-median) 以及 [Aggregate by Mean](#aggregate-by-mean)。

再來，使用類別型欄位，透過 Aggregate 成 Unique Value，進行了一種實驗，放在 [Aggregate by the Number of Unique Value](#aggregate-by-the-number-of-unique-value)。

#### Aggregate by Median

可以看到，Polars 最快，並且 Pandas 使用 pyarrow backend 比使用 numpy backend 較慢。

{{< figure src="/polars-vs-pandas/agg_by_median.png" caption="Fig. 5. Aggregate by Median." align=center >}}

#### Aggregate by Mean

可以看到，Polars 最快，並且 Pandas 使用 pyarrow backend 和使用 numpy backend 的結果差不多。

{{< figure src="/polars-vs-pandas/agg_by_mean.png" caption="Fig. 6. Aggregate by Mean." align=center >}}

#### Aggregate by the Number of Unique Value

可以看到，Polars 最快，並且 Pandas 中，使用 pyarrow backend 較快。

{{< figure src="/polars-vs-pandas/agg_by_nunique.png" caption="Fig. 7. Aggregate by Mean." align=center >}}

### GroupBy

可以看到，Polars 最快，並且 Pandas 中，使用 pyarrow backend 慢上許多。

{{< figure src="/polars-vs-pandas/groupby.png" caption="Fig. 8. GroupBy." align=center >}}

## Conclusion

以速度方面考量，Polars 真的值得一試。Pandas 雖然在近期推出了第二版，加入了使用 pyarrow backend 的選擇，但以不同的 operation 去嘗試，會發現沒辦法在各個面向都取得速度上的進步。

因此，如果有需要快速操作 DataFrame 的需求，就一起來使用 Polars 吧！

## Codebase

整個測試過程所使用到的程式碼都整理在這個 [Repo](https://github.com/u10000129/polars-vs-pandas)。

- 產生假資料：[gen_data.py](https://github.com/u10000129/polars-vs-pandas/blob/main/gen_data.py)
- 測試速度：[test_speed.py](https://github.com/u10000129/polars-vs-pandas/blob/main/test_speed.py)
- 畫圖：[draw.py](https://github.com/u10000129/polars-vs-pandas/blob/main/draw.py)

## Links

- [Polars](https://www.pola.rs/)
- Polars - [User Guide](https://pola-rs.github.io/polars-book/user-guide/)
- Polars - [Migrate from Pandas](https://pola-rs.github.io/polars-book/user-guide/migration/pandas/)
- Polars - [Lazy API](https://pola-rs.github.io/polars-book/user-guide/lazy/using/)