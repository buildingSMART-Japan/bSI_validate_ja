# Swarmデプロイガイド
検証サービスをDocker Swarm上にデプロイして運用するためのコマンドをコピーペーストします。

アーキテクチャの決定、既知の問題点、envファイルの戦略、gotchasについては、[swarm-considerations.mdを](swarm-considerations.md)参照のこと。

---

## デプロイ
```bash
# ビルド、レジストリへのイメージのプッシュ、デプロイ
make swarm-push ENV_FILE=<env_file>
make start-swarm-nodb ENV_FILE=<env_file>    # external DB (Azure DEV/PROD)
# または: make start-swarmENV_FILE=<env_file> # コンテナ型 DB (Hetzner)
# または: make start-swarm-localENV_FILE=<env_file> # ローカルでのテスト (NFS, ClamAVなし)
# 検証 - すべてのサービスが60秒以内に1/1になること
watch docker service ls
```

## 再デプロイ（コード変更後）
`latest`タグによるローリングアップデートはできない。

```bash
make stop-swarm
# ネットワークのクリーンアップに15秒待つ
make swarm-push ENV_FILE=<env_file>
make start-swarm-nodb ENV_FILE=<env_file>
watch docker service ls
```

単一のサービスを強制再起動する（同じイメージ、同じ環境）：
```bash
docker service update --force validate_backend
```

## ワーカーノードの追加と削除
### 前提条件
1. ワーカーVMはマネージャーと同じVNet/サブネットにいなければならない。

1. マネージャの SSH 鍵はワーカー (`~/.ssh/authorized_keys`).Azure では、Portal &gt;Reset password &gt; Add SSH public key"を使用します。

1. envファイルにワーカーを登録します：
   ```
   SWARM_WORKER_1=dev-vm-worker-1:10.0.0.4
   ```

### 追加
```bash
# Dockerのインストール、レジストリの設定、Swarmへの参加、これらすべてを1つのコマンドで実行できます
make add-worker NAME=dev-vm-worker-1 ENV_FILE=.env.DEV_SWARM
```

### 削除
```bash
# タスクの削除、スウォームからの離脱、ノードの削除
make remove-worker NAME=dev-vm-worker-1 ENV_FILE=.env.DEV_SWARM

# 次に、envファイルからSWARM_WORKER_N行を削除し、一時的であればVMを削除する
```

## スケール・ワーカー
```bash
# N個のワーカーコンテナにスケール（ノードに分散）
make scale-workers WORKERS=4

# 各ワーカーがどのノードで動作しているかを確認する
docker service ps validate_worker

# コンテナごとにリソース制限を設定する
make set-worker-limits CPU=2 MEM=2G CPU_RES=1 MEM_RES=1G
```

**用語解説** *ワーカーノードとは*VMのこと。各ノードは*ワーカーレプリカ*（コンテナ）を実行する。各レプリカは複数のCelery*プロセスを*実行します。 `CELERY_CONCURRENCY`、デフォルト4。

## モニタリング
```bash
make swarm-status                            # service overview + worker placement
docker service logs -f validate_worker       # follow logs (also: backend, frontend, scheduler)
docker stats --no-stream                     # CPU/memory per container
docker node ls                               # node health
journalctl -k | grep "out of memory"         # check for OOM kills
```

## ストップ/スタート
```bash
make stop-swarm          # removes stack, keeps volumes and Swarm membership
make start-swarm-nodb ENV_FILE=<env_file>    # redeploy — volumes are still there
```

## フルリセット
スタック、ボリューム、イメージ、Swarmをすべて削除します。初回セットアップから再スタート。

```bash
make stop-swarm
docker rm -f registry
docker volume prune -f          # WARNING: deletes DB data and uploaded files
docker system prune -af
docker swarm leave --force
```

---

## 初回セットアップ（マネージャーノード）
新しいマネージャーのための1回限りのセットアップ。設定が完了したら、上記のコマンドを日常業務に使用してください。

```bash
# 1.イニシャル・スウォーム
docker swarm init --advertise-addr <PRIVATE_IP>

# 2.ローカルレジストリの開始
docker run -d --name registry -p 5000:5000 --restart always registry:2

# 3.安全でないレジストリを設定する（マルチノードに必要）
#    "insecure-registriesを"追加します："[<PRIVATE_IP>:5000]を/etc/docker/daemon.jsonに追加する
#    それから： sudo systemctl restart docker
# 4.NFSのセットアップ
apt install -y nfs-kernel-server
mkdir -p /srv/nfs/files_data /srv/nfs/gherkin_logs
chown nobody:nogroup /srv/nfs/files_data /srv/nfs/gherkin_logs
chmod 777 /srv/nfs/files_data /srv/nfs/gherkin_logs

cat >> /etc/exports << 'EOF'
/srv/nfs/files_data  10.0.0.0/16(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/gherkin_logs 10.0.0.0/16(rw,sync,no_subtree_check,no_root_squash)
EOF

exportfs -ra && systemctl restart nfs-kernel-server

# 5..VERSIONの作成
echo "1.0.0" > .VERSION

# 6.環境ファイルを準備する - 環境ファイルの戦略についてはswarm-considerations.mdを参照
cp .env .env.myserver   # customize: PUBLIC_URL, DJANGO_ALLOWED_HOSTS, NFS_SERVER_IP, REGISTRY, etc.

# 7.サブモジュールの取得、ビルド、デプロイ
make fetch-modules
make swarm-push ENV_FILE=<env_file>
make start-swarm-nodb ENV_FILE=<env_file>
```

### Docker Composeからの移行
```bash
# 古いスタックを止める
docker compose -f docker-compose.load_balanced.nodb.yml --env-file .env.DEV down

# コンポーズボリュームからNFSにデータをコピーする（ボリューム名の違い：validation-service_ *vs validate_）*
docker run --rm -v validation-service_files_data:/src -v /srv/nfs/files_data:/dst alpine sh -c "cp -a /src/. /dst/"
docker run --rm -v validation-service_gherkin_rules_log_data:/src -v /srv/nfs/gherkin_logs:/dst alpine sh -c "cp -a /src/. /dst/"

# SSL証明書のコピー（最初のデプロイ後）
cp -a docker/frontend/letsencrypt/* /var/lib/docker/volumes/validate_letsencrypt_data/_data/
docker service update --force validate_frontend
```

---

## クイック・リファレンス
| タスク | コマンド |
|---|---|
| デプロイ（外部db） | `start-swarm-nodbを作るENV_FILE=<env_file>` |
| デプロイ（db付き） | `start-swarmを作るENV_FILE=<env_file>` |
| ストップ | `ストップスウォーム` |
| ビルド＋プッシュ | `swarm-pushを作るENV_FILE=<env_file>` |
| スケール労働者 | `スケール・ワーカーを作る WORKERS=4` |
| 制限の設定 | `set-worker-limits CPU=2 MEM=2G とする。` |
| 作業員追加 | `make add-worker NAME=<name> ENV_FILE=<env_file>` |
| 作業員の削除 | `make remove-worker NAME=<name> ENV_FILE=<env_file>` |
| ステータス | `群れの状態を作る` |
| 過去ログ | `docker service logs -f validate_<service>` |
| 強制再起動 | `docker service update --force validate_<service>` |
