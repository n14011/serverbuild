## 2-2 Vagrant設定

  1 Boxの追加  
   `$vagrant box add CentOS65 コピーしたboxファイル --force`  
  2 Vagrantの初期設定  
   `$vagrant init CentOS65`  
  3 ホストオンリーアダプターの設定  
    `$vi Vagrantfile`を開いて  

    Vagrant.configure(2) do |config|    
    config.vm.network :private_network, ip:"192.168.33.129"
    end

  ↑のように追加  
  4 Vagrantの起動  
   `$vagrant up`  
  5 Vagrantへ接続  
   `$vagrant ssh`  

## centosプロキシの設定  
 1 プロキシを設定する  
   `$vi /etc/profile` ファイルに下記を追記する  

     PROXY='172.16.40.1:8888'  
     export http_proxy=$PROXY  
     export HTTP_PROXY=$PROXY  
     export HTTPS_PROXY=$PROXY  
     export https_proxy=$PROXY  

  2 yumにプロキシを設定  
   `$vi /etc/yum.conf` ファイルに下記を追記  
   `proxy=http://172.16.40.1:8888/`  
  3 wgetにプロキシを設定  
   `$vi /etc/wgetrc` ファイルに下記を追記  

    http_proxy = 172.16.40.1:8888/  
    https_proxy = 172.16.40.1:8888/  
    ftp_proxy = 172.16.40.1:8888/  

  4 プロキシの設定が終わったのでupdateする  
   `$yum update`  

## 2−2 Wordpressに必要なものをインストール
  1 PHPのインストール  
   `$yum install -y php-mysql php php-gd  php-fpm`  
  2 mariadbのインストール  
   centos7の場合  
   `$yum install -y mariadb mariadb-server`  

   centos6.5の場合  

    mariadbのリポジトリ登録  
    `vi /etc/yum.repos.d/MariaDB.repo`  
    [mariadb]    
    name = MariaDB  
    baseurl = http://yum.mariadb.org/5.5/centos6-amd64  
    gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB  
    gpgcheck=1  

   mariadbのインストール  
   `sudo yum -y install MariaDB-server MariaDB-client`  
  3 nginxのリポジトリ登録  
  `$vi /etc/yum.repos.d/nginx.repo`  

     [nginx]    
     name=nginx repo  
     baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/  
     gpgcheck=0  
     enabled=0  

  4 nginxのインストール
    `$yum --enablerepo=nginx install nginx`

    ※他の方法でインストール
    `$wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`  
    リポジトリの登録  
    `$rpm -i http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm`  
    インストール  
    `$yum -y install nginx`  

  5 nginxの実行と登録   

   centos7の場合  

    $systemctl start nginx  
    $systemctl enable nginx  

   centos6.5の場合  

    $service nginx start  
    $chkconfig nginx on  

  6 php-fpmの実行と登録  

   centos7の場合  

    $systemctl start php-fpm  
    $systemctl enable php-fpm  

   centos6.5の場合  

    $service php-fpm start  
    $chkconfig php-fpm on  

  7 ngixの設定でphp-fpmが動くようにする  
   `$vi /etc/nginx/conf.d/default.conf`  
   下記のように編集する  

      location / {  
      root   /usr/share/nginx/html;  
      index  index.html index.htm;  
      }  
 	↓  
      location / {  
      root   /usr/share/nginx/html;  
      index  index.php index.html index.htm;  
      }  


     location ~ \.php$ {  
      root           html;  
      fastcgi_pass   127.0.0.1:9000;  
      fastcgi_index  index.php;  
      fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;  
      include        fastcgi_params;  
      }  
    ↓  
     location ~ \.php$ {  
        root           /usr/share/nginx/html;  
        fastcgi_pass   127.0.0.1:9000;  
        fastcgi_index  index.php;  
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  
        include        fastcgi_params;  
      }  
    

## 2-2 Wordpressで使うデータベースの作成
  1 Mysqlの起動  
   centos7の場合  
   `$systemctl start mariadb`  
   centos6.5の場合  
   `$service mariadb start`  
  2 MySQLを起動する  
   `$mysql -u root`  
  3 rootユーザーにパスワードを設定する  
   `mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxxxxxxx');`  
  4 1回終了してrootでログイン  
   `mysql>exit`  
   `$mysql -u root -p`  
  5 wordpressで使うデータベースを作成する  
   `mysql> CREATE DATABASE データベース名;`  
  6 ユーザの作成  
   `mysql> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザー名'@'localhost' IDENTIFIED BY '任意パスワード';`  

