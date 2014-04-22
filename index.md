---
layout: page
title: OkinawaDevOps
tagline: 
---
{% include JB/setup %}

はじめに
-------------------

OkinawaDevOps は基本的に[もくもく勉強会](http://www.1x1.jp/blog/2013/12/lets-try-moku-moku-study-event.html)となります。また、すすめかたについては[すごい広島](http://great-h.github.io/rule.html)のルールに沿ってみたいと考えています。

最初の手順
------------------

+ [リポジトリ](https://github.com/OkinawaDevOps/okinawadevops.github.com)の Issue でもくもくなネタを宣言します
+ 作成した Issue の ID と自分の ID をもとに branch を作ります
+ /_posts/yyyy-mm-dd-<自分のid>.md を作成して、とりあえず「自分の名前」および「作成した Issue へのリンク」を作成します
+ Pull Request を作ります
+ もくもくしたり参加者と情報交換したりしてください
+ もしかすると自己紹介タイムを設けるかもしれません

Issue について
------------------

もくもくするネタを投入するだけではなく、自分が何をしているか、とか自分がどんな人なのか、という情報もあると良いかもしれません。

作成する markdown について
------------------

[すごい広島](http://great-h.github.io/rule.html)では以下のフォーマットが推奨されています。暫くこれを使わせて頂く方向で。

    ### [名前][URL]
    
    * [やること宣言][Issue へのリンク]

ファイル作成や Pull Request などについて
-------------------

ファイルを新規に作成する場合は必ず branch を作って作業してください。また、Pull Request については二名以上のメンバによる LGTM が無いと merge できない、というルール設定をしておきたいと思っています。


## 投入されたポスト

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

## 謝辞

[すごい広島](http://great-h.github.io/rule.html)の先進的な取り込みに感謝します。


