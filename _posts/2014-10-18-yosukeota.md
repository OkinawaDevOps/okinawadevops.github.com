---
layout: post
tags : [Linux Kernel]
---
{% include JB/setup %}

### [Yosuke OTA](https://twitter.com/y0t4)

* [やること](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/84)

# "kernelをgdbで追いかける"なログ

KVMを利用してkernelをgdbで追いかけてみよう

## HOST側でkernel make

### vmlinux

* linux-3.9.2

~~~
sudo yum install ncurses-devel
make menuconfig
    (debug option, KGDBをつける)
make -j

sudo yum install zlib-devel
sudo yum install glib2-devel
sudo yum groupinstall "Development Tools"
sudo ./configure <- sudoつけると通る。つけないと通らない。rootでも同じ。

yum --enablerepo=epel search qemu
sudo yum install libvirt
sudo yum install libpciaccess-devel  <- いらなかった？
sudo yum install pciutils-devel

sudo yum groupinstall Virtualization "Virtualization Platform"

wget http://pkgs.repoforge.org/qemu/qemu-img-0.15.0-1.el6.rfx.x86_64.rpm
sudo rpm -ivh qemu-img-0.15.0-1.el6.rfx.x86_64.rpm

qemu-kvm コマンドが qemu-system-x86_64 とかと同じらしい。
qemu-kvm は /usr/libexec/ にあるので、PATHを-devel
sudo yum install glib2-devel
sudo yum groupinstall "Development Tools"
sudo ./configure <- sudoつけると通る
~~~

## VM設定なxml編集

* `<DOMAIN>`エレメント内に記述を追加

  ~~~
  $ virsh edit softsystem_2nd
  <qemu:commandline>
    <qemu:arg value='-s'/>
  </qemu:commandline>
  ~~~

これを記述することで今後は
`/usr/libexec/qemu-kvm -s /tmp/Centos_2nd.img`
とやらずに
`virsh start softsystem_2nd`
で同等となる


## HOST側でKVM環境準備

### KVM のインストール

~~~
sudo yum groupinstall Virtualization "Virtualization Client" "Virtualization Platform" "Virtualization Tools"
chkconfig libvirtd on
reboot
~~~

* ファイルの編集
  * /etc/sysconfig/network-scripts/ifcfg-br0
  * /etc/sysconfig/network-scripts/ifcfg-eth0

~~~
modprobe kvm_intel
qemu-img create -f qcow2 [img名] 10G
virt-install --connect qemu:///system --name [VM名] --vcpus [cpuコア数] --ram=[メモリ(MB)] --hvm --location [DVDのイメージ]  --os-type=Linux --os-variant=virtio26 --disk path=[qemu-imgで作ったimg],size=[diskのサイズ(GB)],format=qcow2  --network bridge=br0 --accelerate --extra-args='console=tty0 console=ttyS0,115200n8'
~~~


## VM側準備

### VM内設定 (minimal版CentOS)

* selinuxをdisabled
* network設定
* resolv.conf
* yum
* yum install openssh-clients


### VM内kernel build

~~~
/usr/local/src/linux-3.9.2.tar.xz
yum install ncurses-devel
yum install xz
tar Jxvf linux-3.9.2.tar.xz
  (.configファイルはvmlinux作成時のモノをscpする。)
yum groupinstall "Development Tools"
rpm -Va --nofiles --nodigest <- 普通のgroupremoveだとエラー出て試したけど効果なし
yum groupremove --setopt=groupremove_leaf_only=TRUE "Development Tools"
yum install make gcc
time make -j 8 <-失敗
yum install bc

yum install perl
time make modules_install
~~~

#### ハマったところ

* diskサイズが足りなくて怒られる。

  * イメージの容量を増やす

    ~~~
    qemu-img resize Centos_2nd.img +4G
    qemu-img info Centos_2nd.img
    ~~~

  * swap容量が大きかったので減らして、root領域に割り当てる

    ~~~
    lvresize -L -7G /dev/mapper/VolGroup-lv_swap
    lvextend -L +7G /dev/mapper/VolGroup-lv_root
    resize2fs /dev/mapper/VolGroup-lv_root
    ~~~