## 2-2 Wordpressのインストール
  1 Wordpressのzipファイルをインストールする  
   `$wget https://ja.wordpress.org/wordpress-4.2.2-ja.zip`  
  2 インストールしたzipファイルを解凍する  
   `$unzip wordpress-4.2.2-ja.zip`  
  3 解凍したファイルを/usr/shard/ngix/htmlに移動する  
   `$mv wordpress-4.2.2-ja.zip /usr/shard/ngix/html`  
  4 wordpressのフォルダに移動  
   `$cd /usr/shard/ngix/html/wordpress`  
  5 wordpressフォルダの`wp-config-sample.php`をコピーして`wp-config.php`を作成する  
   `$cp wp-config-sample.php wp-confing.php`  
  6 コピーしたwp-config.phpの中を編集  
   `$vi wp-config.php`  

     WordPress のためのデータベース名  
     define('DB_NAME', 'database_name_here');  
    ↓
     define('DB_NAME', 'mysqlで作ったデータベース名');  
     MySQL データベースのユーザー名  
     define('DB_USER', 'username_here');  
    ↓
     define('DB_USER', 'mysqlで作ったデータベースの所有者名');  
     MySQL データベースのパスワード  
     define('DB_PASSWORD', 'password_here');  
    ↓
     define('DB_PASSWORD', 'mysqlで作ったデータベースの所有者名のパスワード');  

  7 ブラウザからWordpressにアクセス   
   `192.168.33.129/wordpress/wp-admin/install.php`  

## 2-3 Vagrant設定
  1 Boxの追加  
   `$vagrant box add CentOS65 コピーしたboxファイル --force`  
  2 Vagrantの初期設定  
   `$vagrant init CentOS65`  
  3 ホストオンリーアダプターの設定  
    `$vi Vagrantfile`を開いて  

    Vagrant.configure(2) do |config|    
    config.vm.network :private_network, ip:"192.168.33.130"
    end

   ↑のように追加  
  4 Vagrantの起動  
   `$vagrant up`   
  5 Vagrantへ接続  
   `$vagrant ssh`  

## 2-3 centosプロキシの設定
  1 プロキシを設定する  
   `$vi /etc/profile` ファイルに下記を追記する  

     PROXY='172.16.40.1:8888'    
     export http_proxy=$PROXY  
     export HTTP_PROXY=$PROXY  
     export HTTPS_PROXY=$PROXY  
     export https_proxy=$PROXY  

  2 yumにプロキシを設定  
   `$vi /etc/yum.conf`に下記を追記   
   `proxy=http://172.16.40.1:8888/`  
  3 wgetにプロキシを設定  
   `$vi /etc/wgetrc` ファイルに下記を追記  

    http_proxy = 172.16.40.1:8888/  
    https_proxy = 172.16.40.1:8888/  
    ftp_proxy = 172.16.40.1:8888/  

  4 プロキシの設定が終わったのでupdateする  
   `$yum update`  

