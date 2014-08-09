---
layout: post
tags : [Ansible]
---
{% include JB/setup %}

## [Yosuke OTA](https://twitter.com/y0t4)

* [やること](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/60)

## プロセスどうなってるの

```
[yota@localhost ~]# docker run --name="shell" -t -i -d centos:latest /bin/bash
ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3
[yota@localhost ~]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  0.9 358948  9332 ?        Ssl  00:12   0:00 /usr/bin/docker -d --selinux-enabled
root      2602  0.8  0.1  11740  1376 pts/1    Ss+  01:21   0:00  \_ /bin/bash
~~~ SNIP ~~~
[yota@localhost ~]$ docker run -d -i centos:latest /bin/bash
b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1
[yota@localhost ~]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           4 seconds ago       Up 3 seconds                            drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           7 minutes ago       Up 7 minutes                            shell
[yota@localhost ~]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 515636 12536 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11740  1376 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      2877  0.0  0.1  11604  1104 ?        Ss   01:28   0:00  \_ /bin/bash
~~~ SNIP ~~~
```

`docker -d`はdaemon起動なオプション

docker daemonなプロセスに紐づけられたbashなプロセスが2つ存在。
起動時のオプションの違いはTTYを割り当てるかどうか。
`ps auxf`の結果を見ると明白。

### コンテナ内で子プロセス作ってみる

```
[yota@localhost ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           3 minutes ago       Up 3 minutes                            drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           10 minutes ago      Up 10 minutes                           shell
[yota@localhost ~]$ docker attach shell
bash-4.2# while sleep 1; do echo "hello"; done
hello
hello
hello #(補足: ctrl-p ctrl-qでデタッチしてます。)
[yota@localhost ~]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 515636 12536 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1712 pts/1    Ss   01:21   0:00  \_ /bin/bash
root      2944  0.0  0.0   4312   404 pts/1    S+   01:32   0:00  |   \_ sleep 1
root      2877  0.0  0.1  11604  1104 ?        Ss   01:28   0:00  \_ /bin/bash
~~~ SNIP ~~~
```

`sleep`で`echo hello`動かすとlogがうざいので何もしないように変更。
ついでにバックグラウンド起動に。

```
[yota@localhost ~]$ docker attach shell
hello
hello
^C
bash-4.2# while sleep 1; do :; done &
[1] 146
bash-4.2# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  11744  1736 ?        Ss   06:21   0:00 /bin/bash
root       146  0.5  0.0  11744   696 ?        S    06:35   0:00 /bin/bash
root       149  0.0  0.0   4312   404 ?        S    06:35   0:00 sleep 1
root       150  0.0  0.1  19748  1208 ?        R+   06:35   0:00 ps aux
```

もう一つのコンテナでも子プロセス作ってみる。

```
[yota@localhost ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           8 minutes ago       Up 8 minutes                            drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           15 minutes ago      Up 15 minutes                           shell
[yota@localhost ~]$ docker attach drunk_wilson1
while sleep 2; do :; done &
^P
/bin/bash: line 2: $'\020': command not found
^C^C #(補足: このコンテナはctrl-dで抜けた。ctrl-p ctrl-qでは抜けれない。おそらくttyがないからだと思う)
[yota@localhost ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           8 minutes ago       Up 8 minutes                            drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           16 minutes ago      Up 16 minutes                           shell
[yota@localhost ~]$ ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.1 515764 11996 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   696 pts/1    S    01:35   0:00  |   \_ /bin/bash
root      3238  0.0  0.0   4312   404 pts/1    S    01:37   0:00  |       \_ sleep 1
root      2877  0.0  0.1  11604  1300 ?        Ss   01:28   0:00  \_ /bin/bash
root      3181  0.1  0.0  11604   720 ?        S    01:36   0:00      \_ /bin/bash
root      3237  0.0  0.0   4312   400 ?        S    01:37   0:00          \_ sleep 2
~~~ SNIP ~~~
```

### /var/lib/docker見る

何かファイル書き変わってるか調査する

```
[root@localhost docker]# cd /var/lib/docker
[root@localhost docker]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           17 minutes ago      Up 17 minutes                           drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           24 minutes ago      Up 24 minutes                           shell
[root@localhost docker]# ls -l containers/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1/
total 20
-rw------- 1 root root  117 Aug  9 01:36 b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1-json.log
-rw-r--r-- 1 root root 1365 Aug  9 01:28 config.json
-rw-r--r-- 1 root root  192 Aug  9 01:28 hostconfig.json
-rw-r--r-- 1 root root   13 Aug  9 01:28 hostname
-rw-r--r-- 1 root root  174 Aug  9 01:28 hosts
[root@localhost docker]# cat containers/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1-json.log
{"log":"/bin/bash: line 2: $'\\020': command not found\n","stream":"stderr","time":"2014-08-09T05:36:56.990362511Z"}
```

