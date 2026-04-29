# Docker Swarm - 考慮事項と既知の問題点
IVS-719開発時に作成。カテゴリー別に分類。

## ステータス
- **シングルノードSwarm**：テスト済み、動作中 (Hetzner, 2026-03-10)
- **マルチノードSwarm**：2ノード＋NFSで動作確認済み (Hetzner, 2026-03-15)
- **Azure DEV上のシングルノードSwarm**: 外部DB + NFSで動作確認 (2026-03-15)
- **Azure DEV上のマルチノードSwarm**：テスト済みで動作中 - マネージャ＋ワーカーノード、タスクは両方に分散 (2026-03-16)
- **CI/CD**：Swarmにはまだ対応していない - 5節を参照。
- **SSL/Certbot**:実際のドメインではまだテストしていません。 `CERTBOT_DOMAIN=_`スキップする)
- **ドキュメント**：ユーザー向けのドキュメント（README、デプロイメントガイド）は、Swarmワークフロー用にまだ更新されていない。

---

# 建築・デザイン
## 1.建築概要
すべてのワーカーは `/files_storage`(アップロードされたIFCファイル) と `/gherkin_logs`アクセスする必要があります。Docker Composeでは、これらは1つのマシン上のローカルボリュームです。Swarmでは、ワーカーは**異なるマシン**上で動作するため、ファイルはNFS経由で共有する必要があります。

```
                   ┌─────────┐
                   │ Frontend │  (Nginx + React)
                   │  :80/443 │
                   └────┬─────┘
                        │
                   ┌────▼─────┐
                   │ Backend  │  (Django API — manager node)
                   │  :8000   │
                   └────┬─────┘
                        │ enqueues tasks
                   ┌────▼─────┐
                   │  Redis   │  (Celery broker — manager node)
                   │  :6379   │
                   └────┬─────┘
                        │ workers consume via overlay network
              ┌─────────┼──────────┐
              │         │          │
         ┌────▼───┐ ┌───▼────┐ ┌──▼─────┐
         │Worker 1│ │Worker 2│ │Worker N│  (any node in swarm)
         └────┬───┘ └───┬────┘ └──┬─────┘
              │         │          │
              │     NFS mount      │
              └─────────┼──────────┘
                   ┌────▼─────┐
                   │/srv/nfs/ │  (NFS server on manager node)
                   │files_data│
                   └──────────┘
                        │ same machine
                   ┌────▼─────┐
                   │ Postgres │  (manager node)
                   └──────────┘

         ┌───────────┐
         │ Scheduler │  (1 replica, manager only)
         │  --beat   │  file retention: archive@90d, remove@180d
         └───────────┘
```

**どのように機能するのか：**
- **マネージャーノードは**、フロントエンド、バックエンド、DB、Redis、スケジューラー、NFSサーバーを実行する。
- **ワーカーノードは**Celeryワーカーのみを実行し、Dockerボリュームドライバ経由でNFSボリュームを自動的にマウントする。
- **オーバーレイネットワーク**（Docker Swarmネイティブ）は、マシンを越えてワーカーをRedisとPostgresに接続する
- NFSはワーカーに、アップロードされたファイルに対して、あたかもローカルであるかのような読み書きのアクセス権を与える。

**NFSが** `hard,timeo=600`マウントオプションは、 回復するまでワーカーがハングする（エラーにならない）ことを意味します。これは意図的なもので、黙って失敗するよりは待った方が良いからです。

Azureの場合：NFSエクスポートをVNet CIDR（例 `10.0.0.0/16(rw,sync,...)`制限し、`*`制限しない）。

---

## 2.ビルドとデプロイが別々のステップに
Docker Compose:`docker compose build && docker compose up`- ビルドと実行を1つのフローで。

Docker Swarm: ワーカーノードは**イメージをビルドできない**。レジストリから取得します。

```
Developer machine          Registry              Swarm nodes
     build ──push──>  localhost:5000  <──pull──  worker-1, worker-2
```

