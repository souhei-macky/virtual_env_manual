# 環境構築手順書
---
VirtualBox と Vagrant を使用した仮想環境を構築するための手順を記載します。
## バージョン一覧
---  
| |&nbsp;&nbsp;&nbsp;php&nbsp;&nbsp;&nbsp; |&nbsp;Nginx&nbsp; |MySQL |Laravel |CentOS|
|:-:|:-:|:-:|:-:|:-:|:-:|
|vresion|7.3|1.19|5.7|6.18|7.8|

## 環境構築の手順
---
#### VirtualBox のインストール  
下記のサイトからそれぞれのdmgファイルをダウンロード後、インストールを進めます。
・[Virtual Box公式](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0)
**※ Vagrantの最新バージョンがVirtualBoxの最新バージョンに対応していないため、Virtual Boxはver6.0.14をインストールするようにしてください。**
 **mac**
`OS X hosts` を選択します。
以下のコマンドを実行してVirtualBoxのウィンドウが表示されれば正常にインストールされています。
`$ virtualbox`
コマンド実行後は入力を受け付けない状態となるため、 `Control + c` を押して下さい。

#### Vagrant のインストール
 下記コマンドでインストールできます。
`$ brew cask install vagrant`
#### Vagrant box のインストール
 今回はLinuxのCentOSのバージョン7のbox名 `centos/7` を指定して下記コマンドを実行します。
`$ vagrant box add centos/7`
上記コマンドを実行すると下記の様に表示されるので、3を選択しEnterを押します。
`1) hyperv`
`2) libvirt`
`3) virtualbox`
`4) vmware_desktop`  
`Enter your choice: 3`
下記の様に表示されたら完了です。
`Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!`

#### Vagrantの作業ディレクトリの用意
 自分の作業用ディレクトリでvagrant作業用ディレクトリを作成します。
例 `$ mkdir vagrant_review`
作成したディレクトリに移動します。
`$ cd vagrant_review`
vagrant init box名 下記コマンドを実行し、先ほどダウンロードしたbox(centOS7)を使用します。
`$ vagrant init centOS7`
実行後問題なければ以下のような文言が表示されます。
A Vagrantfile has been placed in this directory. You are now
ready to vagrant up your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
vagrantup.com for more information on using Vagrant.

#### Vagrantfileの編集
`$ vi vagrantfile`
viエディタを使用し3箇所を編集します。
```
config.vm.network "forwarded_port", guest: 80, host: 8080 (今回は10080に変更)
config.vm.network "private_network", ip: "192.168.33.10" (今回は192.168.33.19に変更)
```
上記2箇所の#を外しコメントインします。
※ host, ip が以前と同じ場合は変更を加えます。
また、以下の箇所はコメントインし変更を加えて下さい。
```
config.vm.synced_folder "../data", "/vagrant_data"
↓
config.vm.synced_folder "./", "/vagrant", type:"virtualbox"
```
./ はカレントディレクトリ(vagrant_review)を示しており、ホストOS (Mac or Windows) のvagrant_reviewディレクトリ内とゲストOS (Vagrant) の /vagrant のディレクトリ内をリアルタイムで同期するための設定です。

#### vagrant プラグインのインストール
まず vagrant-vbguest というプラグインをインストールします。
これはBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグインです。
`$ vagrant plugin install vagrant-vbguest`
下記コマンドでインストールの完了の確認をしましょう。
`$ vagrant plugin list`
#### ゲストOSへのログイン
作成した`vagrant_review`ディレクトリで下記コマンドを実行しゲストOSへログインをします。
`$ vagrant up`
初回は時間がかかりますのでお待ち下さい。
`$ vagrant ssh`
以下のような表記になっていればゲストOSにログインしていることになります。
`[vagrant@localhost ~]$`
#### パッケージのインストール
`development tools` というグループパッケージをインストールします。
`$ sudo yum -y groupinstall "development tools"`
このコマンドでgitなどの開発に必要なパッケージを一括でインストールできます。
#### PHPのインストール
次にphpをインストールします。
`$ sudo yum -y install epel-release wget`
`$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm`
`$ sudo rpm -Uvh remi-release-7.rpm`
`$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip`(今回はphp 7.3を指定)
`$ php -v`
バージョンの確認ができればインストール完了です。
#### composerのインストール
次にcomposerをインストールしていきます。
composerはPHPのパッケージの依存関係を管理・解決するツールです。
`$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"`
`$ php composer-setup.php`
`$ php -r "unlink('composer-setup.php');"`
どのディレクトリにいてもcomposerコマンドを使用を使用できるようfileの移動を行います。
`$ sudo mv composer.phar /usr/local/bin/composer`
`$ composer -v`
composer のバージョンが確認できればインストール完了です。

