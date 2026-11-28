---
title: "Bản đề xuất"
date: 2026-11-28
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI Exam Maker
## Nền tảng AWS tối ưu chi phí cho việc soạn và duyệt đề thi có hỗ trợ AI

### 1. Tóm tắt điều hành

AI Exam Maker là ứng dụng web dành cho giáo viên trung học phổ thông Việt Nam. Ứng dụng hỗ trợ tạo câu hỏi nháp bằng AI, rà soát câu hỏi, ghép thành các phiên bản đề và xuất đề học sinh cùng đáp án riêng. Hệ thống được thiết kế như một môi trường trình diễn có kiểm soát trên AWS, với xử lý AI bất đồng bộ, quy trình duyệt theo tổ bộ môn, khả năng truy vết và mục tiêu nằm trong khoản tín dụng 200 USD.

Phần đóng góp của em tập trung vào nền tảng và con đường triển khai: chốt phương án AWS, chuẩn bị ranh giới mạng và bảo mật, vận hành ứng dụng trên EC2, cấu hình domain và HTTPS, thiết lập GitHub OIDC, tự động triển khai frontend. Em cũng phối hợp với nhóm frontend để kiểm tra luồng đăng nhập thật.

### 2. Mục tiêu dự án

#### 2.1. Mục tiêu chung

Xây dựng một nền tảng đề thi chạy được trên AWS, trong đó giáo viên có thể đăng nhập, yêu cầu AI tạo câu hỏi, rà soát nội dung, gửi duyệt, ghép phiên bản đề và xuất tài liệu cuối theo quy trình có kiểm soát.

#### 2.2. Mục tiêu hạ tầng

- Chọn phương án triển khai phù hợp với thời gian hai tuần và khoản tín dụng 200 USD.
- Giữ EC2 có thể phục vụ qua HTTPS nhưng không cần mở SSH từ Internet.
- Tách compute công khai khỏi database riêng tư bằng các security group chỉ cho phép lưu lượng cần thiết.
- Dùng IAM role, Systems Manager Session Manager và GitHub OIDC thay cho access key dài hạn.
- Tạo quy trình phát hành frontend có thể lặp lại và giữ artifact để rollback.
- Kiểm tra callback URL và logout URL của Cognito với domain thật.

#### 2.3. Tiêu chí hoàn thành

- VPC, subnet, route, security group, EC2, EBS và Elastic IP hỗ trợ ứng dụng mà không cần NAT Gateway.
- Nginx phục vụ frontend tĩnh qua HTTPS và chuyển tiếp request API đến backend .NET.
- Có thể quản trị EC2 bằng Session Manager mà không mở port 22.
- GitHub Actions xác thực qua OIDC, upload artifact và gọi triển khai qua SSM.
- Nếu triển khai frontend lỗi, release trước vẫn còn để rollback.
- Người dùng hoàn tất luồng đăng nhập Cognito và quay lại ứng dụng bằng callback URL đã cấu hình.

### 3. Vấn đề cần giải quyết

#### 3.1. Không thể chỉ phụ thuộc vào một đường triển khai

Thiết kế ban đầu dự kiến dùng CDN để phân phối frontend. Khi CloudFront chưa dùng được trong môi trường làm việc, dự án cần một phương án thay thế thực tế nhưng vẫn phục vụ frontend an toàn và có thể vận hành bởi một nhóm nhỏ.

#### 3.2. Hạ tầng cần có khả năng lặp lại

Tạo tài nguyên thủ công hoàn toàn trên AWS Console khiến việc review và dựng lại môi trường khó khăn. Vì vậy dự án cần thư mục hạ tầng và runbook được ghi lại, đồng thời không commit account ID, ARN thật, secret hay giá trị phụ thuộc môi trường.

#### 3.3. Quản trị server trực tiếp làm tăng rủi ro

Mở SSH và lưu credential tĩnh tạo thêm bề mặt tấn công. Nền tảng sử dụng instance profile và Session Manager để quản trị, còn GitHub Actions dùng credential ngắn hạn qua OIDC để triển khai.

#### 3.4. Release frontend cần có đường khôi phục

Một lần deploy có thể lỗi trong khi hệ thống vẫn đang phục vụ người dùng. Mỗi release cần có định danh, được kiểm tra trước khi kích hoạt và được giữ cùng release trước đó để server có thể quay về phiên bản ổn định.

### 4. Kiến trúc giải pháp

AI Exam Maker chạy tại Region Singapore. Public subnet chứa EC2 chạy Nginx, frontend tĩnh, .NET API và AI worker. RDS PostgreSQL đặt trong private subnet. SQS tách rời các job AI, còn Cognito, Parameter Store, CloudWatch, SNS và Budgets hỗ trợ định danh, cấu hình, vận hành và kiểm soát chi phí.

![Kiến trúc AWS của AI Exam Maker](/fcj-workshop-template/images/2-Proposal/project-architecture.jpg)

#### 4.1. Các thành phần chính

