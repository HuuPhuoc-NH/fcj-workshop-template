---
title: "Cleanup and lessons learned"
date: 2026-11-28
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Cleanup

Review old artifacts, logs, alarms, and test resources. Do not delete production resources or real data without team approval.

## Lessons learned

- Use least-privilege permissions and avoid long-lived access keys.
- Separate deployment, testing, observability, and rollback into verifiable steps.
- Prepare a fallback when a service is not yet available.

## Evidence to capture

- The resource list after review.
- Artifact lifecycle or retention policies.
- Notes describing retained resources and the reasons for keeping them.
