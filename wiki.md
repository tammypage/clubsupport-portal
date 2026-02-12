## ClubSupport の開発・運用に関するナレッジを集約しています。

### Dockerfile
使用するイメージや追加モジュールを定義します。

* ベースイメージ:

    * Apacheが必要なため `php:8.4-apache` ベース等に変更・調整します。
    * Xdebug、`vscode`ユーザを追加します。

### devcontainer.json
VSCodeとコンテナの連携設定を記述します。

* extensions: コンテナ内で利用する拡張機能（Xdebugなど）をここに記述します。
* portsAttributes: 必要に応じてポート転送設定を行います（Docker側で設定済みの場合はVSCodeの自動転送を抑制します）。
* remoteUser / updateRemoteUserUID: ホストOSに合わせて以下のように設定します。
    * Mac / Windows / Chromebook (Rootful): `"remoteUser": "vscode"`, `"updateRemoteUserUID": true`
    * Linux (Rootless): `"remoteUser": "root"`, `"updateRemoteUserUID": false`

### docker-compose.yml
コンテナの構成を定義します。

* volumes: プロジェクトの親フォルダ（ワークスペース）をコンテナ内のドキュメントルート（例: `/var/www/html`）にマウントします。
* データベースとなる`mariadb:10`を同時に起動します。

## launch.json
デバッガーの設定を記述します。

1. port: Xdebugのデフォルトである `9003` を指定します。
1. pathMappings: `docker-compose.yml` でワークスペースをそのままマウントしている場合、デフォルト設定（`${workspaceFolder}` = `/var/www/html`）で解決されるため、基本的には追記不要です。

## DBバックアップ

`mysqldump --skip-ssl -h db -u tmc_clubsupport -p tmc_clubsupport > "tmc_clubsup-$(date +%Y-%m-%d_%H).sql"`

## DDL作成

`mysqldump --skip-ssl  -h db -u tmc_clubsupport -p --no-data --routines tmc_clubsupport > db/schema_full.sql`

## DBリセット

1. コンテナ停止
    `docker stop clubsupport_devcontainer-db-1`
1. コンテナ削除
    `docker rm clubsupport_devcontainer-db-1`
1. ボリューム削除
    `docker volume rm clubsupport_devcontainer_db-data`
1. データインポート
    1. テーブルのみ登録
        `mysql -h db -u tmc_clubsupport -p --skip-ssl < db/schema_full.sql`
    1. データ含めて復元
        `mysql -h db -u tmc_clubsupport -p --skip-ssl < tmc_clubsup-YYYY-mm-DD_番号.sql`
       
※ コンテナ名やボリューム名は適宜、`docker ps -a`や`docker volume ls`で確認して下さい。