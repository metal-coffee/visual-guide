# WSL2 + Ubuntu + Docker 練習帳

# Level 2：自分で作った HTML を nginx で公開する

この Level では、WSL2 上の Ubuntu に自分で作成した `index.html` を置き、
Docker 上の nginx コンテナから公開します。

Level 1 では nginx が最初から持っている標準ページを表示しました。

Level 2 では、次の構成を作ります。

```text
Windows ブラウザ
      ↓
http://localhost:8080
      ↓
WSL2 Ubuntu
      ↓
Docker
      ↓
nginx コンテナ
      ↓
WSL 上で自分が作成した index.html
```

---

## 1. この Level の目的

この Level では、次の内容を理解することを目的とします。

- WSL 上に HTML ファイルを作成する
- nginx コンテナを起動する
- WSL 上のディレクトリをコンテナに Bind Mount する
- Windows のブラウザから自分の HTML を確認する
- コンテナ内部から Mount されたファイルを確認する
- WSL 側の HTML を変更し、コンテナを再起動せず反映されることを確認する
- `readonly` の意味を確認する
- コンテナを削除しても WSL 側のファイルが残ることを確認する
- Bind Mount と Named Volume の違いを理解する

---

# 2. WSL Ubuntu を起動する

Windows の PowerShell から実行します。

```powershell
wsl
```

複数の WSL ディストリビューションがある場合は一覧を確認できます。

```powershell
wsl -l -v
```

Ubuntu を指定して起動する場合の例です。

```powershell
wsl -d Ubuntu
```

Ubuntu に入ったら確認します。

```bash
whoami
pwd
```

---

# 3. Docker Engine を起動する

Docker Engine を起動します。

```bash
sudo systemctl start docker
```

確認します。

```bash
sudo docker ps
```

エラーにならなければ Docker Engine は利用できる状態です。

---

## systemctl が使えない場合

WSL 側で systemd が有効になっていない場合は、次を使用します。

```bash
sudo service docker start
```

その後、確認します。

```bash
sudo docker ps
```

---

# 4. Level 2 用ディレクトリを作成する

今回は次の構成にします。

```text
~/wsl-lab/
└── level2/
    └── html/
        └── index.html
```

ディレクトリを作成します。

```bash
mkdir -p ~/wsl-lab/level2/html
```

Level 2 のディレクトリへ移動します。

```bash
cd ~/wsl-lab/level2
```

現在位置を確認します。

```bash
pwd
```

想定例：

```text
/home/<ユーザー名>/wsl-lab/level2
```

---

# 5. index.html を作成する

今回は `nano` を使って HTML ファイルを作成します。

```bash
nano html/index.html
```

次の内容を入力します。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>WSL Docker Practice</title>
</head>
<body>
    <h1>Hello from WSL2 + Docker!</h1>
    <p>Level 2 の nginx 動作確認です。</p>
</body>
</html>
```

---

## nano で保存する

保存：

```text
Ctrl + O
```

その後 Enter を押します。

終了：

```text
Ctrl + X
```

---

# 6. 作成したファイルを確認する

HTML の内容を確認します。

```bash
cat html/index.html
```

先ほど入力した HTML が表示されれば OK です。

ディレクトリ構成も確認します。

```bash
tree
```

想定：

```text
.
└── html
    └── index.html
```

---

# 7. nginx コンテナを起動する

ここが Level 2 の重要な部分です。

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  --mount type=bind,source="$(pwd)/html",target=/usr/share/nginx/html,readonly \
  nginx:alpine
```

起動確認：

```bash
sudo docker ps
```

---

# 8. docker run の指定内容を理解する

今回のコマンド：

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  --mount type=bind,source="$(pwd)/html",target=/usr/share/nginx/html,readonly \
  nginx:alpine
```

各指定の意味です。

| 指定 | 意味 |
|---|---|
| `docker run` | コンテナを作成して起動する |
| `-d` | バックグラウンドで起動する |
| `--name my-nginx` | コンテナ名を `my-nginx` にする |
| `-p 8080:80` | Ubuntu 側の 8080 番ポートをコンテナの 80 番ポートへ接続する |
| `--mount type=bind,...` | WSL 側のディレクトリをコンテナへ Bind Mount する |
| `readonly` | コンテナ側から書き込めないようにする |
| `nginx:alpine` | nginx の Alpine Linux 版イメージを使用する |

---

# 9. Bind Mount の意味

今回の構成は次のようになります。

```text
WSL Ubuntu

