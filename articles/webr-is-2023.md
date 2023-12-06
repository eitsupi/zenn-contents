---
title: ブラウザで動くR、WebRが熱い！
emoji: 🕸️
type: tech
topics:
    - r
    - javascript
    - nodejs
    - wasm
published: true
published_at: 2023-12-07
---

個人的に2023年のR言語界隈でホットだったWebR[^webr]について、日本語での紹介を見かけなかったので、私の知る範囲で簡単にまとめます。

[^webr]: "WebR"なのか"webR"なのか、公式ドキュメント内でも表記が揺れておりよく分かりませんがとりあえず本記事内では"WebR"と表記します。

:::message

これは[R言語 Advent Calendar 2023](https://qiita.com/advent-calendar/2023/rlang)の7日目の記事です。

:::

## WebRとは

https://github.com/r-wasm/webr

[WebR公式ドキュメント](https://docs.r-wasm.org/webr/latest/)には以下のように書かれています。

> WebR is a version of the statistical language [R](https://www.r-project.org/) compiled for the browser and [Node.js](https://nodejs.org/en/) using [WebAssembly](https://webassembly.org), via [Emscripten](https://emscripten.org/).

要するにブラウザ上などでRが動くということですね（技術的なことをよく理解していないため説明できない）。

現在はPosit社のメンバーがメインとなって[r-wasmというGitHub組織](https://github.com/r-wasm)で開発されています。

元々はGeorge Staggさん（恐らくWebR開発のために現在はPosit社に所属されている）の個人プロジェクトだったのだと思われます。\
Gitリポジトリの履歴を見ると最初のコミットは2022年1月18日だったようです。

私は2022年2月に以下のツイートをリツイートしていたので、割とリポジトリのできた直後にWebRを知ったようです。

https://twitter.com/gwstagg/status/1495495339444473858

このツイートからリンクされている以下のデモサイト（まだ生きていて驚きました）で試してみて、ブラウザ上でRが動いたことに対する感動と、読み込みの遅さに対する厳しさを感じたことを覚えています。

試しに今やってみたら、Rの読み込みに**2分**かかりました。

![old webr repl](/images/webr-is-2023/old-webr.jpg)
*WebRさんが`plot(mtcars)`を実行するするまで2分かかりました*

しかし、それは完全に過去の話です。\
最新（バージョン0.2.3-dev）の[webR REPL app](https://webr.r-wasm.org/latest/)を実行してみると、**5秒**でRを使用可能になりました。

![new webr repl](/images/webr-is-2023/new-webr.jpg)
*最新の豪華で早いWebRデモサイト*

このくらい早いと、「ちょっとR動かしてみたいな」というときはこのページを開くだけで充分かもしれません。

## WebRでできること

……は私も詳しくないので、[WebRの公式ドキュメント](https://docs.r-wasm.org/webr/latest/)をご覧いただきたいです。

ところで、この公式ドキュメントはQuartoで作成されたウェブサイトで、WebRが埋め込まれていることに気づくと思います。\
同じようにQuartoウェブサイトにWebRを埋め込みたい場合、`quarto-webr`というQuarto拡張機能を使用すると専用のコードブロック内に書いたRコードをWebRで簡単に実行させられます。

https://github.com/coatless/quarto-webr

また別のQuarto拡張`shinylive`でShinyアプリを埋め込むこともできるとか。（当初はPyodideによるPython版Shinyのみだっただったものが現在ではWebRによるR版Shinyにも対応したようです）

https://github.com/quarto-ext/shinylive

興味のある方はこちらも是非お試しください。

## 利用可能なRパッケージ

通常のRと同じく、WebRのREPL内でも`install.packages()`関数を実行することで外部パッケージをインストールすることができます。\
デフォルトで使用されるリポジトリは<https://repo.r-wasm.org/>で、現在約10,000個のパッケージが利用可能なようです。このページをブラウザで開くとWebRベースのShinyアプリになっていて面白いです。

現在CRANに存在するパッケージは20,000程度らしく、約半分のパッケージをWebRで利用できることになります。私は「意外と多いな」という印象を持ちました。

また、11月からR-universeもすべてのRパッケージをWebR用にビルドするようになったため、（ビルドに成功しているパッケージについては）R-universeからもインストール可能です。詳しくはrOpenSciのブログ記事をどうぞ。

https://ropensci.org/blog/2023/11/17/runiverse-wasm/

### 利用できないRパッケージ？

どのようなパッケージが利用できない（ビルドできていない）のでしょうか？\
私はWASMについて全く理解できていないものの、一つ思い当たったのは、Python界隈でPandas 3.0からpyarrowを必須依存関係にするという話の中で挙がっていた「`pyarrow`をWASM化できていないためPyodideでPandas使えなくなっちゃう」という話です。[^pyarrow]
となれば`pyarrow`と同じlibarrowに基づいている`arrow`パッケージもビルドできていないのでは？と思い試したら、やはりWebRでは利用できませんでした。

[^pyarrow]: 現時点では未解決？ https://github.com/pyodide/pyodide/issues/2933

ビルドの大変なパッケージとして他に思い当たった`duckdb`は普通に使えました。DuckDBはどこでも使えますね。

`gifski`はじめRustソースコードを含むパッケージも全滅しているようですけれども、最近`hellorust`のソースコードを改変してWebRで動かせたという発表があったので、いずれ動くようになるかもしれません。

https://mstdn.social/@gws/111494461618624785

## Node.jsでの利用

最近公開された以下のウェブサイトではNode.jsでWebRを使ったCLIツールを作る方法が解説されています。

https://rud.is/books/webr-cli-book/

この記事ではこの本の通りにCLIツールを作ってみようかとも思っていたのですが時間切れになったのでまたの機会に試してみたいと思います。

## その他

Rパッケージ用ウェブサイトとして広く利用されている`pkgdown`に「ExamplesをWebRで動かせるようにしようぜ」的な話があるのはだいぶ夢があるなと思いました。

https://github.com/r-lib/pkgdown/issues/2348
