# WSL2 + Ubuntu + Docker 練習帳

このファイルは、WSL2 上の Ubuntu で Linux 操作と Docker の基本操作を繰り返し練習するためのハンズオン手順書です。

---

# Level 1：Ubuntu と Docker の基本操作

## 1. この Level の目的

この Level では、次の流れを一通り体験します。

```text
Windows
  ↓
WSL2
  ↓
Ubuntu
  ↓
Linux 基本操作
  ↓
apt でパッケージ導入
  ↓
Docker Engine
  ↓
Docker コンテナ
  ↓
nginx
  ↓
Windows のブラウザから確認
```

この Level を終えると、次の操作を自分でできる状態を目指します。

- WSL2 の Ubuntu を起動する
- Ubuntu の基本情報を確認する
- `apt` でパッケージをインストールする
- Docker をインストールする
- Docker Engine を起動する
- Docker コンテナを起動する
- `hello-world` で Docker の動作確認をする
- nginx コンテナを起動する
- Windows のブラウザから nginx にアクセスする
- コンテナの中に入る
- コンテナを停止・削除する
- Docker イメージを削除する

---

## 2. 前提

この手順では、次の状態を前提とします。

- Windows に WSL2 が導入済み
- WSL2 上に Ubuntu が導入済み
- Ubuntu にログインできる

Windows 側の PowerShell で、次のコマンドを実行します。

```powershell
wsl
```

複数の WSL ディストリビューションがある場合は、一覧を確認できます。

```powershell
wsl -l -v
```

Ubuntu を指定して起動する場合の例です。

```powershell
wsl -d Ubuntu
```

---

## 3. Ubuntu に入ったことを確認する

まず、現在のユーザーを確認します。

```bash
whoami
```

### 確認ポイント

現在ログインしている Ubuntu ユーザー名が表示されれば OK です。

---

現在のディレクトリを確認します。

```bash
pwd
```

通常は次のようなホームディレクトリが表示されます。

```text
/home/<ユーザー名>
```

---

OS 情報を確認します。

```bash
cat /etc/os-release
```

Ubuntu のバージョン情報が表示されます。

---

カーネル情報も確認します。

```bash
uname -a
```

WSL2 上の Linux カーネル情報が表示されます。

---

## 4. Linux の基本操作を少し試す

現在のディレクトリの中身を確認します。

```bash
ls
```

詳細表示します。

```bash
ls -la
```

練習用ディレクトリを作成します。

```bash
mkdir -p ~/wsl-lab/docker
```

移動します。

```bash
cd ~/wsl-lab
```

現在位置を確認します。

```bash
pwd
```

---

## 5. apt でパッケージをインストールする

Ubuntu のパッケージ情報を更新します。

```bash
sudo apt update
```

### 補足

`apt update` は、インストール済みパッケージそのものを更新するコマンドではありません。

取得できるパッケージの一覧情報を最新化します。

```text
Ubuntu
  ↓
apt update
  ↓
パッケージ一覧情報を更新
```

---

今回は練習として `tree` をインストールします。

```bash
sudo apt install -y tree
```

インストールされたか確認します。

```bash
tree --version
```

現在の練習用ディレクトリを表示してみます。

```bash
cd ~/wsl-lab
tree
```

想定例：

```text
.
└── docker
```

---

## 6. Docker をインストールする

Ubuntu のパッケージから Docker をインストールします。

```bash
sudo apt install -y docker.io
```

インストール確認をします。

```bash
docker --version
```

想定例：

```text
Docker version xx.x.x, build xxxxxxx
```

---

## 7. Docker Engine を起動する

Docker は、単に `docker` コマンドがあるだけではコンテナを動かせません。

内部では Docker Engine という常駐プロセスが動作します。

```text
docker コマンド
      ↓
Docker Engine
      ↓
Docker コンテナ
```

まず Docker Engine を起動します。

```bash
sudo systemctl start docker
```

状態を確認します。

```bash
sudo systemctl status docker
```