コマンドの失敗とかのログは`containers/[CONTAINER ID]/[CONTAINER ID]-json.log`に置いてあるらしい。
ってことはこれも`docker`なコマンドで見ることができそう。

### docker logs

```
[yota@localhost ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           22 minutes ago      Up 22 minutes                           drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           29 minutes ago      Up 29 minutes                           shell
[yota@localhost ~]$ docker logs drunk_wilson1
/bin/bash: line 2: $'\020': command not found
```

見れた。

### コンテナ内プロセスの表示

このときわざわざcontainerにattachしなくても`docker top`コマンドでpsを見ることができることを知った。

```
[yota@localhost ~]$ docker top drunk_wilson1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                2877                857                 0                   01:28               ?                   00:00:00            /bin/bash
root                3181                2877                0                   01:36               ?                   00:00:00            /bin/bash
root                4600                3181                0                   01:51               ?                   00:00:00            sleep 2
[yota@localhost ~]$ docker top drunk_wilson1 auxf
USER                PID                 %CPU                %MEM                VSZ                 RSS                 TTY                 STAT                START               TIME                COMMAND
root                2877                0.0                 0.1                 11604               1300                ?                   Ss                  01:28               0:00                \_ /bin/bash
root                3181                0.0                 0.0                 11604               736                 ?                   S                   01:36               0:00                | \_ /bin/bash
root                4782                1.0                 0.0                 4312                400                 ?                   S                   01:53               0:00                | \_ sleep 2
```

### コンテナを殺す

`docker kill`なんてのもある。

```
[yota@localhost ~]$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           25 minutes ago      Up 25 minutes                           drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           33 minutes ago      Up 33 minutes                           shell
[yota@localhost ~]$ docker kill drunk_wilson1
drunk_wilson1
[yota@localhost ~]$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           26 minutes ago      Exited (-1) 4 seconds ago                       drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           33 minutes ago      Up 33 minutes                                   shell
```

`docker stop`が効かないようなときに無理矢理殺すためのだと思う。

```
[yota@localhost ~]$ docker kill

Usage: docker kill [OPTIONS] CONTAINER [CONTAINER...]

Kill a running container (send SIGKILL, or specified signal)

  -s, --signal="KILL"    Signal to send to the container
```

`SIGKILL`送るっぽいのでそうだろうなぁ...くらい。

### 殺しちゃったコンテナの起動

```
[yota@localhost ~]$ docker start drunk_wilson1
drunk_wilson1
[yota@localhost ~]$ docker top drunk_wilson1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                5085                857                 1                   01:56               ?                   00:00:00            /bin/bash
```

立ち上げなおしたら`sleep`なpsがなくなってた。そりゃそうかー、ってことでもう一度たちあげ。

```
[yota@localhost ~]$ docker attach drunk_wilson1
while sleep 2; do :; done &
[yota@localhost ~]$ docker top drunk_wilson1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                5085                857                 0                   01:56               ?                   00:00:00            /bin/bash
root                5247                5085                0                   01:58               ?                   00:00:00            /bin/bash
root                5266                5247                0                   01:58               ?                   00:00:00            sleep 2
```

### killコマンド使う

hostから`kill`送ってみる

```
[yota@localhost ~]$ ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12680 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   736 pts/1    S    01:35   0:01  |   \_ /bin/bash
root      5329  0.0  0.0   4312   400 pts/1    S    01:58   0:00  |       \_ sleep 1
root      5085  0.0  0.1  11604  1296 ?        Ss   01:56   0:00  \_ /bin/bash
root      5247  0.0  0.0  11604   720 ?        S    01:58   0:00      \_ /bin/bash
root      5326  1.0  0.0   4312   400 ?        S    01:58   0:00          \_ sleep 2
~~~ SNIP ~~~
[root@localhost docker]# kill 5247
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12644 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   740 pts/1    S    01:35   0:01  |   \_ /bin/bash
root      5469  1.0  0.0   4312   404 pts/1    S    02:00   0:00  |       \_ sleep 1
root      5085  0.0  0.1  11604  1296 ?        Ss   01:56   0:00  \_ /bin/bash
~~~ SNIP ~~~
```

`5247`に紐づいてた`5326`も死んだ。
もう一度起動しよう。

