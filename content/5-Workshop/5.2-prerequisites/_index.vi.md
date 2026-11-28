---
title: "Điều kiện tiên quyết"
date: 2026-11-28
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Cần chuẩn bị

- AWS Region `ap-southeast-1` và quyền truy cập phù hợp.
- VPC, subnet, security group và EC2 instance.
- RDS PostgreSQL, S3, Cognito, SSM Parameter Store và Systems Manager.
- IAM instance profile cho EC2 và GitHub OIDC role.
- Repository, GitHub Actions variables và domain frontend.

## Bằng chứng cần chụp

- EC2 đang Running và Systems Manager đang Online.
- RDS đang Available.
- Các resource chính trong đúng Region.

Không chụp secret, access key, mật khẩu, token hoặc ARN đầy đủ.
