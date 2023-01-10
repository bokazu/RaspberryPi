# 機器のセットアップ

1. まずはSTARTAR KITのほうを開封し、Rasberry Pi本体を取り出します。
2. これにヒートシンクを取り付け、ケースにはめます。ケースのほうにはファンを取り付けGPIOボードにワイヤを接続したうえで蓋をしめます。ファンの向きとしてはシールの向いている方をラズパイ側に向けます。
<img src = "./img/IMG_7275.JPG" width = "250"> <img src = "./img/IMG_7276.JPG" width = "250">
3. micro SDを差し込み、ディスプレイ、マウス、キーボード、電源ケーブルを接続したうえで電源を入れます。
<img src = "./img/IMG_7279.JPG" width = "250">
4. 虹色の画面が現れた後、以下のような画面が現れOSのインストールが始まります。
<img src = "./img/IMG_7277.JPG" width = "250">
5. 画面に現れる指示に従って、user name、password、wifiの設定を行いました。
<br>
<br>

# 公開鍵認証方式SSH接続の設定
以下のサイトを参考にしながらSSH接続の設定を行った(ラズパイに固定IPアドレスを設定していないのであとで接続できなくなる可能性あり)。


[RaspberryPiに公開鍵認証を使ってSSH接続する](https://tool-lab.com/raspi-key-authentication-over-ssh/#index_id0)


固定IPアドレスを割り振りたい場合は次のページを参照すると良さそうである。


[MacでRaspberryPiセットアップ](https://tool-lab.com/mac%e3%81%a7raspberry-pi%e3%82%bb%e3%83%83%e3%83%88%e3%82%a2%e3%83%83%e3%83%97%e8%a3%9c%e8%b6%b3/)

基本的には上記のサイトを見たほうがわかりやすいが、自分用の備忘録として手順について以下に書いておく。
    
<br>
なお、以下の処理は一貫してmacbookのターミナル上で行っている

### 公開鍵ファイルの設定
Mac側でターミナルを開き
```zsh
~$ ssh-keygen -t rsa
```
と打つ。
```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/tool-lab/.ssh/id_rsa): 
```
のように聞いてくるので問題なければEnterを押す(今回はid_rsa_raspyという名前にした)。その後で、パスフレーズを聞かれるので設定する。

すると、鍵ファイルを生成した旨のメッセージがでてくるので以下のコマンドでファイルを確認する。
```
~$ ls .ssh
```
今回の場合、
```
id_rsa_raspy  id_rsa_raspy.pub
```
の２つのファイルが生成されていればここまではOKである。
<br>
<br>
### 公開鍵ファイルをRaspberryPiに保存する
まずはラズパイのIPアドレスを以下のコマンドで調べる
```zsh
~$ ping raspberrypi.local
```
すると192.xxx.yyy.zzzのような値が出てくるがそれがIPアドレスである(ずっと情報が更新され続けるためCtr + Cで上記のコマンドを終わらせる)。
次に、
```
scp .ssh/id_rsa_raspy.pub pi@[ラズパイのIPアドレス]
```
と打つとRaspberry Piのユーザーpiのパスワードを聞いてくるので入力する。

<br>

### RaspberryPiのsshサーバ設定を変更する
まずはMac側のターミナルで以下のコマンドを打ちラズパイに接続する。
```
~$ ssh pi@[ラズパイのIPアドレス]
```
パスワードを要求されるので再び入力し接続する。

次に以下のコマンドで.sshフォルダを作成する。
```
pi@raspberrypi ~$ mkdir .ssh
```
さらにid_rsa_raspy.pubの情報を.ssh内のauthorized_keysに書き出す。
```
pi@raspberry ~$ cat id_rsa_raspy.pub >> .ssh/authorized_keys
```
その後、以下のコマンドでアクセス権限を変更する。
```
pi@raspberry ~$ chmod 700 .ssh
pi@raspberry ~$ chmod 600 .ssh/authorized_keys
```

以上で公開鍵ファイルを使った処理は終わったのでこれを削除しておく。
```
pi@raspberry ~$ rm id_rsa_raspy.pub
```
<br>

### sshサーバファイルの設定
sshサーバ設定ファイルは`/etc/ssh/sshd_config`でありこれを編集する。
```
pi@raspberry ~$ sudo nano /etc/ssh/sshd_config
```
エディタが起動するのでそこでファイルに書いてある項目を少し変更する。具体的には
- 接続ポート番号の変更
- rootログインの禁止
- 鍵ファイル認証方式の有効化
- パスワード認証の無効化

を行う。
```
　　︙
Port xxxx #別の番号で設定する。
RSAAuthentication yes
PubkeyAuthentication yes 
AuthorizedKeysFile   %h/.ssh/authorized_keys
PasswordAuthentication no
　　︙
```
ポート番号は予め目的が決められたものがあるので[TCP/IPポート番号一覧(外部サイト)](http://www.vwnet.jp/mura/PortNumbers/tcpip-port.asp)を参考にしながら決める。
上記のように変更したらCtrl + Oを押し、Enterで保存する。その後Ctrl + Xで終了することができる。
最後に
```
pi@raspberry ~$ sudo /etc/init.d/ssh restart
```
でsshサーバを再起動する。
<br>


### 確認
Mac上でもう一つターミナルを開き
```
~$ ssh -i .ssh/id_rsa_raspy -p [ポート番号] pi@192.xxx.yyy.zzz
```
と打つ。このとき、パスワード認証ではなく、今回最初に入力したパスフレーズが聞かれれば精巧である。

<br>

### コマンドを省略するための処理

いちいち上記のコマンドを打つのはめんどくさいので
```
~$ nano .ssh/config
```
でファイルを開き以下のようなフォーマットで記述する。
```
Host [任意の名前]
HostName [ラズパイのIPアドレス]
User [ユーザーID]
Port [ポート番号]
IdentityFile [秘密鍵ファイル]
```
今回は
```
Host raspi
HostName 192.xxx.yyy.zzz
User pi
Port [ポート番号]
IdentityFile ~/.ssh/id_rsa_raspy
```
となる。これで次回から
```
~$ ssh raspi
```
で接続できるように成る。
