---
title: "Backend and RDS"
date: 2026-11-28
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Scope

This section explains the .NET backend running on EC2, configuration loaded from SSM Parameter Store, and secure connectivity to RDS PostgreSQL. Migrations run through a deployment workflow with verification and rollback support.

## Evidence to capture

- RDS in the Available state with non-sensitive engine/version details.
- The SSM parameter path without exposing secret values.
- A successful Systems Manager Run Command for the backend.
- `/health` returning `200 OK`.
