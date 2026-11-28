---
title: "5.4 Backend và RDS"
date: 2026-11-28
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Nội dung

Trình bày cách backend .NET chạy trên EC2, lấy cấu hình từ SSM Parameter Store và kết nối an toàn đến RDS PostgreSQL. Migration được chạy trong quy trình triển khai có kiểm tra và rollback.

## Bằng chứng cần chụp

- RDS instance Available và thông tin engine/version không nhạy cảm.
- SSM parameter path, chỉ chụp tên path, không chụp giá trị secret.
- Systems Manager Run Command triển khai backend thành công.
- `/health` trả `200 OK`.
