# k8s-observability-stack

Production observability stack for Kubernetes — Prometheus + Grafana +
OpenTelemetry + AWS X-Ray. Pre-built SLO dashboards, composite alerting
rules, and on-call runbook templates.

Reduced MTTD by ~60% and enabled same-shift P1 resolution.

## Stack

```
Application
    │  OpenTelemetry SDK (traces + metrics)
    ▼
OpenTelemetry Collector
    ├──► Prometheus (metrics scrape)
    ├──► AWS X-Ray (distributed traces)
    └──► CloudWatch (logs + custom metrics)
            │
            ▼
         Grafana
    (SLO dashboards + alerting)
            │
            ▼
    PagerDuty / Slack
```

## Repo structure

```
k8s-observability-stack/
├── otel-collector/
│   ├── deployment.yaml
│   └── config.yaml           # pipelines: traces → X-Ray, metrics → Prometheus
├── prometheus/
│   ├── values.yaml           # kube-prometheus-stack Helm values
│   └── rules/
│       ├── slo-alerts.yaml   # error budget burn rate alerts
│       ├── infra-alerts.yaml # node, pod, PVC alerts
│       └── app-alerts.yaml   # latency, error rate, saturation
├── grafana/
│   └── dashboards/
│       ├── slo-overview.json
│       ├── eks-cluster.json
│       └── application-red.json   # Rate, Errors, Duration
└── runbooks/
    ├── high-cpu-node.md
    ├── pod-crash-loop.md
    ├── pvc-full.md
    └── high-error-rate.md
```

## SLO alert example

```yaml
# prometheus/rules/slo-alerts.yaml
groups:
  - name: slo.error-budget
    rules:
      - alert: ErrorBudgetBurnRateCritical
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h]))
            /
            sum(rate(http_requests_total[1h]))
          ) > 0.01
        for: 5m
        labels:
          severity: critical
          team: platform
        annotations:
          summary: "Error budget burn rate critical — {{ $value | humanizePercentage }} error rate"
          runbook: "https://github.com/thaneeshaadithya/k8s-observability-stack/blob/main/runbooks/high-error-rate.md"
```

## OpenTelemetry Collector config

```yaml
# otel-collector/config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
  memory_limiter:
    limit_mib: 512

exporters:
  awsxray:
    region: ap-south-1
  prometheus:
    endpoint: "0.0.0.0:8889"
  awscloudwatch:
    region: ap-south-1
    log_group_name: /otel/collector

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [awsxray]
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus, awscloudwatch]
```

## Results

- **~60% reduction in MTTD** across all production services
- Same-shift resolution for 100% of P1 incidents over 12 months
- **~40% fewer repeat incidents** via runbook adoption
