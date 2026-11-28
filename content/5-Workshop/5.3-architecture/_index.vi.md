---
title: "Kiến trúc hệ thống"
date: 2026-11-28
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Luồng tổng thể

Người dùng truy cập frontend qua HTTPS. Frontend xác thực bằng Cognito và gọi backend API trên EC2. API đọc/ghi RDS PostgreSQL, đọc cấu hình từ SSM và gửi tác vụ AI vào hàng đợi để worker xử lý.

## Các thành phần

- VPC, subnet và security group.
- EC2 chạy API, worker và Nginx.
- RDS PostgreSQL lưu dữ liệu nghiệp vụ.
- Cognito quản lý đăng nhập và nhóm quyền.
- S3 lưu artifact frontend/backend.
- SSM hỗ trợ cấu hình và triển khai.

## Bằng chứng cần chụp

- Sơ đồ kiến trúc.
- VPC resource map hoặc các màn hình resource chính.
- Security group rules đã che thông tin nhạy cảm.
