#  amazon-linux-extrasにおけるLAMP環境構築

Amazon Linux 2 におけるamazon-linux-extrasリポジトリでのLAMP環境の構築方法を記述する。

##  懸念

amazon-linux-extrasリポジトリを利用する場合はデフォルトのユーザーでそのまま利用するのが良い。

amazon-linux-extrasリポジトリでをそのまま利用するとPHPがfastcgiとなる。

デフォルトのユーザーを利用であれば問題ないが[公開用ユーザーを作成する](#User設定)場合に[PHPの権限を変更する必要がある](#PHP) 。

ファイル修正後に`systemctl restart php-fpm`を実行するが対象ファイルの権限が変更されない。

それにより公開ユーザー権限のPHPファイルが動作せず、`chmod`コマンドで対象ファイルの権限を書き換えることで動作させることができる。

しかし、`php-fpm`の再起動が行われた場合ファイルの権限は上書きされるため、インスタンスが起動（再起動）した場合にphpが動作しなくなるため公開用ユーザーを作成し利用するのはお勧めできない

***


以下、導入手順



***

## 1. パッケージの導入

### PHP7、mariadb、git、php-mbstringの導入

パッケージ導入
```
$ sudo yum update -y
$ sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
$ sudo yum install -y httpd mariadb-server git php-mbstring.x86_64
```

**参考URL**
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/ec2-lamp-amazon-linux-2.html

サーバー起動時のソフトウェア自動起動設定
```
$ sudo systemctl start httpd mariadb
$ sudo systemctl enable httpd mariadb
```



## 2. セキュリティ

### User

サイト公開用のユーザーを作成、.pemで接続できるように
```
$ sudo su
# useradd [[user_name]] && groupadd www
# cp -a /home/ec2-user/.ssh /home/[[user_name]]/
# chown -R [[user_name]]:ec2-user /home/[[user_name]]/.ssh
# chown [[user_name]]:www /srv
```

### PHP

公開用ユーザーに実行権限付与

```
$ sudo vim /etc/php-fpm.d/www.conf

// ;を削除しコメントアウトを解除
;listen.owner = [[user_name]]
;listen.group = [[user_grope]]
```

### httpd

/etc/httpd/conf/http.conf(user、groupを下記に変更)
```
66: User [[user_name]]
67: Group www 
```

## 4. vhosts設定

/etc/httpd/conf.d/(vhost).conf(作成)
```
<VirtualHost *:80>
  ServerAdmin root@[[vhosts.com]]
  ServerName [[vhosts.com]]
  DocumentRoot [[dcoument/root/path]]
  ErrorLog  /var/log/httpd/error_log
  CustomLog /var/log/httpd/access_log combined
  <Directory "[[dcoument/root/path]]">
    AllowOverride All
    Require all granted
  </Directory>
</VirtualHost>
```

上記反映のためにhttpdの再起動
```
$ sudo systemctl restart httpd
```

## 5. ファイル構築

### ssh-agent

ssh-agent(ssh接続前にクライアントで実行)

```
$ ssh-agent
$ ssh-add
```

### git clone

`/srv`のグループとユーザーが変更されていること前提（`sudo chown [[user_name]]:www srv `）

```
// ssh -A [[user_name]]@xxx.xxx... で接続
$ git clone github.... /srv
```

## 6. データベース作成

データベースの作成、ユーザーの作成、権限の付与

- 大文字、小文字、数値を含むパスワードを生成（http://tomari.org/main/java/password.html）
  - 移行の場合はパスワードをそのまま流用
  - 本パスワードを使用するため、保存しておく

```
$ sudo mysql
> CREATE DATABASE [[database_name]];
> CREATE USER '[[database_username]]'@'localhost' idetified by 'PASSWORD STRINGS';
> GRANT ALL ON [[database_name]].* to '[[database_username]]'@'localhost';
```

rootのセキュリティを設定

- rootのセキュリティを確保するためにパスワードを設定
  - mariadbの場合はパスワードが設定されていない
  - 今後rootアカウントを使用しない場合はパスワードを残す必要はない
  - 大文字、小文字、数値を含む12桁の文字列（http://tomari.org/main/java/password.html）
  - 基本的に全てYes

```
sudo mysql_secure_installation
```
