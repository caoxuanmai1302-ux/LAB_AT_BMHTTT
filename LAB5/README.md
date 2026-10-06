# LAB 5 - XÂY DỰNG VÀ CẤU HÌNH FIREWALL PFSENSE

## 1. Thông tin sinh viên

- Họ và tên: Cao Xuân Mai
- MSSV: 1150070028
- Lớp: 11_TTMT
- Tên Lab: LAB 5 - Xây dựng và cấu hình Firewall pfSense
- Phạm vi thực hiện: Tình huống 1, Tình huống 2 và Tình huống 3

---

## 2. Phiên bản môi trường

- VMware Workstation: 17.6.2
- pfSense: 2.7.x
- Domain Controller: Windows Server 2022
- DMZ-Web: Windows Server 2025
- LAN-Test: Ubuntu Server 24.04 LTS
- Mạng WAN: Bridged
- Mạng LAN: VMware Host-only VMnet1
- Mạng DMZ: VMware Host-only VMnet2

---

## 3. Cấu hình hệ thống

### 3.1. pfSense

pfSense được cấu hình với 3 card mạng:

| Interface | Kết nối | Địa chỉ IP |
|---|---|---|
| WAN | Bridged | DHCP |
| LAN | VMnet1 | 10.0.0.1/8 |
| DMZ (OPT1) | VMnet2 | 172.16.0.1/16 |

### 3.2. Domain Controller - Windows Server 2022

- Hệ điều hành: Windows Server 2022
- Vai trò: Domain Controller
- IP: `10.0.0.2/8`
- Gateway: `10.0.0.1`
- DNS: `10.0.0.2`
- Kết nối mạng: VMnet1

### 3.3. LAN-Test - Ubuntu Server 24.04

- Hệ điều hành: Ubuntu Server 24.04 LTS
- IP: `10.0.0.3/8`
- Gateway: `10.0.0.1`
- Kết nối mạng: VMnet1

### 3.4. DMZ-Web - Windows Server 2025

- Hệ điều hành: Windows Server 2025
- Vai trò: Web Server trong vùng DMZ
- IP: `172.16.0.2/16`
- Gateway: `172.16.0.1`
- DNS: `8.8.8.8`
- Kết nối mạng: VMnet2
- IIS được cài đặt để kiểm tra dịch vụ Web.

---

## 4. Mô hình mạng

```text
                         INTERNET
                            |
                         WAN - DHCP
                            |
                     +--------------+
                     |    pfSense   |
                     +--------------+
                       /          \
                      /            \
                    LAN            DMZ
             10.0.0.0/8       172.16.0.0/16
                  |                  |
          +-------+------+     +-----+------+
          |              |     |            |
     Windows Server   Ubuntu  Windows Server
        2022          Server      2025
         DC           LAN-Test    DMZ-Web
      10.0.0.2        10.0.0.3   172.16.0.2
```

---

# 5. Các tình huống đã thực hiện

## 5.1. Tình huống 1 - Kiểm soát truy cập Internet từ LAN

### Mục tiêu

Kiểm soát quyền truy cập Internet của các máy trong mạng LAN bằng firewall pfSense.

### Cấu hình

Tắt rule mặc định:

```text
LAN net → Any
```

Tạo 3 rule:

```text
1. BLOCK ICMP
   Source: LAN net
   Destination: Any
   Protocol: ICMP

2. PASS DNS
   Source: LAN net
   Destination: Any
   Protocol: TCP/UDP
   Destination Port: 53

3. PASS HTTP/HTTPS
   Source: LAN net
   Destination: Any
   Protocol: TCP
   Destination Port: 80, 443
```

### Kiểm tra

Kiểm tra Ping:

```cmd
ping 8.8.8.8
```

Kết quả:

```text
FAIL - Ping bị firewall chặn
```

Kiểm tra DNS:

```powershell
Resolve-DnsName example.com -Server 8.8.8.8
```

Kết quả:

```text
PASS - Phân giải DNS thành công
```

Kiểm tra HTTPS:

```cmd
curl.exe -4 https://example.com
```

Kết quả:

```text
PASS - Kết nối HTTPS thành công
```

### Kết quả

**PASS**

---

## 5.2. Tình huống 2 - Chỉ cho phép Domain Controller ra Internet

### Mục tiêu

Chỉ cho phép Domain Controller có địa chỉ IP `10.0.0.2` truy cập Internet.

Các máy khác trong mạng LAN bị chặn truy cập Internet.

### Cấu hình

Tắt rule:

```text
LAN net → Any
```

Tạo 2 rule theo thứ tự:

```text
1. PASS
   Source: 10.0.0.2
   Destination: Any
   Protocol: Any

2. BLOCK
   Source: LAN net
   Destination: Any
   Protocol: Any
```

Rule cho Domain Controller được đặt phía trên rule BLOCK.

### Kiểm tra Domain Controller

Trên Windows Server 2022:

```cmd
ping 8.8.8.8
```

