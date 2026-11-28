---
title: "Bản đề xuất"
date: 2026-11-28
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# AI Exam Maker
## Kiến trúc AWS tối ưu chi phí cho việc tạo và duyệt câu hỏi có hỗ trợ AI

### 1. Tóm tắt điều hành

AI Exam Maker là ứng dụng web hỗ trợ giáo viên trung học phổ thông Việt Nam trong quá trình chuẩn bị đề thi. Giáo viên chọn môn, khối, chương, dạng câu hỏi, mức độ nhận thức và có thể thêm ghi chú ngắn. Ứng dụng gửi yêu cầu đến pipeline AI bất đồng bộ, kiểm tra kết quả sinh ra và lưu câu hỏi ở trạng thái nháp. Giáo viên phải rà soát nội dung trước khi câu hỏi đi vào quy trình duyệt theo tổ bộ môn. Các câu hỏi đã được duyệt có thể tái sử dụng để ghép thành phiên bản đề, xuất bản phiên bản chính thức và xuất đề học sinh cùng một tài liệu đáp án riêng.

Dự án được thiết kế như một môi trường trình diễn AWS có phạm vi rõ ràng. Mục tiêu là chứng minh được quy trình hoàn chỉnh và có thể truy vết thay vì đạt mức sẵn sàng production. Nhóm đặt mục tiêu giữ môi trường phát triển và demo trong khoản tín dụng 200 USD, đồng thời thể hiện được xác thực, phân quyền, xử lý bất đồng bộ, tự động triển khai, monitoring, rollback và kiểm soát chi phí.

Phần việc của em thuộc phạm vi hạ tầng OPS-1. Nội dung gồm chốt các quyết định AWS, xây dựng ranh giới IaC và runbook, VPC và security group, EC2 và EBS, Elastic IP, Session Manager, domain và HTTPS, thiết kế CloudFront cùng phương án fallback S3–SSM–EC2, GitHub OIDC, CI/CD frontend và phối hợp với nhóm frontend kiểm tra luồng đăng nhập end-to-end.

### 2. Mục tiêu dự án

#### 2.1. Mục tiêu chung

Xây dựng một nền tảng đề thi chạy được trên AWS, cho phép giáo viên đăng nhập, tạo câu hỏi nháp bằng AI, rà soát và gửi duyệt, nhận phê duyệt theo tổ bộ môn, ghép phiên bản đề, xuất bản phiên bản được duyệt và xuất các tài liệu cuối.

#### 2.2. Phạm vi chức năng

- Đăng nhập qua Amazon Cognito bằng Authorization Code với PKCE.
- Hỗ trợ ba vai trò: teacher, department head và administrator.
- Tạo câu hỏi nháp theo môn, khối, chương, dạng câu hỏi và mức độ nhận thức.
- Xử lý yêu cầu AI qua Amazon SQS và worker riêng thay vì chặn giao diện người dùng.
- Kiểm tra kết quả AI, theo dõi trạng thái job và yêu cầu giáo viên review trước khi duyệt.
- Duy trì question bank với các trạng thái Draft, Submitted, Approved, Rejected và Archived.
- Ghép đề dưới dạng các phiên bản bất biến, tạo snapshot và content hash khi submit.
- Cho phép trưởng bộ môn duyệt nội dung do thành viên khác trong cùng tổ tạo ra.
- Mỗi đề chỉ có tối đa một phiên bản chính thức được publish tại một thời điểm.
- Xuất đề học sinh không chứa đáp án và xuất tài liệu đáp án bằng endpoint được bảo vệ riêng.
- Ghi audit cho các thay đổi trạng thái và cung cấp log vận hành cùng thông tin sử dụng AI/chi phí.

#### 2.3. Mục tiêu hạ tầng

- Giữ điểm truy cập công khai trên HTTPS và không cần inbound SSH.
- Đặt EC2 trong public subnet, RDS trong private subnet và không dùng NAT Gateway.
- Giới hạn quyền truy cập database chỉ từ security group của EC2.
- Dùng EC2 instance profile và Systems Manager Session Manager để quản trị.
- Dùng GitHub OIDC và credential ngắn hạn thay cho AWS access key dài hạn.
- Xây dựng quy trình release frontend có thể lặp lại, có checksum, health check và rollback.
- Kiểm tra callback, logout, protected API và giao diện frontend trên domain đã cấu hình.

