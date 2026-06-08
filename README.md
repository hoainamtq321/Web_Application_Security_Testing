# Khung Kiểm Thử Bảo Mật Web - Juice Shop
## Theo chuẩn OWASP Web Security Testing Guide (WSTG)

> Tài liệu này chỉ mang tính **giải thích, học tập và tham khảo**. Mọi nội dung được mô tả dưới dạng khung kiểm thử (test framework) — liệt kê phương pháp, công cụ và bước kiểm tra **để hiểu cách phát hiện lỗ hổng**,
>
> - Juice Shop là ứng dụng lab **có chủ đích chứa lỗ hổng** dùng cho mục đích giáo dục.
> - Tài liệu này không cung cấp payload, exploit code hay hướng dẫn từng bước để khai thác lỗ hổng.
> - Mục đích duy nhất: hiểu kiến trúc bảo mật web, nhận diện lỗ hổng theo OWASP WSTG, phục vụ kiểm thử có trách nhiệm.

---

## Thông tin chung

| Mục | Chi tiết |
|---|---|
| **Mục tiêu** | OWASP Juice Shop (`http://localhost:3000`) |
| **Chuẩn tham chiếu** | OWASP WSTG v4.2 |
| **Ngày tạo** | 2026-06-07 |
| **Bản chất** | **Khung kiểm thử tham khảo — KHÔNG thực hiện khai thác, KHÔNG cung cấp payload** |
| **Mục đích** | Giải thích phương pháp kiểm thử theo OWASP WSTG nhằm hiểu cách phát hiện lỗ hổng |

---

## Các hạng mục kiểm thử

### 1. Information Gathering (WSTG-INFO)
Thu thập thông tin về mục tiêu mà không tương tác trực tiếp.

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-INFO-01 | Search Engine Discovery | Tìm thông tin rò rỉ trên công cụ tìm kiếm | ⬜ |
| WSTG-INFO-02 | Fingerprint Web Server | Xác định phiên bản server (Express/Node.js) qua header | ⬜ |
| WSTG-INFO-03 | Review Webserver Metafiles | Kiểm tra `robots.txt`, `sitemap.xml`, `.well-known/` | ⬜ |
| WSTG-INFO-04 | Enumerate Applications | Liệt kê endpoints qua `application.json` | ⬜ |
| WSTG-INFO-05 | Review Webpage Content | Kiểm tra comment, HTML, JS có lộ thông tin không | ⬜ |
| WSTG-INFO-06 | Identify Application Entry Points | Liệt kê API routes, form fields, URL parameters | ⬜ |
| WSTG-INFO-07 | Map Execution Paths | Xác định luồng điều hướng giữa các trang | ⬜ |
| WSTG-INFO-08 | Fingerprint Web Framework | Xác định Angular frontend, Node.js/Express backend | ⬜ |
| WSTG-INFO-09 | Fingerprint Web Application | Xác định phiên bản Juice Shop qua footer, meta tags | ⬜ |
| WSTG-INFO-10 | Map Application Architecture | Xác định cấu trúc API REST, WebSocket, CDN | ⬜ |

---

### 2. Configuration and Deployment Management Testing (WSTG-CONF)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-CONF-01 | Test Network Infrastructure | Kiểm tra exposed ports, firewall rules | ⬜ |
| WSTG-CONF-02 | Test Application Platform | Kiểm tra Node.js version, npm packages outdated | ⬜ |
| WSTG-CONF-03 | Test File Extensions | Kiểm tra file `.zip`, `.sql`, `.bak` có thể truy cập | ⬜ |
| WSTG-CONF-04 | Review Old Backup Files | Tìm backup file, `.git/` directory | ⬜ |
| WSTG-CONF-05 | Enumerate Admin Interfaces | Tìm `/administration`, `/admin` paths | ⬜ |
| WSTG-CONF-06 | Test HTTP Methods | Kiểm tra OPTIONS, PUT, DELETE, TRACE | ⬜ |
| WSTG-CONF-07 | Test HSTS | Kiểm tra Strict-Transport-Security header | ⬜ |
| WSTG-CONF-08 | Test RIA Cross Domain Policy | Kiểm tra `crossdomain.xml`, `clientaccesspolicy.xml` | ⬜ |
| WSTG-CONF-09 | Test File Permissions | Kiểm tra file `/ftp/`, `/assets/` có thể liệt kê | ⬜ |
| WSTG-CONF-10 | Test Subdomain Takeover | Kiểm tra subdomain (không áp dụng local) | ⬜ |
| WSTG-CONF-11 | Test Cloud Storage | Kiểm tra S3/Azure blob public access | ⬜ |

