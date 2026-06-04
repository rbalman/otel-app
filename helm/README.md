# demo

Helm chart for the otel-demo app — a Go HTTP service with OpenTelemetry tracing, metrics, and Prometheus scraping built in.

## Add the repo

```bash
helm repo add otel-app https://rbalman.github.io/otel-app
helm repo update
```

## Install

```bash
helm install otel-app otel-app/otel-app-chart
```

With custom values:

```bash
helm install otel-app otel-app/otel-app-chart \
  --set image.tag=v0.1.0 \
  --set otel.endpoint=otel-collector.monitoring.svc.cluster.local:4317
```

## Configuration

| Key | Default | Description |
|-----|---------|-------------|
| `image.repository` | `demo-app` | Container image |
| `image.tag` | `v0.0.3` | Image tag |
| `image.pullPolicy` | `IfNotPresent` | Pull policy |
| `replicaCount` | `1` | Number of replicas |
| `service.port` | `8080` | HTTP port |
| `resources.requests.cpu` | `50m` | CPU request |
| `resources.requests.memory` | `64Mi` | Memory request |
| `resources.limits.cpu` | `200m` | CPU limit |
| `resources.limits.memory` | `128Mi` | Memory limit |
| `otel.serviceName` | `demo-app` | `OTEL_SERVICE_NAME` |
| `otel.endpoint` | `otel-collector.monitoring.svc.cluster.local:4317` | OTLP gRPC endpoint |
| `otel.resourceAttributes` | `""` | Extra `OTEL_RESOURCE_ATTRIBUTES` |
| `serviceMonitor.enabled` | `true` | Create a Prometheus ServiceMonitor |
| `serviceMonitor.interval` | `15s` | Scrape interval |
| `serviceMonitor.release` | `prom-stack` | `release` label to match your Prometheus operator |
| `ingress.enabled` | `false` | Enable ingress |
| `ingress.host` | `demo.local` | Ingress hostname |
| `hpa.enabled` | `false` | Enable HorizontalPodAutoscaler |
| `hpa.minReplicas` | `1` | Min replicas |
| `hpa.maxReplicas` | `5` | Max replicas |
| `hpa.targetCPUUtilizationPercentage` | `70` | CPU target % |
| `hpa.targetMemoryUtilizationPercentage` | `80` | Memory target % |

## Endpoints

| Path | Description |
|------|-------------|
| `/health` | Liveness probe |
| `/ready` | Readiness probe |
| `/metrics` | Prometheus metrics |

## Prerequisites

- Kubernetes 1.21+
- Helm 3.x
- An OpenTelemetry Collector reachable at `otel.endpoint`
- Prometheus Operator (if `serviceMonitor.enabled: true`)
