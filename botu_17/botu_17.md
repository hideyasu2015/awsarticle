# (17)Ruby on railsアプリの公開(1)

## 本章の目的：

- WEBサーバー　Apachのインストール 

- Rubyのバージョン管理のソフト　Rbenv　を利用してRubyをインストールする


***

いよいよ今回から自分のアプリケーションをデプロイして行く準備をします。
デプロイとはWEB上に公開すると言うことですね。
WEBで公開するために、IPアドレスを取得したり、VPC仮想ネットワークを前章までで作ってきました。

まずログインします。
```
ssh -i "[秘密鍵のプライベート鍵を入力拡張子は.pem]" centos@ec2-[erasticIP].ap-northeast-1.compute.amazonaws.com
```
####Apachのインストール

前章から　yum というコマンドでインストールしていると思います。
これは何かと言うと、AWSはCentOS系のLinuxになります。
CentOS公式 yum リポジトリに提供されているものはyum でインストールできます。
依存関係のパッケージも一緒に入れてくれますし、間違えた場合でも、最悪yum インストールを利用していると
もとに戻せるのも便利です。

提供されているApacheを 確認します

```
$ yum info httpd

//==
Loaded plugins: priorities, update-motd, upgrade-helper
Available Packages
Name        : httpd
Arch        : x86_64
Version     : 2.2.34
Release     : 1.16.amzn1
Size        : 1.2 M
Repo        : amzn-main/latest
Summary     : Apache HTTP Server
URL         : http://httpd.apache.org/
License     : ASL 2.0
Description : The Apache HTTP Server is a powerful, efficient, and extensible
            : web server.
```
ここでVersion : 2.2.34
と表示されていますので、2.2がインストールできます。
これはサーバーの環境により異なります。
ですから必ず、yumのインストールの場合、バージョン情報を確認するのは大切ですね。

```
$ yum install httpd

//途中でy no と聞かれますが　ｙを入力してください。
Dependency Installed:
  apr.x86_64 0:1.5.2-5.13.amzn1                                                 
  apr-util.x86_64 0:1.5.4-6.18.amzn1                                            
  apr-util-ldap.x86_64 0:1.5.4-6.18.amzn1                                       
  httpd-tools.x86_64 0:2.2.34-1.16.amzn1                                        
//依存関係もインストールしてくれてますね。
Complete!
```
このように表示されると成功です

バージョン確認

```
$ httpd -version

//==
Server version: Apache/2.2.34 (Unix)
```
####Apachの設定
Apachはサーバーを起動しても自動で立ち上がりません。
自動で立ち上がるようにするコマンドです。

```
$ sudo chkconfig httpd on

//とりあえず初回は自分で起動しましょう。

$ sudo service httpd start

//Apachの名前は httpdになります。
```



ちなみにGitの本家サイトです。(参考)
このテキストのとおりにいかないときは、本家サイトを調べると問題が解決します。
本家サイトを調べる習慣をつけましょう！
```
https://git-scm.com/book/ja/v1/%E4%BD%BF%E3%81%84%E5%A7%8B%E3%82%81%E3%82%8B-Git%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB
```

Gitがインストールされたかの確認です。
バージョンが表示されればインストールされているということです。

```
$ git --version

git version 1.8.3.1
```
このようにバージョンが表示されるとインストールされているということです。
これはインストールするたびに習慣づけましょう。


####rbenvのインストール

Githubからダウンロードします。
ここが本家サイト		
```
https://github.com/rbenv/rbenv
```
![図1. EC２ダッシュボード](17-1.png)

本家サイトで必ずインストール方法を確認してくださいね。
1年以上すぎると、WEBの情報は当てになりません。

```
$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv

//こんな感じで表示されると思います。
[centos@ip-10-0-1-24 ~]$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
Cloning into '/home/centos/.rbenv'...
remote: Enumerating objects: 2744, done.
remote: Total 2744 (delta 0), reused 0 (delta 0), pack-reused 2744
Receiving objects: 100% (2744/2744), 515.63 KiB | 0 bytes/s, done.
Resolving deltas: 100% (1720/1720), done.
```

インストールしたらパスを通します。
このパスを通さないと、実際に利用できませんので、インストールしていないのと同じことになります。
大切ですね。
パスは通常ユーザーフォルダの.bash_profileというファイルで管理しています。
そこへ書き込んでおく必要があります。
ユーザーフォルダへは、
```
cd ~/
```
で移動できます。
	そこで
