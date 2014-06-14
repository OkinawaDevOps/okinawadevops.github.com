---
layout: post
tags : [Docker]
---
{% include JB/setup %}

## [kinjo](https://github.com/kinjo)

Dockerfile Reference を読む。

[http://docs.docker.com/reference/builder/](http://docs.docker.com/reference/builder/)

## Dockerfile Reference

手動でイメージを作成する代わりに Dockerfile を使う。

Dockerfile と呼ばれる記述ファイルをリポジトリの root に配置する。

Dockerfile にはイメージを構築するためのステップを記述する。

ソースリポジトリパスを引数に与え、 docker build を実行する。

    sudo docker build .

ビルドは CLI 側でなく Docker デーモン側が実行するため、
全てのコンテキストがデーモンに転送されなければならない。

コンテキストがデーモンに送られると
Docker CLI は ”Sending build context to Docker daemon” と通知する。

ビルド成功時に新規イメージを保存するためのリポジトリとタグを指定できる。

    sudo docker build -t shykes/myapp .

Docker デーモンは、ステップの結果をひとつずつコミットする。
最後に新しいイメージのIDを出力する。

各インストラクションは独立して実行され、そして新しいイメージが作成される。
RUN cd /tmp しても次のインストラクションには効果がない。

Docker は docker build を大幅加速するために可能な限り中間のイメージを再利用する。
（Using cache で示される。）

ビルドが終わったら Pushing a repository to its registry も参照のこと。

### Format

Dockerfile のフォーマットは以下、

    # Comment
    INSTRUCTION arguments

インストラクションは大文字小文字の区別はしない。
しかし引数と区別しやすいよう大文字で書くこと。

Docker は Dockerfile の順番でインストラクションを実行する。

最初のインストラクションは FROM であることが必須。
どのベースイメージからビルドするのかを指定するため。

### FROM

    FROM <image>

または

    FROM <image>:<tag>

FROM にはベースイメージを指定する。

FROM が Dockerfile のコメントでない最初のインストラクションであること。

複数のイメージを作成するために
ひとつの Dockerfile の中で FROM が複数登場することもある。

単に、以前にある各新規 FROM コマンドによる最後のイメージ ID の出力をメモする。（？）

tag が与えられなかった場合 latest と想定される。 tag がなかった場合、エラーが戻る。

### MAINTAINER

    MAINTAINER <name>

イメージの作者を指定する。

### RUN

ふたつの形式がある。

- `RUN <command>` （/bin/sh -c で実行されるコマンド）
- `RUN ["executable", "param1", "param2"]` （exec 形式）

RUN インストラクションは、カレントイメージの新しいレイヤで任意のコマンドを実行し、
結果をコミットする。

コミットされたイメージは次のステップに使用される。

RUN インストラクションによるレイヤリングと、コミットによる生成は、
「コミットし安く、コンテナはまるでソースコントロールのように
イメージの履歴の任意のポイントから作られる」という
Docker のコアコンセプトに準拠する。

exec 形式は shell string の破壊の回避、および
/bin/sh が含まれないベースイメージでの RUN コマンドを可能にする。

RUN インストラクションのキャッシュは次のビルドの際に、自動的に無効化されない。

RUN apt-get dist-upgrade -y のようなインストラクションのキャッシュは、
次のビルドの際に再利用される。

RUN インストラクションのキャッシュは docker build --no-cache で無効化できる。

最初に遭遇される ADD インストラクションは、
もしコンテキストのコンテンツが変わっていたら、
Dockerfile から続く全てのインストラクションのキャッシュを無効化する。
また、 RUN インストラクションのキャッシュを無効化する。（？）

### CMD

３つの形式がある。

- `CMD ["executable", "param1", "param2"]` （exec 形式。これが好ましい形式。）
- `CMD ["param1", "param2"]` （ENTRYPOINT のデフォルトパラメータとして）
- `CMD command param1 param2` （shell として）

Dockerfile においてただひとつの CMD がありうる。

ひとつよりも多く CMD を並べたら、最後の CMD が効果を得る。

CMD はコンテナの実行にデフォルトを提供する。
executable は含めることも、 ENTRYPOINT を指定する場合は省くこともできる。

shell または exec 形式を使うとき、 CMD インストラクションは
イメージを実行時に実行されるコマンドを設定する。

shell 形式の CMD を使うと `<command>` は /bin/sh -c で実行する。

`<command>` を shell なしで実行するなら、
コマンドを JSON 配列のように表現し executable にはフルパスを与えること。

    FROM ubuntu
    CMD ["/usr/bin/wc","--help"]

同じ executable を毎回実行したいなら
ENTRYPOINT と CMD との組み合わせを検討すべき。

docker run に引数を与えた場合は CMD によるデフォルトの指定は上書きされる。

RUN と CMD を混同しないこと。

RUN は実際にコマンドを実行し、結果をコミットする。
CMD はビルド時には何も実行しないが、
イメージにコマンドを提供するよう指示することを指定する。

### EXPOSE

    EXPOSE <port> [<port>...]

EXPOSE インストラクションは、
コンテナが実行時に指定されたネットワークポートをリスンすることを Docker に通知する。

Docker は、リンクを使ってのコンテナ間の内部通信のためにこの情報を使う。

### ENV

    ENV <key> <value>

ENV インストラクションは環境変数を指定する。

値は未来の RUN インストラクションに渡される。

ENV で設定された環境変数は、
結果のイメージからコンテナが実行されるときも持続する。

値は `docker inspect` で参照できる。
また `docker run --env <key>=<value>` で変更できる。

注意：
予期しない結果を引き起こすひとつの例が ENV DEBIGN_FRONTEND noninteractive の設定。
これはコンテナが対話式に実行されるときでも持続する。
例えば docker run -t -i image bash 。

### ADD

    ADD <src> <dest>

`<src>` から新しいファイルをコピーし、
コンテナファイルシステム上の `<dest>` のパスに追加する。

`<src>` は、ビルド中のソースディレクトリ（ビルドのコンテキストとも呼ばれる）と
相対的なファイル、またはディレクトリのパス、またはリモートファイルの URL であること。

`<dest>` は絶対パス。ソースは対象のコンテナ内のその位置にコピーされる

新しいファイルは uid と gid が 0 で作成される。

`<src>` がリモートファイルの URL だった場合、
デスティネーションはパーミッション 600 になる。

注意：
docker build - < somefile のように、標準入力からビルドすると、
ビルドコンテキストがないため、
Dockerfile は URL ベースの ADD ステートメントのみを含む。

注意：
もし URL ファイルが認証でプロテクトされている場合、
ADD は認証をサポートしないため、
RUN wget, RUN curl, またはコンテナ内の他のツールを使う必要がある。

コピーは以下のルールに従う。

- `<src>` に ../something のような指定はできない。
  docker build のとき、コンテキストディレクトリとそのサブディレクトリを
  docker デーモンに送信するため。

- `<src>` が URL かつ `<dest>` の最後がスラッシュで終わらないとき、
  ファイルは URL からダウンロードされ `<dest>` にコピーされる。

- `<src>` が URL かつ `<dest>` の最後がスラッシュで終わらないとき、
  filename が URL から推測され、ファイルは `<dest>/<filename>` に
  ダウンロードされる。
  例えば ADD http://example.com/foobar / はファイル /foobar を作成する。
  この場合、 適切なファイル名が発見できるよう
  URL は nontrivial path でなければならない。
  （http://example.com では動かない）

- `<src>` がディレクトリの場合、
  ファイルシステムメタデータを含むディレクトリ全体がコピーされる

- `<src>` が認識可能な圧縮フォーマットでのローカル tar アーカイブのとき、
  ディレクトリとして展開される。
  リモート URL のリソースは展開されない。
  ディレクトリがコピー、または展開されたとき、
  tar -x と同じように振る舞う。
  結果は以下のものの融合物である。
  1. 対象パスに存在していたものは何でも
  2. the contents of the source tree, with conflicts resolved in
     favor of "2." on a file-by-file basis.

- `<src>` が任意の他の種類のファイルのときメタデータと一緒に個別にコピーされる。
  この場合、もし `<dest>` の最後がスラッシュで終わるならディレクトリとしてみなされ
  `<src>` のコンテンツは `<dest>/base(<src>)` に書き込まれる。

- `<dest>` の末尾がスラッシュで終わらないとき通常ファイルとしてみなされ
  `<src>` のコンテンツは `<dest>` に書き込まれる。

- `<dest>` がないとき、自身のパスで存在していない全てのディレクトリとともに作成される。

### COPY

    COPY `<src>` `<dest>`

`<src>` からファイルをコピーし、それらをコンテナのファイルシステム上のパス `<dest>` に
追加する。

`<src>` にリモートファイルを指定できないだけで、あとは ADD と同じものだろうか・・（？）

### ENTRYPOINT

ふたつの形式がある。

- `ENTRYPOINT ["executable", "param1", "param2"]` （exec のように好ましい形式）
- `ENTRYPOINT command param1 param2` （shell として）

Dockerfile にただひとつの ENTRYPOINT がありうる。
ひとつより多く ENTRYPOINT があれば、最後のひとつだけが効果を得る。

ENTRYPOINT は、
executable として実行できるコンテナを設定できるよう手助けする。

ENTRYPOINT を指定すると、コンテナ全体があたかも executable のように走る。

ENTRYPOINT インストラクションは、 CMD の振る舞いとは異なり
docker run の引数で上書きされないエントリコマンドを追加する。

引数をエントリポイントに渡すこともできる。
例えば `docker run <image> -d` の `-d` は ENTRYPOINT に渡される。

ENTRYPOINT の JSON 配列により、
もしくは CMD ステートメントの使用によりパラメータを指定することもできる。
ENTRYPOINT におけるパラメータは docker run の引数によって上書きされない。
CMD で指定したパラメータは docker run の引数で上書きされる。

CMD のように ENTRYPOINT はプレーンストリングで指定でき /bin/sh -c で実行される。

    FROM ubuntu
    ENTRYPOINT wc -l -

イメージが常に標準入力を入力("-")として受け取り、
行数を表示する ("-l") ものだとした場合、
これをデフォルトでなくオプションとしたい場合に CMD を使う。

    FROM ubuntu
    CMD ["-l", "-"]
    ENTRYPOINT ["/usr/bin/wc"]

### VOLUME

    VOLUME ["/data"]

VOLUME インストラクションは、指定された名前でマウントポイントを作成し
ローカルホストまたは他のコンテナから外部マウントされたボリュームを
保持するものとしてマークする。

値は JSON 配列 VOLUME ["/var/log/"] またはプレーンストリングで VOLUME /var/log 。

マウントの手法は Share Directories via Volumes を参照のこと。

### USER

イメージおよび任意の RUN 命令が起動しているとき使用するユーザ名、
または UID を設定する。

### WORKDIR

RUN の作業ディレクトリを設定する。
Dockerfile のコマンド CMD や ENTRYPOINT にも適用される。

ひとつの Dockerfile で複数回使用されうる。

相対パスが提供されれば、
ひとつまえの WORKDIR　インストラクションの相対パスになる。例えば、

    WORKDIR /a WORKDIR b WORKDIR c RUN pwd

最後の pwd コマンドの出力は /a/b/c になる。

### ONBUILD

    ONBUILD [INSTRUCTION]

ONBUILD インストラクションは、
イメージが他のビルドのベースとして使われる際に実行される
"トリガー" インストラクションをイメージに追加する。

トリガーは下流ビルドのコンテキスト上で、あたかも
下流の Dockerfile における FROM インストラクションの直後に
挿入されているかのように実行される。

任意のインストラクションをトリガーとして登録できる。

他のイメージをビルドする際のベースとなるイメージをビルドするときに便利。

例えば、アプリが環境・またはユーザ固有の設定でカスタマイズされるデーモンをビルドする場合。

イメージが、例えば再利用可能な python アプリビルダーだった場合、
特定のディレクトリに追加されるためにアプリのソースコードを要求するし、
その後呼び出すビルドスクリプトを要求するかもしれない。

まだアプリのソースコードにアクセスできないため、
ADD や RUN を今は呼べないし、そしてそれは各アプリのビルドで異なるだろう。

単に、これらのアプリの中にコピー・ペーストする定型的な Dockerfile と一緒に
アプリを開発者に提供できるが、非効率だし、エラーが発生しやすいし、
アプリ固有のコードと混ざるためアップデートが困難になる。

ソリューションは ONBUILD を使用し、
次のビルドステージの際に実行するインストラクションを予め登録すること。

1. ONBUILD に遭遇したとき、ビルダーがトリガーを
   ビルドされているイメージのメタデータに追加する。

2. ビルドの最後、全てのトリガーが OnBuild キー配下にあるイメージマニフェストに保存される。
   インストラクションは現在のビルドには影響しない。

3. あとで、FROM を使用し新しいビルドのベースとして使われるだろう。
   FROM インストラクションの一部のように、
   下流ビルダーは ONBUILD トリガーをみて、それが登録されている順に実行する。
   トリガーのいずれかが失敗したら FROM インストラクションはアボートし、
   ビルドは失敗する。全てのトリガーが成功したら、
   FROM インストラクションは成功し、
   通常のようにビルドが続く。

4. トリガーは実行された後、最終のイメージからクリアされる。
   孫のビルドには継承されない。

例えば以下のように、

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

- 警告： ONBUILD ONBUILD のようなチェーンは許されていない
- 警告： ONBUILD は FROM または MAINTAINER インストラクションはトリガーしない
