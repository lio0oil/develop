# django

## 目標

- Windows10環境のVSCodeでdjango環境を構築する。
- django + uWSGI + Nginx 環境のコンテナを作成する。
- AWS Fargate で動作させる。

## ガイド

### 1.準備

以下ブランチをクローン
<https://github.com/lio0oil/django>

VSCodeでctrl+shift+@で新規ターミナル

```PowerShell
#仮想環境を作成
$ python -m venv venv
#仮想環境を起動 終わりたい時はdeactivate
$ .\venv\Scripts\activate
```

VSCodeからctrl+shift+pでコマンドパレットを開いて>python: select interpreter
.\venv\Scripts\python.exeを選択

```PowerShell
#パッケージ一括インストール
$ pip install -r .\web\requirements.txt

#実行後再起動 settings.pyのDEBUGを許可するため
$ [Environment]::SetEnvironmentVariable('DEBUG', 'True', 'User')
```

### 2.djanogoのプロジェクトを新規作成 (Option)

```PowerShell
$ rm -r src

#djangoプロジェクトの作成
$ docker-compose run web django-admin startproject config

#uwsgi用の設定ファイルをコピー
$ cp ./web/uwsgi.ini ./src/config/config/
```

プロジェクト名をconfigから変更したい場合は、以下から変更したプロジェクト名に変える。

- docker-compose.yml
- launch.json
- uwsgi.ini
- 上のcpコマンド

### 3.djanogoのアプリを新規作成 (Option)

```PowerShell
#アプリケーションの作成 プロジェクトフォルダに移動してから作成する。
$ cd .\src\config
$ python manage.py startapp polls
$ cd ..\..\
```

下記参考に開発スタート
<https://docs.djangoproject.com/ja/3.2/intro/tutorial01/#creating-the-polls-app>
F5でデバッグ開始 <http://127.0.0.1:8000/polls/> にアクセスでサンプル画面

### 4.docker

```PowerShell
#パッケージの書き出し
$ pip freeze > .\web\requirements.txt

#静的ファイルの抽出
$ rm -r static
$ python src/config/manage.py collectstatic

#dockerを起動
$ docker-compose up -d
```

<http://127.0.0.1:8080/polls/>にアクセスして動作確認

```PowerShell
#dockerを停止
$ docker-compose down
```

### 5.AWS環境準備 (Option)

AWS CLI v2のインストール
<https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/install-cliv2-windows.html>

設定ファイルと認証情報ファイルの設定
<https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-quickstart.html>

Amazon ECS CLI のインストール
<https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/ECS_CLI_installation.html>

### 6.ECR リポジトリの作成 (Option)

```PowerShell
#コンテナ毎に作成する
#XXXXXXXXXXXXはアカウントID
$ aws ecr create-repository --repository-name django-ecs-web

{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:XXXXXXXXXXXX:repository/django-ecs-web",
        "registryId": "XXXXXXXXXXXX",
        "repositoryName": "django-ecs-web",
        "repositoryUri": "XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-web",
        "createdAt": "2021-09-05T14:25:56+09:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}

$ aws ecr create-repository --repository-name django-ecs-nginx

{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-northeast-1:XXXXXXXXXXXX:repository/django-ecs-nginx",
        "registryId": "XXXXXXXXXXXX",
        "repositoryName": "django-ecs-nginx",
        "repositoryUri": "XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-nginx",
        "createdAt": "2021-09-05T14:29:15+09:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
```

### 7.イメージ作成、ECR登録

```PowerShell
#イメージ作成
$ docker build -t django-ecs-web -f ./web/Dockerfile-web .
$ docker build -t django-ecs-nginx -f ./web/Dockerfile-nginx .

#ビルドしたコンテナにタグ付け タグの値はレポジトリのrepositoryUriを付ける
$ docker tag django-ecs-web:latest XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-web
$ docker tag django-ecs-nginx:latest XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-nginx

#イメージを確認すると以下の状況
$ docker images

REPOSITORY                                                           TAG       IMAGE ID       CREATED          SIZE
836529485854.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-nginx   latest    9dc2dedb9ba0   16 minutes ago   145MB
django-ecs-nginx                                                     latest    9dc2dedb9ba0   16 minutes ago   145MB
836529485854.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-web     latest    ecc7e9c55376   20 minutes ago   1GB
django-ecs-web                                                       latest    ecc7e9c55376   20 minutes ago   1GB
django_web                                                           latest    ed816f3f5513   2 hours ago      963MB
nginx                                                                latest    822b7ec2aaf2   46 hours ago     133MB

#ECRへイメージをpushする
$ ecs-cli push XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-web:latest
$ ecs-cli push XXXXXXXXXXXX.dkr.ecr.ap-northeast-1.amazonaws.com/django-ecs-nginx:latest
```

### 8.ECS クラスターの作成(Option)

```PowerShell
$ aws ecs create-cluster --cluster-name fargate-cluster --region ap-northeast-1

{
    "cluster": {
        "clusterArn": "arn:aws:ecs:ap-northeast-1:XXXXXXXXXXXX:cluster/fargate-cluster",
        "clusterName": "fargate-cluster",
        "status": "ACTIVE",
        "registeredContainerInstancesCount": 0,
        "runningTasksCount": 0,
        "pendingTasksCount": 0,
        "activeServicesCount": 0,
        "statistics": [],
        "tags": [],
        "settings": [
            {
                "name": "containerInsights",
                "value": "disabled"
            }
        ],
        "capacityProviders": [],
        "defaultCapacityProviderStrategy": []
    }
}
```

### 9.タスク実行用のIAM ロール作成(Option)

タスク実行 IAM ロールの作成
<https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/task_execution_IAM_role.html>

### 10.タスク作成

書きかけ とりあえずコンソールから実施

fargate配下のdocker-compose.ymlのimageを修正
imageはリポジトリの作成の際のrepositoryUri

ecs-param.ymlを環境に併せて修正

## 参考

<https://qiita.com/kenkono/items/6221ad12670d1ae8b1dd>
<https://qiita.com/Brokenumbrella/items/2ce3fceadfaa3d62e06f>
<https://dev.classmethod.jp/articles/aws-reinvent-fargate-deploy-from-ecs-cli/>
<https://qiita.com/xkent/items/2fef3e57363ae17b674b>

## メモ

AWS Copilot CLI
<https://docs.aws.amazon.com/ja_jp/AmazonECS/latest/developerguide/AWS_Copilot.html>
対話形式でFargate環境が作成できる。ECS CLIの後継となっているがやれることは違う。

fargateアプリケーションをprivate subnetに配置する場合の注意点
<https://qiita.com/nyamada43/items/d3caa54443cc555e0406>
