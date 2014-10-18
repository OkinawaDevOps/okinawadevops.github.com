---
layout: post
tags : [kernel]
---
{% include JB/setup %}

### [Yosuke OTA](https://twitter.com/y0t4)

* [やること](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/73)

9/21(Sun)なkernelvmに向けてとりあえずkernelをgdbで追っかけていました。

### 発表しようかな、と思っていること

KVMを利用してVM自体をdebugしてみる

* kgdbという機能がkernel-2.6.26で追加された
* この機能を使うとkernelがどんなことをやっているか知るのに役立つのではないか？
  * じゃあやってみよう

### やったこと

* KVM環境の用意
* VMの作成
* kernelのビルド
* gdbでの接続
