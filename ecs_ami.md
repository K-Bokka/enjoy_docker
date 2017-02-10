# Docker Hands-on

1. Amazon linux 64bitの適当なEC2を作成する

1. SSHで接続後、AWS CLIの確認とDockerのインストール
<pre>
$ sudo yum update -y  # yumのupdate
$ aws --version	      # aws cliのバージョン確認
aws-cli/1.11.29 Python/2.7.12 Linux/4.4.41-36.55.amzn1.x86_64 botocore/1.4.86
$ sudo yum install -y docker # Dockerのインストール
$ sudo service docker start  # Dockerサービスの登録と開始
$ sudo usermod -a -G docker ec2-user # ec2-userにdockerコマンドの実行権限を付与
$ exit   # 一旦ログオフ
</pre>
1. 再接続してDockerコマンドの確認
<pre>
$ docker search centos # Docker Hub でイメージの検索。ここではcentosのイメージを検索している
NAME                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                                 The official build of CentOS.                   3074      [OK]
jdeathe/centos-ssh                     CentOS-6 6.8 x86_64 / CentOS-7 7.3.1611 x8...   59                   [OK]
</pre>
1. Docker Hubとの連携
	1. [Docker Hub](https://hub.docker.com/) にサインアップ
	1. Docker Hubへのログイン
<pre>
$ docker login -u YOUR_DOCKER_ID  # Docker Hubへのログイン
Password:    # パスワードを入力
Login Succeeded
</pre>
	1. Docker Hub を使ってデモンストレーション
		1. demo用のソースをダウンロード
<pre>
$ sudo yum install -y git  # gitのインストール
$ git clone https://github.com/awslabs/ecs-demo-php-simple-app # demo用のリポジトリをクローン
$ cd ecs-demo-php-simple-app # クローンしたリポジトリの中に移動
$ cat Dockerfile  # Dockerfileの中身を確認
</pre>
		1. Dockerイメージの作成
<pre>
$ docker build -t YOUR_DOCKER_ID/amazon-ecs-sample .   # Docker Imageの作成
$ docker images
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
YOUR_DOCKER_ID/amazon-ecs-sample   latest              dffc995eacd3        2 minutes ago       228.9 MB
ubuntu                             12.04               f0d07a756afd        7 weeks ago         103.6 MB
</pre>
		1. Dockerの実行
<pre>
$ docker run -p 80:80 YOUR_DOCKER_ID/amazon-ecs-sample # hostとcontainerの80番を繋げて起動
</pre>
		1. Public DNSにブラウザからアクセスして、"Simple PHP App"が表示されたらOK。停止はctrl+c。
		1. Docker Hubに登録
<pre>
$ docker push YOUR_DOCKER_ID/amazon-ecs-sample
</pre>
		1. Docker Hubにログインして、pushしたイメージが確認できれば成功
1. Amazon EC2 Container Searviceとの連携
	1. Amazon EC2 Container Service を開始して、 **実行すること** のチェックが2つとも入っている事を確認して **続行**
	1. リポジトリ名を適当につけて **次のステップ** へ
	1. コマンドだけが書かれている謎のページに移動するので、EC2インスタンス上で以下の手順を実行後、 **次のステップ** へ
		1. Docker login コマンドを取得して実行
<pre>
$ \`aws ecr get-login --region ap-northeast-1\`
Flag --email has been deprecated, will be removed in 1.13.
Login Succeeded
</pre>
		1. ECSのリポジトリにイメージをプッシュ
<pre>
$ docker images  # 現在のDocker imageの確認
REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
YOUR_DOCKER_ID/amazon-ecs-sample   latest              dffc995eacd3        2 hours ago         228.9 MB
ubuntu                             12.04               f0d07a756afd        7 weeks ago         103.6 MB
$ docker tag YOUR_DOCKER_ID/amazon-ecs-sample:latest XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/YOUR_REPOGITORY_NAME:latest  # Docker Image に新しいタグをつける
$ docker images  # 新しいタグの確認
REPOSITORY                                                                   TAG                 IMAGE ID            CREATED             SIZE
XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/YOUR_REPOGITORY_NAME       latest              dffc995eacd3        2 hours ago         228.9 MB
YOUR_DOCKER_ID/amazon-ecs-sample                                             latest              dffc995eacd3        2 hours ago         228.9 MB
ubuntu                                                                       12.04               f0d07a756afd        7 weeks ago         103.6 MB
$ docker push XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/YOUR_REPOGITORY_NAME  # ECSにイメージをプッシュ
</pre>
	1. プッシュしたイメージの情報が表示されるので **次のステップ** へ
	1. サービスの設定情報が表示されるので **次のステップ** へ
	1. クラスターの設定が表示されるので **レビューと起動** へ
	1. 確認画面が表示されるので **インスタンスの起動とサービスの実行** へ
	1. 新しいEC2インスタンスが作成されるので、Public DNSにブラウザからアクセスして、"Simple PHP App"が表示されたらOK。
	1. 最後にクラスタを削除しておく。お金もったいないし
1. Dockerで遊ぶ
	1. Docker をバックグラウンドで起動して、コンテナ上のファイルを書き換えて遊ぶ
<pre>
$ docker imges     # docker image の確認
$ docker run -d -p 80:80 YOUR_DOCKER_ID/amazon-ecs-sample  # Dockerをデタッチモードで起動
$ docker ps        # 起動中のContainerの確認
$ docker ps -l -q  # 最後に作成したコンテナのIDだけを返す
$ docker exec -i -t \`docker ps -l -q\` /bin/bash   # Container上でシェルを実行する。iは対話モードでtは仮想端末を使用するというオプション
\# cd /var/www        # index.phpがあるディレクトリに移動
\# cat index.php      # 中身を確認
\# sed -i -e "s/Simple/Hard/g" index.php  # viは使えないので、無理やりコマンドでファイルを書き換える
\# cat index.php      # 中身を確認
\# exit               # ログアウト
</pre>
Public DNSにブラウザからアクセスして、"Hard PHP App"が表示されたらOK。
	1. Docker を止めて上げて遊ぶ
		1. とりあえず止めてみる
<pre>
$ docker stop \`docker ps -lq\`  # 最後に作成したContainerの停止
$ docker ps -a   # コンテナを全て表示
CONTAINER ID        IMAGE                              COMMAND                  CREATED             STATUS                         PORTS               NAMES
192c2a2aba04        YOUR_DOCKER_ID/amazon-ecs-sample   "/usr/sbin/apache2 -D"   18 minutes ago      Exited (0) 11 seconds ago                          gigantic_jennings
</pre>
		1. ブラウザでアクセスすると見えない
		1. 起動してみる
<pre>
$ docker start 192c2a2aba04  # 停止したコンテナをスタート
</pre>
		1. ブラウザでアクセスすると"Hard PHP App"が表示される
		1. 停止して再度コンテナを作りなおしてみる
<pre>
$ docker stop 192c2a2aba04  # スタートしたコンテナを停止
$ docker run -d -p 80:80 YOUR_DOCKER_ID/amazon-ecs-sample  # Dockerをデタッチモードで起動
</pre>
		1. ブラウザからアクセスすると、今度は"Simple PHP App"が表示される
		1. もっかい停止して溜まったコンテナを削除してみる
<pre>
$ docker stop \`docker ps -q\`   # 稼働中の全てのコンテナを停止
$ docker rm \`docker ps -aq\`     # コンテナ全てを削除
</pre>
		1. ポートバインディング＋対話方式仮想ターミナル＋クリーンナップのオプションを使用してシェルをフォアグラウンドで起動
<pre>
$ docker run -p 8080:80 -it --rm akyamamoto/amazon-ecs-sample /bin/bash
\# cat /var/www/index.php  # index.phpの内容を確認
</pre>
		1. 別のSSHで接続して状態を確認する
<pre>
$ docker ps
CONTAINER ID        IMAGE                              COMMAND             CREATED             STATUS              PORTS                  NAMES
e1d4c9970ffd        YOUR_DOCKER_ID/amazon-ecs-sample   "/bin/bash"         3 minutes ago       Up 3 minutes        0.0.0.0:8080->80/tcp   loving_swanson
</pre>