/home/<ユーザー名>/wsl-lab/level2/html
              │
              │ Bind Mount
              ▼
Docker Container

/usr/share/nginx/html
```

nginx は標準で次のディレクトリにあるファイルを Web コンテンツとして公開します。

```text
/usr/share/nginx/html
```

そこで今回は、

```text
WSL 上の

~/wsl-lab/level2/html

        ↓

コンテナ内の

/usr/share/nginx/html
```

として接続しています。

---

# 10. Ubuntu から nginx にアクセスする

Ubuntu 上からアクセスします。

```bash
curl http://localhost:8080
```

HTML が返ってくれば成功です。

内容に次のような文字列が含まれます。

```text
Hello from WSL2 + Docker!
```

---

# 11. Windows のブラウザから確認する

Windows 側の Chrome や Edge で次を開きます。

```text
http://localhost:8080
```

画面に次の内容が表示されれば成功です。

```text
Hello from WSL2 + Docker!

Level 2 の nginx 動作確認です。
```

---

## 通信の流れ

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
nginx Container :80
  │
  ▼
/usr/share/nginx/html/index.html
  │
  ▼
Bind Mount
  │
  ▼
WSL 側の html/index.html
```

---

# 12. コンテナの中に入る

nginx コンテナ内部に入ります。

```bash
sudo docker exec -it my-nginx sh
```

プロンプトが変われば、コンテナ内部に入っています。

---

# 13. コンテナ内部から HTML を確認する

コンテナ内部で移動します。

```sh
cd /usr/share/nginx/html
```

ファイル一覧を確認します。

```sh
ls -l
```

HTML を表示します。

```sh
cat index.html
```

WSL 側で作成した HTML が見えれば成功です。

---

## このときの関係

WSL 側：

```text
~/wsl-lab/level2/html/index.html
```

コンテナ側：

```text
/usr/share/nginx/html/index.html
```

Bind Mount により、コンテナ側から WSL 側のファイルを参照しています。

---

# 14. コンテナから出る

```sh
exit
```

Ubuntu 側に戻ります。

確認：

```bash
pwd
```

---

# 15. HTML を変更する

WSL 側で HTML を編集します。

```bash
nano html/index.html
```

次のように変更します。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>WSL Docker Practice</title>
</head>
<body>
    <h1>Hello from WSL2 + Docker!</h1>
    <p>Level 2 の nginx 動作確認です。</p>
    <p>HTML を変更しました。</p>
</body>
</html>
```

保存して終了します。

```text
Ctrl + O
Enter
Ctrl + X
```

---

# 16. Docker を再起動せずブラウザを更新する

ここでは Docker コンテナを再起動しません。

次の操作は不要です。

```text
docker stop
docker start
docker restart
```

そのまま Windows 側のブラウザを再読み込みします。

```text
http://localhost:8080
```

次の文字が追加されていれば成功です。

```text
HTML を変更しました。
```

---

## なぜ即座に反映されるのか

```text
WSL 側の index.html を変更
          ↓
Bind Mount
          ↓
コンテナから同じファイルが見える
          ↓
nginx が読み込む
          ↓
ブラウザへ返す
```

コンテナ内へファイルをコピーしたわけではありません。

WSL 側のファイルをコンテナから参照しています。

---

# 17. Docker から Mount 情報を確認する

Docker の設定内容を確認します。

```bash
sudo docker inspect my-nginx
```

大量の情報が表示されます。

その中の `Mounts` を確認します。

おおむね次のような情報があります。

```text
"Type": "bind"
```

```text
"Source": "/home/..."
```

```text
"Destination": "/usr/share/nginx/html"
```

---

## Mount 情報だけ確認する

次のコマンドでも確認できます。

```bash
sudo docker inspect my-nginx \
  --format '{{json .Mounts}}'
```

確認したいポイント：

```text
Type        : bind
Source      : WSL 側の html ディレクトリ
Destination : /usr/share/nginx/html
```

---

# 18. readonly を確認する

今回の Mount には次を指定しています。

```text
readonly
```

意味は次のとおりです。

```text
WSL → コンテナ