ワークフロー：
```bash
make build                          # build images locally
make swarm-push ENV_FILE=.env.xxx   # tag + push to registry
make start-swarm ENV_FILE=.env.xxx  # docker stack deploy (nodes pull from registry)
```

Azure PROD の場合は、`localhost:5000`を Azure Container Registry (ACR) に置き換えてください。

---

## 3.労働者の規模と能力
ワーカーレプリカに**ハードキャップはありません**。スケーリングは手動です：

```bash
make scale-workers WORKERS=4
```

**労働者一人当たりの能力計算：**
- ~ClamAVウイルスシグネチャデータベース用に1GBのRAM
- Celery タスク用に ~2-3GB RAM (以下による) `CELERY_CONCURRENCY`)
- 合計: **ワーカーあたり ~3-4GB RAM**
- 各ワーカーは `CELERY_CONCURRENCY`並列タスクを実行 (デフォルト: .env.hetzner に 4 個、.env に 6 個)

| 環境 | 労働者 | コンカレンシー | 並列タスク | RAMが必要（作業者のみ） |
|---|---|---|---|---|
| ヘッツナー（8GB） | 2 | 4 | 8 | ~6-8GB |
| デブ | 2 | 4 | 8 | ~6-8GB |
| プロッド | 4+ | 6 | 24+ | ~12-16GB |

1つのノードに過負荷がかかるのを防ぐには、composeファイルで `max_replicas_per_node`使う：
```yaml
deploy:
    replicas: 4
    placement:
        max_replicas_per_node: 2
```
これにより、Swarmは少なくとも2つのノードにワーカーを分散させる。現在は設定されていません。Swarmが決定すれば、すべてのレプリカを1つのノードに置くことができます。

**リソース制限は**オプションですが、本番環境では推奨されます。デプロイ後に適用する：
```bash
make set-worker-limits CPU=2 MEM=2G                        # limits only
make set-worker-limits CPU=2 MEM=2G CPU_RES=1 MEM_RES=1G   # limits + reservations
```

環境ごとの提案：
| 環境 | CPUリミット | メモリ制限 | 備考 |
|---|---|---|---|
| ヘッツナー（8GB） | 2 | 2G | 小規模サーバー、最大～2ワーカー |
| デブ | 1 | 1G |  |
| プロッド | 4 | 4G | ClamAVを含む ~1GB |

---

## 4. `.env` 戦略
`.env`は安全なデフォルト（localhost、secrets なし）でコミットされます。環境固有のファイルは、`.env.*` によって gitignored されます：

| ファイル | 目的 | コミット？ |
|---|---|---|
| `環境` | ローカル開発/フォーク用の共有デフォルト | はい |
| `.env.ヘッツナー` | ヘッツナー開発サーバー（IP、NFS、レジストリ） | いいえ |
| `.env.DEV` | DEV環境（CI/CDで使用するdocker compose） | いいえ |
| `.env.DEV_SWARM` | DEV Swarmのデプロイ（外部Azure DB、NFS） | いいえ |
| `.env.PROD` | 生産（本当の秘密、ドメイン） | いいえ |

で展開する：
```bash
make start-swarm ENV_FILE=.env.hetzner          # Hetzner (with DB container)
make start-swarm-nodb ENV_FILE=.env.DEV_SWARM   # DEV (external Azure DB)
```

Makefileは`envsubst`、envファイルから**composeレベルのバーのみ**（REGISTRY、NFS_SERVER_IP、CERTBOT_DOMAINなど）をYAMLに代入し、その結果を`docker stack deploy`パイプします。コンテナの env vars は、`docker stack deploy`が `env_file:`ディレクティブで直接読み込まれます。

**なぜcompose-level varsだけなのか**？envファイル全体をソースとする以前のアプローチは、特殊文字(`#`、`(`、スペース)を含む値で破たんしました。現在のアプローチでは、`envsubst`必要とするvars（REGISTRY、CERTBOT_DOMAIN、CERTBOT_EMAIL、NFS_SERVER_IPなど）のみを、`grep`+ Makefileの`cut`使用して抽出しています。

