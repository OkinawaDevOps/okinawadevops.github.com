
---
layout : post
tags : [OpenStack, ]
---

{% include JB/setup %}

## OpenStack 続き

途中からの上に、終われてないので参考にならない説があります。ご注意。日本仮想化技術様から pdf をインストールして、作成しています。

### 環境

* VMWare 上の Fedora 20
* OpenStack havana
* keystone までは終わっている

### mariadb 自動起動

データベースを自動起動するように。Fedora の MySQL は Mariadb になっているので、コマンドもこんな感じ。
    # systemctl enable mariadb.service


### mariadb password

    # mysqladmin -u root password
    New password:
    Confirm new password:


### mariadb の設定

    vim /etc/my.cnf.d/server.cnf
    
    bind-address = 127.0.0.1
    # default-strage-engine = innodb
    collation-server = utf8_general_ci
    init-connect = 'SET NAMES utf8'
    character-set-server = utf8

### MariaDB ってなに？

[MariaDB](https://mariadb.com/about)

* MySQL を fork して立ち上げられたプロジェクト。
* google が MySQL を捨てて MariaDB に移行している。
* MySQL と SQL 部分はさほど変わらない？
* 時間があったらもっと調べてみたい。

### keystone 再起動

fedora では service の名前が openstack-keystone になっているので service 起動時は下記のように。fedora は「openstack-」がついてる。
    # service openstack-keystone start
    # systemctl enable openstack-keystone.service

    # export OS_SERVICE_TOKEN=password
    [root@localhost]/home/shien/openstack# export OS_SERVICE_ENDPOINT=http://localhost:35357/v2.0

keystone の endpoint を、localhost 上なら下記のように設定すれば、
    # export OS\_SERVICE\_ENDPOINT=http://localhost:35357/v2.0

こんな感じで表示できる。

    # keystone user-list
    WARNING: Bypassing authentication using a token & endpoint (authentication credentials are being ignored).
    +----------------------------------+--------+---------+-------------------------------+
    |                id                |  name  | enabled |             email             |
    +----------------------------------+--------+---------+-------------------------------+
    | b4dc3ab16d7f47db819b860ce283564e | admin  |   True  |      sienrizumu@gmail.com     |
    | 4f4824c0e19444eb8cc1cdd22f876d3b |  demo  |   True  | earthquake-moon@hotmail.co.jp |
    | a66db22a936d43129483f245ae659f5d | glance |   True  |      sienrizumu@gmail.com     |
    +----------------------------------+--------+---------+-------------------------------+


/etc/glance/grance-api.conf に設定記述
/etc/glance/grance-registry.conf に設定記述


### サービスを再起動する。

ついでに自動起動するようにも設定
    # service openstack-glance-registry restart
    Redirecting to /bin/systemctl restart  openstack-glance-registry.service
    # service openstack-glance-api restart
    Redirecting to /bin/systemctl restart  openstack-glance-api.service
    # systemctl enable openstack-glance-api.service && systemctl enable openstack-glance-registry.service

### fedora 本家から cloud image を download

    wget http://download.fedoraproject.org/pub/fedora/linux/updates/20/Images/x86_64/Fedora-x86_64-20-20140407-sda.qcow2

### nova の /etc/nova/nova.conf

    rpc_backend=nova.openstack.common.rpc.impl_kombu
rpc_backend=rabbitmq だけじゃだめみたい


## まとめ

* nova / glance / keystone のインストールとか環境設定してました
* nova と glance の ERROR: Unable to sign token. (HTTP 500) に悩まされている最中。早く何とかしないと。
* しかも docker まで行けませんでした
* いっそスクリプトや puppet でインストールしたほうが……って感じに心折れかけてます
* docker 使うには driver いれて nova と glance の設定すれば……のようです。
* [参考](https://wiki.openstack.org/wiki/Docker#Installing_Docker_for_OpenStack)
* 将来的には OpenStack で、ansible との連携しつつ bare machine / container / VM を操作できるようにしたい
