---
title: PositronをDev Containerで使いたい（DevPodを使おう！）
emoji: 📦
type: tech
topics:
    - r
    - docker
    - devcontainer
    - positron
published: true
published_at: 2025-07-11
---

## この記事のゴール

- DevPodについて知る。
- コンテナ内に隔離された開発環境でPositronを使用できることを確認する。

## まえおき

皆さん、[Positron](https://positron.posit.co/)使われていますか？

私は`git clone`した直後にやるルーティンが

1. `.git/info/exclude`を開き
2. `.devcontainer`を除外対象にし
3. `.devcontainer/devcontainer.json`を追加する

なくらいDev Containerを常に使っているコンテナ過激派（？）なので、Dev Containerに対応していないPositronは今まで使っていませんでした。Visual Studio Codeばかり使ってます。

しかしひょっとしたことから[DevPod](https://devpod.sh/)がPositronをサポートしていることを知り使ってみましたので、やり方について軽くご紹介します。

ついでに[^r]、RをDev Containerを使う際のヒントになりそうな情報も共有します。

[^r]: RをDev Containerで良い感じに使えるようにするためのDev Container Featureをいくつも作って管理しており詳しいため。

:::message

執筆時点（Positron 2025.07.0-204、DevPod 0.7.0-alpha.34）ではDevPodによるPositronサポートは実験的とされています。
また、Positronはコンテナ内に[Remote SSH](https://positron.posit.co/remote-ssh.html)で接続するため、現時点では接続先のコンテナはPositron側の制限によりamd64プラットフォームでなければならないと思われます（動作未確認）。

:::

## DevPodとは

2年半ほど前の記事で、以下のように書きました。

https://zenn.dev/eitsupi/articles/japanr2022-dev-container

> - オープン仕様なので、サポートツールがもっと増えると良いなと思います。
> - CodespacesはGitHub全ユーザー無料で使えるようになったので試してみてください。

Codespacesがその後どのくらい普及したのかはよく分かりませんが、DevPodというのは私の待望していた、公式実装（[Dev Container CLI](https://github.com/devcontainers/cli)）とは別の、Dev Container SpecをサポートするOSSです。
GitHub上で開発されており、リポジトリはこちらになります。

https://github.com/loft-sh/devpod

「Codespaces but open-source, client-only and unopinionated」と説明されており、プロプラエタリであるGitHub CodespacesみたいなことのできるOSSで、様々な環境で動作するように作られています。

バックエンド（プロバイダー）としてローカルのDockerや各社のクラウドを選択でき、フロントエンド（IDE）も様々なものに対応しています。詳しくは公式ドキュメントでご確認ください。

https://devpod.sh/docs/what-is-devpod

## DevPodによるDev Containerの作成

### インストール

今回はプロバイダーとして同一マシン上のDockerを使用するので、あらかじめDocker（今回はWindows用のDocker Desktop）をインストールしておきます。
Positronもインストールします。

最後にDevPod（今回はDesktop）もインストールして、プロバイダーにDockerを選択しておきます。

:::message

次のワークスペース作成時にはGitコマンドも使用されるため、Gitもインストールされている必要があると思われます。

:::

### ワークスペースの作成

DevPodにはGitHubのURLを指定するだけで、コンテナ内で作業を開始できる機能があります（DevPodの管理しているディレクトリにgit cloneしてバインドマウントを裏側でやってくれているようです）。

:::message

Windows上で規定の構成だとWindows側のファイルシステムにリポジトリをcloneしてくるため、速度を気にする場合はWSL2側にcloneするよう設定変更なり手動でcloneしたリポジトリをパスで指定する方が良いと思われます。
DevPodの使い方についてはGitHub上のissue等を参考にしてみてください。

:::

今回はRをサクッと動かしてみるために、以下の画像のように`https://github.com/rocker-org/devcontainer-try-r`と指定し、このリポジトリにあるコンテナ定義を使ってみます。

ProviderはDocker、Default IDEはPositronを選択しています。

![DevPodでワークスペースを作成する](/images/devpod-positron-devcontainer/create-workspace.png)

https://github.com/rocker-org/devcontainer-try-r

今回使用するリポジトリを見ていただくと、規定のパスである`.devcontainer/devcontainer.json`にはファイルは存在せず、`.devcontainer/*/devcontainer.json`という一つ下の階層に複数のDev Container定義があります。
これは一つのリポジトリに複数のコンテナ定義を含めるための方法で、このリポジトリはいくつかのコンテナ構成を紹介するためにこのようになっています。

この内のどれか一つを選んでコンテナを作成します。今回は`.devcontainer/template-r2u/devcontainer.json`を指定してみましょう。

:::message

`.devcontainer/devcontainer.json`が存在せず「Devcontainer Path」が指定されていない場合、`mcr.microsoft.com/devcontainers/base:ubuntu`が使用されるようです。

:::

![devcontainer.jsonのパスを指定する](/images/devpod-positron-devcontainer/devcontainer-path.png)

この`devcontainer.json`の中身は以下のようになっており、基本的にベースイメージである`mcr.microsoft.com/devcontainers/base:noble`にRを設定するDev Container Featureである`ghcr.io/rocker-org/devcontainer-features/r-apt:0`を追加しているだけです。

https://github.com/rocker-org/devcontainer-try-r/blob/f93cbfa93f75fc991075bcf436529fb3dfad71ba/.devcontainer/template-r2u/devcontainer.json

「Create Workspace」ボタンを押すとコンテナのビルドが始まり、しばらくするとPositronが起動しSSH接続を始めます。

![PositronからのSSH接続の様子](/images/devpod-positron-devcontainer/settingup-ssh.png)

特にユーザーがすることはなく、待っていればコンテナへの接続は完了します。

### ワークスペース内での作業

IDEからの接続が確立された後は、普通に作業をするだけです。簡単！

今回作ったコンテナはUbuntu上に[r2u](https://eddelbuettel.github.io/r2u/)（と[bspm](https://cran4linux.github.io/bspm/)）を設定する構成なので、`install.packages()`関数を使用してのパッケージインストールが早いです。試してみましょう。

例えば以下のように、みんな大好きなtidyverseパッケージをインストールするコマンドをコンソールから実行します。

```r:R
install.packages("tidyverse")
```

私が実行したところ6秒でインストールされました。早い！

このようにr2uを構成するとRパッケージを高速でインストールできるので何かと便利なのですけれども、最新のRで最新のパッケージしかバイナリインストールできません。
バージョン固定したい場合はr2uを使わない方が良いでしょう。

その辺の、RをDev Container上で構成する場合のヒントは以下のページに書いてあるので、参考にしてください。

https://rocker-project.org/images/devcontainer/features.html

### コンテナの操作

DevPod Desktopの画面上でコンテナの停止や再開、削除などの操作を行えます。Docker Desktopに似ていますね。

![ワークスペースの管理画面](/images/devpod-positron-devcontainer/devpod.png)

IDEを選択しながらの再開もできますので、色々なIDEを試してみるのも簡単そうで良いですね。

![既存コンテナを開始する際にもIDEを選択できる](/images/devpod-positron-devcontainer/start-with.png)

## まとめ

DevPodを使うことでDev Containerの作成とPositronからの接続を簡単に行えることを確認しました。DevPodの体験はとても良かったので、コンテナ内での開発に興味のある方はぜひ試してみてください。

Positronは最近プレリリース扱いではなくなったので、今後このような形で活用する人も増え、良い感じの使い方が共有されていくと良いですね。

## 公式ドキュメント

- [DevPod](https://devpod.sh/)
- [Positron](https://positron.posit.co/)
- [Development Containers](https://containers.dev/)
