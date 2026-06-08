# Danh sách Challenges — OWASP Juice Shop

> Nguồn: `GET /api/challenges` | Tổng: **112 challenges** | Sắp xếp: theo độ khó tăng dần

---

## ⭐ (1 sao) — 13 challenges

| STT | Tên Challenge | Danh mục | Mục tham chiếu WSTG | Nhận xét |
|---|---|---|---|---|
| 1 | Score Board | Miscellaneous | INFO-01 | **ĐÃ LÀM** — Tìm trang Score Board ẩn. Tutorial đầu tiên. |
| 2 | Web3 Sandbox | Broken Access Control | | Tìm code sandbox triển khai ngẫu nhiên. |
| 3 | Confidential Document | Sensitive Data Exposure | WSTG-CONF-09 | **ĐÃ LÀM** Truy cập tài liệu mật — liên quan `/ftp/` directory. |
| 4 | DOM XSS | XSS | | DOM XSS với `<iframe src="javascript:alert('xss')">`. Tutorial. |
| 5 | Error Handling | Security Misconfiguration | ERRH-01 | **ĐÃ LÀM** — Gây error không được xử lý. Stack trace lộ thông tin. |
| 6 | Privacy Policy | Miscellaneous | | Đọc privacy policy — Tutorial. |
| 7 | Repetitive Registration | Improper Input Validation | IDNT-02 | **ĐÃ LÀM** — Đăng ký user với thông tin lặp lại (DRY). |
| 8 | Zero Stars | Improper Input Validation | | Đánh giá 0 sao — input validation bypass. |
| 9 | Missing Encoding | Improper Input Validation | | Lấy ảnh mèo của Bjoern — encoding bypass. |
| 10 | Mass Dispel | Miscellaneous | | Đóng nhiều notification cùng lúc. |
| 11 | Exposed Metrics | Observability Failures | | Tìm Prometheus metrics endpoint — information disclosure. |
| 12 | Outdated Allowlist | Unvalidated Redirects | | Bypass allowlist redirect đến crypto address cũ. |
| 13 | Bonus Payload | XSS | | Dùng bonus payload trong DOM XSS challenge. |

---

## ⭐⭐ (2 sao) — 18 challenges

| STT | Tên Challenge | Danh mục | Mục tham chiếu WSTG | Nhận xét |
|---|---|---|---|---|
| 14 | Admin Section | Broken Access Control | AUTHZ-01 | **ĐÃ LÀM** — Truy cập trang quản trị với admin token. |
| 15 | Password Hash Leak | Sensitive Data Exposure | | Lỗi Excessive Data Exposure — API trả về password hash trong JWT. |
| 16 | View Basket | Broken Access Control | AUTHZ-04 | **ĐÃ LÀM** — Xem giỏ hàng của user khác — IDOR. |
| 17 | Login Admin | Injection | AUTH-04 | **ĐÃ LÀM** — Đăng nhập với tài khoản admin qua SQLi. |
| 18 | Login MC SafeSearch | Sensitive Data Exposure | | Đăng nhập với credentials gốc của MC SafeSearch — OSINT. |
| 19 | Empty User Registration | Improper Input Validation | | Đăng ký user với email và password rỗng. |
| 20 | NFT Takeover | Sensitive Data Exposure | | Chiếm wallet chứa NFT — Web3 challenge. |
| 21 | Deprecated Interface | Security Misconfiguration | | Dùng B2B interface cũ chưa tắt — API endpoint cũ. |
| 22 | Password Strength | Broken Authentication | | Đăng nhập admin với pass mặc định yếu — brute force. |
| 23 | Security Policy | Miscellaneous | INFO-01 | **ĐÃ LÀM** — Đọc security policy trước khi pentest. |
| 24 | Reflected XSS | XSS | | Reflected XSS với `<iframe src="javascript:alert('xss')">`. Tutorial. |
| 25 | Weird Crypto | Cryptographic Issues | | Tìm thuật toán/library dùng sai — crypto misuse. |
| 26 | Exposed credentials | Sensitive Data Exposure | | Tìm credentials hardcoded trong client-side code. |
| 27 | Login Amy | Sensitive Data Exposure | | Đăng nhập với Amy — OSINT tìm credentials. |
| 28 | Meta Geo Stalking | Sensitive Data Exposure | | Tìm câu trả lời security question qua EXIF metadata ảnh. |
| 29 | Visual Geo Stalking | Sensitive Data Exposure | | Tìm câu trả lời security question qua visual ảnh. |
| 30 | AI Debugging | Broken Access Control | | Xem thông tin backend chatbot với non-admin user. |
| 31 | Chatbot Prompt Injection | Injection | | Prompt injection chatbot để lấy coupon. |

