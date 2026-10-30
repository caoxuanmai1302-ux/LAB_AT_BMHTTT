# README.md -- LAB9: Local DNS Attack & Defense

## 1. Thông tin sinh viên

-   **Họ và tên:** Cao Xuân Mai
-   **MSSV:** 1150070028
-   **Lớp:** 11_TTMT
-   **Tên bài lab:** LAB9 -- Local DNS Attack & Defense

## 2. Phiên bản môi trường

  Thành phần            Phiên bản / cấu hình
  --------------------- ------------------------------------------
  Nền tảng ảo hóa       VMware Workstation --  17.6.2
  pfSense: 2.7.x
  DNS-SERVER            Ubuntu Server -- 24.04
  DNS service           BIND9 -- 
  VICTIM                Ubuntu -- 24.04
  ATTACKER              Kali Linux --
  APP-TEST / AUTH-DNS   Python 3 -- 
  Công cụ bắt gói       Wireshark -- 

## 3. Sơ đồ mạng và địa chỉ IP

  ------------------------------------------------------------------------
  Máy / Thành phần                        Địa chỉ IP Vai trò
  --------------------- ---------------------------- ---------------------
  pfSense -- DNSLAB LAN                `10.9.0.1/24` Gateway DNS-LAB

  DNS-SERVER                          `10.9.0.10/24` BIND9 resolver/cache

  VICTIM                             `10.9.0.100/24` Máy khách truy vấn
                                                     DNS

  ATTACKER (Kali)                    `10.9.0.200/24` Máy quan sát và chạy
                                                     kịch bản lab

  pfSense -- DMZ                     `172.16.6.1/24` Gateway DMZ

  DMZ-WEB                           `172.16.6.10/24` Web thử nghiệm

  APP-TEST / AUTH-DNS               `172.16.6.20/24` DNS authoritative giả
                                                     lập
  ------------------------------------------------------------------------

Tên miền thử nghiệm: - `www.lab9.test` - `www.external.lab9.test`

Kết quả phân giải hợp lệ theo kịch bản: - `www.external.lab9.test` →
`172.16.6.10`

**Phạm vi an toàn:** Chỉ thực hiện trong mạng DNS-LAB và trên các máy ảo
của bài thực hành. Không chạy kịch bản trên Wi-Fi, mạng trường học, mạng
công ty hoặc Internet công cộng.

## 4. Cách dựng môi trường

1.  Tạo/cấu hình các máy ảo theo bộ tài liệu LAB9 trên VMware.
2.  Kết nối các card mạng vào đúng mạng ảo DNS-LAB và DMZ theo sơ đồ.
3.  Cấu hình IP tĩnh theo bảng địa chỉ ở mục 3.
4.  Cấu hình pfSense làm gateway và định tuyến giữa các mạng theo yêu
    cầu của tài liệu.
5.  Trên DNS-SERVER, cài và cấu hình BIND9, khai báo zone nội bộ và
    forward zone `external.lab9.test` theo tài liệu.
6.  Trên APP-TEST, chuẩn bị chương trình `auth_dns_delayed.py`.
7.  Trên Kali, chuẩn bị Wireshark và các tài nguyên thử nghiệm được cung
    cấp trong thư mục LAB9.
8.  Trên VICTIM, cấu hình DNS trỏ về `10.9.0.10`.

### Lệnh kiểm tra sau khi dựng

Trên các máy Linux:

``` bash
ip -br addr
ip route
```

Từ VICTIM:

``` bash
ping -c 3 10.9.0.10
dig @10.9.0.10 www.external.lab9.test A +time=3 +tries=1
```

Kết quả baseline mong đợi:

``` text
www.external.lab9.test. ... IN A 172.16.6.10
```

Trên DNS-SERVER:

``` bash
sudo systemctl status bind9 --no-pager
sudo ss -lunpt | grep ':53'
sudo named-checkconf
sudo named-checkzone lab9.test /etc/bind/db.lab9.test
```

Chỉ thực hiện các tình huống tấn công mô phỏng khi DNS baseline hoạt
động đúng.

## 5. Các tình huống đã thực hiện và kết quả

