# 5-1 bindのインストール
## 1 clonekoからserverbuilding-2015をgit cloneしてくる  
   `$git clone https://github.com/cloneko/serverbuilding-2015`  
## 2 Vagrantfileがあるフォルダまで移動  
   `$cd /serverbuilding-2015/Section5/`  
## 3 vagrantを立ち上げる  
   `$vagrant up`  

# 5-2 bindの設定
## 1 自分のゾーンを作成  
   Vagrantfileと同じ場所にあるansible/zone.cloneko.comを編集する  
   `$vi ansible/zone.cloneko.com`  
   clonekoをすべてn14011に変える  
## 2 slaveファイルを編集

     zone "cloneko.com" {
     type slave;
         masters { 192.168.33.14; };   
     file "slave/zone.cloneko.com";
     };
     ↓に変更
     zone "n14011.com" {
     type slave;
         masters { 192.168.33.14; };   
     file "slave/zone.cloneko.com";
     };

## 3 masterファイルを編集  

     zone "cloneko.com" {
     type master;
     file "zone.cloneko.com";
     };
     ↓に変更
     zone "n14011.com" {
     type master;
     file "zone.cloneko.com";
     };

## 4 ubuntuからDNSに問い合わせる
   `dig @192.168.33.14 ns.n14011.com`

## 5 成功したら下記のように表示される  

     ; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> @192.168.33.14 ns.n14011.com  
     ; (1 server found)  
     ;; global options: +cmd  
     ;; Got answer:  
     ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11562  
     ;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 2  
     ;; WARNING: recursion requested but not available  
    
     ;; OPT PSEUDOSECTION:  
     ; EDNS: version: 0, flags:; udp: 4096  
     ;; QUESTION SECTION:   
     ;ns.n14011.com.			IN	A  
     
     ;; ANSWER SECTION:  
     ns.n14011.com.		60	IN	A	172.16.40.70  
    
     ;; AUTHORITY SECTION:  
     n14011.com.		60	IN	NS	ns.n14011.com.  
     n14011.com.		60	IN	NS	ns2.n14011.com.  
    
     ;; ADDITIONAL SECTION:  
     ns2.n14011.com.		60	IN	A	172.16.40.71  
    
     ;; Query time: 1 msec  
     ;; SERVER: 192.168.33.14#53(192.168.33.14)  
     ;; WHEN: Thu Jun 04 14:29:07 JST 2015  
     ;; MSG SIZE  rcvd: 106  


# lesson
0: [講義の前のセットアップ](section0.md)  
1: [基本のサーバー構築](section1.md)  
2: [その他のWebサーバー環境](section2.md)  
3: [Ansibleによる自動化とテスト](section3.md)  
4: [PHP以外の環境を構築する](section4.md)  
5: [DNSサーバーを動作させる](section5.md)  
6: [AWS](section6.md)  
