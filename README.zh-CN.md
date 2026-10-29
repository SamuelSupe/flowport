[![FlowPorts — Collect. Transform. Deliver.](docs/images/flowports-cover.svg)](https://flowport.pages.dev/)

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

# 连接、处理，让数据抵达需要它的地方。

把日志、指标、链路与事件接入 **Source → Pipeline → Destinations**，在来源与观测、分析系统之间完成处理和转发。

**[官网](https://flowport.pages.dev/) · [快速开始](https://flowport.pages.dev/docs/) · [Linux 下载](https://flowport.pages.dev/downloads/) · [版本说明](https://flowport.pages.dev/changelog/#v0-5-3)**

## 0.5.3 更新

- 来源、目的端或容量分配尚未完成时，也能保存草稿；发布时再严格检查完整配置。
- 在数据流编辑器内直接创建并选中发送目标，保留已经填写的来源与 Pipeline。
- 发布后验证本次新增接收、成功处理和每个目标的新增确认，旧积压与历史成功不算本次结果。

## 从来源到送达，一条完整路径

| 接入 | 处理 | 发送 |
| --- | --- | --- |
| 72 种 Source，覆盖应用、基础设施、数据库、消息系统和云服务。 | 通过 Pipeline 解析和转换，使用本地 Lookup 富化，按 Tag 分流，并支持有界缓存与窗口。 | 48 种 Destination，目标通道独立投递，在运行页查看确认、积压与异常。 |

[![FlowPort 0.5.3 本机 Chrome 实拍：HTTP 数据完成接收、处理与本地目标确认。使用隔离测试实例与合成数据，凭证保持打码。](docs/images/first-delivery-0.5.3.jpg)](docs/images/first-delivery-0.5.3.jpg)

*FlowPort 0.5.3 本机 Chrome 实拍：HTTP 数据完成接收、处理与本地目标确认。使用隔离测试实例与合成数据，凭证保持打码。*

## 让第一条数据走通

1. [下载 Linux 安装包](https://flowport.pages.dev/downloads/)并核对 SHA-256。支持 amd64 / arm64 的 deb、rpm 和 tar.gz，安装运行无需 Node.js、npm 或 Go 编译器。
2. 完成 Web 初始化，创建数据流并选择来源。可以随时保存草稿；不需要转换时，空 Pipeline 直接透传。
3. 在编辑器添加目标，发布并等待 Worker 安装。先开始送达验证，再发送匹配样本，最后在下游核对实际内容。

## 使用指南

- **[数据流与路由](docs/zh-CN/flows.md)** — 创建、保存、发布，以及单流配置导入导出。
- **[Lookup 数据富化](docs/zh-CN/lookup.md)** — 本地匹配、后台同步与不可变数据版本。
- **[Pipeline 与窗口](docs/zh-CN/pipeline.md)** — 解析、短期去重、上下文和窗口聚合。

[完整文档目录](docs/index.md) · [交付与恢复](https://flowport.pages.dev/docs/delivery/) · [版本与授权](https://flowport.pages.dev/editions/)

## 清楚理解交付边界

- 接入确认只表示当前 Worker 已持久接受，不等于目标送达。各 Worker 队列独立，不跨节点复制。
- 目的端首次发送后默认最多**重试 5 次**。耗尽即丢弃未确认数据，没有死信重放；随后至少等待 60 秒，再试投一批。
- 新数据流默认每 Worker **256 MiB、500 个内部批次**，固定分配给输入与各目的端。输入满时拒收新数据；输出满时只淘汰本通道。已有明确配额不自动改变。
- 协议确认不等于下游业务处理完成。确认丢失可能导致重复，不保证严格只发送一次或跨目标事务。

### 升级前

0.5.3 使用控制协议 **v8**、检查点格式 **3**。先备份各节点完整数据、master.key、Lookup 与环境配置，再协调更新管理端及全部执行 Worker。历史积压保留原重试次数；内网 HTTP 读取需要逐节点配置必要 CIDR。

## 问题与反馈

通过 [Issues](https://github.com/SamuelSupe/flowport/issues/new/choose)反馈产品问题、可复现故障或文档修正，请附版本和脱敏示例。

---

这里是公开文档入口。官网品牌为 FlowPorts，安装的软件仍使用 FlowPort 名称。本仓库不包含产品源码、官网源码、源码历史或安装包。社区包提供单本机 Worker；商业多 Worker 部署需要发行方提供的构建与授权。
