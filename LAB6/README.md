# LAB 6 – Xác thực Active Directory và chính sách truy cập trên pfSense

## 1. Thông tin sinh viên

- **Họ và tên:** Cao Xuân Mai
- **MSSV:** 1150070028
- **Lớp:** 11_TTMT
- **Bài thực hành:** LAB 6 – Xác thực Active Directory và chính sách truy cập trên pfSense

## 2. Mục tiêu

- Xây dựng mô hình mạng riêng cho LAB6 trên VMware Workstation.
- Cấu hình pfSense với các vùng WAN, MGMT, LAN và DMZ.
- Tích hợp pfSense với Active Directory thông qua LDAP.
- Cấu hình Captive Portal để xác thực người dùng bằng tài khoản miền.
- Tạo Alias và Schedule để quản lý chính sách truy cập.
- Thực hiện và kiểm thử 5 kịch bản kiểm soát truy cập.
- Kiểm tra khả năng cô lập mạng DMZ và ghi nhận kết quả bằng ảnh minh chứng.

## 3. Môi trường thực hành

| Thành phần | Thông tin |
|---|---|
| Phần mềm ảo hóa | VMware Workstation |
| Firewall | pfSense |
| Máy chủ miền | Windows Server 2022 – DC01 |
| Máy khách | Client-A, Client-B |
| Máy chủ web | Ubuntu Server – DMZ-WEB |
| Domain | `lab6.test` |
| LDAP | TCP port 389 |
| Web | HTTP port 80, HTTPS port 443 |

### Địa chỉ IP

| Thiết bị / Interface | Địa chỉ IP |
|---|---|
| pfSense WAN | DHCP từ VMware NAT |
| pfSense MGMT | `192.168.56.254/24` |
| Máy thật quản trị | `192.168.56.1/24` |
| pfSense LAN | `10.6.0.1/24` |
| DC01 | `10.6.0.10/24` |
| Client-A | `10.6.0.21/24` |
| Client-B | `10.6.0.22/24` |
| pfSense DMZ | `172.16.6.1/24` |
| GENERAL_SITE | `172.16.6.10` |
| JOB_SITE | `172.16.6.20` |
| SOCIAL_SITE | `172.16.6.30` |
| RESTRICTED_SITE | `172.16.6.40` |

## 4. Các nội dung đã thực hiện

### 4.1. Xây dựng mô hình mạng

- Tạo các mạng ảo trong VMware Workstation.
- Cấu hình các interface WAN, MGMT, LAN và DMZ trên pfSense.
- Thiết lập địa chỉ IP cho DC01, Client-A, Client-B và Ubuntu DMZ-WEB.
- Cài đặt Nginx để mô phỏng các website phục vụ kiểm thử.

### 4.2. Tích hợp Active Directory bằng LDAP

- Cài đặt Active Directory Domain Services và DNS trên DC01.
- Tạo domain `lab6.test`, OU `LAB6-Users` và các tài khoản thực hành.
- Cấu hình Authentication Server trên pfSense để kết nối đến LDAP của DC01.
- Kiểm tra xác thực bằng hai tài khoản `sv_a` và `sv_b`.

**Kết quả:** Hai tài khoản xác thực LDAP thành công.

### 4.3. Cấu hình Captive Portal

- Tạo Captive Portal zone trên giao diện LAN.
- Cấu hình LDAP làm phương thức xác thực.
- Kiểm tra trang đăng nhập và đăng nhập bằng tài khoản miền.

**Kết quả:** Captive Portal hoạt động và đăng nhập thành công trong quá trình kiểm thử.

### 4.4. Tạo Alias và Schedule

Các Alias được sử dụng:

- `CLIENT_A`: `10.6.0.21`
- `CLIENT_B`: `10.6.0.22`
- `POLICY_CLIENTS`: `10.6.0.21`, `10.6.0.22`
- `GENERAL_SITE`: `172.16.6.10`
- `JOB_SITE`: `172.16.6.20`
- `SOCIAL_SITE`: `172.16.6.30`
- `RESTRICTED_SITE`: `172.16.6.40`
- `WEB_PORTS`: TCP port 80 và 443

Các Schedule:

- `LUNCH_WINDOW`: Thứ Hai đến Thứ Sáu, 11:30–13:00.
- `WORK_HOURS`: Thứ Hai đến Thứ Sáu, 08:00–17:00.
- `TEST_NOW`: Khoảng thời gian ngắn dùng để kiểm thử tạm thời.

## 5. Kết quả thực hiện 5 kịch bản

### Kịch bản 1 – Chặn JOB_SITE đối với Client-A

- Chặn Client-A truy cập `172.16.6.20` qua TCP port 80.
- Cho phép Client-A truy cập `172.16.6.10` qua TCP port 80.
- Kiểm tra thứ tự luật Block và Pass trên pfSense.

**Kết quả: PASS** – Client-A truy cập GENERAL_SITE thành công và bị chặn khi truy cập JOB_SITE.

### Kịch bản 2 – Chặn RESTRICTED_SITE đối với Client-A và Client-B

- Chặn hai máy khách truy cập `172.16.6.40` qua TCP port 80.
- Kiểm tra quyền truy cập GENERAL_SITE theo luật cho phép.

**Kết quả: PASS** – Client-A và Client-B bị chặn khi truy cập RESTRICTED_SITE theo yêu cầu.

### Kịch bản 3 – Giới hạn SOCIAL_SITE theo giờ nghỉ trưa

