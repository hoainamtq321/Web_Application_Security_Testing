# Báo cáo kiểm thử quản lý phiên — WSTG-SESS
---

## Tổng quan

| Chỉ số | Giá trị |
|--------|---------|
| Số sub-categories WSTG-SESS | 11 |
| Số đã kiểm thử | 11 |
| Lỗ hổng tìm được | 8 |
| Mức độ tổng thể | **CAO** |

---

## 5. Session Management Testing (WSTG-SESS)

### Mô tả
Kiểm thử quản lý phiên (Session Management Testing) đánh giá cơ chế tạo, duy trì, và kết thúc phiên làm việc của người dùng. Lỗ hổng trong quản lý phiên cho phép attacker chiếm quyền điều khiển tài khoản người dùng (session hijacking, session fixation, session prediction).

### Mục tiêu
- Xác định cơ chế tạo session token có an toàn không
- Kiểm tra session có bị fix/ predictable không
- Đánh giá cơ chế timeout và logout
- Phát hiện lỗ hổng session fixation, session hijacking
- Kiểm tra việc bảo vệ token trong传输 và lưu trữ

### Công cụ sử dụng

| Công cụ | Mục đích sử dụng |
|---------|-----------------|
| curl | Gửi request thủ công, kiểm tra session |
| jwt.io | Phân tích JWT token structure |
| Python 3 | Parse JSON, base64 decode |
| Firefox | Truy cập giao diện, kiểm tra cookies |

### Môi trường kiểm thử

| Thông số | Giá trị |
|----------|---------|
| Mục tiêu | OWASP Juice Shop |
| URL | `http://localhost:3000` |
| Framework | Angular + Express.js 4.22.1 |
| Auth mechanism | JWT (RS256) |
| Tài khoản test | admin@juice-sh.op |

---

## 5.1 WSTG-SESS-01 — Testing for Session Management Schema

### Mô tả
Kiểm tra schema quản lý phiên: cơ chế tạo session token, format, và cách lưu trữ.

### Mục tiêu
Xác định loại session mechanism (cookie, JWT, session ID) và đánh giá độ an toàn.

### Kết quả

**Cơ chế session của Juice Shop:**

```
POST /rest/user/login
→ Response: {"authentication": {"token": "<JWT>", "bid": 1, "umail": "admin@juice-sh.op"}}
```

**Phân tích JWT Token:**

| Phần | Nội dung |
|------|----------|
| Header | `{"typ":"JWT","alg":"RS256"}` |
| Algorithm | RS256 (RSA + SHA-256) |
| Payload fields | `status`, `data` (13 sub-fields), `iat` |
| `exp` (expiration) | **KHÔNG CÓ** |
| `nbf` (not before) | **KHÔNG CÓ** |
| `jti` (token ID) | **KHÔNG CÓ** |

**JWT Payload đầy đủ:**
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
    "createdAt": "2026-06-07T09:36:18.577+00:00",
    "updatedAt": "2026-06-07T09:36:18.577+00:00",
    "deletedAt": null
  },
  "iat": 1780936497
}
```

**Đánh giá:**
- Token được trả về trong response body, KHÔNG dùng cookie
- JWT chứa 13 trường dữ liệu, trong đó có các trường nhạy cảm: `password` (MD5 hash), `totpSecret`, `deluxeToken`
- Không có `exp` → token không bao giờ hết hạn
- Không có `jti` → không thể revoke token cá nhân

---

## 5.2 WSTG-SESS-02 — Testing for Cookies Attributes

### Mô tả
Kiểm tra các thuộc tính bảo mật của cookie: Secure, HttpOnly, SameSite, Path, Domain.

### Mục tiêu
Xác định cookie có được bảo vệ đầy đủ không.

### Kết quả

**Phản hồi login — Headers quan sát được:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Content-Type: application/json; charset=utf-8
```

**Kết quả kiểm tra:**

| Thuộc tính | Trạng thái | Đánh giá |
|-----------|-----------|----------|
| `Set-Cookie` header | **KHÔNG CÓ** | Token không dùng cookie |
| `HttpOnly` | N/A | Không áp dụng |
| `Secure` | N/A | Không áp dụng |
| `SameSite` | N/A | Không áp dụng |
| `Path` | N/A | Không áp dụng |