---

### 3. Identity Management Testing (WSTG-IDNT)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-IDNT-01 | Test Role Definitions | Phân tích roles: Admin, Customer, — | ⬜ |
| WSTG-IDNT-02 | Test User Registration | Kiểm tra quy trình đăng ký, email verification | ⬜ |
| WSTG-IDNT-03 | Test Account Provisioning | Kiểm tra provisioning flow | ⬜ |
| WSTG-IDNT-04 | Test Account Enumeration | Kiểm tra có thể liệt kê user không (login error, forgot password) | ⬜ |
| WSTG-IDNT-05 | Test Username Enumeration | Phân biệt response giữa user tồn tại và không tồn tại | ⬜ |

---

### 4. Authentication Testing (WSTG-AUTH)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-AUTH-01 | Credentials Transported over Encrypted Channel | Kiểm tra HTTPS-only, password gửi HTTP hay HTTPS | ⬜ |
| WSTG-AUTH-02 | Default Credentials | Kiểm tra tài khoản mặc định (admin/admin) | ⬜ |
| WSTG-AUTH-03 | Weak Lock Out Mechanism | Test brute-force, lockout policy | ⬜ |
| WSTG-AUTH-04 | Bypassing Authentication Schema | Test bypass auth qua URL trực tiếp, token manipulation | ⬜ |
| WSTG-AUTH-05 | Vulnerable Remember Password | Test "Remember me" cookie/token | ⬜ |
| WSTG-AUTH-06 | Browser Cache Weaknesses | Kiểm tra cached credentials, back button | ⬜ |
| WSTG-AUTH-07 | Weak Password Policy | Test password dễ đoán | ⬜ |
| WSTG-AUTH-08 | Weak Security Question | Test security question answer | ⬜ |
| WSTG-AUTH-09 | Weak Password Change/Reset | Test forgot password flow | ⬜ |
| WSTG-AUTH-10 | Weaker Auth in Alternative Channel | Test API auth so với web UI auth | ⬜ |

---

### 5. Authorization Testing (WSTG-AUTHZ)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-AUTHZ-01 | Directory Traversal / File Include | Test path traversal trên file download endpoints | ⬜ |
| WSTG-AUTHZ-02 | Bypassing Authorization Schema | Truy cập trang admin với role customer, bypass role check | ⬜ |
| WSTG-AUTHZ-03 | Privilege Escalation | Nâng quyền từ Customer lên Admin | ⬜ |
| WSTG-AUTHZ-04 | Insecure Direct Object Reference (IDOR) | Thay đổi `id` trong URL/body để truy cập object của user khác | ⬜ |

---

### 6. Session Management Testing (WSTG-SESS)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-SESS-01 | Session Management Schema | Phân tích cách session được tạo/quản lý | ⬜ |
| WSTG-SESS-02 | Cookies Attributes | Kiểm tra HttpOnly, Secure, SameSite, Path, Domain | ⬜ |
| WSTG-SESS-03 | Session Fixation | Test session ID không đổi sau login | ⬜ |
| WSTG-SESS-04 | Exposed Session Variables | Session ID lộ qua URL, logs, referrer | ⬜ |
| WSTG-SESS-05 | Cross Site Request Forgery (CSRF) | Test CSRF token thiếu trên form quan trọng | ⬜ |
| WSTG-SESS-06 | Logout Functionality | Kiểm tra session có bị xóa khi logout không | ⬜ |
| WSTG-SESS-07 | Session Timeout | Test session expiration time | ⬜ |
| WSTG-SESS-08 | Session Puzzling | Kiểm tra logic session giữa các module | ⬜ |
| WSTG-SESS-09 | Session Hijacking | Test session replay, session fixation | ⬜ |