### 成功時の確認ポイント

次のように表示されていれば OK です。

```text
active (running)
```

`systemctl status` の画面を終了する場合は、通常 `q` を押します。

---

### systemctl が使えない場合

WSL 側で systemd が有効になっていない場合、次のようなエラーになることがあります。

```text
System has not been booted with systemd
```

その場合は次を試します。

```bash
sudo service docker start
```

その後、Docker が動作しているか次の手順で確認します。

---

## 8. hello-world コンテナを起動する

Docker の基本動作確認として、公式の `hello-world` イメージを使用します。

```bash
sudo docker run hello-world
```

成功すると、次のようなメッセージが表示されます。

```text
Hello from Docker!
```

このとき内部では、おおむね次の処理が行われています。

```text
docker run hello-world
        ↓
ローカルに hello-world イメージがあるか確認
        ↓
なければ Docker Hub から取得
        ↓
コンテナを作成
        ↓
コンテナを起動
        ↓
メッセージを表示
        ↓
コンテナ終了
```

---

## 9. Docker イメージを確認する

Docker イメージの一覧を確認します。

```bash
sudo docker images
```

`hello-world` が表示されれば OK です。

### イメージとは

Docker イメージは、コンテナを作るための元データです。

```text
Docker Image
    ↓
docker run
    ↓
Docker Container
```

---

## 10. Docker コンテナを確認する

現在動作中のコンテナを確認します。

```bash
sudo docker ps
```

`hello-world` はすでに終了しているため、通常ここには表示されません。

停止済みコンテナも含めて表示します。

```bash
sudo docker ps -a
```

`hello-world` のコンテナが表示されれば OK です。

### ポイント

```text
docker images
    ↓
コンテナを作るための元

docker ps -a
    ↓
実際に作成されたコンテナ
```

---

# 11. nginx コンテナを起動する

ここからは、実際に Web サーバを起動します。

nginx のコンテナをバックグラウンドで起動します。

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  nginx:alpine
```

各オプションの意味です。

| 指定 | 意味 |
|---|---|
| `docker run` | コンテナを作成して起動する |
| `-d` | バックグラウンドで起動する |
| `--name my-nginx` | コンテナ名を `my-nginx` にする |
| `-p 8080:80` | Ubuntu 側の 8080 番ポートをコンテナの 80 番ポートへ接続する |
| `nginx:alpine` | nginx の軽量 Alpine Linux 版イメージを使用する |

---

## 12. nginx コンテナが起動していることを確認する

```bash
sudo docker ps
```

想定例：

```text
CONTAINER ID   IMAGE          PORTS
xxxxxxxxxxxx   nginx:alpine   0.0.0.0:8080->80/tcp
```

ここで重要なのは次の部分です。

```text
0.0.0.0:8080->80/tcp
```

意味は次のとおりです。

```text
Ubuntu 側
TCP 8080
   ↓
Docker
   ↓
nginx コンテナ
TCP 80
```

---

## 13. Ubuntu から nginx にアクセスする

Ubuntu 上から HTTP アクセスします。

```bash
curl http://localhost:8080
```

HTML が表示されれば成功です。

内容の中に次のような文字列が含まれます。

```text
Welcome to nginx!
```

---

## 14. Windows のブラウザから確認する

Windows 側の Chrome、Edge などで次の URL を開きます。

```text
http://localhost:8080
```

nginx の標準画面が表示されれば成功です。

```text
Welcome to nginx!
```

このときの通信イメージです。

```text
Windows
  │
  │ http://localhost:8080
  ▼
WSL2
  │
Ubuntu :8080
  │
  ▼
Docker Engine
  │
  ▼
nginx container :80
```

---

# 15. nginx コンテナの中に入る

Docker コンテナ内部に入ってみます。

```bash
sudo docker exec -it my-nginx sh
```

プロンプトが変われば、コンテナ内部に入っています。

この状態は次のイメージです。

```text
Windows
  ↓
WSL2
  ↓
