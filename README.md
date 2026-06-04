# otel-app

A Go HTTP service instrumented with OpenTelemetry, packaged as a multi-arch Docker image and deployed via Helm.

## Overview

The application exposes a small set of HTTP endpoints that demonstrate distributed tracing (OTLP/gRPC), Prometheus metrics, and structured JSON logs — all correlated by a `request_id` propagated through OTel baggage.

### Endpoints`

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Liveness check — returns `{"status":"ok"}` |
| GET | `/ready` | Readiness check — returns `{"status":"ready"}` |
| GET | `/metrics` | Prometheus metrics scrape endpoint |
| GET | `/api/hello?name=<n>` | Greeting with child span for input validation |
| GET | `/api/items` | Simulates cache lookup + optional DB query |
| GET | `/api/error` | Randomly fails ~60 % of the time (error-rate demo) |

### Observability signals

- **Traces** — exported via OTLP/gRPC to an OpenTelemetry Collector; every request gets a root span with child spans for sub-operations
- **Metrics** — `http_request_duration_seconds` (histogram), `http_requests_total`, `http_errors_total` scraped by Prometheus
- **Logs** — structured JSON via `log/slog`, enriched with `trace_id`, `span_id`, and `request_id`
- **Baggage** — a `request_id` (UUID) is generated per request, stored in [OTel baggage](https://opentelemetry.io/docs/concepts/signals/baggage/), and propagated to downstream services; it is also attached as a span attribute and injected into every log line, enabling correlation across traces, metrics, and logs in tools like Grafana

## Running locally

```bash
go run .
# or
PORT=8080 OTEL_SERVICE_NAME=demo-app go run .
```

Build and run via Docker:

```bash
docker build -t otel-demo:dev .
docker run -p 8080:8080 otel-demo:dev
```

## Configuration

The service is configured entirely through environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8080` | HTTP listen port |
| `OTEL_SERVICE_NAME` | `demo-app` | Service name reported to the collector |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | `otel-collector.monitoring.svc.cluster.local:4317` | OTLP/gRPC collector endpoint |

## Docker image

Multi-arch images (`linux/amd64`, `linux/arm64`) are published to Docker Hub on every merge to `main`:

```
docker pull balman/otel-demo:latest
```

## Helm chart

The chart lives in `helm/` and is published to GitHub Pages as a Helm repository.

```bash
helm repo add otel-app https://rbalman.github.io/otel-app
helm repo update
helm install demo otel-app/otel-app-chart
```

For the full values reference, Ingress/HPA examples, and probe configuration see [`helm/README.md`](helm/README.md).

## CI

Two GitHub Actions workflows run on pushes to `main`:

| Workflow | Trigger | What it does |
|----------|---------|--------------|
| `release.yaml` | Changes to Go source or Dockerfile | `go vet`, `govulncheck`, multi-arch Docker build, Trivy image scan, push to Docker Hub, create GitHub release |
| `helm-release.yaml` | Changes under `helm/` | `helm lint --strict`, dry-run render, publish chart to GitHub Pages via chart-releaser |

The two workflows are path-scoped so a chart-only change doesn't rebuild the image, and a code-only change doesn't re-release the chart.

## Design Decisions

- OTel SDK for instrumenting app, gives unified approach for metrics, logs and traces.
- Helm chart CI and publish in gh_pages
- Container Security Best Practices
  - Runs as a non-root user (`65534:65534`)
  - Read-only root filesystem
  - All Linux capabilities dropped
  - `allowPrivilegeEscalation: false`
- Image Security Best pratices 
  - Final image is based on `scratch` base image
  - Image scanned for `CRITICAL` CVEs on every build (results uploaded to the GitHub Security tab)
  - Multi platform `linux/arm64` and `linux/amd64` image build/push