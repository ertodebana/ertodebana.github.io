---
title: "Terraform vs Pulumi: IaC Araçlarının Karşılaştırması"
date: 2026-03-29 10:00:00 +0300
lang: tr
locale: tr-TR
page_id: terraform-vs-pulumi
permalink: /posts/terraform-vs-pulumi-iac-karsilastirmasi/
categories: [DevOps, Infrastructure]
tags: [terraform, pulumi, iac, infrastructure-as-code, cloud]
description: "Terraform ve Pulumi tartışmasını teori üzerinden değil, ekip yapısı ve günlük kullanım farkları üzerinden değerlendiriyorum."
---

## Infrastructure as Code Neden Önemli?

Terraform vs Pulumi karşılaştırması çoğu zaman "hangi araç daha iyi?" diye soruluyor. Bence daha doğru soru şu: ekibiniz hangi çalışma biçiminde daha az sürtünmeyle ilerliyor?

Altyapıyı kod olarak yönetmek, modern operasyon ekiplerinin temel yeteneklerinden biridir. Bu yaklaşım, tekrar edilebilirlik, gözden geçirilebilirlik ve otomasyon sağlar. Bugün en sık karşılaştırılan iki araç ise **Terraform** ve **Pulumi**.

Kısa cevabım şöyle: varsayılan seçim olarak Terraform hâlâ daha güvenli; ama ekip güçlü yazılım mühendisliği alışkanlıklarıyla çalışıyorsa Pulumi daha üretken olabilir.

## Genel Bakış

| Özellik | Terraform | Pulumi |
|---------|-----------|--------|
| **Dil** | HCL | TypeScript, Python, Go, C# |
| **State** | Local veya remote backend | Pulumi Cloud veya self-hosted |
| **Olgunluk** | Çok geniş ekosistem | Daha esnek programlama modeli |
| **Provider sayısı** | Çok yüksek | Terraform provider'larını da kullanabilir |
| **Öğrenme eğrisi** | HCL öğrenmek gerekir | Bildiğiniz dille ilerlersiniz |

## Terraform Örneği

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

## Pulumi Örneği

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

## Hangi Senaryoda Hangisi?

### Terraform daha uygundur, eğer:

- Ekibiniz HCL ve modül ekosistemine alışkınsa
- Çok sayıda provider ve olgun community desteği istiyorsanız
- Daha standart ve deklaratif bir deneyim tercih ediyorsanız

### Pulumi daha uygundur, eğer:

- Uygulama geliştiricilerinin bildiği dillerle ilerlemek istiyorsanız
- Koşullar, döngüler ve daha karmaşık altyapı mantıkları gerekiyorsa
- Test ve kod tekrarını daha güçlü kullanmak istiyorsanız

## OpenTofu Notu

HashiCorp lisans değişikliğinden sonra ortaya çıkan **OpenTofu**, Terraform'a yakın bir deneyim sunan açık kaynak alternatifidir:

```bash
# OpenTofu kurulumu
brew install opentofu

# Terraform ile benzer komutlar
tofu init
tofu plan
tofu apply
```

## Sonuç

İki araç da güçlü, ama aynı takım için her zaman eşit derecede uygun değiller. Ekibiniz standartlaşma, geniş provider desteği ve düşük sürpriz istiyorsa Terraform çoğu zaman daha sağlam seçimdir.

Pulumi ise "altyapı da sonuçta yazılım" diyen ekiplerde gerçekten hız kazandırabilir. Yine de bu esneklik, disiplin yoksa hızlıca karmaşıklığa dönüşür. Bu yüzden ben seçimi araç fanatizmiyle değil, ekip davranışıyla yapmayı daha doğru buluyorum.
