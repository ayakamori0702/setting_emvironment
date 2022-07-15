### WLSにリモートから接続する  
  
***1.ubuntuのセットアップ***  
`$ sudo apt update`  
`$ sudo apt upgrade -y`  

日本語言語パックのインストール  
`$ sudo apt install -y language-pack-ja`  
ロケールの設定  
`$ sudo update-locale LANG=ja_JP.UTF-8`  
タイムゾーンの設定    
`$ sudo dpkg-reconfigure tzdata`  
日本語のmanpageをインストール
`$ sudo apt install -y manpages-ja manpages-ja-dev`  

***2.WLSにホストキー生成***  
[参考URL](https://mulberrytassel.com/wsl-ssh1/)  
`$ sudo ssh-keygen -A`  
/etc/sshに生成されていればOK  

SSHを再起動
`$ sudo service ssh restart`  

自身のIPアドレスを取得  
`$ ip address show`  
または  
`&ip address show eth0 | grep 'inet ' | awk '{split($2,ipa,"/");print ipa[1]}'`  
```
eth0:  
inet 172.27.25.245/20
```

↑ここまでUbuntuでの作業  
***
↓ここからWLSを搭載するwindows power shellでの操作  
`ipconfig`
自身のIPアドレスを参照する  
```
IPv4  192.168.1.37
```
***2.ポートフォワーディングの設定***  
WSLの仮想ネットワークは、そのままでは外部のパソコンからアクセスできない。  
そこで、WSLが稼働しているWindowsのIPアドレスのポート番号と、WSLのIPアドレスのポート番号を紐付ける必要があります。
ポートフォワーディングをすると、外部のパソコンからWSLが稼働しているWindowsパソコンのIPアドレスを使って、WSLに接続することができるようになる。  

 `$ netsh interface portproxy add v4tov4 listenaddress=<192.168.1.37> listenport=22 connectaddress=<172.27.25.245> connectport=22`

ポートフォワーディングの確認  
`$ netsh interface portproxy show v4tov4`
```
ipv4 をリッスンする:         ipv4 に接続する:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
<192.168.1.37>  22          <172.27.25.245> 22
```  
***ファイアウォールの設定***  
22番のポートを開放する  
`$ New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort 22 -Action Allow -Protocol TCP`  
`$ New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort 22 -Action Allow -Protocol TCP`  

ファイアウォールの確認  
`$ Get-NetFirewallRule -DisplayName 'WSL 2 Firewall Unlock' | Get-NetFirewallPortFilter | Format-Table`  

```
Protocol LocalPort RemotePort IcmpType DynamicTarget
-------- --------- ---------- -------- -------------
TCP      22        Any        Any      Any
TCP      22        Any        Any      Any
```  
***外部からSSH接続する***  
`$ ssh <ユーザ名>@<WindowsのIPアドレス>`