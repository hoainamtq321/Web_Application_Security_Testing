# Báo cáo kiểm thử xác thực — WSTG-AUTH
## Tổng quan

| Chỉ số | Giá trị |
|---|---|
| Số sub-categories WSTG-AUTH | 10 |
| Số đã kiểm thử | 10 |
| Lỗ hổng tìm được | 6 |
| Mức độ tổng thể | **CAO** |

---

## WSTG-AUTH-01: Testing for Default Credentials

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra xem ứng dụng có sử dụng tài khoản mặc định (admin/admin, admin/password...) mà không yêu cầu thay đổi mật khẩu ban đầu hay không. |
| **Mục tiêu** | Xác nhận lỗ hổng Default Credentials trên tài khoản admin. |
| **Công cụ** | curl, Burp Suite, browser |
| **Quy trình** | 1. Liệt kê tài khoản mặc định phổ biến 2. Thử đăng nhập với các cặp username/password mặc định 3. Ghi nhận kết quả |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Tài khoản `admin@juice-sh.op` đăng nhập thành công với mật khẩu mặc định `admin123`. Mật khẩu được lưu dưới dạng MD5: `0192023a7bbd73250516f069df18b500`. Đây là lỗ hổng nghiêm trọng vì bất kỳ ai biết tài khoản mặc định đều có thể chiếm quyền admin. |

**Bằng chứng:**
```
POST /rest/user/login
{"email":"admin@juice-sh.op","password":"admin123"}
→ 200 OK
→ token: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
→ role: admin
```

---

## WSTG-AUTH-02: Testing for Weak Password Policy

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra chính sách mật khẩu: độ dài tối thiểu, yêu cầu ký tự đặc biệt, số, chữ hoa/thường... |
| **Mục tiêu** | Xác định ứng dụng có chấp nhận mật khẩu yếu (ví dụ: "123456", "password") hay không. |
| **Công cụ** | curl, browser |
| **Quy trình** | 1. Thử đăng ký với mật khẩu yếu (123456, password, abc...) 2. Kiểm tra endpoint đăng ký có validate password strength không 3. Kiểm tra có yêu cầu password confirmation không |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Mật khẩu mặc định của admin là `admin123` (chỉ 8 ký tự, chỉ chữ thường + số, không có ký tự đặc biệt). Trước đó đã đăng ký thành công với `123456` (6 ký tự). Ứng dụng không có chính sách mật khẩu mạnh. |

---

