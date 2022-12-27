---
title: 2022年末、Rでcsvを読む
emoji: 📄
type: tech
topics:
    - r
published: true
format:
  md:
    variant: gfm+yaml_metadata_block+hard_line_breaks
engine: knitr
knitr:
  opts_chunk:
    comment: "#>"
execute:
  cache: true
---

データの保存先として手軽で身近なcsvやtsvといったプレーンテキスト形式ですが、Rに読み込む方法について、初学者だった数年前に悩まされた覚えがあります。

-   様々なパッケージに関数があり、どれを使えば良いのかわからない。
-   複数のファイルを読み込むにはどうすれば良いのかわからない。

これらについて書かれているインターネット上の情報も古くなっていたりしますので、あえて今まとめてみました。

2022年末版として、つい先日公開された`purrr` 1.0.0や、来月リリース予定の`dplyr` 1.1.0にも触れます。

## サンプルデータ

サンプルとして、`readr`パッケージに含まれているcsvファイルを使います。これらのファイルは`readr::readr_example()`でパスを呼び出せます。

試しに`mini-gapminder-africa.csv`というファイルを読み込んでみましょう。

```r:R
readr::readr_example("mini-gapminder-africa.csv") |>
  read.csv()
```

    #>        country year lifeExp     pop gdpPercap
    #> 1      Algeria 1952  43.077 9279525 2449.0082
    #> 2       Angola 1952  30.015 4232095 3520.6103
    #> 3        Benin 1952  38.223 1738315 1062.7522
    #> 4     Botswana 1952  47.622  442308  851.2411
    #> 5 Burkina Faso 1952  31.975 4469979  543.2552
    #> 6      Burundi 1952  39.031 2445618  339.2965

mini-gapminderというデータのファイルは地域別で5つありますので、それら5つのパスを文字列型のベクトル`paths_csv_examples`として保存し、以下で使用します。

```r:R
paths_csv_examples <- glue::glue(
  "mini-gapminder-{area}.csv",
  area = c("africa", "americas", "asia", "europe", "oceania")
) |>
  purrr::map_chr(readr::readr_example)
```

## csvファイルを読むための関数達

csvファイルを読み込んでデータフレームを作る関数として、主要なものは以下の4つ（5つ）だと思います。

-   `utils::read.csv()`
-   `data.table::fread()`
-   `readr::read_csv()`
-   `arrow::read_csv_arrow()`, `arrow::open_dataset()`

それぞれの詳細に行く前に、重要機能の比較を載せておきます。

|                      | read.csv | data.table::fread | readr::read_csv | arrow::read_csv_arrow<br/>arrow::open_dataset |
|--------------|:-----:|:----------:|:---------:|:----------------------------:|
| Shift_JIS対応        |    ○     |         ×         |        ○        |                       ○                       |
| 複数ファイル読み込み |    ×     |         ×         |        ○        |             ○ （`open_dataset`）              |

Shift_JIS対応は今回は読み取り関数なので取り上げていますが、Rからの書き出しに関してはMicrosoft Excelで直で開ければ良いだけであれば`readr::write_excel_csv()`でBOM付きUTF-8にする方法もあります。
Shift_JISはできるだけ避けていきたいものです。

複数ファイル読み込みについては、非対応の関数を使って読み込む方法も記事の後ろの方に記載しておきます。

載せていない重要な要素である読み込み速度について、私は計っていないため載せていませんが、過去に見たベンチマーク結果[^1][^2]によると`fread()`が最も速く、`read_csv_arrow()`も同じくらい速く、`read_csv()`は少し遅れて、`read.csv()`はかなり遅い、という傾向のようです。
ただ、読み込み速度はファイルの中身にもよるので、速度で困った場合には`read.csv()`以外をそれぞれ試してみることをおすすめします。

### read.csv

追加のパッケージインストールなしで使える、基本的な関数です。
後述する他のパッケージを使える状況であればあえて使う必要はないでしょう。

#### 使い方

