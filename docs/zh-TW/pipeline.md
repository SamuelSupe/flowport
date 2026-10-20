# Pipeline 使用

[FlowPort 0.2.0](../../README.zh-TW.md) · [使用指南](https://flowport.pages.dev/zh-TW/docs/pipeline/)

使用 GuanceCloud Platypus Go 執行器，語法不是 Vector VRL。源碼、畫布、快捷指令編輯同一型別腳本，可匯入/匯出 .ppl。資源載入失敗可保留草稿重試。工作區至少 600px、總高至少 720px，高視窗延展至視口 80%，面板獨立捲動。

以下畫面於 2026-10-20 使用隔離實例與合成資料，從 0.2.0 候選構建的英文介面拍攝。所示介面已包含於 0.2.0，截圖清單保留原始拍攝構建資訊。地址及數值僅為範例，可點選圖片放大。

## 編輯與樣本

1. 進入 Pipeline 並選訊號型別。
2. 編輯源碼或加入函式、賦值、條件及迴圈，所有視圖同步。
3. 開啟樣本除錯；關閉/Escape 返回編排並保留輸入結果。執行中關閉取消瀏覽器等待，不保證中止已啟動的伺服器執行。
4. 輸入 JSON 物件/陣列，可格式化後執行或 Command/Ctrl+Enter。最多 100 筆，不傳送下游。
5. 查看前後、輸出/過濾數、名稱、Tag、型別與奈秒時間。可切換多輸出/子 Point；單筆入出會標示新增、修改、移除及舊值。
6. 關閉修改、重新執行，再儲存/發布。

處理前是解碼後 Point 而非原始 JSON；原始 JSON 複製保持 before/results 數字文本。欄位每頁 50 筆，過濾取執行器報告而非輸入減輸出。多筆不按位置對應，耗時包含網路。

HTTP json/ndjson/text 是接入格式，樣本使用 JSON。大整數與奈秒文本保持精度。格式化最多 64 層，更深仍可直接提交。修改後舊結果標為過期；解析/執行失敗可修正重試。

![畫布顯示 Lookup 呼叫、Tag 擷取與 healthcheck 過濾；縮放控制可查看完整處理步驟。](https://flowport.pages.dev/assets/screenshots/pipeline-canvas.webp)

*畫布顯示 Lookup 呼叫、Tag 擷取與 healthcheck 過濾；縮放控制可查看完整處理步驟。*

![原始碼檢視編輯同一份 Platypus 腳本：呼叫 lookup("service_info")、擷取 Tag 並過濾 healthcheck。](https://flowport.pages.dev/assets/screenshots/pipeline-source.webp)

*原始碼檢視編輯同一份 Platypus 腳本：呼叫 lookup("service_info")、擷取 Tag 並過濾 healthcheck。*

![真實 Go 執行器處理兩筆樣本：保留一筆、過濾一筆，並顯示輸出欄位與 Tag；樣本不會傳送到目標。](https://flowport.pages.dev/assets/screenshots/pipeline-debug.webp)

*真實 Go 執行器處理兩筆樣本：保留一筆、過濾一筆，並顯示輸出欄位與 Tag；樣本不會傳送到目標。*

## Tag 與過濾

{{code:0}}

測試範例：

{{code:1}}

第一筆保留並增加 Tag，第二筆被過濾。空腳本透傳，不自動將普通欄位轉 Tag。

## 組合分流的 Tag

明確轉換普通 environment、region、tenant：

{{code:2}}

再設定 (env=prod AND region=cn) OR tenant=vip。最終 Tag 區分大小寫、不裁剪；驗證兩類命中和未命中資料的實際下游收據。詳見[分流](https://flowport.pages.dev/zh-TW/docs/pipeline/)。

## 匯入、匯出與發布

匯入 .ppl 後核對型別/樣本。匯出是處理腳本備份，不包含 Source/目標認證或完整流。圖表示語法，Go 驗證最終可執行性。不正確語法和不可用函式不能發布。超時、無法完成迴圈、解析錯誤留下完整原批次以便重播。

目前失敗處理與遷移: [FlowPort](https://flowport.pages.dev/zh-TW/docs/delivery/).

## 版本與失敗釋放

從 0.1.16 起，傳送失敗保留在正常本地佇列，依 1、2、4、8、16、32 秒退避，之後最多每 60 秒重試。認證、設定、HTTP413、限流與不完整確認都保留該分支；正常目標獨立傳送。解析或 Pipeline 失敗仍釋放輸入。不提供獨立失敗頁、部分補傳、413 拆批或人工重放。

重試保留原處理結果、目標 ID 與投遞身份。重新發布後，同一目標 ID 的積壓使用新連線設定，不重跑 Pipeline 或重做分流；新版本移除目標時，原分支繼續用原連線，不轉送其他目標。計數仍歸屬原輸入版本，僅儲存目標或修改草稿不會更新積壓。

processing_failed_batches 計輸入失敗，sending_failed_batches 保留相容欄位名但計失敗傳送嘗試（含重試），失敗率為失敗嘗試除以失敗嘗試加 sent_batches。overflow_dropped_batches / overflow_dropped_bytes 另計 FIFO 淘汰的輸入或輸出分支；discarded_bytes 包含處理失敗及 FIFO 淘汰，傳送失敗駐留不增加。接收、淘汰、計數與有序收據共用持久交易；寫入失敗保留舊載荷且不確認新資料，重試不完成收據。已開始的請求無法撤回，淘汰分支仍可能到達遠端，但不會重新入列；逾時與崩潰仍可能重複。

## 0.1.21

依目前草稿與已儲存的目標類型篩選預演，最多顯示 100 個輸出點。不連線目標、不證明寫入權限；未命中規則使用預設目標，類型篩選後不會繼續匹配下一條規則。


## Lookup

Lookup 使用本機不可變資料，批次固定版本。資源不可用保留佇列輸入。詳見 [Lookup](lookup.md)。目前控制協定 v6、檢查點格式 3，需協調升級，降級恢復升級前完整備份。


## 快取與視窗

在 Pipeline 的「快取與視窗」面板選擇範例並插入腳本，依輸入欄位調整後執行樣本。支援 `cache_get`、`cache_set`、`point_window`、`window_hit`、`agg_create`、`agg_metric`，可與 `lookup()`、條件及分流搭配。

有界記憶體依 **Worker × 流發佈版本 × 輸入資料類型** 隔離，同一範圍內跨批次共用。重新啟動或遷移 Worker 後清空，不提供全域去重、跨 Worker 聚合或恰好一次保證。重新發佈使用新狀態；舊版本處理完已接收輸入後輸出最後一個未滿聚合視窗。停止流也如此。連續 1 小時沒有成功處理輸入時，輸出剩餘聚合並回收，未命中的上下文直接清除。刪除流會清除記憶體狀態。

每次樣本除錯使用全新獨立狀態，結束時主動輸出未滿聚合視窗（`stateful_preview=true`），不存取執行中狀態。時間間隔、跨批次與重新啟動行為須使用已發佈流和真實目標驗證。

![快取與視窗面板提供可插入的場景範例，並說明狀態隔離、容量限制與重新啟動清空行為。](https://flowport.pages.dev/assets/screenshots/pipeline-stateful.webp)

*快取與視窗面板提供可插入的場景範例，並說明狀態隔離、容量限制與重新啟動清空行為。*

### 短期快取去重

`cache_set(key, value, exp=100)` 儲存字串，`exp` 單位為秒。再次寫入替換值並刷新 TTL。`cache_get(key)` 回傳字串，未命中或過期回傳 `nil`。鍵和值不隱式轉型。

{{code:3}}

{{code:4}}

輸出 `first` 和 `second`；同一執行範圍的後續批次也會在 60 秒內過濾 `a`。每次重新執行樣本仍輸出兩筆。各 Worker 獨立判斷。

### 保留錯誤上下文

`point_window(before, after, stream_tags=["filepath", "host"])` 依字串欄位或 Tag 分組；`window_hit()` 必須在它之後呼叫以標記命中。`after` 包含命中記錄，`before` 不包含；`after=0` 只恢復先前記錄。以 `drop()` 過濾視窗外記錄，每筆最多呼叫一次 `point_window()`。

{{code:5}}

{{code:6}}

輸出 `n=1,2,3,4`，`n=5` 保留於快取。未被過濾的記錄立即輸出，但仍占視窗位置且命中時不重複輸出。恢復的是先前處理完成的 Point，保留其發佈與 Lookup 版本、欄位修改，不重新執行腳本或 Lookup。快取記錄被擠出才計入過濾，不可用輸入減輸出推算過濾數。其他流不等待該視窗，也沒有全域順序。

### 視窗聚合指標

`agg_create(bucket, on_interval="60s", on_count=0, keep_value=false, const_tags=nil, category="M")` 定義桶，時間或筆數任一滿足就輸出。`"0s"` 關閉時間觸發，`0` 關閉筆數觸發，不能同時關閉。同一桶每筆參與記錄只計一次，與 `agg_metric()` 呼叫次數無關；筆數針對整個桶，而非每個分組。使用 Worker 處理時間，非事件時間；空閒時也約每 300ms 檢查，負載可能增加延遲。

`agg_metric(bucket, new_field, agg_fn, agg_by, agg_field, category="M")` 支援 `sum/avg/min/max/set`，`set` 取最後值。分組欄位或 Tag 必須為字串，聚合欄位必須為數值，文字須明確轉型。float64 聚合不保證大整數或精確小數無損。桶名為測量名，分組值為 Tag；`"M"` 或 `"metric"` 輸出指標，`const_tags` 可加固定字串 Tag。原始記錄需用 `drop()` 明確過濾。

{{code:7}}

{{code:8}}

樣本結束輸出一筆 `metric`：測量名 `requests`、Tag `service=api`、欄位 `count=2` 和 `duration_avg=20`。正式執行等待 60 秒或 100 筆；傳送目標須允許 `metric`。生成指標直接進入分流與目標選擇，不再執行 metric Pipeline，也不自動複製其他原始 Tag；請用分組鍵或 `const_tags`。預設輸出後清空值；`keep_value=true` 保留累計值，空閒時間視窗也可能重複輸出相同值，應按累計指標處理。

### 容量與失敗行為

| 項目 | 上限 |
|---|---|
| 狀態預算 | 每範圍 8 MiB；每 Worker 已提交狀態 64 MiB；128 個範圍 |
| 快取 | 4,096 個鍵；鍵 1–1,024 位元組，值最多 64 KiB；TTL 1–3,600 秒，預設 100 |
| 上下文 | 256 個分組；before/after 各 0–128，不可同為 0；1–8 個鍵，鍵最多 1 KiB、值最多 4 KiB |
| 聚合 | 16 個桶；每桶 4,096 組，每組 64 欄位；1–8 個分組鍵；鍵及輸出欄位最多 256 位元組、分組值最多 4 KiB |
| 觸發 | 1 秒–1 小時；1–1,000,000 筆；桶名最多 256 位元組 |
| 固定 Tag | 32 項；鍵最多 256 位元組，值最多 4 KiB |

預算保守估算字串、快取 Point 與索引開銷，不等於 RSS 上限。瞬時記憶體另含最多 8 個並行批次的狀態副本、解碼與輸出。參數錯誤、缺少分組鍵、型別不符或超限明確報錯，依一般 Pipeline 失敗規則釋放輸入。

整個批次（含所有 CDC 分塊）的處理結果成功入隊才提交狀態；中途失敗不保留局部修改。定時輸出入隊失敗保留視窗並顯示 Worker 錯誤，等待重試。已入隊結果使用一般持久佇列重試，重新啟動可恢復且不再聚合。記憶體中的快取、上下文和聚合不納入備份，重新啟動會遺失；已接收但未處理輸入會以空狀態恢復，結果可能與不中斷執行不同。
