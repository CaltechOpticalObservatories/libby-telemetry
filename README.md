# libby-telemetry

OpenTelemetry backend for [libby](https://github.com/CaltechOpticalObservatories/libby)
peers: an OTel Collector receiving OTLP, with metrics in Prometheus, traces in
Tempo and logs in Loki, all read from Grafana.

| Service | Image | Port |
|---|---|---|
| Collector | `otel/opentelemetry-collector-contrib` | 4317 (gRPC), 4318 (HTTP) |
| Prometheus | `prom/prometheus` (3.13 LTS) | 9090 |
| Tempo | `grafana/tempo` | 3200 |
| Loki | `grafana/loki` | 3100 |
| Grafana (optional) | `grafana/grafana` | 3000 |

The Collector writes metrics to Prometheus's OTLP receiver, traces to Tempo
and logs to Loki's OTLP endpoint. Tempo derives service graph and span metrics
and remote-writes them to Prometheus.

## Run

```bash
docker compose up -d
```

Everything binds to `127.0.0.1` unless told otherwise. Site settings are given
at install time, in the environment or a `.env` file beside `compose.yaml`:

| Variable | Default | Purpose |
|---|---|---|
| `OTLP_BIND_ADDRESS` | `127.0.0.1` | Address the Collector's OTLP ports listen on; set it so other hosts can export |
| `QUERY_BIND_ADDRESS` | `127.0.0.1` | Address of the Prometheus, Tempo, Loki and Grafana ports, for a Grafana on another host |
| `METRICS_RETENTION` | `5y` | Prometheus retention |

## Grafana

To use an existing Grafana, add Prometheus, Tempo and Loki as datasources with
the URLs reachable from it. `grafana/provisioning/datasources/datasources.yaml`
shows the correlations to set up: Tempo to Loki by trace id, Tempo's service
map from Prometheus, and a Loki derived field that turns `trace_id=...` in a
log line into a Tempo link.

To try the stack on its own, start the bundled Grafana, which is provisioned
with those datasources:

```bash
docker compose --profile grafana up -d
```

## Sending data

libby exports when it is installed with its `otel` extra and an OTLP endpoint
is set:

```bash
pip install "libby[otel]"
export OTEL_EXPORTER_OTLP_ENDPOINT=http://<collector-host>:4318
```
