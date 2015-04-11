---
layout: post
tags : [Docker, Ubuntu 14.04, ]
---
{% include JB/setup %}

# [Takumi KINJO](https://twitter.com/libkinjo)



## やること


-   AMQP で実現されうるアーキテクチャを確認する（AMQP 実装は RabbitMQ を使用）
-   AMQP を使う側のアプリが動いているときのキュー構造を確認する

## 成果


-   アーキテクチャはチュートリアル <http://www.rabbitmq.com/getstarted.html> を見るとよい模様
-   キュー構造は rabbitmqctl list\_queues/list\_exchanges/list\_bindings コマンドで確認できる（実行に要 root）
-   本件に使用した環境（用法は README 参照）
    -   <https://github.com/kinjo/my-ansible-project/tree/rabbitmq>
    -   環境の仕様
        -   Rabbit 導入済
        -   rabbitmq\_management プラグイン （ウェブ UI）導入済
        -   ウェブ UI ログイン用ユーザ作成（デフォルトユーザ guest/guest ではログイン不可）
            -   Username: management
            -   Password: management
        -   ウェブ UI へのアクセスは <http://localhost:15672/>

## 実施内容

以下のチュートリアルを確認する。確認しながら途中途中で実験してみる。

<http://www.rabbitmq.com/getstarted.html>

サンプルコードは以下。

<https://github.com/rabbitmq/rabbitmq-tutorials/tree/master/python>

用語メモ


-   RabbitMQ
    -   標準的な AMQP 実装
    -   アプリ間のメッセージを仲介することが主な役割（メッセージブローカー）
    -   AMQP - アプリ間のメッセージを仲介する手法（キューイング・ルーティング）をまとめたプロトコル
-   Rabbit を使う側のアプリ
    -   `Producer` : メッセージを送るアプリ
    -   `Consumer` : メッセージを受けるアプリ
-   Rabbit の中の要素
    -   `Exchange` : メッセージを最初に受ける（その後、設定された配送方式に従いメッセージをキューに配送）
    -   `Queue` : メッセージを溜めるキュー
    -   `Binding` : `Exchange` とキューの紐付け情報
        -   `Binding Key` を指定できる（キューに配送する際の判定に使用）

    -メッセージ: Rabbit を介してやりとりされるメッセージ

    -   `Routing Key` を指定できる（キューに配送する際の判定に使用）

### [1 "Hello World!"](http://www.rabbitmq.com/tutorials/tutorial-one-python.html)

キューを定義してそのキューでメッセージ "Hello World!" を送受信する。

ひとつのキューを `Producer` と `Consumer` から使うためキューに名前 "hello" をつけている。

初級のため `Exchange` については割愛とのこと。

この節では `Producer` のサンプルとして send.py を `Consumer` のサンプルとして receive.py を作成する。

1.  実験 : キューにメッセージが溜まっている様子を確認してみる

    まず send.py を何回か実行しメッセージをキューに溜める。

        python send.py

    `rabbitmqctl list_queues` でキューの状態を確認する。

    `rabbitmqctl list_queues` で確認する項目は以下。（他にもある）

    -   `name` : キューの名前
    -   `messages_ready` : クライアントへの配信がレディ状態のメッセージ数
    -   `messages_unacknowledged` : クライアントに配送済みだが、まだ Ack されていないメッセージ数
    -   `consumers` : Consumer 数
    -   `state` : キューの状態（通常は running だが、キューが同期している場合は syncing, MsgCount かもしれない）

    以下のとおり実行。

        sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged consumers state | sed 's/\t/ | /g'

    以下のとおり出力。

        Listing queues ...
        hello | 4 | 0 | 0 | running


    -   messages\_ready=4 : メッセージが 4 つ溜まっている
    -   messages\_unacknowledged=0 : アックが戻ったメッセージなし
    -   consumers=0 : `Consumer` が接続していない
    -   status=running : 稼働中のキュー

    次に receive.py を実行しメッセージを取得する。

        python receive.py

    キューの状態を確認。

        sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged consumers state | sed 's/\t/ | /g'

    以下が出力。

        Listing queues ...
        hello | 0 | 0 | 1 | running

    キューから全てのメッセージが取得され messages\_ready は 0 となり,
    receive.py を起動しているため consumers は 1 となる。

