---
title: 今、WebRが熱い！
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

個人的に2023年のR言語界隈でホットだったWebRについて、日本語での紹介を見かけなかったので、私の知る範囲で簡単にまとめます。

:::message

これは[R言語 Advent Calendar 2023](https://qiita.com/advent-calendar/2023/rlang)の7日目の記事です。

:::

## WebRとは

https://github.com/r-wasm/webr

[WebR公式ドキュメント](https://docs.r-wasm.org/webr/latest/)には以下のように書かれています。

> WebR is a version of the statistical language [R](https://www.r-project.org/) compiled for the browser and [Node.js](https://nodejs.org/en/) using [WebAssembly](https://webassembly.org), via [Emscripten](https://emscripten.org/).

要するにブラウザ上などでRが動くということですね（技術的なことをよく理解していないため説明できない）。

現在はPosit社のメンバーがメインとなって[r-wasmというGitHub組織](https://github.com/r-wasm)で開発されています。

元々はGeorge Staggさん（恐らくWebR開発のために現在はPosit社所属されている）の個人プロジェクトだったのだと思われます。\
Gitリポジトリの履歴を見ると最初のコミットは2022年1月18日だったようです。

私は2022年2月に以下のツイートをリツイートしていたので、割とリポジトリのできた直後にWebRを知ったようです。

https://twitter.com/gwstagg/status/1495495339444473858

このツイートからリンクされている以下のデモサイト（まだ生きていて驚きました）で試してみて、ブラウザ上でRが動いたことに対する感動と、読み込みの遅さに対する厳しさを感じたことを覚えています。

試しに今やってみたら、Rの読み込みに**2分**かかりました。

![old webr repl](/images/webr-is-2023/old-webr.jpg)
*WebR君が`plot(mtcars)`を実行するするまで2分かかりました*

しかし、それは完全に過去の話です。\
最新（バージョン0.2.3-dev）の[webR REPL app](https://webr.r-wasm.org/latest/)を実行してみると、**5秒**でRを使用可能になりました。

![new webr repl](/images/webr-is-2023/new-webr.jpg)
*最新の豪華で早いWebRデモサイト*

## WebRでできること

……は私も詳しくないので、[WebRの公式ドキュメント](https://docs.r-wasm.org/webr/latest/)をご覧いただきたいです。

ところで、この公式ドキュメントはQuartoで作成されたウェブサイトで、WebRが埋め込まれていることに気づくと思います。\
同じようにQuartoドキュメントにWebRを埋め込みたい場合、`quarto-webr`というQuarto拡張機能を使用すると専用のコードブロック内に書いたRコードをWebRで簡単に実行させられます。

https://github.com/coatless/quarto-webr

興味のある方はこちらも是非お試しください。

## 利用可能なRパッケージ

## Node.jsでの利用

https://rud.is/books/webr-cli-book/

