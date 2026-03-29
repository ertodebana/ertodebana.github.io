---
title: "GitOps Nedir ve Neden Kullanmaliyiz?"
date: 2026-03-25 10:00:00 +0300
categories: [DevOps, GitOps]
tags: [gitops, argocd, flux, kubernetes, ci-cd]
description: "GitOps yaklasiminin temellerini, avantajlarini ve popüler araclari inceliyoruz."
---

## GitOps Nedir?

GitOps, **Git repository'sini tek dogru kaynak (single source of truth)** olarak kullanan bir operasyonel framework'tur. Tum altyapi ve uygulama konfigurasyonlari Git'te tutulur ve degisiklikler otomatik olarak cluster'a uygulanir.

## Temel Prensipler

### 1. Declarative Configuration
Tum sistem durumu deklaratif olarak tanimlanir:

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

### 2. Git as Single Source of Truth
Her degisiklik Git uzerinden yapilir. Manuel mudahale yoktur. Bu sayede:
- **Audit trail**: Kim, ne zaman, ne degistirdi?
- **Rollback**: Herhangi bir onceki duruma donmek `git revert` kadar kolay
- **Code review**: Altyapi degisiklikleri de PR sureci ile incelenir

### 3. Automated Reconciliation
Bir GitOps operatoru (ArgoCD, Flux) surekli olarak Git'teki istenen durum ile cluster'daki gercek durumu karsilastirir ve farkliliklari otomatik olarak giderir.

## Populer GitOps Araclari

| Arac | Ozellik | Kullanim Alani |
|------|---------|----------------|
| **ArgoCD** | Web UI, multi-cluster | Buyuk ekipler |
| **Flux** | Lightweight, Git-native | Kubernetes-first |
| **Jenkins X** | CI/CD entegrasyonu | End-to-end pipeline |

## ArgoCD ile Basit Bir Ornek

```bash
# ArgoCD kurulumu
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Application tanimlama
argocd app create my-app \
  --repo https://github.com/org/my-app.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production
```

## Neden GitOps?

1. **Guvenilirlik**: Manuel hatalari ortadan kaldirir
2. **Hiz**: Deployment surecleri otomatik ve hizli
3. **Geri donulebilirlik**: Her degisiklik izlenebilir ve geri alinabilir
4. **Guvenlik**: Cluster'a dogrudan erisim gerekmiyor
5. **Isbirligi**: Altyapi degisiklikleri de PR surecinden gecer

## Sonuc

GitOps, modern Kubernetes ortamlarinda altyapi yonetimini kolaylastiran guclu bir yaklasimdir. Ozellikle buyuyen ekipler ve karmasik altyapilara sahip organizasyonlar icin vazgecilmez bir pratiktir.

Bir sonraki yazida **ArgoCD ile multi-cluster GitOps** konusunu detayli inceleyecegiz.