2.  実験 : コネクションの状況を確認する

    receive.py を起動している間、キューに `Consumer` が接続している状態である。
    `Consumer` がキューに接続している様子を確認してみる。

    `rabbitmqctl list_connections` で確認する。


    -   `pid` : コネクションに関連付けられた Erlang プロセス
    -   `name` : コネクションのリーダブルな名前
    -   `state` : コネクションを使用しているチャネルの数
    -   `user` : コネクションに関連付けられているユーザ名
    -   `send_cnt` : 送信されたパケット数

    以下のとおり実行。

        sudo rabbitmqctl list_connections pid name state user send_cnt | sed 's/\t/ | /g'

    以下のとおり出力。

        Listing connections ...
        <rabbit@vagrant-ubuntu-utopic-64.1.9807.0> | 127.0.0.1:47824 -> 127.0.0.1:5672 | running | guest | 9

    state は running, user は guest, send\_cnt は 9 となっている。
    send\_cnt は send.py を実行するたびに増えていく模様。

3.  実験 : キューを削除するには

    ちなみに receive.py の実行を停止してもキュー hello は残り続ける。
    これは rabbitmq-server を再起動すると削除される。

    自動的にキューが削除されるようにするには以下のとおり。（詳しくはスルー）


    -   キュー作成時、 queue arguments に x-expires を指定することで一定時間経過後に自動削除するよう設定できる模様
        <http://ivanyu.me/blog/2015/02/16/delayed-message-delivery-in-rabbitmq/>

    -   RabbitMQ のサーバサイドでも Queue TTL を設定できる模様
        <https://www.rabbitmq.com/ttl.html>

### [2 Work queues](http://www.rabbitmq.com/tutorials/tutorial-two-python.html)

ひとつのキューに複数の `Consumer` を接続しメッセージを分配する。
メッセージはいずれかの `Consumer` （ワーカー）が受け取る。（同時には受け取らない）

メッセージを受けたワーカーはその内容に従い、本来ならば何らかのタスクを実行する。

メッセージは round-robin で配送される。例えば、ワーカーが 2 つある場合はメッセージは交互に配送される。

メッセージがワーカーに配送された後、メッセージはキューから削除される。
よって、ワーカーがエラーなどでタスクを処理できなかった場合、メッセージは消失してしまうこととなる。

Ack を使用すると Ack が受信されるまでメッセージは削除されない。
ワーカーが死んでも、他の生きているワーカーがメッセージを引き継げるようになる。

タスクが偏らないよう配送制限（まだ Ack を戻していないワーカーには新たにメッセージを配信しないなど）も可能。

この節では `Producer` のサンプルとして new\_task.py を、ワーカーのサンプルとして worker.py を作成する。

1.  実験 : コネクションの状況を確認する

    worker.py を 2 つ起動し rabbitmqctl list\_connections でコネクションを確認する。

        sudo rabbitmqctl list_connections name state user send_cnt | sed 's/\t/ | /g'

    以下が出力。

        Listing connections ...
        127.0.0.1:47837 -> 127.0.0.1:5672 | running | guest | 7
        127.0.0.1:47838 -> 127.0.0.1:5672 | running | guest | 7

    Consumer 側である worker.py が 2 つ起動しているためコネクションは 2 つ出力される。
    送信パケット数がすでに 7 となっており、初期に何らかのパケットがやりとりされた模様。

2.  実験 : キューの状況を確認する

    rabbitmqctl list\_queues でキューを確認する。

        sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged consumers state | sed 's/\t/ | /g'

    以下が出力。

        ........
        task_queue | 0 | 0 | 2 | running

    キュー task\_queue が作成されていることを確認できる。

