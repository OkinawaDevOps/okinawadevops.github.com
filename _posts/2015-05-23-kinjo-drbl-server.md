---
layout: post
tags : [Ubuntu, DRBL, Cloneziller, PXE]
---
{% include JB/setup %}

# [Takumi KINJO](https://twitter.com/libkinjo)



## やること

イシュー [issues/133](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/133) より、今回やることは以下。


-   [DRBL(Diskless Remote Boot in Linux)](http://drbl.org) 環境の整理（Vagrant + VirtualBox + Ansible）
-   README.md に補足事項の追加

これで何がしたいかというと、以下。


-   最終的には自宅のサーバ群を DRBL サーバで管理したい
    -   サーバ起動・停止の管理
    -   ネットワーク・インストールの管理
    -   バックアップ・リストアの管理
-   その前段の実験環境を Vagrant + VirtualBox + Ansible で用意しておきたい（イマココ）

## 成果


-   DRBL サーバ自動構築一式を [GitHub にアップ](https://github.com/kinjo/my-ansible-project/tree/drbl-server)
-   VirtualBox でクライアント用の VM を作成し remote-linux-gra モードで Linux が起動することを確認
    -   起動した Linux へのログインは vagrant/vagrant
-   DRBL サーバを netinstall モードに変更することで、クライアント起動時にネットワークインストールが開始されることを確認

成果物の用法は [README.md](https://github.com/kinjo/my-ansible-project/blob/drbl-server/README.md) を参照。

## 課題

以下の課題あり。どこまで Vagrant で DRBL 環境の構築を自動化するのかという話だが、
Vagrant で操作できることにこだわりは無いため以下はスルーの方針。


-   Vagrant を使用して DRBL クライアント用の VM を起動する方法が不明
    -   OS ナシで VM を起動させる方法が不明
    -   アダプター1 に PXE ブート用の Host-only ネットワークを接続させる方法が不明
        -   Vagrant ではアダプター1 に NAT ネットワークが接続される（この動作は変更できず、このため PXE ブートできない）

-   回避方法として直接 VirtualBox で VM を起動する（VM 起動するときは以下の点に注意）
    -   アダプター1 に PXE ブート用のホストオンリーアダプター（192.168.33.0/24 が設定された vboxent）を割り当てること
    -   PXE ブート用のネットワークは DHCP サーバが無効化されていること
    -   VM がネットワークから起動するよう起動順序が設定されていること

## 捕捉（DRBL サーバの起動モードの設定変更）

DRBL サーバの構築後、 DRBL サーバは remote-linux-gra モードで起動しており
DRBL クライアントを起動すると DRBL サーバにインストールされている Linux イメージから起動される。

以下を実行して起動モードを設定・変更できる。

    sudo dcs

起動モードには大雑把に以下のものがある。（他にもある）


-   netinstall モード
    -   ネットワークインストールを行う（設定後にクライアントを再起動すると、クライアントでネットワークインストールが開始される）
    -   Ansible で構築される DRBL サーバにはデフォルトで Ubuntu のインストールイメージがインストール済
-   Clonezilla モード
    -   DRBL クライアントのバックアップ／リストアを行う（クライアント起動時にバックアップ／リストアを行い、その後クライアントを再起動）
