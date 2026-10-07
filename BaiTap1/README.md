# Bài 1: Khảo sát FHS và Phân quyền File/Folder nâng cao

## Mục tiêu
* Thành thạo việc tạo lập thư mục và điều hướng trong cấu trúc cây thư mục Linux FHS.
* Hiểu và áp dụng thành thạo cơ chế phân quyền tệp tin bằng số (octal) và bằng ký tự (symbolic).
* Nắm rõ cách thức thay đổi quyền sở hữu tệp tin và nhóm sở hữu.

## Yêu cầu
**Bối cảnh:** Triển khai một thư mục dùng chung cho ứng dụng web tại `/var/www/my-app`. Thư mục này cần chứa mã nguồn static công khai và thư mục chứa nhật ký hệ thống (logs) bảo mật.

**Ràng buộc:**
* Tạo cấu trúc: `/var/www/my-app/public` (chứa trang tĩnh) và `/var/www/my-app/logs` (chứa logs).
* Thư mục `public`: phân quyền `750` (Owner: `rwx`, Group: `r-x`, Others: `---`).
* Thư mục `logs`: phân quyền `770` (Owner & Group: `rwx`, Others: `---`).
* Chủ sở hữu là user bình thường (tài khoản: `devops`), nhóm sở hữu là `www-data`.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Khởi tạo cấu trúc thư mục
Đăng nhập vào máy chủ bằng quyền sudo để tạo thư mục nằm ngoài khu vực Home (tại `/var/www/`):
```bash
$ sudo mkdir -p /var/www/my-app/public
$ sudo mkdir -p /var/www/my-app/logs
```

### Bước 2: Thiết lập phân quyền bằng hệ bát phân (Octal)
Gán quyền `750` cho thư mục `public` (Chủ sở hữu có mọi quyền, Nhóm sở hữu chỉ đọc/điều hướng, Người khác không có quyền):
```bash
$ sudo chmod 750 /var/www/my-app/public
```

Gán quyền `770` cho thư mục `logs` (Chủ sở hữu và Nhóm sở hữu có mọi quyền, Người khác không có quyền):
```bash
$ sudo chmod 770 /var/www/my-app/logs
```

### Bước 3: Đổi chủ sở hữu và nhóm sở hữu
Thay đổi Owner thành tài khoản `devops` và Group thành `www-data` (nhóm web mặc định trên Ubuntu), áp dụng đệ quy cho toàn bộ thư mục:
```bash
$ sudo chown -R devops:www-data /var/www/my-app
```

### Bước 4: Kiểm tra kết quả (Minh chứng cấu trúc phân quyền)
Sử dụng lệnh `ls -la` để hiển thị chi tiết quyền (permissions), chủ sở hữu (owner) và nhóm (group):
```bash
$ ls -la /var/www/my-app
total 16
drwxr-xr-x 4 devops www-data 4096 Oct  7 18:00 .
drwxr-xr-x 4 root   root     4096 Oct  7 17:59 ..
drwxrwx--- 2 devops www-data 4096 Oct  7 18:00 logs
drwxr-x--- 2 devops www-data 4096 Oct  7 18:00 public
```

**Đánh giá kết quả thực tế:**
* Thư mục `public`: Có chuỗi quyền `drwxr-x---` (tương đương với 750) -> Đạt yêu cầu.
* Thư mục `logs`: Có chuỗi quyền `drwxrwx---` (tương đương với 770) -> Đạt yêu cầu.
* Phân quyền sở hữu: Owner là `devops` và Group là `www-data` -> Khớp với cấu hình yêu cầu.
