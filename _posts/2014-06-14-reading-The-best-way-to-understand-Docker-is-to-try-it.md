---
layout: post
tags : [Docker]
---
{% include JB/setup %}

## [kinjo](https://github.com/kinjo)

以下なトライイットを読む。

[http://www.docker.com/tryit/](http://www.docker.com/tryit/)

Docker 1.0 になるちょっと前に読んだので、少し内容が変わっているかもしれない。

サイトにエミュレータ端末が用意されていて、対話的にチュートリアルを進められる。

Ubuntu 14.04 上の Docker バージョン 0.11.1 でも動作することを確認。 ただし sudo は必要。

## Getting started

Docker デーモンと Docker クライアントがある。

docker と打つと利用できる引数が表示される。

docker version で今の Docker のバージョンを確認する。
デーモンが動いていることと接続できることを確認できる。

    docker version

## Searching for images

`docker search <string>` でコンテナイメージを探せる。

チュートリアル用のイメージを探す。

    docker search tutorial


利用できるコンテナイメージは Docker index にある。

Docker index へは [https://index.docker.io/](https://index.docker.io/) からアクセス。

## Downloading container images

コンテナイメージは docker pull でダウンロードする。

チュートリアル用のイメージをダウンロードする。

    docker pull lean/tutorial

## Hellow world from a container

コンテナとは箱の中のひとつのプロセスのようなもの。
コンテナはプロセスが必要とするあらゆるものを含む。
ファイルシステム、システムライブラリ、シェルなど。

docker run でコンテナの中でプロセスを走らせる。
プロセスが走り終わったらコンテナは停止する。

    docker run learn/tutorial echo hello world

## Installing things in the container

コンテナに ping をインストールする。
コンテナの中で apt-get install -y ping を実行する。

    docker run learn/tutorial apt-get install -y ping

コマンドが完了してコンテナが停止しても、変更点が維持されることに注目。

apt-get install コマンドに -y を指定しないと、失敗する。

## Save your changes

変更の後、後で変更点から再開できるよう、変更を保存したいとする。
Docker では、状態を保存することをコミットと呼ぶ。

ping のインストールで作成されたコンテナの ID を docker ps -l で確認できる。

コンテナ ID 6982a9948422 から新しくコンテナイメージを作成する。

    docker commit 6982a learn/ping

ID を全て打つことはなく、最初の３、４文字で十分な模様。

## Run your new image

ping をインストールし、新しく作成したコンテナイメージで ping を実行する。

    docker run learn/ping ping www.google.com

## Check your running image

docker ps で走っているコンテナの一覧を参照できる。
docker inspect でコンテナの各種情報を参照できる。

    docker ps

コンテナ ID が efefdc74a1d5 のとき、

    docker inspect efef

## Push your image to the index

index にコンテナイメージを push(アップロード)すれば、簡単に取得して再利用したり
他者と共有したりできる。

docker images でローカルにあるコンテナイメージを一覧する。
docker push でコンテナイメージを push する。

    docker push learn/ping

エミュレータでの試行のため push できたが、
実際の push には docker index にリポジトリ作成や認証が必要な模様。

実際やってみた手順は以下、

- docker login でログインする
- 予め docker index にリポジトリ kinjo/ping を作成しておく
- コンテナイメージは kinjo/ping のように作成する
- docker push kinjo/ping で push する
