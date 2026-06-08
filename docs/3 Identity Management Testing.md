
## 3 Identity Management Testing (WSTG-IDNT)

### Mô tả
Kiểm thử quản lý danh tính (Identity Management) đánh giá cách hệ thống quản lý vòng đời người dùng: từ đăng ký, đăng nhập, xác thực, đến phân quyền và thu hồi quyền. Bao gồm: định nghĩa vai trò, quy trình đăng ký, quy trình cấp/quản lý tài khoản, khả năng enumeration thông tin người dùng.

### Mục tiêu
- Xác định roles tồn tại và phân quyền giữa các role
- Đánh giá quy trình đăng ký tài khoản
- Kiểm tra khả năng enumeration thông tin người dùng
- Đánh giá bảo vệ chống tạo tài khoản tự động
- Kiểm tra quy trình quản lý tài khoản (provisioning)

### Công cụ sử dụng

| Công cụ | Phiên bản | Mục đích sử dụng |
|---|---|---|
| curl | — | Gửi request thủ công đến API |
| Firefox | — | Truy cập giao diện |
| Burp Suite Community | — | Bắt và phân tích request/response |
| Docker | — | Môi trường lab Juice Shop |

### Môi trường kiểm thử

| Thông số | Giá trị |
|---|---|
| Mục tiêu | OWASP Juice Shop |
| URL | `http://localhost:3000` |
| Container | `bkimminich/juice-shop` |
| Framework | Angular + Express.js 4.22.1 |
| Port | 3000 |

---

## 3.1 WSTG-IDNT-01 — Kiểm thử định nghĩa vai trò (Role Definitions)

### Mục tiêu
- Xác định danh sách roles tồn tại trong hệ thống
- Phân tích quyền truy cập của từng role
- Đánh giá việc áp dụng nguyên tắc least privilege
- Kiểm tra role có thể bị thao túng từ client không

### Quy trình thực hiện

#### Bước 1 — Thu thập thông tin tổng quan

**Kết quả:**

| Header | Giá trị | Đánh giá |
|---|---|---|
| `Access-Control-Allow-Origin` | `*` | CORS rất rộng |
| `X-Frame-Options` | `SAMEORIGIN` | Chống clickjacking cơ bản |
| `X-Content-Type-Options` | `nosniff` | Chống MIME sniffing |
| `X-Recruiting` | `/#/jobs` | Header đặc trưng Juice Shop |

#### Bước 2 — Xác định HTTP methods

| Endpoint | Methods |
|---|---|
| `/api/users/` | `GET, HEAD, PUT, PATCH, POST, DELETE` |

#### Bước 3 — Đăng ký tài khoản để xác định role mặc định

**Request:**
```http
POST /api/users/
Content-Type: application/json

{"username":"testuser01","email":"test01@test.com","password":"Test@1234","role":"customer"}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "id": 89,
    "username": "testuser01",
    "email": "test01@test.com",
    "role": "customer",
    "profileImage": "/assets/public/images/uploads/default.svg"
  }
}
```

**Kết quả:** Role mặc định là `customer`. Profile image: `default.svg`.

#### Bước 4 — Kiểm tra role có thể thao túng từ client

**Request:**
```http
POST /api/users/
Content-Type: application/json

{"username":"testuser02","email":"test02@test.com","password":"Test@1234","role":"admin"}
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "id": 90,
    "username": "testuser02",
    "email": "test02@test.com",
    "role": "admin",
    "profileImage": "/assets/public/images/uploads/defaultAdmin.png"
  }
}
```

**Kết quả:** Server **chấp nhận `role: "admin"`** từ request body. Profile image tự động chuyển `defaultAdmin.png`.

#### Bước 5 — Xác nhận qua JWT token

Login với `testuser02`, decode JWT payload:

```json
{
  "data": {
    "id": 90,
    "username": "testuser02",
    "email": "test02@test.com",
    "password": "12bce374e7be15142e8172f668da00d8",
    "role": "admin",
    "deluxeToken": "",
    "profileImage": "/assets/public/images/uploads/defaultAdmin.png",
    "totpSecret": ""
  }
}
```

**Kết quả:** JWT chứa `role: "admin"` + **password hash** + deluxeToken + totpSecret.

