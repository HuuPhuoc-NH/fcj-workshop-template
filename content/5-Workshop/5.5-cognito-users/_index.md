---
title: "5.5 Cognito and user management"
date: 2026-11-28
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Scope

Cognito provides login and the `admin`, `head`, and `teacher` groups. The backend validates access tokens, reads the internal profile from the database, and enforces role-based access.

## Evidence to capture

- The User Pool and Managed Login domain, with IDs redacted where appropriate.
- The application groups.
- A verified test account.
- `/api/v1/me` returning `200 OK` after login.
- A successful Cognito reconciliation screen.