---

### 7. Input Validation Testing (WSTG-INPVAL)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-INPVAL-01 | Reflected XSS | Input phản xạ trực tiếp không encode | ⬜ |
| WSTG-INPVAL-02 | Stored XSS | Input lưu DB rồi render (review, product name) | ⬜ |
| WSTG-INPVAL-03 | HTTP Verb Tampering | Gửi request với method khác (POST → PUT, GET → POST) | ⬜ |
| WSTG-INPVAL-04 | HTTP Parameter Pollution | Gửi trùng parameter (`?id=1&id=2`) | ⬜ |
| WSTG-INPVAL-05 | SQL Injection | Input vô SQL query (search, login) | ⬜ |
| WSTG-INPVAL-06 | LDAP Injection | Injection vào LDAP query | ⬜ |
| WSTG-INPVAL-07 | XML Injection | Injection vào XML processing | ⬜ |
| WSTG-INPVAL-08 | SSI Injection | Server-Side Includes injection | ⬜ |
| WSTG-INPVAL-09 | XPath Injection | XPath query injection | ⬜ |
| WSTG-INPVAL-10 | IMAP/SMTP Injection | Injection vào IMAP/SMTP commands | ⬜ |
| WSTG-INPVAL-11 | Code Injection (LFI/RFI) | Local/Remote File Inclusion | ⬜ |
| WSTG-INPVAL-12 | Command Injection | OS command injection (`;`, `|`, `$()`, backticks) | ⬜ |
| WSTG-INPVAL-13 | Format String Injection | Format string attacks | ⬜ |
| WSTG-INPVAL-14 | Incubated Vulnerability | Stored payload kích hoạt sau (time-based) | ⬜ |
| WSTG-INPVAL-15 | HTTP Splitting/Smuggling | CRLF injection, request smuggling | ⬜ |
| WSTG-INPVAL-16 | HTTP Incoming Requests | SSRF via HTTP request smuggling | ⬜ |
| WSTG-INPVAL-17 | Host Header Injection | Host header manipulation | ⬜ |
| WSTG-INPVAL-18 | Server-Side Template Injection | SSTI trong template engine | ⬜ |
| WSTG-INPVAL-19 | Server-Side Request Forgery (SSRF) | Request từ server nội bộ (localhost, metadata) | ⬜ |

---

### 8. Error Handling Testing (WSTG-ERRH)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-ERRH-01 | Improper Error Handling | Kiểm tra error message có lộ thông tin không | ⬜ |
| WSTG-ERRH-02 | Stack Traces | Kiểm tra stack trace hiển thị trong error response | ⬜ |

---

### 9. Cryptography Testing (WSTG-CRYP)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-CRYP-01 | Weak Transport Layer Security | Kiểm tra TLS version, cipher suite | ⬜ |
| WSTG-CRYP-02 | Padding Oracle | Test padding oracle attack (không áp dụng local HTTP) | ⬜ |
| WSTG-CRYP-03 | Sensitive Info via Unencrypted Channels | Kiểm tra có data nhạy cảm gửi HTTP không | ⬜ |
| WSTG-CRYP-04 | Weak Encryption | Kiểm tra thuật toán mã hóa yếu | ⬜ |

---