```
$ ls- la

total 16
drwx------. 5 centos centos 121 Dec 24 03:42 .
drwxr-xr-x. 3 root   root    20 Nov 26 13:39 ..
-rw-------. 1 centos centos 178 Dec 24 03:16 .bash_history
-rw-r--r--. 1 centos centos  18 Apr 11  2018 .bash_logout
-rw-r--r--. 1 centos centos 193 Apr 11  2018 .bash_profile
-rw-r--r--. 1 centos centos 231 Apr 11  2018 .bashrc
drwxrw----. 3 centos centos  19 Dec 24 03:42 .pki
drwxrwxr-x. 9 centos centos 219 Dec 24 03:42 .rbenv
drwx------. 2 centos centos  29 Nov 26 13:39 .ssh

```
.bash_profileがあるのがわかりますね。
ls -laは、ファイルを一覧表示して、権限なども表示する大切なコマンドですので、覚えておきましょう。
Linuxのコマンドは以下のサイト等で、勉強して下さい。

https://dotinstall.com/lessons/basic_unix_v2
https://techacademy.jp/magazine/6406

####Rbenvにパスを通します。
```
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
```
パスを通すとは.bash_profileに　上記の記述をするということです。
ウインドウズのショートカットを作るイメージです。
パスを通さないと、インストールしたソフトが利用できません。
では、バージョンを確認しましょう。
```
[ec2-user@udemy-rails-web1 ~]$ rbenv -v
-bash: rbenv: command not found
```
あれ、確かにインストールしたのに表示されていません。
今の設定を反映する必要があります。
sourceコマンドを利用します。

```  
[ec2-user@udemy-rails-web1 ~]$ source .bash_profile 
[ec2-user@udemy-rails-web1 ~]$ rbenv -v

//==
rbenv 1.1.1-39-g59785f6
```
今度はきちんと表示されましたね。


####rbenv-updateのインストール

rbenv自体もアップデートされます。
そのときに、古いrbenvを削除して、新しいの入れると、ミスが発生してエラーが起きやすくなります。
そのために専用のアップデーターをインストールします。
本家サイトはこちら

```
https://github.com/rkh/rbenv-update
```
ここをみると"rbenv root)/plugins"
pluginsというフォルダを作成してくださいとありますね。

まず今の作業場所とフォルダ構成を確認
```
$ pwd

//==
/home/ec2-user
```

```
$ ls -la

//==
total 36
drwx------ 5 ec2-user ec2-user 4096 Dec 31 03:13 .
drwxr-xr-x 3 root     root     4096 Dec 30 02:18 ..
-rw------- 1 ec2-user ec2-user  114 Dec 31 01:43 .bash_history
-rw-r--r-- 1 ec2-user ec2-user   18 Aug 30  2017 .bash_logout
-rw-r--r-- 1 ec2-user ec2-user  230 Dec 31 03:22 .bash_profile
-rw-r--r-- 1 ec2-user ec2-user  124 Aug 30  2017 .bashrc
drwxrw---- 3 ec2-user ec2-user 4096 Dec 31 03:13 .pki
drwxrwxr-x 9 ec2-user ec2-user 4096 Dec 31 03:13 .rbenv
drwx------ 2 ec2-user ec2-user 4096 Dec 30 02:18 .ssh
```

####.benv配下にpluginフォルダを作成します。
ディレクトリを作ります。
```
$ mkdir ~/.rbenv/plugins
```

```
cd ~/ は最初にログインしたユーザーの位置に移動します。
```

gitからcloneしてインストールします。
先程作成したフォルダにインストールしているのがわかります。

```
$ git clone https://github.com/rkh/rbenv-update.git ~/.rbenv/plugins/rbenv-update
```
rbenvをアップデートしておきます。

```
$ rbenv 

//==
Updating rbenv
Updating rbenv-update
```

####ruby-buildのインストール

rbenvを利用するといろんなバージョンのRubyが利用できます。いろんなバージョンのRubyをインストールするのに便利な	ruby-buildをインストールします。

```
$ git clone https://github.com/rbenv/ruby-build ~/.rbenv/plugins/ruby-build

//表示の例
Cloning into '/home/centos/.rbenv/plugins/ruby-build'...
remote: Enumerating objects: 14, done.
remote: Counting objects: 100% (14/14), done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 9605 (delta 3), reused 11 (delta 2), pack-reused 9591
Receiving objects: 100% (9605/9605), 2.01 MiB | 1.83 MiB/s, done.
Resolving deltas: 100% (6291/6291), done.

```
####gemのインストール先の確認

```
$ gem environment

//==
RubyGems Environment:
  - RUBYGEMS VERSION: 2.0.14.1
  - RUBY VERSION: 2.0.0 (2015-12-16 patchlevel 648) [x86_64-linux]
  - INSTALLATION DIRECTORY: /home/ec2-user/.gem/ruby/2.0
  - RUBY EXECUTABLE: /usr/bin/ruby2.0
  - EXECUTABLE DIRECTORY: /home/ec2-user/bin
  - RUBYGEMS PLATFORMS:
    - ruby
    - x86_64-linux
  - GEM PATHS:
     - /home/ec2-user/.gem/ruby/2.0
     - /usr/share/ruby/gems/2.0
     - /usr/local/share/ruby/gems/2.0
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :backtrace => false
     - :bulk_threshold => 1000
  - REMOTE SOURCES:
     - https://rubygems.org/
```
ここをみてどんな事がわかりますか？
 - GEM PATHS:
     - /home/ec2-user/.gem/ruby/2.0
