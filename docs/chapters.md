




# 統計解析言語「R」

## 「R」とは

[`R`][R]は統計解析とグラフィックスのためのプログラミング言語及び環境です。オープンソースのフリーソフトウェアで、WindowsやMacOSなど様々なプラットフォームで利用することができます。そのため、コンピューターを所有している者なら誰でもRを使って分析方法を共有し、同じ結果を再現する可能性が高くなります。これは、科学的手法である「再現性」にも大いに役立ちます。

さらに、Rにはコード、データ、ドキュメント等が一つになったパッケージがあります。2023年3月現在でCRAN（the **C**omprehensive **R** **A**rchive **N**etworkの略）で利用可能なパッケージは19,000以上あり[@wickham_and_bryan_2023]、統計解析をはじめとしたデータ分析、機械学習、データの可視化など様々な分野の研究に対応しています。

例えば、本書でも利用する[`tidyverse`][tidyverse]に組み込まれた[`dplyr`][dplyr]はデータの編集に、[`ggplot2`][ggplot2]はデータの可視化に特化しており、これらパッケージを使用すれば、分析や研究を効率的に行うことができます。


[R]:https://www.r-project.org


## 「R」と「RStudio」の導入

Rは[`The R Project for Statistical Computing`][R]から最新版をダウンロードし、それぞれのパソコンの指示に従ってインストールします。

[`RStudio`][RStudio]はRの統合開発環境（IDE : **I**ntegrated **D**evelopment **E**nvironment）で、Rを使いやすくするデスクトップアプリケーションです。`RMarkdown`を使ってプログラミングコードとドキュメントを統合した文書を作成し、`Knitr`でhtmlやwordなどに書き出すことができます。本書では記載していませんが、GitHubと連携すればバージョン管理や共同執筆者との共有も可能となります。[公式サイト][RStudio]にアクセスし、インストーラーをダウンロードします（Figure\@ref(fig:RStudio)）。

