---
title: "Testing, observability, and rollback"
date: 2026-11-28
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Scope

Verify health endpoints, Cognito authentication, user creation, frontend assets, logs, and CloudWatch alarms. If deployment fails, the previous release remains available for rollback.

## Evidence to capture

- Green GitHub checks.
- Successful `/health` and `/api/v1/me` responses.
- CloudWatch logs, metrics, and alarms with emails and sensitive identifiers redacted.
- The release list and rollback result if tested.