```
[yota@localhost ~]$ docker attach drunk_wilson1
while sleep 2; do :; done &
[yota@localhost ~]$ ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12408 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   744 pts/1    S    01:35   0:02  |   \_ /bin/bash
root      5752  0.0  0.0   4312   404 pts/1    S    02:03   0:00  |       \_ sleep 1
root      5085  0.0  0.1  11608  1296 ?        Ss   01:56   0:00  \_ /bin/bash
root      5612  0.0  0.0  11604   720 ?        S    02:02   0:00      \_ /bin/bash
root      5750  1.0  0.0   4312   404 ?        S    02:03   0:00          \_ sleep 2
~~~ SNIP ~~~
```

`sleep 2`なプロセス殺してみる。

```
[root@localhost docker]# kill 5750
-bash: kill: (5750) - No such process
```

そうだった、`sleep 2`は`ps`表示してから最大2秒以内に`kill`しないと新しいプロセスが立ち上がるんだ...。
とりあえずsleepな秒数を少し伸ばして試してみよう。

#### PIDの隔離確認

```
[root@localhost docker]# docker top drunk_wilson1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                5085                857                 0                   01:56               ?                   00:00:00            /bin/bash
root                5612                5085                0                   02:02               ?                   00:00:00            /bin/bash
root                6007                5612                0                   02:06               ?                   00:00:00            sleep 2
[root@localhost docker]# docker attach drunk_wilson1
ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   76 ?        00:00:00 bash
  213 ?        00:00:00 sleep
  214 ?        00:00:00 ps
```

ここでPIDが隔離されていることをようやく確認。

### sleepプロセスを殺す

さて`5612`なプロセスを`kill`する。それで`sleep 20`くらいで。

```
[root@localhost docker]# kill 5612
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12412 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   760 pts/1    S    01:35   0:02  |   \_ /bin/bash
root      6424  0.0  0.0   4312   404 pts/1    S    02:10   0:00  |       \_ sleep 1
root      5085  0.0  0.1  11608  1344 ?        Ss   01:56   0:00  \_ /bin/bash
~~~ SNIP ~~~
[yota@localhost ~]$ docker attach drunk_wilson1
while sleep 20; do :; done &
```

`root`になる。

```
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12448 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   760 pts/1    S    01:35   0:02  |   \_ /bin/bash
root      6547  0.0  0.0   4312   400 pts/1    S    02:12   0:00  |       \_ sleep 1
root      5085  0.0  0.1  11608  1344 ?        Ss   01:56   0:00  \_ /bin/bash
root      6501  0.0  0.0  11608   612 ?        S    02:12   0:00      \_ /bin/bash
root      6544  0.0  0.0   4312   404 ?        S    02:12   0:00          \_ sleep 20
~~~ SNIP ~~~
[root@localhost docker]# kill 6544
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12448 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   760 pts/1    S    01:35   0:02  |   \_ /bin/bash
root      6556  0.0  0.0   4312   404 pts/1    S    02:12   0:00  |       \_ sleep 1
root      5085  0.0  0.1  11608  1344 ?        Ss   01:56   0:00  \_ /bin/bash
~~~ SNIP ~~~
```

`sleep`動かしてた親な`bash`も死んだ。

### docker killとkill

多分`5085`なプロセスを`kill`するってのは`docker kill`を同じになるはず。
しかしここで間違えて`docker kill`してしまったのでもう一度立ち上げ直し。
そのせいで`PID`が変わっている。

```
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12516 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   768 pts/1    S    01:35   0:03  |   \_ /bin/bash
root      6858  0.0  0.0   4312   396 pts/1    S    02:16   0:00  |       \_ sleep 1
root      6814  0.1  0.1  11604  1096 ?        Ss   02:16   0:00  \_ /bin/bash
~~~ SNIP ~~~
[root@localhost docker]# kill 6814
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12516 ?        Ssl  00:12   0:01 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   768 pts/1    S    01:35   0:03  |   \_ /bin/bash
root      6858  0.0  0.0   4312   396 pts/1    S    02:16   0:00  |       \_ sleep 1
root      6814  0.1  0.1  11604  1096 ?        Ss   02:16   0:00  \_ /bin/bash
~~~ SNIP ~~~
```

あれ、死なない。

