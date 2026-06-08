# Báo cáo kiểm thử bảo mật — WSTG-INFO
## Information Gathering (Thu thập thông tin)

## 1 Information Gathering (WSTG-INFO)

### Mô tả
Thu thập thông tin (Information Gathering) là bước đầu tiên trong kiểm thử bảo mật web. Attacker/penetration tester thu thập càng nhiều thông tin về mục tiêu thì các bước sau càng dễ dàng. Thông tin thu được bao gồm: phiên bản server/framework, cấu trúc ứng dụng, endpoint API, file backup, thông tin rò rỉ qua search engine...

### Mục tiêu
- Xác định phiên bản web server và framework
- Liệt kê tất cả endpoints và entry points của ứng dụng
- Phát hiện file backup, directory listing, thông tin rò rỉ
- Xây dựng bản đồ kiến trúc ứng dụng
- Thu thập thông tin có thể dùng cho các bước tấn công tiếp theo

### Công cụ sử dụng

| Công cụ | Phiên bản | Mục đích sử dụng |
|---|---|---|
| curl | — | Gửi request thủ công đến các endpoint |
| Firefox | — | Truy cập giao diện, xem HTML source |
| Burp Suite Community | — | Bắt request/response, phân tích traffic |
| OWASP ZAP | — | Quét spider, phát hiện endpoint |
| Nmap | — | Quét port, xác định services |
| WhatWeb | — | Fingerprint web technology |
| ffuf | — | Fuzzing đường dẫn, phát hiện file ẩn |

---

### Môi trường kiểm thử

| Thông số | Giá trị |
|---|---|
| Mục tiêu | OWASP Juice Shop |
| URL | `http://localhost:3000` |
| Container | `bkimminich/juice-shop` |
| Framework | Angular (frontend) + Express.js (backend) |
| Port | 3000 |

---

## 1.1 WSTG-INFO-01 — Search Engine Discovery

### Mục tiêu con
Tìm thông tin rò rỉ về mục tiêu trên các công cụ tìm kiếm (Google, Bing, Shodan, Wayback Machine).

### Phương pháp
- Tìm kiếm `site:localhost:3000` — không áp dụng cho lab local
- Shodan/Censys search cho IP và fingerprint
- Google Dorks: `intitle:"OWASP Juice Shop"`, `inurl:juice-shop`
- Wayback Machine để tìm endpoint cũ

### Kết quả kiểm thử

| Công cụ | Truy vấn | Kết quả | Đánh giá |
|---|---|---|---|
| Google | `site:localhost:3000` | Không áp dụng (lab local) | N/A |
| Shodan | `"OWASP Juice Shop"` | *(lab local, không public)* | N/A |
| Wayback | `juice-shop` | *(lab local)* | N/A |

**Đánh giá:** Môi trường lab local không public nên không thể áp dụng search engine discovery. Trong môi trường thực tế, đây là bước quan trọng để tìm thông tin rò rỉ như: email nhân viên, file backup bị index, API endpoint cũ, subdomain, và thông tin nhân viên từ LinkedIn.

---

## 1.2 WSTG-INFO-02 — Fingerprint Web Server

### Mục tiêu con
Xác định phiên bản web server, framework, và các công nghệ được sử dụng.

### Phương pháp
- Phân tích response headers (`Server`, `X-Powered-By`)
- Phân tích HTML source để tìm công nghệ
- Phân tích JavaScript files
- Sử dụng công cụ WhatWeb/Nmap để fingerprint

### Kết quả kiểm thử

#### Response Headers

| Header | Giá trị | Phân tích |
|---|---|---|
| `Server` | *(không có)* | Không tiết lộ server software — tốt |
| `X-Powered-By` | *(không có)* | Không tiết lộ Express — tốt |
| `X-Frame-Options` | `SAMEORIGIN` | Chống clickjacking cơ bản |
| `X-Content-Type-Options` | `nosniff` | Chống MIME sniffing |
| `Access-Control-Allow-Origin` | `*` | CORS mở — **vấn đề bảo mật** |
| `Feature-Policy` | `payment 'self'` | Giới hạn feature |
| `X-Recruiting` | `/#/jobs` | Header đặc trưng Juice Shop |
| `Content-Type` | `text/html; charset=UTF-8` | HTML response |

#### Error Page Fingerprint

Khi truy cập đường dẫn không tồn tại (`/api/`), server trả về error page chứa thông tin:

