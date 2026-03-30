---
title: "Terraform vs Pulumi: Comparing IaC Tools"
date: 2026-03-29 10:00:00 +0300
lang: en
locale: en-US
page_id: terraform-vs-pulumi
permalink: /posts/terraform-vs-pulumi-comparing-iac-tools/
categories: [DevOps, Infrastructure]
tags: [terraform, pulumi, iac, infrastructure-as-code, cloud]
description: "A more opinionated comparison of Terraform and Pulumi based on team habits, not just feature tables."
---

## Why Infrastructure as Code Matters

The usual Terraform vs Pulumi debate asks which tool is better. I think the better question is which tool creates less friction for the way your team already works.

Infrastructure as Code gives teams repeatability, reviewability, and automation. Instead of relying on manual changes, infrastructure can be described, versioned, and promoted through the same delivery workflow as application code. Two of the most common choices are **Terraform** and **Pulumi**.

My short version is this: Terraform is still the safer default for most teams, while Pulumi becomes compelling when the team has strong software engineering habits and actually benefits from code-level abstraction.

## High-Level Comparison

| Feature | Terraform | Pulumi |
|---------|-----------|--------|
| **Language** | HCL | TypeScript, Python, Go, C# |
| **State** | Local or remote backend | Pulumi Cloud or self-hosted |
| **Maturity** | Very large ecosystem | More flexible programming model |
| **Provider coverage** | Extremely broad | Can also use Terraform providers |
| **Learning curve** | Requires learning HCL | Lets teams use familiar languages |

## Terraform Example

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

## Pulumi Example

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

## When to Choose Which

### Terraform is usually the better fit when:

- Your team already works comfortably with HCL
- You want a very mature ecosystem and broad provider coverage
- You prefer a more standardized declarative workflow

### Pulumi is usually the better fit when:

- Your team wants to stay in a familiar programming language
- Infrastructure logic needs loops, abstractions, or richer reuse
- You want testing patterns that feel closer to application development

## A Note on OpenTofu

After HashiCorp's licensing change, **OpenTofu** emerged as an open-source alternative with a Terraform-like experience:

```bash
# Install OpenTofu
brew install opentofu

# Similar workflow
tofu init
tofu plan
tofu apply
```

## Conclusion

Both tools are good, but they are not equally good for every team shape. If you want standardization, broad ecosystem depth, and fewer surprises in hiring or maintenance, Terraform remains the safer default.

Pulumi can absolutely be the better choice when infrastructure logic is complex and the team is disciplined enough to use that flexibility well. Without that discipline, though, the extra power can become extra mess. That is why I treat this less as a feature comparison and more as a team-design decision.
