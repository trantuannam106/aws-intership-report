---
title: "Worklog Tuần 2: AWS Networking - VPC & Secure Connectivity"
date: 2026-05-06
description: "Hành trình xây dựng VPC từ con số 0, từ subnet cơ bản đến các giải pháp truy cập Zero Trust như EIC Endpoint và SSM Session Manager."
tags: ["AWS", "VPC", "EC2", "Networking", "Security", "First Cloud Journey"]
---

## Tổng quan tuần này

Tuần 2 của First Cloud Journey đưa mình bước vào một trong những chủ đề "khó nhằn" nhất nhưng cũng quan trọng nhất của AWS: **Networking**. Sau tuần đầu làm quen với những thứ "thấy được" như EC2 và S3, lần này mình phải làm việc với những khái niệm trừu tượng hơn — VPC, Subnet, Route Table, Gateway — những thứ vận hành âm thầm phía sau nhưng quyết định toàn bộ tính bảo mật và kết nối của hệ thống.

Workshop chính tuần này là [Bắt đầu với Amazon VPC và AWS VPN Site-to-Site](https://000003.awsstudygroup.com/vi/), và mình đã hoàn thành phần lớn các module liên quan đến VPC nội bộ. Phần VPN Site-to-Site sẽ để dành cho cuối tuần.

---

## 1. Bối cảnh và mục tiêu

Hình dung VPC như một "khu chung cư riêng" của bạn trong thành phố AWS — bạn có rào, có cổng, có sơ đồ đường nội bộ, và bạn quyết định ai được vào, đi đâu, ra ngoài bằng đường nào. Mục tiêu tuần này là tự tay xây dựng "khu chung cư" đó, rồi thử nghiệm 3 cách khác nhau để đi vào "căn hộ Private" bên trong:

1. Cách truyền thống: SSH qua "căn hộ Public" làm trạm trung chuyển (Bastion pattern)
2. Cách hiện đại: dùng EIC Endpoint — kết nối thẳng qua hạ tầng AWS không cần qua public internet
3. Cách Zero Trust: SSM Session Manager — truy cập terminal qua trình duyệt, không cần SSH key, không cần mở port nào cả

**Môi trường làm việc:**
- Region: `us-east-1` (N. Virginia)
- VPC: `vpc-0e3935c27e676dc1a` (CIDR `10.10.0.0/16`)
- 4 Subnets phân bổ trên 2 AZ (1a, 1b) — Public và Private mỗi AZ một cái
- EC2 Public: `10.10.1.131` (Amazon Linux 2023, t3.micro)
- EC2 Private: `10.10.3.142` (Amazon Linux 2023, t3.micro)
- Key pair: `aws-keypair-v2`

---

## 2. Dựng "bộ xương" hạ tầng VPC

Phần đầu tiên là dựng những thành phần cơ bản — và đây cũng là phần dễ "đi sai một li, đi một dặm" nhất nếu không hiểu rõ vai trò của từng thứ.

### Internet Gateway và NAT Gateway — hai cánh cổng khác nhau

Một trong những "aha moment" của mình tuần này là khi hiểu được tại sao AWS lại cần đến **hai loại gateway** khác nhau:

- **Internet Gateway (IGW)**: Cánh cổng hai chiều cho Public Subnet. Cho phép EC2 Public ra Internet và Internet vào EC2 Public.
- **NAT Gateway**: Cánh cổng một chiều cho Private Subnet. Cho phép EC2 Private *ra* Internet (để `yum update`, pull Docker image, gọi API), nhưng *chặn hoàn toàn* chiều từ Internet vào.

Mình đã triển khai NAT Gateway tại Public Subnet 1 với một Elastic IP riêng. Có một chi tiết AWS Console mới mà workshop chưa cập nhật: lúc tạo NAT GW giờ có lựa chọn **Availability mode (Regional vs Zonal)** — cái mới ra năm 2025. Mình chọn **Zonal** vì khớp với nội dung workshop và giúp hiểu rõ hơn về khái niệm "NAT GW gắn với 1 AZ duy nhất" — production thật sẽ cần 2 NAT GW (một cho mỗi AZ) để đạt HA.

> 💡 **Ghi nhớ về chi phí:** NAT Gateway là một trong những "kẻ ngốn tiền" thầm lặng của AWS — $0.045/giờ (khoảng $32/tháng) tính 24/7 ngay cả khi không có traffic, cộng thêm $0.045/GB data đi qua. Đây là lý do nhiều startup chọn dùng NAT Instance (EC2 tự cấu hình) thay vì NAT Gateway managed cho môi trường dev.

### Route Tables — "biển chỉ đường" của VPC

Nếu Subnet là khu phố thì Route Table là biển chỉ đường — nó quyết định khi có một packet đi vào subnet, packet đó sẽ được đẩy đi đâu tiếp theo.

Cấu hình của mình:

| Route Table | Đích `0.0.0.0/0` | Subnet associate |
|---|---|---|
| `Route table-Public` | Internet Gateway | Public Subnet 1, 2 |
| `Route table-Private` | NAT Gateway | Private Subnet 1, 2 |

Cả hai Route Table đều có route mặc định `10.10.0.0/16` → `local`, đảm bảo mọi traffic nội bộ trong VPC luôn đi qua AWS backbone, không bao giờ ra Internet.

---

## 3. Xác thực kết nối — khi lý thuyết gặp thực tế

Dựng xong hạ tầng thì phải kiểm tra. Đây là phần mình thấy thú vị nhất tuần này.

### SSH Jump qua Bastion (pattern truyền thống)

Vì EC2 Private không có Public IP, cách kinh điển để truy cập là dùng EC2 Public làm "trạm trung chuyển":

```bash
# Bước 1: SSH vào EC2 Public (qua VS Code Remote SSH cho tiện)
ssh -i aws-keypair-v2.pem ec2-user@<PUBLIC_IP>

# Bước 2: Copy key vào Public, rồi từ Public SSH tiếp sang Private
scp -i aws-keypair-v2.pem aws-keypair-v2.pem ec2-user@<PUBLIC_IP>:~/
ssh -i ~/aws-keypair-v2.pem ec2-user@10.10.3.142
```

Một bài học nhỏ nhưng "đau thương": nếu không `chmod 400` cho file `.pem` sau khi copy, SSH sẽ từ chối với lỗi `WARNING: UNPROTECTED PRIVATE KEY FILE!`. AWS bắt buộc key phải được bảo vệ chặt — chuyện hiển nhiên nhưng dễ quên.

### Test NAT Gateway hoạt động

Sau khi SSH vào Private, lệnh kiểm tra "kinh điển":

```bash
ping -c 4 google.com
```

Nếu thấy reply về thì NAT Gateway đang làm việc. Trước khi tạo NAT GW thì lệnh này sẽ timeout — đây là cách hay để cảm nhận sự khác biệt rõ ràng giữa "có" và "không có" NAT.

### Reachability Analyzer — debug network không cần ssh

Một công cụ AWS rất ngầu mà mình mới biết: **VPC Reachability Analyzer**. Nó cho phép phân tích "logic" đường đi của packet giữa hai resource (ví dụ EC2-A → EC2-B) mà không cần thực sự gửi packet nào. Nó kiểm tra Route Table, Security Group, NACL, ENI... và báo lỗi cụ thể nếu có rào cản nào.

Mình đã thử phân tích đường đi giữa EC2 Public (SG `sg-0851...`) và EC2 Private (SG `sg-04ca...`) — kết quả `Reachable` ✅. Nếu sai cấu hình SG, công cụ này sẽ chỉ chính xác rule nào đang chặn — cực kỳ hữu ích khi debug những hệ thống lớn.

> 💰 Lưu ý: $0.10/lần phân tích — không phải miễn phí, nên đừng bấm bừa.

---

## 4. EC2 Instance Connect Endpoint — "cánh cửa bí mật" của AWS

Đây là phần làm mình bất ngờ nhất tuần này. **EIC Endpoint** là một feature tương đối mới (ra mắt 2023) cho phép truy cập SSH vào EC2 Private **không cần** Public IP, **không cần** NAT Gateway, **không cần** Bastion Host. Và quan trọng nhất: **miễn phí**.

### Cách nó hoạt động

EIC Endpoint là một "endpoint ảo" trong VPC mà AWS quản lý. Khi bạn gửi yêu cầu SSH qua nó (từ AWS Console hoặc CLI), traffic sẽ đi qua hạ tầng nội bộ của AWS thay vì qua Internet. Nó như một "cánh cửa bí mật" mà chỉ những người có quyền IAM mới biết tới.

### Triển khai

1. Tạo EIC Endpoint (`eice-0f7c...`) trong Private Subnet
2. Cấu hình Security Group cho EC2 Private: cho phép port 22 inbound từ Security Group của EIC Endpoint (chứ không phải mở từ `0.0.0.0/0`)
3. Vào EC2 Console → chọn EC2 Private → **Connect** → tab **EC2 Instance Connect Endpoint** → **Connect using a Private IP**

Boom, terminal mở ra, SSH thành công. Không cần copy key, không cần biết Public IP, không cần Bastion.

### So với Bastion truyền thống

| | Bastion truyền thống | EIC Endpoint |
|---|---|---|
| Cần EC2 Public chạy 24/7 | ✅ Có (tốn tiền) | ❌ Không |
| Cần quản lý key pair | ✅ Có | ❌ Không (key tạm) |
| Cần expose Port 22 ra Internet | ✅ Có (rủi ro) | ❌ Không |
| Audit logging | Phải tự config | ✅ Built-in (CloudTrail) |
| Chi phí | EC2 + EBS | ✅ **Miễn phí** |

Đây có lẽ là một trong những feature ít được biết nhưng đáng dùng nhất của AWS. Mình quyết định **giữ EIC Endpoint** sau khi cleanup workshop, để dùng cho các lab tiếp theo.

---

## 5. SSM Session Manager — đỉnh cao của Zero Trust

Nếu EIC Endpoint đã ngầu thì **SSM Session Manager** còn ngầu hơn. Nó cho phép mở terminal vào EC2 **qua trình duyệt**, không cần SSH key, không cần port 22, không cần network reachability từ máy bạn tới EC2.

### Cơ chế hoạt động

Thay vì máy bạn "kết nối tới" EC2, ở đây EC2 sẽ chủ động "phone home" lên dịch vụ SSM của AWS thông qua **SSM Agent** (đã pre-installed trên Amazon Linux 2023). Khi bạn mở session từ Console, AWS sẽ ghép kết nối browser của bạn với agent đang chờ trên EC2 — toàn bộ traffic qua AWS backbone, được mã hoá end-to-end.

Điều này có nghĩa: kể cả EC2 Private không có Internet (chưa có NAT), vẫn có thể truy cập được nếu cấu hình VPC Endpoints đúng cách.

### Bước 1: IAM Role cho EC2

EC2 cần "danh tính" để giao tiếp với dịch vụ SSM:

- Tạo role `EC2-SessionManager-Role`
- Attach managed policy: `AmazonSSMManagedInstanceCore`
- Gán role cho cả 2 EC2 (Actions → Security → Modify IAM role)

### Bước 2: VPC Interface Endpoints

Đây là phần "đắt giá" — cả về chi phí lẫn kiến thức. Để EC2 Private giao tiếp với SSM mà không cần Internet, cần tạo 3 **Interface Endpoints** (kết nối qua AWS PrivateLink):

| Endpoint | Vai trò |
|---|---|
| `com.amazonaws.us-east-1.ssm` | Endpoint chính của dịch vụ SSM |
| `com.amazonaws.us-east-1.ssmmessages` | Kênh truyền tin nhắn 2 chiều giữa agent và service |
| `com.amazonaws.us-east-1.ec2messages` | Hỗ trợ SSM Agent gửi metadata về |

Mỗi endpoint cần:
- Đặt trong Private Subnet
- Gắn Security Group cho phép HTTPS (port 443) inbound từ VPC CIDR `10.10.0.0/16`
- Bật **Enable DNS name** để EC2 resolve được tên dịch vụ về IP private

> 💸 **Cảnh báo chi phí:** Mỗi Interface Endpoint là $0.01/giờ × 24 × 30 ≈ **$7.2/tháng**, nhân với 3 endpoint thành **$21.6/tháng** chỉ để bật được Session Manager mà không dùng NAT. Đây là trade-off: chấp nhận tốn một khoản cố định để đổi lấy security tối đa và không phụ thuộc Internet.

### Bước 3: Kết quả

Sau khi gắn IAM role và đợi khoảng 2-5 phút để SSM Agent đăng ký lên dịch vụ, cả 2 EC2 hiện ra trong danh sách **Target Instances** của Session Manager. Click **Start session** → terminal hiện ra ngay trong tab trình duyệt. Cảm giác lần đầu thấy được điều này khá là "wow" — hoàn toàn không có một port nào mở ra Internet, không có key pair, mà vẫn shell vào được EC2.

### So sánh tổng quát 3 phương thức

| | SSH qua Bastion | EIC Endpoint | SSM Session Manager |
|---|---|---|---|
| Cần Key Pair | ✅ | ⚠️ Tạm | ❌ |
| Cần mở Port 22 | ✅ | ⚠️ Trong VPC | ❌ |
| Cần IAM Role trên EC2 | ❌ | ❌ | ✅ |
| Cần Internet/NAT cho Private | ❌ | ❌ | ❌ (nếu có VPC Endpoints) |
| Chi phí | EC2 Bastion | Free | $21.6/tháng (3 endpoints) |
| Audit logging | Tự config | CloudTrail | CloudTrail + CloudWatch |
| Phù hợp với | Lab, dev | Mọi môi trường | Production, compliance |

---

## 6. Cost Optimization — kỷ luật của người làm Cloud

Mỗi resource trong AWS đều có thể là một "đồng hồ tính tiền" chạy 24/7. Sau khi hoàn thành các phần Lab và đã chụp đủ minh chứng, mình sẽ thực hiện cleanup theo thứ tự:

- [ ] **Delete NAT Gateway** — Tiết kiệm ~$1.08/ngày
- [ ] **Release Elastic IP** — Để tránh phí $0.005/giờ cho IP "treo" không gắn với resource nào
- [ ] **Delete 3 SSM VPC Endpoints** — Tiết kiệm ~$1.44/ngày
- [ ] **Giữ lại EIC Endpoint** vì miễn phí và hữu ích cho các tuần sau
- [ ] **Giữ lại VPC, Subnet, Route Table, IGW** — không tốn phí

Bài học lớn nhất tuần này: **chi phí không đến từ những thứ to mà từ những thứ "quên tắt"**. Một NAT Gateway dev quên xoá có thể ngốn $32/tháng cho không. Một Elastic IP allocate xong không gắn cũng tự âm thầm tính tiền. Cloud is cheap *if you know what you're doing* — nếu không, hoá đơn cuối tháng sẽ là một bài học rất "đắt".

---

## 7. Những gì mình rút ra được

1. **VPC không phức tạp như mình nghĩ** — chỉ cần hiểu vai trò từng thành phần (CIDR, Subnet, Route Table, Gateway) và cách chúng liên kết, mọi thứ trở nên rất logic.

2. **Có nhiều cách hơn 1 để làm cùng một việc**, và cách "truyền thống" (SSH + Bastion) thường không phải là cách tốt nhất nữa. AWS đang đẩy mạnh các giải pháp Zero Trust như EIC Endpoint và SSM Session Manager — chúng an toàn hơn, dễ audit hơn, và đôi khi còn rẻ hơn.

3. **Chi phí phải được suy nghĩ ngay từ khi thiết kế**, không phải sau khi nhận hoá đơn. NAT Gateway tiện lợi nhưng đắt; VPC Endpoints an toàn nhưng tốn cố định. Kiến trúc tốt là kiến trúc đặt cả security và cost vào bàn cân ngay từ đầu.

4. **Reachability Analyzer là người bạn tốt** khi debug network — luôn dùng nó trước khi nghi ngờ Security Group hay NACL.

---

## Tiến độ tuần tới

Phần còn lại của workshop tuần này sẽ là **VPN Site-to-Site** — kết nối một mạng giả lập "On-premise" (sẽ dùng một VPC khác đóng vai trò datacenter công ty) với VPC chính qua VPN. Đây là kịch bản rất phổ biến trong doanh nghiệp khi migrate workload từ on-prem lên cloud từ từ. Hứa hẹn sẽ có nhiều thứ thú vị để khám phá về Customer Gateway, Virtual Private Gateway, và BGP routing.

*Stay tuned cho Tuần 3!* 🚀