```
OWASP Juice Shop (Express ^4.22.1)
500 Error: Unexpected path: /api/
Stack trace:
  at /juice-shop/build/routes/angular.js:42:18
  at /juice-shop/build/lib/utils.js:225:26
  at Layer.handle [as handle_request] (express/lib/router/layer.js:95:5)
  at /juice-shop/node_modules/express/lib/router/index.js:328:13)
  at /juice-shop/build/routes/verify.js:210:5
  ...
```

**Thông tin tiết lộ qua error page:**

| Thông tin | Giá trị | Mức độ nghiêm trọng |
|---|---|---|
| Application name | `OWASP Juice Shop` | Thấp |
| Framework | `Express ^4.22.1` | Trung bình |
| Backend path | `/juice-shop/build/routes/` | Trung bình |
| File structure | `angular.js`, `verify.js`, `utils.js` | Trung bình |
| Node.js path | `/juice-shop/node_modules/express/` | Thấp |
| Express version | `4.22.1` | Trung bình |

#### HTML Source Analysis

| Chỉ số | Giá trị | Nguồn |
|---|---|---|
| Title | `OWASP Juice Shop` | `<title>` tag |
| Framework | Angular | `data-beasties-container`, `chunk-*.js`, `modulepreload` |
| Angular chunks | `KD3CNUZG`, `PX7UKXVL`, `Y3BEW76R`, `R7LX7FKI`, `7U6P4YGV`... | `<link rel="modulepreload">` |
| Polyfills | `polyfills.js` | Angular polyfills |
| Main bundle | `main.js` | Angular entry point |
| Styles | `styles.css` | Angular styles |
| Font | Google Fonts (VT323, Roboto) | External CDN |
| Description | "Probably the most modern and sophisticated insecure web application" | `<meta name="description">` |
| Copyright | `2014-2026 Bjoern Kimminich & the OWASP Juice Shop contributors` | HTML comment |

#### API Response Fingerprint

| Chỉ số | Giá trị |
|---|---|
| API format | JSON (`{"status":"success","data":[...]}`) |
| CORS | `Access-Control-Allow-Origin: *` |
| XSS protection | `X-Content-Type-Options: nosniff` |

### Kết quả tổng hợp

| Công nghệ | Phiên bản | Phương pháp xác định |
|---|---|---|
| Web Framework | Express.js | Error page, stack trace |
| Express version | 4.22.1 | Error page |
| Frontend Framework | Angular | HTML source, module chunks |
| Node.js path | `/juice-shop/` | Stack trace |
| Build tool | Angular CLI | Bundle structure (`chunk-*.js`) |

### Đánh giá

| Vấn đề | Mức độ | Mô tả |
|---|---|---|
| Stack trace trên error page | **Trung bình** | Error page hiển thị đường dẫn file, phiên bản framework |
| Express version tiết lộ | **Trung bình** | Biết version → tìm CVE tương ứng |
| Angular chunks tiết lộ | **Thấp** | Chunk names cho biết cấu trúc SPA |
| Backend path structure | **Trung bình** | `/juice-shop/build/routes/` tiết lộ cấu trúc thư mục |

---

## 1.3 WSTG-INFO-03 — Review Webserver Metafiles

### Mục tiêu con
Kiểm tra các file metadata do server tự động sinh ra (`robots.txt`, `sitemap.xml`, `.well-known/`, `security.txt`) để tìm thông tin hữu ích.

### Kết quả kiểm thử

#### robots.txt

```
User-agent: *
Disallow: /ftp
```

**Phân tích:**

| Nội dung | Ý nghĩa | Đánh giá |
|---|---|---|
| `Disallow: /ftp` | Chỉ cho phép spider index toàn bộ site **trừ** `/ftp` | `/ftp` chứa file quan trọng — server cố giấu |
| Không có `Allow` | Mặc định cho phép index mọi đường dẫn khác | Hầu hết site structure có thể bị index |

**Kết luận:** `robots.txt` vô tình **chỉ điểm** vào `/ftp/` — thư mục chứa file nhạy cảm. Đây là tín hiệu cho attacker biết `/ftp/` là điểm đáng chú ý.

#### .well-known/ directory

Thư mục `.well-known/` liệt kê được — **directory listing enabled**:

```
.well-known/
├── csaf/                    (directory)
└── security.txt             (file, 475 bytes)
```

**Đánh giá:** Directory listing trên `.well-known/` cho phép attacker thấy cấu trúc — `csaf/` là thư mục có thể chứa thông tin.

#### security.txt

