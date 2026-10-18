# FlowPort

[简体中文](README.md) · [Website](https://flowport.pages.dev/en/) · [Downloads](https://flowport.pages.dev/en/downloads/) · [Documentation](https://flowport.pages.dev/en/docs/) · [Changelog](https://flowport.pages.dev/en/changelog/)

FlowPort collects, processes and routes logs, metrics, traces and events through **Source → Pipeline → Destinations**.

**This repository contains product documentation and website links only. Product source, website source, source history and binaries are not published here.** Linux packages, checksums and release manifests are served from the official Cloudflare website. Public documentation does not make the product open source; see the website for product editions and licensing.

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

Documentation baseline: **0.1.24**. See the [release manifest](https://flowport.pages.dev/en/downloads/manifest/) and [upgrade notes](https://flowport.pages.dev/en/changelog/) before upgrading.

## Documentation and feedback

The [documentation index](docs/index.md) links to all five reading editions. Chinese repository guides cover [getting started](docs/getting-started.md), [delivery](docs/delivery.md) and [operations](docs/operations.md); the website provides full English guides.

Use [Issues](https://github.com/SamuelSupe/flowport/issues) for public product questions or documentation feedback. Include version, operating system and a redacted reproduction. Do not upload credentials, private keys, business payloads or sensitive data directories.
