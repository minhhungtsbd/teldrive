# Kế hoạch triển khai - Thêm tính năng API Key vĩnh viễn cho Teldrive

Kế hoạch này giải quyết nhu cầu sử dụng một **API Key vĩnh viễn (hoặc siêu dài hạn)** để tích hợp tự động hóa ổn định mà không cần định kỳ lấy token từ trình duyệt (vốn sẽ hết hạn sau mỗi 7 ngày).

## Phân tích & Ý tưởng giải pháp

Teldrive là hệ thống đa người dùng, lưu trữ và tải tệp tin thông qua các phiên kết nối Telegram cụ thể (`models.Session`). Do đó, bất kỳ yêu cầu tải lên (upload) hay tạo thư mục nào đều phải ánh xạ tới:
1. ID người dùng Telegram (`UserId`)
2. Phiên kết nối hoạt động của Telegram (`TgSession`)

### Giải pháp kỹ thuật:
Thay vì can thiệp vào cấu trúc cơ sở dữ liệu (tạo bảng khóa API riêng rất phức tạp và dễ lỗi khi Teldrive cập nhật), chúng ta sẽ **tích hợp trực tiếp tính năng Static API Key vào lớp cấu hình và xác thực (auth middleware) của Teldrive**:

1. **Cấu hình:** Thêm hai trường mới vào cấu hình `[jwt]` trong tệp `config.toml` của Teldrive:
   * `api-key`: Khóa API tĩnh tự chọn (ví dụ: `my-secure-automation-key-123`).
   * `api-key-user`: ID người dùng Telegram sở hữu tài khoản cần upload (Tùy chọn, mặc định nếu không điền sẽ lấy session đầu tiên có sẵn trong cơ sở dữ liệu).

2. **Xác thực (Middleware):** Trong hàm xác thực Bearer Token của Teldrive (`handleAuth`):
   * Nếu Bearer token gửi lên trùng với `api-key` đã cấu hình, Teldrive sẽ bỏ qua việc giải mã JWT thông thường.
   * Nó sẽ tự động truy vấn cơ sở dữ liệu để tìm phiên kết nối Telegram (`models.Session`) của người dùng tương ứng (hoặc phiên đầu tiên tìm thấy).
   * Tạo ra một `JWTClaims` giả lập chứa thông tin phiên kết nối này để đưa vào ngữ cảnh xử lý (context).
   * Yêu cầu API sẽ diễn ra thành công như thể đã được đăng nhập bằng cookie.

Với giải pháp này, bạn chỉ cần cấu hình khóa API tĩnh **một lần duy nhất** trong cả Teldrive và ứng dụng `fall_detection_web`, nó sẽ chạy vĩnh viễn mà không bao giờ hết hạn!

---

## Các tệp tin cần sửa đổi trong Repo Teldrive

Dưới đây là các thay đổi chúng ta sẽ thực hiện trực tiếp trong thư mục mã nguồn Teldrive của bạn (`C:\Users\minhh\Desktop\CUSOR\teldrive-main`).

### 1. Cấu hình Teldrive
#### [MODIFY] [config.go](file:///C:/Users/minhh/Desktop/CUSOR/teldrive-main/internal/config/config.go)
* Thêm các trường `APIKey` và `APIKeyUser` vào struct `JWTConfig`.

```go
type JWTConfig struct {
	Secret       string        `validate:"required" default:"" description:"JWT signing secret key"`
	SessionTime  time.Duration `default:"30d" description:"JWT token validity duration"`
	AllowedUsers []string      `default:"" description:"List of allowed usernames"`
	APIKey       string        `default:"" description:"Static API Key for automation"`
	APIKeyUser   int64         `default:"0" description:"User ID for the static API Key"`
}
```

### 2. Bộ xử lý xác thực Teldrive
#### [MODIFY] [auth.go](file:///C:/Users/minhh/Desktop/CUSOR/teldrive-main/internal/auth/auth.go)
* Cập nhật hàm `handleAuth` tại dòng 112 để kiểm tra nếu token truyền vào là static API Key.
* Nhập thêm gói `"strconv"` nếu chưa có (trong tệp này đã có sẵn).

```go
func (s *securityHandler) handleAuth(ctx context.Context, token string) (context.Context, error) {
	if s.cfg.APIKey != "" && token == s.cfg.APIKey {
		var session models.Session
		var err error
		if s.cfg.APIKeyUser != 0 {
			err = s.db.Model(&models.Session{}).Where("user_id = ?", s.cfg.APIKeyUser).First(&session).Error
		} else {
			// Lấy phiên kết nối đầu tiên hoạt động nếu không chỉ định UserID cụ thể
			err = s.db.Model(&models.Session{}).First(&session).Error
		}
		if err == nil {
			claims := &types.JWTClaims{
				RegisteredClaims: jwt.RegisteredClaims{
					Subject: strconv.FormatInt(session.UserId, 10),
				},
				Hash:      session.Hash,
				TgSession: session.Session,
			}
			return context.WithValue(ctx, authKey, claims), nil
		}
	}

	claims, err := VerifyUser(ctx, s.db, s.cache, s.cfg.Secret, token)
	if err != nil {
		return nil, &ogenerrors.SecurityError{Err: err}
	}
	return context.WithValue(ctx, authKey, claims), nil
}
```

---

## Các bước Biên dịch & Cấu hình Teldrive mới

Sau khi sửa đổi mã nguồn, bạn sẽ thực hiện các bước sau để áp dụng:

1. **Biên dịch lại Teldrive:**
   * Mở terminal tại thư mục `C:\Users\minhh\Desktop\CUSOR\teldrive-main`.
   * Chạy lệnh biên dịch:
     ```powershell
     go build -o bin/teldrive main.go
     ```
   * (Hoặc nếu sử dụng Taskfile thì chạy lệnh: `task server`).
   
2. **Cập nhật tệp cấu hình `config.toml` trên VPS:**
   * Mở tệp cấu hình `config.toml` đang chạy Teldrive trên VPS của bạn.
   * Tại mục `[jwt]`, thêm cấu hình khóa API tùy chọn của bạn:
     ```toml
     [jwt]
     secret = "your-jwt-secret"
     session-time = "30d"
     api-key = "fall_detection_web_secure_api_key_2026"
     ```
   * Lưu lại và chép file chạy `teldrive` mới biên dịch lên VPS, sau đó khởi động lại dịch vụ Teldrive:
     ```bash
     sudo systemctl restart teldrive
     ```

3. **Cấu hình trên `fall_detection_web`:**
   * Vào Settings của `fall_detection_web`.
   * Điền khóa API tĩnh vừa chọn (ở ví dụ trên là `fall_detection_web_secure_api_key_2026`) vào ô **Teldrive token**.
   * Nhấn Lưu.

---

## Kế hoạch kiểm chứng (Verification)

1. Kiểm tra biên dịch thành công mã nguồn Teldrive mới.
2. Kiểm tra tính hợp lệ của static API Key thông qua tính năng "Kiểm tra Teldrive" trong ứng dụng.
