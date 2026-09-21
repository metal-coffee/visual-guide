wsl-docker-practice.md

# WSL2 + Ubuntu + Docker 練習

## Level 1：UbuntuとDocker基本操作
- WSL起動
- Linux基本確認
- aptでtreeインストール
- Dockerインストール
- Docker Engine起動
- hello-world
- nginx起動
- Windowsブラウザから確認
- docker exec
- stop / start / rm / rmi

## Level 2：自分のファイルをDockerで公開
- index.html作成
- volume mount
- nginxで表示
- ファイル変更→即反映

## Level 3：Docker Compose
- compose.yaml作成
- nginx起動
- docker compose up/down
- logs / ps

## Level 4：DB追加
- PostgreSQLコンテナ
- volume
- 環境変数
- DB接続

## Level 5：Spring Boot
- Spring Bootアプリ
- PostgreSQL接続
- Dockerfile
- Composeで一括起動

## Level 6：実務に近い構成
ブラウザ
  ↓
nginx
  ↓
Spring Boot
  ↓
PostgreSQL