Ubuntu
  ↓
Docker
  ↓
nginx コンテナ
  ↓
コンテナ内部の Linux
```

---

## 16. コンテナ内部を確認する

OS 情報を確認します。

```sh
cat /etc/os-release
```

nginx の `alpine` イメージを使用しているため、Ubuntu ではなく Alpine Linux と表示されるはずです。

ファイル一覧も見てみます。

```sh
ls
```

プロセスを確認します。

```sh
ps
```

可能であれば nginx 関連のプロセスが動作していることも確認します。

---

## 17. コンテナから出る

コンテナ内部から Ubuntu 側へ戻ります。

```sh
exit
```

その後、Ubuntu 側で確認します。

```bash
pwd
```

---

# 18. nginx コンテナを停止する

現在動作中のコンテナを確認します。

```bash
sudo docker ps
```

nginx を停止します。

```bash
sudo docker stop my-nginx
```

もう一度確認します。

```bash
sudo docker ps
```

`my-nginx` が表示されなくなれば停止しています。

---

## 19. 停止済みコンテナを確認する

```bash
sudo docker ps -a
```

停止していても、コンテナ自体は残っています。

```text
docker stop
    ↓
コンテナ停止
    ↓
コンテナ自体は残る
```

---

## 20. nginx コンテナを再起動する

停止したコンテナは再び起動できます。

```bash
sudo docker start my-nginx
```

確認します。

```bash
sudo docker ps
```

再度ブラウザから次へアクセスします。

```text
http://localhost:8080
```

nginx の画面が表示されれば OK です。

---

## 21. nginx コンテナをもう一度停止する

削除する前に停止します。

```bash
sudo docker stop my-nginx
```

---

## 22. nginx コンテナを削除する

```bash
sudo docker rm my-nginx
```

確認します。

```bash
sudo docker ps -a
```

`my-nginx` が表示されなければ削除完了です。

---

# 23. Docker イメージを確認する

```bash
sudo docker images
```

`nginx:alpine` イメージは、コンテナを削除しても残っています。

```text
コンテナ削除
    ↓
イメージは残る
```

これは、次回同じイメージからコンテナを作る際に再利用できるためです。

---

## 24. nginx イメージを削除する

完全に片付けたい場合は nginx イメージも削除します。

```bash
sudo docker rmi nginx:alpine
```

確認します。

```bash
sudo docker images
```

`nginx` がなくなっていれば削除完了です。

---

# 25. hello-world コンテナを削除する

`hello-world` の終了済みコンテナを確認します。

```bash
sudo docker ps -a
```

CONTAINER ID またはコンテナ名を確認し、削除します。

例：

```bash
sudo docker rm <CONTAINER_ID>
```

終了済みコンテナをまとめて削除する方法もあります。

```bash
sudo docker container prune
```

実行時に確認を求められます。

---

# 26. hello-world イメージを削除する

必要であればイメージも削除します。

```bash
sudo docker rmi hello-world
```

確認します。

```bash
sudo docker images
```

---

# 27. Docker Engine を停止する

Docker 自体も停止してみます。

```bash
sudo systemctl stop docker
```

状態確認：

```bash
sudo systemctl status docker
```

systemd を使用していない場合：

```bash
sudo service docker stop
```

---

# 28. WSL から抜ける

Ubuntu のシェルを終了します。

```bash
exit
```

Windows の PowerShell に戻ります。

WSL 全体を完全に停止したい場合は、PowerShell で次を実行します。

```powershell
wsl --shutdown
```

---

# 29. 今回覚えたい Docker コマンド

| コマンド | 意味 |
|---|---|
| `docker --version` | Docker バージョン確認 |
| `docker images` | Docker イメージ一覧 |
| `docker ps` | 起動中コンテナ一覧 |
| `docker ps -a` | 停止済みを含むコンテナ一覧 |
| `docker run` | コンテナ作成・起動 |
| `docker exec` | 起動中コンテナ内でコマンド実行 |
| `docker stop` | コンテナ停止 |
| `docker start` | 停止中コンテナ起動 |
| `docker rm` | コンテナ削除 |
| `docker rmi` | イメージ削除 |

---

# 30. 今回覚えたい Linux コマンド

| コマンド | 意味 |
|---|---|
| `whoami` | 現在のユーザー確認 |
| `pwd` | 現在ディレクトリ確認 |
| `ls` | ファイル一覧 |
| `ls -la` | 隠しファイルを含め詳細表示 |
| `cd` | ディレクトリ移動 |
| `mkdir` | ディレクトリ作成 |
| `cat` | ファイル内容表示 |
| `sudo` | 管理者権限で実行 |
| `apt update` | パッケージ一覧情報更新 |
| `apt install` | パッケージインストール |
| `curl` | HTTP などでデータ取得 |
| `exit` | シェル終了 |

---

# 31. Level 1 の重要ポイント

## Docker Image と Container

```text
Docker Image
   │
   │ docker run
   ▼