3.  実験 : 永続キューを削除するには

    worker.py 停止後、キュー task\_queue が残ったままとなる。キュー作成時に durable=True を指定しているため
    rabbitmq-server を再起動してもキューは残り続ける。

    durable=True なキューを削除するにはどうすればいいか。

    `rabbitmqctl reset` は永続化している情報を削除するが、ユーザも削除される。（運用に相応しくない）

    rabbitmq\_management プラグインを導入すると管理用ウェブ API が提供される。

    以下の URL リクエストでキューを削除できる。（環境に rabbitmq\_management は導入済）

        curl -i -u guest:guest -H "content-type:application/json" -XDELETE http://localhost:15672/api/queues/%2f/task_queue

    また rabbitmq\_management プラグインは管理用コマンド rabbitmq\_management を提供する。

    rabbitmq\_management でもキューを削除できる。

        python /var/lib/rabbitmq/mnesia/rabbit@vagrant-ubuntu-utopic-64-plugins-expand/rabbitmq_management-3.5.0/priv/www/cli/rabbitmqadmin delete queue name=task_queue

    rabbitmq\_management はウェブ UI(<http://localhost:15672/>) も提供する。（環境ではログイン用アカウント management/management を登録済み）

    <http://localhost:15672/#/queues/%2F/task_queue> の Delete / purge メニューでキューを削除できる。

### [3 Publish/Subscribe](http://www.rabbitmq.com/tutorials/tutorial-three-python.html)

`Exchange` について。

メッセージは、実際にはキューに直接入るのではなく `Exchange` が最初に受け、適切なキューに配送される。
`Exchange` の配送方式（メッセージを特定のキューに配送、複数のキューに配送、または破棄）は `Exchangeタイプ` で指定する。

以下 `Exchangeタイプ`

-   Direct
    `Routing Key` と `Binding Key` が一致した場合に限りメッセージをキューに送信
-   Topic
    `Routing Key` と `Binding Key` がマッチした場合に限りメッセージをキューに送信
-   Headers
    要調査
-   Fanout
    全てのキューに送信

`Exchange` をキューにバインディングすることで `Exchange` はキューにメッセージを配送するようになる。

Fanout を使い syslog と logger コマンドを模したスクリプトを作成する。

これまでは無名の `Exchange` を使用していたため、メッセージ送信時に routing\_key でキューの名前を指定する必要があった。
今回は名前付き `Exchange` を使用するためメッセージ送信時に `Exchange` 名を指定できる。

なおキューにはランダムな名前が使用される模様。

この節では syslog サーバ側のサンプルとして `receive_logs.py` を、
logger コマンド側のサンプルとして `emit_log.py` を作成する。

1.  実験 : デフォルトの `Exchange` の状況を確認する

    最初に `rabbitmq list_exchanges` を実行しデフォルトの `Exchange` の一覧を確認する。

        sudo rabbitmqctl list_exchanges name type durable auto_delete | sed 's/\t/ | /g'

    `rabbitmq list_exchanges` で確認する項目は以下。


    -   `name` : `Exchange` 名
    -   `type` : direct, topic, headers, fanout のいずれか
    -   `durable` : サーバ再起動を生き残った `Exchange` かどうか
    -   `auto_delete` : 未使用となったら自動削除されるかどうか
    -   `internal` : インターナルかどうか
    -   `arguments` : `Exchange` 引数
    -   `policy` : `Exchange` に適用されているポリシー名

    以下が出力。

        Listing exchanges ...
         | direct | true | false
         amq.direct | direct | true | false
         amq.fanout | fanout | true | false
         amq.headers | headers | true | false
         amq.match | headers | true | false
         amq.rabbitmq.log | topic | true | false
         amq.rabbitmq.trace | topic | true | false
         amq.topic | topic | true | false

    デフォルトの `Exchange` が出力されている。これらを強いて使う必要はない模様。

2.  実験 : キューの状況を確認する

    `receive_logs.py` を起動し `rabbitmqctl list_connections` でキューを確認する。

        sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged consumers state | sed 's/\t/ | /g'

    以下が出力。

        Listing queues ...
        amq.gen-jX-1mx2UbPUFH13aigyTJA | 0 | 0 | 1 | running

    ランダムな名前のキューが作成されている。

3.  実験 : 再度 `Exchange` の状況を確認する

    `receive_logs.py` を起動した後、再度 `rabbitmq list_exchanges` で `Exchange` の一覧を確認する。

        sudo rabbitmqctl list_exchanges name type durable auto_delete | sed 's/\t/ | /g'

    以下が出力。

        Listing exchanges ...
         | direct | true | false
        amq.direct | direct | true | false
        amq.fanout | fanout | true | false
        amq.headers | headers | true | false
        amq.match | headers | true | false
        amq.rabbitmq.log | topic | true | false
        amq.rabbitmq.trace | topic | true | false
        amq.topic | topic | true | false
        logs | fanout | false | false

    logs が追加されていることを確認できる。 type は fanout 。

    ちなみに receive\_logs.py を停止しても logs は削除されない。
    Rabbit を再起動すると削除される。

4.  実験 : `Binding` の状況を確認する

    `receive_logs.py` を起動した後 `rabbitmqctl list_bindings` で `Binding` の一覧を確認する。

        sudo rabbitmqctl list_bindings source_name source_kind destination_name destination_kind routing_key | sed 's/\t/ | /g'

    `rabbitmqctl list_bindings` で確認する項目は以下。


    -   `source_name` : `Binding` が接続しているソース
    -   `source_kind` : `Binding` が接続しているソースの種別（常に exchange となる）
    -   `destination_name` : `Binding` が接続している宛先
    -   `destination_kind` : `Binding` が接続している宛先の種別
    -   `routing_key` : `Binding` の `Binding Key`

    以下が出力。

        Listing bindings ...
         | exchange | amq.gen-jX-1mx2UbPUFH13aigyTJA | queue | amq.gen-jX-1mx2UbPUFH13aigyTJA
        logs | exchange | amq.gen-jX-1mx2UbPUFH13aigyTJA | queue | amq.gen-jX-1mx2UbPUFH13aigyTJA

5.  実験 : キューまわりの状況を図示してみる

    `Binding` の状況がわかるとキューまわりの状況を以下のように図示できる。

        Exchanges     Bindings                     Queues                          Consumers
        +-----------+                              +--------------------+          +---------------+
        |logs       | key=amq.gen-jX-1mx2UbP..     |amq.gen-jX-1mx2UbP..|          |receive_logs.py|
        |type=fanout|------------------------------|                    |----------|               |
        +-----------+                              +--------------------+          +---------------+

### [4 Routing](http://www.rabbitmq.com/tutorials/tutorial-four-python.html)

`Binding` について。

`Binding` 定義時にキー（ `Binding Key` ）を設定できる。キーの意味は `Exchangeタイプ` で異なってくる。 Fanout では無視される。

syslog と logger を模してログメッセージを配送するスクリプトを作成したが、ログの緊急度で出力先を分けられるようにしたい。

Fanout `Exchange` ではブロードキャストしかできないため Direct `Exchange` を使用する。

Direct `Exchange` では `Routing Key` と `Binding Key` とが一致した場合に限りメッセージをキューに配送する。

一致しなかった場合メッセージは破棄される。

複数の `Binding` に同一の `Binding Key` を設定することも可。

この節では syslog サーバ側のサンプルとして `emit_log_direct.py` を、
logger コマンド側のサンプルとして `receive_logs_direct.py` を作成する。

### [5 Topics](http://www.rabbitmq.com/tutorials/tutorial-five-python.html)

Direct `Exchange` と似た `Exchangeタイプ` で `Binding Key` でワイルドカードの指定が可能。

`Routing Key` には複数のワードをドットで連結した文字列（255 bytes まで）を指定できる。

`Binding Key` にも同様に複数のワードをドットで連結した文字列を指定できる。

ワイルドカード

-   `*` - ひとつのワード
-   `#` - 0 個以上のワード

この節では syslog サーバ側のサンプルとして `emit_log_topic.py` を、
logger コマンド側のサンプルとして `receive_logs_topic.py` を作成する。

### [6 RPC](http://www.rabbitmq.com/tutorials/tutorial-six-python.html)

`Producer` 側を Client, `Consumer` 側を Server として RPC を実現する。

Client はリクエストメッセージを送信し、サーバはレスポンスメッセージをリプライする。

Server レスポンス用のキューをどう用意するか。

ひとつは、Client がリクエストを送信する際に一緒にレスポンス受信用のコールバックキューをサーバに伝える方法がある。
この方法だとリクエストごとにキューを作成する必要があるため非効率。

代わりに、クライアントごとにひとつのコールバックキューを作成する。
この方法ではレスポンスがどのリクエストのものか判別しなければならない。
判別のためメッセージに correlation\_id プロパティを設定する。
コールバックキューから取得したメッセージの correlation\_id を確認することで、リクエストとレスポンスを対応できるようになる。
correlation\_id が一致しなかったメッセージは破棄すべき。

この節では RPC サーバ側のサンプルとして `rpc_server.py` を、
RPC 呼び出し側のサンプルとして `rpc_client.py` を作成する。