Kết quả:

```text
PASS - Domain Controller được phép truy cập Internet
```

### Kiểm tra LAN-Test

Trên Ubuntu Server:

```bash
ping 8.8.8.8
```

Kết quả:

```text
FAIL - LAN-Test bị chặn truy cập Internet
```

### Kết quả

**PASS**

---

## 5.3. Tình huống 3 - Cô lập DMZ khỏi LAN

### Mục tiêu

Không cho máy trong vùng DMZ truy cập vào mạng LAN, đồng thời vẫn cho phép DMZ truy cập Internet.

### Cấu hình mạng

Mạng LAN:

```text
10.0.0.0/8
```

Mạng DMZ:

```text
172.16.0.0/16
```

DMZ-Web:

```text
172.16.0.2
```

Gateway:

```text
172.16.0.1
```

### Bước 1 - Kiểm tra kết nối ban đầu

Từ DMZ-Web:

```cmd
ping 10.0.0.2
```

Kết nối được sử dụng để kiểm tra trạng thái trước khi áp dụng rule cô lập.

### Bước 2 - Cấu hình rule chặn DMZ → LAN

Tại:

```text
Firewall → Rules → DMZ
```

Tạo rule:

```text
BLOCK
Protocol: Any
Source: DMZ subnets
Destination: LAN subnets
Description: T3 Block DMZ to LAN
```

Giữ rule cho phép DMZ truy cập Internet:

```text
PASS
Protocol: Any
Source: DMZ subnets
Destination: Any
Description: T3 - allow DMZ internet
```

### Thứ tự rule

Rule BLOCK phải nằm phía trên rule PASS:

```text
1. BLOCK  DMZ subnets → LAN subnets
2. PASS   DMZ subnets → Any
```

Sau đó:

```text
Apply Changes
```

và Reset States.

### Bước 3 - Kiểm tra DMZ → LAN

Trên DMZ-Web:

```cmd
ping 10.0.0.2
```

Kết quả:

```text
FAIL - Kết nối bị chặn
```

Điều này chứng minh firewall đã ngăn DMZ truy cập mạng LAN.

### Bước 4 - Kiểm tra DMZ → Internet

Kiểm tra kết nối Internet:

```cmd
ping 8.8.8.8
```

Kiểm tra DNS:

```cmd
nslookup example.com 8.8.8.8
```

Kết quả:

```text
PASS - DMZ-Web truy cập Internet thành công
PASS - Phân giải DNS thành công
```

### Kết quả

**PASS**

---

# 6. Các lỗi gặp phải và cách khắc phục

## 6.1. Windows Firewall trên Domain Controller chặn ICMP

Trong quá trình kiểm tra Tình huống 3, DMZ-Web ban đầu không ping được Domain Controller do Windows Firewall trên Windows Server 2022 chặn ICMP.

Tạo rule ICMP tạm thời:

```cmd
netsh advfirewall firewall add rule name="LAB-Allow-ICMPv4-Echo" protocol=icmpv4:8,any dir=in action=allow
```

Sau khi kiểm tra xong, xóa rule:

```cmd
netsh advfirewall firewall delete rule name="LAB-Allow-ICMPv4-Echo"
```

Sau khi cấu hình lại firewall, việc kiểm tra được thực hiện bình thường.

---

## 6.2. Thứ tự rule trên pfSense

Firewall pfSense xử lý rule theo thứ tự từ trên xuống dưới.

Do đó trong Tình huống 3, rule:

```text
BLOCK DMZ subnets → LAN subnets
```

phải được đặt phía trên:

```text
PASS DMZ subnets → Any
```

Sau khi sắp xếp lại đúng thứ tự, DMZ được cô lập khỏi LAN nhưng vẫn được phép truy cập Internet.

---

# 7. Kết quả tổng hợp

| Tình huống | Nội dung | Kết quả |
|---|---|---|
| Tình huống 1 | Kiểm soát truy cập Internet từ LAN | PASS |
| Tình huống 2 | Chỉ cho phép Domain Controller ra Internet | PASS |
| Tình huống 3 | Cô lập DMZ khỏi LAN và cho phép DMZ ra Internet | PASS |

---

# 8. Kết luận

Đã xây dựng thành công môi trường thực hành firewall pfSense trên VMware với các vùng mạng WAN, LAN và DMZ.

Các tình huống thực hành đã hoàn thành:

- Kiểm soát truy cập Internet từ mạng LAN.
- Chỉ cho phép Domain Controller truy cập Internet.
- Chặn truy cập từ DMZ đến mạng LAN.
- Cho phép DMZ truy cập Internet.
- Kiểm tra và xử lý lỗi Windows Firewall trên Domain Controller.
- Cấu hình đúng thứ tự các rule trên pfSense.

Kết quả cuối cùng:

```text
Tình huống 1: PASS
Tình huống 2: PASS
Tình huống 3: PASS
```

**Tất cả các tình huống đã thực hiện đều đạt yêu cầu.**