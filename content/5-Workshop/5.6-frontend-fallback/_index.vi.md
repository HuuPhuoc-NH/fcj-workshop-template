---
title: "Frontend và phương án CloudFront fallback"
date: 2026-11-28
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Phương án triển khai

Khi CloudFront chưa dùng được, GitHub Actions build frontend, đóng gói release và upload artifact vào S3. SSM gửi lệnh cho EC2 tải artifact, giải nén vào release mới, cập nhật symlink `current` và reload Nginx.

## Bằng chứng cần chụp

- Artifact release trong S3.
- SSM command thành công.
- Thư mục release/current trên EC2 nếu chụp được an toàn.
- Website frontend và asset fingerprint tải thành công.
