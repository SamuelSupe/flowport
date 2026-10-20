# FlowPort

[English](README.md) · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md) · [日本語](README.ja.md) · [한국어](README.ko.md)

[Website](https://flowport.pages.dev/en/) · [Downloads](https://flowport.pages.dev/en/downloads/) · [Documentation](https://flowport.pages.dev/en/docs/) · [Changelog](https://flowport.pages.dev/en/changelog/)

FlowPort collects, processes and routes logs, metrics, traces and events through **Source → Pipeline → Destinations**.

**This repository contains product documentation and website links only. Product source, website source, source history and binaries are not published here.** Linux packages, checksums and release manifests are served from the official Cloudflare website. Public documentation does not make the product open source; see the website for product editions and licensing.

## 0.2.1

- Fix Kafka delivery expiry incorrectly measured from event time against a fixed 20-second limit. Historical logs, metrics, traces and other Point categories no longer fail before transmission because of their age. Preserve both the Kafka message timestamp and original event timestamps in encoded payloads. This covers all Kafka outputs and service presets using the Kafka protocol.

- Each Kafka send attempt uses config.timeout_seconds, default 20 seconds and range 1–120 seconds, while honoring an earlier parent cancellation or deadline. A retry starts a new send deadline. Failed delivery retains the original queue and processed output; recovery does not rewrite event times or clear backlog.

- 0.2.1 retains control protocol v6 and checkpoint format 3. Stop services and back up complete data directories, master.key, Lookup resources and environment configuration before upgrading the manager and all Workers. Retained Kafka backlog retries automatically from the existing data directory; FIFO-evicted data cannot be recovered. Broker and cloud-service timestamp limits and retention policies still apply.

## What is new in 0.2.0

- Lookup supports composite Exact keys, IPv4/IPv6 CIDR, RE2 Regex and MMDB. Upload CSV, CSV.GZ or MMDB, or synchronize complete snapshots from HTTP(S), S3, PostgreSQL and MySQL in the background. Pipeline calls lookup("binding_name") locally and maps results to Fields or Tags.
- Lookup includes automatic updates, pinned versions, test queries, history rollback and Worker installation status. Each batch pins its data versions at admission. Failed synchronization keeps the last valid version; missing queued resources pause processing without releasing payloads. Rollback pauses automatic sync until explicitly resumed.
- Enable cache_get/cache_set, point_window/window_hit and agg_create/agg_metric for short-term deduplication, error context and count/time window aggregation. Bounded memory is isolated by Worker, flow publication and input category, and clears on restart. Sample debugging does not modify runtime state.
- Five scenario templates and single-flow import/export are available (flowport.flow v1, maximum 2 MiB). Map local destinations and Lookup resources, review, save and explicitly publish. Packages exclude queues, cursors, Lookup data and memory state. Connection credentials are removed, but script/SQL literals still require review. Improve form layout, asynchronous cancellation, resource reuse and export with multiple bindings.

## Upgrade to 0.2.0

Control protocol advances to v6 and checkpoints to format 3. Stop services and back up every node’s complete data directory, master.key, Lookup resources and environment before upgrading the manager and all Workers together. Downgrade requires a complete pre-upgrade backup. Linux ARM64/x86_64 tar.gz, deb and rpm packages need no Node.js for installation or runtime.

A queued batch that lacks its pinned Lookup resource pauses until the resource is recovered; it is not discarded as an ordinary Pipeline failure. In-memory cache, context and aggregation state clear on restart. Already queued output retains its original result.

## Usage guides

- [Lookup enrichment](docs/en/lookup.md)
- [Cache and windows](docs/en/pipeline.md)
- [Templates and import/export](docs/en/flows.md)

## Get started

1. [Download and install a Linux package](https://flowport.pages.dev/en/downloads/). Both amd64 and arm64 are available as deb, rpm and tar.gz. Running FlowPort requires no Node.js, npm or Go compiler.
2. Follow the [quick start](https://flowport.pages.dev/en/docs/), configure a Destination, create a dataflow and publish it.
3. Confirm the actual downstream receipt. An intake HTTP 204 confirms local durable acceptance only.

## Operating model

- The catalog includes 72 Source types and 48 Destination types, with tag-based routing after processing. Consult the relevant integration guide and validate your actual service/version/permission combination.
- The community download runs a single Worker. Commercial multi-Worker deployments require a publisher-supplied commercial build and a matching license.
- Workers own independent local persistent queues; business queues are not replicated across nodes.
- A failing target keeps its processed output in the local queue and retries with backoff. Intake continues. Capacity limits evict normal unfinished batches by original receive order (FIFO).
- Healthy target branches proceed independently. Parsing or Pipeline failures release the accepted input. Legacy isolated failed payloads require explicit export or cleanup.
- FIFO eviction is allowed data loss. A crash after remote receipt but before local acknowledgement can cause duplicates. There is no strict exactly-once or global ordering guarantee.

Documentation baseline: **0.2.1**. See the [release manifest](https://flowport.pages.dev/en/downloads/manifest/) and [upgrade notes](https://flowport.pages.dev/en/changelog/) before upgrading.

New 0.2.0 installations use the [Web setup wizard](https://flowport.pages.dev/en/docs/#web-setup) to set the administrator password and management/HTTP ingestion domain or IP and port. Read the local `manager/setup-token` after startup. Existing instances retain their password and queues; offline recovery and verified backups are described in the [operations guide](https://flowport.pages.dev/en/docs/delivery/#offline-maintenance).

## Documentation and feedback

The [documentation index](docs/index.md) links to all five reading editions. Chinese repository guides cover [getting started](docs/getting-started.md), [delivery](docs/delivery.md) and [operations](docs/operations.md); the website provides full English guides.

Use [Issues](https://github.com/SamuelSupe/flowport/issues) for public product questions or documentation feedback. Include version, operating system and a redacted reproduction. Do not upload credentials, private keys, business payloads or sensitive data directories.