- Cho phép Client-B truy cập `172.16.6.30` trong Schedule `LUNCH_WINDOW`.
- Sử dụng `TEST_NOW` để kiểm thử nhanh khi cần.
- Kiểm tra khả năng truy cập khi Schedule không còn hiệu lực.

**Kết quả: PASS** – Chính sách truy cập SOCIAL_SITE theo thời gian được kiểm thử thành công.

### Kịch bản 4 – Giới hạn HTTP/HTTPS theo giờ làm việc

- Cho phép Client-A và Client-B truy cập web qua TCP port 80 và 443 trong `WORK_HOURS`.
- Sử dụng luật Block để kiểm soát lưu lượng còn lại theo yêu cầu.
- Kiểm tra hoạt động của chính sách theo thời gian.

**Kết quả: PASS** – Các quyền truy cập được kiểm tra theo chính sách đã cấu hình.

### Kịch bản 5 – Cô lập DMZ khỏi LAN

- Tạo luật Block từ `DMZ net` đến `LAN net` trên giao diện DMZ.
- Bật ghi log để theo dõi lưu lượng bị chặn.
- Kiểm tra kết nối từ Ubuntu DMZ-WEB đến DC01.
- Kiểm tra Client-A truy cập GENERAL_SITE.

**Kết quả: PASS** – Kết nối khởi tạo từ DMZ đến LAN bị chặn, trong khi truy cập từ LAN đến website được cho phép vẫn hoạt động.

## 6. Bảng tổng hợp kết quả

| Hạng mục | Kết quả |
|---|---|
| Xác thực LDAP bằng `sv_a`, `sv_b` | PASS |
| Đăng nhập Captive Portal | PASS |
| Kịch bản 1 – JOB_SITE | PASS |
| Kịch bản 2 – RESTRICTED_SITE | PASS |
| Kịch bản 3 – SOCIAL_SITE theo Schedule | PASS |
| Kịch bản 4 – WORK_HOURS | PASS |
| Kịch bản 5 – DMZ isolation | PASS |

Các kết quả trên được ghi nhận theo quá trình kiểm thử thực tế của bài lab.

## 7. Lỗi gặp phải và cách khắc phục

### 7.1. LDAP xác thực không thành công

**Nguyên nhân:**
- Bind DN chưa đúng với Distinguished Name thực tế của tài khoản dịch vụ.
- Sai Base DN, Authentication Containers hoặc thông tin đăng nhập.

**Cách khắc phục:**
- Kiểm tra Base DN `DC=lab6,DC=test`.
- Kiểm tra Bind DN theo đúng vị trí tài khoản dịch vụ trong Active Directory.
- Kiểm tra cổng LDAP TCP 389 và thử lại bằng Diagnostics → Authentication.

### 7.2. Truy cập website không đúng chính sách

**Nguyên nhân:**
- Luật Block và Pass được đặt sai thứ tự.
- Một luật rộng hơn cho phép lưu lượng trước khi đến luật Block cần kiểm tra.

**Cách khắc phục:**
- Đặt luật Block cụ thể phía trên luật Pass rộng có thể khớp cùng kết nối.
- Kiểm tra TCP port 80 và Alias tương ứng.
- Apply Changes rồi tạo kết nối mới để kiểm tra lại.

### 7.3. Schedule không hoạt động như dự kiến

**Nguyên nhân:**
- Thời gian kiểm thử không nằm trong khung giờ của Schedule.
- Giờ hệ thống hoặc múi giờ pfSense chưa chính xác.
- Luật Pass có Schedule không còn hiệu lực ngoài giờ.

**Cách khắc phục:**
- Kiểm tra giờ hệ thống và múi giờ pfSense.
- Dùng `TEST_NOW` để kiểm thử tạm thời.
- Bảo đảm có luật Block phù hợp phía dưới để chặn truy cập khi luật Pass không còn hiệu lực.
- Sau khi kiểm thử, khôi phục `LUNCH_WINDOW` và `WORK_HOURS` theo yêu cầu.

### 7.4. Kết quả chưa thay đổi sau khi sửa firewall rule

**Nguyên nhân**
- Kết nối cũ vẫn còn trạng thái trong bảng state của pfSense.
- Trình duyệt đang sử dụng phiên kết nối trước đó.

**Cách khắc phục:**
- Tạo kết nối mới và kiểm tra lại.
- Nếu cần, vào Diagnostics → States để kiểm tra trạng thái kết nối và Reset States phù hợp trong môi trường lab.


## 8. Ảnh minh chứng

Lưu ảnh chụp màn hình thực tế vào word phù hợp


Tên ảnh thay đổi theo tên bước thực tế. Không sử dụng ảnh minh họa thay cho kết quả kiểm thử.

## 9. Kết luận

Bài LAB6 đã hoàn thành các nội dung cấu hình và kiểm thử theo yêu cầu, bao gồm tích hợp Active Directory thông qua LDAP, triển khai Captive Portal, xây dựng Alias và Schedule, thiết lập các luật firewall theo địa chỉ nguồn, địa chỉ đích, cổng dịch vụ và thời gian.

Cả năm kịch bản kiểm soát truy cập đã được kiểm thử thành công. Kịch bản cô lập DMZ cho thấy khả năng ngăn kết nối khởi tạo từ DMZ đến LAN, đồng thời duy trì truy cập từ LAN đến website được cho phép theo chính sách đã cấu hình.
