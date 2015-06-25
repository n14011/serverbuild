# 6-1 AWS EC2 + Ansible
## pipをインストール
   `$sudo apt-get install python-pip`
## awscliをインストール
   `$sudo pip install awscli`  
## awscliの設定

    $ aws configure  
    AWS Access Key ID [None]: 先生からもらったID  
    AWS Secret Access Key [None]: 先生からもらったKey  
    Default region name [None]: ap-northeast-1  
    Default output format [None]: json  

    これでawsコマンドが使える  

## EC2を使用する  

    1 Amazon web servicesにログインする  
    2 EC2のアイコンをクリックしてEC2のダッシュ-ボードにアクセス  
    3 インスタンスの作成をクリック  
    4 Amazon linuxを選択  
    5 表の一番上の`t2.micro`を選択して右下にある確認と作成をクリック  
    6 作成をクリック
    7 新しいキーペアを作成する※ダウンロードしたキーファイルはsshで使うので消さないように！
    8 インスタンスの作成をクリック
    9 動いているマシンの一覧が出てくるから自分のを選択して一番右にあるセキュリティグループをクリックする
    10 グループを選択して右クリックしてインバウンドルールの編集を選択してルールを追加をクリックしてHTTPを追加する
    11 ansibelを実行する※やり方は下の方に書いてる
    12 ブラウザからアクセス(http://52.68.182.11)

## SSH接続  
   さっき作っさキーファイルのアクセス権限を変更  
   `$chmod 400 n14002.pem`  
   ssh接続  
   `$ssh -i キーファイル ec2-user@52.68.182.11`  

## ansibleを実行するための準備  
   hostsファイルの作成  
   `$vi hosts`  

    [all]  
    52.68.182.11(sshした時のIPアドレス)

## ansible実行
`$ansible-playbook 実行したいファイル -i hosts  --private-key ~/.aws/n14011.pem`

   [section6_ansible](section6_ansible)

# 6-2 AWS EC2(AMIMOTO)

    1 Amazon web servicesにログインする  
    2 EC2のアイコンをクリックしてEC2のダッシューボードにアクセス  
    3 インスタンスの作成をクリック  
    4 左のメニューバーから`AWS Marketplace`を選択して`AMIMOTO`を検索する  
    5 `WordPress powered by AMIMOTO (HHVM)`を選択  
    6 表の一番上の`t2.micro`を選択して右下にある確認と作成をクリック  
    7 セキュリティグループのところでエラーが出るので編集を選択  
    8 セキュリティグループを6-1で作ったものに変える  
    9 6−1で作ったキー名を選択してインスタンスの作成をクリック  
    10 ブラウザからアクセス(http://52.68.235.54)

# 6-3 Route53

    1 Amazon web servicesにログインする  
    2 Route53のアイコンをクリックしてアクセス  
    3 左のメニューバーから`Hosted Zones`をクリック  
    4 `Create Hosted Zone`をクリック  
    5 ドメイン名に自分の学籍番号を書いてCreateボタンをクリック  
    6 自分の学籍番号を選択して`Go to Record Sets`をクリック  
    7 `Import Zone File`をクリック
    8 section5で作ったファイルを追加する

# 6-4 S3

    1 ubuntuの端末で`$ aws s3 mb s3://n14011`を実行したらS3のページにn14011というディレクトリができてる  
    2   `$aws s3 ls`で確かめれる  
    3 `$aws s3 cp index.html s3://n14011/index.html`でファイルをアップロードできる  
    4 Amazon web servicesにログインする  
    5 S3のアイコンをクリックしてS3にアクセス  
    6 自分が作ったファイルまで移動して右クリックで公開をクリックして公開する
    7 公開したら右上にあるプロパティをクリック
    8 リンクをクリックしたらブラウザでアクセスできる

    aws s3 コマンド一覧  
    aws s3 ls ・・・ファイルを表示  
    aws s3 mb ・・・ディレクトリを作成  
    aws s3 rb ・・・ディレクトリの削除  
    aws s3 rm ・・・ファイルの削除  
    aws s3 cp ・・・フィルの転送またはファイルの取得  
    ファイルの転送`$aws s3 cp 転送したいファイル名 s3://転送したいフォルダ  
    ファイルの取得`$aws s3 cp s3://取得したいフィアルのパス  


# 6-5 CloudFront

    1 6-1で作ったAMIを起動しとく  
    2 AWSのマネジメントコンソールからCloudFrontのアイコンをクリックしてCloudFrontにアクセス  
    3 Create Distributionをクリック  
    4 WebのGet Startedをクリック  
    5 Origin Domain Name にEC2で動かしてるマシンのパブリック DNS: `ec2-52-69-73-207.ap-northeast-1.compute.amazonaws.com`に書く
    6 Distribution SettingsのPrice Classを`Use Only US, Europe and Asia`にする
    7 Create Distributionをクリックする
    8 自分が作ったもののStatusが`Deployed`になるまで待つ
    9 ベンチマークを取る
    10 EC2の場合は`ipアドレス/wordpress`にabコマンドを実行する
    11 CloudFrontの場合は`Domain Name`にabコマンドを実行する

     EC2にやった場合  
     ab -k -c 100 -n 100 http://52.69.101.97/wordpress/
     This is ApacheBench, Version 2.3 <$Revision: 655654 $>
     Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
     Licensed to The Apache Software Foundation, http://www.apache.org/
     Benchmarking 52.69.87.230 (be patient).....done
     Server Software:        nginx/1.6.2
     Server Hostname:        52.69.87.230
     Server Port:            80
     Document Path:          /wordpress/
     Document Length:        263 bytes
     Concurrency Level:      100
     Time taken for tests:   3.907 seconds
     Complete requests:      100
     Failed requests:        0
     Write errors:           0
     Non-2xx responses:      100
     Keep-Alive requests:    0
     Total transferred:      55500 bytes
     HTML transferred:       26300 bytes
     Requests per second:    25.60 [#/sec] (mean) 
     Time per request:       3906.814 [ms] (mean)
     Time per request:       39.068 [ms] (mean, across all concurrent requests)
     Transfer rate:          13.87 [Kbytes/sec] received
     Connection Times (ms)
                   min  mean[+/-sd] median   max
     Connect:      213  217   1.5    217     219
     Processing:  1408 2576 880.4   2744    3687
     Waiting:     1408 2576 880.4   2744    3687
     Total:       1623 2793 881.8   2961    3906
     Percentage of the requests served within a certain time (ms)
       50%   2961
       66%   3064
       75%   3862
       80%   3888
       90%   3897
       95%   3904
       98%   3904.9
       99%   3906
      100%   3906 (longest request)


    Cloudfrontの場合
    ab -k -c 100 -n 100 http://d5znogp9gkype.cloudfront.net/wordpress/
    This is ApacheBench, Version 2.3 <$Revision: 655654 $>
    Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
    Licensed to The Apache Software Foundation, http://www.apache.org/
    Benchmarking d5znogp9gkype.cloudfront.net (be patient).....done
    Server Software:        nginx/1.6.2
    Server Hostname:        d5znogp9gkype.cloudfront.net
    Server Port:            80
    Document Path:          /wordpress/
    Document Length:        8595 bytes
    Concurrency Level:      100
    Time taken for tests:   0.267 seconds
    Complete requests:      100
    Failed requests:        0
    Write errors:           0
    Keep-Alive requests:    0
    Total transferred:      897500 bytes
    HTML transferred:       859500 bytes
    Requests per second:    374.19 [#/sec] (mean)  ←ここの数字がEC2の場合より大きくなってたらいい
    Time per request:       267.246 [ms] (mean)
    Time per request:       2.672 [ms] (mean, across all concurrent requests)
    Transfer rate:          3279.62 [Kbytes/sec] received
    Connection Times (ms)
                  min  mean[+/-sd] median   max
    Connect:       11   13   1.2     12      14
    Processing:    12   33  55.2     17     252
    Waiting:       11   17   5.1     16      33
    Total:         22   45  55.7     29     266
    Percentage of the requests served within a certain time (ms)
      50%     29
      66%     37
      75%     39
      80%     41
      90%     48
      95%    258
      98%    265
      99%    266
     100%    266 (longest request)




# 6-6 RDS

    0 6-1で作ったAMIを動かしとく
      MySQLのポートも開けておく
    1 Amazon web servicesにログインする  
    2 Route53のアイコンをクリックしてアクセス  
    3 MySQLを選択
    4 いいえを選んで次のステップをクリック
    5 DB詳細の設定
      DB インスタンスのクラス: db.t2.micro
      マルチ AZ 配置: いいえ
      DB インスタンス識別子: n14011
      マスターユーザの名前: n14011
      マスターパスワード:n14011mysql
      パスワードの確認:n14011mysql
      を記入して次のステップをクリック
    6 [詳細設定] の設定
      VPC:EC2で動かしてるマシンと一緒のにする
      VPC セキュリティグループ:EC2で動かしてるマシンと一緒のにする
      データベースの名前:wordpress
      を書いたらDBインタンスの作成をクリック
    7 EC2で動いているマシンにSSHでアクセスする
    8 RDSのダッシュボードにある自分のRDSをクリックしてエンドポイント:n14011.cufhnhpbz2zj.ap-northeast-1.rds.amazonaws.com:3306をコピーしてくる
    9 MySQLに接続する
      $mysql –h n14011.cufhnhpbz2zj.ap-northeast-1.rds.amazonaws.com:3306 -P 3306 –u n14011 –p wordpress
    10 ユーザーの作成
       grant all on wordpress.* to 'n14011'@'%' identifed by 'password';
    11 Wordpressフォルダのwp-config.phpを編集する

       $vi wp-config.php  
       WordPress のためのデータベース名   
       define('DB_NAME', 'mysqlで作ったデータベース名');  
       ↓  
       define('DB_NAME', 'RDSで作ったデータベース名');  

       MySQL データベースのユーザー名  
       define('DB_USER', 'username_here');  
       ↓  
       define('DB_USER', 'RDSで作ったユーザー名');  

       MySQL データベースのパスワード  
       define('DB_PASSWORD', 'password_here');  
       ↓  
       define('DB_PASSWORD', 'RDSで作ったデータベースの所有者名のパスワード');

       MySQL のホスト名
       define('DB_HOST', 'localhost');
       ↓
       define('DB_HOST', 'n14011.cufhnhpbz2zj.ap-northeast-1.rds.amazonaws.com:3306(
       	RDSのエンドポイント)');
    12 ブラウザからアクセスしてWordpressを表示する



# 6-7 ELB

    0 Amazon web servicesにログインする
    1 EC2のアイコンをクリックしてアクセス  
    2 EC2で4代くらいマシンを動かしとく  
    3 左側のメニューからロードバランサーをクリック
    4 ロードバランサーの作成をクリック
      ロードバランサー名:n14011
      内部向け LB の作成:EC2で動いてるマシンと同じにする
      次の手順をクリック
    5 セキュリティグループも動いてるマシンと同じにする
    6 ステップ 5まで移動
      自分のマシンをすべて選択する
    7 ステップ7まで移動
      作成をクリック
    8 すべてのマシンにSSHで接続
    9 ログを表示する
      $cd /var/log/nginx/
      $tail -f accecs.log
    10 ロードバランサーのDNS名をコピーする
    11 分散処理されてるか確認
       ubuntuの端末から$ab -k -c 100 -n 100 n14011-842552940.ap-northeast-1.elb.amazonaws.com
    12 端末で表示してるログが一緒に動いていたらOK


# 6-8 API叩いてみよう






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