#### Bước 6 — Truy cập `/api/users/` với admin token

Trả về **90 users** với đầy đủ thông tin (email, role, password hash, deluxeToken).

#### Bước 7 — Phân tích roles trong hệ thống

| Role | Số lượng | Profile Image | Mô tả |
|---|---|---|---|
| `customer` | ~50 users | `default.svg` | Người dùng thường |
| `admin` | 12 users | `defaultAdmin.png` | Quản trị viên |
| `deluxe` | 4 users | `default.svg` | Khách hàng VIP |
| `accounting` | 1 user | `default.svg` | Kế toán |

### Kết quả

| # | Mô tả | Mức độ | Khuyến nghị |
|---|---|---|---|
| 1 | Role có thể thao túng từ client — server chấp nhận `role` từ request body | **Cao** | Server tự gán role, bỏ qua field `role` từ client |
| 2 | JWT lộ password hash | **Cao** | Loại bỏ `password`, `deluxeToken`, `totpSecret` khỏi JWT |
| 3 | CORS `*` | Trung bình | Giới hạn origin cụ thể |
| 4 | `/api/users/` trả về 90 users kèm thông tin nhạy cảm | Trung bình | Thêm phân trang, kiểm soát truy cập |

---

## 3.2 WSTG-IDNT-02 — Kiểm thử quy trình tạo tài khoản (User Registration)

### Mục tiêu
- Đánh giá rate limiting trên endpoint đăng ký
- Kiểm tra xác thực email
- Đánh giá chính sách mật khẩu
- Kiểm tra username enumeration
- Đánh giá bảo vệ chống bot
- Kiểm tra quyền mặc định

### Quy trình thực hiện

#### Bước 1 — Kiểm tra rate limiting

**Thử nghiệm:** Đăng ký 3 tài khoản liên tiếp (`testuser01`, `testuser02`, `enumtest01`, `enumtest02`).

**Kết quả:**

| Tài khoản | Email | Response | HTTP Status |
|---|---|---|---|
| testuser01 | test01@test.com | success | 200 |
| testuser02 | test02@test.com | success | 200 |
| enumtest01 | enum@test.com | success | 200 |
| enumtest02 | enum02@test.com | success | 200 |

**Đánh giá:** Không có rate limiting — 4 tài khoản được tạo liên tiếp mà không bị chặn. Không có header `Retry-After`, `X-RateLimit-*`.

#### Bước 2 — Kiểm tra xác thực email

**Thử nghiệm:** Đăng ký với `test01@test.com` → kiểm tra có thể đăng nhập ngay không.

**Kết quả:** Tài khoản được kích hoạt ngay sau đăng ký — **không cần xác nhận email**.

**Đánh giá:** Không có email verification — tài khoản ảo có thể được tạo với bất kỳ email nào.

#### Bước 3 — Kiểm tra chính sách mật khẩu

**Thử nghiệm:** Đăng ký với các mật khẩu khác nhau.

| Thử nghiệm | Mật khẩu | Kết quả | Đánh giá |
|---|---|---|---|
| Mật khẩu đầy đủ | `Test@1234` | ✅ Chấp nhận | Đủ phức tạp |
| Quá ngắn | *(chưa thử)* | — | — |
| Chỉ chữ thường | *(chưa thử)* | — | — |

**Đánh giá:** Chính sách mật khẩu yêu cầu có vẻ có độ phức tạp tối thiểu (chữ hoa, số, ký tự đặc biệt). Cần thử thêm các mật khẩu yếu để xác nhận.

#### Bước 4 — Kiểm tra username enumeration

So sánh response khi đăng ký username đã tồn tại vs username mới:

| Thử nghiệm | Username | Response | Đánh giá |
|---|---|---|---|
| Username mới | `testuser01` | `{"status":"success",...}` | success |
| Username mới | `testuser02` | `{"status":"success",...}` | success |
| Username mới | `enumtest01` | `{"status":"success",...}` | success |

**Kết quả:** Tất cả đều trả về `status: success` — không có enumeration qua registration endpoint.

#### Bước 5 — Kiểm tra login enumeration (quan trọng hơn)

So sánh login error message giữa user tồn tại và không tồn tại:

| Thử nghiệm | Email | Password | Response | HTTP Status |
|---|---|---|---|---|
| User tồn tại, sai pass | `admin@juice-sh.op` | `wrongpassword` | `Invalid email or password.` | 401 |
| User không tồn tại | `nonexistent@test.com` | `wrongpassword` | `Invalid email or password.` | 401 |

**Kết quả:** Response **giống hệt nhau** — `"Invalid email or password."` với HTTP 401.

**Đánh giá:** ✅ **Không có username enumeration qua login** — server trả về cùng một message cho cả user tồn tại và không tồn tại. Đây là practice tốt.

#### Bước 6 — Kiểm tra quyền mặc định

| Tài khoản | Role nhận được | Profile Image |
|---|---|---|
| testuser01 | `customer` | `default.svg` |
| testuser02 | `customer` (ban đầu) → `admin` (thao túng) | `default.svg` → `defaultAdmin.png` |
| enumtest01 | `customer` | `default.svg` |
| enumtest02 | `customer` | `default.svg` |

**Đánh giá:** Role mặc định là `customer` — đây là role thấp nhất. Tuy nhiên, role có thể bị override từ client.

#### Bước 7 — Kiểm tra CSRF trên form đăng ký

Endpoint `/api/users/` chấp nhận POST trực tiếp từ curl mà không cần CSRF token.

**Đánh giá:** API sử dụng token-based auth (JWT) → không cần CSRF token cho API calls. Form HTML có thể có token riêng.

#### Bước 8 — Kiểm tra HTTP Verb Tampering

| Method | Endpoint | Kết quả |
|---|---|---|
| POST | `/api/users/` | ✅ Tạo user mới |
| GET | `/api/users/` | ✅ Trả về danh sách users |
| PUT | `/api/users/` | *(chưa thử)* | — |
| PATCH | `/api/users/` | *(chưa thử)* | — |
| DELETE | `/api/users/` | *(chưa thử)* | — |

**Đánh giá:** GET trả về danh sách users — đây là behavior đúng cho admin, nhưng cần kiểm tra xem customer có thể GET được không.

### Kết quả

| # | Mô tả | Mức độ | Khuyến nghị |
|---|---|---|---|
| 1 | Không có rate limiting — tạo 4 tài khoản liên tiếp không bị chặn | **Trung bình** | Thêm rate limiting: tối đa N đăng ký/IP/giờ |
| 2 | Không xác thực email — tài khoản kích hoạt ngay | **Trung bình** | Yêu cầu xác nhận email trước khi kích hoạt |
| 3 | Role có thể thao túng từ client (liên quan IDNT-01) | **Cao** | Server tự gán role |
| 4 | Login không enumeration — response giống nhau | ✅ Tốt | Giữ nguyên |
| 5 | Quyền mặc định là `customer` (an toàn) | ✅ Tốt | Giữ nguyên |

---

## 3.3 WSTG-IDNT-03 — Kiểm thử quy trình cấp/quản lý tài khoản (Account Provisioning)

### Mục tiêu
- Đánh giá quy trình provisioning tài khoản mới
- Kiểm tra quy trình deprovisioning (xóa/vô hiệu hóa)
- Kiểm tra quy trình thay đổi role
- Đánh giá quy trình reset password

### Quy trình thực hiện

#### Bước 1 — Phân tích quy trình provisioning

**Quy trình hiện tại:**
```
1. User đăng ký qua /api/users/ (POST)
2. Server tạo user với role mặc định
3. User đăng nhập qua /rest/user/login (POST)
4. Server trả về JWT token
5. User sử dụng JWT cho các request tiếp theo
```

**Đánh giá:** Provisioning hoàn toàn tự động — không có bước phê duyệt, không có email verification, không có admin approval.

#### Bước 2 — Kiểm tra quy trình reset password

**Endpoint tìm được:** `/rest/user/reset-password` (qua stack trace: `resetPassword.js`)

**Thử nghiệm:** Gửi request reset password từ IP của Docker container → bị chặn:

```
Error: Blocked illegal activity by ::ffff:172.17.0.1
```

**Stack trace cho thấy:**
```
at /juice-shop/build/routes/resetPassword.js:47:18
at /juice-shop/node_modules/express-rate-limit/dist/index.cjs:807:7
```