**Env ファイルのフォーマット規則（Swarm の env ファイルのみ -`.env.hetzner`、`.env.DEV_SWARM` など）：**
- `=` の周りには空白を入れない。`grep '^VAR=' | cut -d= -f2-`
- 値を引用符で囲まない - Dockerは値を文字通りに渡す
- のような角括弧プレースホルダはありません。 `<VALUE>`- 文字列として渡される

これにより、以前のアプローチにおける3つの問題を回避することができる：
1. **型変換のバグ**-`docker compose config` はポートを文字列に、CPUを整数に変換していたが、`docker stack deploy` はこれを拒否した。

1. **`.env` 自動ロードの衝突**-`docker compose config` は、常にプロジェクト・ディレクトリから`.env` をロードします。`--env-file`

1. **特殊文字の改行**-`#` (コメント)、`(` (サブシェル)、または引用符で囲まれていないスペースを含む値で`set -a && . ./file` 、envファイル全体をソーシングする。

---

## 5.ローカル・デベロッパーとサーバー・デプロイが異なるコンフィグになりました
あなたは2つのコンポーズファイルを別々に管理している：
- `docker-compose.yml` - ローカル開発（単一マシン、ローカルボリューム、 `container_name`)
- `docker-compose.swarm.yml` - Swarm の展開（オーバーレイネットワーク、NFS ボリューム、`deploy:` セクション）
- `docker-compose.swarm.nodb.yml` - Swarmと外部DB（コンテナ化Postgresなし）

リスク：時間の経過とともにバラバラになる（異なる環境変数、イメージバージョン、ボリューム設定）。緩和策：PR中に変更を同期させておく。

---

## 6.いいえ `container_name`/ `depends_on`スウォーム
Swarmはコンテナの命名を内部的に管理している（例えば `validate_worker.1.abc123` など）。 `depends_on`無視されます - サービスは同時に開始します。

現在の影響: 最小 - エントリーポイントはDNSサービス検出`redis`、`db`、`backend`と `pg_isready`待ちループを使用します。コードの変更は必要ありません。

---

## 7.PRODカットオーバーのためのDNS移行戦略
本番環境でDocker ComposeからSwarmに切り替える際のダウンタイムを回避するには、一時的なサブドメインを使用します：

1. 新しいサーバにSwarmスタックをデプロイする。

1. 一時的なサブドメインを指定する（例：`swarm.validate.buildingsmart.org` ）。

1. 両方のセットアップを並行して実行する。メインドメインにComposeを、テンポラリドメインにSwarmを配置する。

1. API経由のテスト（一括アップロード、同時検証）対テンポラリドメイン

1. 確信が持てたら、DNSをスワップする：メインドメインをSwarmデプロイメントに向ける

1. 古いComposeの設定を破棄する

ロールバック：Swarmに問題が発生した場合、DNSは数分で古い設定に戻る。

DEVの場合：同じアプローチか、直接カットオーバー（ユーザーと対面しないのでリスクが低い）。

---

# 既知の問題とガッチャ
## 8.オーバーレイ・ネットワークのMTUは1400に設定する必要があります
MTU（Maximum Transmission Unit）とは、ネットワークリンクが伝送できる最大のパケットサイズのことで、デフォルトは1500バイト。ヘッツナーのプライベート・ネットワークはMTU 1450を使用している。DockerのVXLANオーバーレイは、すべてのパケットに50バイトのカプセル化ヘッダを追加するので、基礎となるMTUがすでに≤1500である場合、オーバーサイズのパケットは静かにドロップされるか、断片化されます。オーバーレイのMTUを1400に設定しないと（VXLANのオーバーヘッド分のヘッドルームを残す）、異なるマシン上の**ワーカーノードはマネージャー上のサービス**（db、Redis**）に到達できない**。