* kernel buildに戻る。

  error出て止まる

  ~~~
  real     1m19.214s
  user     8m37.024s
  sys     1m4.322s
  ~~~

  `yum install perl`して
  もっかい make

  ~~~
  real     5m48.625s
  user     37m21.291s
  sys     4m59.829s
  ~~~

* やっとmakeが通る
  ~~~
  time make modules_install -j 8

  real     0m21.190s
  user     1m20.899s
  sys     0m21.629s

  time make install -j 8
  real     0m11.456s
  user     0m6.187s
  sys     0m3.844s
  ~~~

### VM内boot関係設定

* grub.conf の設定をする。
  * kgdboc の設定を忘れないようにする。
    * ttyS
    * default

  ~~~
  default=0
  timeout=5
  serial --unit=0 --speed=115200
  terminal --timeout=5 serial console
  title CentOS (3.9.2)
          root (hd0,0)
          kernel /vmlinuz-3.9.2 ro root=/dev/mapper/VolGroup-lv_root rd_NO_LUKS LAA
  NG=en_US.UTF-8 rd_NO_MD rd_LVM_LV=VolGroup/lv_swap SYSFONT=latarcyrheb-sun16 craa
  shkernel=auto console=ttyS0,115200 kgdboc=ttyS0,115200 rd_LVM_LV=VolGroup/lv_rooo
  t  KEYBOARDTYPE=pc KEYTABLE=us rd_NO_DM
          initrd /initramfs-3.9.2.img
  ~~~

### 起動後にkernel version確認

~~~
[root@localhost ~]# uname -r
3.9.2
~~~

# HOST側

## gdb を使ってみる

~~~
/usr/libexec/qemu-kvm -s -S /tmp/Centos_2nd.img
gdb vmlinux
(gdb) set arch i386 <- これやると微妙に見える
(gdb) target remote :1234

/usr/libexec/qemu-kvm -s /tmp/Centos_2nd.img <- -Sはつけない
gdb vmlinux
(gdb) target remote :1234
~~~

`nm vmlinux`くらいでbreakpointを探す。
do_forkとか。



# linux-3.9.2をdebugger追っかけログ

## breakpointsなログ

~~~
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0xffffffff814e42a0 in tcp_init_sock at net/ipv4/tcp.c:373
2       breakpoint     keep y   0xffffffff8150b060 in inet_ioctl at net/ipv4/af_inet.c:875
3       breakpoint     keep y   0xffffffff81494ae0 in sysctl_core_net_init
                                               at net/core/sysctl_net_core.c:213
4       breakpoint     keep y   0xffffffff81552cb0 in sysctl_net_init at net/sysctl_net.c:68
5       breakpoint     keep y   0xffffffff811f02c0 in setup_sysctl_set
                                               at fs/proc/proc_sysctl.c:1577
~~~


## breakpoints
* setup_sysctl_set
* sysctl_net_init
*  tcp_init_sock
*  inet_ioctl
* sys_socketcall
* sys_socket
* do_fork
* start_kernel
* start_thread
* page_alloc_init
* kthread

* sock_alloc
* sock_alloc_file <=良い感じ
* d_instantiate

https://lists.gnu.org/archive/html/qemu-devel/2012-08/msg03921.html

## backtraceなログ

~~~
(gdb) bt
#0  0xffffffff8135d345 in inb (p=0xffffffff81ebe060, offset=<value optimized out>)
    at /usr/local/src/linux-3.9.2/arch/x86/include/asm/io.h:308
#1  io_serial_in (p=0xffffffff81ebe060, offset=<value optimized out>)
    at drivers/tty/serial/8250/8250_core.c:428
#2  0xffffffff8135d939 in serial_port_in (port=0xffffffff81ebe060)
    at include/linux/serial_core.h:201
#3  serial8250_get_poll_char (port=0xffffffff81ebe060) at drivers/tty/serial/8250/8250_core.c:1897
#4  0xffffffff81358f79 in uart_poll_get_char (driver=<value optimized out>,
    line=<value optimized out>) at drivers/tty/serial/serial_core.c:2227
