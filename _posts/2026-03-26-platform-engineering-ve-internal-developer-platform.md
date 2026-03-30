---
title: "Platform Engineering ve Internal Developer Platform (IDP)"
date: 2026-03-26 10:00:00 +0300
lang: tr
locale: tr-TR
page_id: platform-engineering-idp
permalink: /posts/platform-engineering-ve-internal-developer-platform/
categories: [Platform Engineering]
tags: [platform-engineering, idp, backstage, developer-experience, self-service]
description: "Platform Engineering nedir, IDP nasıl kurulur ve developer experience nasıl iyileştirilir?"
---

## Platform Engineering Nedir?

Platform Engineering, geliştiricilerin self-service olarak altyapı kaynaklarına erişebildiği platformları tasarlama ve işletme disiplinidir. Temel amaç, ekiplerin bilişsel yükünü azaltmak ve teslimat hızını artırmaktır.

## DevOps ve Platform Engineering Arasındaki Fark

| | DevOps | Platform Engineering |
|---|--------|---------------------|
| **Odak** | Kültür ve pratikler | Ürün ve platform |
| **Hedef** | Dev ve Ops iş birliği | Self-service altyapı |
| **Çıktı** | Pipeline'lar ve süreçler | Internal Developer Platform |
| **Ölçek** | Takım düzeyi | Organizasyon düzeyi |

## Internal Developer Platform (IDP)

Bir IDP, geliştiricilerin ihtiyaç duyduğu araçları tek bir deneyim altında toplayan platformdur:

### Temel bileşenler

```
+---------------------------------------------------+
|              Developer Portal (UI)                |
|              (Backstage, Port, vb.)               |
+---------------------------------------------------+
|                                                   |
|  +-------------+  +-------------+  +------------+ |
|  | Service     |  | Infra       |  | Visibility | |
|  | Catalog     |  | Provision   |  | Dashboard  | |
|  +-------------+  +-------------+  +------------+ |
|                                                   |
|  +-------------+  +-------------+  +------------+ |
|  | CI/CD       |  | Security    |  | Cost       | |
|  | Pipelines   |  | Controls    |  | Insights   | |
|  +-------------+  +-------------+  +------------+ |
|                                                   |
+---------------------------------------------------+
|          Kubernetes / Cloud Infrastructure        |
+---------------------------------------------------+
```

## Backstage ile Geliştirici Portalı

Spotify tarafından geliştirilen [Backstage](https://backstage.io), en yaygın IDP çatıları arasında yer alır:

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

## Golden Path Şablonları

Platform ekipleri, yeni servislerin güvenli ve standart biçimde açılabilmesi için golden path şablonları sağlar:

```yaml
# template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: microservice-template
  title: Microservice Template
  description: Yeni bir microservice oluştur
spec:
  parameters:
    - title: Servis Bilgileri
      properties:
        name:
          title: Servis Adı
          type: string
        owner:
          title: Sahip Ekip
          type: string
          ui:field: OwnerPicker
  steps:
    - id: create-repo
      name: Repository Oluştur
      action: publish:github
    - id: deploy
      name: ArgoCD'ye Kaydet
      action: argocd:create-app
```

## Nereden Başlamalı?

1. **Geliştirici ihtiyaçlarını anlayın**: Görüşme ve anketlerle darboğazları bulun.
2. **MVP ile başlayın**: En çok tekrarlanan işleri self-service hâle getirin.
3. **İteratif ilerleyin**: Platformu bir iç ürün gibi yönetin.
4. **Ölçümleyin**: DORA metrikleri, teslimat süresi ve geliştirici memnuniyetini takip edin.

## Sonuç

Platform Engineering, DevOps'un yerine geçen değil, onu ölçekleyen bir yaklaşımdır. Organizasyon büyüdükçe ortak standartları, güvenliği ve geliştirici deneyimini merkezi bir platform üzerinden sunmak daha değerli hâle gelir.
