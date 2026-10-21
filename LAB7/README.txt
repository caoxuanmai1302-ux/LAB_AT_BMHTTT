# LAB7 – pfSense Web Protection

## 1. Giới thiệu

LAB7 xây dựng hệ thống kiểm soát truy cập web trong mạng nội bộ, sử dụng pfSense kết hợp với Squid, E2Guardian và ClamAV.

Mục tiêu:
- Kiểm soát truy cập Internet từ máy Client.
- Chặn các website theo danh sách chính sách.
- Quét nội dung tải xuống bằng antivirus.
- Quản lý luồng truy cập thông qua firewall pfSense.
- Kiểm thử và ghi nhận kết quả cho từng chính sách.

## 2. Mô hình hệ thống

Các máy ảo được kế thừa từ LAB6 và bổ sung máy chủ WEB-PROXY cho LAB7.

| Máy ảo | Địa chỉ IP | Vai trò |
|---|---|---|
| pfSense-LAB6 | `10.6.0.1` (LAN) | Gateway, firewall và định tuyến |
| DC01 | `10.6.0.10` | DNS/Active Directory |
| Client-A | `10.6.0.21` | Máy kiểm thử người dùng |
| Client-B | `10.6.0.22` | Máy kiểm thử bổ sung |
| WEB-PROXY | `10.6.0.30` | Squid, E2Guardian và ClamAV |
| DMZ-WEB | `172.16.6.10` | Máy chủ web kiểm thử |

> Địa chỉ IP trên là cấu hình dự kiến của mô hình. Cần đối chiếu với cấu hình thực tế nếu đã thay đổi trong quá trình triển khai.

## 3. Công nghệ sử dụng

- **pfSense:** Firewall, gateway và quản lý các luật mạng.
- **Squid:** Proxy trung gian cho các yêu cầu HTTP.
- **E2Guardian:** Lọc URL, domain và nội dung web theo chính sách.
- **ClamAV:** Antivirus dùng để kiểm tra tệp tải xuống khi được tích hợp và cấu hình thành công.
- **Ubuntu Server 24.04:** Hệ điều hành của WEB-PROXY.
- **Windows Client:** Máy thực hiện các bài kiểm thử.
- **DNS/Active Directory:** Phân giải tên miền nội bộ phục vụ môi trường lab.

## 4. Cài đặt và cấu hình

### 4.1. Chuẩn bị mạng

1. Khởi động các máy ảo thuộc mô hình LAB6 và WEB-PROXY.
2. Kiểm tra card mạng của từng máy ảo.
3. Đảm bảo WEB-PROXY có thể liên lạc với gateway và DNS.
4. Kiểm tra phân giải DNS của các tên miền kiểm thử.
5. Xác nhận các luật firewall cần thiết trên pfSense.

Thông số WEB-PROXY:

- Hệ điều hành: Ubuntu Server 24.04.
- RAM: 2 GB.
- CPU: 2 vCPU.
- Disk: 20 GB.
- IP LAN: `10.6.0.30/24`.
- Gateway: `10.6.0.1`.
- DNS: `10.6.0.10`.

### 4.2. Cấu hình proxy

Mô hình xử lý yêu cầu web:

```text
Client-A / Client-B
        |
        v
E2Guardian :8080
        |
        v
Squid :3128
        |
        v
pfSense
        |
        v
Internet
```

Trên trình duyệt của máy Client, cấu hình proxy thủ công:

- Proxy server: `10.6.0.30`
- Port: `8080`

Kiểm tra dịch vụ trên WEB-PROXY:

```bash
sudo systemctl status e2guardian
sudo systemctl status squid
```

Nếu sử dụng ClamAV, kiểm tra thêm:

```bash
sudo systemctl status clamav-daemon
```

Trạng thái dịch vụ cần được xác minh trực tiếp trên máy chủ trước khi tiến hành kiểm thử.

### 4.3. Cấu hình chính sách lọc web

E2Guardian sử dụng các danh sách và cấu hình lọc để kiểm soát những website được phép truy cập.

Một số domain kiểm thử:

| Domain | Mục đích |
|---|---|
| `normal.lab7.test` | Kiểm tra website được phép |
| `china.lab7.test` | Kiểm tra chính sách chặn domain |
| `mideast.lab7.test` | Kiểm tra danh sách chặn |
| `jobs.lab7.test` | Kiểm tra danh sách chặn |
| `blockedgame.lab7.test` | Kiểm tra chính sách chặn website giải trí/game |

Các domain trên cần được phân giải đúng trong DNS và được ánh xạ đến máy chủ kiểm thử phù hợp.

### 4.4. Trang thông báo bị chặn

