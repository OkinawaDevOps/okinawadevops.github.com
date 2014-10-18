---
layout: post
tags : [misc, ]
---
{% include JB/setup %}

### [yamanetoshi](https://yamanetoshi.github.io/)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/85)

### K/VM 関連


- [まとめサイト](https://sites.google.com/site/kernelvm/ima-made-no-matome/okinawa)の作成

スライド URL いくつか不明なのがあるのと、動画を youtube に置く必要があるようで途中で断念。


### A Hacker’s Guide to Git 確認

- [A Hacker’s Guide to Git](https://wildlyinaccurate.com/a-hackers-guide-to-git)

着手 1600 なので全部読めないと思います。とりあえず introduction はスルー。

Repositories

- Git repository はシンプルな key-value data store とのこと
- Blob は Git における基本的なデータ型、byte の纒まり、通常 file のバイナリ表現?
- Tree object は blob または他の tree object へのポインタを持つ
- Commit object は一つの tree object を指してて commit に関するメタデータ (parent commit など) を持つ
- Tag object は一つの commit object を指してていくつかのメタデータを持つ
- Reference は commit とか tag などの単一のオブジェクトを指すもの?
- Git repository はプロジェクトの root directory にある .git ディレクトリ
- リポジトリは git init コマンドで作成される
- 重要なディレクトリとして Git が保管する全ての object が保管される .git/object と全ての Reference が保管される .git/refs がある

Tree Objects

- Git における tree object はディレクトリと考えることができる
- tree object (directory) は blobs (files) のリストと他の tree objects (sub directories) を持つ
- README と src/hello.c なリポジトリで確認
- 二つの tree object がある (root directory と src directory)
- blob も tree も hash なキーを持っているのか

Commits

- commit object は基本的にポインタ
- 重要なメタデータのいくつかを持つ
- commit 自体は いくつかの metadata から作られた hash を持つ
- commit 時の tree (Tree Object で見たように Git は working tree 全体を再帰的に tree の中に作ることができる)
- parent commit の hash (commit は必ず直前 commit の hash を持つ)
- 作成者の名前と email address と変更された時刻
- committer の名前と email address と commit が作成された時刻
- commit message
- git show --format=raw というコマンドの出力について
- たしかに commit object と tree object がある

References

- Git では object は hash で識別される
- Git で云々する場合、hash を知っておくことは大切
- hash 覚えとけ、を要求されるorz
- そのために refs ってものがあると
- .git/refs に投入される
- branch って reference なのか
- .git/refs/heads/master というファイル
- 中身は hash (多分 master の HEAD)
- show および rev-parse というコマンド
- git も特別な reference として HEAD がある

とりあえず今日はここまで、ということで。別途他の場所で続きを書きます。
