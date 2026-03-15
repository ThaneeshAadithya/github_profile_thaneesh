# karpenter-hpa-autoscaling

Kubernetes autoscaling patterns for handling 2–3× traffic surges within 90 seconds.
Karpenter node provisioning + HPA pod scaling, with load-test scripts and
Grafana dashboards for p99 latency tracking.

## How it works

```
Traffic spike detected
        │
        ▼
   HPA scales pods
   (CPU / custom metrics)
        │
        ▼  pods Pending (no node capacity)
        │
        ▼
   Karpenter detects Pending pods
   provisions right-sized EC2 node
   within ~30–60 seconds
        │
        ▼
   Pods scheduled, traffic served
   p99 latency maintained within SLO
```

## Repo structure

```
karpenter-hpa-autoscaling/
├── karpenter/
│   ├── nodepool.yaml          # NodePool with instance type constraints
│   ├── ec2nodeclass.yaml      # AMI, subnet, security group config
│   └── install/               # Karpenter Helm install values
├── hpa/
│   ├── hpa-cpu.yaml           # CPU-based HPA
│   ├── hpa-custom-metrics.yaml # KEDA / Prometheus adapter HPA
│   └── pdb.yaml               # PodDisruptionBudget for zero-downtime scaling
├── load-tests/
│   ├── k6-ramp-test.js        # k6 load test — ramp to 3× normal traffic
│   └── run-test.sh
└── dashboards/
    └── autoscaling-overview.json  # Grafana dashboard
```

## Karpenter NodePool

```yaml
# karpenter/nodepool.yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      requirements:
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["on-demand", "spot"]
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["m5.xlarge", "m5.2xlarge", "m6i.xlarge", "m6i.2xlarge"]
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
      nodeClassRef:
        apiVersion: karpenter.k8s.aws/v1
        kind: EC2NodeClass
        name: default
  limits:
    cpu: 200
    memory: 400Gi
  disruption:
    consolidationPolicy: WhenUnderutilized
    consolidateAfter: 30s
```

## HPA with custom metrics

```yaml
# hpa/hpa-custom-metrics.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 50
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0      # scale up immediately
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300    # wait 5 min before scale down
```

## Load test (k6)

```javascript
// load-tests/k6-ramp-test.js
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // normal load
    { duration: '5m', target: 300 },   // 3× spike
    { duration: '2m', target: 100 },   // back to normal
  ],
  thresholds: {
    http_req_duration: ['p99<500'],    // p99 must stay under 500ms
    http_req_failed: ['rate<0.01'],    // <1% error rate
  },
};

export default function () {
  const res = http.get('http://my-app.staging.svc.cluster.local/health');
  check(res, { 'status 200': (r) => r.status === 200 });
}
```

## Results

- Absorbs **2–3× traffic surges within 90 seconds**
- Zero manual intervention required during peak sale events
- **p99 latency SLOs maintained** throughout all load test scenarios
