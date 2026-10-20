# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[官网](https://flowport.pages.dev/) · [下载](https://flowport.pages.dev/downloads/) · [完整文档](https://flowport.pages.dev/docs/) · [版本记录](https://flowport.pages.dev/changelog/)

FlowPort 是面向日志、指标、链路与事件的数据采集和路由平台，通过 Source → Pipeline → Destinations 配置采集、处理与多目标发送。

**本仓库仅发布产品文档和官网链接，不发布产品或官网源码、源码历史及安装包。** Linux 安装包、校验清单和版本信息由 Cloudflare 官网提供。产品许可与商业版本说明以官网为准；文档公开不代表产品源码开源。

## 0.2.1

- 修复 Kafka 将事件时间用于固定 20 秒投递超时判断的问题，历史日志、指标、链路及其他 Point 数据不再因时间较旧而在发送前失败。Kafka 消息时间与编码载荷中的原始事件时间保持不变，适用于全部 Kafka 输出及复用该协议的服务预设。

- 每次 Kafka 发送由 config.timeout_seconds 控制，默认 20 秒、范围 1–120 秒，并遵守上层更短的取消或截止时间。重试从新一次发送开始计时；发送失败仍保留原队列和处理结果，不通过改写事件时间或清空积压恢复。

- 0.2.1 沿用控制协议 v6 和检查点格式 3。停止服务并备份完整数据目录、master.key、Lookup 资源与环境配置后升级管理端及全部 Worker。恢复原数据目录后，保留的 Kafka 积压自动重试；已被 FIFO 淘汰的数据无法恢复。目标 Broker 或云服务自身的时间范围与保留策略仍适用。

## 0.2.0 新能力

- Lookup 支持 Exact 复合键、IPv4/IPv6 CIDR、RE2 Regex 和 MMDB；可上传 CSV、CSV.GZ、MMDB，或从 HTTP(S)、S3、PostgreSQL、MySQL 后台同步完整快照。Pipeline 使用 lookup("绑定名称") 本地查询并映射字段或 Tag。
- Lookup 支持自动更新、固定版本、试查、历史回滚与 Worker 安装状态。批次入队时固定数据版本；同步失败保留最后有效版本，队列所需资源缺失时暂停处理并保留载荷。回滚会暂停自动同步，恢复须显式操作。
- 开放 cache_get/cache_set、point_window/window_hit、agg_create/agg_metric，支持短期去重、错误上下文与计数/时间窗口聚合。有界内存按 Worker、流发布版本及输入类型隔离，重启清空；样本调试不影响运行状态。
- 新增五种场景模板，以及单流配置导入导出（flowport.flow v1，最大 2 MiB）。导入时映射本地目标和 Lookup，检查后保存并显式发布。配置包不包含队列、游标、Lookup 数据或内存状态；连接凭证移除，脚本/SQL 字面值仍须检查。优化配置布局、异步取消、资源读取复用和多绑定导出。

## 升级至 0.2.0

控制协议升级至 v6，检查点格式升级至 3。停止服务，备份每个节点的完整数据目录、master.key、Lookup 资源和环境配置，再协调升级管理端与全部 Worker。回退须恢复升级前完整备份。提供 Linux ARM64/x86_64 的 tar.gz、deb、rpm；安装与运行无需 Node.js。

队列所需的固定 Lookup 资源缺失时暂停处理并等待恢复，不按普通 Pipeline 失败释放输入。内存缓存、上下文和聚合状态重启清空；已经入队的输出保留原结果。

## 使用指南

- [Lookup 数据富化](docs/zh-CN/lookup.md)
- [缓存与窗口](docs/zh-CN/pipeline.md)
- [场景模板与导入导出](docs/zh-CN/flows.md)

## 开始使用

- [快速开始](docs/getting-started.md)：安装、登录、发布与验证第一条数据。
- [交付与恢复](docs/delivery.md)：本地队列、持续重试、FIFO 容量淘汰与告警语义。
- [运维与升级](docs/operations.md)：监控、备份、升级与集群边界。
- [完整文档目录](docs/index.md)：Pipeline、集成、API/MCP 与五语言入口。

## 能力与适用范围

- 72 类 Source、48 类 Destination；支持按处理后的 Tag 组合分流。
- Linux amd64 / arm64，提供 deb、rpm、tar.gz；运行不需要 Node.js、npm 或 Go 编译器。
- 单个静态 Go 二进制内置管理 UI、帮助、Pipeline 执行器与原生持久队列。
- 社区下载包采用单 Worker；商业多 Worker 部署需使用发行方提供的商业构建与匹配 License。
- 发送目标失败时继续接入，保留失败分支并退避重试；容量超限按原始接收顺序 FIFO 淘汰。

本地持久接受不等于下游送达，也不保证严格只发送一次。

## 下载与版本

当前文档基线：**0.2.1**。始终从[官网下载页](https://flowport.pages.dev/downloads/)获取当前版本，核对同版本 SHA-256；[版本清单](https://flowport.pages.dev/downloads/manifest/)和[更新记录](https://flowport.pages.dev/changelog/)说明版本及升级边界。

0.2.0 全新安装通过 [Web 初始化向导](https://flowport.pages.dev/docs/#web-setup) 设置管理员密码、管理/HTTP 接入监听域名或 IP 和端口，启动后读取本机 `manager/setup-token`。已有实例保留密码与队列；离线救援和可校验备份见[运维说明](https://flowport.pages.dev/docs/delivery/#offline-maintenance)。

## 反馈

可在 [GitHub Issues](https://github.com/SamuelSupe/flowport/issues) 提交公开的使用问题或文档反馈。请提供版本、系统和脱敏复现步骤，不要上传密码、Token、私钥、业务载荷或包含敏感信息的数据目录。