### 10. Business Logic Testing (WSTG-BUSLOGIC)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-BUSLOGIC-01 | Business Logic Data Validation | Test input hợp lệ về mặt kỹ thuật nhưng sai logic | ⬜ |
| WSTG-BUSLOGIC-02 | Ability to Forge Requests | Tạo request thủ công bypass business flow | ⬜ |
| WSTG-BUSLOGIC-03 | Test Integrity Checks | Bypass kiểm tra toàn vẹn (price, quantity) | ⬜ |
| WSTG-BUSLOGIC-04 | Test Process Timing | Timing attack trên OTP, 2FA | ⬜ |
| WSTG-BUSLOGIC-05 | Function Use Limits | Vượt giới hạn số lần sử dụng chức năng | ⬜ |
| WSTG-BUSLOGIC-06 | Circumvention of Work Flows | Bỏ qua bước trong workflow (checkout → order trực tiếp) | ⬜ |
| WSTG-BUSLOGIC-07 | Defenses Against Misuse | Test upload file độc hại, oversized input | ⬜ |
| WSTG-BUSLOGIC-08 | Upload Unexpected File Types | Upload file không đúng loại (.php, .exe) | ⬜ |
| WSTG-BUSLOGIC-09 | Upload Malicious Files | Upload file chứa payload độc hại | ⬜ |

---

### 11. Client-Side Testing (WSTG-CLIENT)

| ID | Tên kiểm thử | Juice Shop liên quan | Trạng thái |
|---|---|---|---|
| WSTG-CLIENT-01 | DOM-based XSS | XSS qua DOM manipulation (hash, query) | ⬜ |
| WSTG-CLIENT-02 | JavaScript Execution | Code injection qua JavaScript | ⬜ |
| WSTG-CLIENT-03 | HTML Injection | HTML injection vào rendered content | ⬜ |
| WSTG-CLIENT-04 | Client-side URL Redirect | Open redirect | ⬜ |
| WSTG-CLIENT-05 | CSS Injection | CSS-based attacks | ⬜ |
| WSTG-CLIENT-06 | Client-side Resource Manipulation | Thay đổi resource qua client-side | ⬜ |
| WSTG-CLIENT-07 | Cross Origin Resource Sharing (CORS) | Kiểm tra CORS policy, origin validation | ⬜ |
| WSTG-CLIENT-08 | Cross Site Flashing | Flash-based XSS (không áp dụng) | ⬜ |
| WSTG-CLIENT-09 | Clickjacking | Test `X-Frame-Options`, CSP frame-ancestors | ⬜ |
| WSTG-CLIENT-10 | WebSockets | Kiểm tra WebSocket security | ⬜ |
| WSTG-CLIENT-11 | Web Messaging | postMessage security | ⬜ |
| WSTG-CLIENT-12 | Browser Storage | localStorage, sessionStorage lộ thông tin | ⬜ |
| WSTG-CLIENT-13 | Cross Site Script Inclusion (XSSI) | JSONP, script inclusion attacks | ⬜ |

---

## Tổng hợp

| Hạng mục | Mã | Số test case |
|---|---|---|
| Information Gathering | WSTG-INFO | 10 |
| Configuration & Deployment | WSTG-CONF | 11 |
| Identity Management | WSTG-IDNT | 5 |
| Authentication | WSTG-AUTH | 10 |
| Authorization | WSTG-AUTHZ | 4 |
| Session Management | WSTG-SESS | 9 |
| Input Validation | WSTG-INPVAL | 19 |
| Error Handling | WSTG-ERRH | 2 |
| Cryptography | WSTG-CRYP | 4 |
| Business Logic | WSTG-BUSLOGIC | 9 |
| Client-Side | WSTG-CLIENT | 13 |
| **TỔNG** | | **96** |

---

## Công cụ đề xuất

| Loại | Công cụ |
|---|---|
| Proxy | Burp Suite Community / OWASP ZAP |
| Browser | Firefox + F12 + Hackbar + Wappalyzer |
| Recon | Nmap, WhatWeb, WPScan |
| Fuzzing | ffuf, gobuster, WFuzz |
| Automation | OWASP ZAP automated scan |
| API | Postman / Insomnia |

---

*Tài liệu tham khảo: [OWASP WSTG v4.2](https://owasp.org/www-project-web-security-testing-guide/stable/)*
