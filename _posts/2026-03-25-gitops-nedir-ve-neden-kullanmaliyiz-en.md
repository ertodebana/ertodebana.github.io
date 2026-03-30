---
title: "What Is GitOps and Why Should You Use It?"
date: 2026-03-25 10:00:00 +0300
lang: en
locale: en-US
page_id: gitops
permalink: /posts/what-is-gitops-and-why-use-it/
categories: [DevOps, GitOps]
tags: [gitops, argocd, flux, kubernetes, ci-cd]
description: "A practical take on GitOps as an operating discipline, not just another fashionable platform term."
---

## What Is GitOps?

GitOps is often presented like a silver bullet. In practice, the value is much less magical and much more useful: people stop poking production by hand, and the system has a clear path back to a known-good state.

GitOps is an operating model where **Git becomes the single source of truth** for infrastructure and application state. Desired state lives in version control, changes are reviewed through pull requests, and cluster state is continuously reconciled back to what is declared in Git.

## Core Principles

### 1. Declarative configuration

The target system state is defined explicitly:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    spec:
      containers:
        - name: web
          image: myapp:v1.2.3
```

### 2. Git becomes the source of truth

Every change flows through Git instead of direct cluster mutation. That gives you:

- **Auditability**: You can see who changed what and when.
- **Rollback**: Reverting infrastructure becomes a version-control operation.
- **Code review**: Infrastructure changes follow the same review path as application code.

### 3. Automated reconciliation

A GitOps operator continuously compares the desired state in Git with the actual state in the cluster. When drift appears, it reconciles the cluster back to the declared state. Argo CD and Flux are the most common choices here.

## Common GitOps Tools

| Tool | Strength | Typical fit |
|------|----------|-------------|
| **Argo CD** | Strong UI and multi-cluster support | Larger teams |
| **Flux** | Lightweight and Git-native | Kubernetes-first platforms |
| **Jenkins X** | Tight CI/CD integration | End-to-end delivery pipelines |

## A Simple Argo CD Example

```bash
# Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Create an application
argocd app create my-app \
  --repo https://github.com/org/my-app.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production
```

## Why GitOps Works Well

1. **Reliability**: It reduces manual mistakes.
2. **Speed**: Delivery becomes standardized and repeatable.
3. **Recoverability**: Every change is traceable and reversible.
4. **Security**: Teams need less direct access to production clusters.
5. **Collaboration**: Operational changes move through the same PR workflow as code.

## Conclusion

GitOps will not rescue weak engineering habits on its own. It will not make bad manifests safe or bad decisions reversible by magic. What it does do is remove a huge amount of operational ambiguity.

If your team still relies on shell access and tribal knowledge to manage production, GitOps is one of the cleanest ways to raise the bar. That is why it keeps sticking in real Kubernetes environments.
