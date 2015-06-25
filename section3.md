# section 3-1

## Ansibleのインストール
     sudo /usr/local/apache2/bin/apachectl start
      sudo apt-get install software-properties-common
      sudo apt-add-repository ppa:ansible/ansible
      sudo apt-get update
      sudo apt-get install ansible
### vagrantのプラグインのインストール
    $vagrant plugin install vagrant-proxyconf
### Vagrantfileの作成
   `$vagrant init Centos7`
### ホストオンリーアダプターの設定
    $vi Vagrantfileを開いて
    Vagrant.configure(2) do |config| end
    の間に`config.vm.network :private_network, ip:"192.168.33.124"を追加
### proxyの設定

    if Vagrant.has_plugin?("vagrant-proxyconf")
         config.proxy.http     = ENV["HTTP_PROXY"]
         config.proxy.https    = ENV["HTTP_PROXY"]
         config.proxy.no_proxy = ENV["NO_PROXY"]
    end

### adminでphp,mariadb,nginxのインストールとnginx,mariadb,phpの起動

    - hosts: all
      user: vagrant
      sudo: yes
      tasks:
        - name: php install
          yum: name={{ item  }} state=installed
          with_items:
            - php
            - php-fpm
            - php-gd
            - php-mysql
        - name: mariadb install
          yum: name={{ item }} state=installed
          with_items:
            - mariadb
            - mariadb-server
            - MySQL-python
        - name: nginx
          yum: name=http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm state=present
        - name: nginx install
          yum: name=nginx state=installed
        - name: nginx.conf
          template:
               src: nginx/nginx.conf.j2
               dest: /etc/nginx/nginx.conf
       - name: default.conf
          template:
               src: nginx/default.conf.j2
               dest: /etc/nginx/conf.d/default.conf
          notify:
             - restart nginx
        - name: be sure nginx is running and enabled
           service: name=nginx state=running enabled=yes
           tags: nginx
        - name: php-fpm
           service: name=php-fpm state=running enabled=yes
          tags: php-fpm
        - name: be sure mysqld is running and enabled
           service: name=mariadb state=running enabled=yes
          tags: mariadb

### mysql rootのパスワード設定とデータベースの作成、ユーザーの作成

     - hosts: all
       user: vagrant
       sudo: yes
       vars:
        mysql_root_password: 'vagrant'
        mysql_db_password: "wordpress"
       tasks:
         - name: root アカウントのパスワードを設定
           mysql_user:
                   user: root
                   host: localhost
                   password: "{{ mysql_root_password }}"
         - name: create database
           mysql_db:
               name: wordpress
               state: present
               login_user: root
               login_password: vagrant
         - name: create user
          mysql_user:
                 login_user: root
                 login_password: vagrant
                 name: wordpress
                 password: "{{mysql_db_password}}"
                 priv: "wordpress.*:ALL"
                 host: localhost
                 state: present

###wordpressダウンロード

    - hosts: all
      user: vagrant
      sudo: yes
      vars:
       proxy_env:
           http_proxy: http://172.16.40.1:8888
      tasks:
        - name: wordpress download
          get_url: url=https://ja.wordpress.org/wordpress-4.2.2-ja.zip dest=/tmp use_proxy=yes
              #shell: wget "https://ja.wordpress.org/wordpress-4.2.2-ja.zip"
        - name: unzip insall
          yum: name=unzip state=installed

        - name: unzip wordpress
          shell: unzip wordpress-4.2.2-ja.zip chdir=/tmp

        - name: wordpress move
          shell: mv /tmp/wordpress /usr/share/nginx/html

        - name: wp-config.php
          template:
               src: wp-config.php.j2
               dest: /usr/share/nginx/html/wordpress/wp-config.php

### ユーザー名とパスワードを`/etc/ansible/hosts`に書き込む
    $vi /etc/ansible/hosts

    192.168.33.132 ansible_ssh_user=vagrant ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
### hostsファイルの作成(Vagrantfileがあるディレクトリに！)
    vi hosts

    [all]
    192.168.33.132
###  ansibleの実行の仕方

   `$ansible-playbook 実行したいファイル名 -i hosts -k`


# 3-2

### Vagrantfileに書きを追加

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/Vagrant.yml"
    end

### [section3_ansible](section3_ansible)




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