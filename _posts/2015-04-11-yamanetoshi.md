---
layout: post
tags : [Fab, ]
---
{% include JB/setup %}

### [yamanetoshi](https://yamanetoshi.github.io/)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/116)

### 面白いドキュメント見つけてます、ご参考まで。

- [開発フロー研修 @ Wantedly](http://qiita.com/awakia/items/c571e93e96a1ec28044f)

### 本読み

以下を持ってきてます。

- コンピュータシステムの理論と実装
- Think Stats
- English Grammer in Use

とりあえず以下のリポジトリを作成して本読みおよび課題作成着手します。

- [Machine Language](https://github.com:yamanetoshi/MachineLanguage.git)

Mult.asm をサンプル見つつでっち上げたがとりあえず以下に目を通せとのことで確認。

- [CPU Emulator Tutorial](http://www.nand2tetris.org/tutorials/PDF/CPU%20Emulator%20Tutorial.pdf)
- [Assembler Tutorial](http://www.nand2tetris.org/tutorials/PDF/Assembler%20Tutorial.pdf)

とりあえずアセンブルは GUI でやりました。手順としては以下。

- tools/Assembler.sh で Assembler な GUI 起動
- tools/CPUEmulator.sh で CPUEmulator な GUI 起動
- Assembler な GUI で .asm を読み込んで翻訳
- 機械語への翻訳が成功したなら .hack に保存 (忘れやすい)
- CPUEmulator にて .hack を読み込み
- CPUEmulator にて .test を読み込み
- 試験実行

Mult.asm 作ってみて分かったのですが、初心者向けには 1+...+100 の和なサンプルをいくつか改造して動作を確認しておく必要がある気がしています。

- 1+..+5 程度にしてみるとか
- 1+1+1 を求めてみるとか

そもそも

    while (i <= 100) {

みたいな繰り返しの条件だったので i - 100 が 0 より大きければループ中断、なのがアレです。そういった意味では

    int i = 0;
    int sum = 0;
    while (i < 100) {
        sum += i;
        i++;
    }

みたいなナニを書き替えたらどうなるか、なあたりもヤッておいた方が良い気はしています。

Fill.asm についてもスクリーンの概念をきちんと理解させる必要あり。例えば簡単な例で 2 行目の 2 番目のビットに対応するメモリというのは 0x4000 に 32 ((* 32 (- 2 1))) と 0 ((/ 16 (- 2 1))) を加算した位置の 2 番目のビットという事になります。

そういった意味ではこの問題については 0x0 あるいは 0xff を該当するメモリ (というか全部か) に順に書込めば良いのかな。

あと、キーボードのナニについては乱暴に 0 かそうでないかで判定すれば良いはず。つうか今 4.2.4 見て知ったのですが R0 から R15 はシンボルになっているのか。KBD ってのもあるのですね。

ということで要点を以下に列挙。

- 無限ループ
- キーボードの値を読み込んでレジスタに確保
- 値が 0 なら白くしてそうでなければ黒くする
- ここで jmp な条件分岐なのか
- キーボードの値を D に格納して
- D;JEQ で 0 の場合のアドレスに飛ばすのか
- 条件分岐については 73p に例示されています
- 白は 0 で黒は -1 ね
- 横 32 で縦 256 なループになるのか
- 最初に M に @SCREEN を指させて
- 32x256 (8192) 回繰り返しつつ M に 1 加算
- あ、条件分岐である領域に 0 か -1 を設定しときゃ良いのかな
- 多分レジスタは駄目
- 条件分岐で設定した領域から繰返しな処理でスクリーンなメモリに値を設定すりゃ良さげ

ここまで理解できていればプログラムが書けるのかどうか。

しまった。

- 前の状態をとっておいてそれで描画するかどうかをキメる必要あることにコード書いてて気づきました
- あと、@ で参照した後の A と M の意図を理解しておく必要あり

で、ポインタ参照どうやるの、ってあたりで躓いている所でタイムアップでして、別途作業続行させて頂きます。

### new commer  向けドキュメント

作れれば作る。
