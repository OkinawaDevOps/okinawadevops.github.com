---
layout: post
tags : [Ansible, ]
---
{% include JB/setup %}



## [kinjo](https://github.com/kinjo)


-   [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/75)

Ansible で OpenStack Object Storage サービスを構築する。
認証は OpenStack Identity サービスを使用する。

OpenStack Object Storage サービスの構築手順は割愛し、
構築で得た Ansible 関連の知見を共有することを目的にする。

なお、 OpenStack Object Storage サービスの構成要素は以下に示す。
後述の導入手順書とセグメント構成は異なるが、簡単化のためひとつにする。

    +---------------+  +---------------+  +---------------+  +---------------+  +---------------+
    | ansible       |  | cloud control |  | storage node1 |  | storage node2 |  | storage node2 |
    | control       |  |               |  |               |  |               |  |               |
    |               |  | keystone      |  | account       |  | account       |  | account       |
    |               |  | swift-proxy   |  | container     |  | container     |  | container     |
    |               |  |               |  | object        |  | object        |  | object        |
    |               |  |               |  |               |  |               |  |               |
    +-------+-------+  +-------+-------+  +-------+-------+  +--------+------+  +-------+-------+
            |                  |                  |                   |                 |
    --------+------------------+------------------+-------------------+-----------------+--------

-   ansible control: Ansible を導入し playbook を配置
-   cloud control: keystone, swift-proxy を導入
-   storage node: swift-account, swift-container, swift-object を導入

以下の構築手順を参考に構築する。


-   [Chapter 3. Configure the Identity Service](http://docs.openstack.org/icehouse/install-guide/install/apt/content/ch_keystone.html)
-   [Chapter 10. Add Object Storage](http://docs.openstack.org/icehouse/install-guide/install/apt/content/ch_swift.html)

## 疑問点・気づいた点

以下の疑問点・気づき点を抽出した。解決出来なかった項目については今後の活動で解決していきたいところ。

1.  デバッグ手法

    Ansible の keystone\_user モジュールを使用し、 Keystone の初期ユーザ作成等を行おうとしたが、
    何らかのエラーで行われない事案があった。

    これは ansible-playbook の -vvvv オプションおよび ANSIBLE\_KEEP\_REMOTE\_FILES=1 を活用し解決した。

    -vvvv オプションは Ansible が実際に実行するコマンド等を詳細に標準出力する。
    以下のような詳細情報が出力される。

        TASK: [identity | create admin user] ******************************************
        <192.168.33.11> ESTABLISH CONNECTION FOR USER: vagrant
        <192.168.33.11> REMOTE_MODULE keystone_user user=admin password=VALUE_HIDDEN endpoint=http://localhost:35357/v2.0 token=password
        <192.168.33.11> EXEC ['ssh', '-C', '-tt', '-vvv', '-o', 'ControlMaster=auto', '-o', 'ControlPersist=60s', '-o', 'ControlPath=/home/vagrant/.ansible/cp/ansible-ssh-%h-%p-%r', '-o', 'StrictHostKeyChecking=no', '-o', 'KbdInteractiveAuthentication=no', '-o', 'PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey', '-o', 'PasswordAuthentication=no', '-o', 'ConnectTimeout=10', '192.168.33.11', "/bin/sh -c 'mkdir -p $HOME/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156 && chmod a+rx $HOME/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156 && echo $HOME/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156'"]
        <192.168.33.11> PUT /tmp/tmpkcmmCc TO /home/vagrant/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156/keystone_user
        <192.168.33.11> EXEC ['ssh', '-C', '-tt', '-vvv', '-o', 'ControlMaster=auto', '-o', 'ControlPersist=60s', '-o', 'ControlPath=/home/vagrant/.ansible/cp/ansible-ssh-%h-%p-%r', '-o', 'StrictHostKeyChecking=no', '-o', 'KbdInteractiveAuthentication=no', '-o', 'PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey', '-o', 'PasswordAuthentication=no', '-o', 'ConnectTimeout=10', '192.168.33.11', u"/bin/sh -c 'LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8 /usr/bin/python /home/vagrant/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156/keystone_user'"]

    keystone\_user モジュールの場合、
    対象のリモートマシン上に keystone\_user モジュールの処理スクリプトが一時的に展開される。
    （本件のエラーは以下のコマンドの実行で失敗していた。）

        /bin/sh -c 'LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8 /usr/bin/python /home/vagrant/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156/keystone_user'

    上記コマンドを、対象マシン上で直接実行するとエラーの詳細な内容がわかる。
    しかし対象マシンに展開されたスクリプトは ansible-playbook コマンドが終了すると削除される。

    環境変数 ANSIBLE\_KEEP\_REMOTE\_FILES に 1 を設定することで、スクリプトは削除されなくなる。

    具体的には以下のように実行する。

        ANSIBLE_KEEP_REMOTE_FILES=1 ansible-playbook site.yml -i development -vvvv

    ちなみに、あとは pdb を使用しデバッグできる。

        /bin/sh -c 'LANG=en_US.UTF-8 LC_CTYPE=en_US.UTF-8 /usr/bin/python /usr/lib/python2.7/pdb.py /home/vagrant/.ansible/tmp/ansible-tmp-1410398291.15-256266684151156/keystone_user'

<div id="footnotes">
<h2 class="footnotes">Footnotes: </h2>
<div id="text-footnotes">

<div class="footdef"><sup><a id="fn.1" name="fn.1" class="footnum" href="#fnr.1">1</a></sup> <p>DEFINITION NOT FOUND.</p></div>


</div>
</div>
