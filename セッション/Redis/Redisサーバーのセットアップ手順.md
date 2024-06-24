## Redis サーバーのセットアップ手順

### 1. Redis のインストール

#### macOS の場合

Homebrew を使って Redis をインストールする

```sh
brew install redis
```

#### Ubuntu の場合

APT パッケージマネージャーを使って Redis をインストールする

```sh
sudo apt update
sudo apt install redis-server
```

#### Windows の場合

Redis の公式サイトから Windows 向けのインストーラーをダウンロードしてインストールする

- [公式サイト](https://redis.io/download)

### 2. Redis の設定（必要に応じて）

`redis.conf`ファイルを編集して、必要に応じて設定を変更する。一般的な設定項目は以下の通り：

- `bind`: Redis がバインドする IP アドレス（デフォルトは 127.0.0.1）
- `port`: Redis がリスンするポート（デフォルトは 6379）
- `requirepass`: Redis への接続に必要なパスワード

例：

```sh
bind 127.0.0.1
port 6379
requirepass yourpassword
```

### 3. Redis サーバーの起動

#### macOS の場合

```sh
brew services start redis
```

手動で起動する場合：

```sh
redis-server /usr/local/etc/redis.conf
```

#### Ubuntu の場合

```sh
sudo systemctl enable redis-server.service
sudo systemctl start redis-server.service
```

手動で起動する場合：

```sh
redis-server /etc/redis/redis.conf
```

#### Windows の場合

インストーラーによってサービスが自動的に開始されることが多いが、手動で起動する場合：

```sh
redis-server redis.windows.conf
```

### 4. Redis クライアントのインストールと接続確認

Redis が正しく起動しているか確認するために、`redis-cli`を使って接続し、動作確認を行う。

```sh
redis-cli
```

接続後、以下のようなコマンドを入力して動作を確認：

```sh
ping
```

結果として`PONG`が返ってくれば、Redis サーバーが正常に動作している。

### 5. Redis のステータス確認

Redis サーバーのステータスを確認するには、以下のコマンドを使用する：

```sh
redis-cli info
```

これにより、Redis サーバーの詳細なステータス情報が表示される。

### 6. Redis サーバーの停止

Redis サーバーを停止するには、以下のコマンドを使用する：

#### macOS の場合

```sh
brew services stop redis
```

手動で停止する場合：

```sh
redis-cli shutdown
```

#### Ubuntu の場合

```sh
sudo systemctl stop redis-server.service
```

手動で停止する場合：

```sh
redis-cli shutdown
```

#### Windows の場合

```sh
redis-cli shutdown
```

これで Redis サーバーのセットアップは完了。これにより、セッション管理やキャッシュなどに Redis を利用できるようになる。