> Cập nhật cột PASS/FAIL theo bằng chứng thực tế (ảnh Wireshark, đầu ra
> lệnh, file cache). Không đánh dấu PASS chỉ vì đã chạy lệnh.

  ---------------------------------------------------------------------------------
  STT               Tình huống        Kết quả mong đợi / tiêu chí Trạng thái
                                      kiểm tra                    
  ----------------- ----------------- --------------------------- -----------------
  1                 DNS baseline      VICTIM truy vấn             \[PASS
                                      `www.external.lab9.test`,   xác nhận\]
                                      nhận `172.16.6.10`          

  2                 Task 1 --         Phân biệt kết quả tra cứu   \[PASS
                    HOSTS-file        qua `/etc/hosts` với truy   xác nhận\]
                    redirection       vấn DNS trực tiếp bằng      
                                      `dig`                       

  3                 Task 2 -- Direct  Wireshark ghi nhận truy vấn \[PASS
                    DNS Spoofing      và các phản hồi; đối chiếu  xác nhận\]
                                      transaction ID, port và A   
                                      record                      

  4                 Task 3 -- DNS     Kiểm tra nội dung cache     \[PASS
                    Cache Poisoning   BIND9 và câu trả lời DNS    xác nhận\]
                                      sau truy vấn thử nghiệm     

  5                 Khôi phục môi     Dừng các script, flush      \[PASS
                    trường            cache, phục hồi hosts, dừng xác nhận\]
                                      APP-TEST và đưa Promiscuous 
                                      Mode về Deny                

  6                 Hardening DNS     Xóa ngoại lệ                \[PASS
                    COOKIE            `send-cookie no`, restart   xác nhận\]
                                      BIND9 và lặp lại Task 3 để  
                                      so sánh                     
  ---------------------------------------------------------------------------------

### Wireshark display filters đã sử dụng

``` text
dns
udp.port == 53
ip.addr == 10.9.0.100 && dns
ip.addr == 10.9.0.10 && dns
ip.addr == 172.16.6.20 && dns
dns.qry.name == "www.external.lab9.test"
dns.flags.response == 1 && dns.a == 10.9.0.200
```

### Lệnh kiểm tra cache BIND9

``` bash
sudo rndc dumpdb -cache
sudo find /var/cache/bind /var/lib/bind -name 'named_dump.db' -ls 2>/dev/null
sudo grep -A 8 -B 3 'www.external.lab9.test' /var/cache/bind/named_dump.db
```

`rndc dumpdb -cache` thường không in dữ liệu cache ra Terminal mà ghi ra
file dump. Vì vậy, không có dòng output ngay sau lệnh không đồng nghĩa
lệnh thất bại.

## 6. Lỗi gặp phải và cách khắc phục

  ---------------------------------------------------------------------------------
  Lỗi / hiện tượng        Nguyên nhân cần kiểm    Cách xử lý / xác minh
                          tra                     
  ----------------------- ----------------------- ---------------------------------
  VICTIM báo `SERVFAIL`   Resolver không phân     Kiểm tra `dig @10.9.0.10 ...`,
  khi chạy                giải thành công; có thể trạng thái BIND9, log và kết nối
  `resolvectl query`      do BIND9, forward zone, tới APP-TEST
                          DNSSEC hoặc APP-TEST    

  `dig @10.9.0.10 ...` bị DNS-SERVER không phản   Kiểm tra ping,
  timeout                 hồi qua UDP/TCP 53,     `sudo ss -lunpt \| grep ':53'`,
                          dịch vụ không lắng nghe `systemctl status bind9` và
                          hoặc firewall/rule chặn firewall
                          DNS                     

  Ping VICTIM tới         Ping chỉ xác nhận ICMP; Kiểm tra riêng dịch vụ BIND9 và
  DNS-SERVER thành công   không chứng minh DNS    lưu lượng DNS trên cổng 53
  nhưng DNS timeout       port 53 hoạt động       

  `rndc dumpdb -cache`    Lệnh xuất cache ra      Tìm `named_dump.db`, sau đó dùng
  không hiện output       file, không in nội dung `grep` để kiểm tra bản ghi
                          trực tiếp               

  Không thấy gói DNS mong Có thể chọn sai         Chọn interface DNS-LAB và kiểm
  muốn trong Wireshark    interface hoặc filter   tra filter `dns` trước, sau đó
                          chưa phù hợp            thu hẹp filter

  Không tìm thấy script   Script nằm trong thư    Tìm đúng file trong thư mục
  trong thư mục hiện tại  mục tài nguyên được     `LAB9_Resources` đã mount trên
                          chia sẻ                 máy Kali
  ---------------------------------------------------------------------------------