```
Contact: mailto:donotreply@owasp-juice.shop
Encryption: https://keybase.io/bkimminich/pgp_keys.asc?fingerprint=19c01cb7157e4645e9e2c863062a85a8cbfbdcda
Acknowledgements: /#/score-board
Preferred-languages: en, ar, az, bg, bn, ca, cs, da, de, ga, el, es, et, fi, fr, ka, he, hi, hu, id, it, ja, ko, lv, my, nl, no, pl, pt, ro, ru, si, sv, th, tr, uk, zh
Hiring: /#/jobs
Csaf: http://localhost:3000/.well-known/csaf/provider-metadata.json
Expires: Mon, 07 Jun 2027 09:36:17 GMT
```

**Thông tin thu được:**

| Trường | Giá trị | Ý nghĩa tấn công |
|---|---|---|
| `Contact` | `donotreply@owasp-juice.shop` | Email domain → có thể dùng cho phishing |
| `Encryption` | Keybase PGP key fingerprint | Public key của maintainer |
| `Acknowledgements` | `/#/score-board` | Endpoint hiển thị contributors |
| `Hiring` | `/#/jobs` | Thông tin tuyển dụng — liên kết đặc trưng |
| `Csaf` | provider-metadata.json | CSAF security advisory feed |
| `Preferred-languages` | 24 ngôn ngữ | Thông tin về scope toàn cầu |
| `Expires` | 2027-06-07 | File còn hiệu lực 1 năm |

#### security.txt PGP Keys

Fingerprint: `19c01cb7157e4645e9e2c863062a85a8cbfbdcda`
Maintainer: Björn Kimminich (bkimminich)

#### .well-known/csaf/provider-metadata.json

```json
{
  "canonical_url": "http://localhost:3000/.well-known/csaf/provider-metadata.json",
  "distributions": [{
    "directory_url": "http://localhost:3000/.well-known/csaf/"
  }],
  "last_updated": "2024-03-05T20:20:56.169Z",
  "metadata_version": "2.0",
  "public_openpgp_keys": [
    {
      "fingerprint": "19c01cb7157e4645e9e2c863062a85a8cbfbdcda",
      "url": "https://keybase.io/bkimminich/pgp_keys.asc"
    },
    {
      "fingerprint": "2372B2B12AEA7AE3001BB3FBD08FB16E2029D870",
      "url": "https://keybase.io/wurstbrot/pgp_keys.asc"
    }
  ]
}
```

**Thông tin thu được:**

| Trường | Giá trị | Ý nghĩa |
|---|---|---|
| 2 PGP keys | Björn Kimminich + wurstbrot | 2 maintainer chính |
| Last updated | 2024-03-05 | CSAF feed cũ ~14 tháng |
| CSAF directory | `http://localhost:3000/.well-known/csaf/` | Có thể chứa security advisories |

#### sitemap.xml

Khi truy cập `/sitemap.xml`, server trả về **toàn bộ HTML trang chủ** (status 200, content-type `text/html`) — thay vì XML sitemap.

**Đánh giá:** Server không có sitemap.xml thực sự — nó phục vụ trang chủ cho mọi đường dẫn không xác định (do Angular SPA catch-all route). Điều này không tiết lộ thông tin thêm nhưng cho thấy kiến trúc SPA.

### Kết quả tổng hợp

| File | Trạng thái | Thông tin thu được | Mức độ |
|---|---|---|---|
| `robots.txt` | ✅ Tồn tại | `/ftp` bị Disallow → chỉ điểm thư mục quan trọng | Trung bình |
| `sitemap.xml` | ⚠️ Trả về HTML | Không có sitemap thực | Thấp |
| `.well-known/` | ✅ Directory listing | Liệt kê được `csaf/` và `security.txt` | Thấp |
| `.well-known/security.txt` | ✅ Tồn tại | Contact email, PGP keys, hiring link, languages | Thấp |
| `.well-known/csaf/` | ✅ Directory listing | PGP keys của 2 maintainers | Thấp |

---

## 1.4 WSTG-INFO-04 — Enumerate Applications on Webserver

### Mục tiêu con
Liệt kê tất cả các ứng dụng/dịch vụ chạy trên cùng một server.

### Phương pháp
- Quét port với Nmap để xác định services
- Kiểm tra các endpoint đặc trưng
- Xác định có service khác chạy trên cùng host không

### Kết quả kiểm thử

#### Port Scanning

Port 3000 (HTTP) đang mở — đây là port mặc định của Juice Shop.

**Quan sát:** Không thực hiện quét port toàn diện trên lab local. Trong môi trường thực tế, cần quét toàn bộ port (Nmap) để phát hiện các service khác có thể chứa thông tin rò rỉ.

#### Application Entry Points

Dựa trên phân tích, các entry points chính của ứng dụng:

| Type | Endpoint | Phương thức | Mô tả |
|---|---|---|---|
| Web UI | `/#/` | GET | Angular SPA entry |
| API | `/api/` | — | REST API prefix |
| REST API | `/rest/` | — | Alternative REST prefix |
| Register | `/api/users/` | POST | Đăng ký tài khoản |
| Login | `/rest/user/login` | POST | Đăng nhập |
| Products | `/rest/products/` | GET | Danh sách sản phẩm |
| Search | `/rest/products/search?q=` | GET | Tìm kiếm sản phẩm |
| Admin | `/administration/` | — | Trang quản trị |
| Files | `/ftp/` | GET | Directory chứa file |
| Jobs | `/#/jobs` | GET | Trang tuyển dụng |
| Scoreboard | `/#/score-board` | GET | Bảng điểm contributors |

### Kết quả tổng hợp

| Ứng dụng/Dịch vụ | Port | Trạng thái | Ghi chú |
|---|---|---|---|
| Juice Shop (Express) | 3000 | ✅ Chạy | Angular SPA + Express API |
| *(Khác)* | — | — | Không phát hiện |

---

## 1.5 WSTG-INFO-05 — Review Webpage Content for Information Leakage

### Mục tiêu con
Kiểm tra HTML, JavaScript, CSS, và comments có lộ thông tin nhạy cảm không.

### Phương pháp
- Xem HTML source của tất cả trang
- Tìm HTML comments chứa thông tin
- Phân tích JavaScript bundle
- Tìm debug info, internal paths, credentials

### Kết quả kiểm thử

#### HTML Comments

```html
<!-- ~ Copyright (c) 2014-2026 Bjoern Kimminich & the OWASP Juice Shop contributors. ~
     SPDX-License-Identifier: MIT -->
```

**Phân tích:**

| Thông tin | Giá trị | Đánh giá |
|---|---|---|
| Copyright range | 2014-2026 (12 năm) | Thấp — cho biết tuổi dự án |
| Maintainer | Björn Kimminich | Thấp — tên công khai |
| License | MIT | Thấp — license công khai |
| Team | "OWASP Juice Shop contributors" | Thấp |

#### JavaScript Files

Các file JavaScript được phát hiện trong HTML:

| File | Loại | Ý nghĩa |
|---|---|---|
| `polyfills.js` | Angular polyfills | Cho biết Angular version range |
| `scripts.js` | App scripts | Custom scripts |
| `main.js` | Angular entry | Main bundle (module type) |
| `chunk-KD3CNUZG.js` | Lazy chunk | Feature module |
| `chunk-PX7UKXVL.js` | Lazy chunk | Feature module |
| `chunk-Y3BEW76R.js` | Lazy chunk | Feature module |
| `chunk-R7LX7FKI.js` | Lazy chunk | Feature module |
| `chunk-7U6P4YGV.js` | Lazy chunk | Feature module |
| `chunk-UNFVUBM2.js` | Lazy chunk | Feature module |
| `chunk-KI34UYIE.js` | Lazy chunk | Feature module |
| `chunk-B3RFAYKZ.js` | Lazy chunk | Feature module |
| `chunk-EMJTEAHL.js` | Lazy chunk | Feature module |
| `chunk-6RWMKLEX.js` | Lazy chunk | Feature module |

**Quan sát:** 13 lazy-loaded chunks cho thấy Angular app có nhiều feature modules. Mỗi chunk có thể chứa thông tin về routing, API endpoints, và business logic.

#### External Resources

| Resource | URL | Ý nghĩa |
|---|---|---|
| Fonts | `fonts.googleapis.com`, `fonts.gstatic.com` | Google Fonts CDN — tiết lộ Google services |
| Favicon | `assets/public/favicon_js.ico` | Juice Shop favicon |

#### Cookie Consent Message

```javascript
cookieconsent.initialise({
  "message": "This website uses fruit cookies to ensure you get the juiciest tracking experience.",
  "dismiss": "Me want it!",
  "link": "But me wait!",
  "href": "https://www.youtube.com/watch?v=9PnbKL3wuH4"
})
```

**Thông tin:** Link đến video YouTube hài hước — không rò rỉ thông tin nhạy cảm.

### Kết quả tổng hợp

| Loại thông tin | Tiết lộ | Mức độ |
|---|---|---|
| HTML comments | Copyright, maintainer, license | Thấp |
| JS bundle structure | Angular version, feature modules | Thấp |
| External CDN | Google Fonts | Thấp |
| Stack trace (error) | Express path, file structure, version | Trung bình |
| **Tổng thể** | **Không có thông tin nhạy cảm nghiêm trọng qua HTML** | **Thấp** |

---

## 1.6 WSTG-INFO-06 — Identify Application Entry Points

