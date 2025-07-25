
# アプリケーションの構造

## システム・アーキテクチャ

```{image} ../_static/dev_system_architecture_sketch.png
:alt: System Architure
:align: center
```

バリデーション・サービスは[ジャンゴ](https://www.djangoproject.com),
using [Postgres](https://www.postgresql.org) as the database,
[Redis](https://www.redis.io) for task management,
and [Celery](https://docs.celeryq.dev/en/stable/index.html) for distributing the work of running the validation tasks.
The service consists of multiple containers managed with Docker compose.

## サブモジュール

アプリケーションは3つのメインサブモジュールで構成され、それぞれが別々のGitHubリポジトリでホストされています。 Docker Composeは、ローカルデプロイ用に正しいサブモジュールのバージョンを自動的にバインドするように設定されています。

個別の機能についてのドキュメントは、各サブモジュール内にあります。

1. **ガーキンルール**バリデーションのルールを含んでいます。[リポジトリ](https://github.com/buildingSMART/ifc-gherkin-rules)そして実行する：

   ```shell
   pytest -sv
   ```

   個々のルールのデバッグは、以下のようなコマンドでサポートされている：

   ``````shell
   python test/test_main.py alb001 # For a single rule
   python test/test_main.py alb001 alb002 # For multiple rules
   python test/test_main.py path_to_separate_file.py # For a separate file
   ``````

2. **共有データモデル**この[モジュール](https://github.com/buildingSMART/ifc-validation-data-model)には、メインリポジトリと Gherkin リポジトリで共有される Django データモデルが含まれます、
serving as a submodule for both.
3. **証明書ストア**この[モジュール](https://github.com/buildingsmart-certificates/validation-service-vendor-certificates)IFCモデルに付加されたデジタル証明書を検証するための、信頼できる証明書のリストとして機能する。

注：以前は、IFC-SPFモデルの構文検証を行う4番目のサブモジュールがありました。 これは現在、IfcOpenShellの直接の一部となっています。`ifcopenshell.simple_spf`.

## バリデーション・チェックの実行

このアプリケーションは、1つまたは複数のIFCファイルに対する複数の検証チェックをサポートしており、別々に実行することができます：

- 構文チェック
- スキーマチェック
- 規範規定（ガーキン）チェック
- bSDDチェック

# 何から始めるべきか？

ワークフローに応じて、全部または一部のサービスをDocker Compose経由で実行できます。

以下は、これらのサービスをローカルで実行し、デバッグするための一般的なオプションである。
More scenario's exist - have a look at the various *make* files.

## オプション1 - Docker Compose経由で最小限のサービスセットを実行する（最も簡単に実行できる）

1. Dockerが起動していることを確認する。

2. すべてのサービスを開始する。

```shell
make start

or 

docker compose up
```

3. これは、Docker-hubイメージをプルし、ビルドして起動する。**六**異なるサービス

```
db       - PostgreSQL database
redis    - Redis instance
backend  - Django Admin + API's
worker   - Celery worker
flower   - Celery flower dashboard
frontend - React UI
```

4. 例えば、Django Admin と Celery バックグラウンドワーカー用の Django スーパーユーザアカウントを作成します：

```shell
docker exec -it backend sh

cd backend

DJANGO_SUPERUSER_USERNAME=root DJANGO_SUPERUSER_PASSWORD=root DJANGO_SUPERUSER_EMAIL=root@localhost python3 manage.py createsuperuser --noinput

DJANGO_SUPERUSER_USERNAME=SYSTEM DJANGO_SUPERUSER_PASSWORD=system DJANGO_SUPERUSER_EMAIL=system@localhost python3 manage.py createsuperuser --noinput

exit
```

5. さまざまなサービスにナビゲートする：

- バリデーション・サービス - React UI: http://localhost
- Django Admin UI: http://localhost/admin (または http://localhost:8000/admin) - デフォルトユーザ/パスワード: root/root
- Django API - Swagger: http://localhost/api/swagger-ui
- Django API - Redoc: http://localhost/api/redoc
- セロリの花 UI: http://localhost:5555

6. オプションで、curlやPostmanのようなツールを使ってAPIリクエストを直接呼び出すこともできる。

## オプション2 - ローカルデバッグ + Docker Compose経由のインフラストラクチャ（デバッグが最も簡単）

1. Dockerが起動していることを確認する。

2. インフラサービスのみを起動（Redis、Postgres、Celery Flower）

```shell
make start-infra

or

docker compose -f docker-compose.infra_only.yml up
```


3. これは**みっつ**異なるDocker-hubイメージとサービスをスピンアップする：

```
db       - PostgreSQL database
redis    - Redis instance
flower   - Celery flower dashboard
```

4. Django バックエンドの開始 (Admin + API)

```shell
cd backend
make install
make start-django
```

5. セロリ作業員の開始

```shell
cd backend
make start-worker
```

6. React UIを提供するNode Developmentサーバーを起動する。

```shell
cd frontend
npm install
npm run start
```

7. 例えば、Django Admin と Celery バックグラウンドワーカー用の Django スーパーユーザアカウントを作成します：

```shell
cd backend

DJANGO_SUPERUSER_USERNAME=root DJANGO_SUPERUSER_PASSWORD=root DJANGO_SUPERUSER_EMAIL=root@localhost python3 manage.py createsuperuser --noinput

DJANGO_SUPERUSER_USERNAME=SYSTEM DJANGO_SUPERUSER_PASSWORD=system DJANGO_SUPERUSER_EMAIL=system@localhost python3 manage.py createsuperuser --noinput
```

8. さまざまなサービスにナビゲートする：

- バリデーション・サービス - React UI: http://localhost:3000
- Django Admin UI: http://localhost:8000/admin - デフォルトユーザ/パスワード: root/root
- Django API - Swagger: http://localhost:8000/api/swagger-ui
- Django API - Redoc: http://localhost:8000/api/redoc
- セロリの花 UI: http://localhost:5555

9. オプションで、curlやPostmanのようなツールを使って、APIリクエストを直接呼び出すこともできる。