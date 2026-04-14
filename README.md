# grafana-lgtm-dashboards

A portable collection of Grafana dashboards, provisioning configs, and documentation for the LGTM observability stack (Loki · Grafana · Tempo · Mimir) with an OpenTelemetry Collector.

Designed to be used as a **git submodule** inside any docker-compose LGTM project.

---

## Dashboards

| Dashboard | Description |
|-----------|-------------|
| [Service Overview](docs/service-overview.md) | RED metrics (Rate, Errors, Duration) per service |
| [Traces Explorer](docs/traces-explorer.md) | Span metrics + native Tempo trace search |
| [Logs Explorer](docs/logs-explorer.md) | Loki log browsing with log volume chart |
| [SLO Dashboard](docs/slo-dashboard.md) | Availability, error budget, burn rate |
| [Alerting Overview](docs/alerting-overview.md) | Firing/pending alerts, severity breakdown |
| [Infra Full Observability](docs/infra-full-observability.md) | CPU, memory, network, I/O per container |
| [Infrastructure](docs/infrastructure.md) | Docker host metrics via OTel docker_stats receiver |
| [OTel Collector Health](docs/otel-collector-health.md) | Collector pipeline throughput and queue depth |

---

## Repository Structure

```
grafana-lgtm-dashboards/
├── dashboards/            # Grafana dashboard JSON files
├── provisioning/
│   ├── dashboards/        # dashboards.yaml provider config
│   ├── datasources/       # datasources.yaml (Mimir, Loki, Tempo)
│   └── alerting/          # rules.yaml alert rules
└── docs/                  # Per-dashboard markdown documentation
    └── images/            # Dashboard screenshots (WebP)
```

---

## Usage as a Git Submodule

### 1. Add the submodule

```bash
git submodule add https://github.com/monte97/grafana-lgtm-dashboards.git grafana-dashboards
git submodule update --init
```

### 2. Mount into Grafana in `docker-compose.yaml`

```yaml
services:
  grafana:
    volumes:
      - ./grafana-dashboards/dashboards:/var/lib/grafana/dashboards
      - ./grafana-dashboards/provisioning/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana-dashboards/provisioning/datasources:/etc/grafana/provisioning/datasources
      - ./grafana-dashboards/provisioning/alerting:/etc/grafana/provisioning/alerting
```

### 3. Update the submodule later

```bash
git submodule update --remote grafana-dashboards
git add grafana-dashboards
git commit -m "chore: bump grafana-dashboards submodule"
```

---

## Datasource UIDs

The dashboards and alert rules reference these fixed datasource UIDs. Keep them in sync with `provisioning/datasources/datasources.yaml`:

| UID | Type | Default URL |
|-----|------|-------------|
| `mimir` | Prometheus (Mimir) | `http://mimir:9009/prometheus` |
| `loki` | Loki | `http://loki:3100` |
| `tempo` | Tempo | `http://tempo:3200` |

---

## Alerts

Five alert rules are provisioned in `provisioning/alerting/rules.yaml`:

| Alert | Condition | Severity |
|-------|-----------|----------|
| High Error Rate | Error rate > 2% for 5 min | critical |
| High Latency (p99) | p99 > 2s for 5 min | warning |
| Service Down | No spans for 5 min | critical |
| Collector Dropping Spans | `otelcol_processor_dropped_spans_total` > 0 for 2 min | warning |
| Collector Queue High | Queue > 80% capacity for 2 min | warning |

---

## Prerequisites

- **Tempo** must have `metrics-generator` enabled with `span_metrics` and `service_graphs` processors. Generated metrics are remote-written to Mimir.
- **Loki** must receive logs with the `service_name` label (standard when using the OTel Collector with the `otlphttp` exporter).
- **OTel Collector** must expose its own metrics on port `8888` and scrape itself via a `prometheus/self` receiver.