Tệp mẫu trang chặn:

```text
custom_block_template.html
```

Có thể tùy chỉnh tiêu đề, màu sắc, thông báo và thông tin liên hệ để nhận diện môi trường LAB7.

Sau khi thay đổi, cần xác định đúng vị trí template mà phiên bản E2Guardian đang sử dụng và kiểm tra cấu hình thực tế. Chỉ sao chép tệp vào một thư mục mới chưa đảm bảo hệ thống tự động sử dụng tệp đó.

### 4.5. Kiểm tra antivirus

ClamAV được sử dụng để kiểm tra tệp tải xuống khi bộ quét đã được tích hợp với E2Guardian.

Có thể sử dụng tệp kiểm thử antivirus EICAR trong môi trường lab để xác minh hành vi phát hiện. Không sử dụng mã độc thật để kiểm tra.

Kết quả chỉ được đánh dấu đạt khi đã xác minh được bộ quét hoạt động và yêu cầu tải tệp được xử lý đúng theo chính sách.

## 5. Kiểm thử hệ thống

| STT | Nội dung | Kết quả mong đợi |
|---|---|---|
| 1 | Truy cập `normal.lab7.test` | Truy cập thành công |
| 2 | Truy cập `china.lab7.test` | Bị chặn theo chính sách |
| 3 | Truy cập `mideast.lab7.test` | Bị chặn theo chính sách |
| 4 | Truy cập `jobs.lab7.test` | Bị chặn theo chính sách |
| 5 | Truy cập `blockedgame.lab7.test` | Bị chặn theo chính sách |
| 6 | Tải tệp EICAR kiểm thử | Antivirus phát hiện và xử lý theo cấu hình |
| 7 | Truy cập web qua proxy | Yêu cầu đi qua WEB-PROXY |
| 8 | Kiểm tra trang thông báo chặn | Hiển thị trang chặn tùy chỉnh nếu đã cấu hình đúng |

## 6. Khắc phục sự cố

### Client không truy cập được Internet

- Kiểm tra IP, subnet mask và default gateway của Client.
- Kiểm tra Client có kết nối đúng mạng ảo hay không.
- Kiểm tra khả năng ping gateway.
- Kiểm tra trạng thái interface LAN trên pfSense.
- Kiểm tra DNS và các luật firewall liên quan.

### Website bị chặn vẫn truy cập được

- Xác nhận trình duyệt đang sử dụng proxy `10.6.0.30:8080`.
- Xác nhận domain phân giải đúng địa chỉ IP.
- Kiểm tra danh sách lọc và cấu hình E2Guardian đang được nạp.
- Kiểm tra log của E2Guardian và kết quả HTTP trả về.

### Website được phép cũng bị chặn

- Kiểm tra domain có vô tình nằm trong danh sách chặn hay không.
- Kiểm tra cú pháp và thứ tự áp dụng chính sách.
- Kiểm tra log để xác định thành phần trả về trang chặn.

### Dịch vụ proxy không hoạt động

```bash
sudo systemctl status e2guardian
sudo systemctl status squid
sudo journalctl -u e2guardian -n 50 --no-pager
sudo journalctl -u squid -n 50 --no-pager
```

Đọc thông báo lỗi trước khi chỉnh sửa cấu hình hoặc khởi động lại dịch vụ.

## 7. Giới hạn triển khai

- Việc lọc website phụ thuộc vào DNS, proxy và danh sách chính sách được cấu hình.
- Kiểm soát truy cập HTTPS có thể cần cấu hình bổ sung; không mặc định rằng mọi nội dung HTTPS đều được quét.
- Giới hạn dung lượng quét của E2Guardian không đồng nghĩa với chính sách chặn tuyệt đối mọi tệp lớn hơn một ngưỡng cụ thể.
- Schedule trên pfSense cần được thiết kế phù hợp với luồng mạng. Nếu toàn bộ trình duyệt dùng chung proxy, luật áp dụng cho proxy có thể ảnh hưởng đến mọi website đi qua proxy thay vì chỉ một nhóm domain.
- Giới hạn thời gian phiên Captive Portal không tương đương với quota riêng cho từng danh mục website.

## 8. Kết luận

LAB7 triển khai mô hình kiểm soát truy cập web sử dụng pfSense kết hợp Squid, E2Guardian và ClamAV. Hệ thống được đánh giá thông qua khả năng kết nối mạng, hoạt động của proxy, chính sách cho phép/chặn website và kiểm thử antivirus.

Kết luận cuối cùng cần dựa trên kết quả kiểm thử thực tế và các ảnh minh chứng được thu thập trong quá trình triển khai.