### Mục tiêu con
Xác định tất cả các điểm vào (entry points) của ứng dụng: form fields, URL parameters, API endpoints, WebSocket, file upload.

### Phương pháp
- Phân tích HTML form elements
- Phân tích JavaScript routing
- Fuzz API paths
- Kiểm tra WebSocket endpoints

### Kết quả kiểm thử

#### API Endpoints đã xác định

| Method | Endpoint | Mô tả |
|---|---|---|
| GET | `/rest/products/` | Liệt kê sản phẩm |
| GET | `/rest/products/search?q=` | Tìm kiếm sản phẩm |
| GET | `/rest/products/{id}` | Chi tiết sản phẩm |
| POST | `/api/users/` | Đăng ký tài khoản |
| POST | `/rest/user/login` | Đăng nhập |
| GET | `/api/users/` | Liệt kê người dùng (cần auth) |
| GET/PUT/DELETE | `/rest/user/` | Thông tin user hiện tại |
| GET | `/rest/basket/` | Giỏ hàng |
| POST | `/rest/basket/` | Thêm vào giỏ |
| GET | `/rest/orders/` | Lịch sử đơn hàng |
| POST | `/rest/orders/` | Tạo đơn hàng |
| GET | `/rest/comparison/` | So sánh sản phẩm |
| GET | `/rest/address-suggestions` | Gợi ý địa chỉ |
| GET | `/rest/walkthrough/` | Hướng dẫn sử dụng |
| GET | `/rest/support/` | Hỗ trợ |
| GET | `/rest/feedbacks` | Phản hồi |
| GET | `/rest/challenges` | CTF challenges |
| GET | `/rest/security-question` | Câu hỏi bảo mật |
| POST | `/rest/remember-me` | Remember me token |
| OPTIONS | `/api/users/` | CORS preflight |

#### Form Fields (từ giao diện)

| Form | Fields | Endpoint |
|---|---|---|
| Đăng ký | username, email, password, role, security question, security answer | POST `/api/users/` |
| Đăng nhập | email, password, rememberMe | POST `/rest/user/login` |
| Forgot Password | email | POST (endpoint cần xác định) |
| Search | q (query string) | GET `/rest/products/search?q=` |
| Review | author, text, productId | POST (endpoint cần xác định) |

#### URL Parameters

| Parameter | Endpoint | Mô tả |
|---|---|---|
| `q` | `/rest/products/search` | Từ khóa tìm kiếm |
| `id` | `/rest/products/{id}` | ID sản phẩm |
| `page` | `/rest/products/` | Phân trang |
| `count` | `/rest/products/` | Số lượng mỗi trang |
| `sortBy` | `/rest/products/` | Sắp xếp |
| `sortDesc` | `/rest/products/` | Chiều sắp xếp |

### Kết quả tổng hợp

| Loại entry point | Số lượng | Đánh giá |
|---|---|---|
| REST API endpoints | 18+ | Bề mặt tấn công lớn |
| Form fields | 6+ | Input validation vectors |
| URL parameters | 5+ | IDOR, injection vectors |
| HTTP Methods | 6 (GET, POST, PUT, PATCH, DELETE, HEAD) | Trên `/api/users/` |
| **Tổng entry points** | **30+** | **Ứng dụng có bề mặt tấn công rộng** |

---

## 1.7 WSTG-INFO-07 — Map Execution Paths Through Application

### Mục tiêu con
Xác định luồng điều hướng giữa các trang và cách người dùng di chuyển qua ứng dụng.

### Phương pháp
- Phân tích Angular routing
- Truy cập tuần tự các trang để map luồng
- Xác định trang nào cần auth, trang nào public

### Kết quả kiểm thử

#### Application Routes (Angular SPA)

Dựa trên phân tích HTML source và JavaScript chunks:

| Route | Trạng thái | Mô tả |
|---|---|---|
| `/#/` | Public | Trang chủ |
| `/#/search` | Public | Tìm kiếm sản phẩm |
| `/#/search/` | Public | Chi tiết sản phẩm |
| `/#/basket` | Auth | Giỏ hàng |
| `/#/login` | Public | Đăng nhập |
| `/#/register` | Public | Đăng ký |
| `/#/forgot-password` | Public | Quên mật khẩu |
| `/#/my-account` | Auth | Thông tin tài khoản |
| `/#/my-orders` | Auth | Lịch sử đơn hàng |
| `/#/my-addresses` | Auth | Địa chỉ |
| `/#/my-wishlist` | Auth | Danh sách yêu thích |
| `/#/order-history` | Auth | Lịch sử đơn |
| `/#/track-order` | Auth | Theo dõi đơn |
| `/#/track-result` | Auth | Kết quả theo dõi |
| `/#/payment` | Auth | Thanh toán |
| `/#/checkout` | Auth | Thanh toán |
| `/#/administration` | Admin only | Trang quản trị |
| `/#/complain` | Auth | Khiếu nại |
| `/#/contact` | Public | Liên hệ |
| `/#/deluxe-membership` | Auth | Thành viên Deluxe |
| `/#/recycle` | Auth | Tái chế |
| `/#/score-board` | Public | Bảng điểm |
| `/#/about` | Public | Giới thiệu |
| `/#/imprint` | Public | Thông tin pháp lý |
| `/#/jobs` | Public | Tuyển dụng |
| `/#/sitemap` | Public | Sơ đồ site |