#### 2.4. Tiêu chí hoàn thành

| Hạng mục | Mục tiêu hoàn thành |
|---|---|
| Network | Có public subnet cho EC2, private subnet cho RDS, route được kiểm soát và không phát sinh NAT cost không cần thiết. |
| Security | Port 22 vẫn đóng; RDS chỉ nhận traffic từ EC2; quyền IAM theo nguyên tắc tối thiểu. |
| Compute | EC2, EBS, Elastic IP, Nginx, API và worker hoạt động, có thể quản trị bằng SSM. |
| Delivery | GitHub Actions build frontend, upload release, gọi SSM, kiểm tra health và giữ target rollback. |
| Identity | Cognito trả về đúng callback URL và API chỉ chấp nhận access token hợp lệ. |
| Operations | Log, health check, alarm, budget notification và hướng dẫn cleanup được ghi lại. |

### 3. Vấn đề cần giải quyết

#### 3.1. Soạn đề thủ công chậm và khó tái sử dụng

Giáo viên thường phải viết lại câu hỏi cho nhiều lớp và nhiều lần thi. Việc lưu câu hỏi trong các tài liệu riêng lẻ cũng làm khó việc biết câu nào đã được review, đã duyệt hoặc đã sử dụng. AI Exam Maker cung cấp cơ chế sinh có cấu trúc và question bank có thể tìm kiếm, nhưng kết quả AI vẫn là bản nháp cho đến khi có người kiểm tra.

#### 3.2. Kết quả AI không thể được tin tưởng nếu thiếu kiểm soát

Language model có thể sinh sai cấu trúc, tạo câu hỏi trùng hoặc đưa ra đáp án không khớp với lời giải thích. Vì vậy ứng dụng xem kết quả AI là dữ liệu không đáng tin cậy. Hệ thống kiểm tra schema, ghi lại kết quả job, lọc trùng và yêu cầu giáo viên cùng trưởng bộ môn review trước khi publish.

#### 3.3. Tác vụ AI lâu không nên làm treo giao diện

Lệnh gọi model có thể mất vài giây và có thể lỗi tạm thời. API tạo job rồi gửi message vào SQS. Worker nhận job, gọi Bedrock, lưu kết quả và báo trạng thái kết thúc. Retry, dead-letter queue, lease và cơ chế recovery giúp giảm nguy cơ mất hoặc xử lý trùng công việc.

#### 3.4. CloudFront chưa dùng được cho đường triển khai hiện tại

Thiết kế frontend ban đầu dùng S3 kết hợp CloudFront behaviors và CloudFront Function. Vì CloudFront chưa thể dùng trong môi trường hiện tại, dự án cần phương án fallback không phụ thuộc CDN nhưng vẫn phục vụ được static export. Phương án thay thế lưu artifact có version trong S3 và dùng SSM để EC2 tải xuống rồi kích hoạt.

#### 3.5. Truy cập hạ tầng phải an toàn và có thể lặp lại

Mở SSH hoặc đưa credential vĩnh viễn vào GitHub Actions khiến hệ thống khó bảo vệ và khó review. Thiết kế triển khai dùng EC2 instance profile, Session Manager, IAM least privilege và GitHub OIDC. Các giá trị phụ thuộc môi trường được giữ bên ngoài repository.

### 4. Kiến trúc giải pháp

AI Exam Maker chạy tại Region `ap-southeast-1` (Singapore). Kiến trúc cố ý dùng một EC2 và database RDS Single-AZ cho môi trường demo để giảm chi phí và độ phức tạp vận hành. Đây là trade-off được chấp nhận cho giai đoạn trình diễn, không phải cam kết về high availability production.

![Kiến trúc AWS của AI Exam Maker](/fcj-workshop-template/images/2-Proposal/project-architecture.jpg)

#### 4.1. Trách nhiệm của các thành phần

