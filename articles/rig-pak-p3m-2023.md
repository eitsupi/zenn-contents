---
title: イマドキRインストール事情？ ～rig、pak、p3m～
emoji: 💻
type: tech
topics:
    - r
published: true
published_at: 2023-12-17
---

皆さん、Rust使われていますか？

Rustのプロジェクト管理、便利ですよね。rustupでRust本体のバージョンを楽々切り替え、Cargoで良い感じにパッケージ管理できるのがデフォルトで。

:::message

これは[**R言語** Advent Calendar 2023](https://qiita.com/advent-calendar/2023/rlang)の17日目の記事です。\
（Rustの記事ではありません）

:::

## 他の言語のRustっぽいやつ（？）

Rustの体験が優れているからか、他の言語でもRustっぽいものを見かけます。

Julia公式のJuliaバージョン管理ツールはその名もjuliaupといいます。

https://github.com/JuliaLang/juliaup


これは単にrustupと名前が似ているだけではなく、実際にソースコードを一部流用しているようです。\
juliaupインストール用シェルスクリプトの上の方に、rustupのインストールスクリプトを改変して使用している旨が書かれています。

```sh:bash
$ curl -fsL https://install.julialang.org | head
#!/bin/sh
# shellcheck shell=dash

# This script is adapted for juliaup from the original rustup repository
# over at https://github.com/rust-lang/rustup. Names and urls have been
# changed during the adaptation.

# This is just a little script that can be downloaded from the internet to
# install juliaup. It just does platform detection, downloads the installer
# and runs it.
```

Pythonでは、今年公開されたRyeはまさに「Pythonの世界にもRustのようなものがあってほしい」という動機で作られ公開されたそうです。

https://github.com/mitsuhiko/rye

https://github.com/mitsuhiko/rye/discussions/6
