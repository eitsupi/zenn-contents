---
title: Dev Container Featuresを使ったり公開したりしました
emoji: 🐳
type: tech
topics:
    - r
    - docker
    - vscode
    - devcontainer
published: true
---

:::message

これは[R Advent Calendar 2022](https://qiita.com/advent-calendar/2022/rlang) 18日目の記事です（が、Rとあまり関係のない話です）。

:::

## 前置き

先日開催された[Japan.R 2022](https://japanr.connpass.com/event/265366/)において「Dev Container Featuresで合体だ！」というタイトルで発表しました。（[スライド](https://eitsupi.github.io/tokyorslide/japanr2022lt)）

「プログラミング環境にDev Containerを使用すると捗るぞ！」という内容なのですけれど、一番言いたかったのは2022年後半に本格的に実装された[Dev Container Features](https://containers.dev/implementors/features/)に私はすごく可能性を感じているということでした。
Dev Container Featuresについてまだあまり情報がないように思うので検索に引っかかりやすい記事の形でここに書いておきたいと思います。

なお、もちろんこの記事もGitHub Codespaces上のDev Containerで書かれています。
2コア4GB RAMのスペックでも充分使える感じでオススメです。

![](/images/japanr2022-dev-container/screenshot-vscode.png)
*ローカルのVS Codeから接続する方法が好きです*

## Dev Container Featuresとは

Dockerfileの断片とコンテナ設定の断片をパッケージングしたようなものです。

昔は`VSCode Remote - Containers`拡張機能（最近`VSCode Dev Containers`に改名）に組み込まれていたシェルスクリプトで、拡張機能内での利用に制限されていました。
現在はDev Container Specとして標準化され、proposalのステータスも外れ、誰でも作成して公開・配布・利用できるようになっています。

例として、Dev Container CLIを使用してDev Container Featureをコンテナ内で開発する場合に使用されている[^1]Dev Container定義を見てみましょう。

[^1]: https://github.com/devcontainers/feature-starter/blob/aac19152fc12052fa33f79e1041ae6b1b624c395/.devcontainer.json

```json:.devcontainer.json
{
    "image": "mcr.microsoft.com/vscode/devcontainers/javascript-node:0-18",
    "features": {
        "ghcr.io/devcontainers/features/docker-in-docker:2": {}
    },
    "postCreateCommand": "npm install -g @devcontainers/cli"
}
```

これだけでnodeインストール済のイメージ上にDocker-in-Dockerが設定され、コンテナ内でDev Container CLIが使用可能になります。
このjsonファイル以外にDockerfile等の定義ファイルやコマンドラインオプションは一切必要ありません。

別の例として、最近私の作った[Rustコードを含むRパッケージ](https://github.com/eitsupi/prqlr)のコンテナ定義は以下のようになっています。

```json:.devcontainer/devcontainer.json
{
    "image": "ghcr.io/rocker-org/devcontainer/r-ver:4",
    "features": {
        "ghcr.io/devcontainers/features/rust:1": {
            "version": "latest"
        }
    }
}
```

Rバージョン4のコンテナ上に最新バージョンのRustをインストールしている感じです。

ちなみに、イメージのラベルやDev Container Feature内にVSCode用の設定やインストールする拡張機能の設定が書き込まれているため、ここからVSCodeやCodespacesでビルドするだけでVSCodeの設定はバッチリ決まります。

## Dev Container Featuresの配布

先ほどのファイルの`features`以下にある`ghcr.io/devcontainers/features/docker-in-docker`や`ghcr.io/devcontainers/features/rust`というDev Container Featureの名前でピンときた方もいらっしゃるかと思いますが、
Dev Container FeaturesはDockerイメージなどのようにOCIレジストリで配布可能になりました。

GitHub上で配布する場合、公式のGitHub Actionsを使用して簡単に公開・配布できます。

例えば現在いくつかR関連のDev Container FeatureがVSCode Dev ContainerのUIやGitHub Codespacesの[Dev Containerエディタ](https://github.blog/changelog/2022-10-21-codespaces-configuration-with-the-dev-container-editor/)で見つかりますが、それらは以下のリポジトリで私の公開したものです。

https://github.com/rocker-org/devcontainer-features

![](/images/japanr2022-dev-container/screenshot-r-apt.png)
*GHCRで公開する場合、ちゃんとDockerイメージとは異なる専用表示になります*

## VSCodeやCodespacesのUIからFeatureを選ぼう

![](/images/japanr2022-dev-container/screenshot-add-feature.png =450x)
*VSCodeのDev Container Feature選択UI*

VSCodeやCodespacesのUIに表示させたい場合、devcontainers公式サイト上のインデックスに登録を行います。
具体的には以下のYAMLファイルを更新するプルリクエストを送信します。

https://github.com/devcontainers/devcontainers.github.io/blob/gh-pages/_data/collection-index.yml

基本的に来る物は拒まずの精神のようで、特に審査もなく登録してもらえると思います。
登録されている`ociReference`は定期的にクロールされ、そこに含まれているFeatureは以下のページに表示されます。

https://containers.dev/features

## まとめ

- Dev Container Featureを使用するとDockerfileを書かず、単純なjsonファイルの編集のみで複数のツールをインストールした開発環境を作成できます。
- Dev Container Featureは簡単に作成・配布・公開できます。
- オープン仕様なので、サポートツールがもっと増えると良いなと思います。
- CodespacesはGitHub全ユーザー無料で使えるようになったので試してみてください。