読み取り：OK
書き込み：NG
```

実際に確認します。

コンテナへ入ります。

```bash
sudo docker exec -it my-nginx sh
```

コンテナ内部で実行します。

```sh
echo "test" > /usr/share/nginx/html/test.txt
```

想定：

```text
Read-only file system
```

などのエラーになります。

これは意図した動作です。

コンテナから出ます。

```sh
exit
```

---

# 19. なぜ readonly にするのか

今回の構成では、

```text
HTML の正本
=
WSL 側
```

とします。

nginx は HTML を読み取るだけで十分です。

そのため、

```text
WSL
 │
 │ 読ませる
 ▼
nginx
```

という構成にしておけば、nginx コンテナ側から誤って HTML を変更することを防げます。

---

# 20. コンテナを停止する

現在の状態を確認します。

```bash
sudo docker ps
```

停止します。

```bash
sudo docker stop my-nginx
```

確認：

```bash
sudo docker ps
```

`my-nginx` が表示されなければ停止しています。

---

# 21. 停止済みコンテナを確認する

```bash
sudo docker ps -a
```

`my-nginx` は停止していますが、まだコンテナ自体は存在します。

```text
docker stop
    ↓
停止
    ↓
コンテナは残る
```

---

# 22. コンテナを削除する

```bash
sudo docker rm my-nginx
```

確認：

```bash
sudo docker ps -a
```

`my-nginx` が表示されなければ削除完了です。

---

# 23. WSL 側の HTML が残っていることを確認する

コンテナを削除したあとも HTML が残っていることを確認します。

```bash
ls -l html
```

さらに内容を確認します。

```bash
cat html/index.html
```

HTML は残っています。

---

## なぜ残るのか

今回の HTML は、

```text
Docker コンテナ内部
```

に保存していたのではありません。

正本は、

```text
WSL 側

~/wsl-lab/level2/html/index.html
```

です。

そのため、

```text
Docker Container
      ↓
削除

でも

WSL
└── level2
    └── html
        └── index.html

は残る
```

という状態になります。

---

# 24. 同じ HTML から新しいコンテナを作成する

再度 nginx コンテナを作成します。

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  --mount type=bind,source="$(pwd)/html",target=/usr/share/nginx/html,readonly \
  nginx:alpine
```

確認：

```bash
sudo docker ps
```

Windows のブラウザで開きます。

```text
http://localhost:8080
```

先ほど変更した HTML がそのまま表示されれば成功です。

---

# 25. コンテナを使い捨てられることを理解する

今回の実験から、次の考え方が分かります。

```text
コンテナ
  ↓
削除してよい

データ・ソースコード
  ↓
コンテナ外に保持
```

新しいコンテナを作り直しても、WSL 側のファイルを再利用できます。

---

# 26. Bind Mount と Named Volume

今回使用したのは、

```text
Bind Mount
```

です。

---

## Bind Mount

自分が管理している具体的なファイルやディレクトリを、
コンテナへ接続します。

```text
WSL

~/project/html
      │
      │ Bind Mount
      ▼
Container

/usr/share/nginx/html
```

向いている用途：

- ソースコード
- HTML
- 設定ファイル
- 開発中に頻繁に変更するファイル

---

## Named Volume

Docker が管理する保存領域です。

イメージ：

```text
Docker 管理領域
      │
      │ Named Volume
      ▼
Container
```

向いている用途：

- PostgreSQL のデータ
- MySQL のデータ
- コンテナを削除しても保持したいアプリケーションデータ

Named Volume は後の Level で使用します。

---

# 27. 今回覚えたい Docker コマンド

| コマンド | 意味 |
|---|---|
| `docker run` | コンテナを作成して起動 |
| `docker ps` | 起動中コンテナ一覧 |
| `docker ps -a` | 停止済みを含むコンテナ一覧 |
| `docker exec` | 起動中コンテナ内でコマンド実行 |
| `docker inspect` | コンテナ等の詳細情報確認 |
| `docker stop` | コンテナ停止 |
| `docker rm` | コンテナ削除 |

---

# 28. 今回覚えたい Linux コマンド

