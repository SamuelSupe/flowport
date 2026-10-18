# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[公式サイト](https://flowport.pages.dev/ja/) · [ダウンロード](https://flowport.pages.dev/ja/downloads/) · [ドキュメント](https://flowport.pages.dev/ja/docs/) · [変更履歴](https://flowport.pages.dev/ja/changelog/)

FlowPort は **Source → Pipeline → Destinations** を通じて、ログ、メトリクス、トレース、イベントを収集・処理し、送信先に振り分けるプラットフォームです。

**このリポジトリには製品ドキュメントと公式サイトへのリンクのみを掲載しています。製品や公式サイトのソースコード、ソース履歴、バイナリは公開していません。** Linux パッケージ、チェックサム、リリースマニフェストは Cloudflare 上の公式サイトで提供します。ドキュメントの公開は、製品のソースコードのオープンソース化を意味しません。エディションとライセンスについては公式サイトをご確認ください。

## はじめに

1. [Linux パッケージをダウンロードしてインストール](https://flowport.pages.dev/ja/downloads/)します。amd64 と arm64 向けに deb、rpm、tar.gz を提供しています。実行に Node.js、npm、Go コンパイラは不要です。
2. [クイックスタート](https://flowport.pages.dev/ja/docs/)に従い、Destination を設定し、データフローを作成・公開します。
3. 下流の送信先で実際の受信を確認します。取り込み時の HTTP 204 は、ローカルへの永続保存と受け付けのみを示し、下流への到達を示すものではありません。

## 動作と制約

- カタログには 72 種類の Source と 48 種類の Destination があり、処理後の Tag による振り分けに対応します。各連携ガイドを参照し、実際に利用するサービス、バージョン、権限の組み合わせで検証してください。
- コミュニティ版のダウンロードパッケージは単一 Worker で動作します。商用の複数 Worker 構成には、提供元による商用ビルドと対応するライセンスが必要です。
- 各 Worker は独立したローカル永続キューを持ちます。業務データのキューはノード間で複製されません。
- 送信先で障害が発生した場合、処理済みの出力をローカルキューに保持し、バックオフして再試行します。新しいデータの取り込みは継続します。容量制限を超えた場合、通常の未完了バッチを元の受信順に FIFO で破棄します。
- 正常な送信先の分岐は独立して送信を続けます。解析または Pipeline の失敗時には、受け付け済みの入力を解放します。旧形式で隔離された失敗ペイロードは、明示的なエクスポートまたは削除が必要です。
- FIFO による破棄ではデータが失われます。遠隔の送信先が受信した後、ローカルでの確認前にクラッシュすると、重複送信される場合があります。厳密に一度だけの送信や、全体での送信順序は保証しません。

ドキュメントの基準バージョンは **0.1.25** です。アップグレード前に[リリースマニフェスト](https://flowport.pages.dev/ja/downloads/manifest/)と[アップグレードに関する注意事項](https://flowport.pages.dev/ja/changelog/)をご確認ください。

0.1.25 の新規環境では [Web 初回設定](https://flowport.pages.dev/ja/docs/#web-setup) で管理者パスワードと管理/HTTP 取り込みのドメインまたは IP、ポートを指定します。起動後にホストの `manager/setup-token` を読み取ります。既存のパスワードとキューは維持します。オフライン復旧とバックアップ検証は[運用ガイド](https://flowport.pages.dev/ja/docs/delivery/#offline-maintenance)を参照してください。

## ドキュメントとフィードバック

[ドキュメント一覧](docs/index.md)から 5 言語のガイドにアクセスできます。このリポジトリ内の[クイックスタート](docs/getting-started.md)、[配信と復旧](docs/delivery.md)、[運用とアップグレード](docs/operations.md)は簡体字中国語です。公式サイトには日本語の完全なガイドを用意しています。

公開可能な製品の質問やドキュメントへのフィードバックは [Issues](https://github.com/SamuelSupe/flowport/issues) にお寄せください。バージョン、OS、機密情報を除いた再現手順を記載してください。認証情報、秘密鍵、業務ペイロード、機密情報を含むデータディレクトリはアップロードしないでください。