- **VPC và security group**: public subnet cho EC2, private subnet cho RDS, không dùng NAT Gateway và chỉ mở các rule cần thiết.
- **EC2 và EBS**: máy ARM có Elastic IP, chạy Nginx, .NET API và worker bằng systemd.
- **Domain và HTTPS**: domain ứng dụng trỏ đến điểm vào công khai; Nginx kết thúc HTTPS và chuyển tiếp traffic API.
- **Amazon RDS for PostgreSQL**: chỉ security group của EC2 được phép kết nối đến database riêng tư.
- **Amazon Cognito**: đăng nhập và các group role hỗ trợ teacher, head và admin.
- **Amazon SQS và Amazon Bedrock**: API đưa job vào queue và worker xử lý bất đồng bộ.
- **Amazon S3 và AWS Systems Manager**: S3 lưu gói release; SSM Run Command khởi chạy triển khai trên EC2.
- **GitHub Actions và IAM OIDC**: workflow nhận credential tạm thời mà không lưu access key.
- **CloudWatch, SNS và AWS Budgets**: log, alarm, thông báo và giới hạn chi phí cho môi trường trình diễn.

#### 4.2. Luồng phân phối frontend

GitHub Actions build và kiểm tra bản xuất tĩnh Next.js. Workflow đóng gói release kèm checksum rồi upload vào prefix S3 của dự án. Sau đó workflow gọi SSM. Script trên EC2 tải artifact, kiểm checksum, giải nén vào thư mục release mới, cập nhật symlink `current`, reload Nginx và giữ release trước để rollback. Khi CloudFront có thể sử dụng, thiết kế có thể quay lại CDN; luồng S3–SSM–EC2 là phương án fallback đang dùng.

#### 4.3. Nguyên tắc bảo mật

- Không cần inbound SSH; quản trị bằng Session Manager.
- EC2 và RDS liên lạc bằng tham chiếu security group thay vì mở mạng rộng.
- GitHub nhận credential role tạm thời qua OIDC.
- Secret và định danh phụ thuộc môi trường nằm ngoài repository.
- Backend là nơi quyết định cuối cùng về role, tổ bộ môn, quyền sở hữu và trạng thái nội dung.

### 5. Lộ trình triển khai

Việc triển khai bám theo roadmap của nhóm từ ngày 17/11 đến ngày 01/12/2026.

- **Chốt nền**: thống nhất quyết định AWS, ghi lại phương án hạ tầng và chuẩn bị ranh giới repository.
- **Dựng nền tảng**: tạo VPC, security group, EC2, EBS, Elastic IP, Session Manager, domain và HTTPS.
- **Tự động hóa phát hành**: cấu hình OIDC, deploy role, luồng artifact S3, SSM Run Command và rollback frontend.
- **Tích hợp và kiểm tra**: kiểm tra Cognito với nhóm frontend, kiểm tra health/deploy và chuẩn bị minh chứng cho demo.

### 6. Ước tính ngân sách

Kiến trúc được giữ gọn để dự án nằm trong khoản tín dụng phát triển 200 USD. Các khoản chính gồm thời gian chạy EC2, EBS, public IPv4, thời gian và dung lượng RDS, mức dùng model AI, log và truyền dữ liệu. S3, SQS, Cognito, Parameter Store, SNS và Systems Manager được giữ ở mức sử dụng thấp phù hợp với môi trường trình diễn.

Các biện pháp kiểm soát chi phí gồm dừng EC2 và RDS ngoài giờ làm việc, dùng model rẻ hơn trong giai đoạn phát triển, giới hạn số request và token AI, chỉ giữ log và artifact cần thiết, đồng thời bật cảnh báo AWS Budgets ở 50%, 80% và 100%.

### 7. Đánh giá rủi ro và phương án giảm thiểu

| Rủi ro | Ảnh hưởng | Phương án giảm thiểu |
|---|---|---|
| CloudFront chưa dùng được trong môi trường | Trung bình | Phục vụ frontend tĩnh bằng Nginx trên EC2, dùng S3 và SSM để phát hành. |
| Deploy lỗi sau khi upload artifact | Cao | Kiểm checksum và health; giữ release trước để rollback. |
| EC2 hoặc RDS không khả dụng | Cao | Dùng systemd restart, backup RDS, thử restore và ghi rõ trade-off của môi trường demo. |
| Security group mở quá rộng | Cao | Đóng SSH, giới hạn RDS theo security group của EC2 và review port/prefix list cần dùng. |
| Lộ credential AWS dài hạn | Cao | Dùng instance profile và GitHub OIDC với quyền tối thiểu. |
| Chi phí AI vượt tín dụng | Cao | Giới hạn quota và token, bật budget alert, dừng tài nguyên ngoài giờ. |
| Callback Cognito cấu hình sai | Trung bình | Kiểm tra end-to-end đăng nhập, callback, logout và truy cập API bảo vệ bằng domain thật. |

### 8. Kết luận và kết quả kỳ vọng

AI Exam Maker kết hợp việc soạn câu hỏi có hỗ trợ AI với quy trình rà soát, duyệt và phát hành đề có thể truy vết. Phần hạ tầng của em tạo ra con đường an toàn và có thể lặp lại từ một commit GitHub đến frontend đang chạy, đồng thời vẫn có phương án dự phòng khi CloudFront chưa dùng được. Kết quả kỳ vọng là môi trường AWS sẵn sàng cho demo với ranh giới mạng rõ ràng, HTTPS, quản trị bằng Session Manager, triển khai qua OIDC, log/alarm, rollback và kiểm soát chi phí.

Về sau, kiến trúc có thể mở rộng thêm load balancer, database Multi-AZ, CDN, nhập tài liệu phong phú hơn và mức sẵn sàng cao hơn khi dự án chuyển khỏi phạm vi trình diễn.