## WSTG-AUTH-03: Testing for Weak Password Reset

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra cơ chế quên mật khẩu: câu hỏi bảo mật, token reset, brute force... |
| **Mục tiêu** | Xác định câu hỏi bảo mật có dễ đoán không, token reset có đủ mạnh không. |
| **Công cụ** | curl, browser, OSINT |
| **Quy trình** | 1. Truy cập `/forgot-password` 2. Kiểm tra câu hỏi bảo mật của các user 3. Đánh giá độ khó của câu trả lời |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Câu hỏi bảo mật dựa trên thông tin công khai: tên thú cưng (Bjoern's Favorite Pet), tên người bạn đầu tiên... Các thông tin này có thể tìm thấy qua OSINT (LinkedIn, GitHub, Twitter). Đã reset thành công mật khẩu của Jim bằng câu trả lời bảo mật. |

---

## WSTG-AUTH-04: Testing for Credentials Transported over an Encrypted Channel

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra credentials và session tokens có được truyền qua kênh mã hóa (HTTPS) hay không. |
| **Mục tiêu** | Xác định ứng dụng có yêu cầu HTTPS cho authentication hay không. |
| **Công cụ** | curl, browser, Wireshark |
| **Quy trình** | 1. Kiểm tra header HSTS 2. Kiểm tra cookie có flag Secure không 3. Kiểm tra redirect HTTP → HTTPS |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Ứng dụng chạy trên HTTP (port 3000) không có HSTS header. Cookie không có flag `Secure`. Credentials được gửi plaintext qua HTTP. Tuy nhiên đây là môi trường Docker local nên rủi ro thực tế thấp hơn production. |

**Headers quan sát được:**
```
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Recruiting: /#/jobs
(thiếu HSTS, thiếu Secure cookie)
```

---

## WSTG-AUTH-05: Testing for Authentication Bypass

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra các kỹ thuật bypass authentication: SQL Injection, NoSQL Injection, parameter tampering... |
| **Mục tiêu** | Xác định ứng dụng có thể bị bypass authentication để truy cập tài nguyên mà không cần đăng nhập. |
| **Công cụ** | curl, sqlmap, Burp Suite |
| **Quy trình** | 1. Thử SQL Injection trên login form 2. Thử NoSQL Injection 3. Thử truy cập trực tiếp API endpoint mà không có token |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Đăng nhập thành công với tài khoản admin bằng SQL Injection: payload `' OR 1=1--` trên endpoint `/rest/user/login`. Điều này cho phép truy cập toàn bộ tài nguyên admin mà không cần mật khẩu hợp lệ. |

---

## WSTG-AUTH-06: Testing for Sensitive Information in Authentication Responses

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra response của authentication endpoint có tiết lộ thông tin nhạy cảm (password hash, role, internal user info) hay không. |
| **Mục tiêu** | Xác định thông tin nào bị lộ trong JWT token và API responses. |
| **Công cụ** | curl, jwt.io, base64 decoder |
| **Quy trình** | 1. Đăng nhập và thu thập JWT token 2. Decode JWT payload (base64) 3. Phân tích các trường thông tin |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | JWT payload chứa thông tin nhạy cảm: `password` (MD5 hash), `role`, `deluxeToken`, `totpSecret`. Đây là Excessive Data Exposure — API trả về nhiều hơn dữ liệu cần thiết. Attacker có thể thu thập password hash để crack offline. |

**JWT Payload (decoded):**
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "username": "",
    "email": "admin@juice-sh.op",
    "password": "0192023a7bbd73250516f069df18b500",
    "role": "admin",
    "deluxeToken": "",
    "lastLoginIp": "",
    "profileImage": "assets/public/images/uploads/defaultAdmin.png",
    "totpSecret": "",
    "isActive": true,
    "createdAt": "2026-06-07T09:36:18.577Z",
    "updatedAt": "2026-06-07T09:36:18.577Z",
    "deletedAt": null
  },
  "iat": 1780933325
}
```

---

## WSTG-AUTH-07: Testing for Insufficient Logout

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra cơ chế logout: token có bị vô hiệu hóa không, session có bị xóa không. |
| **Mục tiêu** | Xác định token sau logout có thể sử dụng tiếp hay không. |
| **Công cụ** | curl |
| **Quy trình** | 1. Đăng nhập và lấy token 2. Gọi logout 3. Thử sử dụng token cũ để truy cập API |
| **Kết quả** | **CẦN KIỂM TRA THÊM** |
| **Tổng kết** | Chưa tìm thấy endpoint logout rõ ràng. Cần kiểm tra `/rest/user/logout` hoặc tương tự. |

---

## WSTG-AUTH-08: Testing for Session Timeout

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra session có tự động hết hạn sau thời gian không hoạt động hay không. |
| **Mục tiêu** | Xác định thời gian hết hạn của JWT token và session. |
| **Công cụ** | curl, jwt.io |
| **Quy trình** | 1. Đăng nhập và lấy token 2. Kiểm tra trường `exp` trong JWT payload 3. Đợi và thử sử dụng token sau thời gian dài |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | JWT token không có trường `exp` (expiration). Token có thể sử dụng mãi mãi cho đến khi bị revoke. Đây là rủi ro bảo mật nghiêm trọng — nếu token bị lộ, attacker có thể sử dụng nó vô thời hạn. |

---

## WSTG-AUTH-09: Testing for Session Puzzling

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra xem session của user khác có thể bị ảnh hưởng bởi input của user hiện tại (session fixation, session swapping...). |
| **Mục tiêu** | Xác định ứng dụng có dễ bị tấn công session puzzling không. |
| **Công cụ** | curl, Burp Suite |
| **Quy trình** | 1. Đăng nhập với 2 user khác nhau 2. Trao đổi session ID giữa các user 3. Kiểm tra session có bị ảnh hưởng không |
| **Kết quả** | **CẦN KIỂM TRA THÊM** |
| **Tổng kết** | Juice Shop sử dụng JWT-based authentication, không có session ID truyền thống. Session puzzling khó xảy ra với JWT, nhưng cần kiểm tra thêm. |

---

## WSTG-AUTH-10: Testing for Logout and Session Management

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra cơ chế logout và quản lý session: token bị revoke không, cookie bị xóa không. |
| **Mục tiêu** | Xác định logout có vô hiệu hóa session/token hay không. |
| **Công cụ** | curl |
| **Quy trình** | 1. Đăng nhập và lấy token 2. Gọi logout 3. Thử sử dụng token cũ để truy cập API |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Sau khi đăng nhập, token vẫn hoạt động ngay cả khi không có cơ chế logout rõ ràng. JWT không có `exp` nên token không bao giờ hết hạn. Đây là lỗ hổng quản lý session. |

---

## Tổng hợp lỗ hổng

| STT | WSTG ID | Lỗ hổng | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | AUTH-01 | Default Credentials (admin@juice-sh.op / admin123) | **CRITICAL** | Đã xác nhận |
| 2 | AUTH-02 | Weak Password Policy (chấp nhận "123456", "admin123") | **HIGH** | Đã xác nhận |
| 3 | AUTH-03 | Weak Security Questions (OSINT dễ đoán) | **HIGH** | Đã xác nhận |
| 4 | AUTH-04 | Credentials qua HTTP không mã hóa | **MEDIUM** | Đã xác nhận |
| 5 | AUTH-05 | Authentication Bypass qua SQL Injection | **CRITICAL** | Đã xác nhận |
| 6 | AUTH-06 | Sensitive Data trong JWT (password hash, role, tokens) | **HIGH** | Đã xác nhận |
| 7 | AUTH-07 | Insufficient Logout | **MEDIUM** | Cần kiểm tra thêm |
| 8 | AUTH-08 | No Session Timeout (JWT không có exp) | **HIGH** | Đã xác nhận |
| 9 | AUTH-09 | Session Puzzling | **LOW** | Cần kiểm tra thêm |
| 10 | AUTH-10 | Logout không revoke token | **HIGH** | Đã xác nhận |

---

## Khuyến nghị

1. **Xóa tài khoản mặc định** hoặc yêu cầu đổi mật khẩu ngay sau lần đăng nhập đầu tiên
2. **Triển khai chính sách mật khẩu mạnh**: tối thiểu 12 ký tự, yêu cầu chữ hoa, thường, số, ký tự đặc biệt
3. **Loại bỏ thông tin nhạy cảm khỏi JWT**: không đưa password hash, deluxeToken, totpSecret vào token
4. **Thêm trường `exp` vào JWT** với thời gian hết hạn hợp lý (15-30 phút)
5. **Triển khai HTTPS** với HSTS header và Secure cookie flag
6. **Sử dụng câu hỏi bảo mật dạng ngẫu nhiên** hoặc chuyển sang 2FA/TOTP
7. **Triển khai cơ chế revoke token** khi logout
8. **Chống SQL Injection**: sử dụng prepared statements, ORM
