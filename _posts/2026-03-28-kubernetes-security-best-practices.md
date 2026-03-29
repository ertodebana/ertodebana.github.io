---
title: "Kubernetes Security Best Practices"
date: 2026-03-28 10:00:00 +0300
categories: [DevOps, Security]
tags: [kubernetes, security, rbac, network-policy, pod-security]
description: "Kubernetes cluster'larinizi guvenli hale getirmek icin uygulamaniz gereken best practice'ler."
---

## Kubernetes Guvenligi Neden Onemli?

Kubernetes, varsayilan konfigurasyonu ile **guvenli degildir**. Production ortamlarinda ciddi guvenlik onlemleri almak sart. Bu yazida, Kubernetes cluster'larinizi guvenli hale getirmek icin uygulamaniz gereken temel pratikleri inceliyoruz.

## 1. RBAC (Role-Based Access Control)

Her kullanici ve servis hesabi icin minimum yetki prensibi uygulayin:

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

## 2. Pod Security Standards

Pod'larin guvenlik seviyelerini zorunlu kilin:

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

Guvenli bir Pod ornegi:

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

## 3. Network Policies

Default-deny network policy uygulayarak yalnizca gerekli trafige izin verin:

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

## 4. Image Security

```yaml
# Kyverno ile image policy
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

## 5. Secret Management

Kubernetes Secret'lari base64 encoded'dir, sifrelenmez! Harici secret management kullanin:

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

## Guvenlik Kontrol Listesi

- [ ] RBAC ile minimum yetki prensibi
- [ ] Pod Security Standards (restricted)
- [ ] Network Policies (default deny)
- [ ] Image signing ve scanning
- [ ] Secret'lar icin harici yonetim (Vault, AWS Secrets Manager)
- [ ] Audit logging aktif
- [ ] Resource limits tanimli
- [ ] Regular guvenlik taramasi (Trivy, Kubescape)

## Sonuc

Kubernetes guvenligi katmanli bir yaklasim gerektirir. Tek bir onlem yeterli degildir; yukaridaki tum pratikleri birlikte uygulamak gerekir. Guvenlik, bir defaya mahsus bir is degil, surekli bir surectir.
