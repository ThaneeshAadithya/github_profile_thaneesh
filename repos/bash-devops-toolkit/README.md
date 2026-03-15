# bash-devops-toolkit

Battle-tested shell scripts for DevOps automation — deployment support,
log rotation, health checks, and environment provisioning.
Saved ~4 hours/week of repetitive manual engineering effort.

## Scripts

```
bash-devops-toolkit/
├── deployment/
│   ├── rolling-deploy.sh        # zero-downtime k8s rolling deploy
│   ├── canary-promote.sh        # promote canary to 100%
│   └── rollback.sh              # roll back to previous image
├── health-checks/
│   ├── cluster-health.sh        # EKS node + pod health summary
│   └── service-health.sh        # HTTP health check with retry
├── log-management/
│   └── rotate-logs.sh           # rotate + compress + upload to S3
├── environment/
│   └── env-bootstrap.sh         # bootstrap a new environment
└── utils/
    └── aws-assume-role.sh       # assume IAM role and export creds
```

---

### rolling-deploy.sh

```bash
#!/bin/bash
# Usage: ./rolling-deploy.sh <namespace> <deployment> <image:tag>
set -euo pipefail

NAMESPACE="${1:?Usage: $0 <namespace> <deployment> <image:tag>}"
DEPLOYMENT="${2:?}"
IMAGE="${3:?}"

echo "→ Updating $DEPLOYMENT in $NAMESPACE to $IMAGE"
kubectl set image deployment/"$DEPLOYMENT" \
  app="$IMAGE" \
  --namespace="$NAMESPACE"

echo "→ Waiting for rollout..."
kubectl rollout status deployment/"$DEPLOYMENT" \
  --namespace="$NAMESPACE" \
  --timeout=300s

if [ $? -ne 0 ]; then
  echo "✗ Rollout failed — rolling back"
  kubectl rollout undo deployment/"$DEPLOYMENT" --namespace="$NAMESPACE"
  exit 1
fi

echo "✓ Rollout complete"
kubectl get pods -n "$NAMESPACE" -l app="$DEPLOYMENT"
```

---

### cluster-health.sh

```bash
#!/bin/bash
# Quick EKS cluster health summary
set -euo pipefail

echo "════════════════════════════════"
echo " EKS Cluster Health Summary"
echo "════════════════════════════════"

echo ""
echo "── Node Status ──"
kubectl get nodes -o wide

echo ""
echo "── Unhealthy Pods (all namespaces) ──"
kubectl get pods --all-namespaces \
  --field-selector='status.phase!=Running,status.phase!=Succeeded' \
  2>/dev/null || echo "  All pods healthy"

echo ""
echo "── Recent Events (warnings) ──"
kubectl get events --all-namespaces \
  --field-selector='type=Warning' \
  --sort-by='.lastTimestamp' \
  | tail -20

echo ""
echo "── Node Resource Usage ──"
kubectl top nodes 2>/dev/null || echo "  Metrics server not available"

echo ""
echo "── PVC Status ──"
kubectl get pvc --all-namespaces \
  | grep -v Bound || echo "  All PVCs bound"
```

---

### rotate-logs.sh

```bash
#!/bin/bash
# Rotate and compress application logs, upload to S3
set -euo pipefail

LOG_DIR="${LOG_DIR:-/var/log/app}"
S3_BUCKET="${S3_BUCKET:?S3_BUCKET env var required}"
RETAIN_DAYS="${RETAIN_DAYS:-7}"
DATE=$(date +%Y-%m-%d)

echo "→ Compressing logs older than $RETAIN_DAYS days"
find "$LOG_DIR" -name "*.log" -mtime +"$RETAIN_DAYS" | while read -r logfile; do
  gzip "$logfile"
  echo "  Compressed: $logfile"
done

echo "→ Uploading compressed logs to s3://$S3_BUCKET/logs/$DATE/"
aws s3 sync "$LOG_DIR" "s3://$S3_BUCKET/logs/$DATE/" \
  --exclude "*" \
  --include "*.gz" \
  --storage-class STANDARD_IA

echo "→ Removing uploaded logs from disk"
find "$LOG_DIR" -name "*.gz" -mtime +"$RETAIN_DAYS" -delete

echo "✓ Log rotation complete"
```

---

### aws-assume-role.sh

```bash
#!/bin/bash
# Assume an IAM role and export credentials to current shell
# Usage: source ./aws-assume-role.sh <role-arn> [session-name]
set -euo pipefail

ROLE_ARN="${1:?Usage: source $0 <role-arn> [session-name]}"
SESSION_NAME="${2:-devops-session-$(date +%s)}"

echo "→ Assuming role: $ROLE_ARN"
CREDS=$(aws sts assume-role \
  --role-arn "$ROLE_ARN" \
  --role-session-name "$SESSION_NAME" \
  --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' \
  --output text)

export AWS_ACCESS_KEY_ID=$(echo "$CREDS" | awk '{print $1}')
export AWS_SECRET_ACCESS_KEY=$(echo "$CREDS" | awk '{print $2}')
export AWS_SESSION_TOKEN=$(echo "$CREDS" | awk '{print $3}')

echo "✓ Role assumed. Credentials exported for this session."
aws sts get-caller-identity
```
