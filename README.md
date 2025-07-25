> ## ⚠️ 注意
>
> - buildingSMART が公開している **validate リポジトリ** の直下にある **`README.md`** と  
>   **`docs/` フォルダ以下の `.md` ファイル** を対象に、 **DeepL API** で機械翻訳した内容を掲載しています。  
> - **現在は試験運用中** につき、翻訳フローやドキュメント内容が今後変更される可能性があります。
> - 翻訳後の日本語リポジトリは、リポジトリ名の先頭に **`bSI_`**、末尾に **`_ja`** を付けて公開しています（例: `bSI_validate_ja`）。  
>   ※ 先頭に **`bSI_`** が付いており、末尾に **`_ja`** が付いていないリポジトリは、**翻訳せずコピーのみ** を反映したリポジトリです。  
> - 用語の整合性チェックやレビューは未実施のため、**誤訳や不自然な表現** が含まれる場合があります。  
>   正確な情報が必要な際は、必ず原文もご確認ください。


# ソフトウェア・インフラ

![イメージ](https://github.com/buildingSMART/validate/assets/155643707/5286c847-cf2a-478a-8940-fcdbd6fffeea)


# アプリケーションの構造

アプリケーションは2つのメインサブモジュールで構成され、それぞれが別々のGitHubリポジトリでホストされています。 Docker Composeは、ローカルデプロイ用に正しいサブモジュールのバージョンを自動的にバインドするように設定されています。

### サブモジュール

各機能のドキュメントは各サブモジュール内にあります。

1. **ガーキンルール**リポジトリをクローンして実行することで、独立して実行することができます：
https://github.com/buildingSMART/ifc-gherkin-rules

   ```
   pytest -sv
   ```

   個々のルールのデバッグは、以下のようなコマンドでサポートされている：

   ``````
   python test/test_main.py alb001 # For a single rule
   python test/test_main.py alb001 alb002 # For multiple rules
   python test/test_main.py path_to_separate_file.py # For a separate file
   ``````

2. **共有データモデル**: このモジュールは、メインリポジトリと Gherkin リポジトリの間で共有される Django データモデルを含み、両方のサブモジュールとして機能します。
https://github.com/buildingSMART/ifc-validation-data-model

## バリデーション・チェックの実行

このアプリケーションは、1つまたは複数のIFCファイルに対する複数の検証チェックをサポートしており、別々に実行することができます：

- 構文チェック
- スキーマチェック
- ガーキン・ルール・チェック
- bSDDチェック（無効）

# 何から始めるべきか？

ワークフローに応じて、全部または一部のサービスをDocker Compose経由で実行できます。

以下は、これらのサービスをローカルで実行し、デバッグするための一般的なオプションである。
More scenario's exist - have a look at the various *make* files.

## オプション1 - Docker Compose経由で最小限のサービスセットを実行する（最も簡単に実行できる）

1. このレポをローカルフォルダにクローンする

```shell
mkdir bsi-validate
cd bsi-validate
git clone https://github.com/buildingSMART/validate
cd validate
git checkout <branch> # if not main
make fetch-modules
```

2. Dockerが起動していることを確認する。

```shell
docker info
```

3. すべてのサービスを開始する。

```shell
make start
```
_または_ 
```
docker compose up
```

4. これは、Docker-hubイメージをプルし、ビルドして起動する。**ファイブ**異なるサービス

```
db       - PostgreSQL database
redis    - Redis instance
backend  - Django Admin + API's
worker   - Celery worker
frontend - React UI
```

5. 例えば、Django Admin と Celery バックグラウンドワーカー用の Django スーパーユーザアカウントを作成します：

```shell
docker exec -it backend sh

cd backend

DJANGO_SUPERUSER_USERNAME=root DJANGO_SUPERUSER_PASSWORD=root DJANGO_SUPERUSER_EMAIL=root@localhost python3 manage.py createsuperuser --noinput

DJANGO_SUPERUSER_USERNAME=SYSTEM DJANGO_SUPERUSER_PASSWORD=system DJANGO_SUPERUSER_EMAIL=system@localhost python3 manage.py createsuperuser --noinput

exit
```

6. さまざまなサービスにナビゲートする：

- バリデーション・サービス - React UI: http://localhost
- Django Admin UI: http://localhost/admin - 手順 5 に従ったデフォルトのユーザ/パスワード。
- Django API - Swagger: http://localhost/api/swagger-ui
- Django API - Redoc: http://localhost/api/redoc

7. オプションで、curlやPostmanのようなツールを使ってAPIリクエストを直接呼び出すこともできる。

8. アップデート後にサービスを再起動する
----------------------------------------

```shell
# 1 — Stop running containers
make stop            # or: docker compose down

# 2 — Get the latest code
make checkout        # defaults to main
#       or: make checkout BRANCH=development

# 3 — Rebuild images (if Dockerfiles or base images changed) and start
docker compose up -d --build
```
   

## オプション2 - ローカルデバッグ + Docker Compose経由のインフラストラクチャ（デバッグが最も簡単）

1. このレポをローカルフォルダにクローンする

```shell
mkdir bsi-validate
cd bsi-validate
git clone https://github.com/buildingSMART/validate 
cd validate 
git checkout <branch> # if not main
make fetch-modules
```

2. Dockerが起動していることを確認する。

```shell
docker info
```

3. インフラサービスのみを起動（Redis、Postgres、Celery Flower）

```shell
make start-infra
```
_または_
```
docker compose -f docker-compose.infra_only.yml up
```


4. これは、異なるDocker-hubイメージをプルし、スピンアップする。**みっつ**サービスを提供する：

```
db       - PostgreSQL database
redis    - Redis instance
flower   - Celery flower dashboard
```

5. Django バックエンドの開始 (Admin + API)

```shell
cd backend
make install (or make install-macos/install-macos-m1)
make start-django
```

6. セロリ作業員の開始

```shell
cd backend
make start-worker
```

7. React UIを提供するNode Developmentサーバーを起動する。

```shell
cd frontend
npm install
npm run start
```

8. 例えば、Django Admin と Celery バックグラウンドワーカー用の Django スーパーユーザアカウントを作成します：

```shell
cd backend

. .dev/venv/bin/activate

DJANGO_SUPERUSER_USERNAME=root DJANGO_SUPERUSER_PASSWORD=root DJANGO_SUPERUSER_EMAIL=root@localhost python3 manage.py createsuperuser --noinput

DJANGO_SUPERUSER_USERNAME=SYSTEM DJANGO_SUPERUSER_PASSWORD=system DJANGO_SUPERUSER_EMAIL=system@localhost python3 manage.py createsuperuser --noinput
```

9. さまざまなサービスにナビゲートする：

- バリデーション・サービス - React UI: http://localhost:3000
- Django Admin UI: http://localhost:8000/admin - 手順 8 に従ったデフォルトのユーザー/パスワード。
- Django API - Swagger: http://localhost:8000/api/swagger-ui
- Django API - Redoc: http://localhost:8000/api/redoc
- セロリの花 UI: http://localhost:5555

10. オプションで、curlやPostmanのようなツールを使ってAPIリクエストを直接呼び出すこともできる。

11. コード更新後にローカルサービスを再起動する
---------------------------------------

ローカルまたは GitHub から）コードに変更があった場合、Worker を再起動します。

### 1. ローカルサービスの停止

- バックエンドまたはフロントエンドを実行しているターミナルで、以下を押す。`Ctrl+C`
- ワーカーを潔く停止するには、新しいターミナルで実行します：

```shell
cd backend
make stop-worker
```
### 2. コードを更新する（githubから引っ張ってきた場合）
```shell
make fetch-modules && make checkout
```
### 3. サービスの再起動 
```shell
cd backend
make start-worker
make start-backend
make start-frontend    
```