| Thành phần | Trách nhiệm |
|---|---|
| VPC, subnet và Internet Gateway | Tách compute công khai khỏi database riêng tư và cung cấp đường mạng cần thiết. |
| Security group | Cho phép HTTPS và traffic xác thực certificate đến EC2; chỉ cho EC2 kết nối PostgreSQL đến RDS. |
| EC2, EBS và Elastic IP | Chạy Nginx, frontend tĩnh, .NET API và AI worker bằng systemd. |
| Nginx và domain | Kết thúc HTTPS, phục vụ release đang active và chuyển `/api/*` đến API. |
| Amazon RDS for PostgreSQL | Lưu user, department, question, job, exam, version và audit record trong private subnet. |
| Amazon Cognito | Xác thực người dùng và cung cấp các group teacher, head, admin. |
| Amazon SQS và DLQ | Tách request sinh câu hỏi khỏi worker và giữ message lỗi nhiều lần. |
| Amazon Bedrock | Sinh câu hỏi nháp và tùy chọn kiểm tra đáp án theo workflow. |
| SSM Parameter Store và KMS | Cung cấp cấu hình và giá trị mã hóa mà không commit vào Git. |
| Amazon S3 | Lưu package release frontend và checksum để triển khai/rollback. |
| GitHub Actions và OIDC | Build, test, package và deploy mà không cần AWS credential dài hạn. |
| CloudWatch, SNS và Budgets | Thu log, thông báo alarm quan trọng và theo dõi chi phí. |

#### 4.2. Thiết kế network và security

VPC có public subnet cho EC2 và private subnet cho RDS. Thiết kế không dùng NAT Gateway vì môi trường demo không cần khoản chi phí cố định bổ sung này. EC2 chỉ nhận traffic cần thiết cho HTTPS và xác thực certificate. Port 22 không được mở. RDS chỉ chấp nhận kết nối PostgreSQL từ security group của EC2, không mở theo một CIDR public rộng.

EC2 nhận quyền thông qua instance profile. Session Manager được dùng cho truy cập quản trị và thực thi lệnh. GitHub Actions assume deployment role thông qua GitHub OIDC provider. Role được giới hạn theo repository, branch và các điều kiện workflow mà IAM hỗ trợ, đồng thời chỉ có quyền S3 và SSM cần thiết cho deployment.

#### 4.3. Luồng frontend và phương án CloudFront fallback

Đường CDN dự kiến dùng S3 làm origin frontend và CloudFront behaviors cho static asset và API. Khi CDN chưa thể dùng, luồng triển khai hoạt động như sau:

1. GitHub Actions cài dependency, build static export Next.js và chạy các bước CI.
2. Workflow đóng gói output thành release có version và tạo checksum SHA-256.
3. Artifact cùng checksum được upload vào release prefix trên S3.
4. Workflow gọi SSM Run Command trên EC2 đích.
5. EC2 tải artifact, kiểm checksum và giải nén vào thư mục release mới.
6. Script chỉ cập nhật symlink `current` sau khi release mới đã sẵn sàng.
7. Nginx được reload và thực hiện health check hoặc kiểm tra asset công khai.
8. Release trước vẫn được giữ để có thể rollback nếu activation thất bại.

Luồng này giúp việc triển khai có thể truy vết: GitHub run, S3 artifact, SSM invocation, thư mục release trên EC2 và symlink active đều có thể kiểm tra riêng.

#### 4.4. Luồng xác thực

Người dùng mở ứng dụng qua HTTPS và chọn đăng nhập. Cognito xác thực rồi trả authorization code về callback URL đã cấu hình. Frontend đổi code bằng PKCE, lưu session theo thiết kế ứng dụng và gọi API bằng access token. Backend kiểm tra issuer, audience, hạn token và hồ sơ user nội bộ trước khi trả dữ liệu bảo vệ. Khi logout, session cục bộ được xóa và người dùng trở về sign-out path đã cấu hình.

#### 4.5. Luồng xử lý AI job

API kiểm tra request, role, department, quota và các giá trị catalog được chọn. Sau đó API tạo job và gửi message vào SQS. Worker claim job, kiểm tra token budget trước mỗi lần gọi model, gọi Bedrock, kiểm tra và sanitize kết quả, lọc trùng khi cần và lưu câu hỏi nháp. Frontend polling trạng thái job và hiển thị lỗi có thể phục hồi nếu worker chưa hoàn tất. Các message SQS lặp lại không được tạo ra câu hỏi trùng.

### 5. Kế hoạch triển khai kỹ thuật