```r:R
read.csv(paths_csv_examples[1])
```

    #>        country year lifeExp     pop gdpPercap
    #> 1      Algeria 1952  43.077 9279525 2449.0082
    #> 2       Angola 1952  30.015 4232095 3520.6103
    #> 3        Benin 1952  38.223 1738315 1062.7522
    #> 4     Botswana 1952  47.622  442308  851.2411
    #> 5 Burkina Faso 1952  31.975 4469979  543.2552
    #> 6      Burundi 1952  39.031 2445618  339.2965

R 4.0よりも前のバージョンでは`stringsAsFactors`オプションのデフォルト値が`TRUE`なので`stringsAsFactors = FALSE`を明示的に指定しないと後の処理に困るという注意点があります。

Shift_JISなファイルを読み込む場合は以下のようにオプションを指定します。

```r:R
read.csv("csvファイルのパス", fileEncoding = "Shift_JIS")
```

`fileEncoding = "cp932"`や`fileEncoding = "sjis"`等でも良いようです。

#### 推しポイント

-   追加パッケージなしで使える。

### fread

長い歴史のある`data.table`パッケージの関数で、巨大csvファイルを読み込む際の定番です。

注意点として、日本語環境のWindows版Rは4.2.0（2022年4月リリース）以降UTF-8になったわけですが、今回紹介する中で`fread()`だけはシステムのエンコーディングとUTF-8しか使用できないため、Windows上でもShist_JISのファイルを読み込めなくなってしまったようです。

#### 使い方

```r:R
data.table::fread(paths_csv_examples[1])
```

    #>         country year lifeExp     pop gdpPercap
    #> 1:      Algeria 1952  43.077 9279525 2449.0082
    #> 2:       Angola 1952  30.015 4232095 3520.6103
    #> 3:        Benin 1952  38.223 1738315 1062.7522
    #> 4:     Botswana 1952  47.622  442308  851.2411
    #> 5: Burkina Faso 1952  31.975 4469979  543.2552
    #> 6:      Burundi 1952  39.031 2445618  339.2965

csv文字列を直接渡してもちゃんと認識してくれます。

```r:R
"
foo,bar
1,a
2,b
" |>
  data.table::fread()
```

    #>    foo bar
    #> 1:   1   a
    #> 2:   2   b

区切り文字の自動判別機能もあります。

```r:R
"
foo   bar
  1     a
  2     b
" |>
  data.table::fread()
```

    #>    foo bar
    #> 1:   1   a
    #> 2:   2   b

`readLines()`関数などで取得できる、各要素を行とするベクトルのcsvデータを渡すこともできます（この形式は自動判別できないので`text`引数に明示的に渡す必要があります）。

```r:R
c("foo,bar", "1,a", "2,b") |>
  data.table::fread(text = _)
```

    #>    foo bar
    #> 1:   1   a
    #> 2:   2   b

#### 推しポイント

-   多機能。
-   速い。
-   高パフォーマンスを求めるユーザーに長年使用されているため情報が多い。

### read_csv

tidyverseコアパッケージの一つである`readr`の関数です。`readr::read_tsv()`等の兄弟もいます。

2021年7月にリリースされた`readr`バージョン2.0.0で中身が`vroom::vroom()`に切り替わり、読み込み速度が向上し、複数ファイルも読み込めるようになりました。

#### 使い方

```r:R
readr::read_csv(paths_csv_examples[1])
```

    #> Rows: 6 Columns: 5
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> chr (1): country
    #> dbl (4): year, lifeExp, pop, gdpPercap
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    #> # A tibble: 6 × 5
    #>   country       year lifeExp     pop gdpPercap
    #>   <chr>        <dbl>   <dbl>   <dbl>     <dbl>
    #> 1 Algeria       1952    43.1 9279525     2449.
    #> 2 Angola        1952    30.0 4232095     3521.
    #> 3 Benin         1952    38.2 1738315     1063.
    #> 4 Botswana      1952    47.6  442308      851.
    #> 5 Burkina Faso  1952    32.0 4469979      543.
    #> 6 Burundi       1952    39.0 2445618      339.

