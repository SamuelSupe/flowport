# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[官網](https://flowport.pages.dev/zh-TW/) · [下載](https://flowport.pages.dev/zh-TW/downloads/) · [完整文件](https://flowport.pages.dev/zh-TW/docs/) · [版本紀錄](https://flowport.pages.dev/zh-TW/changelog/)

FlowPort 透過 **Source → Pipeline → Destinations** 收集、處理與路由日誌、指標、追蹤及事件資料。

**本儲存庫僅包含產品文件與官網連結，不公開產品原始碼、官網原始碼、原始碼歷史或安裝套件。** Linux 安裝套件、校驗值與版本清單由 Cloudflare 官網提供。公開文件不代表產品原始碼開源；產品版本與授權方式請以官網說明為準。

## 0.2.1

- 修正 Kafka 以事件時間判斷固定 20 秒投遞逾時的問題，歷史日誌、指標、追蹤及其他 Point 資料不再因時間較舊而在傳送前失敗。Kafka 訊息時間與編碼載荷中的原始事件時間保持不變，適用於所有 Kafka 輸出及使用該協定的服務預設。

- 每次 Kafka 傳送由 config.timeout_seconds 控制，預設 20 秒、範圍 1–120 秒，並遵守上層較早的取消或截止時間。重試從新一次傳送開始計時；傳送失敗仍保留原佇列與處理結果，不透過改寫事件時間或清空積壓恢復。

- 0.2.1 沿用控制協定 v6 與檢查點格式 3。停止服務並備份完整資料目錄、master.key、Lookup 資源與環境設定後，升級管理端及所有 Worker。使用原資料目錄恢復後，保留的 Kafka 積壓自動重試；已被 FIFO 淘汰的資料無法恢復。目標 Broker 或雲端服務的時間範圍與保留政策仍適用。

## 0.2.0 新功能

- Lookup 支援 Exact 複合鍵、IPv4/IPv6 CIDR、RE2 Regex 及 MMDB；可上傳 CSV、CSV.GZ、MMDB，或從 HTTP(S)、S3、PostgreSQL、MySQL 背景同步完整快照。Pipeline 以 lookup("binding_name") 本機查詢並映射欄位或 Tag。
- Lookup 提供自動更新、固定版本、試查、歷史回復及 Worker 安裝狀態。批次入列時固定資料版本；同步失敗保留最後有效版本，佇列所需資源缺失時暫停處理並保留載荷。回復歷史會暫停自動同步，須明確操作才恢復。
- 開放 cache_get/cache_set、point_window/window_hit、agg_create/agg_metric，支援短期去重、錯誤上下文及計數／時間視窗彙總。有界記憶體依 Worker、流發佈版本及輸入類型隔離，重啟清空；樣本除錯不影響執行狀態。
- 新增五種場景範本及單流設定匯入匯出（flowport.flow v1，最大 2 MiB）。映射本機目標及 Lookup，檢查後儲存並明確發佈。套件不含佇列、游標、Lookup 資料或記憶體狀態；連線憑證移除，腳本／SQL 字面值仍須檢查。改善設定版面、非同步取消、資源讀取重用及多綁定匯出。

## 升級至 0.2.0

控制協定升級至 v6，檢查點格式升級至 3。停止服務並備份各節點完整資料目錄、master.key、Lookup 資源與環境設定，再協調升級管理端與全部 Worker。降級須恢復升級前完整備份。提供 Linux ARM64/x86_64 的 tar.gz、deb、rpm；安裝及執行不需 Node.js。

佇列所需的固定 Lookup 資源缺失時暫停處理並等待恢復，不依一般 Pipeline 失敗釋放輸入。記憶體快取、上下文及彙總狀態在重啟後清空；已入列的輸出保留原結果。

## 使用指南

- [Lookup 資料富化](docs/zh-TW/lookup.md)
- [快取與視窗](docs/zh-TW/pipeline.md)
- [場景範本與匯入匯出](docs/zh-TW/flows.md)

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

文件基準版本：**0.2.1**。升級前請查閱[版本清單](https://flowport.pages.dev/zh-TW/downloads/manifest/)與[升級說明](https://flowport.pages.dev/zh-TW/changelog/)。

0.2.0 全新安裝透過 [Web 初始化向導](https://flowport.pages.dev/zh-TW/docs/#web-setup) 設定管理員密碼與管理/HTTP 接入的網域或 IP、連接埠；啟動後讀取本機 `manager/setup-token`。既有實例保留密碼與佇列；離線救援及可校驗備份見[運維說明](https://flowport.pages.dev/zh-TW/docs/delivery/#offline-maintenance)。

## 文件與回饋

[文件目錄](docs/index.md)提供五種語言閱讀版入口。本儲存庫的[快速開始](docs/getting-started.md)、[交付與恢復](docs/delivery.md)、[維運與升級](docs/operations.md)指南為簡體中文；官網提供完整繁體中文指南。

歡迎透過 [Issues](https://github.com/SamuelSupe/flowport/issues) 提交公開的產品問題或文件回饋。請附上版本、作業系統與去識別化的重現步驟，不要上傳密碼、Token、私鑰、業務載荷或含敏感資訊的資料目錄。