症状: ワーカーが`db:5432 - no response`で止まっているDNS が正しく解決されているにもかかわらず。

修正は`docker-compose.swarm.yml`あります：
```yaml
networks:
    validate:
        driver: overlay
        driver_opts:
            com.docker.network.driver.mtu: "1400"
```

これは、内部ネットワークのMTUが1500以下のクラウド・プロバイダーにも当てはまる。

---

## 9.ClamAV は各 Worker の内部で実行されます (~1GB RAM オーバーヘッド)
各ワーカーコンテナは、独自のClamAVデーモンとfreshclam（ウイルスシグネチャのアップデータ）を起動する。これは**以前と同じ**で、Swarmの変更ではありません。しかし、N個のワーカーにスケールすると、N個の独立したClamAVインスタンスが得られます。

インパクトがある：
- ウイルスシグネチャデータベース用にワーカーあたり ~1GB RAM (Hetzner テストで確認 - 8GB サーバで 5 インスタンスが OOM を引き起こしました)
- 各ワーカーは、起動時にシグネチャのアップデートを個別にダウンロードする
- ワーカー(PROD)ごとに4GBのメモリ制限があるのはこのためです：~1GB ClamAV + Celeryタスク用に2-3GB
- ローカル・オーバーライド (`docker-compose.swarm.local.yml`) は、小規模サーバーでのテスト用に ClamAV を完全にスキップします。

ClamAVで4ワーカー＝ウイルスDBだけで4GB。

---

## 10.レジストリはローカルホストではなく、プライベートIPを使用する必要があります
**常に`REGISTRY=<manager-private-ip>:5000`** envファイルで （例 `10.0.0.5:5000`）を**設定**し、決して`localhost:5000`設定しないこと。

理由 `localhost`ローカルマシンに解決される。マネージャー上ではうまくいきます。ワーカーノードでは、`localhost:5000`何も指していません。ワーカーはイメージをプルできず、`No such image`errorsで0/Nレプリカのままです。

**すべてのノード**（マネージャとワーカー）は、`/etc/docker/daemon.json`設定された安全でないレジストリが必要です：

```bash
echo '{ "insecure-registries": ["10.0.0.5:5000"] }' | sudo tee /etc/docker/daemon.json
sudo systemctl restart docker
```

ワーカーについては`make add-worker`ターゲットが自動的に処理します。**マネージャについては**、初期セットアップ時に手動で追加してください（log-driverのような既存の`daemon.json`設定とマージしてください）。

---

## 11.Swarm で DB`postmaster.pid` が消える（コンテナ型 DB のみ）
PostgreSQLが起動し、回復し、準備が整いました：

```
could not open file "postmaster.pid": No such file or directory
performing immediate shutdown because data directory lock file is invalid
```

これはDocker Swarmのボリュームマウントのタイミングの問題です。修正方法 `restart_policy.condition: any`(`on-failure`ではない)をdbサービスに設定し、Swarmが固まるまで再起動し続けるようにした。`docker-compose.swarm.yml`既に適用されています。

---

## 12.DockerはNFSボリュームオプションをキャッシュする
`docker stack deploy`NFS ボリュームを作成する際、ドライバのオプション (`addr=` を含む) がキャッシュされます。最初のdeployで間違ったNFS IP（例えばデフォルトの`10.0.0.1`）が指定された場合、**それ以降のdeployはすべて**、たとえenvファイルを修正しても、**その間違ったIPを再利用**します。

症状：コンテナが"Created（作成中）状態に留まり、起動しない。ログがない。IPが存在しないため、NFSマウントがハングする。

修正する：
```bash
docker stack rm validate
sleep 15
docker container prune -f
docker volume rm validate_files_data validate_gherkin_rules_log_data
# コンテナがNFSマウントのハングアップで立ち往生している場合：
systemctl restart docker
# その後、再展開
make start-swarm-nodb ENV_FILE=.env.DEV_SWARM
```