```
[root@localhost docker]# kill -KILL 6814
[root@localhost docker]# ps auxf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
~~~ SNIP ~~~
root       857  0.0  1.2 655032 12628 ?        Ssl  00:12   0:02 /usr/bin/docker -d --selinux-enabled
root      2602  0.0  0.1  11744  1736 pts/1    Ss+  01:21   0:00  \_ /bin/bash
root      3082  0.1  0.0  11744   772 pts/1    S    01:35   0:03      \_ /bin/bash
root      7011  0.0  0.0   4312   400 pts/1    S    02:18   0:00          \_ sleep 1
~~~ SNIP ~~~
```

死んだ。

`kill`は何も指定しないと`SIGTERM`だった。
で、`docker kill`は`SIGKILL`なのねー。

### dockerのlog

ちなみに`logs`を見てみる。

```
[yota@localhost ~]$ docker logs drunk_wilson1
/bin/bash: line 2: $'\020': command not found
/bin/bash: line 3:     6 Terminated              while sleep 2; do
    :;
done
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   76 ?        00:00:00 bash
   80 ?        00:00:00 sleep
   81 ?        00:00:00 ps
  PID TTY          TIME CMD
    1 ?        00:00:00 bash
   76 ?        00:00:00 bash
  213 ?        00:00:00 sleep
  214 ?        00:00:00 ps
/bin/bash: line 6:    76 Terminated              while sleep 2; do
    :;
done
/bin/bash: line 5:   341 Terminated              sleep 20
```

`json`も`cat`してみよう。

```
[root@localhost docker]# cat containers/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1-json.log
{"log":"/bin/bash: line 2: $'\\020': command not found\n","stream":"stderr","time":"2014-08-09T05:36:56.990362511Z"}
{"log":"/bin/bash: line 3:     6 Terminated              while sleep 2; do\n","stream":"stderr","time":"2014-08-09T06:02:14.007983848Z"}
{"log":"    :;\n","stream":"stderr","time":"2014-08-09T06:02:14.007983848Z"}
{"log":"done\n","stream":"stderr","time":"2014-08-09T06:02:14.007983848Z"}
{"log":"  PID TTY          TIME CMD\n","stream":"stdout","time":"2014-08-09T06:02:21.359330461Z"}
{"log":"    1 ?        00:00:00 bash\n","stream":"stdout","time":"2014-08-09T06:02:21.359330461Z"}
{"log":"   76 ?        00:00:00 bash\n","stream":"stdout","time":"2014-08-09T06:02:21.359330461Z"}
{"log":"   80 ?        00:00:00 sleep\n","stream":"stdout","time":"2014-08-09T06:02:21.359330461Z"}
{"log":"   81 ?        00:00:00 ps\n","stream":"stdout","time":"2014-08-09T06:02:21.359330461Z"}
{"log":"  PID TTY          TIME CMD\n","stream":"stdout","time":"2014-08-09T06:06:47.073484353Z"}
{"log":"    1 ?        00:00:00 bash\n","stream":"stdout","time":"2014-08-09T06:06:47.073484353Z"}
{"log":"   76 ?        00:00:00 bash\n","stream":"stdout","time":"2014-08-09T06:06:47.073484353Z"}
{"log":"  213 ?        00:00:00 sleep\n","stream":"stdout","time":"2014-08-09T06:06:47.073484353Z"}
{"log":"  214 ?        00:00:00 ps\n","stream":"stdout","time":"2014-08-09T06:06:47.073484353Z"}
{"log":"/bin/bash: line 6:    76 Terminated              while sleep 2; do\n","stream":"stderr","time":"2014-08-09T06:12:05.033796079Z"}
{"log":"    :;\n","stream":"stderr","time":"2014-08-09T06:12:05.033796079Z"}
{"log":"done\n","stream":"stderr","time":"2014-08-09T06:12:05.033796079Z"}
{"log":"/bin/bash: line 5:   341 Terminated              sleep 20\n","stream":"stderr","time":"2014-08-09T06:12:52.285978485Z"}
```

1行毎に`json`なオブジェクトになってるのね。

### mountポジションな場所見る

ちなみに`devicemapper/mnt`以下をのぞいてみる。

