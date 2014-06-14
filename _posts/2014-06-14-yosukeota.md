---
layout: post
tags : [Ansible]
---
{% include JB/setup %}

### [Yosuke OTA](https://twitter.com/y0t4)

* [やること](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/34)

#### VagrantなvmをAnsibleでいじいじしたい！

#### Ansibleのインストール(Mavericks)

    $ sudo easy_install pip
    $ sudo pip install ansible

#### Ansibleを使ってVagrantにファイルを投げてみる

    $ touch ~/pantu
    $ cat hosts
    machine ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222
    $ ansible machine -i hosts --private-key=~/.vagrant.d/insecure_private_key -u vagrant -m copy -a "src=~/pantu dest=~/"

投げれた！

#### ちなみに？

`~/.ssh/config`に設定書いてるとさらにらくちん！

    $ cat <<EOF >>~/.ssh/config
    > Host vagrant
    > Hostname 127.0.0.1
    > User vagrant
    > Port 2222
    > IdentityFile ~/.vagrant.d/insecure_private_key
    > EOF
    $ cat hosts <<EOF
    > vagrant
    > EOF
    $ ansible vagrant -i hosts -m copy -a "src=~/pantu dest=~/"

#### 他にも

`Vagrantfile`にprovisionを指定する設定があるようで、そこでplaybookを指定すると実行してくれるらしい。

これは後日...。
