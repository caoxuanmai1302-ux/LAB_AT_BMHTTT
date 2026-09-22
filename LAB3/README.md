# LAB 3: NHẬN DIỆN VÀ ỨNG PHÓ CÁC MỐI ĐE DỌA ĐẾN AN TOÀN THÔNG TIN

## 1. Thông tin sinh viên

* **Họ và tên:** Cao Xuân Mai 
* **MSSV:** 1150070028
* **Tên bài lab:** Lab 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 2. Môi trường thực hành

| Thành phần         | Phiên bản / thông tin                                                   |
| ------------------ | ----------------------------------------------------------------------- |
| Máy thật           | Windows 11 Home                                                         |
| Phần mềm ảo hóa    | VMware Workstation Pro                                           |
| Máy ảo             | Windows Server 2025                                                     |
| Mạng máy ảo        | NAT trong quá trình cài đặt; chuyển về Host-only theo yêu cầu thực hành |
| Công cụ            | Sysmon, Autoruns, Process Explorer, Wireshark                           |
| Thư mục lab        | `C:\LAB3`                                                               |
| Thư mục bằng chứng | `C:\LAB3\Evidence`                                                      |

## 3. Cách dựng môi trường

1. Cài đặt VMware Workstation Pro trên máy thật.
2. Tạo máy ảo Windows Server 2025.
3. Cấu hình card mạng theo sơ đồ thực hành của giảng viên (Host-only).
4. Tạo thư mục `C:\LAB3` và các thư mục chứa công cụ, dữ liệu mẫu, script và bằng chứng.
5. Chuẩn bị các công cụ Sysmon, Autoruns, Process Explorer và Wireshark.
6. Sử dụng các tệp dữ liệu và script mẫu được cung cấp trong bộ tài nguyên LAB3.
7. Lưu ảnh chụp và kết quả thực hành vào `C:\LAB3\Evidence`.

## 4. Các tình huống đã thực hiện

| Tình huống | Nội dung                                                             | Kết quả                                                                                                  |
|TH1         |Baseline, risk register và phân loại 5 nguồn đe dọa                   |Đã thu thập baseline
|TH2         |Kiểm tra EICAR và khả năng phát hiện/cách ly của Defender             |có bằng chứng detection EICAR
| TH3        | Kiểm tra và phân tích sự kiện đăng nhập Windows                      | Đã thực hiện một phần; có lưu `auth_events_before_rotation.txt`                                          |
| TH4        | Quan sát tiến trình và dấu vết persistence bằng công cụ Sysinternals | Đã thực hiện; có kiểm tra Autoruns và lưu `autoruns_after.csv`, `sysmon_persistence.txt`, `task_ran.txt` |
| TH5        | Thực hành bắt và phân tích lưu lượng mạng                            | [Cập nhật PASS/FAIL sau khi hoàn thành]                                                                  |
| TH6        | Phân tích dữ liệu mẫu DDoS và kết quả kiểm thử tải cục bộ            | Đã thực hiện phần thống kê dữ liệu mẫu; có `ddos_sources.txt`, `local_load_test.txt`                     |
| TH7        | Phân tích dữ liệu mẫu mailbomb và kỹ thuật xã hội                    | Đã thực hiện thống kê mail log; có `mail_sender_counts.txt`, `mail_volume.txt`                           |

## 5. Kết quả PASS/FAIL
* **TH1:** Có các file baseline; cần xác nhận đã hoàn thành risk register và phân loại 5 nguồn đe dọa.
* **TH2:** detection EICAR trong Protection history hoặc Get-MpThreatDetection.
* **TH3:** Đã thu thập một phần bằng chứng sự kiện đăng nhập. Việc đăng nhập bằng tài khoản lab gặp lỗi phải đăng nhập login ra máy ảo; cần ghi rõ trong báo cáo.
* **TH4:** Đã quan sát tiến trình và kiểm tra artefact persistence bằng Autoruns.
* **TH5:** Chưa xác nhận hoàn tất capture.
* **TH6:** Đã có kết quả thống kê dữ liệu mẫu và file kết quả kiểm thử tải cục bộ.
* **TH7:** Đã có kết quả thống kê người gửi và dung lượng mail log mẫu.
* **Cleanup/Recovery:** [Cập nhật sau khi chạy verify và chụp H11].

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1: Không đăng nhập được bằng tài khoản lab

