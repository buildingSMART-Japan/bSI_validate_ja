
# アプリケーションの構造
## システム・アーキテクチャ
```{image} ../_static/dev_system_architecture_sketch.png
:alt: System Architure
:align: center
```

バリデーションサービスは[django](https://www.djangoproject.com) で構築されています、  
データベースとして[Postgresを](https://www.postgresql.org)使用している、  
タスク管理のための[Redis](https://www.redis.io)、  
と[Celeryを](https://docs.celeryq.dev/en/stable/index.html)使い、検証タスクの実行作業を分散させた。  
このサービスは、Docker composeで管理される複数のコンテナで構成されている。

## サブモジュール
アプリケーションは3つのメインサブモジュールで構成され、それぞれが別々のGitHubリポジトリでホストされています。Docker Composeは、ローカルのデプロイ用に正しいサブモジュールのバージョンを自動的にバインドするように設定されています。

個別の機能についてのドキュメントは、各サブモジュール内にあります。

1. **ガーキンルール**検証ルールを含みます。[リポジトリを](https://github.com/buildingSMART/ifc-gherkin-rules)クローンして実行することで独立して実行できます：

   ```shell
   pytest -sv
   ```

   個々のルールのデバッグは、以下のようなコマンドでサポートされている：

   ``````shell
   python test/test_main.py alb001 # For a single rule
   python test/test_main.py alb001 alb002 # For multiple rules
   python test/test_main.py path_to_separate_file.py # For a separate file
   ``````

1. **共有データモデル**：この[モジュールには](https://github.com/buildingSMART/ifc-validation-data-model)、メインリポジトリと Gherkin リポジトリで共有される Django データモデルが含まれます、  
のサブモジュールとして機能する。
1. **証明書ストア**：この[モジュールは](https://github.com/buildingsmart-certificates/validation-service-vendor-certificates)、IFC モデルに付加されたデジタル証明書を検証するための、信頼できる証明書のリストとして機能する。

注：以前は、IFC-SPFモデルのシンタックス検証を行う4番目のサブモジュールがあった。これは現在、IfcOpenShell の一部として直接組み込まれています。 `ifcopenshell.simple_spf`.

## バリデーション・チェックの実行
このアプリケーションは、1つまたは複数のIFC ファイルに対して、別々に実行できる複数の検証チェックをサポートしています：

- 構文チェック
- スキーマチェック
- 規範規定（ガーキン）チェック
- bSDDチェック

# 何から始めるべきか？
ワークフローに応じて、全部または一部のサービスをDocker Compose経由で実行できます。

以下は、これらのサービスをローカルで実行し、デバッグするための一般的なオプションである。  
もっと多くのシナリオがある。

## オプション1 - Docker Compose経由で最小限のサービスセットを実行する（最も簡単に実行できる）
1. Dockerが起動していることを確認する。


1. すべてのサービスを開始する。

```shell
make start

or 

docker compose up
```

1. これはDocker-hubイメージをプルし、ビルドし、**6つの**異なるサービスを立ち上げる：

```
db       - PostgreSQL database
redis    - Redis instance
backend  - Django Admin + API's
worker   - Celery worker
flower   - Celery flower dashboard
frontend - React UI
```

1. 例えば、Django Admin と Celery バックグラウンドワーカー用の Django スーパーユーザアカウントを作成します：

```shell
docker exec -it backend sh

cd backend

DJANGO_SUPERUSER_USERNAME=root DJANGO_SUPERUSER_PASSWORD=root DJANGO_SUPERUSER_EMAIL=root@localhost python3 manage.py createsuperuser --noinput

DJANGO_SUPERUSER_USERNAME=SYSTEM DJANGO_SUPERUSER_PASSWORD=system DJANGO_SUPERUSER_EMAIL=system@localhost python3 manage.py createsuperuser --noinput

exit
```

1. さまざまなサービスにナビゲートする：

- バリデーション・サービス - React UI: http://localhost
- Django Admin UI: http://localhost/admin (または http://localhost:8000/admin) - デフォルトユーザ/パスワード: root/root
- Django API - Swagger: http://localhost/api/swagger-ui
- Django API - Redoc: http://localhost/api/redoc
- セロリの花 UI: http://localhost:5555

1. オプションで、curlやPostmanのようなツールを使って、APIリクエストを直接呼び出すこともできる。

## オプション2 - ローカルデバッグ + Docker Compose経由のインフラストラクチャ（デバッグが最も簡単）
1. Dockerが起動していることを確認する。


1. インフラサービスのみを起動（Redis、Postgres、Celery Flower）

```shell
make start-infra

or

docker compose -f docker-compose.infra_only.yml up
```


1. これは**3つの**異なるDocker-hubイメージをプルし、サービスを立ち上げる：

```
db       - PostgreSQL database
redis    - Redis instance
flower   - Celery flower dashboard
```

1. Django バックエンドの開始 (Admin + API)

```shell
cd backend
make install
make start-django
```

1. セロリ作業員の開始

```shell
cd backend
make start-worker
```

1. React UIを提供するNode Developmentサーバーを起動する。

```shell
cd frontend
npm install
npm run start
```

1. 例えば、Django Admin と Celery バックグラウンドワーカー用の Django スーパーユーザアカウントを作成します：

```shell
cd backend

DJANGO_SUPERUSER_USERNAME=root DJANGO_SUPERUSER_PASSWORD=root DJANGO_SUPERUSER_EMAIL=root@localhost python3 manage.py createsuperuser --noinput

DJANGO_SUPERUSER_USERNAME=SYSTEM DJANGO_SUPERUSER_PASSWORD=system DJANGO_SUPERUSER_EMAIL=system@localhost python3 manage.py createsuperuser --noinput
```

1. さまざまなサービスにナビゲートする：

- バリデーション・サービス - React UI: http://localhost:3000
- Django Admin UI: http://localhost:8000/admin - デフォルトユーザ/パスワード: root/root
- Django API - Swagger: http://localhost:8000/api/swagger-ui
- Django API - Redoc: http://localhost:8000/api/redoc
- セロリの花 UI: http://localhost:5555

1. オプションで、curlやPostmanのようなツールを使って、APIリクエストを直接呼び出すこともできる。