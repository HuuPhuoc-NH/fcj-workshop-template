---
title: "5.10 Dọn dẹp và bài học"
date: 2026-11-28
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

## Dọn dẹp

Kiểm tra artifact cũ, log, alarm và tài nguyên thử nghiệm. Không xóa tài nguyên production hoặc dữ liệu thật khi chưa có xác nhận của nhóm.

## Bài học

- Thiết kế quyền tối thiểu và không dùng access key dài hạn.
- Tách deploy, kiểm thử, quan sát và rollback thành các bước có thể xác minh.
- Chuẩn bị phương án fallback khi một dịch vụ chưa thể sử dụng.

## Bằng chứng cần chụp

- Danh sách tài nguyên sau kiểm tra.
- Lifecycle hoặc chính sách lưu trữ artifact.
- Ghi chú các tài nguyên được giữ lại và lý do.
