---
title: "Platform Engineering and Internal Developer Platforms"
date: 2026-03-26 10:00:00 +0300
lang: en
locale: en-US
page_id: platform-engineering-idp
permalink: /posts/platform-engineering-and-internal-developer-platforms/
categories: [Platform Engineering]
tags: [platform-engineering, idp, backstage, developer-experience, self-service]
description: "A more grounded look at platform engineering as an internal product, not just another layer of process."
---

## What Is Platform Engineering?

Platform Engineering has become one of those terms that sounds impressive even when nobody means the same thing by it. My practical definition is simpler: reduce the number of times developers need to wait on another team for routine infrastructure work.

Platform Engineering is the discipline of designing and operating platforms that let developers access infrastructure through self-service workflows. The goal is to reduce cognitive load, improve consistency, and help teams ship faster.

## DevOps vs Platform Engineering

| | DevOps | Platform Engineering |
|---|--------|---------------------|
| **Focus** | Culture and practice | Product and platform |
| **Goal** | Dev/Ops collaboration | Self-service infrastructure |
| **Output** | Pipelines and processes | Internal Developer Platform |
| **Scale** | Team level | Organization level |

## What an Internal Developer Platform Contains

An IDP brings the tools developers need into one coherent experience:

### Core building blocks

```
+---------------------------------------------------+
|              Developer Portal (UI)                |
|              (Backstage, Port, etc.)              |
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

## Building the Portal with Backstage

[Backstage](https://backstage.io), originally created at Spotify, is one of the most common ways to build an internal developer portal:

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

Platform teams can make the preferred path the easiest path by shipping templates for new services:

```yaml
# template.yaml
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: microservice-template
  title: Microservice Template
  description: Create a new microservice
spec:
  parameters:
    - title: Service Information
      properties:
        name:
          title: Service Name
          type: string
        owner:
          title: Owning Team
          type: string
          ui:field: OwnerPicker
  steps:
    - id: create-repo
      name: Create Repository
      action: publish:github
    - id: deploy
      name: Register in Argo CD
      action: argocd:create-app
```

## How to Start

1. **Understand developer pain points** before designing the platform.
2. **Start with an MVP** that removes the highest-friction tasks.
3. **Iterate like a product team** instead of shipping one-off platform projects.
4. **Measure outcomes** using DORA metrics and developer satisfaction signals.

## Conclusion

Platform Engineering works when it behaves like a product discipline, not a platform-flavored ticket queue. A shiny portal is irrelevant if developers still need side-channel help for every real task.

Done well, it does not replace DevOps. It scales it by turning common workflows, guardrails, and infrastructure patterns into a reusable internal product that people actually want to use.