---

## ⭐⭐⭐ (3 sao) — 24 challenges

| STT | Tên Challenge | Danh mục | Mục tham chiếu WSTG | Nhận xét |
|---|---|---|---|---|
| 32 | Admin Registration | Improper Input Validation | IDNT-01 | **ĐÃ LÀM** — Đăng ký với role=admin. Mass Assignment vulnerability. |
| 33 | API-only XSS | XSS | | Stored XSS qua API mà không cần frontend. Nguy hiểm trên Docker. |
| 34 | Bjoern's Favorite Pet | Broken Authentication | | Reset password qua security question — OSINT tìm câu trả lời. |
| 35 | Database Schema | Injection | | Exfiltrate toàn bộ DB schema qua SQL Injection. |
| 36 | Client-side XSS Protection | XSS | | Stored XSS bypass client-side security mechanism. |
| 37 | Ephemeral Accountant | Injection | | Đăng nhập với `acc0unt4nt@juice-sh.op` mà không đăng ký — SQLi. |
| 38 | Forged Feedback | Broken Access Control | | Đăng feedback với tên user khác — IDOR. |
| 39 | Forged Review | Broken Access Control | | Sửa review của user khác — IDOR. |
| 40 | Login Bender | Injection | | Đăng nhập với Bender — SQLi. |
| 41 | Login Jim | Injection | AUTH-04 | **ĐÃ LÀM** — Đăng nhập với Jim — SQLi. |
| 42 | Manipulate Basket | Broken Access Control | | Thêm sản phẩm vào giỏ hàng của user khác — IDOR. |
| 43 | Mint the Honey Pot | Improper Input Validation | | Mint NFT bằng cách thu thập BEEs. Cần Alchemy API Key. |
| 44 | CAPTCHA Bypass | Broken Anti Automation | | Gửi 10+ feedback trong 20s — brute force CAPTCHA. |
| 45 | Deluxe Fraud | Improper Input Validation | | Lấy Deluxe Membership không trả tiền — logic flaw. |
| 46 | Greedy Chatbot Manipulation | Injection | | Đánh lừa chatbot lấy coupon ≥50%. |
| 47 | CSRF | Broken Access Control | | CSRF đổi tên user từ origin khác. |
| 48 | Product Tampering | Broken Access Control | | Đổi link sản phẩm O-Saft — IDOR. |
| 49 | Five-Star Feedback | Broken Access Control | | Xóa hết feedback 5 sao — IDOR. |
| 50 | Payback Time | Improper Input Validation | | Đặt hàng với giá âm — logic flaw. |
| 51 | Privacy Policy Inspection | Security through Obscurity | | Chứng minh đã đọc privacy policy. |
| 52 | Reset Jim's Password | Broken Authentication | IDNT-04 | **ĐÃ LÀM** — Reset pass Jim qua security question. |
| 53 | GDPR Data Erasure | Broken Authentication | | Đăng nhập với tài khoản đã bị xóa (Chris) — GDPR erasure. |
| 54 | Upload Size | Improper Input Validation | | Upload file >100kB — không giới hạn size. |
| 55 | Upload Type | Improper Input Validation | | Upload file không phải .pdf/.zip — bypass extension check. |
| 56 | Security Advisory | Miscellaneous | | Tìm CVE đã được advisory nhưng chưa fix. |

