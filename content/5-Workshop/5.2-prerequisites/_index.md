---
title: "Prerequisites"
date: 2026-11-28
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Required resources

- AWS Region `ap-southeast-1` and appropriate permissions.
- A VPC, subnets, security groups, and an EC2 instance.
- RDS PostgreSQL, S3, Cognito, SSM Parameter Store, and Systems Manager.
- An EC2 instance profile and a GitHub OIDC role.
- The repository, GitHub Actions variables, and the frontend domain.

## Evidence to capture

- EC2 in the Running state and Systems Manager Online.
- RDS in the Available state.
- The main resources in the correct Region.

Do not capture secrets, access keys, passwords, tokens, or full ARNs.
