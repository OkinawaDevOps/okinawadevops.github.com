---
layout: post
tags : [Ansible]
---
{% include JB/setup %}

### [Yosuke OTA](https://twitter.com/y0t4)

* [やること](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/15)

#### ひっかかりまくり

apt-get upgradeするとkernelパニクったり、locale変更しようとしても全くいじれなかったり...。

#### AnsibleのModuleについて

* 言語について

Moduleの作成はサーバ上で実行できる言語ならなんでもいいようです。

ただし、以下の入出力のフォーマットは守る必要があります。

    入力: 引数名1=値1 引数名2=値2 ...
    出力: 基本的にJSON
          Shell向けに以下の内容も許可されています。
          key=value rc=0 changed=true favcolor=green

** 注意点

出力は *必ず* 上記のものである必要があります。

そのためshell等で開発する際には以下のように $CMD の出力は /dev/null に投げましょう。
    $CMD > /dev/null 2>&1
    if [ $? -eq 0 ]; then
      echo "changed=True"
      exit 0
    else
      echo "changed=False"
      exit 1
    fi

* Module 置き場

AnsibleではModuleを以下の場所から検索します。

    ansible.cfg の library で指定されたディレクトリ
    環境変数 ANSIBLE_LIBRARY で指定されたディレクトリ
    コマンドラインで --module-path で指定されたディレクトリ
    実行ディレクトリ直下にある library という名前のディレクトリ

お手軽に試したい場合は、実行ディレクトリ直下に library/ を作成するのが良さそうです。

* 簡単なpingモジュールを書いてみた

<script src="https://gist.github.com/yosukeota/8f0643ce1996a9b69eba.js"></script>