```
[root@localhost docker]# ls -l devicemapper/mnt/
total 4
drwxr-xr-x 2 root root    6 Aug  5 05:46 2758ea31b20b87d5a0c29c11414f981edf56d0557f47d91537a204d9642e5731
drwxr-xr-x 2 root root    6 Aug  5 05:46 2abd6cab7b39c5c0c12f0ee2768bb5ce9cce9bf612b38ae3296e6408a62d63da
drwxr-xr-x 2 root root    6 Aug  5 04:39 34e94e67e63a0f079d9336b3c2a52e814d138e5b3f1f614a0cfe273814ed7c0a
drwxr-xr-x 2 root root    6 Aug  5 05:46 3af9d794ad072dc96991f1ab3603d61507f81f0aefa89b81e9cf6eafd774d356
drwxr-xr-x 2 root root    6 Aug  5 05:46 3db9c44f45209632d6050b35958829c3a2aa256d81b9a7be45b362ff85c54710
drwxr-xr-x 2 root root    6 Aug  5 04:39 511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158
drwxr-xr-x 2 root root    6 Aug  5 05:46 9bad880da3d219b10423804147d6982da1a7bb1e285777a4d746afca6215bebb
drwxr-xr-x 2 root root    6 Aug  5 04:39 b157b77b1a65e87b4f49298557677048b98fed36043153dcadc28b1295920373
drwxr-xr-x 2 root root    6 Aug  9 01:28 b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1
drwxr-xr-x 2 root root    6 Aug  9 01:28 b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1-init
drwxr-xr-x 2 root root    6 Aug  5 05:46 bac448df371dc1d40fe2543d8ce3f06eb1cb8457270f5018f67ea76520a72b3b
drwxr-xr-x 4 root root 4096 Aug  5 04:39 ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3
drwxr-xr-x 2 root root    6 Aug  9 01:20 ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3-init
[root@localhost docker]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
b1a0060399a7        centos:latest       /bin/bash           56 minutes ago      Exited (-1) 5 minutes ago                       drunk_wilson1
ee6b542bf90d        centos:latest       /bin/bash           About an hour ago   Up About an hour                                shell
[root@localhost docker]# ls -l devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3
total 24
-rw-------  1 root root    64 Aug  5 04:39 id
drwx------  2 root root 16384 Aug  5 04:28 lost+found
dr-xr-xr-x 18 root root  4096 Aug  9 01:21 rootfs
[root@localhost docker]# ls -l devicemapper/mnt/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1
total 0
```

あれ、`b1a`なディレクトリの中身は何もない。
コンテナ起動してみよう。

```
[root@localhost docker]# docker start drunk_wilson1
drunk_wilson1
[root@localhost docker]# ls -l devicemapper/mnt/b1a0060399a759b6ebb6ad29e6de452240e919159c0547b50f25b1ff1b278fd1
total 24
-rw-------  1 root root    64 Aug  5 04:39 id
drwx------  2 root root 16384 Aug  5 04:28 lost+found
dr-xr-xr-x 18 root root  4096 Aug  9 02:26 rootfs
```

できた。
てことは、`docker stop`して`docker start`しなおすとまたまっさらな環境ができあがるってことだろうか...。

試してみよう。

### コンテナをstop & start

端末つけて起動している方(`--name="shell"`な方)でやる。

```
[yota@localhost ~]$ docker attach shell
bash-4.2# pwd
/
bash-4.2# cd root/
bash-4.2# ls
bash-4.2# echo hello > hogehoge.txt
bash-4.2# [yota@localhost ~]$
```

ファイルできてるか確認する。

```
[root@localhost docker]# ls -l devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3/rootfs/root/
total 4
-rw-r--r-- 1 root root 6 Aug  9 02:28 hogehoge.txt
[root@localhost docker]# cat devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3/rootfs/root/hogehoge.txt
hello
```

できてた。
じゃあ`stop`して`start`しよう。

```
[yota@localhost ~]$ docker stop shell
shell
[yota@localhost ~]$ docker start shell
shell
```

ファイルがあるか確認してみる。

```
[root@localhost docker]# ls -l devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3/rootfs/root/
total 4
-rw-r--r-- 1 root root 6 Aug  9 02:28 hogehoge.txt
[root@localhost docker]# cat devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3/rootfs/root/hogehoge.txt
hello
```

ある。
さっきの予想は間違っていたみたい。

一度コンテナを止めてみる。

```
[yota@localhost ~]$ docker stop shell
shell
```

このときは`rootfs`みたいなディレクトリはないはず。

```
[root@localhost docker]# ls -l devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3
total 0
[root@localhost docker]# ls -l devicemapper/mnt/ee6b542bf90de885e7c3b5a92b8d06d84f882943d6807ea824e676c7304bc5a3-init/
total 0
```

ない。
ここまではオッケー。

`stop`してるときの状態はどこに保存されているのか...。

[公式ドキュメント](http://docs.docker.com/terms/container/)を読んでみると、「stopしたときはメモリ状態は保存しないけどファイルシステムは保存される」って書いてた。
どこに...?

とりあえずどこかしらに保存されていて、`commit`することでそれが永続化されるようで...。

今回はここまで。
