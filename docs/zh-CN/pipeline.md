# Pipeline 使用

[FlowPort 0.2.0](../../README.zh-CN.md) · [使用指南](https://flowport.pages.dev/docs/pipeline/)

FlowPort · 流港 使用 GuanceCloud Platypus 的 Go 执行器处理 Point，语法不是 Vector VRL。编辑器提供源码、画布和快捷指令视图，可导入/导出 `.ppl` 文件；不同数据类型使用各自的脚本。

若编辑器资源加载失败，可点击「重新加载编辑器」重试，不必刷新整个页面；当前草稿脚本保留。资源加载成功后会复用，不重复下载。

编排工作区至少保留 600px 高度，画布、快捷指令和源码视图切换时保持一致；编辑器总高度至少为 720px，并随较高窗口扩展至视口高度的 80%。矮窗口可滚动页面访问完整工作区，节点库、属性面板和源码内容在各自区域内滚动。

以下截图于 2026-10-20 从 0.2.0 候选构建的英文界面实拍，使用隔离实例与合成数据；所示界面已包含在 0.2.0 中，截图清单保留拍摄构建的原始信息。示例地址与计数仅作说明，点击图片可查看大图。

## 编辑与样本预览

1. 在流编辑页进入 Pipeline，选择要处理的数据类型。
2. 在源码直接编写脚本，或从节点库添加函数、赋值、条件和循环。不同视图同步表达同一脚本。
3. 点击处理规则右上角「样本调试」打开独立工作台。编排页不常驻调试面板；关闭或按 `Esc` 回到原编排位置，当前样本与结果保留。运行中关闭会取消浏览器等待，不保证中止已经开始的服务端执行。
4. 在工作台的「输入 JSON」填写一个对象或对象数组，可先格式化，再点击「运行样本」或在输入框按 `⌘ / Ctrl + Enter`。最多 100 条样本，调试不会发送到 Destination。
5. 查看输入、输出和过滤的数据点数量，在「处理后」检查名称、Tag、字段、JSON 值类型和纳秒时间戳。多条结果可以切换数据点；生成的子 Point 保留其数据类型。单条输入与单条输出会高亮新增、修改和移除的字段，并展示修改前的值。
6. 关闭工作台修改脚本，重新运行样本，再保存或发布。

「处理前」展示接入解码后的 Point，不等于原始输入 JSON；「原始 JSON」可查看完整 `before/results` 响应，并使用「复制 JSON」保留原始文本。字段较多时每页显示 50 项。多条输入可能被过滤或生成子 Point，因此不按位置推断字段对应关系；过滤计数直接读取执行器报告，不用输入与输出数量相减代替。请求耗时包含网络和服务处理时间，不是执行器单独耗时。

普通 HTTP Source 的 `json`、`ndjson`、`text` 是接入解码格式；编辑器样本预览使用 JSON。大整数和纳秒时间戳以原始 JSON 文本提交与展示；格式化和结构化查看同样保留数值文本，避免浏览器重新编码造成精度损失。格式化最多支持 64 层嵌套，超深 JSON 仍可直接提交调试。输入、脚本或类型改变后，上次结果会标记为已过期；解析和执行失败会展示错误，修复后可重新运行。

![画布展示 Lookup 调用、Tag 提取与 healthcheck 过滤；缩放控件用于查看完整处理步骤。](https://flowport.pages.dev/assets/screenshots/pipeline-canvas.webp)

*画布展示 Lookup 调用、Tag 提取与 healthcheck 过滤；缩放控件用于查看完整处理步骤。*

![源码视图编辑同一份 Platypus 脚本：调用 lookup("service_info")、提取 Tag 并过滤 healthcheck。](https://flowport.pages.dev/assets/screenshots/pipeline-source.webp)

*源码视图编辑同一份 Platypus 脚本：调用 lookup("service_info")、提取 Tag 并过滤 healthcheck。*

![真实 Go 执行器处理两条样本：保留一条、过滤一条，并展示输出字段与 Tag；样本不会发送到目标。](https://flowport.pages.dev/assets/screenshots/pipeline-debug.webp)

*真实 Go 执行器处理两条样本：保留一条、过滤一条，并展示输出字段与 Tag；样本不会发送到目标。*

## 设置 Tag 并过滤

JSON/NDJSON 顶层整数在 int64 范围内使用有符号字段；更大的非负整数在 uint64 范围内仍无损保留，例如 `18446744073709551615`。Pipeline 透传及 Point Protobuf 编码不会把它舍入成浮点数。小数、科学计数法使用浮点语义；超出 64 位整数范围或要求精确的小数应作为字符串上报。各 Destination 的数值范围仍以其协议和实际表结构为准。

Platypus 脚本整数为 int64，读取超过该范围的 unsigned 字段时提供精确十进位字符串；`set_tag("id", unsigned)` 可以保留完整数字，不变成负数。未改写的原 Point 字段仍是 uint64，脚本算术需遵守 int64 范围。

```ppl
set_tag("env", "production")
set_tag("service", "checkout")
if message == "healthcheck" {
  drop()
}
```

测试样本：

```json
[
  {"message":"order created","status":200},
  {"message":"healthcheck","status":200}
]
```

第一条保留并增加 Tag；第二条被过滤。空脚本直接透传，不会自动为普通字段创建同名 Tag。

## 为组合分流准备 Tag

假设 HTTP JSON 中包含 `environment`、`region`、`tenant` 普通字段，可显式转换为 Tag：

```ppl
set_tag("env", environment)
set_tag("region", region)
set_tag("tenant", tenant)
```

随后在发送目标步骤配置 `(env=prod AND region=cn) OR tenant=vip`。规则读取处理后的 Tag；Tag 名和值区分大小写，不自动裁剪。先预览这些 Tag，再验证两类命中数据和一类未命中数据的实际下游收据。详见 [组合分流](https://flowport.pages.dev/docs/pipeline/)。

## 导入、导出与发布校验

导入 `.ppl` 后核对对应数据类型和样本。导出用于备份脚本或复用处理规则，不包含来源和发送目标凭证，也不等于完整流备份。

编辑器展示语法结构，最终可执行性由 Go 执行器校验。无效语法或当前执行器不可用的函数不能发布；超时、无法完成的循环或输入解析错误进入处理失败。已接收批次释放载荷，只累计失败计数并保留有界摘要；修复并发布后处理新输入。

## 版本与失败释放

已接收批次继续使用接收时的处理版本。修改脚本并发布只作用于后续新输入，不重写正常未完成队列的旧数据。

发送失败保留原处理结果和目标 ID，在本地队列退避重试，超过每流或 Worker 容量按原始接收顺序 FIFO 淘汰。重新发布后，同一目标 ID 使用新连接配置；移除目标时保留原配置，不重新分流。处理失败仍释放输入。接收、淘汰、持久计数与有序收据原子提交，存储失败不能误报接收成功；在途请求与崩溃恢复仍可能重复。详见[队列契约](https://flowport.pages.dev/docs/delivery/)。

## 0.1.21 样本与路由预演

Pipeline 调试和发送目标步骤共用样本工作台，可切换「路由预演」查看处理后各点命中的首条规则、默认目标及 signals 筛选。最多展示 100 个输出点，支持子数据类型；不连接、不投递、不保存或发布。目标缺失或协议类型不支持会明确标记，类型筛选不继续匹配下一条规则。修改样本、脚本、路由或目标后需要重新运行；连接、试采样、路由预演与真实发送分别验证不同阶段，预演成功不能证明业务投递成功。


## Lookup

详见 [Lookup 数据富化](lookup.md)：本地四种匹配、后台完整同步、批次版本固定和独立资源文件。Lookup 不可用保留队列；与普通 Pipeline 处理失败区分。当前控制协议 v6，检查点格式 3，管理端与 Worker 协调升级，降级恢复升级前完整备份。

## 缓存与窗口

在 Pipeline 的「缓存与窗口」面板选择示例并点击「插入脚本」，按输入字段修改后运行样本。支持 `cache_get`、`cache_set`、`point_window`、`window_hit`、`agg_create`、`agg_metric`，可与 `lookup()`、条件和分流组合使用。

状态按 **Worker × 流发布版本 × 输入数据类型** 隔离，在同一作用域内跨批次复用。它是有界内存：重启或迁移 Worker 后清空，不提供集群全局去重、跨 Worker 聚合或恰好一次保证。重新发布使用新状态；旧版本先处理完已接收输入，再输出最后一个未满聚合窗口。停止流也会完成该过程。一个作用域连续 1 小时没有成功处理输入时输出剩余聚合并回收；没有命中的上下文缓存直接清除。删除流会清除其内存状态。

每次样本调试都创建独立状态，不读取或修改运行中状态；样本末尾主动输出未满聚合窗口，响应中的 `stateful_preview=true` 标识该模式。样本不能验证真实时间间隔、跨批次状态或重启恢复，请用已发布流和真实目标验证这些行为。

![缓存与窗口面板提供可插入的场景示例，并说明状态隔离、容量限制和重启清空行为。](https://flowport.pages.dev/assets/screenshots/pipeline-stateful.webp)

*缓存与窗口面板提供可插入的场景示例，并说明状态隔离、容量限制和重启清空行为。*

### 短期缓存去重

`cache_set(key, value, exp=100)` 保存字符串值，`exp` 为秒；再次写入会替换值并刷新 TTL。`cache_get(key)` 返回字符串，未命中或过期返回 `nil`。键和值不自动转换类型。

```ppl
if cache_get(event_id) != nil {
  drop()
} else {
  cache_set(event_id, "seen", 60)
}
```

```json
[{"event_id":"a","message":"first"},{"event_id":"a","message":"duplicate"},{"event_id":"b","message":"second"}]
```

输出 `first` 和 `second`；同一运行作用域后续批次中的 `a` 在 60 秒内也会被过滤。每次重新运行样本仍得到两条结果。多 Worker 接入时，各 Worker 独立判断。

### 保留错误上下文

`point_window(before, after, stream_tags=["filepath", "host"])` 按指定字符串字段或 Tag 分组；`window_hit()` 标记当前记录命中，必须在 `point_window()` 后调用。`after` 包含当前命中记录，`before` 不包含；`after=0` 只恢复之前的记录。用 `drop()` 过滤未命中且不在上下文窗口内的记录。每条记录最多调用一次 `point_window()`。

```ppl
point_window(2, 2, ["service"])
if level == "error" {
  window_hit()
}
drop()
```

```json
[{"service":"api","level":"info","n":1},{"service":"api","level":"info","n":2},{"service":"api","level":"error","n":3},{"service":"api","level":"info","n":4},{"service":"api","level":"info","n":5}]
```

输出 `n=1,2,3,4`，`n=5` 留在前置上下文缓存。未被 `drop()` 的记录立即输出，但仍占用窗口位置，不会在命中时重复输出。恢复的是之前处理完成的 Point，不会重新执行 Lookup 或脚本；其处理版本、Lookup 数据版本及字段变化保持原样。缓存中的记录在被挤出时才计入过滤，不能用输入减输出推算过滤数量。流之间没有全局顺序，其他流的输出不等待该窗口。

### 窗口聚合指标

`agg_create(bucket, on_interval="60s", on_count=0, keep_value=false, const_tags=nil, category="M")` 定义桶。时间和条数任一满足就输出；`"0s"` 禁用时间触发，`0` 禁用条数触发，两者不能同时禁用。同一桶每条参与记录只计数一次，即使调用了多个 `agg_metric()`；计数针对整个桶，并非每个分组。时间使用 Worker 处理时间，不是事件时间；空闲时也检查到期窗口，调度检查周期约 300ms，负载可能增加延迟。

`agg_metric(bucket, new_field, agg_fn, agg_by, agg_field, category="M")` 支持 `sum/avg/min/max/set`，其中 `set` 取最后值。分组键读取字符串字段或 Tag；输入聚合字段必须为数值，字符串需显式转换。聚合使用 float64，不保证大整数或精确小数无损。桶名成为输出测量名，分组值成为 Tag；`category="M"` 或 `"metric"` 输出指标。`const_tags` 可附加固定字符串 Tag。聚合不自动过滤原始记录，示例用 `drop()` 仅发送指标。

```ppl
agg_create("requests", on_interval="60s", on_count=100)
add_key(request_count, 1)
agg_metric("requests", "count", "sum", ["service"], "request_count")
agg_metric("requests", "duration_avg", "avg", ["service"], "duration")
drop()
```

```json
[{"service":"api","duration":10},{"service":"api","duration":30}]
```

样本结束输出一条 `metric`：测量名 `requests`，Tag `service=api`，字段 `count=2`、`duration_avg=20`。正式运行会等到 60 秒或 100 条；请在发送目标中允许 `metric`。生成的指标直接进入分流和目标选择，不再次执行 metric Pipeline；原记录的其他 Tag 也不会自动复制，需使用分组键或 `const_tags`。默认输出后清空聚合值；`keep_value=true` 保留累计值并按触发条件重复输出，空闲时间窗口也可能输出相同值，需按累积指标消费。

### 容量与失败行为

| 项目 | 边界 |
|---|---|
| 状态预算 | 每作用域 8 MiB；每 Worker 已提交状态 64 MiB；最多 128 个作用域 |
| 缓存 | 最多 4,096 个键；键 1–1,024 字节，值最多 64 KiB；TTL 1–3,600 秒，默认 100 秒 |
| 上下文窗口 | 最多 256 个分组；before/after 各 0–128，不能同时为 0；1–8 个分组键，键最多 1 KiB、值最多 4 KiB |
| 聚合 | 最多 16 个桶；每桶 4,096 个分组，每组 64 个字段；1–8 个分组键，键和输出字段最多 256 字节、分组值最多 4 KiB |
| 聚合触发 | 时间 1 秒–1 小时；条数 1–1,000,000；桶名最多 256 字节 |
| 固定 Tag | 最多 32 项；键最多 256 字节，值最多 4 KiB |

预算按保守估算计入字符串、缓存点及索引开销，不等于进程 RSS。批次执行使用候选副本，瞬时内存还包含最多 8 个并行批次的状态副本、输出和解码数据。参数错误、缺少分组键、类型不符和容量超限明确报错，遵循普通 Pipeline 处理失败释放输入的规则。

一个批次（包括所有 CDC 分块）的状态只有在处理结果成功入队后才生效，处理中途失败不会保留局部修改。定时输出无法入队时保留窗口并显示 Worker 错误，后续重试；已入队结果由普通持久队列重试，重启后可恢复，重发不重新聚合。尚在内存中的缓存、上下文和聚合不会写入备份，重启会丢失；已接收但尚未处理的输入恢复后使用新的空状态，因此结果可能与未重启时不同。