<div class="figure">
<img src="analysis/figures/RStudio_install.png" alt="RStuioのダウンロード画面" width="952" />
<p class="caption">(\#fig:RStudio)RStuioのダウンロード画面</p>
</div>


[RStudio]:https://posit.co/download/rstudio-desktop/

# データの編集

## 概要

ここでは、準備したデータを取り込み、編集するための方法を紹介します。考古学研究でデータといえば、Microsoft社のExcelに記録したデータが思い浮かぶことでしょう。Excelは基本的な機能だけなら簡単な操作で表の作成・編集ができるだけでなく、広く使用されていることから複数の作業者間でのデータの共有等にも役に立つソフトです。一方、`神エクセル`[^1]という用語が端的に示すように、セルの書式をわざわざ文字列としたり、セルの結合や罫線機能などを駆使した見た目重視のファイルも散見され、データ分析に使用しにくいことも知られています。

[^1]: 「ネ申エクセル」ともいう。この弊害については奥村[-@okumura_2013]が詳しい。

RではいくつかのパッケージでExcelファイル（`.xlsx`の拡張子で表示されるもの）を直接取り込むこともできますが、上記の理由から使いづらいことが考えられます。そこで、本稿ではプレーンなテキスト形式で、さまざまなアプリやソフトで使用することができるCSVファイル（`.csv`の拡張子で表示されるもの）を利用することが良いと考えています。幸いExcelにはCSVファイルで保存する機能があるため、これまでの研究で蓄積したデータも利用することができます。

Rでデータを編集する際は[`tidyverse`][tidyverse]パッケージを使います。[`tidyverse`][tidyverse]はデータサイエンスのために設計されたRのパッケージ群で、[`ggplot2`][ggplot2]のほか、[`dplyr`][dplyr]、[`readr`][readr]、[`purrr`][purrr]、[`tibble`][tibble]、[`stringr`][stringr]、[`forcats`][forcats]といったパッケージが含まれています。これらは共通の基本的な設計思想、文法、データ構造を有しており、Rの標準的な表現方法（正規表現）よりも可読性の高いコードを用いてデータの前処理、可視化を実行することができます。Rでパッケージを読み込む際は`library()`関数を使います。

[tidyverse]:https://www.tidyverse.org
[ggplot2]:https://ggplot2.tidyverse.org
[dplyr]:https://dplyr.tidyverse.org
[readr]:https://readr.tidyverse.org
[purrr]:https://purrr.tidyverse.org
[tibble]:https://tibble.tidyverse.org
[stringr]:https://stringr.tidyverse.org
[forcats]:https://forcats.tidyverse.org


```r
# `library()`でパッケージを読み込む
library(tidyverse)
```

## 「R」の演算子について
本題に入る前に、Rでよく使う演算子の説明をしておきます。`<-`は代入演算子で、左辺に右辺を代入するために使用されます。数学の$x=$をイメージすると良いと思います。`<-`の代わりに`=`を使用することもできます。ここでは`fish`という箱に`test_data.csv`というファイルのデータを格納しています。

また`%>%`はパイプ演算子で、左辺の結果を右辺に受け渡す役割があり、コードを繋げて書くことができます。コードを記入するたびに適当な変数に代入する必要がなくなるため、可読性が高まり、効率的な作業を実現できます。["Plumbers, chains, and famous painters: The (updated) history of the pipe operator in R"][pipehist][@Adolfo_2019]によると、2012年1月17日に[`Stack Overflow`][stackoverflow]で原型が誕生し、Stefan Bache氏が2013年12月30日に`plumbr`パッケージを公開し（2日後に[`magrittr`][magrittr]に改名）、その後[`dplyr`][dplyr]のパイプ演算子への統合、`Rstudio`でのショートカットの採用（`Ctrl / Command + Shift + M`）を経て、その利便性が注目されて急速に普及しました。現在では[`magrittr`][magrittr]もしくは[`tidyverse`][tidyverse]パッケージを読み込めば使用できます。

2019年からはRの開発チームが`|>`（ベースパイプ）を開発し、2021年5月にリリースされた`R 4.1.0`で本格的に実装され、その後改良が重ねられて`%>%`と遜色ない機能に発展したため、現在ではユーザーのモードが`%>%`から`|>`に
移りつつあるようです。（本書では切り替えが大変なので`%>%`を利用します。）

[stackoverflow]:https://stackoverflow.com
[magrittr]:https://magrittr.tidyverse.org
[pipehist]:http://adolfoalvarez.cl/blog/2021-09-16-plumbers-chains-and-famous-painters-the-history-of-the-pipe-operator-in-r/

## tidy data

まず初めに、人間が見やすいデータ形式と、Rをはじめとしたプログラミング言語で機械処理しやすいデータ形式が異なるということを理解しておく必要があります。例えば生徒の名前と教科、点数からなる表があるとします。


Table: (\#tab:wide-format)横長型の表

|生徒 | 国語| 数学| 英語| 社会| 理科|
|:----|----:|----:|----:|----:|----:|
|A    |   85|   90|   75|   85|   82|
|B    |   92|   88|   80|   90|   85|
|C    |   78|   95|   85|   88|   90|
|D    |   88|   85|   78|   90|   92|
|E    |   79|   90|   85|   85|   88|
|F    |   95|   92|   90|   88|   85|

表\@ref(tab:wide-format)のような1つの観測値に複数の変数が含まれる表形式を表を横長型（Wide Format）といいます。人間がみる場合はこの表形式が見やすく、教科書や論文で掲載されるものもこのような形式です。

一方、表\@ref(tab:long-format)のような1つの観測値に1つの変数が含まれる表形式を縦長型（Long Format）といい、各列が異なる変数を示し、各行が1つの観測値（属性、数値）を表します。これが[`tidyverse`][tidyverse]でいう**tidy data**（整然データ）で、はじめHadley Wickhamが2014年に論文で定義が示され[@wickham_2014]、"R for Data Science"[@wickham_2017]、"R for Data Science (2e)"[@wickham_2023]で改訂されています（表\@ref(tab:definition)）。最新の定義を翻訳すると次の通りとなります。

> -   各変数は列であり、各列は変数である
> -   各観測値は行であり、各行は観測値である
> -   各変数はセルであり、各セルはひとつの値である\
>     -- [Wickhamほか2023](https://r4ds.hadley.nz/data-tidy) より翻訳


Table: (\#tab:definition)tidy dataの定義の変遷

|Source                   |Definition                                             |
|:------------------------|:------------------------------------------------------|
|Wickham 2014             |Each variable forms a column.                          |
|Wickham 2014             |Each observation forms a row.                          |
|Wickham 2014             |Each type of observational unit forms a table.         |
|Wickham & Grolemund 2017 |Each variable must have its own column.                |
|Wickham & Grolemund 2017 |Each observation must have its own row.                |
|Wickham & Grolemund 2017 |Each value must have its own cell.                     |
|Wickham et.al. 2023      |Each variable is a column; each column is a variable.  |
|Wickham et.al. 2023      |Each observation is a row; each row is an observation. |
|Wickham et.al. 2023      |Each value is a cell; each cell is a single value.     |

### tidy dataに変換する
横長型から縦長型に変換する時は、[`tidyr::pivot_longer`](https://tidyr.tidyverse.org/reference/pivot_longer.html)で変換することができます（表\@ref(tab:long-format)）。その逆を行いたい場合は、[`tidyr::pivot_wider`](https://tidyr.tidyverse.org/reference/pivot_wider.html)を使います（表\@ref(tab:wide-format2)）。


```r
student_data_tidy <- student_data %>%
  tidyr::pivot_longer(cols = !生徒,     # 動かすデータを選択（`!`で動かさないもの）
                      names_to = "科目",
                      values_to = "点数")

# htmlでキャプションを表示する場合。
# wordで書き出す場合は{r tab.cap=""}でいける。
student_data_tidy %>% head(n = 10) %>% knitr::kable(caption = "横長型から縦長型の表へ")
```



Table: (\#tab:long-format)横長型から縦長型の表へ

|生徒 |科目 | 点数|
|:----|:----|----:|
|A    |国語 |   85|
|A    |数学 |   90|
|A    |英語 |   75|
|A    |社会 |   85|
|A    |理科 |   82|
|B    |国語 |   92|
|B    |数学 |   88|
|B    |英語 |   80|
|B    |社会 |   90|
|B    |理科 |   85|


```r
student_data_tidy %>%
  tidyr::pivot_wider(names_from = 科目, values_from = 点数) %>%
  knitr::kable(caption = "縦長型から再び横長型の表へ")
```



Table: (\#tab:wide-format2)縦長型から再び横長型の表へ

|生徒 | 国語| 数学| 英語| 社会| 理科|
|:----|----:|----:|----:|----:|----:|
|A    |   85|   90|   75|   85|   82|
|B    |   92|   88|   80|   90|   85|
|C    |   78|   95|   85|   88|   90|
|D    |   88|   85|   78|   90|   92|
|E    |   79|   90|   85|   85|   88|
|F    |   95|   92|   90|   88|   85|



### クロス集計表の変換

クロス集計表は人間が見やすい横長型の表形式です。クロス集計表は考古学においても論文や報告書で掲載されていることがあり、データの概要を知るのには便利です。一方、自分で得たデータと比較のために再利用したい場合は一工夫必要で、縦長型に変換するだけでは不十分です。

Rでは[`tidyr::uncount(x)`][uncount]で"x"の数値の分、行を複製することができます。これでクロス集計表からヒストグラムなどを作成することが可能となります。あくまでクロス集計表から擬似的に作成したデータであり、生データではないので、先行研究で採用されているグラフの条件（例えばヒストグラムのビン幅など）を変更することは不適切です。

ここでは例として、村木[-@muraki_2004b]で公開されている奈良県天理市中山念仏寺墓地の背光五輪塔の年代と型式データから（Table\@ref(tab:cross-tab)）、縦長型の表に変換し（Table\@ref(tab:long-tab)）、ヒストグラムを再導出してみます（Figure\@ref(fig:cross-hist)）。ヒストグラムについては[後述](#ヒストグラム)も参照してください。

[uncount]:https://tidyr.tidyverse.org/reference/uncount.html


```r
# データは村木2004の表1（p30）の集計部分を使用
muraki2004 <- readr::read_csv("analysis/data/muraki2004.csv")

muraki2004 %>% head() %>% knitr::kable(caption = "奈良県天理市中山念仏寺墓地の背光五輪塔の年代と型式（村木2004より）")
```



Table: (\#tab:cross-tab)奈良県天理市中山念仏寺墓地の背光五輪塔の年代と型式（村木2004より）

| year_start| year_end| type1| type2| type3|
|----------:|--------:|-----:|-----:|-----:|
|       1531|     1540|     2|     0|     0|
|       1541|     1550|     0|     0|     0|
|       1551|     1560|     3|     0|     0|
|       1561|     1570|     2|     0|     0|
|       1571|     1580|    14|     0|     0|
|       1581|     1590|    10|     0|     0|


```r
# 縦長型に変換しuncount()で行の複製
muraki2004_long <- muraki2004 %>%
  tidyr::pivot_longer(cols = !c(year_start, year_end), 
                      names_to = "Type",
                      values_to = "N") %>%
  tidyr::uncount(N) %>%                  # Nの数値分、行を追加する。
  dplyr::mutate(dplyr::across(Type, factor))

muraki2004_long %>% head() %>% knitr::kable(caption = "`tidyr::pivot_longer()`と`tidyr::uncount()`で縦長型に変換")
```



Table: (\#tab:long-tab)`tidyr::pivot_longer()`と`tidyr::uncount()`で縦長型に変換

| year_start| year_end|Type  |
|----------:|--------:|:-----|
|       1531|     1540|type1 |
|       1531|     1540|type1 |
|       1551|     1560|type1 |
|       1551|     1560|type1 |
|       1551|     1560|type1 |
|       1561|     1570|type1 |



```r
muraki2004_long %>%
  ggplot() +
  ggplot2::geom_histogram(aes(x = year_start, 
                              y = after_stat(count),
                              fill = Type),
                          binwidth = 10,    # bin幅の設定
                          position = "identity",
                          # alpha = 0.2,        # 透明度の設定
                          colour = "black",     # グラフの枠線の色
                          boundary = 0) +       # ビンの境界を指定
  scale_x_continuous(breaks = seq(1530, 1770, 20), 
                     limits=c(1530, 1770)) +        # x軸の範囲を設定
  facet_wrap(~Type,
             ncol = 1) +
  labs(x = "年代", y = "個数") +
  theme_bw(base_family = "HiraginoSans-W3")
```

<div class="figure">
<img src="chapters_files/figure-html/cross-hist-1.png" alt="クロス集計表から再構成したヒストグラム" width="672" />
<p class="caption">(\#fig:cross-hist)クロス集計表から再構成したヒストグラム</p>
</div>


データの前処理や操作を行うには[`dplyr`][dplyr]パッケージが便利です。データフレームなどに対して、データの選択や絞り込み、集約といった様々な操作をすることができます。ここでは、主要な機能を使ってデータフレームの加工を行い、実際の分析使用する形にしていきます。

## データを取り込む

Rで`.xlsx`ファイルをそのまま読み込むパッケージもありますが、macでは正常に機能しないので、今回は`.csv`に変換したものを読み込みます。


```r
#### データの読み込み ####
#  `readr::read_csv()`は`readr`パッケージの機能。`readr::`は、書かなくても機能します。
# Rの標準関数の`read.csv()`でも同様なことができます。

fish <- readr::read_csv("analysis/data/testdata2.csv")

#### 簡単な編集をしておく ####
# `stringi::stri_trans_nfkc`関数で全角を半角にする。
library(stringi)

fish <- fish %>%
  dplyr::mutate(dplyr::across(dplyr::everything(), ~ stringi::stri_trans_nfkc(.)))

# データの型を変更
fish <- fish %>%
  dplyr::mutate(dplyr::across(c("Period", "Fish", "Bone", "Site", "Direction"), as.factor)) %>%
  dplyr::mutate(dplyr::across(PML, as.numeric))
```


## `filter()`による行の絞り込み

[`dplyr::filter()`][filter]関数は、条件を満たす行のみにデータを絞り込むものです。条件が一致するものは`=`でなく`==`である点に注意が必要です。また、不等号で数値の絞り込みもできます。

[filter]:https://dplyr.tidyverse.org/reference/filter.html

### 一条件で絞り込む


```r
# `head()`でデータフレームの行頭を表示する。
# `tail()`にするとデータフレームの行末を表示する。
fish %>% dplyr::filter(Period == "2期") %>% head()
```

```
## # A tibble: 6 × 6
##   Fish       Bone     Direction Period   PML Site      
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 クロダイ属 前上顎骨 右        2期       NA とある貝塚
## 2 クロダイ属 前上顎骨 右        2期       NA とある貝塚
## 3 クロダイ属 歯骨     右        2期       NA とある貝塚
## 4 クロダイ属 歯骨     右        2期       NA とある貝塚
## 5 マダイ     前上顎骨 右        2期       NA とある貝塚
## 6 タイ科     前上顎骨 右        2期       NA とある貝塚
```


```r
# 大きさ30mm以上に絞り込む
fish %>% dplyr::filter(PML >= 30.0) %>% head()
```

```
## # A tibble: 6 × 6
##   Fish       Bone     Direction Period   PML Site      
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 クロダイ属 前上顎骨 左        1期     30.2 とある貝塚
## 2 クロダイ属 前上顎骨 右        2期     31.1 とある貝塚
## 3 クロダイ属 前上顎骨 左        2期     35.5 とある貝塚
## 4 クロダイ属 前上顎骨 右        2期     32   とある貝塚
## 5 クロダイ属 前上顎骨 右        2期     32.6 とある貝塚
## 6 クロダイ属 前上顎骨 左        2期     30.2 とある貝塚
```


```r
# `!`で否定条件を指定できる。
fish %>% dplyr::filter(Period != "1期") %>% head()
```

```
## # A tibble: 6 × 6
##   Fish       Bone          Direction Period   PML Site      
##   <fct>      <fct>         <fct>     <fct>  <dbl> <fct>     
## 1 タイ科     前上顎骨/歯骨 不明      3期       NA とある貝塚
## 2 クロダイ属 前上顎骨      右        2期       NA とある貝塚
## 3 クロダイ属 前上顎骨      右        2期       NA とある貝塚
## 4 クロダイ属 歯骨          右        2期       NA とある貝塚
## 5 クロダイ属 歯骨          右        2期       NA とある貝塚
## 6 マダイ     前上顎骨      右        2期       NA とある貝塚
```

### 複数条件による絞り込み

条件を「どちらも満たす（and）」場合は`&`や`,`を使用します。条件を「いずれかを満たす（or）」場合は`|`を、`!`を使って反転させると条件を「いずれも満たさない」行に絞り込むことができます。


```r
# PMLが30cm以上、時期が`2期`のものに絞り込む
fish %>% dplyr::filter(Period == "2期" & PML >= 30.0) %>% head()
```

```
## # A tibble: 6 × 6
##   Fish       Bone     Direction Period   PML Site      
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 クロダイ属 前上顎骨 右        2期     31.1 とある貝塚
## 2 クロダイ属 前上顎骨 左        2期     35.5 とある貝塚
## 3 クロダイ属 前上顎骨 右        2期     32   とある貝塚
## 4 クロダイ属 前上顎骨 右        2期     32.6 とある貝塚
## 5 クロダイ属 前上顎骨 左        2期     30.2 とある貝塚
## 6 クロダイ属 前上顎骨 左        2期     34.1 とある貝塚
```


```r
# PMLが30cm以上または時期が`2期`に絞り込む
fish %>% dplyr::filter(Period == "2期" | PML >= 30.0) %>% head()
```

```
## # A tibble: 6 × 6
##   Fish       Bone     Direction Period   PML Site      
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 クロダイ属 前上顎骨 右        2期       NA とある貝塚
## 2 クロダイ属 前上顎骨 右        2期       NA とある貝塚
## 3 クロダイ属 歯骨     右        2期       NA とある貝塚
## 4 クロダイ属 歯骨     右        2期       NA とある貝塚
## 5 マダイ     前上顎骨 右        2期       NA とある貝塚
## 6 タイ科     前上顎骨 右        2期       NA とある貝塚
```


```r
# PMLが30cm以上または時期が`2期`以外のものに絞り込む
fish %>% dplyr::filter(!(Period == "2期" | PML >= 30.0)) %>% head()
```

```
## # A tibble: 6 × 6
##   Fish       Bone     Direction Period   PML Site      
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 クロダイ属 前上顎骨 左        1期     20.6 とある貝塚
## 2 クロダイ属 前上顎骨 右        1期     29.6 とある貝塚
## 3 クロダイ属 前上顎骨 右        1期     20.4 とある貝塚
## 4 クロダイ属 前上顎骨 左        1期     29.5 とある貝塚
## 5 クロダイ属 前上顎骨 右        1期     24.4 とある貝塚
## 6 クロダイ属 前上顎骨 右        1期     27.1 とある貝塚
```

## `select()`による列の操作

[`dplyr::select()`][select]関数を使うことで、データフレームから特定の列を選択したり、列名の変更ができます。Rの標準機能よりも直接的に指定することができます。

[select]:https://dplyr.tidyverse.org/reference/select.html

### 列を抽出、削除する


```r
#列を抽出する。
fish %>% dplyr::select(Fish,Period) %>% head()
```

```
## # A tibble: 6 × 2
##   Fish       Period
##   <fct>      <fct> 
## 1 マダイ     1期   
## 2 クロダイ属 1期   
## 3 コイ科     1期   
## 4 サケ属     1期   
## 5 サケ属     1期   
## 6 サメ類     1期
```

`:`を使うことでまとめて選択することもできます。


```r
# Site列からPeriod列までをまとめて抽出する。
fish %>% dplyr::select(Bone:Period) %>% head()
```

```
## # A tibble: 6 × 3
##   Bone     Direction Period
##   <fct>    <fct>     <fct> 
## 1 椎骨     中        1期   
## 2 前上顎骨 左        1期   
## 3 主鰓蓋骨 右        1期   
## 4 椎骨     中        1期   
## 5 椎骨     中        1期   
## 6 椎骨     中        1期
```

列名の前に`-`を加えることで、列を削除することができます。


```r
# 不要な列を削除する。
fish %>% dplyr::select(-Direction) %>% head()
```

```
## # A tibble: 6 × 5
##   Fish       Bone     Period   PML Site      
##   <fct>      <fct>    <fct>  <dbl> <fct>     
## 1 マダイ     椎骨     1期       NA とある貝塚
## 2 クロダイ属 前上顎骨 1期       NA とある貝塚
## 3 コイ科     主鰓蓋骨 1期       NA とある貝塚
## 4 サケ属     椎骨     1期       NA とある貝塚
## 5 サケ属     椎骨     1期       NA とある貝塚
## 6 サメ類     椎骨     1期       NA とある貝塚
```

### 列の順番を入れ替える


```r
# 列の順番を入れ替える。
# dplyr::everything()でその他の列をそのまま選択。
fish %>% 
  dplyr::select(Site, dplyr::everything()) %>%
  head()
```

```
## # A tibble: 6 × 6
##   Site       Fish       Bone     Direction Period   PML
##   <fct>      <fct>      <fct>    <fct>     <fct>  <dbl>
## 1 とある貝塚 マダイ     椎骨     中        1期       NA
## 2 とある貝塚 クロダイ属 前上顎骨 左        1期       NA
## 3 とある貝塚 コイ科     主鰓蓋骨 右        1期       NA
## 4 とある貝塚 サケ属     椎骨     中        1期       NA
## 5 とある貝塚 サケ属     椎骨     中        1期       NA
## 6 とある貝塚 サメ類     椎骨     中        1期       NA
```

### 列のリネーム

[`dplyr::select()`][select]関数では列名の変更もできますが、`dplyr::everything()`とセットで使用する必要があります。[`dplyr::rename()`][rename]を使った方が直感的かつ省エネで変更できます。

[select]:https://dplyr.tidyverse.org/reference/select.html
[rename]:https://dplyr.tidyverse.org/reference/rename.html


```r
# "新しい列名 = 修正したい列名"でリネームが可能。
# `dplyr::select()`を使う場合は`dplyr::everything()`が必須。
fish %>% 
  dplyr::select(魚種 = Fish, dplyr::everything()) %>%
  dplyr::rename(遺跡 = Site, 骨の名称 = Bone) %>%
  head()
```

```
## # A tibble: 6 × 6
##   魚種       骨の名称 Direction Period   PML 遺跡      
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 マダイ     椎骨     中        1期       NA とある貝塚
## 2 クロダイ属 前上顎骨 左        1期       NA とある貝塚
## 3 コイ科     主鰓蓋骨 右        1期       NA とある貝塚
## 4 サケ属     椎骨     中        1期       NA とある貝塚
## 5 サケ属     椎骨     中        1期       NA とある貝塚
## 6 サメ類     椎骨     中        1期       NA とある貝塚
```

## `mutate()`による列の編集

[`dplyr`][dplyr]パッケージにおいて、[`dplyr::filter()`][filter]と並んでよく使う関数が[`dplyr::mutate()`][mutate]です。新たに変数を追加したり、既存の変数を置き換えたりすることなどができます。

列の型も[`dplyr::mutate()`][mutate]を使って変更することができます。複数の列を選択する際には[`dplyr::mutate_at()`][mutate_at]を使う方法もありますが、[`dplyr::across()`][across]を併用すれば、[`dplyr::mutate()`][mutate]だけで対応することができます。

[mutate]:https://dplyr.tidyverse.org/reference/mutate.html
[mutate_at]:https://dplyr.tidyverse.org/reference/mutate_all.html?q=mutate_at#grouping-variables
[across]:https://dplyr.tidyverse.org/reference/across.html

### mutate( )による列の追加

[`dplyr::mutate()`][mutate]関数を使用して列を追加します。また、同時に列の型を設定することもできます。



```r
fish %>% 
  # 列を追加し、それぞれ文字を入れる
  dplyr::mutate(時代 = "縄文時代前期", 地域 = "北陸") %>% head()
```

```
## # A tibble: 6 × 8
##   Fish       Bone     Direction Period   PML Site       時代         地域 
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>      <chr>        <chr>
## 1 マダイ     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 2 クロダイ属 前上顎骨 左        1期       NA とある貝塚 縄文時代前期 北陸 
## 3 コイ科     主鰓蓋骨 右        1期       NA とある貝塚 縄文時代前期 北陸 
## 4 サケ属     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 5 サケ属     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 6 サメ類     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸
```


```r
#### 「時代」のみfctrに変更する場合 ####
fish %>%
  dplyr::mutate(時代 = factor("縄文時代前期"), 地域 = "北陸") %>% head()
```

```
## # A tibble: 6 × 8
##   Fish       Bone     Direction Period   PML Site       時代         地域 
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>      <fct>        <chr>
## 1 マダイ     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 2 クロダイ属 前上顎骨 左        1期       NA とある貝塚 縄文時代前期 北陸 
## 3 コイ科     主鰓蓋骨 右        1期       NA とある貝塚 縄文時代前期 北陸 
## 4 サケ属     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 5 サケ属     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 6 サメ類     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸
```

```r
#### 列の型を文字型（chr）から因子型（fctr）にまとめて変更 ####
fish %>% 
  dplyr::mutate(時代 = "縄文時代前期", 地域 = "北陸") %>%
  dplyr::mutate(dplyr::across(c(時代, 地域), factor)) %>% head()
```

```
## # A tibble: 6 × 8
##   Fish       Bone     Direction Period   PML Site       時代         地域 
##   <fct>      <fct>    <fct>     <fct>  <dbl> <fct>      <fct>        <fct>
## 1 マダイ     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 2 クロダイ属 前上顎骨 左        1期       NA とある貝塚 縄文時代前期 北陸 
## 3 コイ科     主鰓蓋骨 右        1期       NA とある貝塚 縄文時代前期 北陸 
## 4 サケ属     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 5 サケ属     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸 
## 6 サメ類     椎骨     中        1期       NA とある貝塚 縄文時代前期 北陸
```


### 場合分けによる列の編集

[`dplyr::case_when()`][case_when]や[`dplyr::if_else()`][if_else]を組み合わせることで、条件を満たす列を作成することもできます。既存の列にある変数を編集（置き換え）することも可能です。

[case_when]:https://dplyr.tidyverse.org/reference/case_when.html
[if_else]:https://dplyr.tidyverse.org/reference/if_else.html


```r
fish %>% 
  filter(PML > 0 & Fish == "クロダイ属") %>%
  # `case_when()`で場合分けをして、時代という列を作成する。
  # 最後を`TRUE ~ NA`で一致しないものを。
  dplyr::mutate(Period = factor(
    case_when(
      Period == "1期" ~ "縄文1期", 
      Period == "2期" ~ "縄文2期", 
      # `grepl`は正規表現で、文字列が一致するものをTRUE、FALSEを返す。
      grepl("3期", Period) ~ "縄文3期",
      TRUE ~ NA))
    )%>%
  # `if_else()`で場合分けをして、大きさという列を作成する。
  dplyr::mutate(Size = if_else(PML < 30.0, "small", "Large")) %>%
  head()
```

```
## # A tibble: 6 × 7
##   Fish       Bone     Direction Period    PML Site       Size 
##   <fct>      <fct>    <fct>     <fct>   <dbl> <fct>      <chr>
## 1 クロダイ属 前上顎骨 左        縄文1期  20.6 とある貝塚 small
## 2 クロダイ属 前上顎骨 右        縄文1期  29.6 とある貝塚 small
## 3 クロダイ属 前上顎骨 右        縄文1期  20.4 とある貝塚 small
## 4 クロダイ属 前上顎骨 左        縄文1期  29.5 とある貝塚 small
## 5 クロダイ属 前上顎骨 右        縄文1期  24.4 とある貝塚 small
## 6 クロダイ属 前上顎骨 右        縄文1期  27.1 とある貝塚 small
```

### 文字列の置き換え

[`dplyr::case_when()`][case_when]以外にも[`dplyr::mutate()`][mutate]関数と組み合わせることで、文字列の置き換えをすることができます。特に[`stringr::str_replace()`][str_replace]はより明示的に文字の置き換えをすることができます。

[str_replace]:https://stringr.tidyverse.org/reference/str_replace.html


```r
fish %>%  filter(Fish == "クロダイ属") %>%
  dplyr::mutate(Site = stringr::str_replace(Site, "とある", "ござる")) %>%
  dplyr::mutate(Fish = stringr::str_replace(Fish, "クロダイ属", "クロダイ")) %>%
  # 型がchr型になってしまうので、factor型に戻す。
  dplyr::mutate(dplyr::across(c(Fish, Site), as.factor)) %>%
  head()
```

```
## # A tibble: 6 × 6
##   Fish     Bone     Direction Period   PML Site      
##   <fct>    <fct>    <fct>     <fct>  <dbl> <fct>     
## 1 クロダイ 前上顎骨 左        1期       NA ござる貝塚
## 2 クロダイ 前上顎骨 右        1期       NA ござる貝塚
## 3 クロダイ 前上顎骨 右        1期       NA ござる貝塚
## 4 クロダイ 歯骨     左        1期       NA ござる貝塚
## 5 クロダイ 前上顎骨 右        2期       NA ござる貝塚
## 6 クロダイ 前上顎骨 右        2期       NA ござる貝塚
```

## `summarize()`による概要表{#概要表}

Rでは`summary()`関数で各列の個数や平均値といったデータの概要を知ることができます。基本的な統計情報を概要表（summary table）としてみることができます。

Table\@ref(tab:summarize)のように、`dplyr::group_by()`でグループ化した変数についての必要な要約統計量を、[`dplyr::summarize()`][summarize]関数で概要表を作成することもできます。

概要表を作成することで全体の傾向や特性といったデータの特徴を理解しやすくなり、さらにはデータを比較する場合でも数値的な違いを把握することができます。

[summarize]:https://dplyr.tidyverse.org/reference/summarise.html



```r
fish %>% summary()
```

```
##          Fish            Bone      Direction       Period          PML       
##  スズキ属  :3054   前上顎骨:2252   -   :  95   1期    :3675   Min.   :15.50  
##  クロダイ属:2900   椎骨    :1974   不明:1243   1期-2期:2158   1st Qu.:24.90  
##  マダイ    :1197   歯骨    :1946   中  :2754   1期-3期: 813   Median :27.90  
##  タイ科    :1077   主鰓蓋骨:1910   右  :4018   2期    :4787   Mean   :27.83  
##  フグ科    : 794   主上顎骨: 603   左  :4200   3期    : 777   3rd Qu.:30.70  
##  コイ科    : 780   角骨    : 486               不明   : 100   Max.   :40.20  
##  (Other)   :2508   (Other) :3139                              NA's   :11786  
##          Site      
##  とある貝塚:12310  
##                    
##                    
##                    
##                    
##                    
## 
```



```r
fish %>%
  dplyr::filter(Period %in% c("1期", "2期", "3期") & PML > 0) %>%
  dplyr::group_by(Period) %>%
  dplyr::summarize(
    count = as.numeric(n()),       # 個数
    mean = round(mean(PML), 1),    # 平均
    median = round(median(PML),1), # 中央値
    var. = round(var(PML), 1),     # 分散
    S.D. = round(sd(PML), 1),      # 標準偏差
    .groups   = "drop") %>%        # グルーピング解除
  data.frame() %>% 
  knitr::kable(caption = "`dplyr::summarize()`による概要表")
```



Table: (\#tab:summarize)`dplyr::summarize()`による概要表

|Period | count| mean| median| var.| S.D.|
|:------|-----:|----:|------:|----:|----:|
|1期    |   172| 28.8|   29.1| 14.3|  3.8|
|2期    |   209| 27.3|   27.5| 23.1|  4.8|
|3期    |    30| 26.6|   26.0| 19.5|  4.4|

# ggplot2を使用したデータビジュアライゼーション

## データ可視化の重要性

データの特徴を知るために概要表を作成が重要であることは前章で示しましたが、それだけでは十分ではありません。インターネットでデータの可視化の重要性を検索すると様々なメリットを知ることができます。論文等で明示されることは少ないようですが、鈴木・鈴村[-@suzuki_suzumura_2015]では以下の6点が指摘されています。

> - 「データの持つ意味」を発見しやすくなる  
> - 状態（状況）の特徴が把握しやすくなる  
> - 数量の比較がしやすくなる  
> - 相関性が把握しやすくなる  
> - 変化（推移）が把握しやすくなる  
> --鈴木・鈴村[-@suzuki_suzumura_2015]pp.474より

[`ggplot2`][ggplot2]の開発者らは、有名な統計学者のTukey氏の次の主張を引用しています[@wickham_2017]。

> The simple graph has brought more information to the data analyst's mind than any other device.  
> （意訳）シンプルな図は、他のどんな装置よりも多くの情報を私たちにもたらした。  
> -- Tukey[-@tukey_1962]pp.49より

Tukey氏は「探索的データ解析（EDA:Explorary Data Analysis）」を提唱し、「箱ひげ図」などの直感的でわかりやすいグラフを提示したことで知られています。これらのメッセージからも可視化の重要性、メリットが端的に理解できるでしょう。

さて話題が脱線しつつありますが、ここで要約統計量だけでなく、データ可視化が重要であることを理解するために、[`Alberto Cairo`][Cairo]が作成・公開している[`datasauRus`][datasauRus]パッケージを利用します。このパッケージには13のグループと$x$と$y$データからなるデータセットが含まれており、それらは同じ平均と標準偏差、ピアソン相関係数を示しています。このうち3つのグループの要約統計量を示したものが表\@ref(tab:datasaurus-summary)です。このデータをプロットすると、恐竜 （*Anscombosaurus*[^Anscombosaurus]）や星形、X字形が現れ、要約統計量は一致するにも関わらず、データの分布は全く異なることがわかります（Figure\@ref(fig:datasaurus-plot)）。[Same Stats, Different Graphs][Same Stats, Different Graphs]にアニメーションも掲載されているので、理解が深まるでしょう。

このように、平均や標準偏差といった基本的な要約統計量だけではデータ全体を把握することはできません。データの分布を可視化することで、より多くの情報を把握することができます。

[^Anscombosaurus]:統計学者Francis Anscombe氏のAnscombe's quartet（「アンスコムの例」）[@Anscombe_1973]に因んで命名された。

[datasauRus]:https://cran.r-project.org/web/packages/datasauRus/vignettes/Datasaurus.html
[Cairo]:http://www.thefunctionalart.com/2016/08/download-datasaurus-never-trust-summary.html
[Same Stats, Different Graphs]:https://www.research.autodesk.com/publications/same-stats-different-graphs/


Table: (\#tab:datasaurus-summary)概要表

|dataset | x_mean| y_mean| x_S.D.| y_S.D.|
|:-------|------:|------:|------:|------:|
|dino    |   54.3|   47.8|   16.8|   26.9|
|star    |   54.3|   47.8|   16.8|   26.9|
|x_shape |   54.3|   47.8|   16.8|   26.9|

<div class="figure">
<img src="chapters_files/figure-html/datasaurus-plot-1.png" alt="可視化の重要性" width="672" />
<p class="caption">(\#fig:datasaurus-plot)可視化の重要性</p>
</div>


可視化には[`ggplot2`][ggplot2]パッケージを使います。[`ggplot2`][ggplot2]では、レイヤーを重ねて、一つのグラフを作成します。レイヤーは、作成したグラフのデータ、グラフの種類（`geom_*()`関数）、見た目や位置調整などを組み合わせるもので、それぞれ`+`で連結させていきます。


## 見た目を設定する

### 日本語フォントを探す

[`ggplot2`][ggplot2]で可視化するデータおよび図で日本語が含まれる場合、そのままの設定では表示されないので、あらかじめ使用するフォントを明示する必要があります。

`systemfonts`パッケージの[`systemfonts::system_fonts()`][system_fonts]関数を使えば、システム中のフォントを調べることができます。この後`theme_*()`で使用する引数`base_family`で各自のPCにある日本語フォントを指定することで表示することができます。


```r
# 日本語フォントを探し設定する。
library(systemfonts)
fonts <- systemfonts::system_fonts()
```


```r
# 基本となる背景を作成
base <- data.frame(x = c(0, 1.5, 3), 
                   y = c(0, 1.5, 3),
                   type = c("A", "B", "C")) %>%
  ggplot() +
  geom_point(aes(x, y, color = type, fill = type)) +
  labs(x = "x軸の名称を設定",
       y = "y軸の名称を設定",
       title = "theme( )での設定",
       subtitle = "サブタイトル",
       colour = "凡例の名称を設定",   # `colour`で設定したものの名称変更
       fill = "凡例の名称を設定",     # `fill`で設定したものの名称変更
       caption = "(このデータは...)")

base +
  labs(x = "日本語", y = "個数",
       title = "`base_family =`で日本語を設定できる") +
    theme_bw()

# 日本語に設定
base +
  labs(x = "日本語", y = "個数",
       title = "`base_family =`で日本語を設定できる") +
  theme_bw(base_family = "HiraginoSans-W3")         # 日本語を使用する場合
```

<img src="chapters_files/figure-html/unnamed-chunk-19-1.png" width="48%" /><img src="chapters_files/figure-html/unnamed-chunk-19-2.png" width="48%" />

### 見た目の設定
[`theme_*()`][theme_]は図の背景を設定することができます（Figure\@ref(fig:theme-plot)）。デフォルトでは`theme_grey`が設定されており、背景がグレーのタイルで表示されます。一般的な論文だとモノクロ印刷であるため`theme_bw()`、`theme_minimal()`、`theme_classic()`が適しています。この[`theme_*()`][theme_]で使用できる引数は`base_size`（文字の大きさ）、`base_family`（フォント）、`base_line_size`（罫線）、`base_rect_size`（枠線）の4つに限られています。

[`theme()`][theme]を用いると、タイトル・ラベル・フォント・背景・罫線・凡例をさらに細かく設定でき、引数はそれに応じて多岐に渡ります（Figure\@ref(fig:theme-plot2)）。[`element_*()`][element_]とセットで使うことが多く、`element_blank()`（何も表示せず、スペースも差し込まない）、`element_rect()`（罫線と背景）、`element_line()`（線関係）、`element_text()`（文字関係）の4つがあります。

[`theme_*()`][theme_]はセット販売とすれば、[`theme()`][theme]は単品販売のようなものでしょうか。[`theme_*()`][theme_]は[`theme()`][theme]で上書きすることも可能ですので、基本的には[`theme_*()`][theme_]を使い、必要に応じて変更するとよいと思います。表示にこだわりがある場合や、[`theme_*()`][theme_]が論文の投稿規定等に合致しない場合などは、[`theme()`][theme]を使って詳細に設定するとよいでしょう（Figure\@ref(fig:my-theme)）。

[`ggplot2`][ggplot2]の制作者らが["ggplot2: Elegant Graphics for Data Analysis"で解説][theme-books]を公開しているので参照してください[@wickham_2016]。


[theme_]:https://ggplot2.tidyverse.org/reference/ggtheme.html
[theme]:https://ggplot2.tidyverse.org/reference/theme.html
[system_fonts]:https://cran.r-project.org/web/packages/systemfonts/systemfonts.pdf
[element_]:https://ggplot2.tidyverse.org/reference/element.html
[theme-books]:https://ggplot2-book.org/themes

<div class="figure">
<img src="chapters_files/figure-html/theme-plot-1.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-2.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-3.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-4.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-5.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-6.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-7.png" alt="`theme_*()`の表示例" width="48%" /><img src="chapters_files/figure-html/theme-plot-8.png" alt="`theme_*()`の表示例" width="48%" />
<p class="caption">(\#fig:theme-plot)`theme_*()`の表示例</p>
</div>


```r
# 基本のプロットに`theme()`を設定する
base +
  labs(x = "x軸の名称を設定",
       y = "y軸の名称を設定",
       title = "theme( )での設定") +
  theme(title = element_text(family = "Osaka",
                             face = "bold",
                             colour = "blue",
                             size = 14),   # タイトルの設定
        axis.title.x = element_text(family = "HiraMaruProN-W4",
                                    colour = "red",
                                    size = 11),   # x軸の名前の設定
        axis.title.y = element_text(family = "HiraMaruProN-W4",
                                    colour = "green"), # y軸の名前を消す
        panel.grid.major = element_line(linetype = "solid",
                                        colour =  "black",
                                        linewidth = 0.2),   # 罫線の主線の設定
        panel.grid.minor = element_line(linetype = "dotted",
                                        colour =  "black",
                                        linewidth = 0.2)    # 罫線の副線の設
  )
```

<div class="figure">
<img src="chapters_files/figure-html/theme-plot2-1.png" alt="`theme()`の設定例" width="672" />
<p class="caption">(\#fig:theme-plot2)`theme()`の設定例</p>
</div>


```r
# `my_theme`に格納しておけば労力を省略できる。
my_theme <- 
  theme(text = element_text(family = "HiraginoSans-W3",
                            size = 9),    # 日本語フォントの設定
        title = element_text(size = 11),   # タイトルの設定
        # 軸（axis）関係
        axis.title = element_text(colour = "black",
                                  size = 9),   # 軸の名称の設定
        panel.grid.major = element_line(linetype = "solid",
                                        colour =  "grey",
                                        linewidth = 0.2),   # 罫線の主線の設定
        panel.grid.minor = element_line(linetype = "dotted",
                                        colour =  "grey",
                                        linewidth = 0.2),   # 罫線の副線の設定
        panel.background = element_blank(),                 # 背景を空白に設定
        panel.border = element_rect(linetype = "solid",
                                    linewidth = 0.5,
                                    fill = NA),
        # 凡例（legend）関係
        legend.direction = "horizontal",         # 水平にする
        legend.position = "bottom",              # 凡例の位置
        legend.background = element_blank(), 
        # キャプション（caption）関係
        plot.caption = element_text(size = 8),
        # facet関係
        strip.background = element_rect(fill = "grey90")  # `facet_wrap`等のタイトル
        )

base + my_theme
```

<div class="figure">
<img src="chapters_files/figure-html/my-theme-1.png" alt="`theme()`で好みの見た目に設定できる" width="672" />
<p class="caption">(\#fig:my-theme)`theme()`で好みの見た目に設定できる</p>
</div>


## ヒストグラム{#ヒストグラム}

### ヒストグラムとは

ヒストグラムは量的データの分布を把握するために用いられるグラフです。一見棒グラフによく似ていますが、ヒストグラムが量的データのうち連続データを用いるのに対し、棒グラフは離散データを使用するという違いがあります。

[`ggplot2`][ggplot2]では、[`ggplot2::geom_histogram`][geom_histogram]を使用します。$x$軸にはPML（前上顎骨長）を、$y$軸では相対度数（`density`）や個数（`count`）を設定します。少数のデータの場合は個数で表現した方が良い場合がありますが、異なるサイズのデータを比較する場合や分布の形状を比較する場合は相対度数で示した方が良いようです。

Figure\@ref(fig:hist)は、[`ggplot2::geom_histogram`][geom_histogram]のデフォルト設定で出力したもので、ここからレイヤーを重ねていくことで、より美しくわかりやすい可視化をすることができます。ビン数はデフォルトで30ですが、`bins`でビン数を指定することもできます。ビン数は計算で算出することもできます（スタージェスの公式など）。

考古学では先行研究の蓄積が前提となる場合がほとんどなので、それに準じる形でビン幅をを設定する場面が多いと思われます。ビン幅は`binwidth`で設定することができます。ビン幅の設定によっては見た目が変わってしまうことがあり、作成者の恣意的な解釈に沿った作図も可能となるため、ビン幅の設定やヒストグラムを読み取る際には留意しなければいけません。

[geom_histogram]:https://ggplot2.tidyverse.org/reference/geom_histogram.html


```r
hist <- fish %>%
  ggplot() +
  ggplot2::geom_histogram(aes(x = PML, y = after_stat(count)))
hist
```

<div class="figure">
<img src="chapters_files/figure-html/hist-1.png" alt="geom_histogramのデフォルト表示" width="672" />
<p class="caption">(\#fig:hist)geom_histogramのデフォルト表示</p>
</div>

### 複数のヒストグラムを表示する

ヒストグラムで次元を追加する場合（グループ分け等）、事前にデータフレームの任意の列の型をcharacter型（文字）からfactor型（因子）に変換します。グラフの描写の中でも`levels =`で任意の順番に変更することもできます。

1つのグラフの中で複数の変数（グループなど）を表示する場合は、`position =`で描写方法を指定します。`stack`のほか、`identity`や`dodge`があります（Figure\@ref(fig:hist2)）。

一方、よく似た分布を示したデータの場合は、それぞれでヒストグラムを作成した方が見やすく、[`ggplot2::facet_wrap()`][facet_wrap]関数や、変数が多い場合は[`facet_grid()`][facet_grid]関数で表現すると比較しやすくなります。多くの場合は[`facet_wrap()`][facet_wrap]関数で十分です（Figure\@ref(fig:hist3)）。

[facet_wrap]:https://ggplot2.tidyverse.org/reference/facet_wrap.html
[facet_grid]:https://ggplot2.tidyverse.org/reference/facet_grid.html


```r
hist_2 <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Direction == "左" & 
                  Period %in% c("1期", "2期", "3期") &
                  PML > 0) %>%
  # `mutate_at`と`as.factor`を組み合わせて型を変更
  dplyr::mutate_at(c("Period", "Fish", "Bone"), as.factor) %>%
  ggplot() +
  ggplot2::geom_histogram(aes(x = PML, 
                              y = after_stat(density),
                              fill = Period),
                          binwidth = 5,          # bin幅の設定
                          colour = "black",      # グラフの枠線の色
                          position = "stack",    # 表示方法
                          boundary = 0) +        # ビンの境界を指定
  labs(x = "PML（mm）") +
  scale_x_continuous(breaks = seq(0, 50, 10), 
                     limits=c(10, 50)) +        # x軸の範囲を設定
  my_theme

hist_2
```

<div class="figure">
<img src="chapters_files/figure-html/hist2-1.png" alt="`position = &quot;stack&quot;`での描写" width="672" />
<p class="caption">(\#fig:hist2)`position = "stack"`での描写</p>
</div>


```r
hist_3 <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Direction == "左" & 
                  Period %in% c("1期", "2期", "3期") &
                  PML > 0) %>%
  ggplot() +
  ggplot2::geom_histogram(aes(x = PML, 
                              y = after_stat(density)),
                          binwidth = 5,     # bin幅の設定
                          fill = "white",   # グラフの塗りの色
                          colour = "black",     # グラフの枠線の色
                          boundary = 0) +       # ビンの境界を指定
  labs(x = "PML（mm）") +
  scale_x_continuous(breaks = seq(0, 50, 10), 
                     limits=c(0, 50)) +        # x軸の範囲を設定
  my_theme +
  # facet_wrapで遺構毎に分割して表示する。
  facet_wrap(~Period,
             # scales = "free_y",      # y軸をそれぞれ設定する時
             nrow = 1,
             labeller = as_labeller(c(`1期` = "縄文1期", 
                                      `2期` = "縄文2期", 
                                      `3期` = "縄文3期" )))
  #as_labellerでラベル名を変更。``で明示することでエラーを回避。

hist_3
```

<div class="figure">
<img src="chapters_files/figure-html/hist3-1.png" alt="`facet_weap()`の描写" width="672" />
<p class="caption">(\#fig:hist3)`facet_weap()`の描写</p>
</div>

### グラフの情報を取得する

また、Table\@ref(tab:hist-build)のように[`ggplot2::ggplot_build()`][ggplot_build]でggplot2で描写したグラフの情報を取得することも可能です。

[ggplot_build]:https://ggplot2.tidyverse.org/reference/ggplot_build.html


```r
hist_info <- hist_3 %>% ggplot_build()
hist_info_df <- data.frame(hist_info$data)

hist_info_df %>% 
  dplyr::select(PANEL, count, xmin, xmax, x, y, density) %>%
  head(n = 9) %>% 
  knitr::kable(caption = "作成したヒストグラムの情報")
```



Table: (\#tab:hist-build)作成したヒストグラムの情報

|PANEL | count| xmin| xmax|    x|         y|   density|
|:-----|-----:|----:|----:|----:|---------:|---------:|
|1     |     0|    0|    5|  2.5| 0.0000000| 0.0000000|
|1     |     0|    5|   10|  7.5| 0.0000000| 0.0000000|
|1     |     0|   10|   15| 12.5| 0.0000000| 0.0000000|
|1     |     2|   15|   20| 17.5| 0.0046512| 0.0046512|
|1     |    11|   20|   25| 22.5| 0.0255814| 0.0255814|
|1     |    44|   25|   30| 27.5| 0.1023256| 0.1023256|
|1     |    26|   30|   35| 32.5| 0.0604651| 0.0604651|
|1     |     3|   35|   40| 37.5| 0.0069767| 0.0069767|
|1     |     0|   40|   45| 42.5| 0.0000000| 0.0000000|

## 棒グラフ

先述した通りヒストグラムは量的データのうち連続データを示すものですが、棒グラフは量的データのうち離散データを示す際に用いるグラフです。棒グラフは棒の高さでデータの大小を示し、値の高低を判別する際に有効です。

### 棒グラフ

[`ggplot2`][ggplot2]では[`ggplot2::geom_bar()`][geom_bar]で描写することができます。棒グラフで変数を追加する場合（グループを加える等）は`fill =`で変数を指定できます。これの機能を利用して帯グラフ（積み上げ棒グラフ）作成することもできます。

[geom_bar]:https://ggplot2.tidyverse.org/reference/geom_bar.html


```r
bar <- fish %>%
  dplyr::filter(Period %in% c("1期", "2期", "3期")) %>%
  ggplot() +
  ggplot2::geom_bar(aes(x = Period)) +
  my_theme    # 日本語を使用する場合

bar
```

<div class="figure">
<img src="chapters_files/figure-html/bar-1.png" alt="geom_barのデフォルト表示" width="672" />
<p class="caption">(\#fig:bar)geom_barのデフォルト表示</p>
</div>

### 帯グラフ

帯グラフを描写したい時も[`ggplot2::geom_bar()`][geom_bar]を使い、$y$軸いっぱいに引き伸ばす`position = "fill"`とパーセンテージで表示する`scale_y_continuous(labels = scales::percent)`を指定します。多くの場合は横方向が好ましいので`coord_flip()`も加えます。


```r
band <- fish %>%
  dplyr::filter(Period %in% c("1期", "2期", "3期")) %>%
  ggplot2::ggplot() +
  geom_bar(aes(x = Period, fill = Fish),
           position = "fill") +   # 帯グラフにする
  scale_y_continuous(labels = scales::percent) +
  my_theme +
  theme(axis.title.x = element_blank(),    # x軸のタイトルを消す
        axis.title.y = element_blank()     # y軸のタイトルを消す
        ) +
  coord_flip()

band
```

<div class="figure">
<img src="chapters_files/figure-html/bar2-1.png" alt="帯グラフ" width="672" />
<p class="caption">(\#fig:bar2)帯グラフ</p>
</div>

割合の少ない項目は「その他」等にまとめた方が視認性が上がることが知られています。ここでは上位5つより下位の魚種を「その他」にまとめ、主要な捕獲対象となった魚種を調べます。


```r
#### ラベルを作成する ####
# 魚種の出土数トップ9を抽出
top5 <- fish %>%
  dplyr::filter(Period %in% c("1期", "2期", "3期")) %>%
  dplyr::group_by(Fish) %>%            # グループ化
  dplyr::summarize(count = n(),        # 魚種の個数をカウント
                   .groups = "drop"    # グループ解除しておく癖をつける。
                   ) %>%
  dplyr::arrange(desc(count)) %>%      # `desc()`関数は降順に、`arrange()`関数で並べ替える。
  top_n(5) %>%                         # `top_n()`関数で上位のデータだけ抽出する
  dplyr::mutate(Fish = as.character(Fish))  # top9$Fishがfactor型だと調子が悪い。

##### 帯グラフに割り付ける ####
band_2 <- fish %>%
  dplyr::filter(Period %in% c("1期", "2期", "3期")) %>%
  droplevels() %>%
  dplyr::mutate(Fish = factor(if_else(Fish %in% top5$Fish, 
                               Fish, "その他"),
                              levels = c(top5$Fish, "その他"))) %>% 
  ggplot() +
  ggplot2::geom_bar(aes(x = Period,
                        fill = Fish),
                    color = "black",
                    position = "fill") +   # 帯グラフにする
  # `scale_y_reverse()` はグラフのy軸を逆順に表示するための関数（見た目のみ）。
  # `breaks=`部分：y軸の目盛りを0から1まで、0.2刻みで設定します。
  # `labels=`部分：0から80まで、20刻みで目盛りを設定し、
  # それを逆順にしています（`rev()`関数を使用）
  scale_y_reverse(breaks = seq(0, 1.0, by = 0.2),
                  labels = c("100 %", rev(seq(0, 80, by = 20)))) + 
  my_theme +
  theme(axis.title.x = element_blank(),    # x軸のタイトルを消す
        axis.title.y = element_blank(),    # y軸のタイトルを消す
        ) +
  coord_flip()

band_2
```

<div class="figure">
<img src="chapters_files/figure-html/bar3-1.png" alt="構成する種類をまとめた帯グラフ" width="672" />
<p class="caption">(\#fig:bar3)構成する種類をまとめた帯グラフ</p>
</div>

また、[`ggpattern`][ggpattern]パッケージの[`ggpattern::geom_bar_pattern()`][geom_bar_pattern]を使用すれば、論文で使うような水玉や斜線、網掛け線などのパターン表示をすることもできます。[`@ocean_fさんのQiita`][`@ocean_fさんのQiita`]で作例が公開されています。

[ggpattern]:https://trevorldavis.com/R/ggpattern/dev/
[geom_bar_pattern]:https://trevorldavis.com/R/ggpattern/dev/reference/geom-docs.html
[`@ocean_fさんのQiita`]:https://qiita.com/ocean_f/items/2b7243e4e19231d27713



```r
#### 時期毎の集計表を作成し、割り付けるラベルを作成 ####
fish_summarize <- fish %>%
  dplyr::filter(Period %in% c("1期", "2期", "3期")) %>%
  droplevels() %>%                           # 次の処理のために不要なレベルを削除しておく（重要）
  #`if_else()`関数で場合分けする。トップ5以外は"その他"としてまとめる。
  #`levels`引数に`top5$Fish`を指定して、top5の順序通りにする。
  dplyr::mutate(Fish = factor(if_else(Fish %in% top5$Fish, 
                                      Fish, "その他"),
                              levels = c(top5$Fish, "その他"))) %>% 
  dplyr::group_by(Fish, Period) %>%         # グループ化
  dplyr::summarize(count = n(),             # 時期毎の個数をカウント
                   .groups = "drop"         # グループ解除しておく癖をつける。
                   ) %>%
  dplyr::group_by(Period) %>%
  dplyr::mutate(pct = count / sum(count),    # 時期毎の割合を算出
                cum_pct = cumsum(pct),       # 時期毎の累積割合を算出
                pct_label = format(          # 時期毎のラベルを作成
                  round(pct * 100, 1),       # `fotmat()`で小数点第1位まで表示
                  nsmall = 1)
                ) %>% 
  ungroup() %>%                    # グループ解除しておく癖をつける。
  arrange(Period)                  # `desc()`関数がない場合は昇順になる。


##### 論文風にする ####
# `ggpattern::geom_bar_pattern`で論文風にできる。
library(ggpattern)
band_3 <- fish_summarize %>% 
  ggplot() +
  ggpattern::geom_bar_pattern(aes(x = Period,
                                  y = pct,                 # `geom_bar_pattern`ではyを指定する必要がある。
                                  fill = Fish,
                                  pattern = Fish,
                                  pattern_spacing = Fish, 
                                  pattern_density = Fish),
                              color = "black",             # グラフの線の色を設定
                              pattern_fill = "black",      # パターンの線の色を設定
                              pattern_color = NA,
                              stat = "identity",
                              position = "fill") +
  # 帯グラフのパターンを設定する
  ggplot2::scale_fill_manual(name = "Fish",
                               values = c("grey50", "white", "grey75", 
                                          "white", "white", "grey95"),
                               guide = guide_legend(byrow = TRUE)) +
  ggpattern::scale_pattern_manual(name = "Fish",
                                  values = c("none", "stripe", "none", 
                                             "circle", "crosshatch", "none"),
                                  guide = guide_legend(byrow = TRUE)) +
  ggpattern::scale_pattern_density_manual(name = "Fish",
                                          values = c(NA, 0.1, NA, 0.2, 0.1, NA),
                                          guide = guide_legend(byrow = TRUE))  +
  ggpattern::scale_pattern_spacing_manual(name = "Fish",
                                          values = c(NA, 0.05, NA, 0.05, 0.05, NA),
                                          guide = guide_legend(byrow = TRUE)) +
  # `scale_y_reverse()` はグラフのy軸を逆順に表示するための関数（見た目のみ）。
  # `breaks=`部分：y軸の目盛りを0から1まで、0.2刻みで設定します。
  # `labels=`部分：0から80まで、20刻みで目盛りを設定し、
  # それを逆順にしています（`rev()`関数を使用）
  scale_y_reverse(breaks = seq(0, 1.0, by = 0.2),
                  labels = c("100 %", rev(seq(0, 80, by = 20)))) + 
  my_theme +
  theme(axis.title.x = element_blank(),    # x軸のタイトルを消す
        axis.title.y = element_blank(),    # y軸のタイトルを消す
        ) +
  coord_flip()

# 帯グラフに配置するラベルと座標を設定する。
fish_summarize <- fish_summarize %>%
  ungroup() %>%                                            # グループを解除しておく。
  mutate(y = round((1 - cum_pct) + pct / 2 , 3))           # `scale_y_reverse()` でy座標の見た目が反転しているので1から引く。


# 作図・出力（library(ggtext)でラベルを設定）
band_3 + 
  ggtext::geom_richtext(aes(x = fish_summarize$Period,
                            y = fish_summarize$y,
                            label = fish_summarize$pct_label),
                        fill = "white",                   # ラベルの塗りの設定
                        label.color = NA,                 # ラベルの枠線の設定
                        size = 3,                         # 文字の大きさ
                        fontface = "bold",                # 文字の太さ
                        hjust = 0.5,                      # 上下の位置を設定
                        vjust = 0.5,                      # 左右の位置を設定
                        label.padding = unit(c(0.15, 0.1, 0.1, 0.1), "lines"),  # ラベルの余白を設定
                        label.r = unit(0, "lines"))       # ラベルの角の丸みを設定
```

<div class="figure">
<img src="chapters_files/figure-html/bar4-1.png" alt="geom_bar_patternでの論文風の帯グラフ" width="672" />
<p class="caption">(\#fig:bar4)geom_bar_patternでの論文風の帯グラフ</p>
</div>

[`ggplot2`][ggplot2]の拡張機能である[`ggtext::geom_richtext()`][geom_richtext]は、例えばテキストを太字、斜体にしたり、フォント、色、サイズを変更するなどの見た目の設定や、テキストを下付き文字や上付き文字として配置したり、簡単な画像を貼り付けたりすることができ、細かな設定もできます。`label.padding`はラベルの文字列と枠の間の余白を`unit()`を使って指定できる機能で、`label.r`はラベルの角の丸みを設定することができます。引数の使用例は[「からっぽのしょこ」さん][からっぽのしょこ]等の個人ブログを参照した方がわかりやすい。

なお、上記のような帯グラフはエクセルでも比較的簡単に作成できる一方、[`ggplot2`][ggplot2]では非常に手間がかかるのが弱点です。あらかじめ1つフォーマットを作っておき、使い回しできるようにしておくと時間の短縮となって良いです。

[geom_richtext]:https://wilkelab.org/ggtext/
[からっぽのしょこ]:https://www.anarchive-beta.com/entry/2022/12/08/131400#labelpadding引数

## 円グラフ

円グラフは、データ全体を円形で、データ構成する各項目を円形を分割する扇形で表現したグラフで、通常は割合（パーセンテージ）で表示されます。円グラフはエクセルで簡単に作成できることから、割合を表現する方法として広く用いられていますが、学術研究では好ましくないことが指摘されています。

考古学では、石井[-@ishii_2020]が下記のRのヘルプを参照しつつ指摘しています（コンソールで`?pie`と入力すれば確認できます）。

> Pie charts are a very bad way of displaying information. The eye is good at judging linear measures and bad at judging relative areas. A bar chart or dot chart is a preferable way of displaying this type of data.  
> （意訳）円グラフは情報を表示するのに非常に不適切な方法です。人間の目は線の長さを判断するのは得意ですが、相対的な面積を判断するのは苦手です。棒グラフやドットチャート（ドットプロット。ヒストグラムの一種）の方が、この種類のデータを表示するのに適しています。  
> Cleveland (1985), page 264: “Data that can be shown by pie charts always can be shown by a dot chart. This means that judgements of position along a common scale can be made instead of the less accurate angle judgements.” This statement is based on the empirical investigations of Cleveland and McGill as well as investigations by perceptual psychologists.  
> （意訳）Cleveland (1985)の264ページには次のように書かれている。「円グラフで表示できるデータは、常にドットチャートで表示することができる。これにより、より正確な位置の判断ができるようになり、角度の判断よりも正確性が向上する。」この主張は、ClevelandとMcGillの実証的な調査や知覚心理学者の調査に基づいている。

米ワシントン大学の考古学者のBen Marwick氏が先行研究等をまとめたところによると、円グラフはカテゴリー間の比較では棒グラフに劣る一方、全体の割合の推定では円グラフは棒グラフと同程度正確であるとのことです[@Marwick_2019]。一概に円グラフが悪いということではなく、データの性質が最も把握できる表現方法を選択すべきということでしょう。

なお、[`ggplot2`][ggplot2]では円グラフ専用の`geom_*()`関数はありませんが、[`ggplot2::geom_bar()`][geom_bar]を利用して作成することができます。



```r
fish_summarize %>% 
  dplyr::mutate(pct2 = pct * 100) %>%  # パーセンテージ表示するため
  ggplot2::ggplot() +
  geom_bar(aes(x = "", y = pct2, fill = Fish),
           stat = "identity") +   
  coord_polar(theta = "y") +
  scale_y_reverse() +                # 「そのほか」が左上に行くようにする。
  facet_wrap(~Period) +
  my_theme +
  theme(                                 # themeの更新
    line = element_blank(),              # 余計なラインを消す
    rect = element_blank(),              # 余計な枠線を消す
    panel.grid.major = element_blank(),  # 余計な罫線を消す
    axis.title = element_blank(),        # 軸のタイトルを消す
    panel.border = element_blank()       # `facet_wrap`のパネルの枠線を消す
    ) 
```

<img src="chapters_files/figure-html/unnamed-chunk-20-1.png" width="672" />


## 箱ひげ図

### 箱ひげ図

箱ひげ図は「中央値」、「第1四分位数」、「第3四分位数」、「最小値」、「最大値」の5つ要約統計量を視覚的に表現するための図です。箱の両端は「第1四分位数」[^2] （Q1）と「第3四分位数」（Q3）を示し、箱のことを「四分位範囲」といいます。箱中央にある線は「中央値」(Median)を示します。ひげの両端は「最小値」と「最大値」、その外側の点は「外れ値」を示します。

[^2]: データを4つに分割した際に下からちょうど1/4にあるデータを「第1四分位数」、2/4にあるデータを「第2四分位数」（中央値）、3/4のところのデータを「第3四分位数」といい、これらを「四分位数」といいます。

箱ひげ図は、アメリカの統計学者John Tukey氏が1977年に著書"Exploratory Data Analysis"で発表した比較的新しいグラフです。今日では、一般的な統計手法として広く使用されています。[`ggplot2`][ggplot2]では[`ggplot2::geom_boxplot()`][geom_boxplot]で導出することができます。

なお、箱ひげ図は複数のピークを持つ分布を適切に表すことができないので、データの「分布」を説明したい場合は、ヒストグラムや後述のバイオリンプロット、シナプロットを使う必要があります。

[geom_boxplot]:https://ggplot2.tidyverse.org/reference/geom_boxplot.html?q=boxplot#null


```r
box <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Period %in% c("1期", "2期", "3期") &
                  Direction == "左" &
                  PML > 0) %>%
  ggplot(aes(x = Period, y = PML)) +   # `x =`でグループを設定
  geom_boxplot(width = 0.5, 
               fill = "white") +
  # 平均を描写する場合
  stat_summary(fun = mean,
               geom = "point",
               shape = 21,
               size = 3.0,
               color = "white",
               fill = "red") +     
  labs(x = "Archaeological Period", y = "PML(mm)") +
  coord_flip() +
  my_theme

box
```

<img src="chapters_files/figure-html/unnamed-chunk-21-1.png" width="672" />

### バイオリンプロット／シナプロット

バイオリンプロットはデータの分布を視覚的に表現するためのグラフです。5つの要約統計量を示す箱ひげ図よりも詳細な分布を知ることができます。バイオリンプロットは`カーネル密度推定`を用いて滑らかな曲線で表現されます。Leland Wilkinson氏が1999年に論文"Dot plots"で発表したグラフです[@wilkinson_1999]。

シナプロットはバイオリンプロットと同じくデータの分布を可視化するためのグラフで、個々のデータをポイントで表示することができます。[`ggplot2`][ggplot2]でも[`ggplot2::geom_jitter`][geom_jitter]を使えば軸に幅を持たせて点をランダムに配置したグラフ（ジッタープロット）を描写することができますが、シナプロットはデータの密度分布に沿ってジッターの幅が制御され、データの数・密度分布・外れ値・広がりの情報がよりわかりやすくなっています。Nikos Sidiropoulos氏やSina Hadi Sohi氏らが2018年に論文"SinaPlot: An Enhanced Chart for Simple and Truthful Representation of Single Observations Over Multiple Classes"で発表しました[@Sidiropoulos_et_al_2018]。

バイオリンプロットは[`ggplot2`][ggplot2]の[`ggplot2::geom_violin()`][geom_violin]を用いて、[`ggplot2::geom_boxplot()`][geom_boxplot]と同じような使い方ができます（Figure\@ref(fig:violin)）。シナプロットは[`ggforce`][ggforce]パッケージの[`ggforce::geom_sina()`][geom_sina]で実装することができます（Figure\@ref(fig:sina)）。バイオリンプロットとシナプロットともに、箱ひげ図などの他のグラフと組み合わせることも可能です（Figure\@ref(fig:box-violin-sina)）。

[geom_jitter]:https://ggplot2.tidyverse.org/reference/geom_jitter.html
[geom_violin]:https://ggplot2.tidyverse.org/reference/geom_violin.html
[ggforce]:https://ggforce.data-imaginist.com
[geom_sina]:https://ggforce.data-imaginist.com/reference/geom_sina.html


```r
#### バイオリンプロット ####
violin <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Period %in% c("1期", "2期", "3期") &
                  Direction == "左" &
                  PML > 0) %>%
  ggplot(aes(x = Period, y = PML)) +
  geom_violin(trim = FALSE,
              fill = "white") +
  labs(x = "Archaeological Period", y = "PML(mm)") +
  coord_flip() +
  my_theme

violin
```

<div class="figure">
<img src="chapters_files/figure-html/violin-1.png" alt="バイオリンプロット" width="672" />
<p class="caption">(\#fig:violin)バイオリンプロット</p>
</div>


```r
#### ジッタープロット ####
jitter <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Period %in% c("1期", "2期", "3期") &
                  Direction == "左" &
                  PML > 0) %>%
  ggplot(aes(x = Period, y = PML)) +
  geom_jitter(width = 0.2) +
  labs(x = "Archaeological Period", y = "PML(mm)") +
  coord_flip() +
  my_theme

#### シナプロット ####
library("ggforce")

sina <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Period %in% c("1期", "2期", "3期") &
                  Direction == "左" &
                  PML > 0) %>%
  ggplot(aes(x = Period, y = PML)) +
  ggforce::geom_sina() +
  labs(x = "Archaeological Period", y = "PML(mm)") +
  coord_flip() +
  my_theme

jitter
sina
```

<div class="figure">
<img src="chapters_files/figure-html/sina-1.png" alt="ジッタープロット（左）とシナプロット（右）" width="48%" /><img src="chapters_files/figure-html/sina-2.png" alt="ジッタープロット（左）とシナプロット（右）" width="48%" />
<p class="caption">(\#fig:sina)ジッタープロット（左）とシナプロット（右）</p>
</div>



```r
#### 箱ひげ図にバイオリンプロットとシナプロットを重ねる ####
box_violin_sina <- fish %>%
  dplyr::filter(Fish == "クロダイ属"  &
                  Period %in% c("1期", "2期", "3期") &
                  Direction == "左" &
                  PML > 0) %>%
  ggplot(aes(x = Period, y = PML)) +
  # バイオリンプロットを描写
  geom_violin(trim = FALSE,
              fill = "black",
              linetype = "blank",    # 確率密度曲線の枠線をなしに。
              alpha = 0.1) +         # fillの透明度を設定する。
  # 箱ひげ図を追加
  geom_boxplot(width = 0.3,
               outliers = FALSE,      # 外れ値の描写
               ) +
  # シナプロットを追加
  ggforce::geom_sina(alpha = 0.15,
                     colour = "black") +
  # 平均や中央値を描写する場合、`stat_summary()`を使えば良い。
  stat_summary(fun = mean,
               geom = "point",
               shape = 21,
               size = 2.0,
               color = "white",
               fill = "red") +     
  labs(x = "Archaeological Period", y = "PML(mm)") +
  coord_flip() +
  my_theme

box_violin_sina
```

<div class="figure">
<img src="chapters_files/figure-html/box-violin-sina-1.png" alt="箱ひげ図と重ねて表示" width="672" />
<p class="caption">(\#fig:box-violin-sina)箱ひげ図と重ねて表示</p>
</div>

## 折れ線グラフ

折れ線グラフは[`ggplot2::geom_line`][geom_path]で描写することができます。今回は、COVID-19の感染者数のデータを利用して、一月ごとの感染者数の推移がわかるようなグラフを作成します。データには[厚生労働省のホームページ][covid-19]で「新規陽性者数の推移（日別）」が公開されているのでこれをダウンロードします。

このデータは横長型のデータなので、まず縦長型の`tidy data`に変換します。

[geom_path]:https://ggplot2.tidyverse.org/reference/geom_path.html
[covid-19]:https://www.mhlw.go.jp/stf/covid-19/open-data.html


```r
# データの読み込み
covid19 <- readr::read_csv("analysis/data/newly_confirmed_cases_daily.csv")
covid19 %>% head()
```

```
## # A tibble: 6 × 49
##   Date        ALL Hokkaido Aomori Iwate Miyagi Akita Yamagata Fukushima Ibaraki
##   <chr>     <dbl>    <dbl>  <dbl> <dbl>  <dbl> <dbl>    <dbl>     <dbl>   <dbl>
## 1 2020/1/16     1        0      0     0      0     0        0         0       0
## 2 2020/1/17     0        0      0     0      0     0        0         0       0
## 3 2020/1/18     0        0      0     0      0     0        0         0       0
## 4 2020/1/19     0        0      0     0      0     0        0         0       0
## 5 2020/1/20     0        0      0     0      0     0        0         0       0
## 6 2020/1/21     0        0      0     0      0     0        0         0       0
## # ℹ 39 more variables: Tochigi <dbl>, Gunma <dbl>, Saitama <dbl>, Chiba <dbl>,
## #   Tokyo <dbl>, Kanagawa <dbl>, Niigata <dbl>, Toyama <dbl>, Ishikawa <dbl>,
## #   Fukui <dbl>, Yamanashi <dbl>, Nagano <dbl>, Gifu <dbl>, Shizuoka <dbl>,
## #   Aichi <dbl>, Mie <dbl>, Shiga <dbl>, Kyoto <dbl>, Osaka <dbl>, Hyogo <dbl>,
## #   Nara <dbl>, Wakayama <dbl>, Tottori <dbl>, Shimane <dbl>, Okayama <dbl>,
## #   Hiroshima <dbl>, Yamaguchi <dbl>, Tokushima <dbl>, Kagawa <dbl>,
## #   Ehime <dbl>, Kochi <dbl>, Fukuoka <dbl>, Saga <dbl>, Nagasaki <dbl>, …
```

```r
# 横長型のデータを縦長型（tidy data）に変換
covid19_tidy <- covid19 %>%
  dplyr::select(-ALL) %>%   # 不要な列の削除
  tidyr::pivot_longer(cols = !Date, 　　# `!`で動かさないデータを指定
                      names_to = "pref",
                      values_to = "infected")
covid19_tidy %>% head()
```

```
## # A tibble: 6 × 3
##   Date      pref     infected
##   <chr>     <chr>       <dbl>
## 1 2020/1/16 Hokkaido        0
## 2 2020/1/16 Aomori          0
## 3 2020/1/16 Iwate           0
## 4 2020/1/16 Miyagi          0
## 5 2020/1/16 Akita           0
## 6 2020/1/16 Yamagata        0
```

変形したデータを[`dplyr::summarize()`][summarize]関数で、週ごとの合計を算出したのちにグラフを作成します。詳しくは[「`summarize()`による概要表」](#概要表)を参照してください。


```r
#### lubridateパッケージを使って日付型に変換。####
library("lubridate")

# `lubridate::ymd()`で日付型にする。`dmy()`や`mdy()`もある。
covid19_tidy <- covid19_tidy %>%
  dplyr::mutate(dplyr::across(Date, lubridate::ymd))

#### データを編集する ####
# `lubridate::floor_date`で週ごとのデータにする。
covid19_tidy <- covid19_tidy %>%
  dplyr::mutate(week_start = lubridate::floor_date(Date, unit = "month"))

# `dplyr::summarize`で週ごとの合計を算出。
covid19_monthly <- covid19_tidy %>%
  dplyr::group_by(week_start) %>%
  dplyr::summarize(total = sum(infected),
                   .groups = "drop")

covid19_monthly %>% head()
```

```
## # A tibble: 6 × 2
##   week_start total
##   <date>     <dbl>
## 1 2020-01-01    14
## 2 2020-02-01   213
## 3 2020-03-01  1936
## 4 2020-04-01 11952
## 5 2020-05-01  2439
## 6 2020-06-01  1741
```


```r
# 折れ線グラフを描写する。
covid19_monthly %>%
  ggplot2::ggplot(aes(x = week_start,
                      y = total)) +
  geom_line() +                                 # 折れ線グラフの表示
  geom_point(size = 0.9) +                      # ポイントを表示する
  labs(x = "月", y = "感染者数") +
  scale_x_date(date_breaks = "3 month",
               date_labels = "%Y-%m") +         # x軸の表示方法を変更する
  scale_y_continuous(labels = scales::comma) +  # 数字にカンマを入れる
  my_theme +
  theme(axis.text.x = element_text(angle = 45, 
                                   hjust = 1))  # x軸のラベルを45度回転させる 
```

<div class="figure">
<img src="chapters_files/figure-html/line-1.png" alt="折れ線グラフ" width="672" />
<p class="caption">(\#fig:line)折れ線グラフ</p>
</div>



# 引用参考文献