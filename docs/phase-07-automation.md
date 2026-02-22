# Phase 7 — Infrastructure Lifecycle Automation

## 🧭 Overview

In this phase, I automated the full infrastructure lifecycle for the AWS EKS platform — from resource provisioning to Kubernetes workload deployment — using shell scripts that orchestrate Terraform and kubectl in the correct dependency order.

The goal was to eliminate all manual steps required to bring the platform from zero to a fully running state, and to make teardown equally safe and reproducible.

Since Terraform provisioning was already verified as stable in Phase 5, this phase focused on **operational orchestration**: encoding the correct execution order and dependency awareness into executable scripts.

---

## ❓ Why This Phase Exists

After Phase 5 established Terraform as the infrastructure layer, two operational gaps remained:

- Terraform stacks had to be applied manually in the correct order across multiple directories
- Kubernetes manifests had to be applied separately, with no enforcement of resource dependency order

In practice, this meant every environment bring-up required careful manual sequencing — a process that was error-prone and not repeatable under time pressure.

This phase closes that gap by introducing orchestration scripts that encode the correct execution order as executable logic.

---

## 🏗 What Was Built

### Terraform Lifecycle Script

A single script drives the full AWS infrastructure lifecycle in dependency order:

**Apply order:**
```
network → eks → (wait for cluster) → addons/external-secrets → istio
```

**Destroy order:**
```
istio → addons/external-secrets → eks → network
```

Key behaviors:
- Each directory runs `terraform init` and `terraform apply` independently
- EKS readiness is verified via `aws eks wait cluster-active` before proceeding to the next step
- Destroy order is explicitly enforced to prevent dependency failures

```bash
#!/usr/bin/env bash
set -e

ENV=prod
BASE_DIR="$(cd "$(dirname "$0")" && pwd)/envs/$ENV"
REGION="ca-central-1"

apply_dir() {
  local dir=$1
  echo "🚀 Applying: $dir"
  cd "$BASE_DIR/$dir"
  terraform init -upgrade
  terraform apply -auto-approve
}

wait_for_eks() {
  echo "⏳ Waiting for EKS cluster to become ACTIVE..."
  CLUSTER_NAME=$(terraform -chdir="$BASE_DIR/eks" output -raw cluster_name)
  aws eks wait cluster-active --name "$CLUSTER_NAME" --region "$REGION"
  echo "✅ EKS cluster is ACTIVE"
}

apply_dir network
apply_dir eks
wait_for_eks
apply_dir addons/external-secrets
apply_dir istio
```

### Kubernetes Deployment Script

A separate script applies all Kubernetes manifests in dependency order:

```
namespace → serviceaccount → configmap → deployment → service
→ clustersecretstore → externalsecret → istio-gateway → istio-virtualservice
```

This ordering ensures that resources which depend on others are never applied before their dependencies exist:
- ExternalSecret requires ClusterSecretStore to be ready
- Deployment requires ConfigMap and Secret to exist
- Istio VirtualService requires Gateway to be registered

---

## 🚧 Key Problem Solved — OIDC Drift

### Problem

Every time the EKS cluster is recreated, AWS generates a new OIDC issuer URL.  
The IAM Role trust policy for External Secrets IRSA must reference this URL.  
Previously this required a manual update after every cluster recreation — easy to forget, and the resulting auth failure was non-obvious to debug.

### Root Cause

The OIDC URL was hardcoded in the IAM trust policy, with no connection to the actual EKS Terraform resource.

### Resolution

The EKS Terraform module now exports the OIDC provider ARN and issuer URL as outputs:

```hcl
output "oidc_provider_arn" {
  value = aws_iam_openid_connect_provider.eks.arn
}

output "cluster_oidc_issuer" {
  value = aws_eks_cluster.this.identity[0].oidc[0].issuer
}
```

The External Secrets IAM module reads these values via `terraform_remote_state` and constructs the trust policy dynamically using `locals`:

```hcl
locals {
  oidc_provider_arn = data.terraform_remote_state.eks.outputs.oidc_provider_arn
  oidc_provider_url = replace(
    data.terraform_remote_state.eks.outputs.cluster_oidc_issuer,
    "https://", ""
  )
}

resource "aws_iam_role" "external_secrets" {
  name = "external-secrets-irsa-prod"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = local.oidc_provider_arn
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringEquals = {
          "${local.oidc_provider_url}:sub" = "system:serviceaccount:external-secrets:external-secrets"
          "${local.oidc_provider_url}:aud" = "sts.amazonaws.com"
        }
      }
    }]
  })
}
```

The trust policy is now always correct regardless of how many times the cluster is recreated — no manual intervention required.

---

## ⚙️ Istio + NLB Configuration (Without AWS Load Balancer Controller)

During this phase, the decision was made to use Istio with an AWS NLB directly via the in-tree Kubernetes controller, without deploying the AWS Load Balancer Controller.

**Annotation configuration:**

```hcl
annotations = {
  "service.beta.kubernetes.io/aws-load-balancer-type"   = "nlb"
  "service.beta.kubernetes.io/aws-load-balancer-scheme" = "internet-facing"
}
```

Key distinction:

| Mode | Annotation | Requires AWS LBC |
|------|------------|-----------------|
| In-tree NLB | `type: nlb` | ❌ No |
| AWS LBC NLB | `type: external` + `nlb-target-type: ip` | ✅ Yes |

The in-tree approach is simpler for this use case and fully supports Istio Gateway + VirtualService routing.

---

## ⚖️ Trade-offs and Limitations

**Benefits:**
- Full bring-up and teardown is now a single script execution
- OIDC trust policy is always correct without manual updates
- Deployment order is enforced rather than remembered
- Destroy ordering prevents resource dependency failures

**Limitations:**
- Scripts are environment-specific and not yet parameterized for multi-environment use
- No rollback logic if a mid-sequence step fails — a partial apply requires manual recovery

---

## 🔁 What I'd Do Differently

- Parameterize scripts to support multiple environments (dev/staging/prod) without code duplication
- Add failure recovery logic so a partial apply can resume from the failed step rather than restarting from scratch
- Consider replacing shell orchestration with a tool like Terragrunt for more robust dependency management at scale

---

## 📂 Repository

Infrastructure code for this phase is available here:

[https://github.com/nemoluv27/finance-app-terraform](https://github.com/nemoluv27/finance-app-terraform)