## 2-3 Wordpressに必要なものをインストール
  1 Apacheをインストールするのに必要なもの  
   `$yum -y install gcc gcc-c++ pcre-devel openssl-devel wget`  
  2 APRのファイルのダウンロード  
   `$wget http://www.apache.org/dist/apr/apr-1.5.2.tar.gz`  
  3 APRファイルを解凍  
  `$tar zxf apr-1.5.2.tar.gz`  
  4 ARPファイルに移動  
   `$cd apr-1.5.2`  
  5 ARPインストール  

    $./configure  
    $make
    $make install`

  6 ARP-utilファイルのダウンロード  
   `$wget http://www.apache.org/dist/apr/apr-util-1.5.4.tar.gz`  
  7 ARP-utilファイルの解凍  
   `$tar zxf apr-util-1.5.4.tar.gz`  
  8 ARP-utilのファイルに移動  
   `$cd apt-util-1.5.4`  
  9 ARP-utilファイルのインストール  

    $./configure --with-apr=/usr/local/apr  
    $make  
    $sudo make install  

  10 Apacheファイルのダウンロード  
   `$wget http://www.apache.org/dist/httpd/httpd-2.2.29.tar.gz`  
  11 Apacheファイルの解凍  
   `$tar zxf httpd-2.2.29.tar.gz`  
  12 Apacheファイルに移動  
   `$cd httpd-2.2.29`  
  13 Apacheのインストール  

    $./configure --enable-so --enable-rewrite --enable-ssl --enable-mods-shared=all --enable-mpms-shared='prefork worker event'  
    $make  
    $sudo make install  

  14 Apacheの起動  
   `$sudo /usr/local/apache2/bin/apachectl start`  

    下記のメッセージが出たら  
    httpd: apr_sockaddr_info_get() failed for vagrant-centos65.vagrantup.com  
    httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName  

   `$sudo vi /etc/hosts`ファイルを下記のように書き換える  

    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4  
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6  
     ↓  
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 vagrant-centos65.vagrantup.com  
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6 vagrant-centos65.vagrantup.com   

  15 PHPファイルのダウンロード  
   `$wget "http://jp1.php.net/get/php-5.5.24.tar.gz/from/this/mirror" -O php-5.5.24.tar.zip`  
  16 PHPファイルの解凍   
  `$tar zxf php-5.5.24.tar.zip`  
  17 PHPのファイルに移動   
   `$cd php-5.5.24`  
  18 PHPのインストール   
   `$./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql`  
    下記のエラーがでたら  
   `configure: error: xml2-config not found. Please check your libxml2 installation.`  
   `$sudo yum install libxml2-devel`を実行  
   もう一度`$./configure --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql
   `を実行  
   `$make test`  
   `$make`  
   `$sudo make install`  
  19 php.ini ファイルを設定する  
   `$cp php.ini-development /usr/local/lib/php.ini`  
  20 Apache が特定の拡張子のファイルを PHP としてパースするよう設定する  
   `$sudo vi /usr/local/apache2/conf/httpd.conf`を開いて下記を追記  

    <FilesMatch \.php$>     
    SetHandler application/x-httpd-php   
    </FilesMatch>   

   wordpress 直下のディレクトリーが表示されないように下記を追記  

    <IfModule dir_module>  
    DirectoryIndex index.html index.php  
    </IfModule>  
      LoadModule php5_module modules/libphp5.so ←なかったら追記する  

  21 MySQLをインストールする  
   centos6の場合  
   `$yum install -y mysql mysql-devel mysql-server mysql-utilities`  
   centos7の場合  
   Mariadbのインストール  
   `$yum -y install mariadb mariadb-server`  
  22 PHPのphp.iniファイルにmysqlのパスを記述する  
   `$sudo vi /usr/local/lib/php.ini`  

    /socketでmysql.default_socket =を探して下記のように編集する  
    $mysql.default_socket =  
    ↓  
    $mysql.default_socket =/var/lib/mysql/mysql.sock  

## 2-3 Wordpressで使うデータベースの作成  
  1 MySQLのサービスを起動する  
    centos7の場合  
   `$systemctl start mysql`  
    centos6.5の場合  
   `$service mysqld start`  
  2 MySQLを起動する  
   `$mysql -u root`  
  3 rootユーザーにパスワードを設定する  
   `mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('xxxxxxxxxx');`  
  4 1回終了してrootでログイン  
   `mysql>exit`  
   `$mysql -u root -p`  
  5 wordpressで使うデータベースを作成する  
   `mysql> CREATE DATABASE データベース名;`  
  6 ユーザの作成  
   `mysql> GRANT ALL PRIVILEGES ON データベース名.* to 'ユーザー名'@'localhost' IDENTIFIED BY '任意パスワード';`  

## 2-3 Wordpressのインストール
  1 Wordpressのzipファイルをインストールする  
   `$wget https://ja.wordpress.org/wordpress-4.2.2-ja.zip`  
  2 インストールしたzipファイルを解凍する  
   `$unzip wordpress-4.2.2-ja.zip`  
  3 解凍したファイルを/var/www/htmlに移動する  
   `$mv wordpress-4.2.2-ja.zip /usr/local/apache2/htdocs`  
  4 wordpressのフォルダに移動  
   `$cd /usr/local/apache2/htdocs/wordpress`  
  5 wordpressフォルダの`wp-config-sample.php`をコピーして`wp-config.php`を作成する  
   `$cp wp-config-sample.php wp-config.php`  
  6 コピーしたwp-config.phpの中を編集  
   `$vi wp-config.php`  

     WordPress のためのデータベース名    
     define('DB_NAME', 'database_name_here');    
    ↓  
     define('DB_NAME', 'mysqlで作ったデータベース名');    
     MySQL データベースのユーザー名    
     define('DB_USER', 'username_here');    
    ↓  
     define('DB_USER', 'mysqlで作ったデータベースの所有者名');    
     MySQL データベースのパスワード    
     define('DB_PASSWORD', 'password_here');    
    ↓  
     define('DB_PASSWORD', 'mysqlで作ったデータベースの所有者名のパスワード');  

  7 ブラウザからWordpressにアクセス  
   `192.168.33.130/wordpress/wp-admin/install.php`  

