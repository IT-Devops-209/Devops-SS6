# Bài 4: Quản lý tiến trình nền với nohup và tín hiệu Kill

## Mục tiêu
* Biết cách khởi chạy tiến trình chạy nền độc lập với phiên làm việc Terminal bằng lệnh `nohup` và ký tự `&`.
* Thành thạo công cụ giám sát tiến trình hệ thống bằng `ps`, `top/htop`.
* Hiểu rõ cơ chế và sử dụng thành thạo lệnh `kill` với các tín hiệu hệ thống.

## Yêu cầu
**Bối cảnh:** Cần chạy một kịch bản giám sát hệ thống dưới nền và đảm bảo nó hoạt động kể cả khi ngắt kết nối SSH.
**Ràng buộc:**
* Tạo shell script `loop-monitor.sh` ghi thời gian vào `/tmp/monitor.log` mỗi 5s.
* Gán quyền và khởi chạy dưới nền bằng `nohup`.
* Tìm PID và dùng lệnh `kill` gửi tín hiệu SIGTERM (15) để tắt tiến trình.

## Báo cáo thực hành (Các bước thực hiện và Log kiểm tra)

### Bước 1: Tạo kịch bản script giám sát (loop-monitor.sh)
File mã nguồn đã được tạo thành công trong thư mục dự án (xem file `loop-monitor.sh` đính kèm). Cấp quyền thực thi:
```bash
$ chmod +x loop-monitor.sh
```

### Bước 2: Chạy ẩn dưới nền độc lập với nohup
Sử dụng công cụ `nohup` và chuyển hướng (redirect) output chuẩn cùng error vào `/dev/null` để đưa tiến trình hoàn toàn vào chế độ chạy nền (background) tránh chiếm dụng Terminal:
```bash
$ nohup ./loop-monitor.sh > /dev/null 2>&1 &
[1] 10567
```

### Bước 3: Giám sát tiến trình và kiểm tra Log
Kiểm tra tiến trình đang hoạt động thông qua lệnh `ps` lọc với `grep`:
```bash
$ ps aux | grep loop-monitor.sh
devops   10567  0.0  0.1   6812  3156 pts/0    S    18:10   0:00 /bin/bash ./loop-monitor.sh
devops   10582  0.0  0.1   6600  2212 pts/0    S+   18:11   0:00 grep --color=auto loop-monitor.sh
```
*(Xác nhận: Script đang chạy thành công dưới nền với mã tiến trình PID là `10567`)*

Đọc 5 dòng cuối của tệp log liên tục xem ứng dụng có đang ghi bình thường không:
```bash
$ tail -n 5 /tmp/monitor.log
System time: Wed Oct  7 18:10:01 UTC 2026
System time: Wed Oct  7 18:10:06 UTC 2026
System time: Wed Oct  7 18:10:11 UTC 2026
System time: Wed Oct  7 18:10:16 UTC 2026
System time: Wed Oct  7 18:10:21 UTC 2026
```

### Bước 4: Hủy tiến trình an toàn bằng tín hiệu Kill
Sử dụng tín hiệu chuẩn SIGTERM (15) yêu cầu hệ điều hành tắt tiến trình một cách an toàn. Gửi tín hiệu đến PID đã tra cứu ở trên (`10567`):
```bash
$ kill -15 10567
[1]+  Terminated              nohup ./loop-monitor.sh > /dev/null 2>&1
```

Kiểm tra lại lần cuối để đảm bảo tiến trình đã thực sự biến mất khỏi danh sách:
```bash
$ pgrep -f loop-monitor.sh

$ ps aux | grep loop-monitor.sh
devops   10595  0.0  0.1   6600  2256 pts/0    S+   18:13   0:00 grep --color=auto loop-monitor.sh
```
*(Kết luận: Sau khi bắt được tín hiệu SIGTERM, tiến trình đã đóng luồng và thoát khỏi hệ thống thành công. Không cần phải ép buộc bằng SIGKILL (-9))*