### Lệnh chẩn đoán tham khảo

Trên VICTIM:

``` bash
resolvectl status
dig @10.9.0.10 www.external.lab9.test A +time=3 +tries=1
ping -c 3 10.9.0.10
```

Trên DNS-SERVER:

``` bash
sudo systemctl status bind9 --no-pager
sudo ss -lunpt | grep ':53'
sudo named-checkconf
sudo journalctl -u bind9 -n 40 --no-pager
```

## 7. Khôi phục môi trường

Thực hiện theo thứ tự tài liệu:

1.  Dừng `dns_spoof_lab9.py` và fake HTTP server trên Kali bằng
    `Ctrl + C` tại Terminal tương ứng.

2.  Trên DNS-SERVER, flush cache BIND9:

    ``` bash
    sudo rndc flush
    ```

3.  Trên VICTIM, flush cache resolver:

    ``` bash
    sudo resolvectl flush-caches
    ```

4.  Khôi phục `/etc/hosts` từ đúng file backup đã tạo. Ví dụ nếu file
    backup tên `/etc/hosts.bak`:

    ``` bash
    sudo cp /etc/hosts.bak /etc/hosts
    ```

    Kiểm tra tên file backup trước khi chạy; không sử dụng đường dẫn ví
    dụ nếu file thực tế có tên khác.

5.  Trên APP-TEST, dừng `auth_dns_delayed.py` bằng `Ctrl + C`.

6.  Đưa Promiscuous Mode của Kali về `Deny` sau buổi thực hành.

7.  Giữ DNSLAB interface nếu bài sau cần dùng; nếu không, disable OPT2
    hoặc lưu snapshot `LAB9_DONE` theo tài liệu.

Kiểm tra sau khôi phục trên VICTIM:

``` bash
dig @10.9.0.10 www.external.lab9.test A
resolvectl query www.external.lab9.test
```

Kết quả DNS trực tiếp mong đợi: `172.16.6.10`.

## 8. Hardening minh họa

  -----------------------------------------------------------------------
  Biện pháp                           Ý nghĩa
  ----------------------------------- -----------------------------------
  DNS egress control                  Chỉ cho client truy vấn DNS tới
                                      resolver được chỉ định

  Segmentation                        Phân đoạn mạng để máy không tin cậy
                                      không dễ dàng cùng lớp 2 với DNS
                                      client/server

  Promiscuous Mode                    Đưa về `Deny` sau khi hoàn tất bắt
                                      gói

  DNSSEC validation                   Xác minh tính xác thực/toàn vẹn cho
                                      zone có ký DNSSEC; không bảo vệ
                                      zone `.test` chưa ký

  DNS COOKIE / source-port            Bổ sung cơ chế làm khó giả mạo phản
  randomization                       hồi DNS

  Monitoring                          Theo dõi phản hồi trùng lặp, thay
                                      đổi cache bất thường và câu trả lời
                                      không khớp chính sách
  -----------------------------------------------------------------------

### Bài tập phòng thủ DNS COOKIE

Theo tài liệu, xóa ngoại lệ sau khỏi `/etc/bind/named.conf.options` nếu
đang tồn tại:

``` conf
server 172.16.6.20 {
    send-cookie no;
};
```

Sau đó:

``` bash
sudo named-checkconf
sudo systemctl restart bind9
sudo systemctl status bind9 --no-pager
sudo rndc flush
```

Lặp lại Task 3 và ghi nhận sự khác biệt thực tế. DNS COOKIE là một lớp
phòng vệ bổ sung, không phải cơ chế duy nhất và không bảo đảm ngăn chặn
mọi tình huống giả mạo DNS.


## 9. Kết luận

Bài thực hành giúp tìm hiểu quá trình phân giải DNS, vai trò của
resolver cache và nguy cơ khi client hoặc resolver tiếp nhận dữ liệu DNS
không đáng tin cậy. Wireshark cùng các công cụ kiểm tra DNS hỗ trợ quan
sát luồng truy vấn, phân tích phản hồi và xác minh trạng thái cache.

Các biện pháp DNS egress control, segmentation, DNSSEC, DNS COOKIE,
source-port randomization và monitoring cần được kết hợp để tăng cường
bảo vệ hệ thống DNS.

