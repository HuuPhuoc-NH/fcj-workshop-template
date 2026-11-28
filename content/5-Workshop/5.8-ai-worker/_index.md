---
title: "AI worker and asynchronous processing"
date: 2026-11-28
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Scope

When a user requests AI-generated questions, the API creates a job and places it on a queue. The worker consumes the job, calls the AI service, stores the result in the database, and updates the status for the frontend.

## Evidence to capture

- The main queue and DLQ with only necessary names/metrics.
- The worker service running on EC2.
- A job moving from pending/running to completed.
- A failed job being retried or routed to the DLQ.