#5  0xffffffff8136454d in kgdboc_get_char () at drivers/tty/serial/kgdboc.c:236
#6  0xffffffff810de5fa in gdbstub_read_wait () at kernel/debug/gdbstub.c:83
#7  0xffffffff810df70d in get_packet (buffer=<value optimized out>) at kernel/debug/gdbstub.c:102
#8  0xffffffff810df90d in gdb_serial_stub (ks=0xffff88021fc49db8) at kernel/debug/gdbstub.c:975
#9  0xffffffff810dde88 in kgdb_cpu_enter (ks=0xffff88021fc49db8, regs=0xffff88021fc49f58,
    exception_state=<value optimized out>) at kernel/debug/debug_core.c:611
#10 0xffffffff810de488 in kgdb_handle_exception (evector=<value optimized out>, signo=5, ecode=3,
    regs=0xffff88021fc49f58) at kernel/debug/debug_core.c:693
#11 0xffffffff81042881 in __kgdb_notify (args=0xffff88021fc49ed8, cmd=<value optimized out>)
    at arch/x86/kernel/kgdb.c:564
#12 0xffffffff81042957 in kgdb_notify (self=<value optimized out>, cmd=<value optimized out>,
    ptr=<value optimized out>) at arch/x86/kernel/kgdb.c:597
#13 0xffffffff81575045 in notifier_call_chain (nl=<value optimized out>, val=3,
    v=0xffff88021fc49ed8, nr_to_call=-4, nr_calls=0x0) at kernel/notifier.c:93
#14 0xffffffff81575082 in __atomic_notifier_call_chain (nh=<value optimized out>,
    val=<value optimized out>, v=<value optimized out>, nr_to_call=<value optimized out>,
    nr_calls=<value optimized out>) at kernel/notifier.c:182
#15 0xffffffff815750a6 in atomic_notifier_call_chain (nh=<value optimized out>,
    val=<value optimized out>, v=<value optimized out>) at kernel/notifier.c:191
#16 0xffffffff815750de in notify_die (val=<value optimized out>, str=<value optimized out>,
    regs=<value optimized out>, err=<value optimized out>, trap=<value optimized out>,
    sig=<value optimized out>) at kernel/notifier.c:541
#17 0xffffffff81571993 in do_debug (regs=0xffff88021fc49f58, error_code=0)
    at arch/x86/kernel/traps.c:430
---Type <return> to continue, or q <return> to quit---
#18 0xffffffff815712eb in ?? () at arch/x86/kernel/entry_64.S:1464
#19 0xffffffff810e9c57 in arch_local_irq_restore ()
    at /usr/local/src/linux-3.9.2/arch/x86/include/asm/paravirt.h:833
#20 rcu_idle_exit () at kernel/rcutree.c:548
#21 0xffffffff8101ccc5 in cpu_idle () at arch/x86/kernel/process.c:350
#22 0xffffffff815655be in start_secondary (unused=<value optimized out>)
    at arch/x86/kernel/smpboot.c:287
#23 0x0000000000000000 in ?? ()
~~~


* net/socket.c
SYSCALL_DEFINE2


## kernel REmake
~~~
(gdb) info b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0xffffffff811f02c9 in setup_sysctl_set
                                               at fs/proc/proc_sysctl.c:1578
2       breakpoint     keep y   0xffffffff81552cb9 in sysctl_net_init at net/sysctl_net.c:69
3       breakpoint     keep y   <MULTIPLE>
3.1                         y     0xffffffff814e42ae in tcp_init_sock
                                               at include/linux/skbuff.h:1012
3.2                         y     0xffffffff814e42cc in tcp_init_sock
                                               at include/linux/skbuff.h:1012
3.3                         y     0xffffffff814e42df in tcp_init_sock
                                               at include/linux/skbuff.h:1012
3.4                         y     0xffffffff814e430f in tcp_init_sock
                                               at include/linux/skbuff.h:1012
