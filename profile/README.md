# ByteFreezer

High-performance data ingestion and analytics platform for log and telemetry data.

## What is ByteFreezer?

ByteFreezer collects, processes, and stores machine data at scale. It ingests logs and telemetry from any source, transforms them through configurable pipelines, and stores them in an optimized columnar format for fast querying.

## Architecture

```
Data Sources                    ByteFreezer Platform                         Storage & Query
─────────────                   ────────────────────                         ───────────────

  Syslog      ┐                 ┌─────────┐
  Filebeat    ├──── UDP ───────►│  Proxy  │────┐
  Agents      ┘                 └─────────┘    │
                                               ▼
  Webhooks    ─── HTTP POST ──►┌──────────┐   S3      ┌───────┐    Parquet
  APIs                         │ Receiver │──────────►│ Piper │───────────►┌────────┐
  Custom      ─────────────────►└──────────┘          └───────┘            │ Packer │
                                                          │                └────────┘
                                                          ▼                     │
                                               ┌─────────────────────┐          │
                                               │  Transformations    │          ▼
                                               │  - Field extraction │    ┌───────────┐
                                               │  - GeoIP enrichment │    │  Query    │
                                               │  - Custom parsing   │    │  Engine   │
                                               └─────────────────────┘    └───────────┘
```

## Components

### Open Source (This Repository)

| Component | Description |
|-----------|-------------|
| **Proxy** | Edge collector. Receives data via UDP/syslog, batches, compresses, and forwards to Receiver. |
| **Receiver** | Ingestion endpoint. Accepts HTTP webhooks, validates data, stores raw events to S3. |
| **Piper** | Processing engine. Applies transformations, parsing, and enrichment to raw data. |
| **Packer** | Storage optimizer. Compacts processed data into Parquet files for efficient querying. |
| **Query** | Analytics engine. Executes SQL queries against Parquet data via DuckDB. |

### Subscription Service

| Component | Description |
|-----------|-------------|
| **Control** | Management plane. Configuration, coordination, health monitoring, multi-tenant API. |
| **UI** | Web interface. Dashboards, query builder, data exploration, and administration. |

## Key Features

- **High Throughput** - Handle millions of events per second with horizontal scaling
- **Flexible Ingestion** - UDP, syslog, HTTP webhooks, and custom protocols
- **Configurable Pipelines** - Transform, parse, and enrich data with tenant-specific rules
- **Columnar Storage** - Parquet format with Snappy compression for 10-50x size reduction
- **SQL Queries** - Standard SQL interface powered by DuckDB
- **Multi-Tenant** - Isolated data processing with per-tenant configuration
- **S3-Native** - Works with any S3-compatible storage (AWS S3, MinIO, etc.)

## Deployment Options

### Managed (Subscription)
Fully hosted solution. Data processing infrastructure managed by ByteFreezer with access to the Control plane and web UI. Contact [sales@bytefreezer.com](mailto:sales@bytefreezer.com) for pricing.

### On-Premises
Deploy data processing components in your own infrastructure using Helm charts. Components connect to the ByteFreezer Control service for configuration, coordination, and monitoring.

**Requires a ByteFreezer subscription** for Control service access.

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
