# Tài liệu làm việc & tích hợp hệ thống với AccessTrade Pub2

## 1. Mục tiêu

Tài liệu này mô tả phạm vi hợp tác và yêu cầu kỹ thuật giữa **Hệ thống của Bên A** (bên bạn) và **AccessTrade Pub2** (Bên B), nhằm:

* Đồng bộ định danh user giữa hai hệ thống
* Lấy link affiliate theo campaign và user
* Lấy báo cáo hiệu suất (click, đơn hàng, doanh thu/hoa hồng)
* Cấu hình và ánh xạ campaign giữa hai bên

Mục tiêu ưu tiên: **tích hợp nhanh, rõ ràng trách nhiệm, dễ mở rộng về sau**.

---

## 2. Phạm vi hợp tác

### 2.1. Phía AccessTrade Pub2 (Bên B)

Cung cấp API cho các chức năng sau:

1. **Định danh & ánh xạ user**

   * Cho phép ánh xạ user của Bên A ↔ user (publisher) của Pub2
   * Hỗ trợ truyền và lưu `external_user_id` (ID user của Bên A)

2. **Lấy link affiliate**

   * Lấy link affiliate theo:

     * Campaign của Pub2
     * User (publisher) đã được ánh xạ

3. **Báo cáo & thống kê**

   * Cung cấp báo cáo theo user và campaign, bao gồm tối thiểu:

     * Click
     * Đơn hàng (order)
     * Doanh thu / hoa hồng
     * Trạng thái đơn hàng
   * Hỗ trợ filter theo thời gian (from_date, to_date)

---

### 2.2. Phía Bên A (bên bạn)

1. **Cấu hình campaign**

   * Cho phép cấu hình `pub2_campaign_id` vào campaign nội bộ của Bên A
   * Mỗi campaign nội bộ có thể ánh xạ 1–n campaign Pub2 (nếu cần mở rộng sau)

2. **Quản lý user nội bộ**

   * Quản lý user (publisher) của Bên A
   * Lưu trữ mapping:

     * `internal_user_id`
     * `pub2_user_id`
     * `external_user_id` (nếu Pub2 dùng)

3. **Đồng bộ & hiển thị dữ liệu**

   * Gọi API Pub2 để:

     * Sinh link affiliate cho user
     * Đồng bộ báo cáo
   * Hiển thị dữ liệu cho user cuối (publisher) trên hệ thống Bên A

---

## 3. Định danh & ánh xạ user

### 3.1. Nguyên tắc

* Bên A là **nguồn gốc user** (source of truth)
* Pub2 lưu `external_user_id` do Bên A cung cấp để truy vết

### 3.2. Luồng đề xuất

1. Bên A gửi request tạo / ánh xạ user sang Pub2
2. Pub2 trả về `pub2_user_id`
3. Bên A lưu mapping:

   ```
   internal_user_id ↔ pub2_user_id
   ```

### 3.3. Trường dữ liệu đề xuất

* internal_user_id
* pub2_user_id
* external_user_id
* status
* created_at

---

## 4. Lấy link affiliate

### 4.1. Input yêu cầu

* pub2_campaign_id
* pub2_user_id (hoặc external_user_id)

### 4.2. Output mong đợi

* affiliate_link
* campaign_id
* user_id
* tracking_params (nếu có)

### 4.3. Nguyên tắc

* Link sinh ra phải gắn đúng user và campaign
* Có thể tái sử dụng (idempotent) nếu gọi nhiều lần

---

## 5. Báo cáo & thống kê

### 5.1. Loại báo cáo

| Nhóm       | Dữ liệu                       |
| ---------- | ----------------------------- |
| Traffic    | Click                         |
| Conversion | Đơn hàng                      |
| Revenue    | Doanh thu / hoa hồng          |
| Status     | Pending / Approved / Rejected |

### 5.2. Filter bắt buộc

* user_id / external_user_id
* campaign_id
* from_date
* to_date

### 5.3. Tần suất đồng bộ

* Giai đoạn đầu: on-demand (manual / cron ngày)
* Có thể mở rộng: webhook hoặc incremental sync

---

## 6. Bảo mật & xác thực

* API authentication: API Key / OAuth2 / HMAC (đề xuất Pub2 cung cấp chi tiết)
* IP whitelist (nếu cần)
* Rate limit rõ ràng

---

## 7. Trách nhiệm hai bên

### Bên A

* Quản lý user và campaign nội bộ
* Gọi API đúng chuẩn
* Hiển thị và giải thích dữ liệu cho user cuối

### AccessTrade Pub2

* Đảm bảo API ổn định, có versioning
* Dữ liệu báo cáo chính xác theo campaign & user
* Thông báo khi có thay đổi schema

---

## 8. Kế hoạch triển khai nhanh (1 tuần)

| Ngày    | Công việc                   |
| ------- | --------------------------- |
| Day 1   | Thống nhất API spec & auth  |
| Day 2   | Mapping user & campaign     |
| Day 3   | Tích hợp lấy link affiliate |
| Day 4   | Tích hợp báo cáo            |
| Day 5   | Test dữ liệu & edge cases   |
| Day 6–7 | Fix lỗi, go-live            |

---

## 9. Phần cần AccessTrade Pub2 xác nhận

* Danh sách API & endpoint
* Auth method
* Schema response chi tiết
* SLA & rate limit

---

*Phiên bản: v1.0 – Dùng cho trao đổi kỹ thuật & làm việc ban đầu*
