---
title: Polars R パッケージについて
emoji: 🐻‍❄️
type: tech
topics:
    - r
    - rust
    - polars
published: true
published_at: 2023-12-01
format:
  md:
    variant: gfm+yaml_metadata_block+hard_line_breaks
engine: knitr
knitr:
  opts_chunk:
    comment: "#>"
---

高速データフレームライブラリのPolarsといえばPythonパッケージが有名で、[H2O.aiのベンチマーク](https://h2oai.github.io/db-benchmark/)で上位に入ったことで知られていますが、今日紹介するのはRパッケージです。

Polarsという名前は聞いたことあってもRパッケージについては知らない方も多いかも知れません。実は一年くらい前からPolarsの開発を行っているPolarsのGitHub組織にリポジトリがあります。

https://twitter.com/eitsupi/status/1608013971428368385

:::message

これは[R言語 Advent Calendar 2023](https://qiita.com/advent-calendar/2023/rlang)の1日目の記事です。\
12月2日開催予定の[Japan.R 2023](https://japanr.connpass.com/event/302622/)での発表内容と関連しています。[^japanr]

（2023-12-02 追記） 発表しました。 → [発表資料](https://eitsupi.github.io/tokyorslide/japanr2023/)

:::

[^japanr]: 発表資料はこれから作ります……。

:::message

後付けで[Polars Advent Calendar 2023](https://qiita.com/advent-calendar/2023/polars)の1日目の記事ということにもしました。

:::

## はじめに

### Polarsとは

Rustで書かれたデータフレームライブラリで、内部でApache Arrowフォーマットを使用しています。
競合する側面を持っておりApache Arrowの一部である[Arrow DataFusionのFAQ](https://arrow.apache.org/datafusion/user-guide/faq.html)に「Polarsは執筆時点で最も高速なデータフレームライブラリの一つです（Polars is one of the fastest DataFrame libraries at the time of writing.）」と書かれるほど、高速です。

開発が行われている[Gitリポジトリ](https://github.com/pola-rs/polars)にはPythonバインディング（polars Pythonパッケージ、通称py-polars）も含まれており、py-polarsはPandasの代替として人気を博しています。

### Polars R パッケージとは

https://rpolars.github.io/

PolarsのRバインディングです。主要機能は実装されているので、Pythonパッケージのドキュメントとほぼ同じことができるはずです。（できない場合はissueで機能リクエストできます）
なお私もメンテナの一人にしてもらっています。

以下のissueの通りPolarsをRから使いたいという要望は以前からあったもののかなり大変な作業が必要で、結局[@sorhawell](https://github.com/sorhawell)さんが一年半ほどフルタイムに近い時間を費やして開発（！）してようやく今の完成度になっています。

https://github.com/pola-rs/polars/issues/996

CRANには登録されておらず、R-universeからのインストール推奨です。現在R-universe上のバージョンは最新リリースに固定されています。

```r:R
Sys.setenv(NOT_CRAN = "true") # ソースインストールの場合、この環境変数を設定することでビルド済バイナリをGitHubからダウンロードしてRustソースからのビルドを回避できます
install.packages("polars", repos = "https://rpolars.r-universe.dev")
```

なおビルド済バイナリをGitHubからダウンロードしてRustソースのコンパイルを避ける機能はバージョン0.9.0から導入されているため、`NOT_CRAN="true"`を設定している場合は`remotes::install_github()`等で過去のバージョンをインストールする場合もRustは不要です。

## 使い方

（今現在少し古いバージョンですが）以下のウェブサイトによくまとまっているので、こちらもご覧ください。

https://ddotta.github.io/cookbook-rpolars/

Polarsは凄まじい勢いで破壊的な変更を行っておりRパッケージも例外ではない点はご留意ください。

### メソッドチェーン

Polarsは「データフレームライブラリ」と呼ばれている通り、Rのデータフレームとほぼ同じ機能を持ったものを提供します。
つまりこのパッケージをインストールすることによりRのデータフレームとは別にRのデータフレームのようなものを導入することになるわけで、Rのデータフレームを操作するための関数と同じような機能を持ったPolarsのデータフレームを操作するための、大量の関数が必要になります。
実際、`polars`パッケージのRdファイル（ヘルプページ）の数は現在570で、これは`ggplot2`でも131（現時点での最新開発バージョン）であることと比較するととてつもない数です。

これの何が問題かというと、Rでは関数を名前空間の区別なくフラットにインポートするのが一般的ということで、普通ならば関数名が衝突して困る、もしくは全関数に`pl_`のような接頭辞を付ける（※rOpenSci等で推奨されている方）ことで衝突を回避しなければなりません。

しかしそれにしてもPolarsの関数は多いということで、`polars`パッケージでは`R6`パッケージによって提供されるR6クラスのように`$`によるメソッド呼び出しを基本とし、関数のエクスポートを回避しています。（かつては`R6`パッケージを使用していたが現在は未使用）

トップレベルの関数も基本的に`pl`というオブジェクト（環境、`environment`型）の下にくっついています。

```r:R
library(polars)

pl$polars_info()
```

    #> r-polars package version : 0.11.0
    #> rust-polars crate version: 0.35.2
    #> 
    #> Features:                         
    #> default              TRUE
    #> full_features        TRUE
    #> simd                 TRUE
    #> sql                  TRUE
    #> rpolars_debug_print FALSE

`base::data.frame()`に相当する関数は`pl$DataFrame()`です。[^1]

```r:R
mtcars_pl = pl$DataFrame(mtcars)

mtcars_pl
```

    #> shape: (32, 11)
    #> ┌──────┬─────┬───────┬───────┬───┬─────┬─────┬──────┬──────┐
    #> │ mpg  ┆ cyl ┆ disp  ┆ hp    ┆ … ┆ vs  ┆ am  ┆ gear ┆ carb │
    #> │ ---  ┆ --- ┆ ---   ┆ ---   ┆   ┆ --- ┆ --- ┆ ---  ┆ ---  │
    #> │ f64  ┆ f64 ┆ f64   ┆ f64   ┆   ┆ f64 ┆ f64 ┆ f64  ┆ f64  │
    #> ╞══════╪═════╪═══════╪═══════╪═══╪═════╪═════╪══════╪══════╡
    #> │ 21.0 ┆ 6.0 ┆ 160.0 ┆ 110.0 ┆ … ┆ 0.0 ┆ 1.0 ┆ 4.0  ┆ 4.0  │
    #> │ 21.0 ┆ 6.0 ┆ 160.0 ┆ 110.0 ┆ … ┆ 0.0 ┆ 1.0 ┆ 4.0  ┆ 4.0  │
    #> │ 22.8 ┆ 4.0 ┆ 108.0 ┆ 93.0  ┆ … ┆ 1.0 ┆ 1.0 ┆ 4.0  ┆ 1.0  │
    #> │ 21.4 ┆ 6.0 ┆ 258.0 ┆ 110.0 ┆ … ┆ 1.0 ┆ 0.0 ┆ 3.0  ┆ 1.0  │
    #> │ …    ┆ …   ┆ …     ┆ …     ┆ … ┆ …   ┆ …   ┆ …    ┆ …    │
    #> │ 15.8 ┆ 8.0 ┆ 351.0 ┆ 264.0 ┆ … ┆ 0.0 ┆ 1.0 ┆ 5.0  ┆ 4.0  │
    #> │ 19.7 ┆ 6.0 ┆ 145.0 ┆ 175.0 ┆ … ┆ 0.0 ┆ 1.0 ┆ 5.0  ┆ 6.0  │
    #> │ 15.0 ┆ 8.0 ┆ 301.0 ┆ 335.0 ┆ … ┆ 0.0 ┆ 1.0 ┆ 5.0  ┆ 8.0  │
    #> │ 21.4 ┆ 4.0 ┆ 121.0 ┆ 109.0 ┆ … ┆ 1.0 ┆ 1.0 ┆ 4.0  ┆ 2.0  │
    #> └──────┴─────┴───────┴───────┴───┴─────┴─────┴──────┴──────┘

このpolarsデータフレームに対して、`filter`メソッドと`select`メソッドを使用して行と列を選択する例はこのようになります。列は`pl$col()`関数で指定します。

```r:R
mtcars_pl$
  filter(pl$col("mpg") > 22)$
  select(pl$col("cyl"))
```

    #> shape: (9, 1)
    #> ┌─────┐
    #> │ cyl │
    #> │ --- │
    #> │ f64 │
    #> ╞═════╡
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> └─────┘

これはpy-polarsのドット`.`を使用したメソッドチェーンをドルマーク`$`で書き直したようなものなので、基本的にPython用のドキュメントを見ればどのように書けば良いのか分かるはずです。（ただし一部関数名はRust、Python、Rのそれぞれで異なります）

### Expression

Polars独特の仕組みとして、Expressionがあります。
例えば上記の例で出てきた以下のようなものです。

```r:R
pl$col("mpg")
```

    #> polars Expr: col("mpg")

NSEによって引用符で囲まずに列名を指定するのが一般的なRの世界では冗長に見えますが、Expressionが独自のクラスを持っているお陰で関数定義は単純です。（一方`arrow`パッケージのdplyrクエリ内で自作関数を使おうとすると大変です）

```r:R
# Expressionを返す関数を定義する
foo <- function(col_name) {
  polars::pl$col(col_name) * 100
}

# selectメソッド内で評価させる
mtcars_pl$select(new = foo("cyl"))
```

    #> shape: (32, 1)
    #> ┌───────┐
    #> │ new   │
    #> │ ---   │
    #> │ f64   │
    #> ╞═══════╡
    #> │ 600.0 │
    #> │ 600.0 │
    #> │ 400.0 │
    #> │ 600.0 │
    #> │ …     │
    #> │ 800.0 │
    #> │ 600.0 │
    #> │ 800.0 │
    #> │ 400.0 │
    #> └───────┘

なお上記で使用している`*`演算子も、Expressionクラスの`mul`メソッドを呼び出すように定義されたS3メソッドとして定義されています。

```r:R
getAnywhere(`*.Expr`)
```

    #> A single object matching '*.Expr' was found
    #> It was found in the following places
    #>   registered S3 method for * from namespace polars
    #>   namespace:polars
    #> with value
    #> 
    #> function (e1, e2) 
    #> unwrap(result(wrap_e(e1)$mul(e2)), "using the '*'-operator")
    #> <bytecode: 0x55794679c698>
    #> <environment: namespace:polars>

### LazyFrame

Polarsの目玉機能の一つが、LazyFrameを使用した遅延評価です。
`dplyr`に慣れ親しんだ方ならば`dtplyr::lazy_dt`や`dbplyr::lazy_frame`をご存じかも知れませんが、それらと同じようにクエリを保存し最後に評価する機能です。

`as_polars_lf()`関数でRデータフレームやpolarsデータフレームからLazyFrameを作成できます。

```r:R
mtcars_lf <- as_polars_lf(mtcars_pl)$
  filter(pl$col("mpg") > 22)$
  select(pl$col("cyl"))

mtcars_lf
```

    #> [1] "polars LazyFrame naive plan: (run ldf$describe_optimized_plan() to see the optimized plan)"
    #>  SELECT [col("cyl")] FROM
    #>   FILTER [(col("mpg")) > (22.0)] FROM
    #> 
    #>   DF ["mpg", "cyl", "disp", "hp"]; PROJECT */11 COLUMNS; SELECTION: "None"

`collect`メソッド（もしくは`fetch`メソッド）で評価します。

```r:R
mtcars_lf$collect()
```

    #> shape: (9, 1)
    #> ┌─────┐
    #> │ cyl │
    #> │ --- │
    #> │ f64 │
    #> ╞═════╡
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> │ 4.0 │
    #> └─────┘

私は試したことはないのですが、評価時にストリーミングモードを選択することでRAMに乗り切らない巨大なデータセットも処理できるらしいです。

`collect`メソッドの`streaming`引数でストリーミングモードを有効化できます。

```r:R
data_dir <- tempfile()
arrow::write_dataset(mtcars, data_dir, partitioning = c("cyl", "gear"))

pl$scan_parquet(paste0(data_dir, "/**/*.parquet"))$　# Hive-styleパーティションを読み込めるぞ！
  filter(pl$col("mpg") > 22)$
  select(pl$col("cyl"))$
  collect(streaming = TRUE)
```

    #> shape: (9, 1)
    #> ┌─────┐
    #> │ cyl │
    #> │ --- │
    #> │ i64 │
    #> ╞═════╡
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> │ 4   │
    #> └─────┘

### nanoarrow

nanoarrowというApache Arrowのサブプロジェクトをご存じでしょうか？
巨大でビルドが大変なArrow C++ライブラリ（libarrow）の代わりに最小限の機能を提供するCライブラリで、RパッケージがCRANから利用可能です。

最近PythonのSnowflakeコネクターは`pyarrow`から`nanoarrow`に切り替えて色々捗るようになったそうです。

https://medium.com/snowflake/announcing-the-ga-release-of-snowflake-python-connector-with-nanoarrow-1d70fd8ba3b1

`polars`はnanoarrowと統合した最初期のパッケージで、polarsデータフレームからnanoarrow array streamへの変換をサポートしています。

```r:R
mtcars_pl |>
  nanoarrow::as_nanoarrow_array_stream()
```

    #> <nanoarrow_array_stream struct<mpg: double, cyl: double, disp: double, hp: double, drat: double, wt: double, qsec: double, vs: double, am: double, gear: double, carb: double>>
    #>  $ get_schema:function ()  
    #>  $ get_next  :function (schema = x$get_schema(), validate = TRUE)  
    #>  $ release   :function ()

今年中にCRANリリース予定[^2]の`DBI`パッケージ新バージョンではnanoarrow array streamによるデータ交換がサポートされる予定など、`nanoarrow`は今後要注目です。

## よくある質問

### CRANでリリースしないのですか？

2023年7月、苦闘の末にリリースしたのですが、わずか数時間後アーカイブ送りになりました。なのでCRAN上にはバージョン0.7.0のソースコードがあります。

Polarsは要求するRustのバージョンが新しくCRAN上で利用可能なRustではビルドできないという問題があり、それ以降はリリースを試みていません。
7月に一瞬だけCRAN上のバージョンでビルド可能になったのですが、またビルド不能な状況に戻っています。

また、現状Polarsは頻繁に破壊的変更が入るため、CRANでリリースして逆依存関係が発生した場合に破壊的変更の対応が大変になるという問題もあります。なのでCRANからインストールできるようになるのはまだ先になりそうです。

### dplyrのAPIで操作したいのですが？

`tidypolars`を使用できます。

https://github.com/etiennebacher/tidypolars

### DBIのAPIで操作したいのですが？

`polarssql`を使用できます。

https://github.com/rpolars/r-polarssql

## 実行環境

```r:R
sessioninfo::session_info()
```

    #> ─ Session info ───────────────────────────────────────────────────────────────
    #>  setting  value
    #>  version  R version 4.3.2 (2023-10-31)
    #>  os       Ubuntu 22.04.3 LTS
    #>  system   x86_64, linux-gnu
    #>  ui       X11
    #>  language (EN)
    #>  collate  en_US.UTF-8
    #>  ctype    en_US.UTF-8
    #>  tz       Etc/UTC
    #>  date     2023-11-28
    #>  pandoc   3.1.1 @ /usr/local/bin/ (via rmarkdown)
    #> 
    #> ─ Packages ───────────────────────────────────────────────────────────────────
    #>  package     * version  date (UTC) lib source
    #>  arrow         13.0.0.1 2023-09-22 [1] RSPM (R 4.3.0)
    #>  assertthat    0.2.1    2019-03-21 [1] RSPM (R 4.3.0)
    #>  bit           4.0.5    2022-11-15 [1] RSPM (R 4.3.0)
    #>  bit64         4.0.5    2020-08-30 [1] RSPM (R 4.3.0)
    #>  cli           3.6.1    2023-03-23 [1] RSPM (R 4.3.0)
    #>  digest        0.6.33   2023-07-07 [1] RSPM (R 4.3.0)
    #>  dplyr         1.1.3    2023-09-03 [1] RSPM (R 4.3.0)
    #>  evaluate      0.22     2023-09-29 [1] RSPM (R 4.3.0)
    #>  fansi         1.0.5    2023-10-08 [1] RSPM (R 4.3.0)
    #>  fastmap       1.1.1    2023-02-24 [1] RSPM (R 4.3.0)
    #>  generics      0.1.3    2022-07-05 [1] RSPM (R 4.3.0)
    #>  glue          1.6.2    2022-02-24 [1] RSPM (R 4.3.0)
    #>  htmltools     0.5.6.1  2023-10-06 [1] RSPM (R 4.3.0)
    #>  jsonlite      1.8.7    2023-06-29 [1] RSPM (R 4.3.0)
    #>  knitr         1.44     2023-09-11 [1] RSPM (R 4.3.0)
    #>  lifecycle     1.0.3    2022-10-07 [1] RSPM (R 4.3.0)
    #>  magrittr      2.0.3    2022-03-30 [1] RSPM (R 4.3.0)
    #>  nanoarrow     0.3.0    2023-09-29 [1] RSPM
    #>  pillar        1.9.0    2023-03-22 [1] RSPM (R 4.3.0)
    #>  pkgconfig     2.0.3    2019-09-22 [1] RSPM (R 4.3.0)
    #>  polars      * 0.11.0   2023-11-28 [1] https://rpolars.r-universe.dev (R 4.3.2)
    #>  purrr         1.0.2    2023-08-10 [1] RSPM (R 4.3.0)
    #>  R6            2.5.1    2021-08-19 [1] RSPM (R 4.3.0)
    #>  rlang         1.1.1    2023-04-28 [1] RSPM (R 4.3.0)
    #>  rmarkdown     2.25     2023-09-18 [1] RSPM
    #>  rstudioapi    0.15.0   2023-07-07 [1] RSPM (R 4.3.0)
    #>  sessioninfo   1.2.2    2021-12-06 [1] RSPM (R 4.3.0)
    #>  tibble        3.2.1    2023-03-20 [1] RSPM (R 4.3.0)
    #>  tidyselect    1.2.0    2022-10-10 [1] RSPM (R 4.3.0)
    #>  utf8          1.2.4    2023-10-22 [1] RSPM (R 4.3.0)
    #>  vctrs         0.6.4    2023-10-12 [1] RSPM (R 4.3.0)
    #>  xfun          0.40     2023-08-09 [1] RSPM (R 4.3.0)
    #>  yaml          2.3.7    2023-01-23 [1] RSPM (R 4.3.0)
    #> 
    #>  [1] /usr/local/lib/R/site-library
    #>  [2] /usr/local/lib/R/library
    #> 
    #> ──────────────────────────────────────────────────────────────────────────────

[^1]: Rデータフレームをpolarsデータフレームに変換する場合、`polars::as_polars_df()`という関数を使うこともできます。

[^2]: https://fosstodon.org/@kirill/111477378191048567