#### Giai đoạn 0 — Chốt quyết định AWS và ranh giới hạ tầng

- Chốt region, phương án domain, loại EC2, giả định model/quota, ranh giới IaC/runbook và chiến lược fallback.
- Đặt các ví dụ hạ tầng và script trong `/infra`, không chứa ID, ARN, secret hoặc giá trị phụ thuộc account thật.
- Xác định các điểm review cho VPC rule, IAM permission, deployment artifact và cleanup.

#### Giai đoạn 1 — Network, compute và HTTPS

- Tạo VPC, public/private subnet, route table, Internet Gateway và security group.
- Chuẩn bị EC2, EBS, Elastic IP, instance profile, SSM Agent access và systemd service boundary.
- Cấu hình domain, Nginx reverse proxy, HTTPS certificate, timer gia hạn và kiểm tra header origin nếu cần.
- Xác nhận backend truy cập RDS qua private network và các đường inbound không cần thiết vẫn đóng.

#### Giai đoạn 2 — Identity và tự động hóa triển khai

- Cấu hình GitHub OIDC provider và deployment IAM role với quyền S3/SSM tối thiểu.
- Xây dựng workflow frontend và đóng gói mỗi release kèm checksum.
- Upload artifact lên S3 và trigger triển khai EC2 qua SSM thay vì SSH.
- Giải nén vào release mới, cập nhật `current`, reload Nginx, kiểm tra kết quả và giữ release trước để rollback.

#### Giai đoạn 3 — Tích hợp và minh chứng

- Kiểm tra Cognito sign-in, callback, logout, protected API và frontend trên domain thật.
- Kiểm tra deployment thành công/thất bại, bao gồm checksum failure và rollback.
- Chụp minh chứng không nhạy cảm từ VPC, EC2, SSM, IAM, S3, GitHub Actions và health check.
- Review cost control, log, alarm và cleanup trước buổi demo nhóm.

### 6. Lộ trình và mốc triển khai

Roadmap của nhóm chạy từ ngày 17/11 đến ngày 01/12/2026. Công việc được chia song song để nhóm ứng dụng phát triển trong khi phần hạ tầng được chuẩn bị.

| Thời gian | Trọng tâm | Sản phẩm hạ tầng |
|---|---|---|
| 17–19/11 | Chốt nền | Quyết định AWS, runbook hạ tầng, rules repository và ranh giới CI ban đầu. |
| 20–24/11 | Nền tảng | VPC, security group, EC2, EBS, Elastic IP, Session Manager, domain, Nginx và HTTPS. |
| 25–29/11 | Tính năng và delivery | OIDC role, S3 release path, SSM deployment, frontend rollback và tích hợp đăng nhập. |
| 30/11–01/12 | Hoàn thiện | E2E check, minh chứng recovery, review budget, cleanup và chuẩn bị demo. |

Luồng demo chính được ưu tiên hơn các mục tùy chọn. Nếu thiếu thời gian, các cải tiến về availability production và automation không thiết yếu sẽ được để sau thay vì làm yếu network, authentication hoặc rollback.

### 7. Ước tính ngân sách và kiểm soát chi phí

Ước tính áp dụng cho giai đoạn phát triển và demo tại `ap-southeast-1`. Chi phí thực tế phụ thuộc số giờ chạy, dung lượng lưu trữ, traffic, số token AI và bảng giá AWS tại thời điểm sử dụng; các con số dưới đây là giả định lập kế hoạch, không phải hóa đơn cố định.

| Nhóm chi phí | Giả định | Biện pháp kiểm soát |
|---|---|---|
| EC2 và public IPv4 | Máy ARM nhỏ và Elastic IP chỉ chạy trong giờ làm việc. | Dừng instance ngoài lịch và release IP không dùng sau demo. |
| EBS | Một volume general-purpose nhỏ cho release và log. | Xóa volume rời và chỉ giữ số release cần thiết. |
| RDS | PostgreSQL Single-AZ nhỏ, encrypted storage và backup ngắn hạn. | Dừng ngoài giờ, theo dõi storage và xóa snapshot tạm sau khi review. |
| S3 | Một số lượng nhỏ package release frontend và checksum. | Dùng release prefix và lifecycle cleanup artifact cũ. |
| AI services | Dùng model rẻ hơn khi phát triển và giới hạn request khi demo. | Enforce quota theo user/toàn hệ thống và token ceiling trước model call. |
| Log và alarm | Log retention 7 ngày và nhóm alarm có giá trị cao. | Không log payload quá chi tiết và kiểm soát số lượng alarm. |