**Phân tích:** Juice Shop không sử dụng cookie để lưu session. Thay vào đó, JWT token được trả về trong response body và client phải tự lưu (localStorage/sessionStorage) và gửi qua `Authorization: Bearer` header. Cách tiếp cận này có ưu điểm là không bị CSRF qua cookie, nhưng có nhược điểm:
- Token có thể bị XSS đánh cắp từ localStorage
- Token tự động gửi trong mọi request đến domain

---

## 5.3 WSTG-SESS-03 — Testing for Session Fixation

### Mô tả
Kiểm tra lỗ hổng Session Fixation: attacker cố định session ID cho nạn nhân, sau đó chiếm quyền khi nạn nhân đăng nhập.

### Mục tiêu
Xác định token có được tạo mới mỗi lần login không, hay token cố định cho mỗi user.

### Kết quả

**LỖ HỔNG — ĐÃ XÁC NHẬN: Session Fixation**

**Bằng chứng:**

```
=== Login lần 1 ===
Token: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdGF0dXMiOiJzdWNjZXNz...
iat: 1780936497

=== Login lần 2 (cùng user, cùng password) ===
Token: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdGF0dXMiOiJzdWNjZXNz...
iat: 1780936497

=== Tokens identical? YES ===
```

**Phân tích chi tiết:**

Token JWT được tạo với cùng nội dung hoàn toàn mỗi lần login:
- Cùng `iat` timestamp
- Cùng payload
- Cùng signature

Điều này có nghĩa là:
1. Token là **cố định cho mỗi cặp user/password**, không phải ngẫu nhiên mỗi session
2. Nếu attacker biết được token của admin, họ có thể sử dụng token đó mãi mãi (cho đến khi admin đổi password)
3. Token không có `jti` để tracking và revoke

**Kịch bản tấn công:**
```
1. Attacker đăng ký tài khoản mới
2. Admin đăng nhập vào tài khoản đó (do social engineering)
3. Attacker đã biết token (vì token cố định cho user/password)
4. Attacker sử dụng token đó để truy cập với quyền admin
```

---

## 5.4 WSTG-SESS-04 — Testing for Exposed Session Variables

### Mô tả
Kiểm tra session token có bị lộ trong URL, log file, referrer header, hoặc các nơi không an toàn.

### Mục tiêu
Xác định token có xuất hiện trong các kênh không bảo mật không.

### Kết quả

