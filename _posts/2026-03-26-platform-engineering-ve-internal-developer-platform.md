---
title: "Platform Engineering ve Internal Developer Platform (IDP)"
date: 2026-03-26 10:00:00 +0300
categories: [Platform Engineering]
tags: [platform-engineering, idp, backstage, developer-experience, self-service]
description: "Platform Engineering nedir, IDP nasil kurulur ve developer experience nasil iyilestirilir?"
---

## Platform Engineering Nedir?

Platform Engineering, **developer'larin self-service olarak altyapi kaynaklarina erisebilecegi platformlari tasarlama ve insa etme** disiplinidir. Amac, cognitive load'u azaltmak ve developer productivity'yi artirmaktir.

## DevOps vs Platform Engineering

| | DevOps | Platform Engineering |
|---|--------|---------------------|
| **Odak** | Kultur ve pratikler | Urun ve platform |
| **Hedef** | Dev-Ops isbirligi | Self-service altyapi |
| **Cikti** | Pipeline'lar, surec | Internal Developer Platform |
| **Olcek** | Ekip bazli | Organizasyon bazli |

## Internal Developer Platform (IDP)

Bir IDP, developer'larin ihtiyac duydugu tum araclari ve servisleri tek bir noktadan sundugu bir platformdur:

### Temel Bilesenler

```
+---------------------------------------------------+
|              Developer Portal (UI)                 |
|              (Backstage, Port, etc.)               |
+---------------------------------------------------+
|                                                     |
|  +-------------+  +-----------+  +---------------+ |
|  | Service      |  | Infra     |  | Observability | |
|  | Catalog      |  | Provisioning | | Dashboard  | |
|  +-------------+  +-----------+  +---------------+ |
|                                                     |
|  +-------------+  +-----------+  +---------------+ |
|  | CI/CD        |  | Security  |  | Cost          | |
|  | Pipelines    |  | Scanning  |  | Management    | |
|  +-------------+  +-----------+  +---------------+ |
|                                                     |
+---------------------------------------------------+
|           Kubernetes / Cloud Infrastructure         |
+---------------------------------------------------+
```

## Backstage ile Developer Portal

Spotify tarafindan gelistirilen [Backstage](https://backstage.io), en populer IDP framework'udur:

```typescript
// catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: payment-service
  description: Payment processing microservice
  annotations:
    github.com/project-slug: org/payment-service
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: team-payments
  system: payment-platform
  dependsOn:
    - resource:payment-db
    - component:auth-service
```

## Golden Path Templates

Platform ekipleri, developer'larin yeni servisler olusturmasi icin **golden path template'leri** sunar:

```yaml
# template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: microservice-template
  title: Microservice Template
  description: Yeni bir microservice olustur
spec:
  parameters:
    - title: Servis Bilgileri
      properties:
        name:
          title: Servis Adi
          type: string
        owner:
          title: Sahip Ekip
          type: string
          ui:field: OwnerPicker
  steps:
    - id: create-repo
      name: Repository Olustur
      action: publish:github
    - id: deploy
      name: ArgoCD'ye Kaydet
      action: argocd:create-app
```

## Platform Engineering'e Baslarken

1. **Developer ihtiyaclarini anlayin**: Anket ve gorusmeler yapin
2. **MVP ile baslayin**: En cok ihtiyac duyulan self-service ozelliklerle
3. **Iteratif gelistirin**: Feedback loop olusturun
4. **Olcumleyin**: DORA metrics, developer satisfaction

## Sonuc

Platform Engineering, DevOps'un dogal evrimidir. Organizasyonlar buyudukce, merkezi bir platform ekibi olusturmak ve developer'lara self-service yetenekler sunmak kacinilmaz hale gelir.
