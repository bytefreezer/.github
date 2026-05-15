# ByteFreezer

Cut your SIEM bill 80-90%. Keep all your data.

ByteFreezer sits before your SIEM. Stores everything as Parquet on your infrastructure. Forward what you need, now or later.

## Quickstart

The fastest way to try ByteFreezer is with [Claude Code](https://docs.anthropic.com/en/docs/claude-code) and our MCP server — describe what you want in plain English and Claude provisions accounts, generates configs, deploys the stack, and verifies the pipeline end to end.

**Want to see it working without standing up infrastructure?** Use the **managed demo path** — you only deploy the proxy on a host that can receive your tapped traffic, we host receiver, piper, packer, control, storage, and the dashboard. Data shows up in the UI a few minutes after the proxy starts. See [Managed Quickstart](https://github.com/bytefreezer/installer/blob/main/docs/guide-managed.md).

**Prerequisites:** [install Claude Code](https://docs.anthropic.com/en/docs/claude-code/quickstart) (`npm install -g @anthropic-ai/claude-code` or follow the platform instructions), then:

1. Sign up at [bytefreezer.com](https://bytefreezer.com) and grab an [API key](https://bytefreezer.com/dashboard/api-keys).
2. Add the ByteFreezer MCP server to Claude Code. Run this in any terminal where the `claude` CLI is on your PATH — it edits your `~/.claude.json` so subsequent sessions can call our MCP tools. (See [Claude Code MCP docs](https://docs.anthropic.com/en/docs/claude-code/mcp) for background.)

   ```bash
   claude mcp add --transport http bytefreezer \
     https://mcp.bytefreezer.com/mcp \
     --header "Authorization: Bearer YOUR_BYTEFREEZER_API_KEY"
   ```

3. Restart your Claude Code session, then ask: *"please see if bytefreezer mcp is accessible"*.
4. Pick a guide from the [installer](https://github.com/bytefreezer/installer) — [Managed Quickstart](https://github.com/bytefreezer/installer/blob/main/docs/guide-managed.md) (proxy only), [Docker Compose](https://github.com/bytefreezer/installer/blob/main/docs/guide-onprem-docker.md), [Kubernetes](https://github.com/bytefreezer/installer/blob/main/docs/guide-onprem-k8s.md), or [HA Kubernetes](https://github.com/bytefreezer/installer/blob/main/docs/guide-onprem-k8s-ha.md) — and Claude takes it from there.

Prefer to drive manually? Every guide is a plain Markdown runbook with the same steps Claude follows.

## Architecture

```
Data Sources              ByteFreezer Platform                      Storage & Query
────────────              ────────────────────                      ───────────────

  Syslog     ┐            ┌─────────┐
  Filebeat   ├── UDP ────►│  Proxy  │───┐
  Agents     ┘            └─────────┘   │
                                        ▼
  Webhooks   ── HTTP ───────────────►┌──────────┐      ┌───────┐      ┌────────┐
  APIs                               │ Receiver │─────►│ Piper │─────►│ Packer │
                                     └──────────┘      └───────┘      └────────┘
                                           │               │               │
                                           ▼               ▼               ▼
                                     ┌─────────────────────────────────────────┐
                                     │                   S3                    │
                                     │  intake/    │    piper/    │   packer/  │
                                     │  (raw)      │  (processed) │  (parquet) │
                                     └─────────────────────────────────────────┘
                                                                         │
                                                                         ▼
                                                                   ┌───────────┐
                                                                   │   Query   │
                                                                   │  Engine   │
                                                                   └───────────┘
```

## Components

All components are open source under Apache 2.0.

### Data Plane

| Component | Description |
|-----------|-------------|
| **Proxy** | The ingest point. Sits where you tap or mirror your traffic — SPAN/TAP ports, syslog relays, agent forwarders, webhooks — and is the only component that *needs* to live near your data. Batches, compresses, and forwards to Receiver. Everything downstream runs wherever you want (your own infra, or our managed stack). |
| **Receiver** | Ingestion endpoint. Accepts HTTP webhooks, validates data, stores raw events to S3. |
| **Piper** | Processing engine. Applies transformations, parsing, and enrichment to raw data. |
| **Packer** | Storage optimizer. Compacts processed data into Parquet files for efficient querying. |
| **Query** | Reference analytics engine using DuckDB — customize to your needs. |

### Control Plane

| Component | Description |
|-----------|-------------|
| **Control** | Management plane. Configuration, coordination, health monitoring, multi-tenant API. |
| **UI** | Web interface. Dashboards, configuration, data exploration, and administration. |

## How It Works

1. Data flows toward your SIEM
2. ByteFreezer intercepts. Stores everything as Parquet.
3. Only what you choose gets forwarded to the SIEM.

Example filter chatty eBPF to IO operations. Send only password changes. Route only critical alerts. The rest stays in Parquet — queryable, compliant, nearly free.

## AI Configuration

Describe what you need in plain English. MCP gives AI full knowledge of your deployment. Pipeline configs built in minutes, not months.

## Data Output

ByteFreezer outputs data as **Parquet files** — an open columnar format supported by most analytics tools:

- **Query Engines** — DuckDB, Trino, Presto, Apache Spark
- **Data Warehouses** — Snowflake, BigQuery, Redshift, Databricks
- **BI Tools** — Tableau, Power BI, Grafana, Superset
- **Data Lakes** — Delta Lake, Apache Iceberg, Apache Hudi

## Pricing

| Tier | Cost | Details |
|------|------|---------|
| **Free** | $0 | 1 proxy, 1 tenant, 1 dataset. Full control plane, no expiration. |
| **Self-Service** | $400/proxy/mo | Unlimited proxies, tenants, datasets. Email support. |
| **White Glove** | Flat onboarding + retainer | Dedicated expert deployment and ongoing optimization. |

## Deployment

Your infrastructure. Your data never touches a third party.

Deploy the data plane on your servers, Kubernetes, Docker, or systemd. The control plane provides configuration and health monitoring.

```
bytefreezer/
├── proxy/          # Edge data collector
├── receiver/       # HTTP ingestion service
├── piper/          # Data processing pipeline
├── packer/         # Parquet file generator
├── query/          # SQL query engine
└── docs/           # Documentation
```

## Links

- **Website**: [bytefreezer.com](https://bytefreezer.com)
- **Documentation**: [docs.bytefreezer.com](https://docs.bytefreezer.com)
- **GitHub**: [github.com/bytefreezer](https://github.com/bytefreezer)
- **For CISOs**: [bytefreezer.com/ciso](https://bytefreezer.com/ciso)
- **For Operators**: [bytefreezer.com/operators](https://bytefreezer.com/operators)
- **Contact**: [bytefreezer.com/contact](https://bytefreezer.com/contact)

## License

All ByteFreezer components are licensed under Apache 2.0. See each component's `LICENSE` file for details.
