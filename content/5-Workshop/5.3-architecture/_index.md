---
title: "System architecture"
date: 2026-11-28
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## End-to-end flow

Users access the frontend over HTTPS. The frontend authenticates through Cognito and calls the backend API on EC2. The API reads and writes PostgreSQL on RDS, reads configuration from SSM, and sends AI jobs to a queue for worker processing.

## Main components

- VPC, subnets, and security groups.
- EC2 running the API, worker, and Nginx.
- RDS PostgreSQL for application data.
- Cognito for login and application groups.
- S3 for frontend and backend artifacts.
- SSM for configuration and deployment.

## Evidence to capture

- The architecture diagram.
- A VPC resource map or the main resource screens.
- Security-group rules with sensitive information redacted.
