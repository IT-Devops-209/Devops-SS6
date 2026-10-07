# Bài 2: Cấu hình phân quyền Nhóm và sudoers bằng visudo

## Mục tiêu
* Biết cách tạo tài khoản người dùng (User) và nhóm (Group) mới trên Linux.
* Hiểu được cơ chế hoạt động của đặc quyền quản trị `sudo` và tệp tin cấu hình `/etc/sudoers`.
* Biết cách phân quyền giới hạn cho một nhóm người dùng chỉ được thực hiện một số lệnh hệ thống nhất định.

## Yêu cầu
**Bối cảnh:** Đội ngũ vận hành (Ops) cần cấp quyền khởi động lại các dịch vụ hệ thống cho nhân viên Deployer tự động hóa, nhưng không được phép cấp toàn quyền root nhằm giảm thiểu rủi ro an ninh.
**Ràng buộc:**
* Tạo nhóm người dùng mới tên là `devops-admin`.
* Tạo tài khoản người dùng mới tên là `deployer`, đưa vào nhóm `devops-admin`.
* Cấu hình `/etc/sudoers` để tài khoản thuộc nhóm `devops-admin` có quyền chạy các lệnh quản lý dịch vụ `systemctl` (start, stop, restart, status) mà không cần mật khẩu (NOPASSWD).

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Tạo nhóm và tài khoản người dùng
Thực hiện chạy các lệnh tạo nhóm và tạo tài khoản `deployer` không cần nhập mật khẩu tương tác (dành cho automation):
```bash
$ sudo groupadd devops-admin

$ sudo adduser --disabled-password --gecos "" deployer
Adding user `deployer' ...
Adding new group `deployer' (1001) ...
Adding new user `deployer' (1001) with group `deployer' ...
Creating home directory `/home/deployer' ...
Copying files from `/etc/skel' ...

# Đưa user deployer vào nhóm devops-admin
$ sudo usermod -aG devops-admin deployer
```

### Bước 2: Cấu hình tệp sudoers an toàn qua visudo
Mở trình soạn thảo visudo:
```bash
$ sudo visudo
```
**Dòng cấu hình đã thêm vào cuối tệp `/etc/sudoers`:**
```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

### Bước 3: Kiểm tra kết quả thực tế
Đăng nhập vào tài khoản `deployer` và kiểm tra quyền được phép thực thi thông qua lệnh `sudo -l`:

```bash
$ su - deployer

$ sudo -l
Matching Defaults entries for deployer on ubuntu-s-1vcpu-1gb-sgp1-01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User deployer may run the following commands on ubuntu-s-1vcpu-1gb-sgp1-01:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Chạy thử lệnh điều khiển dịch vụ hệ thống (Ví dụ: khởi động lại dịch vụ `cron`):
```bash
$ sudo systemctl restart cron
$ systemctl status cron | grep Active
     Active: active (running) since Mon 2026-10-07 18:05:12 UTC; 2s ago
```
*(Xác nhận: Lệnh restart dịch vụ chạy thành công, không hề bị gián đoạn để hỏi mật khẩu nhờ cờ `NOPASSWD`, đáp ứng hoàn hảo yêu cầu cho các kịch bản chạy kịch bản deploy tự động hóa).*
