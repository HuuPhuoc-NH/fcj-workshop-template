---
title: "5.5 Cognito và quản lý người dùng"
date: 2026-11-28
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Nội dung

Cognito cung cấp đăng nhập và group `admin`, `head`, `teacher`. Backend xác minh access token, đọc hồ sơ nội bộ từ database và kiểm tra quyền theo role.

## Bằng chứng cần chụp

- User Pool và Managed Login domain, che ID nếu cần.
- Các group ứng dụng.
- Tài khoản test đã xác thực.
- API `/api/v1/me` trả `200 OK` sau đăng nhập.
- Màn hình đối soát Cognito thành công.
