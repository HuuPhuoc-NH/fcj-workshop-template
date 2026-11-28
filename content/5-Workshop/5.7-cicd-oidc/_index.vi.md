---
title: "CI/CD và GitHub OIDC"
date: 2026-11-28
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Nội dung

GitHub Actions build và kiểm thử code. GitHub OIDC cho phép workflow assume IAM role tạm thời để upload artifact, gọi SSM và triển khai mà không lưu access key dài hạn.

## Bằng chứng cần chụp

- Workflow `Deploy frontend` hoặc workflow backend chạy thành công.
- OIDC provider và trust policy đã che account ID.
- IAM role và quyền giới hạn theo bucket/EC2.
- GitHub repository variables, chỉ chụp tên biến, không chụp secret.