デプロイ後にボリュームが正しいIPを持っていることを確認します： `docker volume inspect validate_files_data`

---

## 13.ファイルアップロード：`f.seek(0)`
`views.py` では、アップロードハンドラはファイルサイズを計測するためにファイルの最後までシークし (`f.seek(0, 2)`+`f.tell()`) 、`serializer.save()`前に巻き戻し (`f.seek(0)`) しなければなりません。巻き戻しがないと、Django はファイルポインタが末尾にあるため、0 バイトのファイルを保存します。

これは、バッファリングの動作がローカルボリュームと異なるNFSバッキングストレージでのみ発生する可能性があります。コミット `012776c`

---

## 14. `determine_aggregate_status()`サイレント・エラーを隠す
検証タスクがゼロの結果(サブプロセスのクラッシュ、ワーカーのOOM、NFSのハングなど)を出した場合、ステータスのデフォルトはVALIDになります(`models.py:1297`-`# assume valid if no outcomes - TODO: is this correct?`)。これは Swarm よりも古いものですが、ノード間でワーカーがクラッシュ/再起動したときに見えるようになります。

**なぜINVALIDを返せないのか：**ファイルを無効とマークすることは、ベンダーが調査し、修正しなければならないという現実的な結果をもたらします。クラッシュしたタスクに対してINVALIDを返すことは、偽の否定を生むことになる。実際の問題は**サイレント・エラー**です。タスクが完全に失敗しても、パスしたように見えるので誰も気づきません。

**代わりに何が起こるべきか：**ゼロの結果が生成された場合、システムはデフォルトのVALIDではなく、開発者に警告を出すべきです（例えば、エラーをログに記録したり、通知を送ったり、`ERROR`、`INCONCLUSIVE`ような明確なステータスを設定したりします）。ファイルは、有効または無効としてマークされるのではなく、再検証のためのフラグが立てられるべきです。

スウォームのブロッキングではないが、修正する価値はある。

---

## 15.dbコネクションプーリング：オーバーレイネットワーク上の古いコネクション
Django の`"pool": True`(psycopg3 コネクションプール) は、再利用のために db 接続を開いておきます。Swarm オーバーレイネットワークは、 ~13 分後にアイドル状態の TCP 接続を切断します。プールが死んだコネクションを渡すと、 Django はそれを上げます：

```
OperationalError: consuming input failed: server closed the connection unexpectedly
```

**修正**しました (`backend/core/settings.py`)：
- `"pool": False` - psycopg3の組み込み接続プールを無効にします。 `CONN_HEALTH_CHECKS`健全性チェックが通過した後、クエリに到達する前にプールが古くなった接続を渡す可能性があるためです。
- `CONN_HEALTH_CHECKS = True`- Django は接続を使う前に ping を送ります。
- `CONN_MAX_AGE = 600`(10分) - プール層なしで再利用できるようにコネクションをオープンにしておく。

`CONN_MAX_AGE`POSTGRES_CONN_env varで設定可能です。 `POSTGRES_CONN_MAX_AGE`環境変数で設定可能です。デフォルトの600sはSwarmで動作します。`0`設定すると、各リクエストの後にコネクションを閉じます（最も安全ですが、遅いです）。

症状を示すDBのログ（～13分ごと）：
```
LOG:  could not receive data from client: Connection reset by peer
```

---

## 16.SSL証明書：バインドマウント対名前付きボリューム
Docker ComposeはLet's Encryptの証明書用にバインドマウントを使用しました：`./docker/frontend/letsencrypt:/etc/letsencrypt`.Swarmは名前付きボリューム(`validate_letsencrypt_data`)を使用する。

移行時には、証明書を手動でSwarmボリュームにコピーする必要があります：
```bash
cp -a docker/frontend/letsencrypt/* /var/lib/docker/volumes/validate_letsencrypt_data/_data/
docker service update --force validate_frontend
```