#### Execution Flow

```
Public Routes:
  / → Homepage (products, search, banners)
  /#/search → Product listing + search
  /#/login → Login form
  /#/register → Registration form
  /#/forgot-password → Password reset
  /#/contact → Contact form
  /#/about → About page
  /#/imprint → Legal info
  /#/jobs → Hiring info
  /#/score-board → Challenge scoreboard

Auth Required:
  /#/basket → Shopping cart
  /#/my-account → User profile
  /#/my-orders → Order history
  /#/checkout → Checkout flow
  /#/payment → Payment
  /#/complain → Complaints

Admin Only:
  /#/administration → Admin panel
```

### Kết quả tổng hợp

| Route type | Số lượng | Đánh giá |
|---|---|---|
| Public routes | ~9 | Không cần authentication |
| Auth required | ~8 | Cần đăng nhập |
| Admin only | 1 | Cần role admin |
| **Tổng routes** | **~18** | **Phân chia rõ ràng public/auth/admin** |

---

## 1.8 WSTG-INFO-08 — Fingerprint Web Application Framework

### Mục tiêu con
Xác định frontend framework, backend framework, thư viện, và các component được sử dụng.

### Kết quả kiểm thử

| Layer | Công nghệ | Phiên bản | Phương pháp xác định |
|---|---|---|---|
| **Frontend Framework** | Angular | *(xác định qua bundle)* | `chunk-*.js`, `modulepreload`, `polyfills.js` |
| **CSS Framework** | Angular Material | *(xác định qua CSS variables)* | `--mat-sys-*` CSS custom properties |
| **UI Theme** | Angular Material Theme | Bluegrey + Lightgreen | CSS theme classes |
| **Backend Framework** | Express.js | 4.22.1 | Error page stack trace |
| **Runtime** | Node.js | *(không tiết lộ)* | Stack trace path |
| **HTTP Server** | Express | — | Stack trace `express/lib/router/` |
| **Logger** | Morgan | *(xác định qua stack)* | Stack trace `node_modules/morgan/` |
| **Build System** | Angular CLI | — | Bundle structure |
| **SPA Router** | Angular Router | — | Hash-based routing (`/#/`) |
| **Cookie Library** | cookieconsent | — | External cookie consent library |

### Angular Version Analysis

Dựa trên các chỉ số:

| Chỉ số | Giá trị |
|---|---|
| polyfills.js | Angular polyfills (legacy support) |
| modulepreload | ES modules lazy loading |
| chunk naming | Hash-based chunk names |
| CSS variables | `--mat-sys-*` (Angular Material v17+) |

### Kết quả tổng hợp

| Framework | Version | CVE Risk | Đánh giá |
|---|---|---|---|
| Express.js | 4.22.1 | Cần check CVE | Biết version → dễ tìm exploit |
| Angular | *(unknown)* | Cần check | Bundle không hiển thị version rõ |
| Morgan | *(unknown)* | Thấp | Logger middleware |

---

## 1.9 WSTG-INFO-09 — Fingerprint Web Application

### Mục tiêu con
Xác định phiên bản cụ thể của ứng dụng Juice Shop.

### Kết quả kiểm thử

| Thông tin | Giá trị | Nguồn |
|---|---|---|
| Tên ứng dụng | OWASP Juice Shop | `<title>` tag |
| Version | *(không hiển thị rõ)* | Không có version trong HTML/API |
| Author | Björn Kimminich | Copyright, security.txt |
| Organization | OWASP | Tên, security.txt |
| Repository | github.com/juice-shop/juice-shop | Product description |
| GitHub Archive | v9.3.1-PERMAFROST (2020) | Product description |
| License | MIT | HTML comment |
| Description | "Probably the most modern and sophisticated insecure web application" | Meta description |
| Scoreboard | `/#/score-board` | security.txt |

### Đặc trưng Juice Shop đặc biệt

