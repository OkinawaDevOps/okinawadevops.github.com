---
layout: post
tags : [Linux, Kernel, ]
---
{% include JB/setup %}

### [yamanetoshi](https://yamanetoshi.github.io/)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/32)

### ソースコード

以下にて取得。

    $ git clone https://code.google.com/p/xv6-rpi/

### 確認

可能な限り [pi-baremetal](https://github.com/brianwiddas/pi-baremetal) と比較しつつ読みたいがどうなるか。

### 掘削ログ

エントリポイントは src/entry.S なんですが、kernel.ld 見てみるに以下な記述あり。

    SECTIONS
    {
      /* the entry point, before enabling paging. The code to enable paing
       needs to have the same virtual/physical address. entry.S and start.c
       run in this initial setting.*/
      . = 0x10000;

こちら、QEMU 向け、な記述になっている模様。これを

      . = 0x8000

にすれば良いのかどうか。

#### entry.S ざっくり

kernel.ld で定義されている以下の番地を 0 で初期化しています。

    /*define a stack for the entry*/
    . = ALIGN(0x1000);
    . += ENTRY_SVC_STACK_SIZE;
    
    PROVIDE (svc_stktop = .);
    
    /* define the kernel page table, must be 16K and 16K-aligned*/
    . = ALIGN(0x4000);
    PROVIDE (_kernel_pgtbl = .);
    . += 0x4000;
    
    /* we also need a user page table*/
    PROVIDE (_user_pgtbl = .);
    . += 0x1000;
    
    PROVIDE(end_entry = .);

src/entry.S に以下なコメントの記述があります。

    # clear the entry bss section, the svc stack, and kernel page table

その後、supervisor (SVC) mode に移行してスタックポインタ設定している模様。

    # initialize stack pointers for svc modes
    MSR     CPSR_cxsf, #(SVC_MODE|NO_INT)
    LDR     sp, =svc_stktop

CPSR_cxsf という命令については別途確認が必要。そして直後で start という手続きに制御を移しています。

    BL      start

#### start 手続きについて

定義は src/start.c です。とりあえず set_bootpgtbl という手続きの確認が必要みたいです。コメントにあるように最初は KERNBASE な 0x80000000 と 0x00000000 が同じページを指すようにするはず。

    // double map the low memory, required to enable paging
    // we do not map all the physical memory
    set_bootpgtbl(0, 0, INIT_KERNMAP, 0);
    set_bootpgtbl(KERNBASE, 0, INIT_KERNMAP, 0);

あと INIT_KERNMAP は 0x100000 なんですが、これって 1MB って理解で良いのかな。

### うーん

なんかどうもこれって QEMU 向けに書かれてるような気がしてならないので、やっぱこちら掘削続けます。

- https://github.com/brianwiddas/pi-baremetal

微妙で申し訳ありません。



### 参考にしたエントリなど

- [pi でベアメタルプログラミング](http://blog.bobuhiro11.net/2014/01-13-baremetal.html)

