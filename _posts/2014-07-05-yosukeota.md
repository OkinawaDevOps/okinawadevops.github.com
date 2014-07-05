---
layout: post
tags : [Ansible, Vagrant]
---
{% include JB/setup %}

### [Yosuke OTA](https://twitter.com/y0t4)

* [やること](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/48)

### Vagrantの運用方法考えてみた
### Why?
Vagrantは初期状態ではセキュリティ的によろしくない

* user id = vagrant
* password = vagrant
* rsa key = ~/.vagrant.d/insecure_private_key

> ファイアウォールの中だとパスワードいらない、ってのは暴論だよね。おかしいよね。  
by kono先生

それじゃあ、**Vagrantの運用方法について考えてみよう**

### そもそもvagrantって
開発環境を簡単に作ったり、壊したり、クローンしたり、コピーして人に投げることができるツール

クローンする場合を考えると**ログイン方法は同一の方が良い？**

### パッケージし直すときのめんどくささ
`~/.bash_history`とか、自分向けな内容はできるだけ消したい

### 解決策
自分向けのuserを作ってvagrantはlogin shellを`/bin/false`にしてしまおう！

* パッケージし直すとき
  * userを消すと同時に`~user/`を消す
  * vagrantのlogin shellを`/bin/bash`に戻す

### 自動化ノススメ
**Ansibleを使ってProvisioning!**

[VAGRANT DOCS](https://docs.vagrantup.com/v2/provisioning/ansible.html)

* vagrantのprovisioningが走るのは...
  * 初回の`vagrant up`
  * `vagrant provision`
  * `vagrant reload --provision`

### 今回の必要条件
* 新しいuserを`useradd`
  * Home Directoryも作成
  * sudoersに追加
    * NOPASSWDで
  * ssh用の鍵を作成
    * `~user/.ssh/authorized_keys`に公開鍵を記述
* VM内の`/vegrant/Vagrantfile`を変更
  * `config.ssh.username`を記述
  * `config.ssh.private_key_path`を記述
* user: vagrantを`/sbin/nologin`に

### 作成したansibleなrepository
[ansible-setuser-for-vagrant](https://github.com/y0t4/ansible-setuser-for-vagrant)

### 使い方(2014-07-05時点)
1. 鍵の作成
  1. `ssh-keygen -t rsa -f [key_path]`
1. `group_vars/all`の編集
  1. `new_user`に自分の作成したいアカウントIDを指定
    1. `new_user: 'example'`
  1. パスワードハッシュの生成
    1. `python -c "from passlib.hash import sha512_crypt; print sha512_crypt.encrypt('<PASSWORD>')"`
  1. 鍵のpathを指定
    1. `public_key_path: '~/.ssh/example.pub`
    1. `private_key_path: '~/.ssh/example`
1. Vagrantfileの編集
  1. 以下のコードを追記
  ```
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
  end
  ```
1. 実行
  1. `vagrant provision`
