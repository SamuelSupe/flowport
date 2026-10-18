# 快速开始

[返回首页](../README.md) · [完整安装指南](https://flowport.pages.dev/docs/) · [English guide](https://flowport.pages.dev/en/docs/)

## 安装与登录

1. 在[官网下载页](https://flowport.pages.dev/downloads/)选择 Linux 架构与安装格式，下载同版本校验清单并校验 SHA-256。
2. Debian / Ubuntu 使用 `sudo apt install ./flowport_<VERSION>_linux_amd64.deb`；RHEL 系使用对应 rpm 和 `sudo dnf install`；Alpine 等使用 tar.gz，解压后执行 `sudo sh install.sh`。ARM64 将文件名架构替换为 `arm64`。
3. systemd 使用 `sudo systemctl status flowport`；Alpine / OpenRC 使用 `sudo rc-service flowport status`。Alpine 先安装 `ca-certificates` 和 `openrc`。
4. 首次安装生成独立管理密码，通过 `sudo cat /etc/flowport/flowport.env` 查看；文件包含敏感配置，不要公开。访问 `http://127.0.0.1:8080` 登录，随后可在系统设置修改密码。

远程管理可建立 SSH 隧道：`ssh -L 8080:127.0.0.1:8080 user@server`。安装包默认管理端监听 `127.0.0.1:8080`，HTTP 接入监听 `0.0.0.0:8081`；对公网开放时配置 HTTPS、访问限制与独立 Source Token。修改 env 后重启服务。直接运行二进制的默认监听地址和配置方式不同，见官网指南。

## 走通数据链路

1. 在「发送目标」添加真实可达的目标。
2. 创建数据流，选择 HTTP Server、JSON 格式，设置独立 Source Token。
3. 配置 Pipeline 与默认目标；需要时按处理后的 Tag 设置路由。
4. 保存后发布，等待 Worker 安装当前版本；保存草稿本身不会切换运行配置。
5. 按运行页面生成的地址发送样本，核对目标实际收据、队列与异常统计。

```sh
curl -i -X POST 'http://127.0.0.1:8081/ingest/YOUR_FLOW_ID' \
  -H 'Authorization: Bearer YOUR_SOURCE_TOKEN' \
  -H 'Content-Type: application/json' \
  --data '{"message":"hello FlowPort","service":"checkout"}'
```

替换示例中的流 ID 和 Token。HTTP 204 仅确认本地持久接受；发送失败可能继续驻留重试，也可能在容量超限后被 FIFO 淘汰。完整语义见[交付与恢复](delivery.md)。
