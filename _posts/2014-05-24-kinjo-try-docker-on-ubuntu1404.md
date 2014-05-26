---
layout: post
tags : [Docker, Ubuntu 14.04, ]
---
{% include JB/setup %}

### [kinjo](https://github.com/kinjo)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/18)

#### Ubuntu 14.04な環境にDockerインストールまで

勉強会用に Ubuntu Server 14.04 な Box をアップしてあったので、それを使用。

    mkdir try-docker-on-ubuntu-1404
    cd try-docker-on-ubuntu-1404
    vagrant init kinjo/ubuntu-14.04-server-amd64

で Docker のデプロイまでを行う Vagrantfile を用意することに。

Docker なマニュアルによれば、

[http://docs.docker.io/installation/ubuntulinux/#ubuntu-trusty-1404-lts-64-bit](http://docs.docker.io/installation/ubuntulinux/#ubuntu-trusty-1404-lts-64-bit)

インストール手順はこう、

    sudo apt-get update
    sudo apt-get install docker.io
    sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker

上にならい同ディレクトリに bootstrap.sh を作成。これを Vagrantfile からキックする。

    #!/usr/bin/env bash
    apt-get update
    apt-get install -y docker.io
    ln -sf /usr/bin/docker.io /usr/local/bin/docker

Vagrantfile の末尾に bootstrap.sh のキックを追加。

    diff --git a/Vagrantfile b/Vagrantfile
    index 82bfe37..ff7bc51 100644
    --- a/Vagrantfile
    +++ b/Vagrantfile
    @@ -119,4 +119,5 @@ Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
       # chef-validator, unless you changed the configuration.
       #
       #   chef.validation_client_name = "ORGNAME-validator"
    +  config.vm.provision :shell, :path => "bootstrap.sh"
     end

動作確認はこう。歴代のイメージを pull してくれるらしく、十数分程度はかかる。こやつめ。

    sudo docker run -i -t ubuntu /bin/bash

以下なプロンプトが表示され、多分シェルに入ったと思われる。これで成功なのではないかと、おそらく。

    root@fcf1e4a1c3c1:/#

最後、シェルを出て sudo docker ps してヘンなコンテナが残ってないか確認しておわり。

以下、成果物。

Vagrant が入っていれば git clone して vagrant up/vagrant ssh すれば Docker 環境になるはず。
当方 OS X 10.9.2 にて確認。

[https://gist.github.com/kinjo/992e0622f538d8794faa](https://gist.github.com/kinjo/992e0622f538d8794faa)

<script src="https://gist.github.com/kinjo/992e0622f538d8794faa.js"></script>
