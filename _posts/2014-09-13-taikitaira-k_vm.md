---
layout : post
tags : [KernelVM, efi, ]
---
{% include JB/setup %}

# Kernel/VM について

## 発表どうしよう
* LT 枠 (15分 @y0t4 と別枠)
* 資料作成
* 発表練習

## 発表ネタ

### ELILO で何か
* ELILO とは
* EFI とは
* ライブラリ
    * gnu-efi
    * EDKII
    * UDK2010
    * UDK2014
        * なんか新しいの出てる！
* ELILO のソースよみ【Linux のブートまで】
    * ELILO
        * efi_main
        * kernel_load
        * do_kernel_load
        * main_loop
        * create_boot_params
        * call ExitBootServices API
        * go to kernel
    * Linux
        * x86_64_start_kernel
        * start_kernel
        * efi_init
        * setup_memory_map
* PTE 
* API 叩き
    * ファイルを open / read / close する
    * ExitBootService 
* EFI Application として kernel は書けるか？

### 研究について
* 自作 OS 
* まだできてません
* 構成
    * CbC
    * データセグメント
    * コードセグメント
* 関連研究
    * Alice
    * Cerium
    * Jungle Database

### ie-virsh / ie-docker
* virsh / docker とは
    * libvirt / container
* ie-virsh / ie-docker とは
* 権限分離と管理の話
* 追加する予定のもの
* こんなことがあったよ
