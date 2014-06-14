---
layout: post
tags : [Docker]
---
{% include JB/setup %}

## [kinjo](https://github.com/kinjo)
Docker 1.0 がリリースされ、何が変わったのか知りたく以下なドキュメントを読む。

[http://blog.docker.com/2014/06/announcing-docker-hub-and-official-repositories/](http://blog.docker.com/2014/06/announcing-docker-hub-and-official-repositories/)

結論： Docker Hub が提供開始されたということ。以下、読んだときのメモ。

## 読みメモ
DockerCon で以下な重要発表あり

- エンタープライズサポートもする Docker 1.0 リリース
- Docker Hub とオフィシャルリポジトリ

Docker をサポートするプラットフォーム

- Red Hat
- OpenStack
- Google
- AWS
- Ubuntu

Docker の名前のあるプロジェクトも多数あり

- [https://github.com/search?q=docker&ref=cmdform](https://github.com/search?q=docker&ref=cmdform)
  - ちなみに emacs の名前も一緒に検索してみたところwktk
  - [https://github.com/search?q=docker+emacs&type=Repositories&ref=searchresults](https://github.com/search?q=docker+emacs&type=Repositories&ref=searchresults)

コンテナエンジンが主目的から、配布アプリケーションのビルド、運搬、
実行のためのフルオープンなプラットフォームに Docker はなる。

    PLATFORM= Docker Engine + Docker Hub +APIs + Ecosystem

[http://blog.docker.com/wp-content/uploads/2014/06/platform.gif](http://blog.docker.com/wp-content/uploads/2014/06/platform.gif)

## Docker Hub
- Docker Hub はコンテンツであり、コラボレーションであり、ワークフローである
  - Docker Engine と密接に連携
  - コンテナイメージの配布など、アプリケーション配布に向けた幅広いサービスを提供
  - 管理者、利用者、チームコラボレーション、ライフサイクルフローの自動化、
    サードパーティサービスとの統合を変化させる

Build/Ship/Run のサイクルと Docker Hub の位置づけを説明する絵

[http://blog.docker.com/wp-content/uploads/2014/06/build_ship_run.gif](http://blog.docker.com/wp-content/uploads/2014/06/build_ship_run.gif)

いっそ、チームの作り方、コンテンツの見つけ方、アプリのビルドの仕方、
Jenkins のようなサービスとの統合の仕方、
同時に複数の異なるプラットフォームにデプロイする仕方を紹介するデモを見に
DockerCon のオープニングセッションにきてください。

これは Docker Hub の最初のリリース。多くの目玉の機能がまだまだ追加される。

メジャー機能は以下、

- ユーザ、チーム、コンテナ、リポジトリ、ワークフローを管理する統合コンソール
- Docker Hub Registry, 14,000 件以上の Dockerized なアプリケーションを提供
- 開発のコラボレーションツール
- 自動化されたビルドサービス
- Webhook で GitHub, AWS, Jenkins などと RESTful API を介して相互運用

サードパーティのアプリケーションとサービスが、
ユーザのパブリック、またはプライベートリポジトリにおいて、
アプリケーションへの認証済みのアクセスを得られるように
Docker Hub API はユーザ認証を含む。

サードパーティサービス、
AWS Elastic BeansTalk, Deis, Drone.io, Google Compute Engine, Orchard
Rackspace, Red Hat, Tutum, などなど
すでに Docker Hub API に統合されている。

Docker Hub のアカウントについて、とくに言っとく価値はたぶん、
登録されたユーザに対してフリーであること。
GitHub みたくプライベートリポジトリに課金するけど、それだけ。
フリーユーザにひとつのフリープライベートリポジトリを与える。

## オフィシャルリポジトリについて
ひとつの最もクリティカルな局面は、オフィシャルリポジトリプログラムのローンチである。

一般公開リポジトリにて、14,000を超えるDockerizedアプリケーションを保持する傍ら、
出展と品質の入念な保証とともに、
もっと監修されたイメージセットに対して大きな需要がある。

当初、Docker Hub Registryにおいて、包括的な検索最多のアプリケーショントップ13
 - CentOS, MongoDB, MySQL, Nginx, Redis, Ubuntu, WordPress を含む -

オフィシャルリポジトリプログラムは、任意のコミュニティグループに、
もしくはプログラムガイドラインに従ったアプリケーションの
進行中のメンテナンスにリソースをコミットすることをいとわない
独立系ソフトウェアベンダに対してオープンである。

例えば SyncSort は Irocluster ETL 製品のとあるバージョンを
Docker オフィシャルリポジトリにもつ。