複数ファイルを読む場合は、ファイルパスの文字列を収納したベクトルを第1引数に渡すだけです。`id`オプションを使用することで、ファイルパスを列に追加できます。

```r:R
readr::read_csv(paths_csv_examples, id = "file_path")
```

    #> Rows: 26 Columns: 6
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> chr (1): country
    #> dbl (4): year, lifeExp, pop, gdpPercap
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    #> # A tibble: 26 × 6
    #>    file_path                                country  year lifeExp    pop gdpPe…¹
    #>    <chr>                                    <chr>   <dbl>   <dbl>  <dbl>   <dbl>
    #>  1 /usr/local/lib/R/site-library/readr/ext… Algeria  1952    43.1 9.28e6   2449.
    #>  2 /usr/local/lib/R/site-library/readr/ext… Angola   1952    30.0 4.23e6   3521.
    #>  3 /usr/local/lib/R/site-library/readr/ext… Benin    1952    38.2 1.74e6   1063.
    #>  4 /usr/local/lib/R/site-library/readr/ext… Botswa…  1952    47.6 4.42e5    851.
    #>  5 /usr/local/lib/R/site-library/readr/ext… Burkin…  1952    32.0 4.47e6    543.
    #>  6 /usr/local/lib/R/site-library/readr/ext… Burundi  1952    39.0 2.45e6    339.
    #>  7 /usr/local/lib/R/site-library/readr/ext… Argent…  1952    62.5 1.79e7   5911.
    #>  8 /usr/local/lib/R/site-library/readr/ext… Bolivia  1952    40.4 2.88e6   2677.
    #>  9 /usr/local/lib/R/site-library/readr/ext… Brazil   1952    50.9 5.66e7   2109.
    #> 10 /usr/local/lib/R/site-library/readr/ext… Canada   1952    68.8 1.48e7  11367.
    #> # … with 16 more rows, and abbreviated variable name ¹​gdpPercap

csv文字列を直接渡してもちゃんと認識してくれます。

```r:R
"
foo,bar
1,a
2,b
" |>
  readr::read_csv()
```

    #> Rows: 2 Columns: 2
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> chr (1): bar
    #> dbl (1): foo
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    #> # A tibble: 2 × 2
    #>     foo bar  
    #>   <dbl> <chr>
    #> 1     1 a    
    #> 2     2 b

`I()`でラップすることでリテラルデータであることを明示し、各要素を行とするベクトルのcsvデータを渡すこともできます。
`readr::read_lines()`で読み込んだテキストファイルから`stringr`パッケージの関数などで文字列処理しテーブル部分だけを抜き出した後に`readr::read_csv()`に渡して読み込めたりします。

```r:R
c("foo,bar", "1,a", "2,b") |>
  I() |>
  readr::read_csv()
```

    #> Rows: 2 Columns: 2
    #> ── Column specification ────────────────────────────────────────────────────────
    #> Delimiter: ","
    #> chr (1): bar
    #> dbl (1): foo
    #> 
    #> ℹ Use `spec()` to retrieve the full column specification for this data.
    #> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    #> # A tibble: 2 × 2
    #>     foo bar  
    #>   <dbl> <chr>
    #> 1     1 a    
    #> 2     2 b

Shift_JISなファイルを読み込む場合は以下のようにオプションを指定します。

```r:R
readr::read_csv("csvファイルのパス", locale = readr::locale(encoding = "Shift_JIS"))
```

`encoding = "cp932"`や`encoding = "sjis"`等でも良いようです。

#### 推しポイント

-   多機能。
-   複数ファイルを簡単に読み込める。
-   tidyverseのコアパッケージなので情報が多い。

### read_csv_arrow & open_dataset

Apache Arrow C++ライブラリによる読み込みを行う関数です。複数ファイルを読み込む場合は`read_csv_arrow()`の代わりに`open_dataset()`を使用します。

