#  amazon-linux-extrasにおけるLAMP環境構築

Ubuntu におけるaptによるLAMP環境の構築方法を記述する。

## 1. パッケージの導入

### PHP7、MySQL、git、php-mbstringの導入

パッケージ導入
```
$ sudo apt update -y
$ sudo apt upgrade -y
$ sudo apt install -y vim apache2 php mysql-server
```

apache2、mysqlの起動
```
$ sudo service apache2 start
$ sudo service mysql start
```

### 2. PHP設定
ドキュメンルートでPHPが実行出来るよう設定する必要がある
`/etc/apache2/apache2.conf`の160行目あたりを変更する

変更前
```
<Directory />
  Options FollowSymlinks
  AllowOverride None
  Require All denied
</Directory>
```

変更後
```
<Directory />
  Options None ExecCGI
  AllowOverride All
  Require All denided
</Directory>
```

## 3. セキュリティ

### User

サイト公開用のユーザーを作成、.pemで接続できるように
```
$ sudo su
# useradd [[user_name]] && groupadd www
# cp -a /home/ec2-user/.ssh /home/[[user_name]]/
# chown -R [[user_name]]:ec2-user /home/[[user_name]]/.ssh
# chown [[user_name]]:www /srv
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
$ sudo service apache2 restart
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

#### 注意
MySQL5.7以上ではrootユーザーに初期パスワードが設定されている場合がある。
初期パスワードが空でmysqlに入れる場合は良いが、入れない場合は`/var/log/mysqld.log`を確認しセキュリティのためにrootユーザーに新たなパスワードを設定する。

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
