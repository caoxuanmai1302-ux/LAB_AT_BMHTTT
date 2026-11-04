# LAB10: REMOTE DNS CACHE POISONING / KAMINSKY SIMULATION

## 1. Thông tin sinh viên

- **Họ và tên:** Cao Xuân Mai
- **MSSV:** 1150070028
- **Tên bài LAB:** LAB10 – Remote DNS Cache Poisoning / Kaminsky Simulation
- **Môi trường thực hiện:** VMware
- **Hệ điều hành:** Ubuntu Server, Kali Linux
- **Các thành phần chính:** pfSense, BIND9, DNS-SERVER, VICTIM, AUTH-DNS/RELAY, DMZ-WEB

## 2. Tên LAB và phiên bản môi trường

### 2.1. Tên LAB

LAB10 – Remote DNS Cache Poisoning / Kaminsky Simulation.

### 2.2. Mục tiêu

- Tìm hiểu sự khác nhau giữa Local DNS Spoofing và Remote DNS Cache Poisoning.
- Tìm hiểu vai trò của Transaction ID, UDP Source Port và DNS Cache.
- Quan sát cơ chế DNS phân giải và các bản ghi NS/Glue.
- Kiểm tra kết quả phân giải DNS trước và sau mô phỏng.
- Khôi phục BIND9 và áp dụng các biện pháp bảo vệ DNS.

### 2.3. Môi trường thực hiện

Bài thực hành được triển khai trên các máy ảo VMware, sử dụng mạng nội bộ và các địa chỉ IP riêng.

| Thành phần | Vai trò | Địa chỉ IP |
|---|---|---|
| pfSense DNSLAB | Gateway DNSLAB | 10.9.0.1 |
| DNS-SERVER | DNS Resolver BIND9 | 10.9.0.10 |
| VICTIM | Máy khách DNS | 10.9.0.100 |
| pfSense ATTACKNET | Gateway ATTACKNET | 10.10.0.1 |
| Kali Linux | Máy ATTACKER | 10.10.0.200 |
| AUTH-DNS/RELAY | DNS có thẩm quyền và relay | 172.16.6.20 |
| DMZ-WEB | Website hợp lệ | 172.16.6.10 |

## 3. Cách dựng và cấu hình môi trường

### 3.1. Chuẩn bị máy ảo

- Chuẩn bị các máy ảo VMware theo yêu cầu LAB10.
- Tạo snapshot trước khi thay đổi cấu hình.
- Kiểm tra các máy ảo và card mạng.
- Thiết lập mạng DNSLAB và ATTACKNET.
- Kiểm tra kết nối giữa các thành phần trong phạm vi lab.

### 3.2. Cấu hình mạng

Các máy được cấu hình địa chỉ IP theo sơ đồ LAB10.

- DNS-SERVER: `10.9.0.10`
- VICTIM: `10.9.0.100`
- Kali ATTACKER: `10.10.0.200`
- AUTH-DNS/RELAY: `172.16.6.20`
- DMZ-WEB: `172.16.6.10`

pfSense được sử dụng để quản lý các mạng và kiểm soát lưu lượng giữa các thành phần.

### 3.3. Cấu hình DNS-SERVER

DNS-SERVER sử dụng BIND9 để thực hiện chức năng DNS Resolver.

Các nội dung cấu hình gồm:

- Thiết lập DNS baseline.
- Cấu hình forwarder đến AUTH-DNS.
- Giới hạn recursion ACL.
- Kiểm tra cấu hình BIND9.

Lệnh kiểm tra cấu hình:

```bash
sudo named-checkconf
```

### 3.4. Cấu hình VICTIM

VICTIM được cấu hình sử dụng DNS-SERVER làm DNS Resolver.

Kiểm tra phân giải tên miền:

```bash
resolvectl query www.example.lab10.test
```

Kết quả baseline mong đợi:

```text
www.example.lab10.test -> 172.16.6.10
```

### 3.5. Cấu hình các thành phần còn lại

- AUTH-DNS/RELAY cung cấp dịch vụ DNS trong môi trường mô phỏng.
- DMZ-WEB cung cấp website hợp lệ.
- Kali Linux được sử dụng trong mạng ATTACKNET.
- pfSense thực hiện kiểm soát và phân tách mạng.
- Wireshark hoặc tcpdump được sử dụng để quan sát lưu lượng DNS.

## 4. Các tình huống đã thực hiện

Tình huống 1: Chuẩn bị môi trường LAB10

Thực hiện chuẩn bị máy ảo, cấu hình mạng và kiểm tra các thành phần.

Nội dung:

Kiểm tra các máy ảo.

Cấu hình địa chỉ IP.

Thiết lập mạng DNSLAB và ATTACKNET.

Kiểm tra các quy tắc firewall trên pfSense.

Tình huống 2: Kiểm tra DNS baseline

Kiểm tra khả năng phân giải DNS bình thường trước khi mô phỏng.

Domain kiểm tra:

www.example.lab10.test

Kết quả baseline:

172.16.6.10

Kết quả cho thấy VICTIM có thể phân giải tên miền thử nghiệm về địa chỉ IP của DMZ-WEB hợp lệ.

Tình huống 3: Kiểm tra cách ly mạng

Kiểm tra khả năng phân tách mạng DNSLAB và ATTACKNET.

Nội dung:

Kiểm tra kết nối mạng.

Kiểm tra các quy tắc firewall.

Quan sát lưu lượng DNS.

Kiểm tra khả năng cách ly giữa các mạng.

Đảm bảo môi trường thực hành nằm trong phạm vi mạng lab.

