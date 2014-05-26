### [新垣雄志](https://twitter.com/arakaji)

* [入門Chefsoloを読みながら色々試します](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/16)

#### Chefで使われる言葉

+ レシピ -- 「コード化された手順書」あるいは「サーバーの状態」
+ クックブック　-- 特定のレシピに必要なデータやファイルをまとめる入れもの（ファイルに対するディレクトリやクラスに対する名前空間）
+ レポジトリ（キッチン）-- クックブック群を含む、chefの実行に必要な一連のファイルをまとめる入れもの。レポジトリは特定のシステムに１個くらいの粒度の大きさ
+ Resource -- logやpackageなどサーバーの状態になにがしかの影響を与える命令をChefでは"Resource"と呼ぶ
+ Attribute -- templateで使われる変数のこと. 変数の値は別途JSONファイルに記述する
+ node  --  管理対象のサーバーのことをノード(node)という.JSONファイルに書いてるデータ構造をNode Object という.つまりNode Object(JSONファイル)は基本、Chefで管理する対象ノードごとに一つ作る
+ knife-solo -- knifeのプラグイン

#### Vagrantのプラグイン -- sahara --
  URL : https://github.com/jedi4ever/sahara
  このプラグインを入れると仮想環境を色々イジッタ後にロールバック出来るという優れもの


    # sahara を インストール
    $ vagrant plugin install sahara

    # sandbox モードを有効にする
    $ vagrant sandbox on

    # sandbox on した所なでOSの状態を戻す
    $ vagrant sandbox rollback

    # OSの状態変更を確定
    $ vagrant sandbox commit

    # sandbox モードを解除
    $ vagrant sandbox off


#### knifeのプラグイン -- knife-solo --

  URL : http://matschaffer.github.io/knife-solo/
  knife に chef-soloをより便利に使うための機能を追加する
    
    # Chefスタンダードのディクレク構造を作成する(キッチン、レポジトリ)
    $ knige solo init

    # 指定してhostにChefをinstallさせる。OSの違いを自動で検知して対応してくれる
    $ knife solo prepare

    # キッチン(Chef repository)を指定したhostにuploadして、chef-soloを実行する。
    $ knife solo cook 

    # prepare と cook を同時に実行してくれる
    $ knife solo bootstrap

    # Upload したキッチンを指定したhostから削除する
    $ knife solo clean


  knife-solo がssh経由でchef-soloを実行する都合上、sshに使われるログインユーザーが
  sudoかつパスワードなしでchef-soloを実行できる権限をもっている必要があります。
  (鍵認証しているならその設定も必要)

                      


