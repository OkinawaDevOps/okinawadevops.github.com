### [やまねとしあき](http://twitter.com/yamanetoshi)

- [Docker 修行](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/17)

Ubuntu 14.04 な仮想ホストを作って

    $ sudo docker pull ubuntu

してます。以下から一式取得して

- [yss44/docker_supervisord](https://github.com/yss44/docker_supervisord)

Dockerfile を弄って

    docker build -t rms/supervisord .
    docker run -p 80 -p 2222 -d rms/supervisord

してたのですが Exit 1 してるのが今です。

その後のログは以下にて。。

- [Docker もくもく](http://yamanetoshi.github.io/blog/2014/05/24/okinawa-devops/)