AWS Budgets được cấu hình cảnh báo ở 50%, 80% và 100% của khoản tín dụng 200 USD. Ứng dụng cũng giới hạn số câu hỏi mỗi request và số lần sinh theo ngày. Việc kết hợp lịch chạy hạ tầng với quota trong ứng dụng giúp giảm nguy cơ một lần test hoặc job lặp tiêu hết ngân sách.

### 8. Đánh giá rủi ro và phương án giảm thiểu

| Rủi ro | Ảnh hưởng | Khả năng | Giảm thiểu và dự phòng |
|---|---|---|---|
| AI sinh đáp án sai | Cao | Trung bình | Giữ output ở dạng draft, kiểm tra cấu trúc, cho phép verify và yêu cầu con người duyệt. |
| SQS giao trùng hoặc worker crash | Trung bình | Trung bình | Idempotent job claim, unique constraint, lease, heartbeat, retry và DLQ. |
| Truy cập trái phép vào draft/đáp án | Nghiêm trọng | Thấp | Backend kiểm tra role, department, ownership, content state; test permission matrix. |
| CloudFront không dùng được | Trung bình | Cao | Dùng fallback S3–SSM–EC2 và phục vụ frontend tĩnh qua Nginx/HTTPS. |
| Release hỏng hoặc deployment thất bại | Cao | Trung bình | Kiểm SHA-256, health check, activation qua `current` và giữ release trước để rollback. |
| EC2 hoặc RDS Single-AZ lỗi | Cao | Thấp | Chấp nhận trade-off demo, dùng systemd restart, RDS backup và thử restore. |
| EC2 bị scan từ Internet | Trung bình | Trung bình | Chỉ mở 443 và traffic xác thực cần thiết; đóng SSH, dùng Session Manager. |
| GitHub credential bị lộ | Nghiêm trọng | Thấp | Dùng OIDC ngắn hạn, giới hạn repository/branch, least privilege và secret scanning. |
| Cognito callback sai | Trung bình | Trung bình | Kiểm tra login, callback, logout và protected API trên frontend đã deploy. |
| Trễ tiến độ hai tuần | Cao | Trung bình | Làm song song, ưu tiên demo chính và hoãn tính năng production tùy chọn. |

### 9. Sản phẩm bàn giao kỳ vọng

- Tài liệu kiến trúc AWS và quyết định hạ tầng.
- Cấu hình VPC, subnet, route, security group, EC2, EBS, Elastic IP, Nginx, HTTPS và Session Manager.
- GitHub OIDC provider và deployment role theo nguyên tắc quyền tối thiểu.
- Workflow CI/CD frontend, S3 release artifact, SSM deployment script, health check và rollback path.
- Luồng tích hợp Cognito sign-in end-to-end với nhóm frontend.
- Screenshot không nhạy cảm và runbook làm minh chứng cho worklog và buổi demo.
- Hướng dẫn cleanup gồm snapshot, EBS volume, Elastic IP, S3 artifact và tài nguyên đã stop.

### 10. Kết luận và hướng phát triển

AI Exam Maker kết hợp việc tạo câu hỏi có hỗ trợ AI với quy trình duyệt đề có thể review và truy vết. Thiết kế hạ tầng cân bằng giữa bảo mật, chi phí và tốc độ triển khai: database nằm trong private subnet, quản trị dùng Session Manager, GitHub dùng OIDC, còn frontend release được kiểm tra và rollback. Phương án fallback CloudFront giúp dự án vẫn có thể demo ngay cả khi đường CDN ban đầu chưa dùng được.

Trong giai đoạn tiếp theo, kiến trúc có thể mở rộng thêm load balancer, database Multi-AZ, CDN/WAF, mức độ IaC cao hơn, sinh câu hỏi từ tài liệu, exam blueprint, nhiều biến thể đề và DOCX export. Các cải tiến đó nên thực hiện sau khi luồng demo cốt lõi ổn định và giả định ngân sách được kiểm tra lại.
