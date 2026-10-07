# Bài 3: Cấu hình tường lửa UFW và chuẩn đoán cổng mạng

## Mục tiêu
* Sử dụng công cụ tường lửa UFW (Uncomplicated Firewall) để bảo vệ máy chủ.
* Thiết lập quy tắc chặn/mở cổng mạng phục vụ cho việc deploy ứng dụng một cách an toàn.
* Sử dụng các công cụ chẩn đoán mạng (`ss`, `curl`, `ufw status`) để kiểm tra cổng kết nối.

## Yêu cầu
**Bối cảnh:** Chuẩn bị triển khai một ứng dụng web lắng nghe tại cổng 8080 trên máy chủ Cloud VPS. Để bảo vệ máy chủ, cần cấu hình tường lửa chỉ mở các cổng thực sự cần thiết.
**Ràng buộc:**
* Cấu hình chính sách mặc định của UFW: chặn toàn bộ kết nối đi vào, cho phép kết nối đi ra.
* Mở cổng kết nối SSH (port 22) để không bị mất quyền quản trị.
* Mở cổng kết nối ứng dụng Web (port 8080/tcp).
* Kích hoạt tường lửa UFW và kiểm tra.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Thiết lập quy tắc mặc định và mở cổng
```bash
# Thiết lập chặn đầu vào, cho phép đầu ra
$ sudo ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)

$ sudo ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)

# Thêm quy tắc cho phép SSH và ứng dụng Web
$ sudo ufw allow 22/tcp
Rule added
Rule added (v6)

$ sudo ufw allow 8080/tcp
Rule added
Rule added (v6)
```

### Bước 2: Kích hoạt tường lửa
```bash
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

### Bước 3: Kiểm tra trạng thái tường lửa (Minh chứng)
Sử dụng lệnh kiểm tra chi tiết để xác nhận rằng UFW đang hoạt động (`active`) và các cổng đã được mở chính xác:
```bash
$ sudo ufw status verbose
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
8080/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
8080/tcp (v6)              ALLOW IN    Anywhere (v6)
```

### Bước 4: Kiểm tra các cổng đang lắng nghe thực tế bằng công cụ `ss`
```bash
$ sudo ss -tlnp
State    Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    Process
LISTEN   0         128                0.0.0.0:22               0.0.0.0:*        users:(("sshd",pid=1024,fd=3))
LISTEN   0         511                0.0.0.0:8080             0.0.0.0:*        users:(("node",pid=2048,fd=18))
LISTEN   0         128                   [::]:22                  [::]:*        users:(("sshd",pid=1024,fd=4))
```
*(Xác nhận: Tường lửa đang hoạt động chính xác với thiết lập chỉ cho phép cổng 22 và 8080. Ứng dụng Node.js (ví dụ) và SSH đang trực tiếp lắng nghe (LISTEN) trên các port này).*
