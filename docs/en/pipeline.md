# Using Pipeline

[FlowPorts 0.5.3](../../README.md) · [Docs](../index.md) · [Website](https://flowport.pages.dev/en/docs/)

FlowPort uses GuanceCloud Platypus's Go executor, not Vector VRL. Source, canvas and shortcuts views edit the same per-category script; .ppl import/export is supported. Failed editor resources can be retried without discarding the draft. The work area stays at least 600px high; total editor height is at least 720px and grows to 80% of tall viewports. Panels scroll independently.

The per-record wall-clock limit remains 100 ms. An accepted input batch gets at most three attempts: the first two budget failures retain it, roll back candidate state and wait 1 and 2 seconds; a third failure releases that batch as pipeline_error and records the processing failure and discarded bytes. The count survives restart. Retries keep the queued version and FIFO rules. Exhausted Worker shared bytes or scope slots retain input for retry; the 8 MiB per-scope limit, parsing, type and output errors still release input.

## Editing and sample preview

1. Open Pipeline and choose the signal category.
2. Edit source or add function, assignment, condition and loop nodes. All views stay synchronized.
3. Open Sample debugging. Closing or Escape returns to orchestration and preserves input/results. Closing while running cancels the browser wait, but does not guarantee cancellation of execution already started on the server.
4. Enter a JSON object or array, format if needed, then Run sample or Command/Ctrl+Enter. Up to 100 samples are supported; debugging never sends to a Destination.
5. Inspect before/after, output and dropped counts, measurement, Tags, fields, types and nanosecond timestamps. Switch between multiple outputs and child Point categories. One input/one output highlights added, changed and removed fields with previous values.
6. Close, edit and rerun, then save or publish.

Before means decoded Points, not the original JSON. Raw JSON exposes the complete before/results response, and copying retains the original numeric text. Field tables show 50 entries per page. Drops come from executor reports, not input minus output; multi-record outputs are not paired by position. Request duration includes network and service time.

HTTP json/ndjson/text are ingestion formats; sample debugging uses JSON. Formatting and display preserve large integers and nanosecond lexemes. Formatting and backend JSON decoding both support at most 64 nesting levels. Changed input, script or category marks old results stale. Parsing or execution errors can be corrected and rerun.

## Tags and filtering

{{code:0}}

Test with:

{{code:1}}

The first record survives with Tags; the second is dropped. Empty scripts pass through and do not automatically convert ordinary fields into Tags.

## Tags for combined routing

Convert environment, region and tenant fields explicitly:

{{code:2}}

Then configure (env=prod AND region=cn) OR tenant=vip. Rules read processed Tags with case-sensitive names/values and no trimming. Test both matching groups and an unmatched record, and confirm actual downstream receipts. See [Routing](https://flowport.pages.dev/en/docs/pipeline/).

## Import, export and publication checks

Current failure handling and migration: [FlowPort](https://flowport.pages.dev/en/docs/delivery/).

## Versions and failure release

Intake acknowledgment means durable local enqueue only. Each destination defaults to 5 retries after the first attempt (configurable 0–100). Exhaustion irreversibly discards unacknowledged data; there is no dead-letter payload or manual replay. Full input rejects new data; full output evicts by FIFO only within that channel. Shrinking preserves old backlog while draining. Processing failures may also release input. Publishing does not prove Worker installation or downstream receipt. Check the actual downstream content.

Retry preserves the processed body, destination ID and delivery identity. Republish the same destination ID to use its newly published connection settings on the next attempt, without rerunning Pipeline or new routing. If the new publication removes that target, its queued branch retains its original connection. Counters remain attributed to the original input version. Editing a draft or saving a Destination alone does not update backlog.

processing_failed_batches counts input failures; sending_failed_batches keeps its compatibility name but counts failed send attempts including retries. Send failure rate is failed attempts divided by failed attempts plus sent_batches. overflow_dropped_batches and overflow_dropped_bytes separately count FIFO evictions of inputs or output branches; discarded_bytes includes processing failure and FIFO eviction, never a retained send failure. Acceptance, eviction, counters and ordered receipts share a durable transaction; write failure preserves old payloads and does not acknowledge new data. A retry does not complete its receipt. In-flight requests cannot be recalled: an evicted branch may still reach the destination, but cannot reappear in the queue. Timeout and crash duplicates remain possible.

## 0.1.21

Previews the current draft and saved destination signal filters for up to 100 output points. Does not connect or verify write permissions. Unmatched points use defaults; signal filtering does not fall through to another rule.


## Lookup

Lookup uses local immutable data with batch-pinned versions. Unavailable resources retain queued input. See [Lookup](lookup.md). Current control protocol v8 and checkpoint format 3 require coordinated upgrades and a full pre-upgrade backup for downgrade.


## Cache and windows

Open **Cache and windows** in Pipeline, select an example, insert it and adapt the field names before running a sample. `cache_get`, `cache_set`, `point_window`, `window_hit`, `agg_create` and `agg_metric` can be combined with `lookup()`, conditions and routing.

Bounded memory is shared across batches only within **Worker × flow publication × input category**. Restarting or moving Workers clears it. There is no global deduplication, cross-Worker aggregation or exactly-once guarantee. Publishing starts new state; the old publication drains accepted inputs before flushing its final partial aggregate. Stopping does the same. After one hour without successfully processed input, a scope flushes remaining aggregates and is reclaimed; unmatched context is discarded. Deleting a flow clears its memory.

Each preview starts with fresh isolated state and flushes partial aggregates at the end (`stateful_preview=true`). It never accesses live state. Use a published flow and a real destination to verify timing, cross-batch state and restart behavior.

### Short-term deduplication

`cache_set(key, value, exp=100)` stores a string for `exp` seconds. Rewriting replaces the value and resets its TTL. `cache_get(key)` returns a string or `nil` when missing/expired. Keys and values are not implicitly converted.

{{code:3}}

{{code:4}}

Output is `first` and `second`. A later batch in the same live scope also drops `a` for 60 seconds. Every new preview returns two records again. Workers deduplicate independently.

### Keep error context

`point_window(before, after, stream_tags=["filepath", "host"])` groups by string fields or Tags. Call `window_hit()` after `point_window()` to mark a match. `after` includes the matching record; `before` excludes it. `after=0` restores only earlier records. Use `drop()` to suppress records outside a matching window. Call `point_window()` at most once per record.

{{code:5}}

{{code:6}}

Output is `n=1,2,3,4`; `n=5` remains buffered. Records not dropped are emitted immediately but still occupy window positions and are not duplicated on a hit. Restored Points retain their earlier processing, publication and Lookup version; neither Lookup nor the script runs again. Buffered records count as filtered only on eviction, so input minus output is not a filter count. Other streams do not wait for this window and have no global ordering.

### Aggregate window metrics

`agg_create(bucket, on_interval="60s", on_count=0, keep_value=false, const_tags=nil, category="M")` creates a bucket. Either trigger flushes it; `"0s"` disables time and `0` disables count, but both cannot be disabled. Each contributing record counts once per bucket, regardless of the number of `agg_metric()` calls. The count is bucket-wide, not per group. Time means Worker processing time, not event time. Idle windows are checked about every 300 ms; load can add delay.

`agg_metric(bucket, new_field, agg_fn, agg_by, agg_field, category="M")` supports `sum/avg/min/max/set`; `set` keeps the last value. Grouping fields or Tags must be strings, and aggregate input must be numeric; convert text explicitly. Arithmetic uses float64 and does not preserve all large integers or exact decimals. The bucket name becomes the measurement and grouping values become Tags. `"M"` and `"metric"` produce metrics; `const_tags` adds fixed string Tags. Original records are retained unless explicitly dropped.

{{code:7}}

{{code:8}}

The preview flushes one metric: measurement `requests`, Tag `service=api`, fields `count=2`, `duration_avg=20`. Live processing waits for 60 seconds or 100 records. Enable `metric` at the destination. Generated metrics go directly to routing and target selection, without another metric Pipeline pass. Other original Tags are not copied; use grouping or `const_tags`. By default values reset after output. `keep_value=true` retains cumulative values and can emit identical values during idle time windows; consume these as cumulative metrics.

### Capacity and failures

| Item | Limit |
|---|---|
| State budget | 8 MiB per scope; 64 MiB committed per Worker; 128 scopes |
| Cache | 4,096 keys; key 1–1,024 bytes; value ≤64 KiB; TTL 1–3,600 seconds, default 100 |
| Context | 256 groups; before/after each 0–128, not both zero; 1–8 keys, key ≤1 KiB and value ≤4 KiB |
| Aggregation | 16 buckets; 4,096 groups per bucket, 64 fields per group; 1–8 grouping keys; key/output field ≤256 bytes, group value ≤4 KiB |
| Triggers | 1 second–1 hour; 1–1,000,000 records; bucket name ≤256 bytes |
| Fixed Tags | 32 entries; key ≤256 bytes, value ≤4 KiB |

Budgets conservatively account for strings, buffered Points and index overhead; they are not RSS limits. Transient memory also includes candidate state for up to eight parallel batches, decoded input and output. Invalid arguments, missing group keys, wrong types or capacity limits fail explicitly and follow the normal Pipeline policy of releasing failed input.

State changes commit only when results for the whole batch, including all CDC chunks, enter the queue successfully. Failed batches retain no partial changes. Failed timer enqueue retains the window and reports a Worker error until retry succeeds. Queued output uses ordinary durable delivery retries, survives restart and is not aggregated again. In-memory cache, context and aggregates are excluded from backups and lost on restart. Accepted but unprocessed inputs resume with fresh state, so results can differ from an uninterrupted run.

Worker drain and flow deletion wait for in-flight processing and final aggregate delivery. If the final window cannot be queued, an empty durable queue does not permit drain confirmation or deletion; the window is retained for retry. Process exit still clears in-memory state and does not replace online draining before shutdown.

Each admitted SQL CDC input is decoded in bounded chunks and atomically replaced only after that input succeeds. A PostgreSQL/MySQL transaction above 32 MiB consists of multiple admitted inputs; their Pipeline commits are independent and their shared transfer receipt waits for every input.

State scopes for different flows, published revisions or categories can proceed while another scope commits to disk. Each scope retains ordered processing and publishes candidate state only after queue commit succeeds. Concurrent growth reserves the shared Worker budget first and releases the reservation on failure. Limits remain 8 MiB per scope and 64 MiB per Worker; state is volatile and requires explicit online draining before shutdown.


## Admission, retries and resource boundaries

Each context-window group expires after one hour without input, independently of activity in other groups. Idle keep_value groups are reclaimed after one hour; pending values first emit through normal durable output, including a partial count-only bucket below its count threshold. The next input starts a new total: keep_value is not permanent storage. The existing limits of 256 active windows, 4096 aggregation groups per bucket and memory budgets remain. Only modified aggregation groups are copied; cancellation or persistence failure does not publish candidate state. Compiled scripts are evicted individually.


Pipeline debug, connection diagnostics, save and publish use the current source configuration, including an open advanced JSON edit. Invalid advanced JSON blocks debug instead of falling back to the previous configuration.


Ordinary Pipeline output and timer/final aggregation windows are prepared independently within each destination’s byte and batch quotas. An oversized branch or trimmed output prefix cannot fail another channel that has room. A single oversized branch does not clear existing backlog. Retained outputs, capacity-discard counters, input completion and cursor receipts commit in one WAL transaction. In-flight data remains protected. A quota change during preparation requires a fresh plan; a failed write retains the input or window without advancing counters or cursors.

Legacy inputs without installed capacity reservations still use the legacy total output-preparation budget. Exceeding it terminates the input with `pipeline/output_limit`, durably records a processing failure and completes its receipt in order; it does not retry indefinitely as a storage failure. With installed isolation, oversized branches continue to be discarded only in their own channel. A quota change during preparation retains the input or window for fresh preparation and logs at debug level. Actual storage failures still retain data and report an error.

For a channel that is not draining after a quota reduction, trimming an output prefix also evicts older, non-in-flight backlog in FIFO order. Timer/final windows follow existing backlog, so all earlier evictable entries in that channel are terminated, even if the retained window suffix and some old entries could still fit. This preserves the newest FIFO suffix. A single oversized branch does not advance this boundary or clear old backlog. In-flight branches remain protected; when they occupy the quota, only new branches in that channel are discarded, without blocking other destinations that have room.

Pipeline preview and publish errors retain line and column. Recognized unknown functions, invalid argument counts and incompatible types also return a safe reason (`pipeline_compile_error_reason` with nested cause codes). Other compilation errors retain `pipeline_compile_error`; those without a position use `pipeline_compile_failed`. Execution failures use `pipeline_runtime_error`, or `pipeline_runtime_failed` without a position. Positions are in `params[0]` and `params[1]`, and reasons in `causes`. The workbench translates each level; plain-text clients receive safe explanations too. Script literals, input values and raw third-party errors are not exposed.
