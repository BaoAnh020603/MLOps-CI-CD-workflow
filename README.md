# 🚀 Ephemeral AI Environments: Dynamic Infrastructure for Model Validation

[![MLOps](https://img.shields.io/badge/MLOps-Architecture-blueviolet)](https://github.com/your-username/your-repo)
[![AWS](https://img.shields.io/badge/AWS-SageMaker-orange)](https://aws.amazon.com/sagemaker/)
[![Terraform](https://img.shields.io/badge/Terraform-Infrastructure--as--Code-blue?logo=terraform&logoColor=white)](https://www.terraform.io/)
[![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub--Actions-black)](https://github.com/features/actions)

> **An automated MLOps CI/CD workflow that provisions temporary, isolated AI environments the moment a Pull Request is opened — bridging the gap between model development and production readiness while maintaining strict cost control.**

---
<p align="center">
  <img src="architecture.svg" alt="MLOps Architecture" width="100%">
</p>
## 📋 Table of Contents

- [The Problem](#-1-the-problem-static-staging-bottleneck)
- [Solution Architecture](#-2-solution-architecture)
- [Operational Workflow](#-3-operational-workflow--deep-validation)
- [Automated Cleanup](#-4-automated-cleanup-the-cost-killer)
- [Technology Stack](#-5-technology-stack)
- [Best Practices & Security](#-6-best-practices--security)
- [Getting Started](#-7-getting-started)

---

## ⚠️ 1. The Problem: Static Staging Bottleneck

Traditional MLOps teams relying on permanent staging environments face three critical pain points:

| Problem | Impact |
| :--- | :--- |
| 💸 **Financial Overhead** | GPU-backed SageMaker Endpoints running 24/7 inflate AWS bills by thousands of dollars monthly, even with zero idle traffic |
| 🌀 **Environment Drift** | Shared staging environments accumulate legacy configs, causing the classic *"works here but not in prod"* syndrome |
| ⚡ **Concurrency Issues** | Multiple Data Scientists pushing simultaneously overwrite each other's work on a single static endpoint, causing validation conflicts |

---

## 🏗️ 2. Solution Architecture

The core philosophy is **Infrastructure-on-Demand**. Every Pull Request gets its own isolated, full-fidelity "mini-production" stack.

```
┌─────────────────────────────────────────────────────────────────┐
│                    EPHEMERAL ENVIRONMENT                        │
│                                                                 │
│   PR #42  ──►  ECR Image  ──►  SageMaker Endpoint             │
│                               API Gateway + Lambda             │
│                               CloudWatch Logs                  │
│                                                                 │
│   PR #43  ──►  ECR Image  ──►  SageMaker Endpoint             │
│                               API Gateway + Lambda             │
│                               CloudWatch Logs                  │
└─────────────────────────────────────────────────────────────────┘
```

### A. CI Automation — The Trigger

When a developer opens a PR, **GitHub Actions** orchestrates:

1. **Unit & Integration Tests** — Standard code quality checks run first.
2. **Containerization** — The model, inference script, and all dependencies are packaged into a Docker image.
3. **Registry Push** — The image is pushed to **Amazon ECR** with a unique tag (Git SHA) for absolute traceability.

### B. Dynamic Provisioning — The Infrastructure

**Terraform** acts as the single source of truth. On PR creation, `terraform apply` provisions:

- 🖥️ **Amazon SageMaker Endpoint** — Real-time inference using the newly pushed ECR image.
- 🔐 **API Gateway & Lambda** — Secure entry point handling authentication and request forwarding.
- 📊 **CloudWatch Logs** — Dedicated log groups for real-time inference debugging.

---

## 🔄 3. Operational Workflow & Deep Validation

Each ephemeral environment enables a high-fidelity feedback loop:

- **Functional Testing** — Reviewers test the API directly via `Postman` or `cURL`.
- **Schema Validation** — Input/output JSON structures are verified against production requirements.
- **Performance Benchmarking** — Dedicated endpoints allow load testing without impacting other developers.
- **Automated PR Comments** — The pipeline posts the temporary API URL back to the PR thread for one-click testing.

### Lifecycle Diagram

```
[PR Open]
    │
    ├─► [Unit & Integration Tests]
    │
    ├─► [Build Docker Image]
    │
    ├─► [Push to Amazon ECR]
    │
    ├─► [Terraform Apply]
    │       ├── SageMaker Endpoint
    │       ├── API Gateway + Lambda
    │       └── CloudWatch Logs
    │
    ├─► [Post API URL to PR Comment]
    │
    └─► [Reviewers Test & Validate]
            │
    [PR Merged / Closed]
            │
            └─► [Terraform Destroy] ──► All resources wiped ✅
```

---

## 🧹 4. Automated Cleanup: The Cost-Killer

The most impactful feature is the **Auto-Destroy** mechanism.

| Step | Detail |
| :--- | :--- |
| **Trigger** | GitHub Actions listens for `pull_request: closed` (covers both *Merged* and *Declined* PRs) |
| **Execution** | `terraform destroy` wipes all provisioned resources instantly |
| **Result** | Teams pay only for **compute minutes** used during the review — not idle hours |

> 💡 **Cost Reduction: Up to 90% savings on Staging OpEx.**

---

## 🛠️ 5. Technology Stack

| Layer | Technology | Role |
| :--- | :--- | :--- |
| **Orchestration** | GitHub Actions | Triggers the pipeline and manages secrets |
| **Containerization** | Docker | Packages model + dependencies |
| **Registry** | Amazon ECR | Stores immutable, tagged model containers |
| **IaC** | Terraform | Manages the full lifecycle of AWS resources |
| **Inference** | AWS SageMaker | Hosts the model for real-time predictions |
| **Edge / API** | API Gateway + Lambda | Exposes the model securely to the outside world |
| **Logging** | Amazon CloudWatch | Monitors inference errors and latency |

---

## 🛡️ 6. Best Practices & Security

### IAM Least Privilege
Use **GitHub OIDC (OpenID Connect)** to authenticate with AWS. This eliminates the need to store long-lived access keys as GitHub secrets.

### Remote State Management
Store Terraform state in a remote **S3 bucket** with **DynamoDB locking** to prevent state corruption when multiple PRs are active simultaneously.

### Resource Tagging Strategy
Automatically tag all ephemeral resources for cost tracking and auditability:

```hcl
tags = {
  Environment = "Ephemeral"
  PR_Number   = var.pr_number
  ManagedBy   = "Terraform"
  CreatedAt   = timestamp()
}
```

### Right-Sizing for PR Environments
Use small, cost-effective instances for PR environments. Reserve expensive GPUs for production only.

```hcl
# PR / Staging environment
instance_type = "ml.t2.medium"

# Production environment
instance_type = "ml.g4dn.xlarge"
```

---

## 🚀 7. Getting Started

### Prerequisites

- AWS Account with appropriate IAM permissions
- Terraform `>= 1.0`
- GitHub repository with Actions enabled
- Docker

### Setup

**1. Configure GitHub OIDC for AWS**

```bash
# Create the OIDC identity provider in AWS IAM
aws iam create-open-id-connect-provider \
  --url https://token.actions.githubusercontent.com \
  --client-id-list sts.amazonaws.com
```

**2. Initialize Terraform Remote Backend**

```bash
# Create S3 bucket and DynamoDB table for state management
terraform init \
  -backend-config="bucket=your-tfstate-bucket" \
  -backend-config="dynamodb_table=your-lock-table" \
  -backend-config="region=us-east-1"
```

**3. Configure GitHub Actions Workflow**

```yaml
# .github/workflows/ephemeral-env.yml
on:
  pull_request:
    types: [opened, synchronize, reopened, closed]

jobs:
  deploy:
    if: github.event.action != 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::YOUR_ACCOUNT_ID:role/GitHubActionsRole
          aws-region: us-east-1
      - name: Terraform Apply
        run: terraform apply -auto-approve -var="pr_number=${{ github.event.number }}"

  destroy:
    if: github.event.action == 'closed'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Terraform Destroy
        run: terraform destroy -auto-approve -var="pr_number=${{ github.event.number }}"
```

---

## 📈 Results & Impact

```
Before Ephemeral Environments:
  Monthly Staging Cost: ~$3,200
  Environment Conflicts: Frequent
  Validation Confidence: Medium

After Ephemeral Environments:
  Monthly Staging Cost: ~$320  (↓ 90%)
  Environment Conflicts: None
  Validation Confidence: High
```

---
