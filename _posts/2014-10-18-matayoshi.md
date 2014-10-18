---
layout: post
tags : [Linux,PHP,Apache,worker MPM,mod_fcgid]
---
{% include JB/setup %}

### [Matayoshi](https://twitter.com/nmatayoshi)

* [Apache worker mpm + PHP な環境構築](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/87])


## CentOS 6.5 x64 な vagrant を起動

    $ mkdir -p ~/vagrant/worker_test
    $ cd ~/vagrant/worker_test
    $ vagrant box add ~/vagrant/boxes/CentOS_6.5.x86_64/centos6.5.x86_64.20140616.box --name centos_6.5
    $ vagrant init centos_6.5
    $ vi Vagrantfile

適当にメモリーとか調整して

    vagrant up

準備完了

とりあえずはApacheとPHPのソースを落とす
と、その前に諸々update

    $ vagrant ssh
    $ sudo -i
    # yum update
    # yum clean all

まぁ、リブートですよね

    $ vagrant reload


## Apache を worker mpm でビルド

使うのは httpd-2.2.29

    $ cd /vagrant/src
    $ wget http://www.apache.org/dist/httpd/httpd-2.2.29.tar.gz.md5
    $ wget http://ftp.tsukuba.wide.ad.jp/software/apache//httpd/httpd-2.2.29.tar.gz
    $ md5sum --check httpd-2.2.29.tar.gz.md5
    $ sudo -i
    # cd /usr/local/src
    # tar xf /vagrant/src/httpd-2.2.29.tar.gz
    # cd httpd-2.2.29/
    # yum install zlib zlib-devel
    # yum install openssl openssl-devel
    # ./configure \
        --prefix=/usr/local/httpd-2.2.29 \
        --enable-deflate \
        --enable-expires \
        --enable-headers \
        --enable-ssl \
        --enable-rewrite \
        --enable-so \
        --enable-fastcgi
        --with-included-apr \
        --with-mpm=worker
    # make
    # make install


## mod_fcgid をビルド&インストール

mod_fastcgi と mod_fcgid があるが、 mod_fcgid を使う

    # cd /vagrant/src
    # wget http://ftp.tsukuba.wide.ad.jp/software/apache//httpd/mod_fcgid/mod_fcgid-2.3.9.tar.gz
    # wget http://www.apache.org/dist/httpd/mod_fcgid/mod_fcgid-2.3.9.tar.gz.md5
    # md5sum --check mod_fcgid-2.3.9.tar.gz.md5
    # cd /usr/local/src/
    # tar xf /vagrant/src/mod_fcgid-2.3.9.tar.gz
    # cd mod_fcgid-2.3.9
    # APXS=/usr/local/httpd-2.2.29/bin/apxs ./configure.apxs
    # make
    # make install

Apacheを起動

    # /usr/local/httpd-2.2.29/bin/httpd -V
    Server version: Apache/2.2.29 (Unix)
    Server built:   Oct 18 2014 15:50:18
    Server's Module Magic Number: 20051115:36
    Server loaded:  APR 1.5.1, APR-Util 1.5.3
    Compiled using: APR 1.5.1, APR-Util 1.5.3
    Architecture:   64-bit
    Server MPM:     Worker
      threaded:     yes (fixed thread count)
        forked:     yes (variable process count)
    Server compiled with....
     -D APACHE_MPM_DIR="server/mpm/worker"
     -D APR_HAS_SENDFILE
     -D APR_HAS_MMAP
     -D APR_HAVE_IPV6 (IPv4-mapped addresses enabled)
     -D APR_USE_SYSVSEM_SERIALIZE
     -D APR_USE_PTHREAD_SERIALIZE
     -D SINGLE_LISTEN_UNSERIALIZED_ACCEPT
     -D APR_HAS_OTHER_CHILD
     -D AP_HAVE_RELIABLE_PIPED_LOGS
     -D DYNAMIC_MODULE_LIMIT=128
     -D HTTPD_ROOT="/usr/local/httpd-2.2.29"
     -D SUEXEC_BIN="/usr/local/httpd-2.2.29/bin/suexec"
     -D DEFAULT_SCOREBOARD="logs/apache_runtime_status"
     -D DEFAULT_ERRORLOG="logs/error_log"
     -D AP_TYPES_CONFIG_FILE="conf/mime.types"
     -D SERVER_CONFIG_FILE="conf/httpd.conf"
    # /usr/local/httpd-2.2.29/bin/httpd -M
    httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain for ServerName
    Loaded Modules:
     core_module (static)
     authn_file_module (static)
     authn_default_module (static)
     authz_host_module (static)
     authz_groupfile_module (static)
     authz_user_module (static)
     authz_default_module (static)
     auth_basic_module (static)
     include_module (static)
     filter_module (static)
     deflate_module (static)
     log_config_module (static)
     env_module (static)
     expires_module (static)
     headers_module (static)
     setenvif_module (static)
     version_module (static)
     ssl_module (static)
     mpm_worker_module (static)
     http_module (static)
     mime_module (static)
     status_module (static)
     autoindex_module (static)
     asis_module (static)
     cgid_module (static)
     negotiation_module (static)
     dir_module (static)
     actions_module (static)
     userdir_module (static)
     alias_module (static)
     rewrite_module (static)
     so_module (static)
     fcgid_module (shared)
    Syntax OK


    # /usr/local/httpd-2.2.29/bin/apachectl start

http://localhost/ にアクセスすると"It works"がでた


## PHP をビルド&インストール

    # cd /vagrant/src
    # wget http://jp1.php.net/get/php-5.5.18.tar.gz/from/this/mirror -O php-5.5.18.tar.gz
    # echo "53cdc7589cb301871888c7776eed3cf9 *php-5.5.18.tar.gz" > php-5.5.18.tar.gz.md5
    # md5sum --check php-5.5.18.tar.gz.md5
    # cd /usr/local/src
    # tar xf /vagrant/src/php-5.5.18.tar.gz
    # cd php-5.5.18/
    # yum install libxml2 libxml2-devel
    # ./configure \
        --prefix=/usr/local/php-5.5.18 \
        --with-openssl \
        --with-mysql \
        --with-mysqli \
        --with-pdo-mysql \
        --enable-mbstring \
        --enable-cgi \
        --enable-fpm \
        --with-fpm-user=daemon \
        --with-fpm-group=daemon
    # make


今日はここまで


## 参考

* [PHP: Apache 2.x (Unixシステム用) - Manual](http://php.net/manual/ja/install.unix.apache2.php)
* [Apache ウェブサーバーで FastCGI を利用する : Movable Type 6 ドキュメント](http://www.movabletype.jp/documentation/mt6/reference/apache-fastcgi.html)
* [PHP: FastCGI Process Manager (FPM) - Manual](http://php.net/manual/ja/install.fpm.php)