4       breakpoint     keep y   0xffffffff8150b069 in inet_ioctl at net/ipv4/af_inet.c:876
5       breakpoint     keep y   0xffffffff81485229 in sys_socketcall at net/socket.c:2436
6       breakpoint     keep y   0xffffffff81482a39 in sys_socket at net/socket.c:1395
     breakpoint already hit 1 time
~~~


~~~
(gdb) bt
#0  sys_socket (family=16, type=3, protocol=9) at net/socket.c:1395
#1  0xffffffff81579999 in ?? () at arch/x86/kernel/entry_64.S:644
#2  0x00007f1751c2d9b7 in __brk_reservation_fn_early_pgt_alloc__ ()
#3  0xffff880215364000 in __brk_reservation_fn_early_pgt_alloc__ ()
#4  0xffff880216738800 in __brk_reservation_fn_early_pgt_alloc__ ()
#5  0x0000000000000000 in ?? ()
~~~

http://www.slideshare.net/libfetion/linux-kernel-debugging



EXTRA_CFLAGS += -O0

~~~
(gdb) bt
#0  sock_alloc () at net/socket.c:535
#1  0xffffffff814834d0 in __sock_create (net=0xffffffff81ac4140, family=16, type=3, protocol=9,
    res=0xffff8802152bff58, kern=0) at net/socket.c:1296
#2  0xffffffff814836d4 in sock_create (family=16, type=3, protocol=9, res=0xffff8802152bff58)
    at net/socket.c:1372
#3  0xffffffff81483771 in sys_socket (family=16, type=3, protocol=9) at net/socket.c:1402
~~~

# エラー

* Remote 'g' packet reply is too long

  ~~~
  Remote 'g' packet reply is too long: 00000000000000001000a081ffffffff0000000000004001000000000000000000000000000040018200000000000000c81ea081ffffffffc81ea081ffffffff00000000000000000100000000000000000000000000000000000000000000000000000000000000e042ba81ffffffff0000000000000000c03d09000000000016570481ffffffff4602000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007f0300000000000000000000000000000000000000000000000000000000000000000000ffffffffffffffffffffffffffffffff0001010101010101010100015445524d0053494753544b464c5400530000000000000000000000000000000000000000000000003100000000000000404040404040404040404040404040405b5b5b5b5b5b5b5b5b5b5b5b5b5b5b5b202020202020202020202020202020200000000000000000000000000000000000000000000000ff0000000000ffffff20202020202000202020202020202000ffffffffffffffffffffffffffffffff00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000801f0000
  ~~~

  * 対処法

    ~~~
    (gdb) set architecture i386:x86-64:intel
    ~~~

# aiueo

* read write 部分を読もう
  * do_sock_write
  * do_sock_read
* 深いところにbreakpointを仕掛ける
  * btチェック
    * ご自由にvimとかで追いかける

# 参考

* [QEMU上のLinuxカーネルをGDBでデバッグする](http://hiroom2.jimdo.com/2014/01/15/qemu%E4%B8%8A%E3%81%AElinux%E3%82%AB%E3%83%BC%E3%83%8D%E3%83%AB%E3%82%92gdb%E3%81%A7%E3%83%87%E3%83%90%E3%83%83%E3%82%B0%E3%81%99%E3%82%8B/)
* [LinuxカーネルHack: GDBとKVMによるカーネルデバッグ](http://d.hatena.ne.jp/fixme/20101120/1290242209)
* [GDB stub on kvm, 復活の巻](http://d.hatena.ne.jp/big-eyed-hamster/20091211/1260540819)
* [GDBの便利技: (gdb)> でコードビューアを立ち上げて快適デバッグ](http://d.hatena.ne.jp/fixme/20100906/1283727489)
* [KVM上のFedora 16 kernelをgdbでデバッグ](http://blog-ja.intransient.info/2011/11/kvmfedora-16-kernelgdb.html)
* [kgdbを用いたカーネルデバッグ環境の構築](http://d.hatena.ne.jp/big-eyed-hamster/20090331/1238470673)
