## Dockerコンテナを用いてAnaconda環境下のJupyter notebookにアクセスする　　

### <まずは　docker image　を　Docker Hub　から　pullしてくる　>  

参照：  
[Docker Hub](https://hub.docker.com/r/continuumio/anaconda3)  
[docker imagesのpullの仕方](https://qiita.com/yaiwase/items/3a58313e028315004a56)　　


## 手順
　　
### **1. Anacondaのイメージ(continuumio/anaconda3)をDockerHubからpullする**  
  
### **2. 作業ディレクトリを作成** 
  
**docker-test**というディレクトリを作成し、titanicに必要なデータとスクリプトを投入  
  
*titanic.ipynb  
*test.csv  
*train.csv  
*gender_submission.csv  


### **3. docker run**  

オプションで、マウントするフォルダパスを適切な箇所にして
`$ docker run`  　コマンドを出す
`$ Docker ps -a`　で確認  


### **4. jupyterlabを立ち上げる**


ブラウザ上で　`localhost:8888:8888` を立ち上げる  

  
 ***  


  
### <今度はDckerfileを書いてJupyternotebookにアクセスしてみる>    
参照：  
[Dockerfileを書いて構築](https://traveler0401.com/docker-anaconda/)  

### **1. 作業ディレクトリを作成** 

**docker-test**というディレクトリを作成し、titanicに必要なデータとスクリプトを投入  
  
<span style="color: skyblue; ">titanicのデータ分析に必要なもの</span>  
*titanic.ipynb  
*test.csv  
*train.csv  
*gender_submission.csv  
<span style="color: skyblue; ">dockerコンテナを立ち上げるのに必要なもの</span>  
*Dockerfile  

### **2. Dockerfileに必要事項をかきこむ**  

[jupyterのオプションの意味参照](https://traveler0401.com/docker-anaconda/)  


### **3. cdコマンドで、カレントディレクトリに移動し、Dockerfileのbuild**  

`$ docker image build -t docker-anaconda .`  
-tはdocker imageに名前をつけるオプション  

<span style="color: red; ">エラー</span>：platformエラー  
M1チップだとplatformを指定する必要がある

`$ docker build --platform linux/amd64 -t docker-anaconda .`  

### **3. docker run**  

オプションで、マウントするフォルダパスを適切な箇所にして
`$ docker run -it -p 8888:8888 --rm -v type=bind,src=`pwd`,dst=/opt docker-anaconda`  　コマンド  

### **4. jupyterlabを立ち上げる**


ブラウザ上で　`localhost:8888:8888` を立ち上げる  

起動はできたけれど、マウントできていない・・・
オプションが多いので、まとめてdocker-composeに記載  
 ***  
  

### <docker-composeから、jupyternotebookを立ち上げて、~/Desktop/docker-testをマウントする>  


**docker-test**のなかにymlファイルを投入  

＊docker-compose.yml  

### **2. docker-compose.ymlに必要事項をかきこむ**  
  
[docker-composeの書き方参照](https://qiita.com/yCroma/items/df4b05cf5a977d9e82cf)  

ここでplatformも指定する必要がある

### **3. コンテナを立ち上げる**



docker-testディレクトリにカレントディレクトリをうつしてコマンド  
    
`$ docker-compose up -d`  


<span style="color: pink; ">まとめ</span>  
docker imageをそのまま使いたい場合は、docker Hubから持ってくると楽。
しかし、それだとパッケージのinstallを独自で決められないのでdockerの意味があまりない。  
dockerfileを書いて、docker-composeにオプションを記載することで、コンテナを渡された人も起動が簡単になる
  

***
***
***


## 作成したdockerコンテナを、自宅パソコンのubuntu20で立ち上げてみる
　

### docker-test フォルダに入っているもの　　

<span style="color: skyblue; ">titanicのデータ分析に必要なもの</span>  
*titanic.ipynb  
*test.csv  
*train.csv  
*gender_submission.csv  
<span style="color: skyblue; ">dockerコンテナを立ち上げるのに必要なもの</span>  
*Dockerfile  
*docker-compose.yml
　　
***
　　
## 手順
　　
### **1. dockerの環境構築**

ubuntuOSに docker,docker-compose をインストールする  
  
参照：  
[dockerのインストール](https://www.trifields.jp/how-to-install-docker-on-ubuntu-2004-4436)  
[docker-composeのインストール](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04-ja)  
  
### **2. 作業フォルダを入手** 

今回はGoogle　Driveからダウンロードした。
  

### **3. コンテナを立ち上げる**



docker-testディレクトリにカレントディレクトリをうつしてコマンド  
    
`$ docker-compose up -d`  

<span style="color: red; ">エラー１</span>：platformｎエラー　　platformについての記載をdocker-compose.ymlに記載したままだったので、該当部分を消去してみた。→OK  


<span style="color: red; ">エラー2</span>：容量が足りなくなった。　そもそも１０Gしか割り当ててなかったので、３０Gに増やした。→OK
テスト先の容量のことも考えないといけない？
  

### **4. jupyterlabを立ち上げる**


ブラウザ上で　`localhost:8888:8888` を立ち上げる  

　　　　　　　　　　　

### **5. 実行してみる**
ライブラリのimportは難なくクリア  

フォルダパスの書き換えが必要。書き換えたら難なくクリア＜完＞
***



<span style="color: pink; ">dockerの利点</span>  
一度 Dockerfileを作ってしまえば、もらった人は　$ docker-compose up -d　とコマンドするだけで同じ環境をつくれるのでとても便利だった(環境つくる人だけdockerの知識があればいい？)
　大人数で使用したり、いろんな環境を使い分けたいときには良さそう  
<span style="color: skyblue; ">dockerの欠点</span>  
初回はdockerをインストールする手間がある
　


***  
***  
*** 



## 前回作成したdockerコンテナを、AWSのEC2で立ち上げてみる　　





### docker-test フォルダに入っているもの　　



<span style="color: skyblue; ">titanicのデータ分析に必要なもの</span>  
*titanic.ipynb  
*test.csv  
*train.csv  
*gender_submission.csv  
<span style="color: skyblue; ">dockerコンテナを立ち上げるのに必要なもの</span>  
*Dockerfile  
*docker-compose.yml
　　
***

## 手順
今回は、講習で教えていただいたCloud9のサービスを利用してUbuntuを動かした。SSHキーを作成してEC2にアクセスすればOK

### **1. dockerの環境構築**

ubuntuOSに docker,docker-compose をインストールする


### **2. 作業フォルダを入手**  

どのようにやるかわからなかったので、適当にフォルダをドラッグしてアップロードしてみたらできた！（簡単）  


### **3. コンテナを立ち上げる**


今回は、前回の反省を活かし、最初にDockerfileのplatformの記載を削除

docker-testディレクトリにカレントディレクトリをうつしてコマンド  
   
`$ docker-compose up -d`
   

　　　　　
<span style="color: red; ">エラー1</span>：容量が足りなくなった。　講習のとおり、８Gしかストレージの確保していなかったので、足りない。
今回の変更可能最大容量、30Gに変更。→OK

<span style="color: red; ">エラー2</span>：管理者権限の問題なのか、apt-getできなかった。。。  
今回は、仮に割り当ててもらったアカウントだったからなのか、原因わからずタイムアウト。

***



<span style="color: pink; ">AWSの利点</span> 
簡単に素早くサーバを利用することができる。
容量の割り当ての変更がものすごく簡単だった。  



<span style="color: skyblue; ">AWSの欠点</span>  
実際課金して使用するとなると、計画的に使うサービスや容量を把握する必要がありそう。

***
***
***
## 前回作成したdockerコンテナを、CentOS7で立ち上げてみる 
### docker-test フォルダに入っているもの　　

<span style="color: skyblue; ">titanicのデータ分析に必要なもの</span>  
*titanic.ipynb  
*test.csv  
*train.csv  
*gender_submission.csv  
<span style="color: skyblue; ">dockerコンテナを立ち上げるのに必要なもの</span>  
*Dockerfile  
*docker-compose.yml
　　
***
　　
## 手順
　　
### **1. dockerの環境構築**
dockerのインストール  
`$ sudo yum -y install docker`  
`$ sudo systemctl start docker`  
`$ sudo systemctl enable docker`  
`$ sudo systemctl status docker`  
Docker管理者をdockerグループに追加  
これをすることで、sudoをつけづにdockerが使えるようになる  [#参考URL](https://docs.docker.com/engine/install/linux-postinstall/)  

`$ sudo usermod -aG docker ayaka`  
ならない！！要課題

docker-composeのインストール  
`$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`  
実行権限の付与  
`sudo chmod +x /usr/local/bin/docker-compose`  

シンボリックリンクの作成  
`sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`


### **2. 作業フォルダを入手**  

前回作成したデスクトップ上のdocker-testディレクトリをSSHでファイル転送する 

***
***
***
## 前回作成したdockerコンテナを、VPS(ubuntu)で立ち上げてみる  


### **1. dockerの環境構築**  
`$ sudo apt install docker docker-compose`

`$ docker version`  
```
   Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied  
```  
dockerコマンドがデフォルトではrootユーザでのみ使用可能だから    
`$ sudo gpasswd -a ayaka docker`  
Adding user ayaka to group docker  
`$ id ayaka`  
uid=1001(ayaka) gid=1001(ayaka) groups=1001(ayaka),27(sudo),118(docker)  
再ログインしたらOK  

`$ docker-compose version`  

### **2. 作業フォルダを入手**  
SSHを使用して、ファイルの転送を行う  
オプションが面倒なら、config記載する  
[参考資料](https://qiita.com/odacp/items/78af073bfcc5f258c06f)  

`$ scp -P 10022 -r /Users/nakamuraayaka/Desktop/docker-test ayaka@133.242.133.61:/home/docker-test`  
```  
Permission denied  
```  
書き込み権限がない・・・  
リモートで権限付与するも入れない・・・  
`$  chmod 666 docker-test`  
/home/docker-test　が違っていた！
正確な場所は、  
/home/ayaka/docker-testだった・・・  
`$ scp -P 10022 -r /Users/nakamuraayaka/Desktop/docker-test ayaka@133.242.133.61:/home/ayaka/docker-test`  
しかもdocker-testディレクトリをリモートで作成してしまったので名前が被ってしまった・・・。(~/docker-test/docker-test)  　　
/optなどの名前にすればよかった。  


<span style="color: red; ">エラー１</span>：platformｎエラー　　platformについての記載をdocker-compose.ymlに記載したままだったので、該当部分を消去してみた。→OK  

ブラウザでの確認はできなかったが、設定は出来たようだった。  