## 2-4 ベンチマーク
  1 Ubuntu側(ホスト側)に`apache2-utils`インストール  
   `$sudo apt-get install apache2-utils`  
  2 abコマンドを使って性能を確かめる  
   1000ユーザが同時に1000リクエストを発行した場合を想定。  
   `$ab -k -c 1000 -n 1000 192.168.xxx.xxx/wordpress`  
   を実行したらこのような結果が表示される


     Server Software:        nginx/1.9.0  
     Server Hostname:        192.168.33.129  
     Server Port:            80  
  
     Document Path:          /wordpress  
     Document Length:        184 bytes  

     Concurrency Level:      1000  
     Time taken for tests:   0.701 seconds  
     Complete requests:      1000  
     Failed requests:        0  
     Non-2xx responses:      1000  
     Keep-Alive requests:    1000  
     Total transferred:      390000 bytes  
     HTML transferred:       184000 bytes  
     Requests per second:    1426.35 [#/sec] (mean)  
     Time per request:       701.089 [ms] (mean)  
     Time per request:       0.701 [ms] (mean, across all concurrent requests)  
     Transfer rate:          543.24 [Kbytes/sec] received  

     Connection Times (ms)  
                min  mean[+/-sd] median   max  
     Connect:        0   60  29.5     60     110  
     Processing:     0  302 137.7    302     585  
     Waiting:        0  302 137.7    302     585  
     Total:          0  361 159.7    365     680  
 
     Percentage of the requests served within a certain time (ms)  
      50%    365  
      66%    409  
      75%    430  
      80%    444  
      90%    614  
      95%    651  
      98%    674  
      99%    678  
     100%    680 (longest request)  

  3 [日本語ローカルサイト](https://ja.wordpress.org/)のプラグインから[WP Super Cache](https://downloads.wordpress.org/plugin/wp-super-cache.1.4.4.zip)をダウンロードする  
  4 wordpressにログインする  
  5 プラグインをクリック  
   新規追加をクリック  
  6 プラグインのアップロードをクリック  
  7 ダウンロードしたプラグインを選択する  
  8 インストールして適用する  
  9 もう一度abコマンドを使って性能を確かめる 

       $ab -k -c 1000 -n 1000 192.168.xxx.xxx/wordpress  
       Server Software:        nginx/1.9.0  
       Server Hostname:        192.168.33.129  
       Server Port:            80  

      Document Path:          /wordpress  
      Document Length:        184 bytes  

      Concurrency Level:      1000  
      Time taken for tests:   1.010 seconds  
      Complete requests:      1000  
      Failed requests:        0  
      Non-2xx responses:      1000  
      Keep-Alive requests:    998  
      Total transferred:      389990 bytes  
      HTML transferred:       184000 bytes  
      Requests per second:    990.07 [#/sec] (mean)  
      Time per request:       1010.030 [ms] (mean)  
      Time per request:       1.010 [ms] (mean, across all concurrent requests)  
      Transfer rate:          377.07 [Kbytes/sec] received  

      Connection Times (ms)  
                  min  mean[+/-sd] median   max  
      Connect:        0   15  15.5      0      37  
      Processing:     0  178 230.9     13     954  
      Waiting:        0  178 230.9     13     954  
      Total:          0  193 242.9     13     989  
 

      Percentage of the requests served within a certain time (ms)  
      50%     13  
      66%    271  
      75%    308  
      80%    503  
      90%    529  
      95%    538  
      98%    981  
      99%    987  
     100%    989 (longest request)  

   10 abコマンドを実行した結果の最終行を何も入れてない状態のとプラグイン入れたあとを比較する  
   プラグイン入れたほうが数字が大きかったら速くなってる  

## 2-5 Vagrant設定  
  1 Boxの追加  
   `$vagrant box add CentOS65 コピーしたboxファイル --force`  
  2 Vagrantの初期設定  
   `$vagrant init CentOS65`
  3 ホストオンリーアダプターの設定  
    `$vi Vagrantfile`を開いて  

    Vagrant.configure(2) do |config|    
    config.vm.network :private_network, ip:"192.168.33.120"
    end

  4 Vagrantの起動  
   `$vagrant up`  
  5 Vagrantへ接続  
   `$vagrant ssh`  

## 2-5 centosプロキシの設定
  1 プロキシを設定する  
     $vi /etc/profile ファイルに下記を追記する  

     PROXY='172.16.40.1:8888'  
     export http_proxy=$PROXY  
     export HTTP_PROXY=$PROXY  
     export HTTPS_PROXY=$PROXY  
     export https_proxy=$PROXY  

  2 yumにプロキシを設定  
   `$vi /etc/yum.conf`に下記を追記  
   `proxy=http://172.16.40.1:8888/`  
  3 wgetにプロキシを設定  
   `$vi /etc/wgetrcに下記を追記`  

    http_proxy = 172.16.40.1:8888/  
    https_proxy = 172.16.40.1:8888/  
    ftp_proxy = 172.16.40.1:8888/  

  4 プロキシの設定が終わったのでupdateする  
   `$yum update`  

## 2-5 セキュリティチェック
  1 [Tenable Network Security社](http://www.tenable.com/)の[nessus](http://downloads.nessus.org/nessus3dl.php?file=Nessus-6.3.6-es7.x86_64.rpm&licence_accept=yes&t=0ebe68870bb9459536c353a9107b53b0)をダウンロードする  
  2 ダウロードしたNessus-6.3.6-es7.x86_64.rpmを起動中のvagrantのフォルダ(Vagrantfileがあるとこ)に移動する  
   `$mv ~/Download/Nessus-6.3.6-es7.x86_64.rpm vagrantfileがあるフォルダ`  
  3 vagrantにログインする  
   `$vagrant ssh`  
  4 nessusをインストール  

    $cd /vagrant  
    $rpm -ivh Nessus-6.3.6-es7.x86_64.rpm  
    $yum -y install nessus  

  5 オフラインでライセンス取得する  
   `$sudo /opt/nessus/sbin/nessuscli fetch --challenge`  
   下記のような分が表示されるのコードをメモしとく  
   `Challenge code: 7235205e82b3f8767273f8ebace64da802ea17ed`  
  6 [Nessus HomeFeed](http://www.tenable.com/products/nessus-home)で登録して無料ライセンスを習得する
   登録したメールアドレスにNessus Registrationからメールが来る  
   下記のような分がある  
   `Your activation code for the Nessus Home is`  
   `7187-AC57-944B-9F71-A6CB`  
  7 [オフラインでの設定が書いてるサイト](https://plugins.nessus.org/offline.php)にアクセスして
   上の四角の中には5でメモしたコードを貼り付ける  
   下の四角の中にはメールで送られてきたactivation codeを貼り付ける  
  8 [http://plugins.nessus.org/get.php?f=all-2.0.tar.gz&u=333822f86bd6835de6d55cd5b21955e5&p=5cdf937155b5bd62467660d5beed8778](http://plugins.nessus.org/get.php?f=all-2.0.tar.gz&u=333822f86bd6835de6d55cd5b21955e5&p=5cdf937155b5bd62467660d5beed8778)をクリックしてダウンロードする  
  9 サイトの下に書いてある[nessus-fetch.rc](https://plugins.nessus.org/mkconfig.php?ac=7187-AC57-944B-9F71-A6CB&c=7235205e82b3f8767273f8ebace64da802ea17ed)をクリックしてダウンロードする  
  10 2でやったようにダウンロードしたファイルを移動する  
  11 vagrantにsshでログインする  
  12 ダウンロードした`nessus-fetch.rc`を`/opt/nessus/etc/`に移動する  
   `$mv /vagran/nessus-fetch.rc /opt/nessus/etc/`  
  13 nessus-fetch.rcファイルを開いてプロキシ設定をする  
   `$ vi /opt/nessus/etc/nessus-fetch.rc`  
   下記のように  

    proxy=172.16.40.1  
    proxy_port=8888  

  14 nessusの起動  
   `$systemctl start nessus`  
  15 nessusのインストール  
   `$/opt/nessus/sbin/nessuscli update --plugins-only /vagrant/all-2.0.tar.gz`   
  16 ブラウザからアクセス  
   `192.168.33.120:8834`でアクセス  
  17 あとは指示通り登録していく  


# lesson
0: [講義の前のセットアップ](section0.md)  
1: [基本のサーバー構築](section1.md)  
2: [その他のWebサーバー環境](section2.md)  
3: [Ansibleによる自動化とテスト](section3.md)  
4: [PHP以外の環境を構築する](section4.md)  
5: [DNSサーバーを動作させる](section5.md)  
6: [AWS](section6.md)  

