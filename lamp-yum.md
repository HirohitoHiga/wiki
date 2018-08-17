# yumによるLAMP構築

CentOS、Redhat、Amazon Linuxなどで用いられるパッケージ管理コマンド「yum」によるLAMP環境の構築を記述する。

## 面倒な点

CentOS7からデフォルトのデータベースが「MariaDB」に変更された。
上記に伴いMySQLはyumの管理外となり、MySQLを利用したい場合は別途リポジトリ経由でインストールする必要がある。

PHPについてもyumで管理されている最新バージョンが5.4.16となりかなり古い。
そのため、5.5、5.6、5.7、7.xを利用したい場合は別途リポジトリ経由でインストールする必要がある。

MySQLの最新は下記となる（2018/08/17現在）
- 5.6.40
- 5.7.22
- 8.0.11

参考URL
http://openstandia.jp/oss_info/mysql/version/

## 1.MySQLのインストール

1) デフォルトで入っているDBの削除
```
$ sudo yum remove mariadb-libs
```

2) mysql57のリポジトリインストール

```
$ sudo yum install http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

3) mysqlマイナーバージョンをダウンロードし解凍（5.7.23）

下記URLを参考にバージョンを指定する
https://dev.mysql.com/downloads/mysql/5.7.html#downloads

```
$ sudo wget https://downloads.mysql.com/archives/get/file/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
```

4) ダウンロードしたtarファイルを解凍
```
$ tar -xvf mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
```

5) 解凍したrpmをインストール
```
$ sudo yum localinstall mysql-community-common-5.7.23-1.el7.x86_64.rpm / 
mysql-community-libs-5.7.23-1.el7.x86_64.rpm / 
mysql-community-devel-5.7.23-1.el7.x86_64.rpm / 
mysql-community-client-5.7.23-1.el7.x86_64.rpm / 
mysql-community-server-5.7.23-1.el7.x86_64.rpm
```

## PHPのインストール

```
$ sudo yum -y --enablerepo=remi-php72 install remi-php72 php php-devel php-mbstring php-pdo php-gd php-xml php-mcrypt php-mysqlnd
$ sudo service httpd restart
```

## vhosts設定
vhost.confの作成
```
$ cd /etc/httpd/conf.d/
$ sudo vim vhosts.conf

<VirtualHost *:80>
  DocumentRoot /var/www/html/website_name
  ServerName websitename_dummy.com
</VirtualHost>

$ sudo service httpd restart
```

httpd.confへの追加
```
#
# Write htaccess code for site speed
#
<Directory "/var/www/html/musee-marketing/container/wp/html">
    RewriteEngine On
    RewriteBase /
    RewriteRule ^index\.php$ - [L]
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.php [L]
</Directory>

#
# Basic authentication of wp-admin
#
<Directory "/var/www/html/musee-marketing/container/wp/html/wp">
    AllowOverride All
</Directory>
```

上記反映のため再起動
```
$ sudo service httpd restart
```

## ImageMagickの追加（必要であれば）

下記URLからImageMagickの最新バージョンを確認
https://www.imagemagick.org/download/linux/CentOS/x86_64/


```
$ sudo yum --enablerep=remi-php71 install php-pear
$ sudo yum install gcc
$ sudo yum -y install https://www.imagemagick.org/download/linux/CentOS/x86_64/ImageMagick-libs-7.0.8-10.x86_64.rpm
$ sudo yum -y install https://www.imagemagick.org/download/linux/CentOS/x86_64/ImageMagick-7.0.8-10.x86_64.rpm
$ sudo yum -y install https://www.imagemagick.org/download/linux/CentOS/x86_64/ImageMagick-devel-7.0.8-10.x86_64.rpm
$ sudo pecl install imagick
```

php.iniファイルの編集（下記を追加）

```
include_path=".:/usr/bin/pear”
```
```
[imagick]
extension=imagick.so
```