* **Hiện tượng:** Lệnh `runas /user:.\lab3user cmd` báo `RUNAS ERROR: Unable to acquire user password`.
* **Cách xử lý:** Ghi nhận lỗi và tiếp tục các phần thực hành không phụ thuộc vào phiên đăng nhập đó; tiến hành đăng nhập thông qua login logout trên máy ảo luôn và thành công.

### Lỗi 2: Không tìm thấy file `autoruns_before.csv`

* **Hiện tượng:** Lệnh `Compare-Object` báo không tìm thấy `C:\LAB3\Evidence\autoruns_before.csv`.
* **Nguyên nhân:** Chưa có file baseline Autoruns trước cleanup.
* **Cách xử lý:** Ghi nhận thiếu baseline; không tạo kết quả so sánh trước–sau khi chưa có dữ liệu đầu vào.

### Lỗi 3: Lệnh PowerShell `Format-Table - Wrap` bị lỗi

* **Hiện tượng:** PowerShell báo lỗi tham số `Wrap`.
* **Nguyên nhân:** Có khoảng trắng giữa dấu `-` và tên tham số.
* **Cách khắc phục:** Sử dụng cú pháp đúng `Format-Table -Wrap`.

### Lỗi 4: Wireshark tải chậm

* **Hiện tượng:** Quá trình tải bằng `winget` diễn ra chậm trong máy ảo.
* **Cách xử lý:** Kiểm tra trạng thái tiến trình tải và kết nối mạng; không xác nhận cài đặt thành công khi chưa kiểm tra được.

### Lỗi 5: Lệnh xuất thống kê mail bị lỗi

* **Hiện tượng:** Nhập `Count,Name |` trực tiếp khiến PowerShell báo lỗi cú pháp.
* **Cách khắc phục:** Lấy các thuộc tính từ kết quả `Group-Object` bằng `Select-Object Count, Name` trước khi xuất bảng hoặc lưu file.

## 7. Danh sách bằng chứng

Các file kết quả được lưu trong `C:\LAB3\Evidence`, gồm:

* `auth_events_before_rotation.txt`
* `autoruns_after.csv`
* `baseline_defender.txt`
* `baseline_firewall.txt`
* `baseline_network.txt`
* `baseline_os.txt`
* `baseline_processes.txt`
* `ddos_sources.txt`
* `local_load_test.txt`
* `mail_sender_counts.txt`
* `mail_volume.txt`
* `network.txt`
* `processes.txt`
* `start_time.txt`
* `sysmon_persistence.txt`
* `systeminfo.txt`
* `task_ran.txt`
* `evidence_sha256.csv`

Các file cần được rà soát nội dung trước khi công khai trên GitHub. Ảnh minh chứng được đặt theo tên H1–H11 tương ứng với báo cáo.

## 8. Cleanup và phục hồi

Sau khi thu thập đầy đủ bằng chứng:

1. Thực hiện cleanup các artefact do bài lab tạo.
2. Kiểm tra lại persistence, listener cổng 8080 và trạng thái Windows Defender.
3. Chụp ảnh H11 – `H11_Recovery_Verification.png`.
4. Tạo hoặc cập nhật `evidence_sha256.csv` sau khi chốt các tệp bằng chứng.
5. Khôi phục máy ảo về snapshot `LAB3_CLEAN_20260914` hoặc snapshot sạch do giảng viên quy định.

## 9. Kết luận

Bài thực hành giúp sinh viên làm quen với việc nhận diện dấu vết kỹ thuật, phân tích dữ liệu mẫu, sử dụng công cụ giám sát và thực hiện quy trình cleanup, kiểm tra phục hồi. Các kết quả PASS/FAIL được cập nhật theo bằng chứng thực tế thu được trong quá trình thực hành.
