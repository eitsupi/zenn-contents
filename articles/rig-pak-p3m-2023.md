---
title: イマドキRのインストール事情？ ～rig、pak、p3m～
emoji: 💻
type: tech
topics:
    - r
published: true
published_at: 2023-12-17
---

## まえおき

皆さん、Rust使われていますか？

Rustのインストール体験、良いですよね。rustupでRust本体のバージョンを楽々切り替え、Cargoでパッケージ管理できるのがデフォルトで。

:::message

これは[**R言語** Advent Calendar 2023](https://qiita.com/advent-calendar/2023/rlang)の17日目の記事です。\
（Rustの記事ではありません）

:::

### 他の言語のRustっぽいやつ（？）

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

もちろんjuliaupもRust製です。

Pythonでは、今年Armin Ronacherさんによって公開されたRyeはまさに「Pythonの世界にもRustのようなものがあってほしい」という動機で作られたそうです。

https://github.com/mitsuhiko/rye

https://github.com/mitsuhiko/rye/discussions/6

こちらもRust製です。

## Rインストールマネージャー rig

次に気になるのはRでも同じようなものはあるのかということですよね？
ここに"The R Installation Manager"と書かれたrig[^rig]があります。

[^rig]: 当初R Install Managerを略したrimでしたが、恐らくCRAN上に存在するrimというRパッケージと名前が被っているためか、rigに改称されました。

https://github.com/r-lib/rig

詳しくはREADMEを読んでいただきたいのですけれども、rigはWindows（amd64）、macOS（amd64およびARM64）、Linux（の一部ディストリビューション、amd64およびarm64）に対応した、Rを管理し便利な機能を提供するCLIツールです。
例えば複数バージョンのRをコマンド一つで切り替えることができます。

注意点としてrigで管理できるのはrig経由でインストールされたRのみということで、既にインストールされているRを使用することはできません。
なのでrigを使用する場合、Rより先にrigをインストールする必要があります。

### rigでRをインストールする

Windows上でrigを試してみましょう。ここではWindows Sandboxを使用し、wingetでrigをインストールするところから始めます[^sandbox]。

[^sandbox]: Windows Sandbox上にwingetを導入する方法はこちら。 https://learn.microsoft.com/ja-jp/windows/package-manager/winget/#install-winget-on-windows-sandbox

`winget search`でrigを検索してみます。今更ですがPosit社が開発しているのでwinget-pkgsには`posit.rig`で登録されています[^winget-pkgs]。

[^winget-pkgs]: 私が登録しました。 https://github.com/microsoft/winget-pkgs/pull/108337

```powershell:powershell
> winget search posit.rig
Name Id        Version Source
------------------------------
rig  Posit.rig 0.6.0   winget
```

現在バージョン0.6.0をインストールできるようですね。インストールしてみましょう。

```powershell:powershell
> winget install posit.rig
Found rig [Posit.rig] Version 0.6.0
This application is licensed to you by its owner.
Microsoft is not responsible for, nor does it grant any licenses to, third-party packages.
Downloading https://github.com/r-lib/rig/releases/download/v0.6.0/rig-windows-0.6.0.exe
  ██████████████████████████████  4.68 MB / 4.68 MB
Successfully verified installer hash
Starting package install...
Successfully installed
```

小さいツールなので、数秒でインストールできました。
次に`rig add`コマンドでRをインストールしてみましょう。バージョンを指定しない場合、最新バージョンがインストールされます。

```powershell:powershell
> rig add
[INFO] Cleaning leftover registry entries
[INFO] Downloading https://cran.rstudio.com/bin/windows/base/R-4.3.2-win.exe -> C:\Users\WDAGUT~1\AppData\Local\Temp\rig\x86_64-R-4.3.2-win.exe
[INFO] Installing C:\Users\WDAGUT~1\AppData\Local\Temp\rig\x86_64-R-4.3.2-win.exe
[INFO] Adding R-4.3.2 -> C:\Program Files\R\R-4.3.2
[INFO] Adding R-release -> 4.3.2 alias
[INFO] Setting default CRAN mirror
[INFO] Installing pak for R 4.3.2 (if not installed yet)
[INFO] > Installing package into 'C:/Users/WDAGUtilityAccount/AppData/Local/R/win-library/4.3'
[INFO] > (as 'lib' is unspecified)
[INFO] > trying URL 'https://r-lib.github.io/p/pak/stable/win.binary/mingw32/x86_64/bin/windows/contrib/4.3/../../../../../../../mingw32/x86_64/pak_0.7.1_R-4-3_x86_64-mingw32.zip'
[INFO] > Content type 'application/zip' length 8615600 bytes (8.2 MB)
[INFO] > ==================================================
[INFO] > downloaded 8.2 MB
[INFO] >
[INFO] >
[INFO] > The downloaded binary packages are in
[INFO] >        C:\Users\WDAGUtilityAccount\AppData\Local\Temp\RtmpMdsPJj\downloaded_packages
```

今日時点での最新バージョンであるR 4.3.2がインストールされました。
次に`rig add 4.2`で、R 4.2をインストールしてみましょう。

```powershell:powershell
> rig add 4.2
[INFO] Cleaning leftover registry entries
[INFO] Downloading https://cran.rstudio.com/bin/windows/base/old/4.2.3/R-4.2.3-win.exe -> C:\Users\WDAGUT~1\AppData\Local\Temp\rig\x86_64-R-4.2.3-win.exe
[INFO] Installing C:\Users\WDAGUT~1\AppData\Local\Temp\rig\x86_64-R-4.2.3-win.exe
[INFO] Adding R-4.2.3 -> C:\Program Files\R\R-4.2.3
[INFO] Setting default CRAN mirror
[INFO] Installing pak for R 4.2.3 (if not installed yet)
[INFO] > Installing package into 'C:/Users/WDAGUtilityAccount/AppData/Local/R/win-library/4.2'
[INFO] > (as 'lib' is unspecified)
[INFO] > trying URL 'https://r-lib.github.io/p/pak/stable/win.binary/mingw32/x86_64/bin/windows/contrib/4.2/../../../../../../../mingw32/x86_64/pak_0.7.1_R-4-2_x86_64-mingw32.zip'
[INFO] > Content type 'application/zip' length 8159272 bytes (7.8 MB)
[INFO] > ==================================================
[INFO] > downloaded 7.8 MB
[INFO] >
[INFO] >
[INFO] > The downloaded binary packages are in
[INFO] >        C:\Users\WDAGUtilityAccount\AppData\Local\Temp\RtmpmWhH9Q\downloaded_packages
```

R 4.2.Xの最終バージョンであるR 4.2.3がインストールされました。
`rig list`コマンドでインストール済みのRを一覧表示できます。`*`マークが付いているのが現在選択されているバージョンです。

```powershell:powershell
> rig list
* name   version  aliases
------------------------------------------
  4.2.3
* 4.3.2           release
```

`rig run`コマンドを実行すると、現在選択されているR 4.3.2が起動します。
同じように、cmd.exeから`R`、PowerShellから`R.bat`を実行しても、選択されているRが実行されます。
`R`の実体が`R.bat`であることから推測できるように、rigは`R.bat`の中身を書き換えて`R`で呼び出されるRのバージョン変更しています。

macOSとLinuxの場合はシムリンクを書き換えることで同じ機能が実現されています。

https://github.com/r-lib/rig/blob/1e335785f95c3669bf04a43bc6a0da4862e401db/src/alias.rs

WindowsにRをインストールすると古いバージョンのファイル達が残ったまま新しいバージョンが入るので新しいバージョン入れるときにパスを通し直したり面倒だと思っていたのですが、rigを使えばrigのコマンドで切り替えられて便利ですね。

## Rパッケージインストールの何でも屋 pak

先ほどrigでRをインストールしたときのログで、`pak`なるものもインストールされていたことに気付いたでしょうか？
[`pak`](https://pak.r-lib.org/)はRパッケージをインストールするための多機能パッケージです。

様々な機能があるので詳細はドキュメントを確認してください。
一番覚えやすい関数は`pak::pak()`で、これは`install.packages()`と同じ単純なデフォルトリポジトリからのインストールに加えて、`remotes::install_github()`のようにGitHubからのインストールなどもできます。

最近では`dplyr`などのパッケージのREADMEでは開発バージョンのインストール用として書かれている関数が`remotes::install_github()`から`pak::pak()`に切り替わっています。

```r:R
# GitHubのtidyverse/dplyrリポジトリのHEADからdplyrをインストール
pak::pak("tidyverse/dplyr")
```

https://github.com/tidyverse/dplyr/blob/b359331a448a693546d77245b0de4d405bab3886/README.md?plain=1#L75-L83

GitHub上でRパッケージを開発している方はGitHub Actionsで`r-lib/actions/setup-r-dependencies`を使って依存Rパッケージをインストールしているかも知れません。このアクション内で`pak`が使用されています。
Linux上にRパッケージをインストールする場合はシステム依存関係をR外でインストールしなければならないことが多々ありますが、`pak`はシステム依存関係を自動的にインストールする機能を持っているため、GitHub ActionsでUbuntuランナーを使用してもシステム依存関係を気にしなくて済むようになっています。

ソースインストール時にビルド中のログを表示しないのでデバッグしづらいなどの細かい欠点もありますが多機能で様々な場面で役に立つので、ぜひ一度公式ドキュメントを読んでみてください。

数ヶ月前にバージョン1.0.0に到達した、プロジェクト毎のパッケージ固定を可能にする`renv`パッケージには`pak`をバックエンドとして使用するオプションがあったりもします。

## バイナリパッケージインストールの強い味方 p3m

Posit Package Manager（旧称RStudio Package Manager）という、Posit社の製品があります。
