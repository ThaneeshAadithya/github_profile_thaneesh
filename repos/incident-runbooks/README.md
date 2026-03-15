# incident-runbooks

Production runbook templates for EKS, Terraform, and AWS P1/P2 incidents.
Based on real postmortems — reduced repeat incidents by ~40% within 6 months.

## Runbooks

| Runbook | Severity | Avg resolve time |
|---|---|---|
| [Pod CrashLoopBackOff](./pod-crashloop.md) | P2 | 15 min |
| [Node NotReady](./node-not-ready.md) | P1 | 20 min |
| [High Error Rate (>1%)](./high-error-rate.md) | P1 | 25 min |
| [EKS Cluster Unreachable](./eks-unreachable.md) | P1 | 30 min |
| [Terraform State Lock](./tf-state-lock.md) | P2 | 10 min |
| [RDS High CPU / Slow Queries](./rds-slow-queries.md) | P2 | 20 min |
| [ALB 5xx Spike](./alb-5xx.md) | P1 | 15 min |

---

## pod-crashloop.md

```markdown
# Pod CrashLoopBackOff — Runbook

## Detection
Alert: `KubePodCrashLooping` (Prometheus)
Grafana: Error rate spike on service dashboard

## Immediate triage (< 5 min)

1. Identify the pod
   kubectl get pods -n <namespace> | grep CrashLoop

2. Check recent logs
   kubectl logs <pod> -n <namespace> --previous --tail=100

3. Describe the pod (check Events section)
   kubectl describe pod <pod> -n <namespace>

## Common causes & fixes

| Cause | Signal | Fix |
|---|---|---|
| OOM killed | `reason: OOMKilled` in describe | Increase memory limit in deployment |
| Config error | Exit code 1, config-related logs | Check ConfigMap / Secret values |
| Liveness probe too aggressive | `Liveness probe failed` in events | Increase `failureThreshold` or `initialDelaySeconds` |
| Image pull error | `ImagePullBackOff` | Check ECR permissions and image tag |
| Secrets missing | `secret not found` in logs | Verify Secret exists in namespace |

## Escalation
If not resolved in 20 min → escalate to on-call lead
Post in #incidents Slack channel with pod name, namespace, and logs snippet
```

---

## tf-state-lock.md

```markdown
# Terraform State Lock — Runbook

## Detection
Error: `Error acquiring the state lock`
DynamoDB table shows lock record that is not being released

## Triage

1. Check who holds the lock
   aws dynamodb get-item \
     --table-name terraform-state-lock \
     --key '{"LockID": {"S": "your-state-path"}}' \
     --region ap-south-1

2. Verify the CI job that created the lock
   Check GitHub Actions / Jenkins for the pipeline run ID in the lock info

3. If the locking job is dead/stuck — force unlock
   # Only do this if you are CERTAIN no apply is running
   terraform force-unlock <LOCK_ID>

## Prevention
- Never cancel a CI apply mid-run — always let it complete or fail cleanly
- Set `timeout` on Terraform CI steps so stuck jobs release lock
- Use `run_id` in lock metadata to trace back to CI pipeline
```
