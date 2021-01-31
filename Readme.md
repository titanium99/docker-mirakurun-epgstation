# docker-mirakurun-epgstation
[Mirakurun](https://github.com/Chinachu/Mirakurun) + [EPGStation](https://github.com/l3tnun/EPGStation) の Docker コンテナ

## 前提条件
- Docker, docker-compose の導入が必須
- ホスト上の pcscd は停止する
- px-w3u4/q3u4での使用を想定

## 事前準備
### b-casカードリーダーの設定
```sh
# ライブラリインストール
sudo apt install -y libpcsclite-dev pcscd pcsc-tools libccid
 
# IC面が上になるよう、カードの向きに注意してB-casをリーダに挿入。
$ pcsc_scan
(略)
Japanese Chijou Digital B-CAS Card (pay TV)
# ↑の表示が出たら Ctrl + C で終了
```

### チューナードライバの設定
```sh
# ライブラリインストール
sudo apt install -y dkms git

mkdir ~/src
cd ~/src
git clone https://github.com/nns779/px4_drv
cd px4_drv/fwtool/
make
wget http://plex-net.co.jp/plex/pxw3u4/pxw3u4_BDA_ver1x64.zip -O pxw3u4_BDA_ver1x64.zip
unzip -oj pxw3u4_BDA_ver1x64.zip pxw3u4_BDA_ver1x64/PXW3U4.sys
./fwtool PXW3U4.sys it930x-firmware.bin
sudo mkdir -p /lib/firmware
sudo cp it930x-firmware.bin /lib/firmware/


# カーネルヘッダーのインストール（カーネル変更時）
uname -r
sudo apt install -y linux-headers-$(uname -r)


# px4_drvドライバのインストール
cd ~/src/px4_drv/driver/
make
sudo make install

cd ~/src/px4_drv
sudo cp -a ./ /usr/src/px4_drv-0.2.1
sudo dkms add px4_drv/0.2.1
sudo dkms install px4_drv/0.2.1

# カーネルモジュールのロードの確認
lsmod | grep -e ^px4_drv
#> px4_drv                81920  0
# px4_drvから始まる行が表示されれば、カーネルモジュールが正常にロードされています

# ホストのb-casカードリーダー停止
sudo systemctl stop pcscd.socket
sudo systemctl disable pcscd.socket
```
### arib25のインストール(docker版は同梱されているので不要)
```sh
$ mkdir ~/git
$ cd ~/git
$ git clone https://github.com/stz2012/libarib25.git
$ cd libarib25/
$ cmake .    <--- '.'を忘れずに!!
$ make
# sudo make install
# sudo /sbin/ldconfig
```

### recpt1のインストール
```sh
$ cd ~/src/
$ git clone https://github.com/stz2012/recpt1.git
$ cd recpt1/recpt1
$ ./autogen.sh
# autogen.sh を実行する際エラーでたら下記入れる
$ sudo apt install autoconf automake
$ ./configure --enable-b25
$ make
$ sudo make install
```

## インストール手順

```sh
$ git clone https://github.com/titaium99/docker-mirakurun-epgstation.git
$ cd docker-mirakurun-epgstation
$ which recpt1
/usr/local/bin/recpt1
$ cp /usr/local/bin/recpt1 ./mirakurun/opt/bin/
$ vim /usr/local/mirakurun/config/tuners.yml
$ sudo docker-compose pull
$ sudo docker-compose build

#チャンネル設定
$ vim mirakurun/conf/channels.yml

#コメントアウトされている restart や user の設定を適宜変更する
$ vim docker-compose.yml
```

## 起動

```sh
$ sudo docker-compose up -d
```
mirakurun の EPG 更新を待ってからブラウザで http://DockerHostIP:8888 へアクセスし動作を確認する

## 停止

```sh
$ sudo docker-compose down
```

## 設定

### Mirakurun

* ポート番号: 40772

### EPGStation

* ポート番号: 8888
* ポート番号: 8889

### 各種ファイル保存先

* 録画データ

```./recorded```

* サムネイル

```./epgstation/thumbnail```

* 予約情報と HLS 配信時の一時ファイル

```./epgstation/data```

* EPGStation 設定ファイル

```./epgstation/config```

* EPGStation のログ

```./epgstation/logs```
