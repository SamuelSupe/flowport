# 运维与升级

[FlowPorts 0.5.3](../README.zh-CN.md) · [运维文档](https://flowport.pages.dev/docs/delivery/) · [集群指南](https://flowport.pages.dev/docs/cluster/)

## 日常检查

- 区分接收、处理、每个目标确认、重试耗尽、容量丢弃和活动告警。查看 Worker 报告是否新鲜；离线报告不代表当前健康。
- 统一抓取管理端口的 `/metrics`，Worker 状态来自控制报告；**Worker 接入端口不提供 `/metrics`**。使用系统设置中的监控配置，按部署方式配置 Prometheus / Alertmanager。
- 分别检查逻辑队列预算、区域预留与真实磁盘余量。不能把日志、master.key 或检查点当作缓存删除。
- 故障持续期间，扩大队列不能保证不丢数据；还需结合上游背压、暂停接入和目标恢复安排。

## 升级到 0.5.3

0.5.3 保持控制协议 **v8**、检查点格式 **3**。更早的 v6 协议与 v8 不兼容，管理端与全部执行 Worker 须协调升级。旧积压沿用原重试次数，升级后可能立即耗尽；先阅读[交付策略](delivery.md)和[完整版本说明](https://flowport.pages.dev/changelog/)。

1. 核对版本、积压、失败次数与各区域容量，安排必要的接入间断。
2. 停止相应服务，备份每个节点**整个数据目录**、master.key、Lookup 资源及实际使用的环境配置，包括 `/etc/flowport/flowport.env`。
3. 从[官网下载页](https://flowport.pages.dev/downloads/)获取同版本包与 SHA256SUMS，核对后安装，保留原数据目录与密钥。
4. 启动后检查版本、登录、Worker 安装状态、队列恢复、目标收据与告警，再验证一条真实数据。

```sh
sudo systemctl stop flowport
# 按官方指南完成完整备份后，再安装新包
sudo apt install ./flowport_0.5.3_linux_amd64.deb
collectway -version
sudo journalctl -u flowport -n 50
```

回退需恢复升级前完整备份，并考虑升级后新接收的数据；只换回旧二进制不构成完整回退。同一数据目录只能有一个有效写入实例，不能让多个 Worker 同时使用备份副本投递。

## 内网读取

默认拒绝内网 HTTP 读取。私网 Prometheus、Kubernetes、HTTP Client、Schema Registry、Lookup 等需要管理员在管理端及相关 Worker 上配置必要 CIDR。显式启动参数或非空环境变量优先于界面设置。来源、API Key 或 MCP 不能自行扩大此策略。

## 版本与授权

社区包提供单本机 Worker。商业集群需要发行方提供、包含可信公钥的商业构建和匹配的授权；公共社区二进制不能仅通过导入 License 变成商业集群。详见[版本与授权](https://flowport.pages.dev/editions/)。
