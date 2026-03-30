---
title: "GitOps Nedir ve Neden Kullanmalıyız?"
date: 2026-03-25 10:00:00 +0300
lang: tr
locale: tr-TR
page_id: gitops
permalink: /posts/gitops-nedir-ve-neden-kullanmaliyiz/
categories: [DevOps, GitOps]
tags: [gitops, argocd, flux, kubernetes, ci-cd]
description: "GitOps yaklaşımının temellerini, avantajlarını ve popüler araçlarını inceliyoruz."
---

## GitOps Nedir?

GitOps, **Git deposunu tek doğru kaynak (single source of truth)** olarak kullanan operasyonel bir yaklaşımdır. Altyapı ve uygulama tanımları Git'te tutulur, değişiklikler gözden geçirilir ve küme üzerindeki gerçek durum otomatik olarak bu tanımlarla hizalanır.

## Temel Prensipler

### 1. Deklaratif yapılandırma

Sistemin hedef durumu açık biçimde tanımlanır:

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

### 2. Git tek doğru kaynak olur

Her değişiklik Git üzerinden ilerler. Manuel müdahale en aza iner. Bunun sonucu olarak:

- **Audit trail**: Kim, ne zaman, neyi değiştirdi sorusu yanıtlanabilir.
- **Rollback**: Önceki duruma dönmek bir commit geri almak kadar kolaylaşır.
- **Code review**: Altyapı değişiklikleri de uygulama kodu gibi incelenir.

### 3. Otomatik uzlaştırma

Bir GitOps operatörü, Git'teki istenen durum ile kümedeki gerçek durumu sürekli karşılaştırır. Fark oluştuğunda sistemi tekrar hedef duruma getirir. Bu görev için en yaygın araçlar Argo CD ve Flux'tır.

## Popüler GitOps Araçları

| Araç | Öne çıkan özellik | Uygun senaryo |
|------|-------------------|---------------|
| **Argo CD** | Güçlü arayüz, çoklu küme desteği | Daha büyük ekipler |
| **Flux** | Daha hafif, Git merkezli yaklaşım | Kubernetes odaklı platformlar |
| **Jenkins X** | CI/CD ile daha sıkı entegrasyon | Uçtan uca teslimat zinciri |

## Argo CD ile Basit Bir Örnek

```bash
# Argo CD kurulumu
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Application tanımlama
argocd app create my-app \
  --repo https://github.com/org/my-app.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace production
```

## Neden GitOps?

1. **Güvenilirlik**: Manuel hataları azaltır.
2. **Hız**: Teslimat sürecini standartlaştırır ve otomatikleştirir.
3. **Geri alınabilirlik**: Her değişiklik kayıt altındadır.
4. **Güvenlik**: Doğrudan küme erişimi ihtiyacını azaltır.
5. **İş birliği**: Operasyonel değişiklikler de PR sürecinden geçer.

## Sonuç

GitOps, özellikle Kubernetes kullanan ekiplerde altyapı yönetimini daha öngörülebilir hâle getirir. Ekip büyüdükçe ve ortam karmaşıklaştıkça bu yaklaşımın değeri daha net görülür.
