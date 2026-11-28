---
title: "Frontend and CloudFront fallback"
date: 2026-11-28
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Deployment approach

When CloudFront is unavailable, GitHub Actions builds the frontend, packages a release, and uploads the artifact to S3. SSM instructs EC2 to download the artifact, extract a new release, update the `current` symlink, and reload Nginx.

## Evidence to capture

- The release artifact in S3.
- A successful SSM command.
- The release/current layout on EC2 if it can be shown safely.
- The frontend and a fingerprinted asset loading successfully.
