# 第一条数据，从接入到送达

[FlowPorts 0.5.3](../README.zh-CN.md) · [完整安装指南](https://flowport.pages.dev/docs/) · [English](https://flowport.pages.dev/en/docs/)

## 1. 安装与初始化

在[官网下载页](https://flowport.pages.dev/downloads/)选择 amd64 或 arm64 的 deb、rpm、tar.gz 安装包，将同版本 SHA256SUMS 放在同一目录并校验。安装及运行无需 Node.js、npm 或 Go 编译器。

```sh
sha256sum --ignore-missing -c SHA256SUMS
# Debian / Ubuntu；ARM64 将 amd64 改为 arm64
sudo apt install ./flowport_0.5.3_linux_amd64.deb
sudo systemctl status flowport
sudo cat /var/lib/flowport/manager/setup-token
```

RHEL 系使用 rpm 与 `sudo dnf install`；Alpine 使用 tar.gz，先安装 `ca-certificates` 和 `openrc`，解压后执行包内 `sudo sh install.sh`。

新实例使用本机一次性 setup-token 完成 Web 初始化，设置管理员密码与监听。安装包默认管理地址为 `127.0.0.1:8080`，HTTP 接入地址为 `0.0.0.0:8081`；已有实例保留密码和数据。远程管理可建立 SSH 隧道：

```sh
ssh -L 8080:127.0.0.1:8080 user@server
```

在本机打开 `http://127.0.0.1:8080`。公网部署使用 HTTPS 与访问控制，采集使用独立 Source Token；不要使用管理员密码接入数据。Web 保存的监听设置优先于启动参数和安装包 env，详见[监听与网络](https://flowport.pages.dev/docs/#network)。

## 2. 选择来源，保存草稿

在总览点击「创建第一条数据流」，使用 HTTP 日志转发模板或空白创建。选择 HTTP Server，解码格式为 JSON，填写名称与独立接入 Token。

可以先保存未完成的草稿，稍后继续配置。草稿不会启动采集或替换运行版本；已填写的配额仍须合法。发布前必须补齐目标与容量分配。

## 3. 添加目标并发布

不需要转换时直接进入发送目标，空 Pipeline 会透传。在「添加并选择目标」中创建可达的接收端，保存后自动选中。已有来源和处理规则保留；手工设置的配额不会自动重分配。

需要转换、Lookup 或路由时，先用样本调试检查结果。连接测试与样本调试都不能证明真实数据已送达。发布后进入运行页，等待当前版本被 Worker 安装、来源就绪并收到新鲜报告。

## 4. 开始验证，再发送数据

等待旧积压排空后点击「开始验证」，再发送一条匹配来源格式及路由的真实测试数据。替换以下流 ID、地址及 Token：

```sh
curl -i -X POST 'http://127.0.0.1:8081/ingest/YOUR_FLOW_ID' \
  -H 'Authorization: Bearer YOUR_SOURCE_TOKEN' \
  -H 'Content-Type: application/json' \
  --data-binary '{"message":"hello FlowPorts","service":"demo","level":"info"}'
```

观察「新接收 → 成功处理 → 各目标新增确认」，并在下游核对实际内容。HTTP 204 只确认本地持久接受，协议确认也不等于下游业务处理完成。分流时应为每条路由分别发送匹配样本。

![0.5.3 本地合成数据的实际送达验证](images/first-delivery-0.5.3.jpg)

验证依据同一 Worker 实例和版本的计数增量，不关联单条记录。旧积压、过期报告、计数重置和期间丢弃不能作为本次送达证据；切换 Worker、刷新或离开运行页后需重新验证。

目的端默认最多重试 5 次，耗尽或容量丢弃后的载荷不可重放。参见[交付与恢复](delivery.md)与[数据流完整指南](zh-CN/flows.md)。
