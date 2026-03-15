# Hi, I'm Thaneesh Aadithya 👋

### Senior DevOps Engineer · AWS · Kubernetes (EKS) · Terraform · GitOps · CI/CD

I design, automate, and operate large-scale cloud-native platforms on AWS. 6+ years
building production infrastructure that handles millions of daily transactions — focused
on reliability, automation, and making deployments boring (in the best way).

Currently at **Target Corporation** (Bengaluru) — owning a production EKS cluster
running 30+ microservices with 99.9%+ uptime.

---

## 🛠 Tech Stack

| Domain | Tools |
|---|---|
| **Cloud / AWS** | EC2 · EKS · VPC · IAM · ALB · S3 · RDS · Aurora · ECR · Route 53 · Secrets Manager · CloudWatch · X-Ray |
| **Containers** | Kubernetes · Docker (multi-stage) · Helm · Karpenter · HPA · VPA · RBAC |
| **IaC** | Terraform (modular · remote state · workspaces · OPA policy-as-code) |
| **CI/CD & GitOps** | GitHub Actions · Argo CD · Jenkins · blue-green · canary |
| **Observability** | Prometheus · Grafana · OpenTelemetry · AWS X-Ray · CloudWatch |
| **Scripting** | Bash / Shell · Python · YAML · Linux |

---

## 📌 Featured Projects

<!-- Each of these maps to a pinned repo — see repo list below -->

### 🏗 [terraform-aws-modules](https://github.com/thaneeshaadithya/terraform-aws-modules)
Production-grade, reusable Terraform modules for AWS.
VPC · EKS · RDS · IAM · ALB · ECR · Secrets Manager.
Remote state with S3 + DynamoDB locking. Multi-environment workspaces.

### ☸️ [eks-gitops-platform](https://github.com/thaneeshaadithya/eks-gitops-platform)
End-to-end GitOps platform on AWS EKS.
GitHub Actions → Helm → Argo CD pipeline with automated rollback,
environment-gated promotions, and canary deployments.

### 📊 [k8s-observability-stack](https://github.com/thaneeshaadithya/k8s-observability-stack)
Prometheus + Grafana + OpenTelemetry + AWS X-Ray setup for Kubernetes.
Pre-built SLO dashboards, composite alerting rules, and on-call runbook templates.

### 🔐 [aws-secrets-hardening](https://github.com/thaneeshaadithya/aws-secrets-hardening)
AWS Secrets Manager integration patterns for containerised workloads.
IAM least-privilege bindings, automatic rotation, and zero-hardcoded-credentials
enforcement via OPA policy-as-code.

### 🚀 [karpenter-hpa-autoscaling](https://github.com/thaneeshaadithya/karpenter-hpa-autoscaling)
Kubernetes autoscaling patterns: HPA + Karpenter node provisioning.
Handles 2-3× traffic surges within 90 seconds. Includes load-test scripts
and Grafana dashboards for p99 latency tracking.

### 🐳 [docker-multistage-builds](https://github.com/thaneeshaadithya/docker-multistage-builds)
Multi-stage Dockerfile patterns that reduced image sizes by 40-50% across 30+ services.
Before/after comparisons, size benchmarks, and CI pipeline integration examples.

---

## 📈 Impact at a Glance

```
3×    deployment frequency (GitHub Actions + Argo CD GitOps pipeline)
40-50% smaller Docker images (multi-stage build patterns)
~60%  faster incident detection (OpenTelemetry + Prometheus + Grafana)
~40%  reduction in repeat incidents (blameless postmortems + runbooks)
6h→30min  environment provisioning (Terraform module library)
99.9%+    platform uptime over 24 months (EKS + Karpenter + HPA)
```

---

## 🗂 All Repositories

| Repo | What it covers |
|---|---|
| [terraform-aws-modules](https://github.com/thaneeshaadithya/terraform-aws-modules) | VPC, EKS, RDS, IAM, ALB — modular, remote-state, multi-env |
| [eks-gitops-platform](https://github.com/thaneeshaadithya/eks-gitops-platform) | GitHub Actions + Helm + Argo CD · blue-green · canary |
| [k8s-observability-stack](https://github.com/thaneeshaadithya/k8s-observability-stack) | Prometheus · Grafana · OpenTelemetry · X-Ray · SLO dashboards |
| [aws-secrets-hardening](https://github.com/thaneeshaadithya/aws-secrets-hardening) | Secrets Manager · IAM least-privilege · OPA policy-as-code |
| [karpenter-hpa-autoscaling](https://github.com/thaneeshaadithya/karpenter-hpa-autoscaling) | Karpenter + HPA · traffic surge handling · load tests |
| [docker-multistage-builds](https://github.com/thaneeshaadithya/docker-multistage-builds) | Multi-stage builds · size benchmarks · CI integration |
| [jenkins-terraform-ci](https://github.com/thaneeshaadithya/jenkins-terraform-ci) | Jenkins pipeline for Terraform plan/apply · drift detection |
| [bash-devops-toolkit](https://github.com/thaneeshaadithya/bash-devops-toolkit) | Shell scripts for deployments · log rotation · health checks |
| [incident-runbooks](https://github.com/thaneeshaadithya/incident-runbooks) | Runbook templates for EKS · Terraform · AWS P1/P2 incidents |

---

## 📬 Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-thaneesh--aadithya-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/thaneesh-aadithya)
[![Email](https://img.shields.io/badge/Email-thaneeshaadithya5%40gmail.com-EA4335?style=flat&logo=gmail)](mailto:thaneeshaadithya5@gmail.com)

---

*Based in Bengaluru, India · Open to senior DevOps / Platform Engineering / SRE roles*