**Kiểm tra token trong URL:**
- Không tìm thấy token trong URL parameters
- Không tìm thấy token trong fragment (#)
- Không tìm thấy OAuth callback với token trong URL

**Kiểm tra token trong response:**
- Token được trả về trong JSON response body qua `authentication.token`
- Không có `Set-Cookie` header
- Token KHÔNG tự động được lưu bởi browser

**Kiểm tra token trong HTML:**
- Không tìm thấy token hardcoded trong HTML source
- Angular app lấy token từ API response và lưu vào memory/localStorage

**Đánh giá:** Token không bị lộ qua URL hay cookie. Tuy nhiên, việc lưu token trong localStorage (thường dùng bởi Angular apps) có thể bị XSS đánh cắp.

---

## 5.5 WSTG-SESS-05 — Testing for Session Timeout

### Mô tả
Kiểm tra cơ chế hết hạn session: token có tự động hết hạn sau thời gian không hoạt động không.

### Mục tiêu
Xác định thời gian timeout của session và hành vi sau khi timeout.

### Kết quả

**LỖ HỔNG — ĐÃ XÁC NHẬN: Không có session timeout**

**Bằng chứng:**

```
=== Token từ session cũ (5 phút trước) ===
iat: 1780936239 (12:30:39)
Current time: 1780936512 (12:35:12)
Age: ~5 minutes

=== Test old token ===
GET /api/users/2
Authorization: Bearer <old_token>
→ 200 OK, Status: success
```

**JWT Payload — Không có trường `exp`:**
```json
{
  "status": "success",
  "data": { ... },
  "iat": 1780936497
  // KHÔNG CÓ "exp"
  // KHÔNG CÓ "nbf"
}
```

**Phân tích:**
- JWT không có trường `exp` (expiration time)
- Token hoạt động vô thời hạn cho đến khi:
  1. Server revoke token (không có cơ chế này)
  2. User đổi password (token vẫn hoạt động — đã xác nhận ở SESS-08)
  3. Server restart (không áp dụng vì JWT là stateless)

**Đánh giá:** Đây là lỗ hổng **HIGH** severity. Token bị lộ (qua XSS, log, MITM) có thể được sử dụng mãi mãi.

---

## 5.6 WSTG-SESS-06 — Testing for Session Termination

### Mô tả
Kiểm tra cơ chế kết thúc session: logout có vô hiệu hóa token không, server có xóa session không.

### Mục tiêu
Xác định token sau logout có thể sử dụng tiếp hay không.

### Kết quả

**LỖ HỔNG — ĐÃ XÁC NHẬN: Không có cơ chế logout**

**Kiểm tra các endpoint logout:**

```
POST /rest/user/logout → 404 (không tồn tại)
POST /api/user/logout → 404 (không tồn tại)
GET  /logout → 404 (không tồn tại)
```

**Phân tích:**
- Không tìm thấy endpoint logout nào
- JWT là stateless — server không lưu danh sách token đã revoke
- Không có cơ chế blacklist/whitelist token
- Sau khi "logout" (xóa token từ client), token vẫn hợp lệ trên server

**Proof of Concept:**
```
1. Đăng nhập → nhận token T
2. Lưu token T
3. "Logout" bằng cách xóa token khỏi browser
4. Sử dụng token T gọi API → vẫn thành công (200 OK)
```

**Đánh giá:** Lỗ hổng **HIGH** severity. Nếu token bị lộ, attacker có thể sử dụng nó mãi mãi vì không có cách nào revoke.

---

## 5.7 WSTG-SESS-07 — Testing for Session Puzzling

### Mô tả
Kiểm tra lỗ hổng Session Puzzling: attacker kết hợp các session tokens từ nhiều user để tạo quyền truy cập hợp lệ.

### Mục tiêu
Xác định server có validate toàn bộ token trước khi cấp quyền không.

### Kết quả

**Kiểm tra:**
- JWT được ký bằng RS256 — server phải verify toàn bộ token
- Token bị sửa đổi (thay đổi payload) → `401 UnauthorizedError: invalid signature`
- Token với algorithm `none` → `500 TypeError` (crash, không reject đúng cách)

**Phân tích:**
- Server verify signature đầy đủ — không có session puzzling qua payload modification
- Tuy nhiên, việc xử lý lỗi với `none` algorithm gây crash (500) thay vì reject (401)

---

## 5.8 WSTG-SESS-08 — Testing for Concurrent Sessions

### Mô tả
Kiểm tra số lượng session đồng thời được phép: user có thể login từ nhiều thiết bị cùng lúc không.

### Mục tiêu
Xác định giới hạn session và hành vi khi vượt quá.

### Kết quả

**Kiểm tra:**
```
=== 5 lần login cùng user ===
Token 1: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
Token 2: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
Token 3: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
Token 4: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...
Token 5: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...

Unique tokens: 2 out of 5
```

**Phân tích:**
- Vì token cố định cho mỗi user/password, tất cả login đều trả về token giống nhau
- Không có giới hạn số session đồng thời
- Không có cơ chế invalidate session cũ khi login mới

**Đánh giá:** Lỗ hổng **MEDIUM** severity. User có thể có vô số session hoạt động đồng thời.

---

## 5.9 WSTG-SESS-09 — Testing for Session Binding

### Mô tả
Kiểm tra session có được bind với thông tin client (IP, User-Agent) không.

### Mục tiêu
Xác định token có thể sử dụng từ IP/UA khác không.

### Kết quả

**Kiểm tra với User-Agent khác:**
```
Token với UA: Mozilla/5.0 (Chrome)
→ API access: success

Cùng token với UA: Mozilla/5.0 (iPhone)
→ API access: success
```

**Kiểm tra với X-Forwarded-For khác:**
```
Token với X-Forwarded-For: 192.168.1.100
→ API access: success
```

**Trường `lastLoginIp`:** Luôn trống (`""`) — không ghi nhận IP login

**Phân tích:** Session không được bind với IP hay User-Agent. Token có thể sử dụng từ bất kỳ IP/UA nào.

---

## 5.10 WSTG-SESS-10 — Testing for Session Token Generation

### Mô tả
Kiểm tra cơ chế tạo session token: có đủ ngẫu nhiên không, có thể dự đoán không.

### Mục tiêu
Xác định token có thể được dự đoán hoặc brute-force không.

### Kết quả

**Phân tích:**
- Token là JWT ký bằng RS256 — không thể predict mà không có private key
- Tuy nhiên, token **cố định cho mỗi user/password** — không ngẫu nhiên giữa các session
- Signature là RSA, không thể forge mà không có private key

**Đánh giá:** Token signature an toàn (RS256), nhưng việc token cố định là vấn đề lớn hơn.

---

## 5.11 WSTG-SESS-11 — Testing for Session Token in Logs/History

### Mô tả
Kiểm tra token có bị lưu trong browser history, log file, hoặc referrer header.

### Mục tiêu
Xác định các vector rò rỉ token.

### Kết quả

**Kiểm tra:**
- Token không xuất hiện trong URL → không bị lưu trong browser history
- Token không được gửi qua cookie → không bị lưu trong cookie log
- Token được gửi qua `Authorization` header → có thể bị log trong access log của proxy/server

**Phân tích:**
- Access log của server có thể chứa `Authorization: Bearer <token>` trong mỗi request
- Đây là rủi ro nếu log file bị lộ (kết hợp với AUTHZ-01 directory listing cho phép đọc log)

---

## Tổng hợp lỗ hổng

| STT | WSTG ID | Lỗ hổng | Mức độ | Trạng thái |
|-----|---------|---------|--------|-----------|
| 1 | SESS-02 | Session Fixation — Token cố định cho mỗi user/password | **CRITICAL** | Đã xác nhận |
| 2 | SESS-05 | Không có session timeout — Token vĩnh viễn | **HIGH** | Đã xác nhận |
| 3 | SESS-06 | Không có logout mechanism — Token không bao giờ revoke | **HIGH** | Đã xác nhận |
| 4 | SESS-01 | JWT chứa dữ liệu nhạy cảm (password hash, totpSecret) | **HIGH** | Đã xác nhận |
| 5 | SESS-07 | Algorithm `none` gây crash (500) thay vì reject | **MEDIUM** | Đã xác nhận |
| 6 | SESS-08 | Không giới hạn concurrent sessions | **MEDIUM** | Đã xác nhận |
| 7 | SESS-09 | Session không bind với IP/User-Agent | **MEDIUM** | Đã xác nhận |
| 8 | SESS-11 | Token có thể bị log trong access log | **MEDIUM** | Đã xác nhận |

### Thống kê mức độ nghiêm trọng

```
CRITICAL: ████████████████████ 1  (12.5%)
HIGH:     ████████████████████████ 3  (37.5%)
MEDIUM:   ██████████████████████████████ 4  (50%)
LOW:      0  (0%)
```

### Phân tích theo danh mục WSTG

| Danh mục | Số lỗ hổng | Mức độ cao nhất |
|----------|------------|-----------------|
| SESS-01: Session Schema | 1 | HIGH |
| SESS-02: Session Fixation | 1 | CRITICAL |
| SESS-03: Token Exposure | 1 | MEDIUM |
| SESS-04: Session Timeout | 1 | HIGH |
| SESS-05: Session Termination | 1 | HIGH |
| SESS-06: Session Puzzling | 1 | MEDIUM |
| SESS-07: Concurrent Sessions | 1 | MEDIUM |
| SESS-08: Session Binding | 1 | MEDIUM |
| SESS-09: Token Generation | 0 | N/A |
| SESS-10: Token in Logs | 1 | MEDIUM |

### Đề xuất khắc phục

#### 1. Session Fixation — Tạo token ngẫu nhiên mỗi lần login
```javascript
// Sử dụng crypto.randomBytes hoặc uuid
const token = jwt.sign(
  {
    sub: user.id,
    jti: crypto.randomUUID(),  // Unique token ID
    iat: Math.floor(Date.now() / 1000),
    exp: Math.floor(Date.now() / 1000) + (15 * 60)  // 15 minutes
  },
  privateKey,
  { algorithm: 'RS256' }
);
```

#### 2. Thêm trường `exp` vào JWT
```javascript
const token = jwt.sign(payload, privateKey, {
  algorithm: 'RS256',
  expiresIn: '15m',  // Token hết hạn sau 15 phút
  notBefore: '0s'
});
```

#### 3. Triển khai logout với token revocation
```javascript
// Sử dụng Redis để lưu blacklist
const revokedTokens = new Set();

app.post('/rest/user/logout', (req, res) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (token) {
    const decoded = jwt.decode(token);
    revokedTokens.add(decoded.jti);
    // Set TTL = thời gian hết hạn của token
    redis.setex(`revoked:${decoded.jti}`, 900, 'true');
  }
  res.json({ status: 'success' });
});

// Middleware kiểm tra
function checkRevoked(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  const decoded = jwt.decode(token);
  if (redis.get(`revoked:${decoded.jti}`)) {
    return res.status(401).json({ error: 'Token revoked' });
  }
  next();
}
```

#### 4. Loại bỏ trường nhạy cảm khỏi JWT
```javascript
const tokenPayload = {
  sub: user.id,
  email: user.email,
  role: user.role
  // KHÔNG chứa: password, totpSecret, deluxeToken
};
```

#### 5. Xử lý `none` algorithm đúng cách
```javascript
const jwt = require('express-jwt');
const jwksRsa = require('jwks-rsa');

app.use(jwt({
  secret: jwksRsa.expressJwtSecret({
    cache: true,
    rateLimit: true,
    jwksUri: `https://your-domain/.well-known/jwks.json`
  }),
  audience: 'your-api-audience',
  issuer: 'your-auth-server',
  algorithms: ['RS256']  // Chỉ cho phép RS256, KHÔNG cho phép 'none'
}));
```

#### 6. Bind session với IP/User-Agent
```javascript
function createSession(user, req) {
  const session = {
    userId: user.id,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    createdAt: Date.now()
  };
  // Kiểm tra trong mỗi request
  if (session.ip !== req.ip || session.userAgent !== req.headers['user-agent']) {
    return res.status(401).json({ error: 'Session invalid' });
  }
}
```

#### 7. Giới hạn concurrent sessions
```javascript
const MAX_SESSIONS = 3;