| Header | Value | Đặc trưng |
|---|---|---|
| `X-Recruiting` | `/#/jobs` | Chỉ có ở Juice Shop |

### Kết quả: Không xác định được phiên bản cụ thể qua fingerprint. Cần kiểm tra GitHub release hoặc response API chi tiết.

---

## 1.10 WSTG-INFO-10 — Map Application Architecture

### Mục tiêu con
Xây dựng bản đồ kiến trúc tổng thể của ứng dụng: frontend, backend, API, database, external services.

### Kết quả kiểm thử

#### Kiến trúc tổng quan

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT (Browser)                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Angular SPA (Frontend)                       │   │
│  │  ┌──────────┬──────────┬──────────┬──────────────┐  │   │
│  │  │ Product  │  Auth    │  Basket  │  Admin Panel │  │   │
│  │  │ Catalog  │  Module  │  Module  │   Module     │  │   │
│  │  └──────────┴──────────┴──────────┴──────────────┘  │   │
│  └──────────────────────────────────────────────────────┘   │
│                           │                                  │
│                    HTTP/REST API                              │
└───────────────────────────┼──────────────────────────────────┘
                            │
┌───────────────────────────┼──────────────────────────────────┐
│                    SERVER (Express.js 4.22.1)                │
│  ┌────────────────────────┼──────────────────────────┐       │
│  │              REST API Layer                       │       │
│  │  ┌──────────┬──────────┬──────────┬──────────┐  │       │
│  │  │ /rest/   │ /api/    │ /api/    │ /ftp/    │  │       │
│  │  │ products │ users    │ users    │ files    │  │       │
│  │  │ orders   │ basket   │ (CRUD)   │ (static) │  │       │
│  │  └──────────┴──────────┴──────────┴──────────┘  │       │
│  └─────────────────────────────────────────────────┘       │
│                           │                                  │
│  ┌────────────────────────┼──────────────────────────┐       │
│  │              Middleware Stack                     │       │
│  │  CORS → Logger → Body Parser → Auth → Routes     │       │
│  └─────────────────────────────────────────────────┘       │
│                           │                                  │
│  ┌────────────────────────┼──────────────────────────┐       │
│  │              Database Layer                       │       │
│  │         SQLite (default) / Others                 │       │
│  └─────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────┘
                            │
                    External Services
                    ┌──────┴──────┐
                    │ Google Fonts │
                    │  YouTube     │
                    │  GitHub      │
                    │  Keybase     │
                    └─────────────┘
