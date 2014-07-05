---
layout: post
tags : [Docker]
---
{% include JB/setup %}

# [kinjo](https://github.com/kinjo)
Docker コンテナ内のプロセスの諸々がよくわからない中、以下なモノを発見したので読む。

[https://github.com/phusion/baseimage-docker](https://github.com/phusion/baseimage-docker)

結論：なんとなく使う分にも便利そうだが、美味しい問題がラップされているため、内容を理解しないとなんか勿体無い。

## 以降、読んだ内容
Baseimage-docker は Docker コンテナで正しく使用できるよう設定された Docker イメージ。
Docker フレンドリーな変更を加えた Ubuntu 。

Docker Registry で公開している。

### 普通の Ubuntu ベースイメージだと何が問題か？
Ubuntu は Docker の中で走るようデザインされていない。
Upstart の init system は、 Docker コンテナ中ではなく、物理ハードまたは仮想ハードで走っていることを仮定する。
しかし、コンテナ内では、あなたはいずれにせよフルシステムは望まず、小さなシステムを望む。
しかし、コンテナ内で使用するための小さなシステムの設定は、多くの未知の側面の場面があり、
Unix system モデルに精通していなければ正しく得ることは困難。
このことは、多くの見知らぬ問題を引き起こす。

Baseimage-docker は、全てを正しく得る。
”Container" セクションでは変更する事柄を示す。

### なぜ Baseimage-docker を使うのか？
- Docker フレンドリーなベースシステムを設定することは簡単でない。前に明記したとおり、
  多くの未知の側面がある。全てを正しく得るまでに、あなたは Baseimage-docker を再発明する。
  Baseimage-docker を使うことは、その努力を省く。
- 正しい Dockerfile を書くのに必要となる時間を減らす。
  ベースシステムを心配することはなく、あなたのスタックとアプリにフォーカスできる。
- docker build を走らせる時間を削減し、 Dockerfile のイテレーションをもっと早くできる。
- リデプロイの間のダウンロード時間を削減する。 Docker は、最初のデプロイの際、
  一度のベースイメージのダウンロードを必要とするだけである。
  全ての後続のデプロイについては、ベースイメージのトップに作成した変更だけがダウンロードされる。

### イメージの中は何か？

#### 概要

##### Ubuntu 14.04 LTS
割愛。

##### A correct init process
Unix process モデルによれば、 init プロセス -- PID 1 -- は全ての孤児の子プロセスを継承し、
それらを回収する。多くの Docker コンテナは、これを正しく行う init プロセスを持たず、
結果、時間とともに、これらのコンテナはゾンビプロセスで満たされるようになる。

加えて、 docker stop は init プロセスに全てのサービスを停止する目的で SIGTERM を送る。
不幸にも、多くの init systems は、それらはかわりにハードウェアシャットダウンのためにビルドされたため、
これを Docker の中で正しく行えない。
これは、プロセスを、かれらに物事を正しくデイニシャライズするチャンスを与えない SIGKILL により、
ハードキルされることを引き起こす。
これはファイル破壊を引き起こす。

Baseimage-docker には、これらのタスクの両方を正しく実行する。 init プロセス /sbin/my_init が付属する。

##### Fixes APT incompatibilities with Docker
割愛。

##### syslog-ng
カーネル自身を含む多くのサービスが正しく /var/log/syslog にログできるよう、 Syslog デーモンが必要である。
Syslog デーモンが走っていないなら、多くの重要なメッセージは静かに飲み込まれる。

ローカルにリスンするのみ。

##### logrotate
割愛。

##### ssh server
物事の検査や管理のために、コンテナに容易にログインできるようにする。
Password と Challenge-response 認証はデフォルトで無効化される。
鍵認証のみ許可される。

SSH アクセスは、そう望めば、容易に無効化できる。

##### cron
割愛。

##### runit
Ubuntu の Upstart を置き換える。サービスの監督や管理に使われる。
SysV init より使いやすく、デーモンがクラッシュしたときの再起動をサポートする。
Upstart より使いやすく、よりライトウェイトである。

##### setuser
他のユーザとしてコマンドを実行するためのツール。
su より使いやすく、sudo より小さな attack vector(攻撃者からの攻撃の手法等のことらしい) を持ち、
chpst とは異なり、このツールは $HOME を正しくセットする。
/sbin/setuser として利用できる。

#### 待って、 Docker はひとつのコンテナではシングルプロセスを走らせるものだと思ったけど？
そうでない。 Docker はひとつのコンテナでマルチプロセスとともによく走る。

Baseimage-docker は runit の使用によりマルチプロセスを後押しする。

### baseimage-docker の確認
以下を実行。

    docker.io run --rm -t -i phusion/baseimage /sbin/my_init -- bash -l

### ベースイメージとして baseimage-docker を使用する

#### Getting started
イメージは phusion/baseimage で、 Docker registry で利用できる。

    # Use phusion/baseimage as base image. To make your builds reproducible, make
    # sure you lock down to a specific version, not to `latest`!
    # See https://github.com/phusion/baseimage-docker/blob/master/Changelog.md for
    # a list of version numbers.

    # ベースイメージに phusion/baseimage を使う。
    # あなたのビルドを再現可能にするため、 latest でなく、特定のバージョンにロックすること。
    # https://github.com/phusion/baseimage-docker/blob/master/Changelog.md にバージョン一覧あり。
    FROM phusion/baseimage:<VERSION>

    # Set correct environment variables.
    ENV HOME /root

    # Regenerate SSH host keys. baseimage-docker does not contain any, so you
    # have to do that yourself. You may also comment out this instruction; the
    # init system will auto-generate one during boot.

    # SSH ホスト鍵を再生成する。 baseimage-docker はどれも含んでいないため、自身でそれをする必要がある。
    # あなたはこの手順をコメントアウトするかも。init system が boot のとき自動生成するなら。
    RUN /etc/my_init.d/00_regen_ssh_host_keys.sh

    # Use baseimage-docker's init system.
    # baseimage-docker の init system を使う。
    CMD ["/sbin/my_init"]

    # ...put your own build instructions here...

    # Clean up APT when done.
    RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

#### デーモンの追加
runit エントリを作成することでデーモンを追加する。例えばあなたのアプリなどを。
あなたは、あなたのデーモンを走らせる小さなシェルスクリプトを書く必要があるだけであり、
runit は、それをあなたのために起動し走らせ続け、それがクラッシュしたときは再起動等を行う。

シェルスクリプトは run として呼ばれなければならず、実行権が必要で、
ディレクトリ /etc/service/<NAME> に配置されなければならない。

シェルスクリプトは、daemonize/fork することなく、 daemon を実行しなければならない。
一般的に、 daemon はコマンドラインフラグか設定ファイルでそれを行うためのオプションを提供する。

#### コンテナ起動時にスクリプトを実行

baseimage-docker の init system は以下のスクリプトを以下の順番で起動時に実行する。

- /etc/my_init.d ディレクトリがあれば、それにある全ての実行権のあるスクリプトを実行する。
  スクリプトは辞書式順序に実行される。
- ファイルがあれば /etc/rc.local

全てのスクリプトは終了コード 0 で正常終了しなければならない。
スクリプトが 0 でない終了コードで終えたら、起動は失敗する。

#### コンテナでワンショットコマンドを実行する

一般的に、コンテナでシングルコマンドを実行し、コマンドの後、すぐ戻りたいとき、このように Docker を呼び出す。

    docker run YOUR_IMAGE COMMAND ARGUMENTS...

しかし、このアプローチの欠点は init system が起動されないこと。
よって、 COMMAND を呼び出す間、 cron や syslog といった重要なデーモンが起動しない。
また、 COMMAND の PID は 1 であるため、孤立した子プロセスは正しく回収されない。

Baseimage-docker は、シングルのワンショットコマンドを実行するための、その上で前述の問題を解決する機能を提供する。

    docker run YOUR_IMAGE /sbin/my_init -- COMMAND ARGUMENTS ...

これは以下を行う。
- /etc/my_init.d/* や /etc/rc.local といった、全てのシステムスタートアップファイルを実行する
- 全ての runit サービスを実行する
- 指定されたコマンドを実行する
- 指定されたコマンドが終えたら、全ての runit サービスを停止する

例。

    docker run phusion/baseimage:<VERSION> /sbin/my_init -- ls

詳細は docker run YOUR_IMAGE /sbin/my_init --help を実行のこと。

以下の例は、全ての runit サービスを走らせた上で、スタートアップファイルを実行せず、少ないメッセージで ls を実行する。

    docker run phusion/baseimage:<VERSION> /sbin/my_init --skip-startup-files --quiet -- ls

#### 環境変数
/sbin/my_init をメインコンテナコマンドとして使う場合、
docker run --env や Dockerfile での ENV コマンドでセットした環境変数が、
my_init によりピックアップされる。
これらの変数は /etc/my_init.d スタートアップスクリプトを含む全ての子プロセス、
Runit および Runit が管理するサービスにも反映される。

以下の留意点あり。

- Unix の環境変数は、プロセスごとを基礎として継承される。
  これは、子プロセスについて、他のプロセスの環境変数を変更することが
  一般的に不可能であることを意味する。
- 前述の点により、全てのアプリおよびサービスの環境変数を一元定義する場所が無い。
  Debian は /etc/environment ファイルを持つが、いくらかの条件下でしか動作しない。
- いくつかのサービスは子プロセスの環境変数を変更する。
  Nginx がいい例。Nginx は、env 設定オプションを通じ、
  それを維持することを明示的に指定しない限り全ての環境変数を削除する。
  Nginx 上で任意のアプリをホストする場合、
  （例えば passenger-docker イメージを使用する場合、または
  あなた自身のイメージで Phusion Passenger を使用する場合）
  もとは Docker によって渡された環境変数について配慮しない。

my_init は、これらの懸念事項について解決策を提供する。

##### 環境変数の一元定義
スタートアップ時に、任意のスタートアップスクリプトが走る前に、
my_init は /etc/container_environment ディレクトリから環境変数をインポートする。

このディレクトリは、環境変数の名前の後で、誰が名付けられたかのファイルを含む。（？）
ファイル内容は環境変数の値を含む。
このディレクトリは従って環境変数を一元定義するに良い場所であり、
全てのスタートアップスクリプトおよびRunitサービスにより継承される。

例えば、 Dockerfile で環境変数を定義する仕方は以下。

    RUN echo Apachai Hopachai > /etc/container_environment/MY_NAME

###### 改行の扱い
my_init は末尾の改行を取り除く。
改行のある値を指定する場合、以下のように、改行を追加する必要あり。

    RUN echo -e "Apachai Hopachai\n" > /etc/container_environment/MY_NAME

##### 環境変数のダンプ
すでに説明したメカニズムは環境変数の一元定義に良いが、
それ自身は子プロセスからの環境変数の変更やリセットからサービス(例えば Nginx)を防がない。
しかし、 my_init メカニズムはオリジナル環境変数の照会を簡単にする。

スタートアップの間、 /etc/container_environment から環境変数のインポート直後、
my_init は、自身の環境変数全て（つまり、
/etc/container_environment からインポートされる全ての変数、および
docker run --env からピックアップされる全ての変数）を
以下の場所、以下の書式でダンプする。

- /etc/container_environment
- /etc/container_environment.sh - Bash 書式での環境変数のダンプ。
  Bash シェルスクリプトからファイルを直接 source できる。
- /etc/container_environment.json - JSON 形式での環境変数のダンプ

複数の書式は、スクリプトやアプリがどの言語で記述されているかによらず、
オリジナル環境変数の照会を簡単にする。

##### 環境変数の変更
/etc/container_environment のファイルを変更することで可能。
my_init がスタートアップスクリプトを実行するそれぞれの後、
自身の環境変数を /etc/container_environment にある状態にリセットし、
新しい環境変数を /etc/container_environment.sh と
/etc/container_environment.json にダンプする。

留意点：

- container_environment.sh や container_environment.json を変更しても効果ない
- Runit サービスはこの手法のように環境を変更できない。
  my_init だけが、スタートアップスクリプト実行時に /etc/container_environment の
  変更を活性化させる。

##### セキュリティ
環境変数は、潜在的にセンシティブな情報を含み得るため、
/etc/container_environment とその Bash および JSON ダンプは、
デフォルトで root により所有され、 docker_env グループのみアクセス可能。
（このグループに追加された任意のユーザが自動的にロードされたこれらの変数を持つ。）

環境変数がセンシティブなデータを含まないことを確認したなら、
ワールドリーダブルにすることにより、
ディレクトリやこれらのファイルの権限を緩和することも可能である。

#### SSH 経由によるコンテナへのログイン
baseimage-docker がベースの任意のコンテナに対しては、ログインに SSH を使える。

コンテナの中に SSH 鍵がインストールされている必要あり。
デフォルトは SSH 鍵がインストールされていないため、ログインできない。

われわれは "a pregenerated insecure key(PuTTY format)" も提供するが、
あくまで一時的利用目的でのみ使用のこと。プロダクト環境においては、
自身の鍵を使用のこと。

##### ひとつのコンテナのみで insecure key を使用
ひとつのコンテナのみで insecure key を一時的に有効化できる。
これは、コンテナブート時に insecure key がインストールされることを意味する。

コンテナに docker stop や docker start しても、
insecure key はまだそこに存在し、
新しいコンテナをスタートするために docker run しても、
コンテナは insecure key を含まない。

コンテナを --enable-insecure-key でスタートのこと。

    docker run YOUR_IMAGE /sbin/my_init --enable-insecure-key

走らせたコンテナの ID を確認する。

    docker ps

ID を得たら、 IP アドレスを確認する。

    docker inspect -f "{{ .NetworkSettings.IPAddress }}" <ID>

以下のように SSH 。

    curl -o insecure_key -fSL https://github.com/phusion/baseimage-docker/raw/master/image/insecure_key
    chmod 600 insecure_key
    ssh -i insecure_key root@<IP address>

##### insecure key を永続的に有効化
insecure key をイメージに永続化することも可能。

一般的に推奨されないが、セキュリティが問題にならなければ便利。

    RUN /usr/sbin/enable_insecure_key

##### 自身の鍵を使用
SSH 公開鍵をインストールするため Dockerfile を編集のこと。

    ## Install an SSH of your choice.
    ADD your_key.pub /tmp/your_key.pub
    RUN cat /tmp/your_key.pub >> /root/.ssh/authorized_keys && rm -f /tmp/your_key.pub

あなたのイメージをリビルドする。それを得たらすぐ、そのイメージをもとにコンテナを起動する。

    docker run your-image-name

走らせたコンテナの ID を確認する。

    docker ps

ID を得たらすぐ、その IP アドレスをみる。

    docker inspect -f "{{ .NetworkSettings.IPAddress }}" <ID>

以下のように、コンテナに SSH のこと。

    ssh -i /path-to/your_key root@<IP address>

##### docker-bash ツール
コンテナの IP アドレスを確認し SSH を実行する手順を自動化するため、
docker-bash ツールを提供する。

このツールは Docker コンテナ内部ではなく Docker ホストで実行されるものである。

Docker ホストへのインストール。

    # 割愛

SSH でコンテナにログインする。

    docker-bash YOUR-CONTAINER-ID

YOUR-CONTAINER-ID は docker ps で調べられる。

デフォルトでは、 docker-bash は Bash セッションを開く。
コマンドを実行し戻るよう支持することもできる。

    docker-bash YOUR-CONTAINER-ID echo hello world

### あなた自身のイメージをビルドする
何らかの理由で、Docker レジストリからダウンロードする代わりに、
あなた自身のイメージをビルドしたい場合、以下の手順に従うこと。

割愛

#### SSH の無効化
SSH の有効化を望まない場合、無効化の仕方は以下。

    RUN rm -rf /etc/service/sshd /etc/my_init.d/00_regen_ssh_host_keys.sh
