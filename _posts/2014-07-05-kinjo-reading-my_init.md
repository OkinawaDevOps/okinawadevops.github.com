---
layout: post
tags : [Docker]
---
{% include JB/setup %}

# [kinjo](https://github.com/kinjo)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/47)

# baseimage-docker なイメージをビルドして使用
イメージ my-baseimage-docker をビルドして ssh できることを確認する。

環境は Vagrant(VirtualBox) 。詳しくは以下。

[https://gist.github.com/kinjo/992e0622f538d8794faa](https://gist.github.com/kinjo/992e0622f538d8794faa)

事前に鍵を作成。手法は以下。

    ssh-keygen -t rsa -N ''

手順に従い、まず baseimage-docker のバージョンを確認。

[https://github.com/phusion/baseimage-docker/blob/master/Changelog.md](https://github.com/phusion/baseimage-docker/blob/master/Changelog.md)

最新は、 2014/7/5 時点で 0.9.11 (release date: 2014-06-24) 。

次に、 Dockerfile を作成。公開鍵の仕込みも行う。

    FROM phusion/baseimage:0.9.11
    ENV HOME /root
    RUN /etc/my_init.d/00_regen_ssh_host_keys.sh
    CMD ["/sbin/my_init"]
    Add id_rsa.pub /tmp/id_rsa.pub
    RUN cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys && rm -f /tmp/id_rsa.pub

my-baseimage-docker イメージをビルドする。

    docker.io build -t my-baseimage-docker .

my-baseimage-docker イメージからコンテナを起動する。

    docker.io run --rm -t -i my-baseimage-docker

以下な出力あり。

    *** Running /etc/my_init.d/00_regen_ssh_host_keys.sh...
    *** Running /etc/rc.local...
    *** Booting runit daemon...
    *** Runit started as PID 10

コンテナ ID を確認する。

    docker.io ps

IP アドレスを確認する。（コンテナ ID が 89e4a6a70b20 だったとして）

    docker.io inspect -f "{{ .NetworkSettings.IPAddress }}" 89e4a6a70b20

コンテナに SSH する。（IP アドレスが 172.17.0.2 だったとして）

    ssh -i ~/.ssh/id_rsa root@172.17.0.2

認証されるまでが遅い。これは別立てで調べたほうが良さげだが、今はスルー。

最後に ps ajxf でプロセス構造を確認。

     PPID   PID  PGID   SID TTY      TPGID STAT   UID   TIME COMMAND
        0     1     1     1 ?            1 Ss+      0   0:00 /usr/bin/python3 -u /sbin/my_init
        1    10     1     1 ?            1 S+       0   0:00 /usr/bin/runsvdir -P /etc/service
       10    11    11    11 ?           -1 Ss       0   0:00  \_ runsv syslog-ng
       11    16    11    11 ?           -1 S        0   0:00  |   \_ syslog-ng -F -p /var/run/syslog-ng.pid --no-caps
       10    12    12    12 ?           -1 Ss       0   0:00  \_ runsv cron
       12    15    12    12 ?           -1 S        0   0:00  |   \_ /usr/sbin/cron -f
       10    13    13    13 ?           -1 Ss       0   0:00  \_ runsv sshd
       13    14    13    13 ?           -1 S        0   0:00      \_ /usr/sbin/sshd -D
       14    20    20    20 ?           -1 Ss       0   0:00          \_ sshd: root@pts/0
       20    22    22    22 pts/0       39 Ss       0   0:00              \_ -bash
       22    39    39    22 pts/0       39 R+       0   0:00                  \_ ps ajxf

# my_init 内容確認

簡単に読んだ結果、 my_init はプロセスグループ配下のプロセスを回収したり、終了させたりといった、管理機能を備えている Python スクリプト。

詳細は以下。

- def error(message):
  割愛。

- def warn(message):
  割愛。

- def info(message):
  割愛。

- def debug(message):
  割愛。

- def ignore_signals_and_raise_keyboard_interrupt(signame):
  SIGTERM 、 SIGINT に signal.SIG_IGN ハンドラをセット（シグナルを無視する）。
  代わりに signame を引数に KeyboardInterrupt 例外を起こす。

- def raise_alarm_exception():
  文字列 'Alarm' を引数に AlarmException 例外を起こす。

- def listdir(path):
  path がディレクトリの場合、内容のリストを配列で戻す。それ以外は空の配列を戻す。

- def is_exe(path):
  path が実行ファイルの場合は True 、それ以外は False を戻す。

- def import_envvars(clear_existing_environment = True, override_existing_environment = True):
  /etc/container_environment にあるファイルをひとつずつ開き os.environ にて環境変数を登録する。
  clear_existing_environment が True なら、一旦、環境変数を全てクリアしたあと、環境変数の登録を行う。
  環境変数の登録は、 override_existing_environment が True 、または環境変数が存在していない場合に行われる。

- def export_envvars(to_dir = True):
  環境変数を全取得し、取得した環境変数ひとつひとつから、シェル向けのダンプ文字列を組み立てる。
  ひとつひとつ処理するとき to_dir が True の場合、 /etc/container_environment 配下に環境変数ファイルを作成する。
  また、ひとつひとつ処理するとき、 HOME, USER, GROUP, UID, GID, SHELL はスキップする。
  組み立てたダンプ文字列は /etc/container_environment.sh に書き出す。
  ただし /etc/container_environment.json には、取得した環境変数全てダンプする模様。

- def shquote(s):
  docstring には、引数 s についてシェルエスケープした版を戻す、とある。
  not s が True の場合、文字列 '' を、 s が期待した範囲の文字種から外れていた場合はそのまま s を戻す。
  それ以外は s をエスケープ処置して戻す。

- def waitpid_reap_other_children(pid):
  コメントには、終了した任意の他の子プロセスを享受しつつ（例えば、終了した養子プロセス）、
  与えられた PID で子プロセスを待つ、とある。
  処理を読むと、最初に、 terminated_child_processes 変数の中を pid でサーチする。
  terminated_child_processes には、以前終了したプロセスの状態が、 pid をキーにして保存されている模様。
  pid に該当するプロセスの状態が見つかれば、それを戻す。
  pid に該当するものが terminated_child_processes になければ os.waitpid に入る。
  os.waitpid は、 PID=1 のプロセスグループについてプロセスの終了を待つ。
  終了したプロセスの番号が、引数 pid に一致すればそのプロセスの状態を戻す。
  終了したプロセスの番号が、 pid と異なれば再度 os.waitpid に入る。
  （プロセスグループ配下に pid に一致するプロセスがなかった場合、無限ループに入るのでは・・）

- def stop_child_process(name, pid, signo = signal.SIGTERM, time_limit = KILL_PROCESS_TIMEOUT):
  pid のプロセスにシグナル signo (デフォルト SIGTERM)を送り、 waitpid_reap_other_children で終了を待つ。
  KILL_PROCESS_TIMEOUT（5秒間）でタイムアウトした場合、
  pid のプロセスに SIGKILL を送り、 waitpid_reap_other_children で終了を待つ。
  （pid に一致するプロセスが存在しなければ、無限ループに入るのでは・・）

- def run_command_killable(*argv):
  argv[0] を実行ファイルとして os.spawnvp に入る。
  次に、 os.spawnvp が戻す pid で waitpid_reap_other_children に入り、 pid のプロセスの終了を待つ。
  （Python 力はあまりないが、 os.spawnvp は fork するようにはコードからは見えない。
  run_command_killable は、実行が終了するまで処理をブロックするのだろうか・・）

- def run_command_killable_and_import_envvars(*argv):
  run_command_killable, import_envvars, export_envvars を順に実行する。

- def kill_all_processes(time_limit):
  PID=1 のプロセスグループに SIGTERM を送る。
  time_limit で指定された秒間だけプロセスグループを os.waitpid する。
  os.waitpid の待ちがタイムアウトした場合、今度はプロセスグループに SIGKILL を送る。

- def run_startup_files():
  /etc/my_init.d 配下のファイルひとつずつに対し、実行可能ファイルであれば
  run_command_killable_and_import_envvars に入っていく。

- def start_runit():
  引数 /usr/bin/runsvdir -P /etc/service で os.spawnl に入る。
  os.spawnl から戻った pid を戻り値として戻す。

- def wait_for_runit_or_interrupt(pid):
  引数 pid で wait_for_runit_or_interrupt に入る。
  wait_for_runit_or_interrupt から戻れば、 True と wait_for_runit_or_interrupt の戻り値を戻す。
  KeyboardInterrupt 例外の場合は False と None を戻す。

- def shutdown_runit_services():
  引数 /usr/bin/sv down /etc/service/* で os.system に入る。
  その後、すぐに戻る。

- def wait_for_runit_services():
  引数 /usr/bin/sv status /etc/service/* | grep -q '^run:' で os.system に入り、ループする。
  os.system が 0 以外を戻せば、ループから抜けてすぐに戻る。
  os.system が 0 を戻せば 0.1 秒間待った後、再びループに入る。

- def install_insecure_key():
  引数 /usr/sbin/enable_insecure_key で run_command_killable に入る。
  その後、すぐに戻る。

- def main(args):
  まず import_envvars, export_envvars を呼ぶ。
  args で enable_insecure_key が真なら install_insecure_key を呼ぶ。
  args で skip_startup_files が真でなければ run_startup_files を呼ぶ。
  args で skip_runit が真でなければ start_runit を呼ぶ。
  args で main_command が与えられなかった場合、 runit プロセスの pid で wait_for_runit_or_interrupt に入る。
  args で main_command が与えられた場合、引数 main_command で os.spawnvp に入り、 os.spawnvp から戻った pid で
  waitpid_reap_other_children に入る。
  KeyboardInterrupt または BaseException 例外の場合、 stop_child_process に入る。

- その他
  引数のパースを行う。その後 SIGTERM, SIGINT, SIGALRM を設定しているが、ここの記述は解らない。
