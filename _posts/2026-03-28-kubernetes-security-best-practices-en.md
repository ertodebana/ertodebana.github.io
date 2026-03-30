---
title: "Kubernetes Security Best Practices"
date: 2026-03-28 10:00:00 +0300
lang: en
locale: en-US
page_id: kubernetes-security
permalink: /posts/kubernetes-security-best-practices/
categories: [DevOps, Security]
tags: [kubernetes, security, rbac, network-policy, pod-security]
description: "The core practices you should apply to harden Kubernetes clusters."
---

## Why Kubernetes Security Needs Intentional Work

Kubernetes is not secure just because it is production-grade. Real-world hardening requires identity controls, workload restrictions, network isolation, image policy, and secrets management to work together.

## 1. Enforce least privilege with RBAC

Every user and service account should get only the permissions it actually needs:

```yaml
# read-only-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["pods", "pods/log"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
  - kind: User
    name: developer@company.com
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## 2. Use Pod Security Standards

Apply namespace-level enforcement so unsafe workloads are blocked early:

```yaml
# restricted-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

Example of a more secure Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: myapp:v1.0.0@sha256:abc123...
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop:
            - ALL
      resources:
        limits:
          memory: "256Mi"
          cpu: "500m"
        requests:
          memory: "128Mi"
          cpu: "250m"
```

## 3. Default-deny with NetworkPolicy

Start by blocking traffic, then open only what is required:

```yaml
# default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
---
# allow-specific.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

## 4. Secure the image supply chain

Prefer signed images, fixed digests, and policy enforcement:

```yaml
# Kyverno image policy
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-signed-images
spec:
  validationFailureAction: Enforce
  rules:
    - name: verify-signature
      match:
        any:
          - resources:
              kinds:
                - Pod
      verifyImages:
        - imageReferences:
            - "registry.company.com/*"
          attestors:
            - entries:
                - keys:
                    publicKeys: |-
                      -----BEGIN PUBLIC KEY-----
                      ...
                      -----END PUBLIC KEY-----
```

## 5. Manage secrets outside the cluster

Kubernetes Secrets are base64-encoded objects, not a complete secrets strategy. External systems such as Vault or cloud secrets managers are safer:

```yaml
# External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: db-credentials
  data:
    - secretKey: password
      remoteRef:
        key: secret/data/production/db
        property: password
```

## Security checklist

- [ ] Least privilege with RBAC
- [ ] Pod Security Standards enforcement
- [ ] Default-deny network policies
- [ ] Image signing and scanning
- [ ] External secrets management
- [ ] Audit logging
- [ ] Resource limits
- [ ] Regular cluster security scans

## Conclusion

Kubernetes security is a layered discipline. Identity, workload, network, image, and secret controls reinforce each other. Relying on only one of them leaves large gaps.