の表記から、いまgemをインストールすると、グローバルのRuby配下にインストールされてしまいます。
rbenvで管理することができなくなるので、gemのパスも設定しておきます。


####パスを通します。

```
//gem関係
$ echo 'export GEM_HOME=$HOME/.gem' >> ~/.bash_profile

$ echo 'export GEM_PATH=$HOME/.gem/bin' >> ~/.bash_profile

$ echo 'export PATH="$HOME/.rbenv/bin:$GEM_PATH:$PATH"' >> ~/.bash_profile

//rbenv
$ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

$source ~/.bash_profile

```
echo で書き出し　>>のマークはファイル場所を指します。
''の中を　~/.bash_profileへ書き込んでねという意味です。

ちなみに.bash_profileの中を確認するときちんとパスが設定されています。

```
export PATH
export GEM_HOME=$HOME/.gem
export GEM_PATH=$HOME/.gem/bin
export PATH="$HOME/.rbenv/bin:$GEM_PATH:$PATH"
eval "$(rbenv init -)"
```

####rbenv-updateのインストール
本家サイト
https://github.com/rkh/rbenv-update

```
$ git clone https://github.com/rkh/rbenv-update.git ~/.rbenv/plugins/rbenv-update
```

インストールしたらupdateします。

```
$ rbenv update

//こんなのが表示されます
Updating rbenv
Updating rbenv-update
Updating ruby-build
```

####ここまでの確認
ここまでで、Railsをインストールする環境を作ってきました。
途中で抜けや間違いがあるかもしれません。
そんなときのために、便利なコマンドがあります。

```
$ curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash
```
rbenvのdoctorというスクリプトです。
ruby野version以外がうまく行っていれば良いと思います。


####いよいよRbenvによる、Rubyのインストールです。
インストールできるRubyのバージョン一覧表示

```
$ rbenv install --list
```

```
  2.4.4
  2.4.5
  2.5.0-dev
  2.5.0-preview1
  2.5.0-rc1
  2.5.0
  2.5.1
  2.5.2
  2.5.3
  2.6.0-dev
  2.6.0-preview1
  2.6.0-preview2
  2.6.0-preview3
  2.6.0-rc1
  2.6.0-rc2
```
数字だけのを見ます。
ここから2.5.3が最新で有ることがわかります。
devとかpreview1などは、開発版やリリース前に出される開発者向けのバージョンです。

2.5.3をここではインストールします。
各自バージョンを読み替えて実行して下さい。

```
$rbenv install -v 2.5.3
```
しばらく時間がかかりますが、コーヒーでも飲みながら少し待ちましょう。
リターンキーなど途中で押さないようにしましょう。

おやエラーが出ました。

yum install -y openssl-devel readline-devel zlib-devel


最後にこのように表示されれば成功です。
```
installing rdoc:                    /home/centos/.rbenv/versions/2.5.3/share/ri/2.5.0/system
installing capi-docs:               /home/centos/.rbenv/versions/2.5.3/share/doc/ruby
Installed ruby-2.5.3 to /home/centos/.rbenv/versions/2.5.3
```

インストールの確認
今回はrbenvを利用しました。
それでrubyコマンドはこのバージョンでしか存在しませんと表示されています。

```
$ ruby -v
rbenv: ruby: command not found

The `ruby' command exists in these Ruby versions:
  2.5.3
```


#####環境確認
再読込
```
$ rbenv rehash
```

#####インストールされているruby一覧を確認

```
$ rbenv versions

//複数インストールされいているときは以下のように出ます。
* system (set by /home/vagrant/.rbenv/version)
  2.0.0-p353
  2.5.3
```

その場合はグローバルで使うバージョンを指定します。
今回は下記のように入力します

```
$ rbenv global 2.5.3
             
```

これでRubyのインストールが環境構築が終わりました。
次はいよいよRailsです。
その前に
####gemをアップデートしておきます。
Railsもgemの一つの種類だからです。

```
$ gem update

//このように表示されたら成功です
Gems updated: bigdecimal csv etc fileutils ipaddr psych rdoc stringio
```

####bundlerのインストール

```
$ gem install bundler

//このように表示されたら成功です。
Done installing documentation for bundler after 3 seconds
1 gem installed
```
bundlerでgemのインストールバージョンを管理しています。
せっかくRubyのバージョンを変更するためにrbenvを入れましたが、そのgemが同じところにインストールされると
バージョンを変更した意味がありません。
Rubyのバージョンとともにgemのバージョンも変わらないといけません。
そのためのgemのバージョン管理のためのgemがbundlerです。


これでいよいよRailsをインストールする準備が整いました。
次のレッスンでは、Railsのインストールです。
	






