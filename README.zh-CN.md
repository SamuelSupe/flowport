# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[官网](https://flowport.pages.dev/) · [下载](https://flowport.pages.dev/downloads/) · [完整文档](https://flowport.pages.dev/docs/) · [版本记录](https://flowport.pages.dev/changelog/)

FlowPort 是面向日志、指标、链路与事件的数据采集和路由平台，通过 Source → Pipeline → Destinations 配置采集、处理与多目标发送。

**本仓库仅发布产品文档和官网链接，不发布产品或官网源码、源码历史及安装包。** Linux 安装包、校验清单和版本信息由 Cloudflare 官网提供。产品许可与商业版本说明以官网为准；文档公开不代表产品源码开源。

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

目录支持不等于所有第三方版本与云权限组合均已深度验收。请依据对应集成指南验证实际环境；本地持久接受不等于下游送达，也不保证严格只发送一次。

## 下载与版本

当前文档基线：**0.1.25**。始终从[官网下载页](https://flowport.pages.dev/downloads/)获取当前版本，核对同版本 SHA-256；[版本清单](https://flowport.pages.dev/downloads/manifest/)和[更新记录](https://flowport.pages.dev/changelog/)说明版本及升级边界。

0.1.25 全新安装通过 [Web 初始化向导](https://flowport.pages.dev/docs/#web-setup) 设置管理员密码、管理/HTTP 接入监听域名或 IP 和端口，启动后读取本机 `manager/setup-token`。已有实例保留密码与队列；离线救援和可校验备份见[运维说明](https://flowport.pages.dev/docs/delivery/#offline-maintenance)。

## 反馈

可在 [GitHub Issues](https://github.com/SamuelSupe/flowport/issues) 提交公开的使用问题或文档反馈。请提供版本、系统和脱敏复现步骤，不要上传密码、Token、私钥、业务载荷或包含敏感信息的数据目录。
