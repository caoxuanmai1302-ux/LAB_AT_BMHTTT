
# LAB3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 1. Thông tin sinh viên

- **Họ và tên:** Cao Xuân Mai
- **MSSV:** [Bổ sung MSSV]
- **Tên lab:** LAB3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 2. Phiên bản môi trường thực hành

| Thành phần | Thông tin |
|---|---|
| Máy thật | Windows 11 Home |
| Phần mềm ảo hóa | VMware Workstation Pro 17.6.2 (Build 24409262) |
| Máy ảo | Windows Server 2025 |
| RAM máy thật | 16 GB |
| Địa chỉ IPv4 máy ảo | 192.168.80.133 |
| Sysmon | Chưa xác nhận phiên bản và trạng thái hoạt động |
| Autoruns | Chưa xác nhận |
| Process Explorer | Chưa xác nhận |

## 3. Cách dựng môi trường

1. Sử dụng VMware Workstation Pro 17.6.2 để chạy máy ảo Windows Server 2025.
2. Kiểm tra trạng thái Windows Defender và Windows Firewall.
3. Chuẩn bị tài nguyên thực hành tại thư mục `C:\LAB3`.
4. Chuẩn bị tệp cấu hình Sysmon `sysmon-lab.xml`.
5. Thực hiện các tình huống trong phạm vi máy ảo và thu thập bằng chứng.

## 4. Các tình huống đã thực hiện

| Tình huống | Nội dung / trạng thái |
|---|---|
| TH1 | làm được kết quả trong word|
| TH2 | làm được kết quả trong word|
| TH3 | Đã tạo tài khoản thực hành `lab3user`. Đang gặp lỗi khi sử dụng `runas`, chưa hoàn tất kiểm tra đăng nhập bằng tài khoản này. |
| TH4 | Đã kiểm tra đường dẫn Sysmon và tìm thấy tệp `sysmon-lab.xml`. Lệnh kiểm tra service Sysmon chưa hiển thị kết quả; chưa xác nhận Sysmon hoạt động. |
| TH5 | làm hỏng kịp |
| TH6 | làm hỏng kịp  |
| TH7 | làm hỏng kịp  |

## 5. Kiểm tra an toàn hệ thống

- Windows Defender: AntivirusEnabled = True.
- Windows Defender: RealTimeProtectionEnabled = True.
- Windows Defender: AMServiceEnabled = True.
- Windows Firewall: Domain, Private và Public đều được bật.
- Tamper Protection: Trạng thái kiểm tra là False.

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1: Không đăng nhập được bằng runas

- **Hiện tượng:** Lệnh `runas /user:.\lab3user cmd` báo `RUNAS ERROR: Unable to acquire user password`.
- **Trạng thái:** Chưa khắc phục được hoàn toàn.
- **Ghi chú:** Tài khoản `lab3user` đã được tạo và kiểm tra là đang hoạt động.

### Lỗi 2: Không tìm thấy service Sysmon

- **Hiện tượng:** Lệnh `Get-Service Sysmon*` không hiển thị kết quả.
- **Kiểm tra:** Tệp `sysmon-lab.xml` được tìm thấy tại:
  - `C:\LAB3\Downloads\LAB3_Threats_Assets\lab3_assets\sysmon-lab.xml`
  - `C:\LAB3\LAB3_Threats_Assets\lab3_assets\sysmon-lab.xml`
- **Trạng thái:** Chưa xác nhận Sysmon đã được cài đặt và hoạt động.


7. Kết quả PASS/FAIL

- **PASS:** Đã kiểm tra được trạng thái Windows Defender và Windows Firewall; đã tìm thấy tệp cấu hình Sysmon.
- **FAIL / Chưa hoàn tất:** Đăng nhập bằng `runas` chưa thành công; Sysmon chưa xác nhận hoạt động.
- **Chưa thực hiện / chưa xác nhận:** Các tình huống còn lại.

8 Quy định lưu trữ bằng chứng

- Chỉ đưa báo cáo, README.md, log/output đã làm sạch và `evidence_sha256.csv` vào repository.
- Không đưa installer hoặc executable của Sysinternals, Wireshark, Python vào repository.
- Không đưa tệp bị Windows Defender quarantine vào repository.
- Không công khai mật khẩu, token hoặc thông tin nhạy cảm.
- Ghi hash SHA-256 cho các tệp bằng chứng được lưu trữ.