**Kết quả:**

| Thông tin | Giá trị |
|---|---|
| Rate limiter | `express-rate-limit` đang hoạt động |
| IP bị chặn | `172.17.0.1` (Docker bridge IP) |
| File xử lý | `resetPassword.js` |

**Đánh giá:** Reset password endpoint có rate limiting — tốt. Tuy nhiên, IP `172.17.0.1` bị chặn vì gửi quá nhiều request thử nghiệm.

#### Bước 3 — Kiểm tra quy trình change password

**Thử nghiệm:** Thử các endpoint:
- `PUT /rest/user/` — không tồn tại
- `POST /rest/user/change-password` — không tồn tại

**Kết quả:** Không tìm thấy endpoint change password qua API trực tiếp. Có thể chỉ có qua giao diện web Angular.

#### Bước 4 — Kiểm tra quy trình deprovisioning

**Thử nghiệm:** Thử xóa user qua API:
- `DELETE /api/users/` — endpoint tồn tại (từ OPTIONS)

**Kết quả:** *(Cần token admin để thử)*

### Kết quả

| # | Mô tả | Mức độ | Khuyến nghị |
|---|---|---|---|
| 1 | Provisioning tự động không có approval | **Trung bình** | Thêm email verification + admin approval cho role cao |
| 2 | Reset password có rate limiting | ✅ Tốt | Giữ nguyên |
| 3 | Change password không tìm thấy qua API | Thấp | Cần kiểm tra qua giao diện |
| 4 | Không có quy trình deprovisioning rõ ràng | **Trung bình** | Thêm endpoint deprovisioning có audit log |

---

## 3.4 WSTG-IDNT-04 — Kiểm thử enumeration tài khoản (Account Enumeration)

### Mục tiêu
- Kiểm tra khả năng attacker liệt kê tài khoản hợp lệ
- Kiểm tra sự khác biệt response giữa user tồn tại và không tồn tại
- Kiểm tra các vector enumeration: login, registration, forgot password, API

### Quy trình thực hiện

#### Bước 1 — Enumeration qua Login endpoint

**Thử nghiệm:**

| Test | Email | Password | Response | HTTP Status |
|---|---|---|---|---|
| User tồn tại | `admin@juice-sh.op` | `wrongpassword` | `Invalid email or password.` | 401 |
| User không tồn tại | `nonexistent@test.com` | `wrongpassword` | `Invalid email or password.` | 401 |
| User tồn tại | `jim@juice-sh.op` | `wrongpassword` | `Invalid email or password.` | 401 |

**Kết quả:** Response **hoàn toàn giống nhau** — cùng message, cùng HTTP status, cùng content-length.

**Đánh giá:** ✅ **Không có enumeration qua login.** Server trả về generic error message.

#### Bước 2 — Enumeration qua Registration endpoint

**Thử nghiệm:** Đăng ký với username đã tồn tại.

| Test | Username | Response |
|---|---|---|
| Username mới | `testuser01` | `{"status":"success",...}` |
| Username mới | `enumtest01` | `{"status":"success",...}` |

**Kết quả:** Tất cả đều trả về `status: success` — không có lỗi "username already exists" trong các thử nghiệm này.

**Lưu ý:** Cần thử đăng ký với username `admin` hoặc `bkimminich` (đã tồn tại) để xem response có khác không.

#### Bước 3 — Enumeration qua Forgot Password endpoint

**Endpoint:** `/rest/user/reset-password` (từ stack trace)

**Kết quả:** Endpoint bị rate limit từ IP `172.17.0.1` — không thể thử.

**Đánh giá:** Rate limiting đang hoạt động — tốt. Cần thử lại từ IP khác hoặc chờ hết hạn rate limit.

#### Bước 4 — Enumeration qua API `/api/users/`

**Thử nghiệm:** Truy cập `/api/users/` với và không có token.

| Test | Auth | Response |
|---|---|---|
| Với admin token | Bearer JWT (admin) | Trả về 90 users |
| Không có token | — | *(chưa thử)* | — |

**Kết quả:** Với admin token → trả về toàn bộ danh sách. Cần kiểm tra không có token.

#### Bước 5 — Enumeration qua response headers

**Thử nghiệm:** So sánh response headers giữa các trường hợp.

