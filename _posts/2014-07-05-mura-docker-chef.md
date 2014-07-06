---
layout: post
tags : [Docker, Chef]
---
{% include JB/setup %}

# [mura-](https://github.com/mura-)

Docker環境でもChef(chef-solo)の資産を使うべく、DockerとChefの連携を考える。

## why?
* Dockerfileをあまりいじりたくない。
* Chefで書いた資産（Recipe）を流用したい。

## how to
Chefとの連携自体やったことがないので...

1. Using Chef (Docker Official)  ChefでDockerをいれる
    http://docs.docker.com/articles/chef/
    * dockerをinstallしたあとcookbookをどうするかは書かれてないのでここでは割愛。
    * OPSCODEにdockerでchefが動くchef-dockerというものがある
        http://community.opscode.com/cookbooks/docker
        (source: http://community.opscode.com/cookbooks/docker )

2. Packerを使う
   参考: http://deeeet.com/writing/2014/03/02/build-docker-image-by-packer/

3. Dockerfileにchef実行コマンドを書き、docker build時に実行する
   参考: http://dev.classmethod.jp/server-side/docker-provisioning-use-chef/

### Using Chef
1. 環境設定
    * Chefをinstall
      オムニバスインストーラーを使う
    * knife-soloをinstall
    * berkshelfをinstall
    * Chefリポジトリを作成

### Packerを使う

当初boot2dockerで検証を試みるも、なぜかprovisioningが終わらなかったので、
一旦ubuntu上で検証。

1. vagrantにubuntuイメージを用意し、起動
2. ubuntu上にchefとpackerとdockerをinstallする
* chefのinstall
オムニバスインストーラーを使う。楽ちん

    curl -L http://www.opscode.com/chef/install.sh | bash

* packerのinstall
    ここからhttp://www.packer.io/downloads.html　ダウンロードしてきて、
    適当なところへ展開。PATHを通す。

* dockerのinstall
    パッケージ管理で。
    sudo apt-get update
    sudo apt-get install docker.io
    sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker

下記の用にVagrantfileに記述してしまってもよい。

    Vagrant.configure("2") do |config|
        config.vm.box = "precise64"
        config.vm.box_url = "http://files.vagrantup.com/precise64.box"
        config.vm.provision :docker do |d|
            d.pull_images "ubuntu"
        end
    
        config.vm.provision :shell, :inline => <<-PREPARE
    apt-get -y update
    apt-get install -y wget unzip curl
    
    mkdir /home/vagrant/packer
    cd /home/vagrant/packer
    wget https://dl.bintray.com/mitchellh/packer/0.5.2_linux_amd64.zip
    unzip 0.5.2_linux_amd64.zip
    echo "export PATH=$PATH:/home/vagrant/packer" > /home/vagrant/.bashrc
    PREPARE
    
    end

3. cookbook用意
Recipeの中身は省略します。

    knife cookbook create hogehoge -o site-cookbooks

4. Packerの設定ファイル用意

    {
        "builders":[{
            "type": "docker",
            "image": "ubuntu",
            "export_path": "image.tar"
        }],

        "provisioners":[
        {
            "type": "shell",
            "inline": [
                "apt-get -y update",
                "apt-get install -y curl"
            ]
        },
        {
            "type": "chef-solo",
            "cookbook_paths": ["site-cookbooks"],
            "run_list": ["hogehoge::default"]
        }
        ],

        "post-processors": [{
            "type": "docker-import",
            "repository": "name/hogehoge",
            "tag": "latest"
        }]
    }

5. packerでdockerイメージbuild

    packer build machine_chef.json

6. docker imageを見るとイメージが作成されている
    vagrant@vagrant-ubuntu-trusty-64:~$ sudo docker images
    REPOSITORY           TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    name/hogehoge   latest                 83e7a18fd734        18 minutes ago      380 MB

### Dockerfileにchef実行コマンドを書き、docker build時に実行する
Packerをいれると確かにDockerfileはノータッチでイメージの作成ができたが、
逆にPackerいれたりめんどくさい、という場合は下記のように最低限をDockerfileに記述するのはありかもしれない。

(Dockerがinstallされている前提、recipeが用意されている前提で)
1. Dockerfile用意

    FROM centos
     
    ENV CHEFHOME /chef-repo
    ADD chef-repo /chef-repo
     
    RUN apt-get -y update
    RUN apt-get -y curl
    RUN curl -L http://www.opscode.com/chef/install.sh | bash
    RUN cd ${CHEFHOME} && chef-solo -c ${CHEFHOME}/solo.rb -j ${CHEFHOME}/nodes/docker.json

2. DockerイメージBuild

    docker build -t name/hogehoge

3. docker imageを見るとイメージが作成されている
    vagrant@vagrant-ubuntu-trusty-64:~$ sudo docker images
    REPOSITORY           TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    name/hogehoge            latest              8f2151ea5244        59 seconds ago      420.2 MB
