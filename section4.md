# section4

## 開発ツールのインストール
   `$yum -y groupinstall "Development Tools"`
## RubyとPassengerのビルドに必要なヘッダファイルなどのインストール
   `$yum -y install openssl-devel readline-devel zlib-devel curl-devel libyaml-devel libffi-devel`
## MariaDBのインストール
   `$yum -y install mariadb-server mariadb-devel`
## Apacheのインストール
   `$yum -y install httpd httpd-devel`
## ImageMagickとヘッダファイル・日本語フォントのインストール
   `$yum -y install ImageMagick ImageMagick-devel ipa-pgothic-fonts`
## Rubyのインストール
   `$curl -O http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz`
## Rubyのビルド

    tar zxvf ruby-2.2.2.tar.gz
    cd ruby-2.2.2
    ./configure --disable-install-doc
    make
    make install
    cd

## bundlerのインストール
   `$gem install bundler --no-rdoc --no-ri`
## MariaDBの起動および自動起動の設定
   `$service mariadb start`
   `$systemctl enable mariadb`

## mysqlのrootのパスワードを変更

    $mysql -u root
    MariaDB [(none)]> set password for root@localhost=password('vagrant');
    MariaDB [(none)]>exit

## Redmine用データベースとユーザーの作成

    $mysql -u root -p
    MariaDB [(none)]> create database db_redmine default character set utf8;
    MariaDB [(none)]>grant all on db_redmine.* to user_redmine@localhost identified by '********';
    MariaDB [(none)]> exit;

## Redmineのインストール
   `$curl -O http://www.redmine.org/releases/redmine-3.0.3.tar.gz`
## Redmineの展開と配置

    $tar xvf redmine-3.0.0.tar.gz
    $mv redmine-3.0.0 /var/lib/redmine

## データベースへの接続設定
   `$vi /var/lib/redmine/config/database.yml`

    production:
      adapter: mysql2
      database: db_redmine
      host: localhost
      username: user_redmine
      password: "********"
      encoding: utf8

## 設定ファイル config/configuration.yml の作成
   `$vi /var/lib/redmine/config/configuration.yml `

    production:
      email_delivery:
        delivery_method: :smtp
        smtp_settings:
          address: "localhost"
          port: 25
          domain: "example.com"
      rmagick_font_path: /usr/share/fonts/ipa-pgothic/ipagp.ttf

## Gemパッケージのインストール
   `$bundle install --without development test --path vendor/bundle`

## セキュリティトークン作成
   `$bundle exec rake generate_secret_token`

## RAILS_ENV=production bundle exec rake db:migrate
   `$RAILS_ENV=production REDMINE_LANG=ja bundle exec rake redmine:load_default_data`

## Passengerのインストール
   `$gem install passenger --no-rdoc --no-ri`

## PassengerのApache用モジュールのインストール

    $passenger-install-apache2-module
    を実行して成功したら
    LoadModule passenger_module /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.2/buildout/apache2/mod_passenger.so
    <IfModule mod_passenger.c>
    PassengerRoot /usr/local/lib/ruby/gems/2.2.0/gems/passenger-5.0.2
    PassengerDefaultRuby /usr/local/bin/ruby
    </IfModule>
    上記のような内容が表示されるので
    /etc/httpd/conf.d/redmine.confファイルを作成して貼り付ける

## RedmineのCSSや画像へのアクセスを許可

    $vi etc/httpd/conf.d/redmine.conf
    に下記を追加
    <Directory "/var/lib/redmine/public">
    Require all granted
    </Directory>

## Apacheの起動および自動起動の設定
   `$service httpd start`
   `$systemctl enable httpd`

## Apache上のPassengerでRedmineを実行するための設定
   `$chown -R apache:apache /var/lib/redmine`

## webサーバをRedmine専用として使用

    $vi /etc/httpd/conf/httpd.conf
    を開いて下記のように変更
    DocumentRoot "/var/www/html"
    ↓
    DocumentRoot "/var/lib/redmine/public"
    設定後
    apacheを再起動する


# lesson
0: [講義の前のセットアップ](section0.md)  
1: [基本のサーバー構築](section1.md)  
2: [その他のWebサーバー環境](section2.md)  
3: [Ansibleによる自動化とテスト](section3.md)  
4: [PHP以外の環境を構築する](section4.md)  
5: [DNSサーバーを動作させる](section5.md)  
6: [AWS](section6.md)  
[section3_ansible](section3_ansible)  
[section6_ansible](section6_ansible)  
