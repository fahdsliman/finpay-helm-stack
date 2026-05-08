# finpay-helm-stack

A production-style Helm chart for deploying a three-service payment microservice
stack on Kubernetes. Built to reflect real-world SRE practices in high-stakes
financial infrastructure.

---

## Architecture

┌─────────────────────────────────────────────────┐
│                finpay-helm-stack                 │
│                                                  │
│  ┌─────────────┐      ┌─────────────────┐        │
│  │ api-gateway │ ───▶ │  txn-processor  │        │
│  │  port 8080  │      │   port 8081     │        │
│  └─────────────┘      └────────┬────────┘        │
│                                │                 │
│                       ┌────────▼────────┐        │
│                       │  fraud-checker  │        │
│                       │   port 8082     │        │
│                       └─────────────────┘        │
└─────────────────────────────────────────────────┘

---

## Services

| Service | Port | Min Replicas (prod) | Max Replicas (prod) | CPU Trigger |
|---|---|---|---|---|
| api-gateway | 8080 | 5 | 50 | 70% |
| txn-processor | 8081 | 10 | 100 | 60% |
| fraud-checker | 8082 | 5 | 20 | 70% |

The transaction processor scales most aggressively and triggers autoscaling at
60% CPU — lower than the others because it is the latency-critical hot path in
the payment flow.

---

## Stack

- **Helm** — chart packaging and templating
- **Kubernetes** — Deployments, Services, HPAs
- **Prometheus** — ServiceMonitor for metrics scraping
- **Autoscaling/v2** — CPU-based horizontal pod autoscaling

---

## Quick Start

```bash
# Dev environment (default values)
helm template finpay-dev ./finpay

# Staging
helm template finpay-staging ./finpay -f environments/staging.yaml

# Production
helm template finpay-prod ./finpay -f environments/production.yaml

# Deploy to a cluster
helm install finpay-prod ./finpay -f environments/production.yaml
```

---

## Project Structure

finpay-helm-stack/
├── finpay/
│   ├── Chart.yaml
│   ├── values.yaml                        # Default (dev) values
│   └── templates/
│       ├── deployment-api-gateway.yaml
│       ├── deployment-txn-processor.yaml
│       ├── deployment-fraud-checker.yaml
│       ├── service-api-gateway.yaml
│       ├── service-txn-processor.yaml
│       ├── service-fraud-checker.yaml
│       ├── hpa-api-gateway.yaml
│       ├── hpa-txn-processor.yaml
│       ├── hpa-fraud-checker.yaml
│       ├── servicemonitor.yaml
│       └── NOTES.txt
└── environments/
├── staging.yaml                       # Staging overrides
└── production.yaml                    # Production overrides

---

## Environment Overrides

Each environment overrides only what changes — replica counts, resource limits,
and autoscaling thresholds. The base chart stays unchanged.

| Setting | Dev | Staging | Production |
|---|---|---|---|
| txn-processor replicas | 3 | 5 | 10 |
| txn-processor maxReplicas | 20 | 30 | 100 |
| api-gateway CPU limit | 500m | 750m | 2000m |

---

## Observability

Each service exposes `/metrics` and is configured with Prometheus annotations.
A `ServiceMonitor` resource is included for Prometheus Operator scraping at a
15s interval.

---

**Author:** Fahd Sliman — Senior SRE  
Designed to reflect payment infrastructure reliability patterns from
high-stakes financial environments.