以上でゲストOS内にPHPとcomposerコマンドの実行環境が整いました。

#### Webサーバソフトウェアのインストール
まず、ゲストOSにログインしていない場合はログインして下さい。
今回はNginxの最新版をインストールします。
viエディタを使用して以下のファイルを作成します。
`$ sudo vi /etc/yum.repos.d/nginx.repo`
書き込む内容は以下になります。
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
gpgcheck=0
enabled=1
```
書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行します。
`$ sudo yum install -y nginx`
`$ nginx -v`
Nginxのバージョンが確認できたら起動させます。
`$ sudo systemctl start nginx`
ブラウザにて http://192.168.33.19 (Vagrantfileでipを書き換えた方はそのipアドレス)と入力し、NginxのWelcomeページが表示されたら問題なく起動しているので完了です。

#### データベースのインストール
今回データベースはMySQL version5.7をインストールします。
rpmに新たにリポジトリを追加し、インストールを行います。
`$ sudo wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm`
`$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm`
`$ sudo yum install -y mysql-community-server`
`$ mysql --version`
バージョンの確認ができたらインストール完了です。
次にMySQLを起動し接続を行います。
`sudo systemctl start mysqld`
今回はデフォルトでrootにパスワードが設定されてしまっています。
まずはpasswordを調べ、接続しpassswordの再設定を行います。
`$ sudo cat /var/log/mysqld.log | grep 'temporary password'`
上記コマンドの実行結果が下記のように表示されたらOKです。
`2020-10-08T07:54:31.644793Z 1 [Note] A temporary password is generated for root@localhost: "文字列"`
"文字列"に当たる箇所がパスワードです。
mysqlにログインすることができたらパスワードを変更しましょう。
`mysql > set password = "新たなpassword";`
MySQL5.7のパスワードポリシーは厳格で開発段階では非常に面倒のため、今回は以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更します。
`$ sudo vi /etc/my.cnf`
下記エディタの編集箇所を編集して下さい。
```
# 省略

[mysqld]

# 省略

# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# 下記の一行を追加
validate-password=OFF
```
編集後はMySQLサーバの再起動が必要です。
`$ sudo systemctl restart mysqld`
もう一度mysqlにログインしpasswordを変更しましょう。

#### LaravelのProjectを作成
ゲストOS内で動かすLaravelアプリケーションを作成していきます。
ゲストOSにログインしている場合はログアウトして下さい。
vagrantfileがある親ディレクトリへ移動します。
`cd vagrant_review`
下記コマンドでインストールして下さい(今回はバージョン6.0を指定)
`composer create-project laravel/laravel --prefer-dist laravel_login_app 6.0`

#### Laravelを動かす
ゲストOSへログインして下さい。
まずNginxの設定ファイルを編集していきます。
使用しているOSがCentOSの場合、`/etc/nginx/conf.d` ディレクトリ下の `default.conf` ファイルが設定ファイルとなります。
`$ sudo vi /etc/nginx/conf.d/default.conf`
```
server {
  listen       80;
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述して下さい。
  root /vagrant/laravel_login_app/public; # 追記
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #    root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略
```
Nginxの設定ファイルの変更は、以上です。
次に php-fpm の設定ファイルを編集していきます。
`$ sudo vi /etc/php-fpm.d/www.conf`
変更箇所配下になります。
```
;24行目近辺
user = apache
↓ 変更
user = nginx
group = apache
↓ 変更
group = nginx
```
設定ファイルの変更に関しては、以上となります。
早速起動しましょう(Nginxは再起動になります)。
`$ sudo systemctl restart nginx`
`$ sudo systemctl start php-fpm`
再度ブラウザにて、 **http://192.168.33.19** を入力して確認して下さい。
ブラウザにLaravelのwelcomeページが表示されたら終了です。

また2点エラーが起こる可能性があるので、その場合はページ最後のエラー解決の項目を確認してみて下さい。

#### ゲストOS内のデータベースの作成
ゲストOS内で動かすLaravelのloginアプリケーションで使用するデータベースの作成を行います。
ゲストOSへログインして下さい。
その後mysqlにログインして下さい。
`mysql > create database laravel_login_app;`
Query OKと表示されたら作成は完了となります。

#### ログイン機能の実装
今回はLaravelのプロジェクトにログイン機能のみ実装します。
ゲストOSにログインして下さい。
`$ cd /vagrant/laravel_login_app`
下記コマンドで**laravel/ui**パッケージを導入します。
`$ composer require laravel/ui "^1.0" --dev`
`$ php artisan ui vue --auth`
また下記コマンドprojectのenvファイルのDB設定を各自変更して下さい。
`$ vi .env`
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_login_app
DB_USERNAME=root
DB_PASSWORD=
```
その後下記コマンドでマイグレーションを実行して下さい。
`$ php artisan migrate`
ブラウザで新規登録とログインが行えるか確認して下さい。
問題なく行えたら完了です。

#### エラー解決
**Forbidden 403** というエラーが出た場合の対処法を以下に記載します。
viエディタを使用してSELinuxの設定を変更します。
「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化します。
`$ sudo vi /etc/selinux/config`
viエディタが開き設定ファイルが表示されるので下記の部分を探して下さい。
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=enforcing
```
こちらの記述を下記のように書き換えて保存して下さい。
`SELINUX=disable`
設定を反映させるためにゲストOSを再起動する必要があるので、ゲストOSをから一度ログアウトして下記コマンドを実行して下さい。
`$ exit`
`$ vagrant reload`
再度ゲストOSにログインしNginxとphp-fpmを起動して下さい。
`$ sudo systemctl start nginx`
`$ sudo systemctl start php-fpm`
ブラウザを確認して下さい。

また下記のようなエラーが出た時の対処法を以下に記載します。
`The stream or file "/vagrant/laravel_app/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied`
Permission denied は主に権限に関するエラーです。
Nginxの操作権限が無いためのエラーなので、操作権限を変更します。
`$ cd /vagrant/laravel_login_app`
`$ sudo chmod -R 777 storage`
ブラウザを確認して下さい。

## 環境構築の所感
---
・yumやcomposerなど依存関係を管理解決するパッケージがインストールを簡易化していくれている。
・Permission denied が権限によるエラーと理解したことで早急にエラーの解決ができた。
・vagrantのエラーで何度も失敗したことで理解が進んだので、  練習のうちは繰り返しチャレンジする必要性を感じた。

## 参考サイト
---
[GizTech](http://giztech.gizumo-inc.work/categories/18)
[VagrantにSaharaを導入](https://qiita.com/sudachi808/items/09cbd3dd1f5c25c23eaf)
[PHP7.3+VirtualBox+Vagrantの開発環境構築](https://qiita.com/KZ-taran/items/57cde1c21958ede20032)
[認証6.x Laravel](https://readouble.com/laravel/6.x/ja/authentication.html)
[Laravel6 ログイン機能を実装する](https://qiita.com/ucan-lab/items/bd0d6f6449602072cb87)
[Markdown記法 サンプル集](https://qiita.com/tbpgr/items/989c6badefff69377da7)
[MarkdownでTable(表テーブル)を書く](https://notepm.jp/help/markdown-table)










