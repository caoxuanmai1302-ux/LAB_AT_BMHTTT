# LAB 4 – KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên

- Họ tên: Cao Xuân Mai
- MSSV: 1150070028
- Lớp: 11_TTMT
- Tên Lab: LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap

## 2. Phiên bản môi trường

- VMware Workstation: 17.6.2
- Windows Server: Windows Server 2025
- Kali Linux: Kali Linux
- Metasploitable: Metasploitable 2
- Nmap: sử dụng phiên bản được kiểm tra bằng `nmap --version`
- Kiểu mạng: Host-Only
- Network: 192.168.112.0/24

### Địa chỉ IP

| Máy | Vai trò | IP |
|---|---|---|
| Windows Server 2025 | Máy quản lý/kiểm tra | 192.168.112.128 |
| Metasploitable 2 | Máy mục tiêu | 192.168.112.129 |
| Kali Linux | Máy quét | 192.168.112.130 |

## 3. Cách dựng môi trường

1. Tạo 3 máy ảo gồm Windows Server 2025, Kali Linux và Metasploitable 2 trên VMware.
2. Cấu hình cả 3 máy sử dụng mạng Host-Only.
3. Kiểm tra các máy nằm trong cùng mạng 192.168.112.0/24.
4. Kiểm tra kết nối giữa Kali và Metasploitable 2.
5. Kiểm tra và cài đặt Nmap trên Kali.
6. Tạo snapshot môi trường trước khi thực hiện LAB.

## 4. Các tình huống đã thực hiện

- Kiểm tra phiên bản Nmap.
- Xác định địa chỉ IP và kiểm tra kết nối.
- Host Discovery bằng `-sn`.
- TCP Connect Scan bằng `-sT`.
- SYN Scan bằng `-sS`.
- FIN, Xmas và NULL Scan.
- ACK Scan.
- UDP Scan.
- Service/Version Detection bằng `-sV`.
- OS Detection bằng `-O`.
- Aggressive Scan bằng `-A`.
- NSE kiểm tra thông tin SMB.
- NSE kiểm tra MS17-010.
- Xuất kết quả Nmap dạng TXT, XML và grepable.
- Hardening Windows Server bằng Firewall và so sánh kết quả Before/After.

## 5. Kết quả PASS/FAIL

| Nội dung | Kết quả |
|---|---|
| Kiểm tra Nmap | PASS |
| Xác định IP và kết nối | PASS |
| Host Discovery | PASS |
| TCP Scan | PASS |
| FIN/Xmas/NULL/ACK Scan | PASS |
| UDP Scan | PASS |
| Service Detection | PASS |
| OS Detection | PASS |
| Aggressive Scan | PASS |
| NSE SMB | PASS |
| Xuất kết quả | PASS |
| Hardening Firewall | PASS |
| So sánh Before/After | PASS |

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1: Sai tùy chọn Nmap

Đã nhập:

`sudo nmap -sv 192.168.112.129 -on before.txt`

Nmap báo lỗi do tùy chọn có phân biệt chữ hoa chữ thường.

Cách khắc phục:

`sudo nmap -sV 192.168.112.128 -oN before.txt`

### Lỗi 2: Xác định nhầm máy mục tiêu khi hardening

Ban đầu cần kiểm tra Windows Server 2025 nên phải sử dụng IP:

`192.168.112.128`

Metasploitable 2 sử dụng:

`192.168.112.129`

Sau khi kiểm tra lại sơ đồ IP, thực hiện quét đúng máy Windows Server.

### Lỗi 3: Cổng 5985 vẫn truy cập được trước khi hardening

Trước hardening, Nmap phát hiện:

`5985/tcp open`

Đã tạo rule Firewall `LAB4_Block_5985` trên Windows Server để chặn cổng 5985.

Sau khi quét lại, kết quả:

- Open: 1 → 0
- Filtered: 999 → 1000
- 5985/tcp: open → filtered

## 7. Kết luận

Bài LAB 4 đã hoàn thành các nội dung khảo sát bề mặt mạng bằng Nmap trong môi trường Host-Only. Kết quả Before/After cho thấy việc cấu hình Firewall đã làm cổng 5985 chuyển từ trạng thái open sang filtered, qua đó giảm khả năng truy cập dịch vụ từ máy quét.