# Lookup enrichment

[FlowPort 0.2.0](../../README.md) · [Usage guides](https://flowport.pages.dev/en/docs/pipeline/)

Lookup resources provide managed, synchronized, versioned local enrichment. The manager fetches a complete snapshot; Workers download validated immutable files without source credentials. Event processing never calls external services. No extra database or scheduler is required.

These screenshots were captured on 2026-10-20 from the 0.2.0 release candidate in English, using an isolated instance and synthetic data. The captured interface is included in 0.2.0; capture build provenance is retained in the screenshot manifest. Addresses and counters are illustrative; select an image to view it full size.

## Configure and use

Create a resource on the Lookup page. Upload UTF-8 CSV, CSV.GZ or your own MMDB, or configure HTTP(S), a specific S3-compatible Bucket/Key, PostgreSQL or MySQL. CSV headers must be nonempty and unique. Values are strings. The page shows test queries, sync status, versions, flow references and each Worker's installed version.

Add a binding in the Pipeline panel. Select automatic updates or a fixed version, explicit Field/Tag input keys, lookup columns and output mappings. Insert the function at the desired script position. Its argument must be a string literal; the function applies the same binding and returns whether it matched. Save and publish the flow normally.

{{code:0}}

The sample workbench's Lookup view shows actual versions, bounded matched rows and diagnostics. Lookup does not discard events. Use existing Pipeline conditions and Tag routing for filtering and routing.

Search and select resources on the left. Resource configuration shows only the selected source fields, with authentication, TLS and advanced options collapsed. Test lookup lets you choose columns and enter values, then shows result rows and the actual version in a table. Versions & status collects history, Worker installations and referencing flows. Choose a common synchronization interval or enter a custom one. Unsaved changes are protected when leaving; save before uploading, synchronizing or rolling back.

The save bar stays visible while editing long forms. A new resource highlights upload or synchronization as its next step. Successful uploads and the first manual sync open Test lookup automatically. Asterisks in a saved connection address preserve its original value. Authentication headers accept an empty object or a JSON object whose values are strings.

![The Lookup resource page groups source settings, the current version and synchronization status. Advanced options start collapsed.](https://flowport.pages.dev/assets/screenshots/lookup-resource.webp)

*The Lookup resource page groups source settings, the current version and synchronization status. Advanced options start collapsed.*

## Complete example: enrich service ownership

This example uses a log's `service` field to add its team, business tier and cost center. Follow the complete path: create a resource → upload and test the data → add a Pipeline binding → call it from the script → debug samples → publish and update. All language editions share the same samples. Keep the Chinese sample values unchanged when copying them so the expected results match.

### 1. Prepare the CSV

Save the following as UTF-8 `services.csv`. The first row contains column names. `service` is the only lookup key in this example, and each value must be unique.

{{code:1}}

### 2. Create and test the resource

Open Lookup → Add resource, enter the following settings, then click Save configuration.

| Setting | Value |
| --- | --- |
| Name | `服务归属表` |
| Data source | File upload / API |
| File format | CSV |

Click Upload complete snapshot and select `services.csv`. A successful upload opens Test lookup. Choose Exact match, set the lookup column to `service` and the input value to `payment`. The query should match this row:

{{code:2}}

Testing on the resource page only verifies the lookup data. The resource is not yet connected to a dataflow.

![The test matches service=checkout exactly and returns team, business level and cost center with the actual data version.](https://flowport.pages.dev/assets/screenshots/lookup-query.webp)

*The test matches service=checkout exactly and returns team, business level and cost center with the actual data version.*

### 3. Bind the resource in Pipeline

Open the intended dataflow's Pipeline step, select the log category `logging`, and expand Lookup · Data enrichment. Enter `service_info` as the binding name, click Add binding, and configure it as follows.

| Setting | Value |
| --- | --- |
| Lookup resource | `服务归属表` |
| Data version | Automatic updates |
| Match mode | Exact match |
| Ignore case | Unchecked |
| Write policy | Fill missing values only |

Add one input key to read the event's ordinary `service` field and match the CSV's `service` column.

| Input type | Input name | Lookup column |
| --- | --- | --- |
| Field | `service` | `service` |

If `service` is actually a Tag in your input, select Tag instead. Ordinary top-level JSON fields do not automatically become Tags.

Add three output mappings. Enable Default value for each and enter the values below.

| Output column | Output type | Output name | Default value |
| --- | --- | --- | --- |
| `team` | Tag | `team` | `未归属` |
| `level` | Tag | `level` | `普通` |
| `cost_center` | Field | `cost_center` | `UNKNOWN` |

Write `team` and `level` as Tags for later routing, and `cost_center` as an ordinary field.

![The Pipeline binding reads the service field and explicitly maps lookup columns to the team, business_level and cost_center Tags.](https://flowport.pages.dev/assets/screenshots/lookup-binding.webp)

*The Pipeline binding reads the service field and explicitly maps lookup columns to the team, business_level and cost_center Tags.*

### 4. Insert the script call

If you only need enrichment, the complete script is this single line. Insert into script can also generate it.

{{code:3}}

To also label whether the query matched, replace that line with the following script. Choose one script; do not leave duplicate calls.

{{code:4}}

`lookup()` applies the output mappings and returns whether it matched. Filling defaults after a miss still returns `false`. The argument `"service_info"` is the Pipeline binding name, not the resource's display name `服务归属表`, and must be a string literal.

If earlier processing parses the log or extracts `service`, place the call after that extraction.

### 5. Run samples and check the results

Open Sample debugging, use the conditional script above, paste the following input JSON, and click Run samples.

{{code:5}}

All three events remain, their original fields are unchanged, and the following values are added.

| Input service | Tag: team | Tag: level | Field: cost_center | Tag: lookup_result |
| --- | --- | --- | --- | --- |
| `payment` | `交易组` | `核心` | `FIN` | `matched` |
| `search` | `搜索组` | `普通` | `SEARCH` | `matched` |
| `unknown_service` | `未归属` | `普通` | `UNKNOWN` | `unmatched` |

Check fields and Tags in After processing. Switch to Lookup to see the actual data version, matched rows and diagnostics. A miss does not filter an event, and defaults do not turn a miss into a match.

Fill missing values only preserves any existing field or Tag with the same name, including an empty string. Choose Overwrite existing values if the lookup table should replace existing values.

![The sample workbench displays the Lookup data version and matched rows used by this run. Preview does not deliver business data.](https://flowport.pages.dev/assets/screenshots/lookup-preview.webp)

*The sample workbench displays the Lookup data version and matched rows used by this run. Preview does not deliver business data.*

### 6. Route or filter by the result

To send core business events to a separate destination, enable Route by Tag in the dataflow's destination step and configure the following rule.

| Condition | Destination |
| --- | --- |
| Tag `level` equals `核心` | Your configured core-business destination |
| No rule matches | Your configured default destination |

Routing reads Tags after Pipeline processing. With multiple rules, the first match wins; unmatched events use the default destinations. Check the selected destinations in the sample workbench's routing preview. See [Combined Tag routing](https://flowport.pages.dev/en/docs/pipeline/).

If your business explicitly requires keeping only services present in the table, replace the previous script with the following. It actively drops unmatched events and is unnecessary for ordinary enrichment.

{{code:6}}

### 7. Publish and update the lookup data

After checking the samples, confirm that the source and destinations are configured. Click Save draft → Publish dataflow and confirm Worker installation in the run view. Sample debugging does not send data; actual delivery still requires checking destination receipts.

For example, change the team for `payment` in the CSV to `交易平台组` and upload the complete CSV again:

- With Automatic updates selected for the binding, newly admitted batches use the new version after each Worker installs it, without republishing the flow. Workers may briefly use different versions.
- Already queued batches retain the version pinned when they were received, so their results do not change with table updates.
- A fixed-version binding continues using its selected version. Change the version selection and republish to switch it.

Automatic updates controls which data version the binding follows; the resource's Automatic sync controls when data is fetched from an external source. File upload / API does not watch changes to `services.csv` on your computer: upload it again. For scheduled refreshes, change the resource source to HTTP(S), S3, PostgreSQL or MySQL, retain the column schema, and enable Automatic sync. You can keep the Pipeline binding and script. Failed synchronization preserves the last valid version.

## Match modes and output

Exact supports up to four composite keys, case-sensitive by default, with optional case folding. Duplicate query keys invalidate the index. Integers keep their decimal text; floating-point input is not implicitly converted to a key. Use existing conversion functions explicitly.

CIDR supports IPv4/IPv6, default longest prefix, first match or all matches. Equal prefixes use table order. Regex uses precompiled Go RE2 and table order, with first or all matches. Lookaround, backreferences and a “most specific regex” mode are unsupported. MMDB uses the database's longest-prefix IP match. Presets cover country, region, city, ASN and organization; custom paths such as `subdivisions.0.names.en` are supported. Users supply the database.

Exact example:

{{code:7}}

CIDR example (IP input maps to `network`):

{{code:8}}

Regex example (string input maps to `pattern`):

{{code:9}}

MMDB binding:

{{code:10}}

By default only absent values are filled; an empty string already exists. Explicit overwrite and per-output defaults are available. Unmatched input remains unchanged unless defaults are configured. All matches return at most 64 rows in table order. Each output column becomes a **JSON array string**, only in Fields, never Tags; Point types are unchanged. Overflow writes nothing, applies no defaults, returns false and records `match_limit`. Oversized regex input records `regex_input_limit`.

## Complete snapshots and freshness

Default synchronization is every 15 minutes, configurable from 60 to 86400 seconds. Sync now is available. One job per resource, at most two manager jobs, 60 seconds total per job; failures retry next interval. HTTP supports authentication headers, TLS, ETag/Last-Modified and never follows redirects. S3 reads one object, optionally an object version, without scanning the bucket.

SQL uses a read-only account and read-only Repeatable Read transaction for one SELECT without semicolons. No paging, CDC or stitching queries. Integers and exact numeric values retain textual precision. SQL CIDR/Regex tables require a unique, non-null integer order column; sorting happens after the complete read. Cancellation, partial reads or limits invalidate the whole candidate. Source permissions must constrain database functions as well.

The sequence is complete fetch → schema and running-binding compatibility validation → index construction → persistence → distribution → Worker installation. Unchanged content creates no version but updates successful-check freshness. Empty snapshots are rejected unless explicitly allowed. Missing columns, duplicate Exact keys, invalid indexes and capacity errors preserve the last valid version. Two intervals without a successful check show a stale warning without stopping local queries.

## Batch consistency and recovery

Flow configuration version, Lookup data version and actual batch versions are distinct. Automatic updates do not republish flows. Each Worker switches after validation at the next batch admission transaction; brief version differences across Workers are expected. Producers cannot set these version references.

All records, CDC chunks and crash recovery use the batch's pinned versions. Already prepared outbound backlog resends the stored result. Fixed versions take effect through normal publication. Activating history pauses automatic synchronization; resume explicitly by enabling Automatic sync in Resource configuration and saving.

No valid initial version means publication/start is rejected. A missing or corrupt queued version pauses affected processing and requests recovery while retaining payloads. Resource unavailability is neither a miss nor an ordinary Pipeline failure. During manager loss, complete local versions remain usable; existing nonexclusive continuation and exclusive ownership leases still apply.

## Limits, permissions and upgrade

Initial guards: CSV/SQL 32 MiB, 100000 rows, 64 columns; Regex 1000 rules, 2 KiB per pattern and 64 KiB per input; MMDB 128 MiB; per-process candidate/loading/index budget 256 MiB; per-node files 1 GiB. These are guards, not capacity promises. Decompression, candidate construction, concurrent sync and referenced old versions count, so an update may be rejected below the file limit. Pipeline execution keeps its existing time budget.

Current, pinned, queued, in-flight and Worker-granted versions are protected. Keep five unreferenced successful historical versions. Never evict business queues to make Lookup space.

Metadata uses read permission; content, test queries and mutations require write. Credentials return as `********`; audits exclude full tables and secrets. Binary upload has separate streaming limits; ordinary management requests remain 2 MiB. MCP manages resources, synchronization, test queries and status; upload/download uses REST.

Encrypted resource files live separately under `native/lookups/`; configuration and queues store references. Control protocol is **v6**, checkpoint format **3**. Upgrade manager and all Workers together. Old binaries reject the new format. Stop nodes and back up every complete data directory with `master.key` and resource files. Downgrade requires a complete pre-upgrade backup, not only an older binary.


Lookup automatic synchronization is audited with actor `system`. Worker status includes the selected automatic version and installed versions for fixed bindings and queued data. Mapping serialization is bounded by the existing 32 MiB batch output limit; large mappings may fail before allocating output.


Point keys are unique: output names cannot repeat across Fields and Tags. Fill-missing preserves an existing key of either kind; overwrite writes the explicit target kind. MySQL consistent snapshots require transactional tables such as InnoDB.
