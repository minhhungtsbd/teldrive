# Teldrive - Trình Quản Lý Tệp Tin Telegram VFS

Teldrive là một ứng dụng mạnh mẽ được viết bằng Go, hoạt động như một hệ thống tệp ảo (VFS) phía trên tài khoản Telegram. Dự án này cho phép bạn lưu trữ, quản lý, chia sẻ và truyền phát (stream) tệp tin trực tiếp trên máy chủ Telegram thông qua giao diện Web thân thiện hoặc tích hợp Rclone.

Phiên bản này đã được tùy biến để hỗ trợ **Khóa API tĩnh vĩnh viễn (Static API Key)**, giúp tích hợp tự động hóa ổn định và đăng nhập giao diện Web không cần qua Telegram.

---

## 1. Hướng Dẫn Cài Đặt & Biên Dịch (Installation)

Bạn có thể biên dịch trực tiếp Teldrive trên máy tính cá nhân/VPS hoặc sử dụng môi trường Docker cô lập.

### Cách 1: Biên dịch bằng Docker (Khuyên dùng cho VPS)
Nếu VPS của bạn đã cài đặt Docker và bạn muốn biên dịch sạch sẽ mà không cần cài đặt Go trực tiếp:
```bash
# Di chuyển vào thư mục dự án
cd ~/teldrive-src

# Chạy container để tải giao diện, sinh API và biên dịch server
docker run --rm -v "$PWD":/app -w /app golang:alpine sh -c "
  apk add --no-cache git curl bash unzip &&
  go install github.com/go-task/task/v3/cmd/task@latest &&
  /go/bin/task gen &&
  /go/bin/task ui &&
  CGO_ENABLED=0 go build -trimpath -ldflags '-s -w' -o bin/teldrive
"
```
Tệp chạy sau khi biên dịch sẽ nằm ở thư mục `~/teldrive-src/bin/teldrive`.

### Cách 2: Biên dịch trực tiếp bằng Go (Yêu cầu Go >= 1.22)
```bash
# 1. Cài đặt các công cụ sinh mã tự động
go generate ./...

# 2. Biên dịch server
go build -o bin/teldrive main.go
```

---

## 2. Hướng Dẫn Cấu Hỏi & Cấu Hình (Configuration)

Teldrive sử dụng một tệp cấu hình duy nhất ở định dạng TOML (mặc định đặt tại `/etc/teldrive/config.toml`).

### Tạo Cơ Sở Dữ Liệu PostgreSQL
Teldrive lưu trữ sơ đồ thư mục ảo trên PostgreSQL. Hãy chạy các lệnh sau trong PostgreSQL để chuẩn bị cơ sở dữ liệu:
```sql
CREATE DATABASE teldrive_db;
CREATE USER teldrive_user WITH PASSWORD 'MatKhauCuaBan';
GRANT ALL PRIVILEGES ON DATABASE teldrive_db TO teldrive_user;
```

### Cấu hình tệp `config.toml`
Dưới đây là mẫu cấu hình cơ bản, bao gồm phần cấu hình **Static API Key**:

```toml
[server]
port = 8080
graceful-shutdown = '10s'

[db]
data-source = 'postgres://teldrive_user:MatKhauCuaBan@127.0.0.1:55432/teldrive_db?sslmode=disable'

[jwt]
secret = 'ma-secret-jwt-ngau-nhien-cua-ban'
session-time = '30d'
allowed-users = ["minhhungtsbdme"] # Whitelist người dùng Telegram được phép truy cập
api-key = 'fall_detection_web_secure_api_key_2026' # Khóa API tĩnh tự chọn của bạn
api-key-user = 0 # ID Telegram sở hữu session, để 0 để tự nhận diện session đầu tiên

[tg]
app-id = 2496 # Telegram App ID lấy từ my.telegram.org
app-hash = '8da85b0d5bfe62527e5b244c209159c3' # Telegram App Hash
auto-channel-create = true
channel-limit = 500000

[tg.session]
type = 'postgres'
key = 'session'
```

---

## 3. Khóa API Tĩnh & Đăng Nhập Giao Diện Web (Static API Key & Bypass)

Tính năng này giúp bạn tích hợp Teldrive với các hệ thống tự động hóa khác (như tải lên video/hình ảnh) và truy cập giao diện web mà không cần đăng nhập qua Telegram.

### Xác thực API cho ứng dụng khách (Client Integration)
Bạn có thể gọi bất kỳ API nào của Teldrive (tải lên, tạo thư mục, liệt kê tệp) bằng cách gửi kèm khóa API tĩnh thông qua 3 cách sau:
1. **Authorization Header:** `Authorization: Bearer <api-key-tinh>`
2. **X-API-Key Header:** `X-API-Key: <api-key-tinh>`
3. **URL Parameter:** `?token=<api-key-tinh>`

### Nhúng trực tuyến vào HTML (Streaming & Embed)
Khi cần hiển thị ảnh hoặc phát video trực tiếp trên giao diện web của ứng dụng khác, hãy chèn tham số `?token=` vào đường dẫn tệp tin:
```html
<!-- Nhúng ảnh -->
<img src="http://vps-ip:8080/api/files/id-file/anh.jpg?token=khoa_api_cua_ban" />

<!-- Phát video trực tuyến -->
<video controls>
  <source src="http://vps-ip:8080/api/files/id-file/video.mp4?token=khoa_api_cua_ban" type="video/mp4">
</video>
```

### Đăng nhập nhanh vào Web UI (Bypass Telegram Login)
Để truy cập trang quản trị Web UI trên trình duyệt mới mà không cần xác thực qua Telegram, hãy truy cập đường dẫn sau:
```
http://<vps-ip>:8080/api/auth/static?key=<api-key-tinh>
```
Hệ thống sẽ tự động xác thực khóa, cấp Cookie phiên làm việc và chuyển hướng bạn thẳng vào bảng điều khiển Web UI với tư cách là người dùng đã được cấu hình.

---

## 4. Quy Trình Cập Nhật Hệ Thống (Update)

Mỗi khi có phiên bản mới hoặc thay đổi mã nguồn trên GitHub, bạn thực hiện quy trình cập nhật trên VPS theo các bước sau:

```bash
# 1. Vào thư mục và kéo mã nguồn mới nhất
cd ~/teldrive-src
git pull origin main

# 2. Biên dịch lại tệp chạy bằng Docker
docker run --rm -v "$PWD":/app -w /app golang:alpine sh -c "
  apk add --no-cache git curl bash unzip &&
  go install github.com/go-task/task/v3/cmd/task@latest &&
  /go/bin/task gen &&
  /go/bin/task ui &&
  CGO_ENABLED=0 go build -trimpath -ldflags '-s -w' -o bin/teldrive
"

# 3. Thay thế tệp chạy của dịch vụ hệ thống
systemctl stop teldrive
cp ~/teldrive-src/bin/teldrive /usr/bin/teldrive
chmod +x /usr/bin/teldrive

# 4. Khởi động lại dịch vụ và kiểm tra nhật ký
systemctl start teldrive
journalctl -u teldrive -f
```