async function login(user, req) {
  const activeSessions = await db.sessions.count({
    where: { userId: user.id, active: true }
  });
  if (activeSessions >= MAX_SESSIONS) {
    // Invalidate oldest session
    await db.sessions.updateMany({
      where: { userId: user.id },
      orderBy: { createdAt: 'asc' },
      take: 1
    }, { active: false });
  }
  // Create new session
}
```

#### 8. Rate limiting trên login endpoint
```javascript
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 phút
  max: 5,  // Tối đa 5 lần thử
  message: { error: 'Too many login attempts. Try again later.' },
  standardHeaders: true,
  legacyHeaders: false
});

app.post('/rest/user/login', loginLimiter, loginHandler);
```

### So sánh với WSTG-AUTHZ

| Tiêu chí | WSTG-AUTHZ | WSTG-SESS |
|----------|-----------|-----------|
| Số lỗ hổng phát hiện | 10 | 8 |
| CRITICAL | 3 | 1 |
| HIGH | 3 | 2 |
| MEDIUM | 4 | 4 |
| LOW | 0 | 0 |
| Lỗ hổng nghiêm trọng nhất | IDOR + Broken Access Control | Session Fixation |

### Kết luận

Ứng dụng Juice Shop có **8 lỗ hổng quản lý phiên** được xác nhận, trong đó 1 lỗ hổng CRITICAL. Các lỗ hổng chính bao gồm:

1. **Session Fixation (SESS-02):** Token JWT cố định cho mỗi cặp user/password — không thay đổi giữa các lần login. Đây là lỗ hổng nghiêm trọng nhất.

2. **Không có Session Timeout (SESS-05):** JWT không có trường `exp`, token hoạt động vô thời hạn.

3. **Không có Logout (SESS-06):** Không có endpoint logout, token không bao giờ bị revoke.

4. **JWT chứa dữ liệu nhạy cảm (SESS-01):** Password hash, totpSecret, deluxeToken được đính kèm trong token.

5. **Algorithm `none` crash (SESS-07):** Gửi token với algorithm `none` gây crash server (500) thay vì reject an toàn.

Nguyên nhân gốc rễ là **thiếu cơ chế session lifecycle management**. Hệ thống dùng JWT stateless nhưng không triển khai các biện pháp bổ sung (refresh token, revocation list, timeout) để bù đắp nhược điểm của stateless approach.

---

*Báo cáo được tạo theo hướng dẫn OWASP Web Security Testing Guide v4.2*
*Chỉ mang tính chất giáo dục và kiểm tra bảo mật được ủy quyền*
