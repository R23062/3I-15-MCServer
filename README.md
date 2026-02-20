# 最終課題：Docker Compose を用いた Minecraft マルチプレイサーバ構築手順書

## 1. 目的と完成条件

### 目的
Docker Compose を活用し、Minecraft (PaperMC) サーバーをコンテナベースで構築する。設定のコード化（IaC）とデータの永続化を両立させ、迅速かつ再現可能なゲームサーバ環境を構築することを目的とする。

### 完成条件
* `docker compose up` によりエラーなくコンテナが起動すること。
* ログにおいて `Starting Minecraft server on *:25565` およびレベル生成の開始が確認できること。

---

## 2. 前提条件
* **対象 OS:** Ubuntu 22.04 LTS (検証環境：Google Cloud Shell / Debian 12)
* **実行環境:** Google Cloud Shell (Cloud インフラ)
* **利用ツール:** Docker, Docker Compose V2
* **ネットワーク条件:** 25565ポートを使用

---

## 3. 追加する発展要素
**【例 B 応用】Docker Compose によるコンテナ管理とデータ永続化**
単一の `docker run` コマンドによる構築ではなく、`docker-compose.yml` を用いて、環境変数による EULA 同意・サーバタイプの指定、およびボリュームマウントによる「コンテナ破棄後もワールドデータが消えない」永続化設計を導入した。

---

## 4. 全体構成図とポート設計
* **外部ポート:** 25565 (TCP)
* **コンポーネント:**
  * `minecraft-server`: PaperMC イメージを使用
  * `data-volume`: ホストOS上の `./data` ディレクトリをマウント



---

## 5. 構築手順

### 5.1 事前準備
まず、パッケージ情報の更新と必要なツールのインストール状態を確認する。

```bash
sudo apt update
docker compose version

```

### 5.2 作業ディレクトリの作成

管理を容易にするため、専用のディレクトリを作成する。

```bash
mkdir mc-server && cd mc-server

```

### 5.3 設定ファイルの作成（完全版）

以下の内容で `docker-compose.yml` を作成する。

* **保存パス:** `~/mc-server/docker-compose.yml`
* **権限:** 644 (所有者：現在のユーザー)

```yaml
version: '3.3'
services:
  minecraft:
    image: itzg/minecraft-server
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"      # EULAへの同意
      TYPE: "PAPER"     # 高速なサーバタイプ PaperMC を指定
      MEMORY: "1G"      # 使用メモリ制限
    volumes:
      - ./data:/data    # データの永続化
    restart: unless-stopped

```
<img width="955" height="166" alt="cat" src="https://github.com/user-attachments/assets/97e915cf-4bb4-4da2-96dd-0680ed3429c6" />

### 5.4 サービスの起動

バックグラウンドでコンテナを起動する。

```bash
docker compose up -d

```

---

## 6. 動作確認と検証

### 6.1 プロセス確認

コンテナが正常に `Up` していることを確認する。

```bash
docker compose ps

```

<img width="959" height="46" alt="composeps" src="https://github.com/user-attachments/assets/153c4612-dca7-4fa5-8742-ce26ff0dfe19" />


### 6.2 ログ確認

サーバの起動プロセスが完了していることをログで確認する。

```bash
docker compose logs

```

<img width="959" height="401" alt="log1" src="https://github.com/user-attachments/assets/95cecb89-8127-4337-bcf3-63361d72c388" />
<img width="959" height="396" alt="log2" src="https://github.com/user-attachments/assets/31a5eb13-1ca1-4f5c-8238-520c02647a35" />
<img width="959" height="164" alt="log3" src="https://github.com/user-attachments/assets/b4b437e2-9541-4559-b73e-156dcf066c12" />


---

## 7. トラブルシューティング

| 問題 | 原因 | 対処 |
| --- | --- | --- |
| `docker-compose` (ハイフンあり) 実行時の API バージョンエラー | 実行環境の Docker エンジンと古い V1 系の互換性問題 | 最新の `docker compose` (スペース区切り) コマンドを使用する |
| `version` 属性に関する警告 | Docker Compose V2 では version 指定が非推奨 | 最新環境では `version` 行を削除しても問題ない |

---

## 8. セキュリティ配慮

* **不要ポートの閉鎖:** 25565 以外のポートはコンテナ側で開放していない。
* **秘密情報の扱い:** 今回は秘密情報（パスワード等）はないが、必要に応じて `.env` ファイルに切り出し、`.gitignore` で管理する。
* **権限最小化:** コンテナ内プロセスは可能な限り非特権ユーザーで実行されるよう、イメージのベストプラクティスに従った。

---

## 9. 参考資料

* [itzg/minecraft-server - Docker Hub](https://hub.docker.com/r/itzg/minecraft-server)
* [Docker Compose 公式ドキュメント](https://docs.docker.com/compose/)
