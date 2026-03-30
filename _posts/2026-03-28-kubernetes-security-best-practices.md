---
title: "Kubernetes Security Best Practices"
date: 2026-03-28 10:00:00 +0300
lang: tr
locale: tr-TR
page_id: kubernetes-security
permalink: /posts/kubernetes-security-best-practices/
categories: [DevOps, Security]
tags: [kubernetes, security, rbac, network-policy, pod-security]
description: "Kubernetes güvenliğini varsayılan ayarlara bırakmanın neden kötü bir fikir olduğunu ve hangi kontrollerin gerçekten fark yarattığını anlatıyorum."
---

## Kubernetes Güvenliği Neden Önemli?

Kubernetes'in en tehlikeli tarafı, ilk bakışta "zaten enterprise-grade" görünmesidir. Oysa varsayılan kurulumların büyük kısmı güvenli değil; sadece henüz başınıza bir şey gelmemiş olabilir.

Kubernetes, varsayılan ayarlarıyla güvenli kabul edilmemelidir. Üretim ortamında çalışan kümelerde kimlik, ağ, imaj, secret ve çalışma zamanı güvenliği birlikte düşünülmelidir.

## 1. RBAC ile minimum yetki

Her kullanıcı ve servis hesabı için minimum yetki prensibini uygulayın:

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

Ad alanı seviyesinde güvenlik politikalarını zorunlu kılın:

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

Güvenli bir Pod örneği:

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

## 3. Network Policy ile varsayılan reddetme

Önce tüm trafiği kapatıp sonra gereken erişimi açmak daha güvenlidir:

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

## 4. İmaj güvenliği

İmzalı, taranmış ve sabit digest ile kullanılan imajlar tercih edilmelidir:

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

## 5. Secret yönetimi

Kubernetes Secret nesneleri yalnızca base64 encoded'dir; asıl güvenlik için harici secret yönetimi gerekir:

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

## Kontrol listesi

- [ ] RBAC ile minimum yetki
- [ ] Pod Security Standards seviyeleri
- [ ] NetworkPolicy ile default deny
- [ ] İmaj signing ve scanning
- [ ] Secret'lar için harici yönetim
- [ ] Audit logging
- [ ] Kaynak limitleri
- [ ] Düzenli güvenlik taramaları

## Sonuç

Kubernetes güvenliği tek bir YAML veya tek bir policy ile çözülmez. RBAC'i bırakıp sadece image scan yapmak ya da NetworkPolicy yazıp secret yönetimini boş vermek yarım güvenlik üretir.

Benim gördüğüm pratik yaklaşım şu: önce en bariz açıkları kapatın, sonra cluster'ı katman katman sertleştirin. Kimlik, ağ, imaj ve çalışma zamanı kontrolleri birlikte uygulandığında küme gerçekten daha dayanıklı hâle gelir.