---

## ⭐⭐⭐⭐ (4 sao) — 27 challenges

| STT | Tên Challenge | Danh mục | Mục tham chiếu WSTG | Nhận xét |
|---|---|---|---|---|
| 57 | NoSQL Manipulation | Injection | | NoSQL injection update nhiều reviews cùng lúc. |
| 58 | Christmas Special | Injection | | Đặt hàng Christmas special 2014 — Injection qua order. |
| 59 | Expired Coupon | Improper Input Validation | | Dùng coupon đã hết hạn — logic flaw. |
| 60 | HTTP-Header XSS | XSS | | Stored XSS qua HTTP header. |
| 61 | Server-side XSS Protection | XSS | | Stored XSS bypass server-side security mechanism. |
| 62 | Reset Bender's Password | Broken Authentication | | Reset pass Bender qua security question — OSINT. |
| 63 | Forgotten Developer Backup | Sensitive Data Exposure | | Truy cập backup file của developer — liên quan `/ftp/`. |
| 64 | Forgotten Sales Backup | Sensitive Data Exposure | | Truy cập backup file của salesman — liên quan `/ftp/`. |
| 65 | Misplaced Signature File | Observability Failures | | Tìm SIEM signature file bị đặt sai vị trí. |
| 66 | GDPR Data Theft | Sensitive Data Exposure | | Đánh cắp data cá nhân mà không dùng Injection. |
| 67 | Leaked Unsafe Product | Sensitive Data Exposure | | Tìm sản phẩm nguy hiểm bị gỡ — OSINT. |
| 68 | Legacy Typosquatting | Vulnerable Components | | Tìm typosquatting trong npm dependencies v6.2.0. |
| 69 | Vulnerable Library | Vulnerable Components | | Tìm library có vulnerability — OSINT/CVE. |
| 70 | Login Bjoern | Broken Authentication | | Đăng nhập với Gmail của Bjoern mà không đổi pass/SQLi. |
| 71 | NoSQL DoS | Injection | | NoSQL injection làm server sleep — DoS. |
| 72 | Nested Easter Egg | Cryptographic Issues | | Easter egg ẩn — cryptanalysis. |
| 73 | Steganography | Security through Obscurity | | Tìm character ẩn trong hình — steganography. |
| 74 | Reset Uvogin's Password | Sensitive Data Exposure | | Reset pass Uvogin qua security question — OSINT. |
| 75 | Allowlist Bypass | Unvalidated Redirects | | Bypass allowlist redirect — open redirect. |
| 76 | Poison Null Byte | Improper Input Validation | | Bypass security control với Null Byte injection. |
| 77 | User Credentials | Injection | | SQL Injection lấy toàn bộ user credentials. |
| 78 | XXE Data Access | XXE | | XXE đọc `/etc/passwd` hoặc `C:\Windows\system.ini`. |
| 79 | Access Log | Observability Failures | | Truy cập file log của server — Information Disclosure. |
| 80 | Easter Egg | Broken Access Control | | Tìm easter egg ẩn trong ứng dụng. |
| 81 | CSP Bypass | XSS | | Bypass CSP với `<script>alert('xss')</script>` trên legacy page. |

---

## ⭐⭐⭐⭐⭐ (5 sao) — 19 challenges

