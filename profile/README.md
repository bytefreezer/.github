# ByteFreezer

High-performance data ingestion and analytics platform for log and telemetry data.

## What is ByteFreezer?

ByteFreezer collects, processes, and stores machine data at scale. It ingests logs and telemetry from any source, transforms them through configurable pipelines, and stores them in an optimized columnar format for fast querying.

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

### Open Source (This Repository)

| Component | Description |
|-----------|-------------|
| **Proxy** | Edge collector. Receives data via UDP/syslog, batches, compresses, and forwards to Receiver. |
| **Receiver** | Ingestion endpoint. Accepts HTTP webhooks, validates data, stores raw events to S3. |
| **Piper** | Processing engine. Applies transformations, parsing, and enrichment to raw data. |
| **Packer** | Storage optimizer. Compacts processed data into Parquet files for efficient querying. |
| **Query** | Reference analytics engine. Example implementation using DuckDB - customize to your needs. |

### Subscription Service

| Component | Description |
|-----------|-------------|
| **Control** | Management plane. Configuration, coordination, health monitoring, multi-tenant API. |
| **UI** | Web interface. Dashboards, query builder, data exploration, and administration. |

## Agentic Configuration

Setting up data pipelines is effortless with ByteFreezer's AI-powered configurators. Describe your data source in plain language - the agentic system analyzes sample data, detects formats, and automatically generates parsing rules, field mappings, and transformations. No regex expertise required. The configurator learns from your feedback and continuously improves extraction accuracy.

## Key Features

- **High Throughput** - Handle millions of events per second with horizontal scaling
- **Flexible Ingestion** - UDP, syslog, HTTP webhooks, and custom protocols
- **Configurable Pipelines** - Transform, parse, and enrich data with tenant-specific rules
- **Columnar Storage** - Parquet format with Snappy compression for 10-50x size reduction
- **Multi-Tenant** - Isolated data processing with per-tenant configuration
- **S3-Native** - Works with any S3-compatible storage (AWS S3, MinIO, etc.)

## Data Output

ByteFreezer outputs data as **Parquet files** - an open columnar format supported by most analytics tools. Use the data with systems you already have:

- **Data Warehouses** - Snowflake, BigQuery, Redshift, Databricks
- **Query Engines** - Trino, Presto, Apache Spark, DuckDB
- **BI Tools** - Tableau, Power BI, Grafana, Superset
- **Data Lakes** - Delta Lake, Apache Iceberg, Apache Hudi

The included Query component is a reference implementation. Customize it or integrate Parquet output directly into your existing analytics stack.

## Deployment Options

### Managed (Subscription)
Fully hosted solution. **ByteFreezer provides the compute infrastructure** - we run and scale all data processing components (proxy, receiver, piper, packer, query) on your behalf. Includes access to Control plane and web UI. Contact [sales@bytefreezer.com](mailto:sales@bytefreezer.com) for pricing.

### On-Premises (Subscription)
**You provide the compute infrastructure** - deploy data processing components in your own Kubernetes cluster or servers. Components connect to ByteFreezer Control service for configuration, coordination, and monitoring. Your data stays in your environment.

Both options require a ByteFreezer subscription for Control service access.

```bash
helm install bytefreezer ./helm/bytefreezer \
  --set minio.enabled=true \
  --set controlService.url=https://api.bytefreezer.com \
  --set controlService.apiKey=YOUR_API_KEY
```

See [Helm Chart Documentation](./helm/bytefreezer/README.md) for full deployment guide.

## Repository Structure

```
bytefreezer/
├── proxy/          # Edge data collector
├── receiver/       # HTTP ingestion service
├── piper/          # Data processing pipeline
├── packer/         # Parquet file generator
├── query/          # SQL query engine
├── helm/           # Kubernetes deployment
└── docs/           # Documentation
```

## License

Elastic License 2.0 - See [LICENSE](LICENSE) for details.