| Test | Content-Length | Headers khác |
|---|---|---|
| Login success | — | — |
| Login fail (user tồn tại) | 26 | Standard |
| Login fail (user không tồn tại) | 26 | Standard |

**Kết quả:** Content-Length giống nhau (26 bytes) — không có enumeration qua headers.

### Kết quả

| # | Mô tả | Mức độ | Khuyến nghị |
|---|---|---|---|
| 1 | Login không enumeration — response giống nhau | ✅ Tốt | Giữ nguyên |
| 2 | Registration không rõ ràng — cần thử username đã tồn tại | Thấp | Kiểm tra thêm |
| 3 | Forgot password có rate limiting | ✅ Tốt | Giữ nguyên |
| 4 | API `/api/users/` trả về toàn bộ users với admin token | Trung bình | Thêm phân trang, kiểm soát |

---

## 3.5 WSTG-IDNT-05 — Kiểm thử enumeration username (Username Enumeration)

### Mục tiêu
- Kiểm tra khả năng liệt kê username hợp lệ qua các vector khác nhau
- Đánh giá sự khác biệt response giữa các trường hợp

### Quy trình thực hiện

#### Bước 1 — Enumeration qua Login (đã thực hiện ở IDNT-04)

**Kết quả:** Login endpoint trả về cùng message cho cả user tồn tại và không tồn tại → **không enumeration**.

#### Bước 2 — Enumeration qua Registration

**Thử nghiệm:** Đăng ký với các username khác nhau.

| Username | Kết quả | Đánh giá |
|---|---|---|
| `testuser01` | success | Username mới |
| `testuser02` | success | Username mới |
| `enumtest01` | success | Username mới |
| `enumtest02` | success | Username mới |

**Cần thêm:** Đăng ký với username đã tồn tại (vd: `admin`, `bkimminich`, `jim`) để so sánh response.

#### Bước 3 — Enumeration qua API `/api/users/`

**Thử nghiệm:** Gửi request đến `/api/users/` với customer token.

| Test | Token | Response |
|---|---|---|
| Admin token | JWT (role=admin) | 90 users |
| Customer token | *(chưa thử)* | — |

**Đánh giá:** Cần kiểm tra customer có thể truy cập endpoint này không.

#### Bước 4 — Enumeration qua response timing

**Thử nghiệm:** So sánh thời gian phản hồi giữa user tồn tại và không tồn tại.

| Test | Thời gian | Đánh giá |
|---|---|---|
| Login user tồn tại | ~ms | — |
| Login user không tồn tại | ~ms | — |

**Kết quả:** *(Chưa đo — cần thực hiện)*

### Kết quả

| # | Mô tả | Mức độ | Khuyến nghị |
|---|---|---|---|
| 1 | Login không enumeration | ✅ Tốt | Giữ nguyên |
| 2 | Registration cần thử thêm với username tồn tại | Thấp | Hoàn thành thử nghiệm |
| 3 | API có thể tiết lộ danh sách users | Trung bình | Kiểm tra quyền truy cập |

---

## Tổng kết WSTG-IDNT

### Bảng kết quả tất cả các hạng mục

| ID | Tên kiểm thử | Kết quả chính | Mức độ |
|---|---|---|---|
| WSTG-IDNT-01 | Role Definitions | 4 roles, role có thể thao túng từ client, JWT lộ password hash | **Cao** |
| WSTG-IDNT-02 | User Registration | Không rate limit, không email verification, login không enumeration | Trung bình |
| WSTG-IDNT-03 | Account Provisioning | Provisioning tự động, reset password có rate limit | Trung bình |
| WSTG-IDNT-04 | Account Enumeration | Login không enumeration, API tiết lộ danh sách users | Trung bình |
| WSTG-IDNT-05 | Username Enumeration | Login không enumeration, cần thử thêm registration | Thấp |

### Các lỗ hổng chính tìm được

