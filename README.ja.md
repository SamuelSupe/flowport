[![FlowPorts — Collect. Transform. Deliver.](docs/images/flowports-cover.svg)](https://flowport.pages.dev/ja/)

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

# データをつなぎ、処理し、必要な場所へ届ける。

ログ、メトリクス、トレース、イベントを **Source → Pipeline → Destinations** で処理し、観測・分析システムへ転送します。

**[公式サイト](https://flowport.pages.dev/ja/) · [はじめに](https://flowport.pages.dev/ja/docs/) · [Linux ダウンロード](https://flowport.pages.dev/ja/downloads/) · [リリースノート](https://flowport.pages.dev/ja/changelog/#v0-5-13)**

## 0.5.13 の更新

- FlowPort 0.5.13 は、Stream Tunnel を使用する Alibaba Cloud MaxCompute への配信に対応しました。
- 既存テーブルへの追記に対応し、動的パーティション、列マッピング、AccessKey または STS の認証情報を使用できます。
- Worker はパックの受領確認を WAL に保存し、未確認のレコードのみを再試行します。同一 Worker 上で Tunnel エンドポイント、プロジェクト、クォータが同じフローは、HTTP 429 または 503 を受信するとバックオフを共有します。
- 配信は少なくとも 1 回です。書き込みに成功して受領確認が失われた場合、再試行で行が重複することがあります。

[MaxCompute の設定](https://flowport.pages.dev/ja/docs/integrations/#maxcompute)

## ソースから配信まで、ひとつの経路

| 収集 | 処理 | 配信 |
| --- | --- | --- |
| アプリ、インフラ、データベース、メッセージ基盤、クラウド向けの 72 種類の Source。 | Pipeline で解析・変換し、ローカル Lookup で補完。Tag ルーティングと上限付きキャッシュ・ウィンドウを利用できます。 | 50 種類の Destination。出力チャネルを独立させ、実行画面で確認、滞留、異常を表示します。 |

[![FlowPort 0.5.3 をローカル Chrome で撮影。実際の HTTP レコードが受信・処理され、ローカル受信先が確認しました。隔離テスト環境の合成データを使用し、認証情報はマスクしています。](docs/images/first-delivery-0.5.3.jpg)](docs/images/first-delivery-0.5.3.jpg)

*FlowPort 0.5.3 をローカル Chrome で撮影。実際の HTTP レコードが受信・処理され、ローカル受信先が確認しました。隔離テスト環境の合成データを使用し、認証情報はマスクしています。*

## 最初のレコードを届ける

1. [Linux パッケージをダウンロード](https://flowport.pages.dev/ja/downloads/)し、SHA-256 を確認します。amd64 / arm64 の deb、rpm、tar.gz を提供。実行に Node.js、npm、Go コンパイラーは不要です。
2. Web 初期設定を完了し、データフローを作成してソースを選択します。下書きはいつでも保存できます。変換が不要なら空の Pipeline でそのまま転送します。
3. エディターで送信先を追加して公開し、Worker のインストールを待ちます。配信検証を開始してからサンプルを送り、送信先で実データを確認します。

## 実用ガイド

- **[データフローとルーティング](docs/ja/flows.md)** — 作成、保存、公開、フロー設定のインポートとエクスポート。
- **[Lookup データ補完](docs/ja/lookup.md)** — ローカル照合、バックグラウンド同期、不変データバージョン。
- **[Pipeline とウィンドウ](docs/ja/pipeline.md)** — 解析、短期間の重複排除、コンテキスト、集約。

[ドキュメント一覧](docs/index.md) · [配信と復旧](https://flowport.pages.dev/ja/docs/delivery/) · [エディションとライセンス](https://flowport.pages.dev/ja/editions/)

## 配信の境界を理解する

- 取り込み確認は現在の Worker での永続受理を示し、送信先への配信は保証しません。Worker のキューは独立し、ノード間で複製しません。
- 初回送信後の高速再試行は宛先ごとに既定で 5 回です。上限に達した場合、未確認データを保持し、少なくとも 60 秒待ってから 1 バッチを試送します。
- 新しいフローの既定値は Worker ごとに 256 MiB、500 内部バッチです。入力領域と各宛先に分割されます。出力領域が満杯なら元の入力を保持し、入力領域が満杯なら新規データを拒否します。
- プロトコル確認は下流の業務処理完了を意味しません。確認の消失で重複が起こり得ます。厳密な一度限りの送信や送信先をまたぐトランザクションは保証しません。

### 更新する前に

0.5.13 は制御プロトコル v9、チェックポイント形式 3 を使用します。取り込みを停止し、各ノードのデータ、master.key、Lookup リソースと環境設定をバックアップしてから、管理サーバーと全 Worker をまとめて更新してください。ロールバック時は更新前のバックアップを復元してください。

## 質問とフィードバック

製品の質問、再現可能な問題、文書の修正は [Issues](https://github.com/SamuelSupe/flowport/issues/new/choose) へ。バージョンと機密情報を除いた例を添えてください。

---

ここは公開ドキュメントの入口です。公式サイトのブランドは FlowPorts、インストールするソフトウェアは FlowPort です。製品・サイトのソース、ソース履歴、バイナリーは公開していません。コミュニティ版はローカル Worker 1 台。商用の複数 Worker 構成には発行元のビルドとライセンスが必要です。
