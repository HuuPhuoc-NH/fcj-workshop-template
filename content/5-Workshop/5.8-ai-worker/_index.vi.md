---
title: "5.8 AI worker và xử lý bất đồng bộ"
date: 2026-11-28
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Nội dung

Khi người dùng yêu cầu tạo đề bằng AI, API tạo job và đưa vào hàng đợi. Worker lấy job, gọi dịch vụ AI, lưu kết quả vào database và cập nhật trạng thái để frontend theo dõi.

## Bằng chứng cần chụp

- Queue chính và DLQ, chỉ chụp tên/metrics cần thiết.
- Worker service đang chạy trên EC2.
- Job chuyển từ pending/running sang completed.
- Trường hợp lỗi được retry hoặc đưa vào DLQ.