Tình huống 4: Phần F – Chuẩn bị mô phỏng DNS Cache Poisoning

Thực hiện chuẩn bị các thành phần phục vụ mô phỏng Remote DNS Cache Poisoning theo tài liệu LAB10.

Nội dung:

Kiểm tra DNS-SERVER và resolver.

Chuẩn bị các thành phần trên Kali ATTACKER.

Kiểm tra AUTH-DNS/RELAY.

Kiểm tra các thành phần DNS và website trong môi trường lab.

Chuẩn bị môi trường mô phỏng và kiểm tra cấu hình mạng.

Tình huống 5: Phần G – Remote DNS Cache Poisoning / Kaminsky Simulation

Thực hiện mô phỏng DNS Cache Poisoning trong môi trường mạng nội bộ LAB10.

Nội dung quan sát:

Transaction ID.

UDP Source Port.

DNS Cache.

DNS response.

Additional/Glue records.

Kết quả phân giải DNS trên VICTIM.

Website được truy cập sau mô phỏng.

Tình huống 6: Phân tích cache và kết quả

So sánh kết quả phân giải DNS trước và sau mô phỏng.

Nội dung:

Địa chỉ IP được phân giải.

DNS upstream của resolver.

Website được hiển thị.

Lưu lượng DNS quan sát được.

Phân tích vai trò của Additional/Glue records và bailiwick.

Tình huống 7: Khôi phục BIND9 và Hardening

Thực hiện khôi phục DNS Resolver và áp dụng các biện pháp bảo vệ.

Nội dung:

Khôi phục BIND9.

Thu hẹp recursion ACL.

Kiểm tra source-port randomization.

Duy trì DNSSEC validation.

Loại bỏ các rule phục vụ mô phỏng không còn cần thiết.

Tắt hoặc tháo ATTACKNET sau buổi lab.

Khôi phục snapshot nếu cần.

## 5. Kết quả từng tình huống PASS/FAIL

**Lưu ý:** Bảng dưới đây là mẫu ghi nhận. Cần cập nhật PASS/FAIL dựa trên kết quả thực tế và minh chứng của từng tình huống.

| STT | Tình huống | Kết quả | Ghi chú |
|---|---|---|---|
| 1 | Chuẩn bị môi trường LAB10 | PASS | Các máy ảo đã được tạo và cấu hình |
| 2 | Kiểm tra DNS baseline | PASS | VICTIM phân giải về `172.16.6.10` |
| 3 | Kiểm tra cách ly mạng | PASS | Kali đã sử dụng ATTACKNET |
| 4 | Mô phỏng Remote DNS Cache Poisoning | PASS | Căn cứ kết quả thực tế |
| 5 | Phân tích kết quả DNS | PASS | Ghi nhận IP trước và sau mô phỏng |
| 6 | Khôi phục BIND9 | PASS | Kiểm tra dịch vụ sau khôi phục |
| 7 | Hardening DNS | PASS | Kiểm tra ACL, DNSSEC và firewall |

## 6. Lỗi gặp phải và cách khắc phục

### 6.1. Lỗi cấu hình mạng VMware

**Hiện tượng:** Các máy ảo không thể kết nối với nhau hoặc sử dụng sai mạng nội bộ.

**Nguyên nhân:** Card mạng VMware chưa được cấu hình đúng Network Segment hoặc Internal Network.

**Cách khắc phục:**
- Kiểm tra lại card mạng của từng máy.
- Đảm bảo các máy sử dụng đúng mạng theo sơ đồ LAB10.
- Kiểm tra địa chỉ IP và gateway.
- Kiểm tra quy tắc firewall trên pfSense.

### 6.2. Lỗi DNS không phân giải được tên miền

**Hiện tượng:** VICTIM không phân giải được `www.example.lab10.test`.

**Nguyên nhân:** Có thể do cấu hình DNS client, BIND9 hoặc kết nối đến AUTH-DNS.

**Cách khắc phục:**
- Kiểm tra DNS Server trên VICTIM.
- Kiểm tra trạng thái BIND9.
- Kiểm tra cấu hình DNS baseline.
- Kiểm tra kết nối đến AUTH-DNS.
- Kiểm tra log DNS để xác định lỗi.

### 6.3. Lỗi kết nối website

**Hiện tượng:** VICTIM không truy cập được website.

**Nguyên nhân:** Có thể do dịch vụ web, DNS hoặc firewall.

**Cách khắc phục:**
- Kiểm tra dịch vụ Nginx trên DMZ-WEB.
- Kiểm tra kết quả phân giải DNS.
- Kiểm tra quy tắc firewall cho HTTP.
- Kiểm tra kết nối đến DMZ-WEB.

### 6.4. Lỗi không quan sát được lưu lượng DNS trên Kali

**Hiện tượng:** Kali không thấy lưu lượng DNS giữa DNS-SERVER và AUTH-DNS.

**Nguyên nhân:** Kali nằm trong ATTACKNET, tách biệt với DNSLAB và DMZ.

**Cách khắc phục:**
- Kiểm tra cấu hình card mạng Kali.
- Kiểm tra các quy tắc firewall.
- Xác định đúng interface dùng để bắt gói.
- Đối chiếu với sơ đồ mạng của LAB10.

### 6.5. Lỗi khi khôi phục BIND9

**Hiện tượng:** DNS Resolver không hoạt động bình thường sau khi mô phỏng.

**Cách khắc phục:**
- Kiểm tra cấu hình BIND9.
- Kiểm tra trạng thái dịch vụ.
- Đối chiếu cấu hình baseline.
- Khôi phục snapshot nếu cấu hình không thể phục hồi an
