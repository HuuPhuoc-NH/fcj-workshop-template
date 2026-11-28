---
title: "5.9 Kiểm thử, quan sát và rollback"
date: 2026-11-28
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Nội dung

Kiểm tra health endpoint, xác thực Cognito, luồng tạo user, frontend asset, log và cảnh báo CloudWatch. Khi deploy lỗi, release trước được giữ lại để rollback.

## Bằng chứng cần chụp

- GitHub checks xanh.
- `/health` và `/api/v1/me` thành công.
- CloudWatch log/metric/alarm, che email và định danh nhạy cảm.
- Danh sách release và kết quả rollback nếu đã kiểm thử.