大量のオプションを指定可能なのですがR arrowパッケージのドキュメント上では多くは語られておらず、そもそも指定方法が難解です。
ほぼ同じ機能を持っているPythonの[`pyarrow.csv.read_csv()`関数](https://arrow.apache.org/docs/python/generated/pyarrow.csv.read_csv.html)等のドキュメントも確認すると良いです。

#### 使い方

デフォルトでは`readr::read_csv()`と同じようにtibble(`tbl_df`クラスオブジェクト)が返されます。

```r:R
arrow::read_csv_arrow(paths_csv_examples[1])
```

    #> # A tibble: 6 × 5
    #>   country       year lifeExp     pop gdpPercap
    #>   <chr>        <int>   <dbl>   <int>     <dbl>
    #> 1 Algeria       1952    43.1 9279525     2449.
    #> 2 Angola        1952    30.0 4232095     3521.
    #> 3 Benin         1952    38.2 1738315     1063.
    #> 4 Botswana      1952    47.6  442308      851.
    #> 5 Burkina Faso  1952    32.0 4469979      543.
    #> 6 Burundi       1952    39.0 2445618      339.

後の処理で`arrow`パッケージの関数を使用する場合は`as_data_frame = FALSE`を設定し、Apache Arrowフォーマットである`Table`クラスオブジェクトとして読み込めます。

```r:R
arrow::read_csv_arrow(paths_csv_examples[1], as_data_frame = FALSE)
```

    #> Table
    #> 6 rows x 5 columns
    #> $country <string>
    #> $year <int64>
    #> $lifeExp <double>
    #> $pop <int64>
    #> $gdpPercap <double>

Shift_JISなファイルを読み込む場合は以下のようにオプションを指定します。オプションの指定方法は分かり辛すぎますが、pyarrowと同じです。

```r:R
arrow::read_csv_arrow("csvファイルのパス", read_options = arrow::CsvReadOptions$create(encoding = "Shift_JIS"))
```

`encoding = "cp932"`や`encoding = "sjis"`等でも良いようです。

複数ファイルを読み込む場合は`arrow::open_dataset()`でオプション`format = "csv"`を指定します。

```r:R
arrow::open_dataset(paths_csv_examples, format = "csv")
```

    #> FileSystemDataset with 5 csv files
    #> country: string
    #> year: int64
    #> lifeExp: double
    #> pop: int64
    #> gdpPercap: double

この時点ではデータセットとしてスキーマ（列名と列の型）しか読み取っていません（余談：複数ファイルを読み込む場合、`readr::read_csv()`は列位置を合わせて読み込むのに対し`arrow::open_dataset()`は列名を合わせて読み込むためファイル間で列位置が入れ替わっていても正常に読み込めるようです）。

データセットを実際にデータフレームや`arrow::Table`としてメモリに読み込むときには、`dplyr::collect()`や`dplyr::compute()`を使用します（2023年1月リリース予定のバージョン11で`as.data.frame()`も使用可能になるようです）。

```r:R
arrow::open_dataset(paths_csv_examples, format = "csv") |>
  dplyr::compute()
```

    #> Table
    #> 26 rows x 5 columns
    #> $country <string>
    #> $year <int64>
    #> $lifeExp <double>
    #> $pop <int64>
    #> $gdpPercap <double>

ファイル名を列として追加するときには、`dplyr::mutate()`内で`arrow::add_filename()`を実行します。

```r:R
arrow::open_dataset(paths_csv_examples, format = "csv") |>
  dplyr::mutate(file_path = arrow::add_filename()) |>
  dplyr::collect()
```

    #> # A tibble: 26 × 6
    #>    country    year lifeExp      pop gdpPercap file_path                         
    #>    <chr>     <int>   <dbl>    <int>     <dbl> <chr>                             
    #>  1 Argentina  1952    62.5 17876956     5911. /usr/local/lib/R/site-library/rea…
    #>  2 Bolivia    1952    40.4  2883315     2677. /usr/local/lib/R/site-library/rea…
    #>  3 Brazil     1952    50.9 56602560     2109. /usr/local/lib/R/site-library/rea…
    #>  4 Canada     1952    68.8 14785584    11367. /usr/local/lib/R/site-library/rea…
    #>  5 Chile      1952    54.7  6377619     3940. /usr/local/lib/R/site-library/rea…
    #>  6 Colombia   1952    50.6 12350771     2144. /usr/local/lib/R/site-library/rea…
    #>  7 Algeria    1952    43.1  9279525     2449. /usr/local/lib/R/site-library/rea…
    #>  8 Angola     1952    30.0  4232095     3521. /usr/local/lib/R/site-library/rea…
    #>  9 Benin      1952    38.2  1738315     1063. /usr/local/lib/R/site-library/rea…
    #> 10 Botswana   1952    47.6   442308      851. /usr/local/lib/R/site-library/rea…
    #> # … with 16 more rows

#### 推しポイント

-   多機能。
-   速い。
-   複数ファイルを簡単に読み込める。

## 複数ファイルを読み込む方法

前述のように`readr::read_csv()`と`arrow::open_dataset()`にはそもそも複数ファイルを読み込む機能があるわけですが、それ以外の関数で複数ファイルを読み込みたい場合には以下のような方法を使用できます。

### map_dfr

tidyverseユーザーにとって、複数ファイルを読み込む場合の定番は`purrr::map_dfr()`関数と組み合わせる方法だと思います。[^3]

例えば`data.table::fread()`でサンプルcsvファイル5つを読み込んでファイルのパスを列として追加する場合にはこんな感じになります。

```r:R
paths_csv_examples |>
  purrr::map_dfr(~ data.table::fread(.x)[, file_path := .x])
```

今後もこの方法でも良いのですが、先日リリースされた`purrr` 1.0.0では`map_dfr()`関数の代わりに`purrr::map()`の後で`purrr::list_rbind()`を使う方法が推奨されるようになりました。
また、purrr-style lambdaと呼ばれる`purrr`等で使用されていた`~ fun(.x)`のような書き方よりも近年R本体に導入された`\(x) fun(x)`という関数の短縮記法が推奨され、`purrr`パッケージ内の例はすべてこの記法に置き換えられました。

というわけで、今後はこんな書き方が増えていくかもしれません。

```r:R
paths_csv_examples |>
  purrr::map(\(path) data.table::fread(path)[, file_path := path]) |>
  purrr::list_rbind()
```

    #>                    country year lifeExp       pop  gdpPercap
    #>  1:                Algeria 1952  43.077   9279525  2449.0082
    #>  2:                 Angola 1952  30.015   4232095  3520.6103
    #>  3:                  Benin 1952  38.223   1738315  1062.7522
    #>  4:               Botswana 1952  47.622    442308   851.2411
    #>  5:           Burkina Faso 1952  31.975   4469979   543.2552
    #>  6:                Burundi 1952  39.031   2445618   339.2965
    #>  7:              Argentina 1952  62.485  17876956  5911.3151
    #>  8:                Bolivia 1952  40.414   2883315  2677.3263
    #>  9:                 Brazil 1952  50.917  56602560  2108.9444
    #> 10:                 Canada 1952  68.750  14785584 11367.1611
    #> 11:                  Chile 1952  54.745   6377619  3939.9788
    #> 12:               Colombia 1952  50.643  12350771  2144.1151
    #> 13:            Afghanistan 1952  28.801   8425333   779.4453
    #> 14:                Bahrain 1952  50.939    120447  9867.0848
    #> 15:             Bangladesh 1952  37.484  46886859   684.2442
    #> 16:               Cambodia 1952  39.417   4693836   368.4693
    #> 17:                  China 1952  44.000 556263527   400.4486
    #> 18:       Hong Kong, China 1952  60.960   2125900  3054.4212
    #> 19:                Albania 1952  55.230   1282697  1601.0561
    #> 20:                Austria 1952  66.800   6927772  6137.0765
    #> 21:                Belgium 1952  68.000   8730405  8343.1051
    #> 22: Bosnia and Herzegovina 1952  53.820   2791000   973.5332
    #> 23:               Bulgaria 1952  59.600   7274900  2444.2866
    #> 24:                Croatia 1952  61.210   3882229  3119.2365
    #> 25:              Australia 1952  69.120   8691212 10039.5956
    #> 26:            New Zealand 1952  69.390   1994794 10556.5757
    #>                    country year lifeExp       pop  gdpPercap
    #>                                                                   file_path
    #>  1:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-africa.csv
    #>  2:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-africa.csv
    #>  3:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-africa.csv
    #>  4:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-africa.csv
    #>  5:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-africa.csv
    #>  6:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-africa.csv
    #>  7: /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-americas.csv
    #>  8: /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-americas.csv
    #>  9: /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-americas.csv
    #> 10: /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-americas.csv
    #> 11: /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-americas.csv
    #> 12: /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-americas.csv
    #> 13:     /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-asia.csv
    #> 14:     /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-asia.csv
    #> 15:     /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-asia.csv
    #> 16:     /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-asia.csv
    #> 17:     /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-asia.csv
    #> 18:     /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-asia.csv
    #> 19:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-europe.csv
    #> 20:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-europe.csv
    #> 21:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-europe.csv
    #> 22:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-europe.csv
    #> 23:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-europe.csv
    #> 24:   /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-europe.csv
    #> 25:  /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-oceania.csv
    #> 26:  /usr/local/lib/R/site-library/readr/extdata/mini-gapminder-oceania.csv
    #>                                                                   file_path

### summarise

`dplyr`の関数達を使って複数ファイルを読み込む方法もあります。

```r:R
paths_csv_examples |>
  tibble::as_tibble_col(column_name = "file_path") |>
  dplyr::rowwise() |>
  dplyr::summarise(
    file_path,
    data.table::fread(file_path),
    .groups = "drop"
  )
```

2020年5月にリリースされた`dplyr` 1.0.0において`dplyr::summarise()`内でデータフレームを返す関数を使えるようになった際、tidyverseブログで紹介された方法です。[^4]
経験上`purrr::map()`を使用するのと比べて遅いので私はあまり使いませんけれども、こちらの書き方の方が分かりやすいと感じる方もいらっしゃると思います。

2023年1月リリース予定の`dplyr` 1.1.0では`dplyr::summarise()`の中で複数行を返す関数を使用すると警告が出るようになり、代わりに`dplyr::reframe()`という関数が追加される予定です。[^5]

2022年末時点では`reframe()`という名前に関してフィードバック受付中で名前変更の可能性がないとは言えませんが[^6]、現時点での開発版`dplyr`ではこのようになります。

```r:R
paths_csv_examples |>
  tibble::as_tibble_col(column_name = "file_path") |>
  dplyr::rowwise() |>
  dplyr::reframe(
    file_path,
    data.table::fread(file_path),
  )
```

    #> # A tibble: 26 × 6
    #>    file_path                                country  year lifeExp    pop gdpPe…¹
    #>    <chr>                                    <chr>   <int>   <dbl>  <int>   <dbl>
    #>  1 /usr/local/lib/R/site-library/readr/ext… Algeria  1952    43.1 9.28e6   2449.
    #>  2 /usr/local/lib/R/site-library/readr/ext… Angola   1952    30.0 4.23e6   3521.
    #>  3 /usr/local/lib/R/site-library/readr/ext… Benin    1952    38.2 1.74e6   1063.
    #>  4 /usr/local/lib/R/site-library/readr/ext… Botswa…  1952    47.6 4.42e5    851.
    #>  5 /usr/local/lib/R/site-library/readr/ext… Burkin…  1952    32.0 4.47e6    543.
    #>  6 /usr/local/lib/R/site-library/readr/ext… Burundi  1952    39.0 2.45e6    339.
    #>  7 /usr/local/lib/R/site-library/readr/ext… Argent…  1952    62.5 1.79e7   5911.
    #>  8 /usr/local/lib/R/site-library/readr/ext… Bolivia  1952    40.4 2.88e6   2677.
    #>  9 /usr/local/lib/R/site-library/readr/ext… Brazil   1952    50.9 5.66e7   2109.
    #> 10 /usr/local/lib/R/site-library/readr/ext… Canada   1952    68.8 1.48e7  11367.
    #> # … with 16 more rows, and abbreviated variable name ¹​gdpPercap

グループ化は自動的に解除されるようになったので`dplyr::ungroup()`や`.groups = "drop"`を書く必要はなくなったのがちょっと嬉しいです。

## まとめ

巨大なcsvファイルは一刻も早くParquetに置換してしまいましょう。

## 実行環境

```r:R
sessionInfo()
```

    #> R version 4.2.2 (2022-10-31)
    #> Platform: x86_64-pc-linux-gnu (64-bit)
    #> Running under: Ubuntu 22.04.1 LTS
    #> 
    #> Matrix products: default
    #> BLAS:   /usr/lib/x86_64-linux-gnu/openblas-pthread/libblas.so.3
    #> LAPACK: /usr/lib/x86_64-linux-gnu/openblas-pthread/libopenblasp-r0.3.20.so
    #> 
    #> locale:
    #>  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
    #>  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
    #>  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
    #>  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
    #>  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
    #> [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
    #> 
    #> attached base packages:
    #> [1] stats     graphics  grDevices utils     datasets  methods   base     
    #> 
    #> loaded via a namespace (and not attached):
    #>  [1] pillar_1.8.1      compiler_4.2.2    tools_4.2.2       digest_0.6.31    
    #>  [5] bit_4.0.5         jsonlite_1.8.4    evaluate_0.19     lifecycle_1.0.3  
    #>  [9] tibble_3.1.8      pkgconfig_2.0.3   rlang_1.0.6       cli_3.5.0        
    #> [13] yaml_2.3.6        parallel_4.2.2    xfun_0.35         fastmap_1.1.0    
    #> [17] withr_2.5.0       stringr_1.5.0     dplyr_1.0.99.9000 knitr_1.41       
    #> [21] generics_0.1.3    vctrs_0.5.1       hms_1.1.2         bit64_4.0.5      
    #> [25] tidyselect_1.2.0  glue_1.6.2        data.table_1.14.6 R6_2.5.1         
    #> [29] fansi_1.0.3       vroom_1.6.0       rmarkdown_2.19    tzdb_0.3.0       
    #> [33] readr_2.1.3       purrr_1.0.0       magrittr_2.0.3    codetools_0.2-18 
    #> [37] ellipsis_0.3.2    htmltools_0.5.4   assertthat_0.2.1  arrow_10.0.1     
    #> [41] utf8_1.2.2        stringi_1.7.8     crayon_1.5.2

[^1]: [大量で巨大なcsvをRで読み込む時の最適解【vroom,readr,data.table】](https://qiita.com/Ringa_hyj/items/434e3a6794bb7ed8ee14)

[^2]: <https://twitter.com/MattDowle/status/1360073970498875394>、この投稿は`arrow`パッケージ開発者の方が「`read_arrow_csv()`は`fread()`より速い」と発表したことに対する反論で、`data.table`パッケージのNEWSからもリンクされています。

[^3]: [read all csv files in a directory using `purrr:map()`](https://stackoverflow.com/questions/62598787/read-all-csv-files-in-a-directory-using-purrrmap)

[^4]: [dplyr 1.0.0: new summarise() features](https://www.tidyverse.org/blog/2020/03/dplyr-1-0-0-summarise/#non-summaries)

[^5]: [dplyr 1.1.0 is coming soon](https://www.tidyverse.org/blog/2022/11/dplyr-1-1-0-is-coming-soon/#reframe-a-generalization-of-summarise)

[^6]: [Feedback for alternate names for `reframe()`](https://github.com/tidyverse/dplyr/issues/6565)