| コマンド | 意味 |
|---|---|
| `mkdir -p` | 親ディレクトリを含めディレクトリ作成 |
| `cd` | ディレクトリ移動 |
| `pwd` | 現在位置確認 |
| `nano` | テキスト編集 |
| `cat` | ファイル内容表示 |
| `tree` | ディレクトリ構成表示 |
| `curl` | HTTP アクセス |

---

# 29. Level 2 の重要ポイント

今回特に理解しておきたいのは次の4点です。

1. コンテナ内のファイルと WSL 側のファイルは別物
2. Bind Mount を使うと WSL 側のファイルをコンテナから利用できる
3. WSL 側のファイルを変更すると、コンテナを再起動せず変更が見える
4. コンテナを削除しても WSL 側のファイルは消えない

---

# 30. 完成形

```text
Windows
│
│ Chrome / Edge
│ http://localhost:8080
│
▼
WSL2
│
├── Ubuntu
│
│   ~/wsl-lab/level2/
│   └── html/
│       └── index.html       ← 正本
│            │
│            │ Bind Mount
│            ▼
│   Docker Engine
│            │
│            ▼
│   ┌──────────────────────────┐
│   │ nginx container          │
│   │                          │
│   │ /usr/share/nginx/html    │
│   │          │               │
│   │          ▼               │
│   │        nginx :80         │
│   └──────────────────────────┘
│
└── Ubuntu :8080
```

---

# 31. 繰り返し練習用の短縮版

一度理解した後は、次の手順で Level 2 を復習できます。

## WSL 起動

PowerShell：

```powershell
wsl
```

---

## Docker 起動

```bash
sudo systemctl start docker
```

または：

```bash
sudo service docker start
```

---

## 作業ディレクトリへ移動

```bash
cd ~/wsl-lab/level2
```

---

## HTML 確認

```bash
cat html/index.html
```

---

## nginx 起動

```bash
sudo docker run -d \
  --name my-nginx \
  -p 8080:80 \
  --mount type=bind,source="$(pwd)/html",target=/usr/share/nginx/html,readonly \
  nginx:alpine
```

---

## 動作確認

```bash
sudo docker ps
curl http://localhost:8080
```

Windows：

```text
http://localhost:8080
```

---

## コンテナ内部から確認

```bash
sudo docker exec -it my-nginx sh
```

```sh
cd /usr/share/nginx/html
cat index.html
exit
```

---

## HTML を変更

```bash
nano html/index.html
```

ブラウザを再読み込みし、変更が即座に反映されることを確認します。

---

## Mount 確認

```bash
sudo docker inspect my-nginx \
  --format '{{json .Mounts}}'
```

---

## 停止・削除

```bash
sudo docker stop my-nginx
sudo docker rm my-nginx
```

---

## HTML が残っていることを確認

```bash
cat html/index.html
```

---

# 32. Level 2 完了チェックリスト

以下を自分で実行・説明できれば Level 2 完了です。

- [ ] WSL 上に `index.html` を作成できる
- [ ] nginx コンテナを起動できる
- [ ] `-p 8080:80` の意味を説明できる
- [ ] Bind Mount の意味を説明できる
- [ ] `--mount type=bind` の Source と Destination を説明できる
- [ ] Windows ブラウザから自分の HTML を表示できる
- [ ] コンテナ内部から WSL 側の HTML を確認できる
- [ ] WSL 側の HTML を変更し、再起動なしで反映できる
- [ ] `readonly` の意味を説明できる
- [ ] `docker inspect` で Mount 情報を確認できる
- [ ] コンテナ削除後も WSL 側の HTML が残ることを説明できる
- [ ] Bind Mount と Named Volume の用途の違いを説明できる

---

# 次の Level

Level 3 では Docker Compose を使用します。

Level 2 では長い `docker run` コマンドで、

- コンテナ名
- ポート
- Bind Mount
- 使用するイメージ

を指定しました。

Level 3 では、これらを `compose.yaml` に定義します。

```text
compose.yaml
    ↓
docker compose up -d
    ↓
環境をまとめて起動
```

さらに、

```bash
docker compose ps
docker compose logs
docker compose down
```

などを使い、Docker Compose を使った環境管理を練習します。
