# Dataflow configuration and publishing

[FlowPorts 0.5.16](../../README.md) · [Docs](../index.md) · [Website](https://flowport.pages.dev/en/docs/)

Unfinished drafts may omit sources, destinations or destination allocations. Explicit quotas must remain positive and within the total budget. Saving does not start intake or change the running version. Publication requires complete targets and matching quotas. Implicit allocations are materialized at publication; adding a destination never silently redistributes manual quotas.

After publishing, use Delivery verification in the runtime view. Wait for current-version installation, a ready source, fresh reports and an empty backlog. Start verification before sending real upstream data. Supported HTTP and DataKit sources offer a copyable example with a published-token placeholder. Compare new received batches, successfully processed batches and acknowledgments for every target. Send matching samples for each route and inspect downstream content. The guide does not send automatically or trace individual records. Stale or disconnected reports cannot confirm delivery; changes in version, Worker assignment, process instance or counters require a new verification. Refreshing or changing scope resets it. Protocol acknowledgment does not prove downstream business processing.

> After the initial send, each destination defaults to five fast retries. When those retries are exhausted, unconfirmed data stays queued and the channel waits at least 60 seconds before probing one batch. New flows default to 256 MiB and 500 internal batches per Worker, partitioned between input and destinations. A full output keeps its upstream input queued. A full input rejects new intake.

Existing TCP and UDP Socket listeners stay active when their Source settings are unchanged. After the Worker installs a Pipeline update, newly admitted records use the new version, even on an existing TCP connection. Queued batches keep their original version.

When the input queue is full, TCP Socket, Syslog and StatsD close the affected connection and count decoded records refused before durable admission. See the rejection reason in the runtime view, source_rejected_records in the API and collectway_flow_source_rejected_records_total in Prometheus. Accepted backlog stays queued.


Flow IDs are shown with a copy action at the top of the editor and runtime view. Signed-in administrators can explicitly show, hide and copy inbound Source tokens (including Vector). Draft and published credentials are separate; draft changes take effect only after publication and Worker installation. Tokens are shown only in the aggregate runtime view, not when viewing an individual Worker. Other upstream credentials remain masked. Ordinary flow reads and exports are sanitized; API Keys and MCP cannot reveal ingestion tokens.

Before the first publication, Overview offers Create your first flow or Continue flow setup. Templates open source configuration first and retain their scripts and samples. Choose Next: destinations to pass data through, or add optional processing rules. Add and select destination creates a target inside the flow and selects it without losing source settings or scripts.

Preview with the current flow uses its configured measurement (default: Source type) and the runtime source_type and collectway_flow tags. Samples must represent decoded source records; preview does not contact upstreams or deliver data.

The summary shows the latest published configuration; selecting a Worker shows its installed pipeline. Publication and installation are displayed separately, and backlog counts include historical versions.

The editor marks unsaved changes and shows progress while saving or publishing. Publishing opens the runtime view directly so you can check deployment status, installed Worker versions and actual delivery without reopening the flow from the list. The flow list uses Stop intake and Resume intake. Stopping intake only stops new input; backlog processing and delivery continue. Destination failures do not automatically stop intake.

The editor explains activation as Save draft → Publish flow → Worker installation. Saving only keeps your changes. After publishing, confirm the installed version in the runtime view, then confirm actual delivery. Saving a destination opens its referring flows and configuration status so you can inspect and publish each affected draft. Name-only destination changes do not require republishing. The final destination step also offers Save and publish, so you can submit after reviewing without returning to the top. Save or publish errors bring the specific message into view at the top.

A flow contains a Source, per-signal Pipelines, default Destinations, optional Tag routing, placement and queue capacity. Publishing creates an immutable version that Workers install asynchronously. Drafts are independent of the running version. Connection testing validates the current form before saving; select the actual Worker. Pipeline samples validate scripts, not the connection. See [Diagnostics](https://flowport.pages.dev/en/docs/#configure).

## Source selection and ingestion

Search Dataflows by name, flow ID, Source type ID or display name. Source selection is part of New dataflow and supports ingestion-mode filters and signal labels. Push sources need a token or listener; consumers need queues/topics and credentials; polling sources need a full remote URL.

Advanced JSON represents the entire Source configuration and must be a complete object. Container listeners usually bind to 0.0.0.0 with a mapped port. Saved sensitive values appear as ********; keeping the placeholder preserves the value for the same protocol. Enter a new credential to replace it, then publish. Changing protocol requires new connection configuration. See [Source boundaries](https://flowport.pages.dev/en/docs/integrations/).

## Processing rules

Choose logging, metrics, tracing or another supported signal. Scripts are separate per category; empty scripts pass through. DataKit supports all 15 Point categories. Sample debugging opens a separate workspace and does not send to a Destination. It shows decoded input, output, drops, typed fields, tags and raw JSON. Changing a sample, script or category marks the previous result stale. Closing returns to orchestration while preserving the sample and result. See [Pipeline](pipeline.md).

## Default destinations and combined routing

Select multiple default Destinations to copy each point to independent delivery branches. A flow may reference at most 32 distinct Destinations. A draft with no target can be saved but cannot be published.

Rules support eq, neq, exists, not_exists and nested AND/OR groups. The first match replaces the default target set; no match uses the defaults. Each rule binds its own one or more Destinations. Include an always-on archive in both defaults and every rule. Match final Tags, not ordinary fields; use set_tag first. Tag names/values are case-sensitive and are not trimmed. Destination signals filters do not continue to the next rule. See [Routing](https://flowport.pages.dev/en/docs/pipeline/).

Current failure handling and migration: [FlowPort](https://flowport.pages.dev/en/docs/delivery/).

## Placement and capacity

Automatic placement uses permitted Workers, or restrict the eligible nodes explicitly. Push/shared consumption can run on multiple nodes; polling and some subscriptions use an exclusive lease. Network access, TLS files and cloud identity must exist on the executing Worker.

Intake acknowledgment confirms local durable enqueue. Receiving and delivering are separate. After publishing, verify Worker installation and inspect a new record downstream.

## Drafts, publishing and versions

| Action | Result |
| --- | --- |
| Save draft | Preserve edits without changing runtime version or ingestion |
| Publish | Validate Source, Pipeline, targets and routing; create and dispatch a version |
| Stop intake | Stop new input; accepted data continues with its original processing result and target IDs |
| Resume intake | Resume the last published version; drafts cannot start |
| Save destination | Store parameters; after each referencing flow is republished and installed, new input and retained branches for the same target ID use the new connection |

Inspect desired/installed versions, Source state and errors while synchronizing. Publication alone does not prove ingestion. On revision conflict, reread and review before saving. Leaving unsaved edits prompts confirmation.

## Configuration from the runtime graph

Pencil icons open Source configuration, Pipeline/category, Tag routing or the corresponding Destination dialog. Opening does not save or publish. The graph represents the published version while editing loads the latest draft. Returning restores Worker selection, animation pause, scroll position and opener focus. Graph polling pauses during configuration without stopping collection; late responses cannot reopen an editor after leaving the flow. Script lists support keyboard scrolling and complete tooltips.

## Stopping and deletion

Intake acknowledgment confirms local durable enqueue. Receiving and delivering are separate. After publishing, verify Worker installation and inspect a new record downstream.

Once published, a dataflow ID remains a historical resource identity after deletion and cannot be reused. Create a new flow with a new ID to avoid inheriting Worker cursors, lease history or counters and skipping initial collection. Stopping and restarting the same flow still resumes its durable progress. A draft that was never published can reuse its ID after deletion.

Drafts have an independent `draft_revision`. Use 0 for creation; updates must return both the `revision` and `draft_revision` that were read. A successful save returns a new draft revision. Publication requires that saved `draft_revision` in the request body: missing values return400 and changed drafts return409, without publishing another editor's changes. Changing Source protocol requires explicit sensitive configuration. Publication history in the flow list supports pagination and configuration comparison. Restore a historical version as a draft, review it and publish normally. Restoring does not switch the running version; republication uses current destination addresses and credentials.

Removed Workers retain historical receive, delivery and target counters consistently in lists and run views, with offline samples. An unknown queue on a forcibly removed node still blocks flow deletion. If that node is permanently lost, stop the flow, remove the node from upstream systems/load balancers, then use Resolve lost node backlog in the flow list. Select the removed node and enter the full flow ID. `POST /flows/{id}/abandon-worker` takes `worker` and a `confirm` equal to the flow ID. It waives only that node's deletion block for this flow, leaves disk and last report intact, and does not bypass other nodes' drain checks. The decision is durable and audited as forced; query it with `GET /flows/{id}/recovery`.

Current failure handling and migration: [FlowPort](https://flowport.pages.dev/en/docs/delivery/).


From 0.1.19, a send that began before its branch was evicted by FIFO still records its eventual failure or confirmed success and updates destination health durably. It never restores the payload, reverses eviction counters or completes the receipt twice. The same branch can count as both FIFO-evicted and confirmed delivered; these counters cannot be subtracted to derive unique inputs. A late success on the effective connection can clear that target alert. Obsolete publication results cannot change current target health, and a late failure cannot recreate an alert for a removed target without backlog.


## 0.1.21 · Historical behavior from earlier releases

Republish referring flows and wait for Worker installation. Backlog for the same destination ID retains its processed result and uses the new connection on subsequent retries. Old in-flight requests cannot be recalled; recovery may produce duplicates.

Compares saved destination settings; unsaved edits are excluded. Renaming does not require publication. Workers with the current connection installed use it on subsequent retries; old in-flight requests cannot be recalled.

Recent evictions cover the current and previous four minute buckets and survive restarts. Queue times use original receipt times. Offline nodes show the last sample, which cannot predict current retention.

Evictions count local inputs or destination branches and cannot be converted into a unique-input loss rate. Destination copies and batch limits shorten retention. Logical quota does not reserve physical disk space.

Previews the current draft and saved destination signal filters for up to 100 output points. Does not connect or verify write permissions. Unmatched points use defaults; signal filtering does not fall through to another rule.


## 0.1.24 resource and history limits

History keeps the latest 100 versions per flow within a shared 64 MiB encrypted-snapshot budget. Oldest unreferenced history is collected. Active, normal/legacy queue, in-flight delivery and manager-approved/Worker-reported versions remain protected and can exceed the history count. New publication fails if protected snapshots fill the budget; existing intake/queues continue, and pre-upgrade excess references remain. Deleting a drained flow releases full snapshots while retaining its reserved identity. Previously assigned remote Workers collect retired configurations only after both local normal and legacy queues drain; offline Workers apply the notice after reconnecting. History/restore cover retained versions only. Checkpoints and normal log reclamation release physical space later. Legacy failed payloads are never silently deleted.

The current release uses control protocol v8 and checkpoint format 3, and reads formats 1/2. Stop services and back up every node’s complete data directory, master.key, Lookup resources and environment before coordinating the manager and all Worker upgrades. Downgrade requires the complete pre-upgrade backup and reconciliation of newly accepted data; never open or overwrite an upgraded directory with an old binary.

## Scenario templates and single-flow import/export

Choose Start from scratch, Scenario templates or Import configuration from New flow. Templates cover HTTP log forwarding, log-level routing, 60-second deduplication, error context and window metrics. Each supplies an HTTP JSON source, a script and sample data; the source can be changed. Enter credentials, select destinations and Workers, test under Pipeline → Sample debugging, then save and publish. Metric templates require a destination that supports metric. Cache and window state uses bounded memory isolated by Worker, publication and input type, and clears on restart.

From the flow list, choose Export configuration and select Saved draft or Latest published configuration to download JSON. Unsaved edits are excluded. The file uses `format: "flowport.flow"`, `version: 1` and includes source settings, per-type Pipelines, default and routing destination references, Lookup bindings, queue capacity and ACK policy. It excludes shared destination connections, Lookup contents and synchronization credentials, Worker placement, queues, collection cursors, publication history and memory state. It is reusable configuration, not a complete backup.

Sensitive connection fields (tokens, passwords, headers, URLs, connection strings and private keys) are removed recursively and listed in `removed_fields`. Literal values in scripts, SQL queries and custom configuration remain unchanged; inspect the file before sharing. Import removes sensitive fields again and never uses `********` to recover old credentials.

Import steps:

1. Create the required destinations and Lookup resources in the destination workspace. Upload or synchronize Lookup data separately.
2. Open New flow → Import configuration and select a JSON file up to 2 MiB. The system checks the format version, Source type and structure, then displays a summary and removed fields.
3. Explicitly map every default/routing destination and Lookup reference to local resources before continuing. Matching names or IDs do not bind automatically; shared resources are never created or overwritten.
4. Review the name, connection, authentication, Workers, queue capacity and rules. Lookup automatic-update policies and pinned versions are retained. A pinned version must exist in the selected resource and be compatible; inspect or change it in the Pipeline Lookup panel.
5. Test samples, save a new draft, then publish explicitly. Selecting a file or opening the editor does not save, start intake or modify the original flow. The new flow receives a new ID with reset publication and runtime state. Collection progress is not inherited; check starting positions for consuming sources.

REST: `GET /api/v1/flows/{id}/export?configuration=draft|published` returns a sanitized package (draft by default). `POST /api/v1/flows/import-preview` accepts the whole package and returns normalized configuration with its complete dependency list without writing storage. Map dependencies, assign a new Flow ID, then use the existing draft PUT and publish POST endpoints. Existing authentication and the ordinary 2 MiB management request limit apply. Export requires read; import preview requires write. Saving and publishing retain existing audit behavior. MCP tools are `exportFlow` and `previewFlowImport`. Scenario templates are available in the UI; see [Pipeline usage](pipeline.md) for script semantics.


Import shows the number of mapped resources and keeps the action buttons visible while the content scrolls. You can cancel a file check or choose another file; an old response will not replace the new selection. Lookup lists are reused only while this dialog stays open. If loading fails or resources change, select Refresh resources without uploading the file again; the file summary and destination selections remain available. Closing the export dialog also cancels an unfinished download request.

Published history compares the draft and selected version, including sources, Pipelines, Lookup bindings, destinations, routing, queue capacity and Worker placement. Restoring replaces only the draft; review and publish it to change the running configuration. Check pinned Lookup data versions as part of the bindings. Signing in after session expiry does not reopen a previous creation dialog.


The editor keeps the ID, Token controls and main actions visible. Expand Save and publish details for the deployment steps. Saving keeps a draft; after publishing, check the installed Worker version and actual delivery.


Run sample scripts before completing targets or routing. Script results remain available; Route preview shows the routing validation error separately. Complete routing and run again. Publishing still validates the full configuration.


Pipeline debug, connection diagnostics, save and publish use the current source configuration, including an open advanced JSON edit. Invalid advanced JSON blocks debug instead of falling back to the previous configuration.


The publication success notice confirms that configuration was saved. Check configuration status in the runtime view to confirm Worker installation.

## Fixed capacity isolation

Set `queue_capacity.input` and `queue_capacity.outputs` within the flow budgets. A full output retains the entire input or window until all required output regions can accept it. Each Worker displays the installed quotas, usage and oldest backlog. Lowering quotas keeps existing backlog while draining. Configure reserved capacity explicitly when adding destinations.

[Capacity and delivery policy](https://flowport.pages.dev/en/docs/delivery/)
