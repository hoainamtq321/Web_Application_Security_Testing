# Báo cáo kiểm thử xác nhận đầu vào — WSTG-INPVAL
---

## Tổng quan

| Chỉ số | Giá trị |
|--------|---------|
|  |  |
|  |  |
|  |  |
|  |  |

---

## 7. Input Validation Testing (WSTG-INPVAL)

### Mô tả
Kiểm tra xác thực đầu vào là quá trình đánh giá cách ứng dụng xử lý dữ liệu từ phía người dùng trước khi đưa vào xử lý logic nghiệp vụ, lưu trữ trong cơ sở dữ liệu hoặc hiển thị lại trên giao diện.

### Mục tiêu


### Phương pháp
### Công cụ sử dụng

| Công cụ | Mục đích sử dụng |
|---------|-----------------|
| Burpsuite |  |

### Môi trường kiểm thử

| Thông số | Giá trị |
|----------|---------|
| Mục tiêu | OWASP Juice Shop |
| URL | `http://localhost:3000` |
| Framework | Angular + Express.js 4.22.1 |


---

## 7.1 WSTG-INPV-01 — Testing for Cross Site Scripting

### Mô tả


### Mục tiêu
- Xác định các biến số được phản ánh trong câu trả lời.
- Đánh giá dữ liệu đầu vào mà họ chấp nhận và mã hóa được áp dụng khi trả về (nếu có).

### Kết quả

| Mục | Nội dung |
|---|---|
| **Vị trí** | Chức năng Tìm kiếm ( localhost:3000/#/search?q= ). |
| **Mục tiêu** | Xác định việc thiếu cơ chế lọc/mã hóa dữ liệu đầu vào (Input Validation/Output Encoding), dẫn đến nguy cơ thực thi mã độc trên trình duyệt người dùng. |
| **Công cụ** | browser |
| **Quy trình** | 1. Nhập payload XSS (ví dụ: <svg/onload=alert(1)>) vào các trường input. <br> 2. Quan sát DOM để kiểm tra xem payload có bị chèn trực tiếp hay không. <br> 3. Kiểm tra khả năng thực thi của các payload khác nhau (script, img, svg). 
| **Kết quả** | **LỖ HỔNG — ĐÃ XÁC NHẬN** |
| **Tổng kết** |  |

### Quy trình thực hiện

#### Chức năng tìm kiếm sản phẩm (/search )
**Payload:  `<img src=x onerror=alert(1)>`**
<br>
**Kết quả**
<br>
```
<div _ngcontent-ng-c1161564479>
  <span _ngcontent-ng-c1161564479>Search Results</span>
  <span_ngcontent-ng-c1161564479 id="searchValue"> == $0
    <img src="x" onerror="alert(1)">
  </span>
</div>
```
<br>
<img width="1026" height="281" alt="image" src="https://github.com/user-attachments/assets/b2703d36-b280-49c1-a90c-7f79ce389f2c" />
<br>

**Kiểm tra khả năng thực thi của các payload khác**
**Payload:  `<iframe width="100%" height="166" src="https://zingmp3.vn/bai-hat/Dao-Buoc-HongKong-1999-MCK/Z8E9E6U6.html"></iframe>`**
<br>
<img width="1452" height="531" alt="image" src="https://github.com/user-attachments/assets/49727f75-3057-489b-8dc5-e53d66d901dc" />
<br>
**Đánh giá: `Hệ thống xác nhận tồn tại lỗ hổng DOM-based XSS tại chức năng tìm kiếm sản phẩm (/search). Ứng dụng không thực hiện kiểm tra hoặc mã hóa (sanitization) đối với dữ liệu đầu vào từ tham số q trước khi thực hiện thao tác chèn trực tiếp vào cấu trúc DOM (thông qua innerHTML hoặc các phương thức tương tự của framework Angular).`**
