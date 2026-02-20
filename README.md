最終課題：Docker Compose を用いた Minecraft マルチプレイサーバ構築手順書

1. 目的と完成条件

目的

これまでの授業で学んだOS設定、ネットワーク、コンテナ技術の基礎を応用し、Docker Compose を活用して Minecraft (PaperMC) サーバーをコンテナベースで構築する。設定のコード化（Infrastructure as Code）とデータの永続化を両立させ、迅速かつ再現可能なゲームサーバ環境を構築することを目的とする。

完成条件

docker compose up によりエラーなくコンテナが起動すること。

ログにおいて Starting Minecraft server on *:25565 およびレベル（ワールド）生成の開始メッセージが確認できること。

2. 前提条件

対象 OS: Ubuntu 22.04 LTS (検証環境：Google Cloud Shell / Debian 12)

実行環境: Google Cloud Shell (ブラウザベースのクラウド環境)

利用ツール: Docker, Docker Compose V2

ネットワーク条件: デフォルトポート 25565 (TCP) を使用

3. 追加する発展要素

【例 B 応用】Docker Compose によるコンテナ管理とデータ永続化

単一の docker run コマンドによる一時的な構築ではなく、docker-compose.yml を定義することで以下の発展的要素を導入した。

環境変数による制御: EULAへの同意やサーバタイプ（PaperMC）の指定をコード上で管理。

データの永続化: ボリュームマウント（Bind Mount）を用い、コンテナを破棄・再作成してもワールドデータや設定が維持される設計とした。

リソース管理: 起動時のメモリ割り当て制限（1G）を環境変数から指定。

4. 全体構成図とポート設計

外部アクセスポート: 25565 (TCP)

コンポーネント構成:

Host OS: Google Cloud Shell

Container: Minecraft Server (itzg/minecraft-server イメージ)

Storage: ./data ディレクトリをコンテナ内の /data にマウント

5. 構築手順

5.1 事前準備

システムパッケージの更新と、Docker Compose V2 が利用可能であることを確認する。

sudo apt update
docker compose version


5.2 作業ディレクトリの作成

環境を分離し管理を容易にするため、専用のディレクトリを作成する。

mkdir mc-server && cd mc-server


5.3 設定ファイルの作成（完全版）

以下の内容で docker-compose.yml を作成する。このファイルにより、サーバの構成が定義される。

保存パス: /home/smnzn_3t/mc-server/docker-compose.yml

所有権/権限: 現在の実行ユーザー / 644

version: '3.3'
services:
  minecraft:
    image: itzg/minecraft-server
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"      # Minecraft EULA への同意
      TYPE: "PAPER"     # パフォーマンスに優れた PaperMC を採用
      MEMORY: "1G"      # 最大メモリ使用量を 1GB に制限
    volumes:
      - ./data:/data    # ホストのカレントディレクトリ内 data フォルダにデータを永続化
    restart: unless-stopped


5.4 サービスの起動

作成した設定ファイルに基づき、バックグラウンドでサービスを開始する。

docker compose up -d


6. 動作確認と検証

6.1 プロセス確認

コンテナが正常に生成され、起動（Up）状態にあることを確認する。

docker compose ps


実行結果スクリーンショット 1：
(ここに撮影した docker compose ps の画像を貼り付けてください)

6.2 ログ確認・疎通確認

サーバ内部のログを確認し、起動プロセスが完了（Done）またはレベル生成が開始されていることを確認する。

docker compose logs


実行結果スクリーンショット 2：
(ここに撮影したログの画像を貼り付けてください)

7. トラブルシューティング

代表的な失敗例: docker-compose (ハイフンあり) 実行時の API バージョンエラー。

原因: 実行環境の Docker エンジンと、古い V1 系の Compose バイナリの互換性問題。

対処方法: 最新の V2 形式である docker compose (スペース区切り) コマンドを使用することで正常に動作することを確認した。

考慮すべき点: メモリ不足で起動しない場合は、MEMORY 環境変数の値を調整する必要がある。

8. セキュリティ配慮

ポート開放の最小化: サービスの運用に必要な 25565 ポートのみをコンテナ外部へ公開している。

非特権ユーザーの利用: 使用している Docker イメージ（itzg/minecraft-server）のベストプラクティスに従い、コンテナ内プロセスが適切に管理されるよう構成した。

データの分離: 秘密情報（設定ファイル等）が含まれる data ディレクトリはホスト側で管理し、適切なパーミッション設定を推奨する。

9. 参考資料

itzg/minecraft-server - Docker Hub Official Documentation

Docker Compose File Reference (V3)
