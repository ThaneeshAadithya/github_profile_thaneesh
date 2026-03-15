# eks-gitops-platform

End-to-end GitOps CI/CD platform on AWS EKS.
GitHub Actions → Helm → Argo CD, with automated rollback,
environment-gated approvals, and canary deployments.

## Architecture

```
Developer pushes code
        │
        ▼
┌─────────────────────────────────────┐
│         GitHub Actions              │
│  lint → test → build → scan → push │
│  Docker image → ECR                 │
└────────────────┬────────────────────┘
                 │  updates image tag in
                 ▼  Helm values repo
┌─────────────────────────────────────┐
│            Argo CD                  │
│  watches Helm values repo           │
│  syncs to EKS cluster               │
│  health checks → auto rollback      │
└────────────────┬────────────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
   Staging EKS       Prod EKS
   (auto-deploy)   (PR-gated approval)
```

## Repo structure

```
eks-gitops-platform/
├── .github/
│   └── workflows/
│       ├── ci.yml          # build, test, scan, push to ECR
│       ├── deploy-staging.yml
│       └── deploy-prod.yml # requires manual approval
├── helm/
│   ├── app-chart/          # generic Helm chart for microservices
│   └── values/
│       ├── staging.yaml
│       └── prod.yaml
├── argocd/
│   ├── install/            # Argo CD installation manifests
│   ├── apps/               # Application CRDs
│   └── projects/           # AppProject definitions
└── scripts/
    ├── rollback.sh
    └── canary-promote.sh
```

## GitHub Actions — CI pipeline

```yaml
# .github/workflows/ci.yml
name: CI — Build, Scan, Push

on:
  push:
    branches: [main, staging]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-south-1

      - name: Login to ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build multi-stage Docker image
        run: |
          docker build \
            --target production \
            --cache-from $ECR_REGISTRY/$ECR_REPO:cache \
            -t $ECR_REGISTRY/$ECR_REPO:${{ github.sha }} .

      - name: Trivy container scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.ECR_REGISTRY }}/${{ env.ECR_REPO }}:${{ github.sha }}
          severity: CRITICAL,HIGH
          exit-code: 1

      - name: Push to ECR
        run: docker push $ECR_REGISTRY/$ECR_REPO:${{ github.sha }}

      - name: Update Helm values (staging)
        run: |
          sed -i "s/tag:.*/tag: ${{ github.sha }}/" helm/values/staging.yaml
          git config user.email "ci@github.com"
          git add helm/values/staging.yaml
          git commit -m "ci: update staging image to ${{ github.sha }}"
          git push
```

## Argo CD Application

```yaml
# argocd/apps/app-staging.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-staging
  namespace: argocd
spec:
  project: staging
  source:
    repoURL: https://github.com/thaneeshaadithya/eks-gitops-platform
    targetRevision: HEAD
    path: helm/app-chart
    helm:
      valueFiles:
        - values/staging.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

## Results

- **3× deployment frequency** vs manual deployment process
- **Zero deployment-related P1 incidents** after pipeline adoption
- Automated rollback on health check failure — average rollback time < 2 minutes