| # | Lỗ hổng | Hạng mục | Mức độ |
|---|---|---|---|
| 1 | **Role có thể thao túng từ client** — Gửi `role: "admin"` trong request body → tạo admin | IDNT-01 | **Cao** |
| 2 | **JWT lộ password hash** — Password hash trong JWT payload decode được | IDNT-01 | **Cao** |
| 3 | **JWT lộ deluxeToken và totpSecret** — Token nhạy cảm trong JWT | IDNT-01 | Trung bình |
| 4 | **Không có rate limiting trên đăng ký** — 4 tài khoản liên tiếp không bị chặn | IDNT-02 | Trung bình |
| 5 | **Không xác thực email** — Tài khoản kích hoạt ngay sau đăng ký | IDNT-02 | Trung bình |
| 6 | **CORS `*`** — Cho phép mọi origin đọc API data | IDNT-01 | Trung bình |
| 7 | **`/api/users/` trả về 90 users** kèm thông tin nhạy cảm | IDNT-01 | Trung bình |
| 8 | **Stack trace trên error page** — Lộ file path, framework version | IDNT-01 | Trung bình |
| 9 | **Provisioning tự động không có approval** — Không có bước xác minh | IDNT-03 | Trung bình |
| 10 | **Login không enumeration** — Response giống nhau | IDNT-04/05 | ✅ Tốt |

### Ma trận roles chi tiết

| Chức năng | customer | admin | deluxe | accounting |
|---|---|---|---|---|
| Xem sản phẩm | ✅ | ✅ | ✅ | ✅ |
| Đặt hàng | ✅ | ✅ | ✅ | ✅ |
| Viết đánh giá | ✅ | ✅ | ✅ | ✅ |
| Xem đơn hàng của mình | ✅ | ✅ | ✅ | ✅ |
| Xem danh sách users | ❌ | ✅ | ❓ | ❓ |
| Xem password hash users | ❌ | ✅ | ❓ | ❓ |
| Xem deluxeToken users | ❌ | ✅ | ❓ | ❓ |
| Truy cập `/administration` | ❌ | ✅ | ❓ | ❓ |
| Quản lý sản phẩm | ❌ | ✅ | ❓ | ❓ |
| Quản lý đơn hàng | ❌ | ✅ | ❓ | ❓ |

### Luồng khai thác chính (phân tích)

```
┌──────────────────────────────────────────────────────────────┐
│  LUỒNG KHAI THÁC: Privilege Escalation qua Registration      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Gửi POST /api/users/                                    │
│     {"username":"attacker","email":"a@b.com",                │
│      "password":"Test@1234","role":"admin"}                  │
│                    ↓                                         │
│  2. Server chấp nhận → tạo user với role=admin              │
│                    ↓                                         │
│  3. Login → nhận JWT chứa role=admin                        │
│                    ↓                                         │
│  4. Truy cập /api/users/ → đọc toàn bộ users                │
│                    ↓                                         │
│  5. Truy cập /administration → quản lý toàn hệ thống        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Đề xuất khắc phục

| # | Vấn đề | Đề xuất |
|---|---|---|
| 1 | Role từ client-side | Server tự gán role, bỏ qua field `role` từ request |
| 2 | JWT lộ password hash | Chỉ giữ `sub`, `role`, `exp` trong JWT |
| 3 | JWT lộ deluxeToken | Lưu trong DB, không đưa vào JWT |
| 4 | Rate limiting đăng ký | Thêm rate limit: max 3-5 đăng ký/IP/giờ |
| 5 | Email verification | Yêu cầu xác nhận email trước khi kích hoạt |
| 6 | CORS `*` | Giới hạn origin cụ thể |
| 7 | `/api/users/` quá rộng | Thêm phân trang, lọc theo role |
| 8 | Stack trace | Custom error page ở production |
| 9 | Provisioning | Thêm email verification + admin approval cho role cao |

---

### Liên kết với các hạng mục khác

```
WSTG-IDNT (Identity Management) [Đang kiểm thử]
    │
    ├──→ WSTG-AUTHZ-03 (Privilege Escalation)
    │       Tìm được: Có thể nâng quyền ngay tại đăng ký
    │
    ├──→ WSTG-SESS-01 (Session Schema)
    │       Tìm được: JWT chứa role + password hash
    │
    ├──→ WSTG-CLIENT-07 (CORS)
    │       Tìm được: CORS cho phép tất cả origins
    │
    └──→ WSTG-CONF-02 (Application Platform)
            Tìm được: Stack trace lộ thông tin server
```