Docker Container
```

イメージはコンテナの元です。

1つのイメージから複数のコンテナを作ることもできます。

---

## docker run と docker start の違い

### docker run

```text
イメージ
  ↓
新しいコンテナを作成
  ↓
起動
```

### docker start

```text
既に存在する停止中コンテナ
  ↓
再起動
```

---

## docker stop と docker rm の違い

### docker stop

コンテナを停止するだけです。

```text
Container
   ↓
停止
   ↓
残っている
```

### docker rm

コンテナそのものを削除します。

```text
Container
   ↓
削除
```

---

# 32. 繰り返し練習するときの短縮版

一度理解した後は、次の手順だけでも Level 1 を復習できます。

## Ubuntu 起動

PowerShell：

```powershell
wsl
```

## Docker 起動

```bash
sudo systemctl start docker
```

または：

```bash
sudo service docker start
```

## hello-world

```bash
sudo docker run hello-world
```

## nginx 起動

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  nginx:alpine
```

## 確認

```bash
sudo docker ps
curl http://localhost:8080
```

Windows：

```text
http://localhost:8080
```

## コンテナへ入る

```bash
sudo docker exec -it my-nginx sh
```

終了：

```sh
exit
```

## 停止

```bash
sudo docker stop my-nginx
```

## 再起動

```bash
sudo docker start my-nginx
```

## 削除

```bash
sudo docker stop my-nginx
sudo docker rm my-nginx
sudo docker rmi nginx:alpine
```

## WSL 終了

```bash
exit
```

PowerShell：

```powershell
wsl --shutdown
```

---

# 33. Level 1 完了チェックリスト

以下を自分で説明・実行できれば Level 1 完了です。

- [ ] WSL2 の Ubuntu を起動できる
- [ ] `whoami` の意味が分かる
- [ ] `pwd` の意味が分かる
- [ ] `apt update` と `apt install` の違いが分かる
- [ ] Docker Engine を起動できる
- [ ] `docker run hello-world` を実行できる
- [ ] Docker Image と Container の違いを説明できる
- [ ] `docker images` を使える
- [ ] `docker ps` と `docker ps -a` の違いが分かる
- [ ] nginx コンテナを起動できる
- [ ] `-p 8080:80` の意味を説明できる
- [ ] Windows のブラウザから nginx を確認できる
- [ ] `docker exec` でコンテナ内部に入れる
- [ ] `docker stop` と `docker rm` の違いが分かる
- [ ] Docker Image を削除できる
- [ ] WSL を終了できる

---

# 次の Level

Level 2 では、自分で作成した `index.html` を nginx から表示します。

予定している内容：

```text
Windows
  ↓
WSL2 Ubuntu
  ↓
自分で index.html 作成
  ↓
Docker volume mount
  ↓
nginx
  ↓
Windows ブラウザ
```

Level 1 では nginx の標準画面を表示しました。

Level 2 では、自分で作ったファイルを Docker コンテナから公開することで、Docker のボリュームマウントを学びます。