これがないと、HTTPSは機能せず、サイトにはHTTPでしかアクセスできない。CERTBOT_DOMAINが設定されているので、Certbotの更新はコンテナ内部で機能し続けるはずです。 `CERTBOT_DOMAIN`が設定されているので、コンテナ内では引き続き動作するはずです。

---

## 17.スタックrm後のオーバーレイ・ネットワークの競合状態
`docker stack rm`後、オーバーレイネットワークのクリーンアップは非同期です。再デプロイが早すぎると `network validate_validate not found`エラーが発生します。

修正:`docker stack rm`と`docker stack deploy`間で ~15 秒待つ。ゴーストネットワークが残っている場合(`docker network ls`には表示されているが、`docker network rm`には"見つからないと"表示されている)、Docker を再起動してください:`systemctl restart docker`.

---

## 18.`latest` タグのローリングアップデートはない
Swarmは画像のタグが変更されたかどうかを確認してからプルします。すべての画像は`:latest`使用しているので、Swarmは"同じタグを見て、画像の内容が変更されていても、プルをスキップします。

**影響：** `docker service update --force`はコンテナを再起動するが、**キャッシュされた**イメージを使用する。新しいコードをデプロイするには、一旦取り壊して再デプロイする必要がある：

```bash
make stop-swarm
make swarm-push ENV_FILE=.env.xxx
make start-swarm ENV_FILE=.env.xxx
```

あるいは、単一サービスのためにプルを強制する：
```bash
docker service update --image localhost:5000/validationsvc-backend:latest --force validate_backend
```

---

## 19. `docker service update --force` 環境変数の再読込は行わない
`docker service update --force`は、デプロイ時と**同じ**設定でコンテナを再起動します。env ファイルは再読み込みしない。もし `.env.DEV_SWARM`変更し、その変更を有効にしたい場合は、完全な再デプロイを行う必要があります：

```bash
make stop-swarm
# 15秒待つ
make start-swarm ENV_FILE=.env.DEV_SWARM
```

---

## 20.VSコードのポート転送がSwarmのイングレスと競合する
VS CodeのSSHトンネルが、Swarmのイングレス・ルーティングと競合することがある（IPv6の問題）。VS Codeの転送ポート経由で`localhost:80`アクセスすると、うまくいかないことがあります。

**回避策**localhostの代わりにサーバーのパブリックIPを直接使用する。

---

# メンテナンス
## 21.CI/CDはまだSwarmに適応していない
現在の GitHub Actions のワークフロー (`.github/workflows/ci_cd.yml`) は、`docker compose up`DEV と PROD のデプロイに使っています。Swarm はサポートして**いません**。

Swarm CI/CDのために何を変えるべきか：
- `docker compose up` → `make start-swarm ENV_FILE=.env.XXX`(ビルド、レジストリへのプッシュ、スタックデプロイ)
- ランナー/デプロイターゲットは、Swarmマネージャにアクセスする必要がある（SSHまたはマネージャノード上のセルフホストランナー）。
- ワーカーノードがレジストリから自動的にイメージを取り込みます。
- `ENV_FILE`はすでに GitHub のアクション変数 (`${{ vars.ENV_FILE }}`) - 正しいファイルを指す必要があります。

オプション
1. **マネージャー・ノード上のセルフホスト・ランナー**- 最もシンプルで、ランナーはDockerとレジストリに直接アクセスできる。

1. **SSHデプロイステップ**- GitHubでホストされたランナーがマネージャーにSSH接続してmakeコマンドを実行する。

1. **独立したワークフロー**- Swarmデプロイメント用の新しいワークフローファイル。

開発へのマージをブロックしない - CI/CDが適応されるまで、Swarmは手動でデプロイできる。

---

## 22.DEVサーバーの定期的なクリーンアップ
> **DEV固有** - DEVサーバーのルートディスクは小さい（29GB）。より大きなディスクを持つHetzner/PRODはあまり影響を受けませんが、それでも定期的にクリーンアップする必要があります。

