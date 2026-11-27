# 交付、容量与恢复

[FlowPorts 0.5.15](../README.zh-CN.md) · [完整交付指南](https://flowport.pages.dev/docs/delivery/) · [English](https://flowport.pages.dev/en/docs/delivery/)

## 从接收到送达

接收确认对应当前 Worker 的本地持久入队。各 Worker 的队列独立保存。目标确认对应协议回执，响应丢失仍可能导致重复投递。

| 阶段或事件 | 行为 |
| --- | --- |
| 输入区满 | 拒收新输入，HTTP 返回可重试的错误。 |
| 输出区满 | 保留整个待处理输入或窗口，容量释放后再提交。 |
| 发送失败 | 保留未确认载荷并退避重试。 |
| Pipeline 状态容量达到上限 | 保留队头和来源回执，持久退避后重试。 |
| 解析或其他 Pipeline 失败 | 记录诊断，按错误类型保留或释放输入。 |

## 通道容量与重试

目的端首次发送后默认快速重试 5 次。快速重试耗尽后保留未确认数据，通道至少等待 60 秒，再试投一批。 新数据流默认每 Worker 256 MiB 和 500 个内部批次，分配给输入和各目的端。输出区满时保留原输入，输入区满时拒收新数据。

节点上限还受字节预算、10000 个内部批次和实际磁盘余量限制。各区域按各自配额运行。修改同一个目标的连接并重新发布后，旧积压用新连接重试，处理结果保留。已移除的目标沿用历史连接排空。

## Socket 发布与回压诊断

Socket 来源设置相同时，原有 TCP 和 UDP 监听器继续运行。Worker 安装新版后，即使沿用原 TCP 连接，新入队记录也使用新版 Pipeline。积压批次沿用入队时的版本。

输入队列满时，TCP Socket、Syslog 和 StatsD 关闭受影响的连接，并统计已解码但未持久入队的记录。运行视图显示拒收原因，API 提供 source_rejected_records，Prometheus 提供 collectway_flow_source_rejected_records_total。已入队积压继续保留。

普通 TCP Socket 缺少逐批持久入队确认。容量恢复后客户端可重连。断开时服务端尚未读到的字节无法计数。

## 部分回执与退避

投递保留有效的部分回执，只重试未确认记录。缺少协议回执的写出计入 sent_without_receipt。

支持的目标将 Retry-After 与本地退避一并持久化。队头诊断显示实际重试时间。

## 重启与恢复

Worker 从本地日志恢复队列和持久窗口。批次入队时固定的 Pipeline 与 Lookup 版本继续用于处理，已生成的输出重试时使用原处理结果。升级前备份整个节点数据目录、密钥、Lookup 资源与环境配置。线上历史错误和计数可在运行视图与管理 API 检查。

### Pipeline 超时恢复

Pipeline 在独立进程运行。超时可以终止卡住的内置函数，已入队输入和收据保留重试。

### 0.5.11 接入确认与恢复

- Beats 将压缩窗口分组落盘，只对已持久化的前缀发送一次累计确认。Beats 和 Fluent 在等待落盘或写确认时释放共享解码名额。
- Datadog Logs 在返回收据前校验 JSON 和批次上限。格式错误返回 HTTP 400，超限返回 HTTP 413，队列满返回可重试的 HTTP 503。
- CDC 快照、SLS、搜索和 Kinesis 采用有界分批，并在恢复时使用已持久化的进度。CloudWatch Logs 以压缩块保存去重游标。
- NATS Core 断连时保留已取出的缓冲消息，MQTT、Pulsar 和 Pub/Sub 在确认前复检 Worker 接入资格。集群交接使用原归属最后上报的已持久化游标。

### 0.5.12 指标、TLS 与 Lookup

- MongoDB 的宽 serverStatus 结果拆成多个点，共用采样时间戳并在同一批次接收。嵌套 BSON 使用驱动默认解码。
- PostgreSQL 切换到备用主机时继续使用 TLS，直接 TLS 模式保留 ALPN。MongoDB 保留 URI 中未被覆盖的信任根和客户端证书。
- Azure Blob 和 Service Bus 分别校验证书。Pulsar 每次建立新连接时重读私有 CA，指定 TLS 服务端名称时也重读客户端证书和私钥。
- 管理端最多同时下载两个 Lookup 文件，文件大小合计最多 128 MiB。内存紧张时先回收空闲缓存。管理面传输和 Worker 下载遇到临时容量不足时返回 HTTP 503 和 Retry-After。

### 0.5.13 MaxCompute

- 追加写入已有表，支持动态分区、列映射以及 AccessKey 或 STS 凭证。
- Worker 将包回执写入 WAL，只重试未确认的记录。同一 Worker 上使用相同 Tunnel Endpoint、Project 和 Quota 的流，在收到 HTTP 429 或 503 后共享退避。
- 交付采用至少一次语义。写入成功后若回执丢失，重试可能产生重复行。

[MaxCompute 配置指南](https://flowport.pages.dev/docs/integrations/#maxcompute)

### 0.5.15 Pipeline 状态容量恢复

遇到 `state_limit` 时，Worker 回滚本次试跑的状态和部分输出，保留原批次、入队时的 Pipeline 版本及来源回执，持久退避后重试。持续超限时可检查或导出队头。旧版已释放的载荷须从上游副本重放。

### 0.5.14 投递与恢复

- HTTP 目标明确返回 413 时，FlowPort 按记录拆批并保留已确认回执。单条记录仍超限时，保留在队列中退避重试。
- Kafka 在首次拉取前保存并提交 latest 位点。Kinesis 和 SLS 按分片独立读取及退避。对象存储目标重试同一持久批次时复用对象名。
- Worker 在投递引擎退出后上报终态。Manager 持久化归属、游标、授权及命令变更，普通续租更新内存快照。
- Pipeline 超时后保留队列输入及收据重试，窗口刷新使用批次预算。预览保留大整数精度，长字段可按需展开。
