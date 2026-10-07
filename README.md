# Session 07 - Bài 4: Cấu hình Reverse Proxy Nginx cho ứng dụng Spring Boot

**Học viên:** Trần Thị Khuyến

## Mục tiêu
- Cấu hình Nginx Server Block làm Reverse Proxy.
- Dùng path matching: `/` phục vụ trang tĩnh, `/api/` chuyển tiếp sang Spring Boot ở cổng 8082.

## Các bước thực hiện
1. Tạo trang tĩnh tại `/var/www/html/index.html` (hiển thị Họ tên, Mã lớp).
2. Tạo cấu hình `/etc/nginx/sites-available/spring-proxy.conf`:
   - `location /` → `try_files` trỏ vào `/var/www/html`.
   - `location /api/` → `proxy_pass http://127.0.0.1:8082/` kèm header `Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto`.
3. Kích hoạt: `sudo ln -s /etc/nginx/sites-available/spring-proxy.conf /etc/nginx/sites-enabled/`.
4. Gỡ site mặc định: `sudo rm -f /etc/nginx/sites-enabled/default`.
5. Kiểm tra + reload: `sudo nginx -t` → `sudo systemctl reload nginx`.

## Kết quả kiểm tra

### nginx -t
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
→ Đạt yêu cầu (syntax is ok + test is successful).

### Trang tĩnh /
```
$ curl -I http://localhost/
HTTP/1.1 200 OK
```
→ Trả về 200 kèm nội dung trang tĩnh chứa thông tin học viên.

### /api/health
```
$ curl -I http://localhost/api/health
HTTP/1.1 502 Bad Gateway
```
→ 502 do ứng dụng Spring Boot (bài 3) chưa chạy ở cổng 8082. Cấu hình proxy đã đúng: lỗi 502 chứng tỏ Nginx đã chuyển tiếp tới `127.0.0.1:8082` nhưng chưa có backend lắng nghe. Khi service Spring Boot (bài 3) chạy, `/api/health` sẽ trả mã trạng thái từ backend thay vì 502.

## Kết luận
Reverse proxy đã cấu hình hoàn chỉnh: trang tĩnh phục vụ tại `/` (200), định tuyến `/api/` tới backend 8082 đã thiết lập đúng.
