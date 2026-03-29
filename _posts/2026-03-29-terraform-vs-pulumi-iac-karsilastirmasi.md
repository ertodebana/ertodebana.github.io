---
title: "Terraform vs Pulumi: IaC Araclarinin Karsilastirmasi"
date: 2026-03-29 10:00:00 +0300
categories: [DevOps, Infrastructure]
tags: [terraform, pulumi, iac, infrastructure-as-code, cloud]
description: "Terraform ve Pulumi'yi karsilastirarak hangi IaC aracinin hangi senaryoda daha uygun oldugunu inceliyoruz."
---

## Infrastructure as Code (IaC) Neden Onemli?

Altyapiyi kod olarak yonetmek, modern DevOps'un temel taslarindan biridir. Bu yazida, en populer iki IaC aracini -- **Terraform** ve **Pulumi** -- karsilastiriyoruz.

## Genel Bakis

| Ozellik | Terraform | Pulumi |
|---------|-----------|--------|
| **Dil** | HCL (kendi DSL'i) | Python, TypeScript, Go, C# |
| **State** | Remote/Local | Pulumi Cloud / Self-hosted |
| **Olgunluk** | 2014'ten beri | 2018'den beri |
| **Provider Sayisi** | 3000+ | Terraform provider'larini da kullanabilir |
| **Lisans** | BSL (1.6+) / OpenTofu (MPL) | Apache 2.0 |
| **Ogrenme Egrisi** | HCL ogrenmek gerekir | Bildigi dili kullanabilir |

## Terraform Ornegi

```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket = "my-terraform-state"
    key    = "production/terraform.tfstate"
    region = "eu-west-1"
  }
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags = {
    Name        = "production-vpc"
    Environment = "production"
  }
}

resource "aws_subnet" "public" {
  count             = 3
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index)
  availability_zone = data.aws_availability_zones.available.names[count.index]

  tags = {
    Name = "public-subnet-${count.index + 1}"
    Type = "public"
  }
}

resource "aws_eks_cluster" "main" {
  name     = "production-cluster"
  role_arn = aws_iam_role.eks.arn

  vpc_config {
    subnet_ids = aws_subnet.public[*].id
  }
}
```

## Pulumi Ornegi (TypeScript)

```typescript
import * as aws from "@pulumi/aws";
import * as pulumi from "@pulumi/pulumi";

const vpc = new aws.ec2.Vpc("main", {
  cidrBlock: "10.0.0.0/16",
  enableDnsHostnames: true,
  tags: {
    Name: "production-vpc",
    Environment: "production",
  },
});

const azs = aws.getAvailabilityZones({ state: "available" });

const subnets = azs.then((az) =>
  az.names.slice(0, 3).map(
    (azName, index) =>
      new aws.ec2.Subnet(`public-${index}`, {
        vpcId: vpc.id,
        cidrBlock: `10.0.${index}.0/24`,
        availabilityZone: azName,
        tags: {
          Name: `public-subnet-${index + 1}`,
          Type: "public",
        },
      })
  )
);

const cluster = new aws.eks.Cluster("main", {
  name: "production-cluster",
  roleArn: eksRole.arn,
  vpcConfig: {
    subnetIds: subnets.then((s) => s.map((subnet) => subnet.id)),
  },
});

export const clusterName = cluster.name;
export const kubeconfig = cluster.kubeconfig;
```

## Ne Zaman Hangisini Secmeli?

### Terraform Secin Eger:
- Ekip HCL'ye zaten asina ise
- Cok sayida provider kullaniyorsaniz
- Stabil, battle-tested bir cozum istiyorsaniz
- Community ve dokumantasyon oncelikliyse

### Pulumi Secin Eger:
- Ekip belirli bir programlama diline hakimse
- Karmasik logic gerektiren altyapi varsa (loops, conditions)
- Testing onemli ise (unit test yazabilirsiniz)
- Monorepo yaklasimi kullaniyorsaniz

## OpenTofu: Ucuncu Secenek

HashiCorp'un lisans degisikliginden sonra ortaya cikan **OpenTofu**, Terraform'un acik kaynakli fork'udur:

```bash
# OpenTofu kurulumu
brew install opentofu

# Terraform ile ayni komutlar
tofu init
tofu plan
tofu apply
```

## Sonuc

Her iki arac da guclu ve olgun cozumlerdir. Secim, ekibinizin yeteneklerine, projenizin karmasikligina ve organizasyonunuzun ihtiyaclarina bagli olmalidir. Kucuk-orta projeler icin Terraform'un sadeligi, buyuk ve karmasik altyapilar icin Pulumi'nin esnekligi one cikabilir.
