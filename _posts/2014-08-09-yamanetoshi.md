---
layout: post
tags : [Docker, ]
---
{% include JB/setup %}

### [yamanetoshi](https://yamanetoshi.github.io/)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/57)

### お仕事

おかげさまでいくつか不具合倒せました。

### お仕事そのに

K/VM 沖縄のうちあわせをたいらさんと。

- 発表者なかたがたへのアクセス
- 懇親会
- UStream 機材

など、TODO 整理。あとタイムテーブルも作りました。

- https://docs.google.com/spreadsheets/d/1rMzUnacdKXP2AvjJLWlSwP8GUBuD0HZWpLl9ag9aL4c/edit#gid=0

### Docker

残り一時間ですがとりあえず以下を。

- MySQL なコンテナを作ってみる
- Apache (ぺちぴーが動く) なコンテナを作ってみる

というか、CakePHP て Rails みたいな migration な機能ってあるんかな。とりあえず MySQL については以下なソレが公開されてました。

- https://github.com/tutumcloud/tutum-docker-mysql

あるいは apache/php もあり

- https://github.com/tutumcloud/tutum-docker-php

あるいは php-fpm なソレ?

- https://registry.hub.docker.com/u/jprjr/php-fpm/

ええと、基本的な環境、ってあたりで言うと

- 以下な Dockerfile で動く MySQL なコンテナ起動
 - https://github.com/tutumcloud/tutum-docker-mysql
 - $ sudo docker.io run -d --name mysql mysql:latest
- 以下から取得したソレを起動
 - https://registry.hub.docker.com/u/jprjr/php-fpm/
 - sudo docker.io run -d -p 9000:9000 --link mysql:mysql --name php-fpm hoge:latest

なカンジなのかどうか。課題としては

 - アプリをどのディレクトリに置くか、が分かってない
 - migration な仕組みがあるのかどうか
 - よく考えたら attach して云々すりゃ良いの? (駄目でしょ

あたりなのかどうか。

次回、これを踏まえてぺちぴーで云々な環境をアレしてみたいと思ってます。検討だけで終わっちゃいましたがご容赦頂けますと幸いです。
 
