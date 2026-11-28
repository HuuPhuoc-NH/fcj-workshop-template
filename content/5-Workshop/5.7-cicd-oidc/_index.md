---
title: "5.7 CI/CD and GitHub OIDC"
date: 2026-11-28
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Scope

GitHub Actions builds and tests the code. GitHub OIDC lets a workflow assume a temporary IAM role to upload artifacts, call SSM, and deploy without storing long-lived access keys.

## Evidence to capture

- A successful `Deploy frontend` or backend workflow.
- The OIDC provider and trust policy with the account ID redacted.
- The IAM role and least-privilege permissions.
- GitHub repository variables by name only; never capture secret values.
