# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[官網](https://flowport.pages.dev/zh-TW/) · [下載](https://flowport.pages.dev/zh-TW/downloads/) · [完整文件](https://flowport.pages.dev/zh-TW/docs/) · [版本紀錄](https://flowport.pages.dev/zh-TW/changelog/)

FlowPort 透過 **Source → Pipeline → Destinations** 收集、處理與路由日誌、指標、追蹤及事件資料。

**本儲存庫僅包含產品文件與官網連結，不公開產品原始碼、官網原始碼、原始碼歷史或安裝套件。** Linux 安裝套件、校驗值與版本清單由 Cloudflare 官網提供。公開文件不代表產品原始碼開源；產品版本與授權方式請以官網說明為準。

## 開始使用

1. [下載並安裝 Linux 套件](https://flowport.pages.dev/zh-TW/downloads/)。支援 amd64、arm64，以及 deb、rpm、tar.gz 格式。執行 FlowPort 不需要 Node.js、npm 或 Go 編譯器。
2. 按照[快速開始](https://flowport.pages.dev/zh-TW/docs/)設定 Destination、建立資料流並發布。
3. 確認下游確實收到資料。接入端回傳 HTTP 204 僅表示本機已持久化接收，不代表下游已送達。

## 運作方式

- 目錄包含 72 類 Source 與 48 類 Destination，支援依處理後的 Tag 分流。請參閱各整合指南，並驗證實際服務、版本與權限組合。
- 社群下載版使用單一 Worker。商業多 Worker 部署需要發行方提供的商業建置版本與相符的授權。
- 每個 Worker 擁有獨立的本機持久佇列；業務佇列不會在節點之間複寫。
- 目標傳送失敗時，處理結果會保留在本機佇列，並以退避方式重試；新資料持續接入。超過容量限制時，正常未完成批次依原始接收順序以 FIFO 淘汰。
- 健康的目標分支繼續獨立傳送。解析或 Pipeline 失敗會釋放已接收的輸入；舊隔離失敗載荷須明確匯出或清理。
- FIFO 淘汰允許資料遺失。若遠端已接收，但本機尚未確認便發生崩潰，可能重複傳送；不保證嚴格只傳送一次或全域順序。

文件基準版本：**0.1.25**。升級前請查閱[版本清單](https://flowport.pages.dev/zh-TW/downloads/manifest/)與[升級說明](https://flowport.pages.dev/zh-TW/changelog/)。

0.1.25 全新安裝透過 [Web 初始化向導](https://flowport.pages.dev/zh-TW/docs/#web-setup) 設定管理員密碼與管理/HTTP 接入的網域或 IP、連接埠；啟動後讀取本機 `manager/setup-token`。既有實例保留密碼與佇列；離線救援及可校驗備份見[運維說明](https://flowport.pages.dev/zh-TW/docs/delivery/#offline-maintenance)。

## 文件與回饋

[文件目錄](docs/index.md)提供五種語言閱讀版入口。本儲存庫的[快速開始](docs/getting-started.md)、[交付與恢復](docs/delivery.md)、[維運與升級](docs/operations.md)指南為簡體中文；官網提供完整繁體中文指南。

歡迎透過 [Issues](https://github.com/SamuelSupe/flowport/issues) 提交公開的產品問題或文件回饋。請附上版本、作業系統與去識別化的重現步驟，不要上傳密碼、Token、私鑰、業務載荷或含敏感資訊的資料目錄。
