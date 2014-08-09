---
layout: post
tags : [Ansible, ]
---
{% include JB/setup %}

### [kinjo](https://github.com/kinjo)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/55)

#### docs.ansible.com の Introduction を読む

[docs.ansible.com](http://docs.ansible.com/) な内容のうち [Introduction](http://docs.ansible.com/intro.html) を読む。

Introduction 以下な内容あり

  * Installation
  * Getting Started
  * Inventory
  * Dynamic Inventory
  * Patterns
  * Introduction To Ad-Hoc Commands
  * The Ansible Configuration File
  * Windows Support

Introduction To Ad-Hoc Commands まではすでに読了のため、
本日は The Ansible Configuration File 以降を読む。

The Ansible Configuration File, Windows Support の概略は以下。

さらに [Playbooks](http://docs.ansible.com/playbooks.html) も少し読めたため、以下に記載。

##### [The Ansible Configuration File](http://docs.ansible.com/intro_configuration.html)

Ansible の特定の設定は設定ファルで調整可能。
大抵の利用者にはデフォルトの設定で十分だが、変更したい理由はあるかも。

以下な設定ファイルが使用され、以下な順番で処理される。

  * ANSIBLE_CONFIG (an environment variable)
  * ansible.cfg (in the current directory)
  * .ansible.cfg (in the home directory)
  * /etc/ansible/ansible.cfg

1.5 より前の順番は以下だった。

  * ansible.cfg (in the current directory)
  * ANSIBLE_CONFIG (an environment variable)
  * .ansible.cfg (in the home directory)
  * /etc/ansible/ansible.cfg

上な一覧のファイルのうち、 Ansible は最初に見つかったものを処理する。
ファイルの設定はマージされない。

パッケージマネジャーを使用する場合、最新の ansible.cfg は /etc/ansible の中にあるべきで、
場合により、アップデートのケースに応じて .rpmnew ファイル（または他）にある。

pip またはソースからインストールする場合、
Ansible のデフォルトの設定を上書きするために、あなたがこのファイルを作成するかもしれない。
（pip でインストールする場合 /etc/ansible/ansible.cfg は作成されないということ ?）

Ansible は環境変数を介した設定もできる。環境変数変数が設定されていれば、
設定ファイルの設定を上書きする。これらの変数は簡単のためここでは定義されないが、
お望みならソースツリーにある constants.py を参照のこと。

設定ファイルはセクションに分割される。大抵のオプションは "general" セクションにあり、
いくつかのファイルセクションは特定のコネクション形式に固有にある。

以降、スルー。

##### [Windows Support](http://docs.ansible.com/intro_windows.html)

Ansible はデフォルトでは SSH を使用し Linux/Unix マシンを管理するものと読んだかもしれない。

バージョン 1.7 から Ansible は Windows マシンのサポートも含む。
これは SSH でなくネイティブ powershell remoting を使う。

Ansible は Linux コントロールマシンで走り、
Python モジュール winrm を使いリモートマシンと対話する。

リモートマシンに Ansible の管理のための追加パッケージをインストールする必要はなく、
エージェントレスという特性は維持される。

Windows はしばらく使用する想定がないためスルー。

##### [Playbooks](http://docs.ansible.com/playbooks.html)

Playbooks は Ansible の設定、デプロイ、オーケストレーション用言語。

基本的なレベルは、 Playbooks はリモートマシンに対する設定やデプロイを管理など。

より進んだレベルにおいては、モニタリングサーバやロードバランサと対話しつつ、
ローリングアップデートを行う複数層の展開を順序付けしたり、
他のホストへのアクションを委託できる。

Playbooks はヒューマンリーダブルにデザインされ、基本的なテキスト言語で開発される。

playbooks と playbooks を含むファイルの管理には複数の手法があり、
Ansible を最大限活用できるいくつかの提案を提供する。

playbook ドキュメントを読みながら Example Playbooks を見ることを勧める。
ベストプラクティス、および様々な概念が説明されている。
