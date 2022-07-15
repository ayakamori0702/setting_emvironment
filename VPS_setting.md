# VPSでサーバー構築してみた
***
### 環境詳細  
サーバー名　ayakamori  
IPアドレス 133.242.133.61  
確認方法  
`$ hostname -I`
プラン　512M・SSD 25GB  
OS Ubuntu 20.04 amd64  
***
**1.Ubuntuへのログイン**  
ログイン名：ubuntu  
パスワード：初期設定のもの  

デフォルトでrootユーザーのパスワードを設定  
`$ sudo su -`  
Ubuntu の最新のパッケージ情報を再取得  
`$ sudo apt update`  
パッケージを最新に更新、不要なパッケージの削除、カーネルの更新  
`$ sudo apt dist-upgrade`  
***
**2. ユーザーの追加**   
`$ sudo useradd ayaka`→のちのちダメ！！  
`$ sudo adduser ayaka`  
確認   
`$ sudo passwd ayaka`  
***
**3. SSHの設定を変更**  
`# vi /etc/ssh/sshd_config`  
```
sshdがリッスンするポート番号の変更  
#Port 22 → Port 10022  
ポート番号22は攻撃されやすいので、別番号に設定(VPSのグローバルネットワーク設定で、接続可能ポートを追加)  
rootでのログインを禁止  
#PermitRootLogin prohibit-password → PermitRootLogin no  
パスワード認証を無効化  
#PasswordAuthentication yes → PasswordAuthentication yes  
```
再起動  
`$ systemctl restart sshd.service`
***
**3. sudoユーザーの管理**  
sudoユーザーの追加  
`$ sudo gpasswd -a ayaka sudo`  
確認  
`$ id ayaka`  
sudoのパスワードを入れたくない場合、/etc/sudoers　を編集する  
`$ sudo visudo`  
```
#User privilege specification  
root         ALL=(ALL:ALL) ALL
ayaka        ALL=(ALL:ALL) NOPASSWD:ALL
```
ユーザー名等を追記する  

ログインしてみる  
` $ SSH -p 10022 ayaka@133.242.133.61`  
ログインはできたが、ユーザディレクトリがない。。。     
```
Could not chdir to home directory /home/ayaka: No such file or directory
```  

「useradd」コマンドではホームディレクトリが作成されないので、   
Ubuntuではユーザー追加時は「adduser」コマンドを使うと  
パスワード発行からホームディレクトリ作製まで対話式に進むので、    
そのコマンドを利用してユーザーを作成する  
[参考資料](https://ex1.m-yabe.com/archives/3259)  

ユーザーを消去して作成し直す  
`$ sudo -r ayaka`
確認  
`$ id ayaka`  
id: ‘ayaka’: no such user
作成  
`$ sudo adduser ayaka` 
確認  
`$ id ayaka`  
uid=1001(ayaka) gid=1001(ayaka) groups=1001(ayaka),27(sudo)

***
**4. OSの自動アップデート**  
Ubuntuは自動でやってくれる！！  
確認だけしておく  
`$ cat /etc/apt/apt.conf.d/20auto-upgrades`
```
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Unattended-Upgrade "1";
```

"1"を"0"にすると無効になるらしい。  
[参考資料](https://linux.just4fun.biz/?Ubuntu/自動アップデートを有効・無効にする手順)  
***
**5. ソフトウェアのインストール**  
[参考資料](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04-ja)
パッケージリスト更新  
`$ sudo apt update`  
インストール  
`$ sudo apt install apache2`  

Apacheをテストする前に、ファイアウォール設定を変更して、外部からデフォルトWebポートへのアクセスを許可する必要がある。  
`$ sudo ufw app list `

```
  Available applications:
  Apache
  Apache Full
  Apache Secure
  OpenSSH
```
ポート80 通常の暗号化されていないWebトラフィック  
ポート443 TLS/SSL暗号化トラフィック
Apache: このプロファイルは、ポート80のみを開きます。  
Apache Full: このプロファイルは、ポート80とポート443の両方を開きます。  
Apache Secure: このプロファイルは、ポート443のみを開きます。  

ufWを使用してファイウォールの設定を行う  
`$ sudo ufw allow 'Apache' `  
```
Rules updated  
Rules updated (v6)  
```
`$ sudo ufw reload`  
確認  
`$ sudo ufw status`  
```
Status: inactive  
```
最初はufwの設定が無効になっている  

[参考資料](https://qiita.com/shimakaze_soft/items/c3cce2bfb7d584e1fbce)  
ufw（Uncomplicated FireWall）とは、Linuxの「Netfilter」によるファイアウォールを管理して操作するためのiptablesをラッパーした機能のことです。  
有効化する  
`$ sudo ufw enable`  
```Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup  
```
確認  
`$ sudo ufw status`  
```
Status: active

To                         Action      From
--                         ------      ----
Apache                     ALLOW       Anywhere
Apache (v6)                ALLOW       Anywhere (v6)  
``` 

systemdでのサービスの稼働状態を確認  
`$ sudo systemctl status apache2`
```
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-07-04 10:43:46 JST; 11min ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 2021 (apache2)
      Tasks: 55 (limit: 462)
     Memory: 5.5M
     CGroup: /system.slice/apache2.service
             ├─2021 /usr/sbin/apache2 -k start
             ├─2026 /usr/sbin/apache2 -k start
             └─2027 /usr/sbin/apache2 -k start

Jul 04 10:43:46 ik1-103-58307 systemd[1]: Starting The Apache HTTP Server...
Jul 04 10:43:46 ik1-103-58307 systemd[1]: Started The Apache HTTP Server.
```
正常に稼働している。  

テストしてみる。  
Webブラウザで出なかった・・・。  
`http://133.242.133.61/`  

しかもSSHログインができなかった　
`$ SSH -p 10022 ubuntu@133.242.133.61`
``` 
ssh: connect to host 133.242.133.61 port 10022: Operation timed out  
```  
→Part2へ  


HTMLページを編集して、HPに載せる  
`$ sudo vi /var/www/html/htmltest.html`  
[アクセス](http://133.242.133.61/htmltest.html)   

***
**SSHログインができないときPart1**  
ローカル：
`$ ssh ubuntu@133.242.133.61`  
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:k0F56ZJuQTl6FUDqMcDYcYGP6PPj1CmT5tq8IlTDqsM.
Please contact your system administrator.
Add correct host key in /Users/nakamuraayaka/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/nakamuraayaka/.ssh/known_hosts:7
Host key for 133.242.133.61 has changed and you have requested strict checking.
Host key verification failed.
```
インスタンスを変更したりしてサーバーが別のものになっていても、  
ローカルはもとの情報を保持したままなので、エラーが出てしまう。  
ローカル:  
`$ ssh-keygen -R 133.242.133.61`  

再度SSHで接続を試みる  
ローカル：
`$ ssh ubuntu@133.242.133.61`  
OK  
***
**SSHログインができないときPart2**  

port 10022: Operation timed out  
ポートが開放されていない  
[参考URL](https://ccsr.aori.u-tokyo.ac.jp/~obase/unixtips.html)    

`$ sudo ufw allow 10022/top`
OK  
***







**基本的なサーバー管理**  
・システム負荷の確認    
`$ uptime`  
3つの数値がコア数以下ならOK  
・ディスクの使用状況  
`$ df -h`  
tmpfsは仮想的なファイルシステムなので気にする必要なし。  
・メモリとスワップの使用状況  
`$ free -h`  
スワップが使用されていると、メモリ不足  
・実行中プロセスの確認
`$ ps aux`