| STT | Tên Challenge | Danh mục | Mục tham chiếu WSTG | Nhận xét |
|---|---|---|---|---|
| 82 | Email Leak | Sensitive Data Exposure | | Lộ thông tin email qua cross-domain — XS-Leaks. |
| 83 | Forged Coupon | Cryptographic Issues | | Tạo coupon giả với discount ≥80%. |
| 84 | Blockchain Hype | Security through Obscurity | | Tìm thông tin Token Sale trước khi công bố — Code Analysis. |
| 85 | Extra Language | Broken Anti Automation | | Lấy file ngôn ngữ không bao giờ được deploy — brute force. |
| 86 | Leaked Access Logs | Observability Failures | | Tìm password bị lộ trên Internet, đăng nhập với nó. |
| 87 | NoSQL Exfiltration | Injection | | NoSQL injection lấy orders của user khác. |
| 88 | Cross-Site Imaging | Security Misconfiguration | | SVG injection đưa ảnh cross-domain lên delivery box. |
| 89 | Reset Bjoern's Password | Broken Authentication | | Reset pass Bjoern qua security question — OSINT. |
| 90 | Reset Morty's Password | Broken Anti Automation | | Reset pass Morty với obfuscated answer — brute force. |
| 91 | Retrieve Blueprint | Sensitive Data Exposure | | Download blueprint sản phẩm — file access. |
| 92 | Leaked API Key | Sensitive Data Exposure | | Tìm API key bị lộ trong code. |
| 93 | Login Support Team | Security Misconfiguration | | Đăng nhập support team — brute force/code analysis. |
| 94 | Two Factor Authentication | Broken Authentication | | Bypass 2FA của user "wurstbrot". |
| 95 | Unsigned JWT | Vulnerable Components | | JWT không có signature — impersonate user. |
| 96 | Change Bender's Password | Broken Authentication | | Đổi pass Bender mà không dùng SQLi hay Forgot Password. |
| 97 | Multiple Likes | Broken Anti Automation | | Like review 3 lần cùng user — timing attack. |
| 98 | SSRF | Broken Access Control | | SSRF request resource ẩn trên server. |
| 99 | Forged Signed JWT | Vulnerable Components | | JWT giả có RSA signature — cần khóa. |
| 100 | Imaginary Challenge | Cryptographic Issues | | Giải challenge #999 không tồn tại — crypto trick. |

---

## ⭐⭐⭐⭐⭐⭐ (6 sao) — 11 challenges

| STT | Tên Challenge | Danh mục | Mục tham chiếu WSTG | Nhận xét |
|---|---|---|---|---|
| 101 | Premium Paywall | Cryptographic Issues | | Unlock premium content — crypto challenge. |
| 102 | Supply Chain Attack | Vulnerable Components | | Tìm vulnerability trong supply chain — OSINT/CVE. |
| 103 | Frontend Typosquatting | Vulnerable Components | | Tìm typosquatting trong frontend dependencies. |
| 104 | Video XSS | XSS | | XSS qua video promo — stored payload. |
| 105 | XXE DoS | XXE | | XXE DoS — làm server xử lý lâu. |
| 106 | Wallet Depletion | Miscellaneous | | Rút nhiều ETH hơn đã nạp — Web3 challenge. |
| 107 | Memory Bomb | Insecure Deserialization | | YAML bomb — làm server hết memory. |
| 108 | Local File Read | Vulnerable Components | | Đọc file local trên server — LFI. |
| 109 | Successful RCE DoS | Insecure Deserialization | | RCE chiếm server một thời gian — không dùng infinite loop. |
| 110 | Blocked RCE DoS | Insecure Deserialization | | RCE giữ server bận mãi mãi. Nguy hiểm trên Docker. |
| 111 | Arbitrary File Write | Vulnerable Components | | Ghi đè file trên server. Nguy hiểm trên Docker. |
| 112 | SSTi | Injection | | Server-Side Template Injection — RCE. |

---

## Thống kê

| Độ khó | Số lượng | % |
|---|---|---|
| ⭐ (1) | 13 | 11.6% |
| ⭐⭐ (2) | 18 | 16.1% |
| ⭐⭐⭐ (3) | 24 | 21.4% |
| ⭐⭐⭐⭐ (4) | 27 | 24.1% |
| ⭐⭐⭐⭐⭐ (5) | 19 | 17.0% |
| ⭐⭐⭐⭐⭐⭐ (6) | 11 | 9.8% |
| **TỔNG** | **112** | **100%** |

---