```

#### CORS Analysis

| Header | Value | Đánh giá |
|---|---|---|
| `Access-Control-Allow-Origin` | `*` | Cho phép mọi origin — **rất rộng** |
| `Access-Control-Allow-Methods` | `GET, HEAD, PUT, PATCH, POST, DELETE` | Cho phép tất cả methods |
| `Vary` | `Accept-Encoding` | Standard |

**Rủi ro:** CORS `*` cho phép bất kỳ website nào đọc dữ liệu từ API. Nếu user đang đăng nhập Juice Shop và truy cập malicious site → site đó có thể đọc toàn bộ API response (CSRF không cần token).

#### Directory Listing

| Path | Kết quả | Đánh giá |
|---|---|---|
| `/ftp/` | ✅ Directory listing enabled | **Vấn đề** — liệt kê file nhạy cảm |
| `.well-known/` | ✅ Directory listing enabled | Thấp |

**File trong `/ftp/`:**

| File | Loại | Mô tả |
|---|---|---|
| `acquisitions.md` | Text | Ghi chú về acquisitions |
| `announcement_encrypted.md` | Encrypted | Thông báo đã mã hóa |
| `coupons_2013.md.bak` | Backup | File backup coupons — **2013** |
| `eastere.gg` | — | Easter egg |
| `encrypt.pyc` | Python bytecode | Script mã hóa (đã compile) |
| `incident-support.kdbx` | KeePass | Database mật khẩu — **quan trọng** |
| `legal.md` | Text | Thông tin pháp lý |
| `package-lock.json.bak` | Backup | Dependency lock file backup |
| `package.json.bak` | Backup | Package config backup |
| `suspicious_errors.yml` | YAML | Log lỗi nghi vấn |
| `quarantine/` | Directory | Thư mục cách ly |

**Đánh giá:** `/ftp/` directory listing là lỗ hổng Information Disclosure nghiêm trọng — attacker thấy ngay danh sách file backup và KeePass database.

### Kết quả tổng hợp

| Thành phần | Công nghệ | Ghi chú |
|---|---|---|
| Frontend | Angular SPA | Hash routing, lazy loading |
| Backend | Express.js 4.22.1 | REST API |
| Database | SQLite (mặc định) | Không xác định chắc chắn |
| Logger | Morgan | Request logging |
| Static Files | Serve từ `/ftp/` | Directory listing enabled |
| CORS | `Access-Control-Allow-Origin: *` | Rất rộng |
| External | Google Fonts, YouTube, GitHub, Keybase | CDN + services |

---

## Tổng kết WSTG-INFO

### Bảng kết quả tất cả các hạng mục

| ID | Tên kiểm thử | Kết quả chính | Mức độ |
|---|---|---|---|
| WSTG-INFO-01 | Search Engine Discovery | Không áp dụng (lab local) | N/A |
| WSTG-INFO-02 | Fingerprint Web Server | Express 4.22.1, Angular, stack trace lộ path | Trung bình |
| WSTG-INFO-03 | Review Metafiles | robots.txt chỉ điểm /ftp, .well-known listing, security.txt | Trung bình |
| WSTG-INFO-04 | Enumerate Applications | 18+ API endpoints, Angular routes | Trung bình |
| WSTG-INFO-05 | Review Content | Không có leakage nghiêm trọng qua HTML | Thấp |
| WSTG-INFO-06 | Identify Entry Points | 30+ entry points (API + forms + params) | Trung bình |
| WSTG-INFO-07 | Map Execution Paths | 18 routes, 3 loại (public/auth/admin) | Thấp |
| WSTG-INFO-08 | Fingerprint Framework | Express 4.22.1, Angular + Angular Material | Trung bình |
| WSTG-INFO-09 | Fingerprint Application | Juice Shop, không rõ version cụ thể | Thấp |
| WSTG-INFO-10 | Map Architecture | CORS *, /ftp/ listing, SQLite, 11 external services | **Cao** |

### Các lỗ hổng chính tìm được

| # | Lỗ hổng | Nguồn | Mức độ |
|---|---|---|---|
| 1 | **Stack trace trên error page** | `/api/` trả về full stack trace | Trung bình |
| 2 | **Express version tiết lộ** | Error page: `Express ^4.22.1` | Trung bình |
| 3 | **Backend path structure** | Stack trace: `/juice-shop/build/routes/` | Trung bình |
| 4 | **robots.txt chỉ điểm /ftp** | `Disallow: /ftp` → attacker biết có gì đó | Trung bình |
| 5 | **Directory listing trên /ftp/** | Liệt kê 11 file nhạy cảm | **Cao** |
| 6 | **Directory listing trên .well-known/** | Liệt kê `csaf/` và `security.txt` | Thấp |
| 7 | **CORS cho phép tất cả origins** | `Access-Control-Allow-Origin: *` | Trung bình |
| 8 | **File backup trong /ftp/** | `*.bak` files tiết lộ config cũ | Trung bình |
| 9 | **KeePass database trong /ftp/** | `incident-support.kdbx` | **Cao** |
| 10 | **6 HTTP methods trên endpoint** | `/api/users/` chấp nhận GET/POST/PUT/PATCH/DELETE | Thấp |

### Đề xuất khắc phục

| # | Vấn đề | Đề xuất |
|---|---|---|
| 1 | Stack trace tiết lộ | Custom error page, tắt stack trace ở production |
| 2 | Express version | Ẩn header `X-Powered-By`, không hiển thị version |
| 3 | Directory listing | Tắt directory listing trên toàn bộ server |
| 4 | /ftp/ access | Chuyển file nhạy cảm ra khỏi web root, hoặc bảo vệ bằng auth |
| 5 | CORS `*` | Giới hạn origin cụ thể |
| 6 | robots.txt | Không nên Disallow thư mục quan trọng — chỉ làm tăng sự tò mò |
| 7 | File backup | Xóa `.bak` files, không deploy lên production |
| 8 | KeePass database | Xóa khỏi web root, lưu ở nơi an toàn |

---

### Liên kết với các hạng mục khác

```
WSTG-INFO (Information Gathering) [Đang kiểm thử]
    │
    ├──→ WSTG-CONF (Configuration) — Directory listing, CORS, file permissions
    │       Tìm được: /ftp/ listing, CORS *, file backup
    │
    ├──→ WSTG-IDNT (Identity) — Endpoint /api/users/, registration
    │       Tìm được: /api/users/ chấp nhận 6 methods, role manipulation
    │
    ├──→ WSTG-INPVAL (Input Validation) — 30+ entry points để test
    │       Tìm được: /rest/products/search?q=, form fields, API endpoints
    │
    └──→ WSTG-CLIENT (Client-Side) — Angular SPA, CORS
            Tìm được: Angular routes, chunk structure, CORS *
```