Dockerイメージ、ビルド・キャッシュ、オーファンボリューム、アップロードされたIFCファイルなどがどんどん溜まっていきます。定期的なクリーンアップを行わないと、ディスクはいっぱいになり、デプロイは失敗する。

**何が蓄積されるのか：**
- Dockerビルド・キャッシュ（フル・ビルド・サイクルあたり～2GB）
- 古い/未使用のイメージ（以前のデプロイメント）
- CI/CD実行で発生した孤児ボリューム（例：GitHub Actionsの`repo-clone_*` ボリューム）
- IFCファイルのアップロード量 `files_data`ボリューム(4GB以上で増加中)

**クリーンアップコマンド：**
```bash
# ディスク使用量のチェック
df -h /

# Dockerの概要
docker system df

# 未使用の画像を削除し、キャッシュを構築する
docker builder prune -af
docker image prune -af

# 孤児となったボリュームを削除する（注意：どのコンテナにもアタッチされていないボリュームのみを削除する）
docker volume prune -f

# 大きな孤児を見つけるための巻数リスト
docker system df -v | grep -A 50 "Local Volumes"
```

**推奨** `docker system prune -af`と`docker volume prune -f`を主要なデプロイサイクルごとに実行する。CI/CD パイプラインまたは cron ジョブにこれを追加することを検討してください。`/mnt`ディスク (74GB ephemeral Azure temp disk) は一時的なストレージとして使用できるが、**VM のデアロケーション/リサイズ時にデータが失われる**。

---

## 23. `makemigrations` バックエンドの起動時に実行される
`server-entrypoint.sh`は`python manage.py makemigrations`と`python manage.py migrate`をコンテナ起動毎に実行します。これは以下の理由で機能する：
- バックエンドはマネージャーノード上の**1レプリカに**制約される - マイグレーションの競合状態は発生しない
- 生成されたマイグレーション・ファイルはコンテナ内に保存される（エフェメラル）。

**リスク：**マイグレーションファイルとしてコミットされていないモデル変更が存在する場合、`makemigrations`はコンテナ内で実行時にそれらを生成します。これらのマイグレーションはコンテナが再起動すると消えてしまうため、不整合が発生する可能性があります。本番環境では、マイグレーションはビルド時にイメージにベイクする必要があります。

**決定：**今のところそのまま。バックエンドは常に1レプリカで、実際にはすべてのマイグレーションはデプロイ前にgitにコミットされる。しかし、PRODのハードニングのために再検討する価値はある。

---

## 24.歴史的な大群の不安定性
"&gt; 原因不明の墜落事故／腐敗した状態（5年以上前）-今はなくなっていることを望む"

最新の Docker Engine (24+) は安定しているはずです。緩和策はすでにある：
- `CELERY_TASK_ACKS_LATE = True`- タスクは完了するまでキューに留まる
- `CELERY_TASK_REJECT_ON_WORKER_LOST = True`- クラッシュしたタスクは再キューされる
- `restart_policy: condition: any`DBについて（セクション11参照）、`on-failure` その他のサービスについて
- `update_config: failure_action: rollback`- ロールバック

---

# ローカル開発のみ
## 25.リマ特有：ヴィルティオフ＋セロリ・プレフォーク＝エラーノ35
Celery の`prefork`pool + Lima の virtiofs read-only mounts が`EDEADLK`デッドロックを引き起こす。回避策 `--pool=solo`。

**本番環境での問題ではありません**- Limaを使用したmacOS上のローカル開発にのみ影響します。Linux上のDockerコンテナは、適切なext4/overlay2ファイルシステムを使用します。

---

## 26. macOS NFSの問題点:`/tmp` vs.`/private/tmp`
macOSでは、`/tmp`、`/private/tmp`シンボリックリンクです。NFSエクスポートでは、実際のパス`/private/tmp/...`を使用する必要があります。Linuxサーバー（Hetzner/Azure）には関係ないが、macOSのローカル開発には関係する。
