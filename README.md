# 1. Phân trang Limit/Offset

# Cách hoạt động:

- Sử dụng hai tham số: `LIMIT` (số lượng bản ghi) và `OFFSET` (bỏ qua bao nhiêu bản ghi)

- Ví dụ câu lệnh query: <pre> ``` SELECT * FROM users LIMIT 10 OFFSET 20 ``` </pre>

# Ưu điểm:

- Linh hoạt: Có thể nhảy đến bất kỳ trang nào. 

# Nhược điểm:

- Hiệu năng kém với OFFSET lớn: Database phải đếm và bỏ qua nhiều bản ghi

- Khi có dữ liệu mới được thêm/xóa, có thể bị trùng lặp hoặc thiếu dữ liệu (khi xoá thì bản ghi đầu tiên của trang sau sẽ bị đẩy lên trang trước làm khi chuyển sang trang sau bị thiếu dữ liệu)

<pre> ```| ID | Name | Created_at |
|----|------|------------|
| 1  | A    | 2024-01-01 |
| 2  | B    | 2024-01-02 |
| 3  | C    | 2024-01-03 |
| 4  | D    | 2024-01-04 |
| 5  | E    | 2024-01-05 |
| 6  | F    | 2024-01-06 |
| 7  | G    | 2024-01-07 |
| 8  | H    | 2024-01-08 |
 ``` </pre>

Khi user xem trang 1 với `LIMIT = 3 OFFSET = 0`, cùng lúc đó `id = 1` bị xoá. Khi đó sang trang 2 sẽ bắt đầu với `id = 5` làm `id = 4` biến mất.

- Tốn tài nguyên: OFFSET càng lớn càng chậm

-> Dataset nhỏ hoặc vừa (< 10,000 records)

-> Ví dụ trong Google Search dùng limit/offset vì người dùng cần nhảy đến trang cụ thể

# 2. Phân trang dựa trên con trỏ (Cursor-based)

# Cách hoạt động:

- Sử dụng một giá trị duy nhất (cursor) để xác định vị trí hiện tại

- Thường dùng ID, timestamp, hoặc composite key

- Ví dụ query: 

    + ID: <pre> ``` SELECT * FROM users WHERE id > 1000 LIMIT 10 ``` </pre>

    + Timestamp: <pre> ``` SELECT * FROM posts WHERE created_at < '2025-09-09 17:00:00' ``` </pre>

    + Composite key (khi trùng timestamp): <pre> ``` SELECT * FROM posts WHERE created_at < '2025-09-09 12:00:00' OR (created_at = '2025-09-09 12:00:00' AND id < 42) ``` </pre>

(Kết hợp `ORDER BY` để lọc được chính xác)

## Ưu điểm:

- Hiệu năng cao: Luôn nhanh bất kể vị trí và độ lớn dữ liệu vì Cursor trỏ đến một giá trị cụ thể, không phải vị trí

- Không bị ảnh hưởng bởi việc thêm/xóa dữ liệu (điều kiện WHERE cố định: `WHERE id > cursor_value` không thay đổi dù có thêm/xóa dữ liệu)

- Phù hợp với real-time data, lý tưởng cho feed, timeline 

## Nhược điểm:

- Phức tạp hơn so với Limit/Offset: Không thể nhảy trang, chỉ có thể next/previous và phụ thuộc vào thứ tự: cần có trường có thể sắp xếp duy nhất

# FourSeason Restaurant System - Slide Deck

## Slide 1: Tiêu đề
- **Đồ án tốt nghiệp:** Hệ thống quản lý nhà hàng tích hợp gợi ý món ăn thông minh
- **Sinh viên:** Lê Thế Quang
- **Mã sinh viên:** 202272023
- **Chuyên ngành:** Hệ thống Thông tin Quản lý
- **Giảng viên hướng dẫn:** TS. Lê Quang Hoà
- **Năm:** 2026

---

## Slide 2: Thực trạng và vấn đề
- Nhà hàng Việt Nam nhiều nơi vẫn quản lý thủ công bằng giấy hoặc Excel
- Lỗi đặt món, mất đơn, thanh toán chậm, ghi thông tin sai và khó kiểm soát
- Thiếu dữ liệu doanh thu theo ngày, món, nhân viên để ra quyết định chính xác
- Khách hàng quay lại không được cá nhân hoá gợi ý món phù hợp
- Giải pháp hiện có:
  - POS truyền thống: tốn kém, cứng nhắc
  - SaaS nước ngoài: không phù hợp văn hoá VN, phụ thuộc internet
  - Open-source: cần phát triển nhiều module

---

## Slide 3: So sánh giải pháp
| Tiêu chí | POS truyền thống | SaaS nhập khẩu | Open-source | **FourSeason** |
|----------|------------------|----------------|-------------|---------------|
| Chi phí | Cao (50-100tr) | Thuê bao | Miễn phí | **Miễn phí tự host** |
| Tiếng Việt | Có | Chuyển dịch kém | Cần dịch | **Native VN** |
| ML gợi ý | Không | Có nhưng tách biệt | Không | **Tích hợp sẵn** |
| Module bếp realtime | Cơ bản | Có | Cần dev | **HTMX polling realtime** |
| Tuỳ biến | Khó | Hạn chế | Linh hoạt | **Full control (Django)** |

---

## Slide 4: Khoảng trống & Định hướng
- Cần giải pháp:
  - Chi phí thấp, tự host
  - Giao diện tiếng Việt native
  - ML gợi ý tích hợp sẵn
  - Module bếp realtime
  - Phân quyền rõ ràng cho 3 role
- Hướng xây dựng:
  - Full-stack Django, MTV pattern
  - Deploy Docker
  - Có test và CI/CD

---

## Slide 5: Mục tiêu & Phạm vi
- **Mục tiêu chính:** xây dựng hệ thống web quản lý nhà hàng end-to-end, từ đặt bàn → order → bếp → thanh toán → báo cáo, tích hợp ML gợi ý món
- **Đối tượng sử dụng:**
  - Khách hàng
  - Nhân viên bếp/phục vụ
  - Quản lý/Chủ nhà hàng
- **Phạm vi:**
  - ✅ Web app responsive (Bootstrap 5)
  - ✅ Đặt bàn realtime, giỏ hàng, thanh toán VNPAY/COD
  - ✅ Màn hình bếp HTMX polling (< 2s latency)
  - ✅ Dashboard Chart.js
  - ✅ ML Hybrid Recommender (CF + Content-based)
  - ✅ Phân quyền 3 role, soft delete, audit log
  - ❌ Mobile app native
  - ❌ Multi-branch (chuỗi) — v2

---

## Slide 6: Screenshot trang chủ
- Ảnh chụp màn hình homepage
- Caption: "Trang chủ: Hero CTA đặt bàn, Carousel món nổi bật, Section Gợi ý cho bạn (ML)"
- Nội dung hiển thị:
  - Hero call-to-action
  - Món nổi bật
  - Section gợi ý cá nhân hoá

---

## Slide 7: Ba trụ cột giải pháp
1. **Khách hàng**
   - Đặt bàn nhanh
   - Order trực tiếp
   - Review, lịch sử, gợi ý cá nhân hoá
2. **Nhà hàng**
   - Màn hình bếp realtime
   - Quản lý bàn trạng thái, ghép/tách
   - In hóa đơn, gọi món thêm
3. **Quản trị**
   - Dashboard doanh thu
   - Top món, hiệu suất nhân viên
   - User management, cài đặt

---

## Slide 8: Kiến trúc tổng thể
- Browser → Nginx → Django (MTV) → Database
- Redis: cache, session, queue
- Celery: background tasks
- ML service / recommender
- HTMX: realtime kitchen polling

---

## Slide 9: Tech Stack
| Layer | Công nghệ | Phiên bản | Lý do chọn |
|---|---|---|---|
| Backend | Django | 5.2 | MTV, ORM, Admin, Security |
| API | Django REST Framework | 3.15 | Serializer, Permission |
| Async | Celery + Redis | 5.4 / 7.2 | Tasks nền, retrain ML |
| Database dev | SQLite | 3.x | Zero-config |
| Database prod | PostgreSQL | 16 | Production-ready |
| ML | scikit-learn, pandas | 1.5 / 2.2 | Lightweight, hybrid |
| Frontend | Bootstrap | 5.3 | Responsive |
| Interactivity | HTMX | 2.0 | Realtime không JS framework |
| Charts | Chart.js | 4.4 | Responsive visualization |
| Container | Docker Compose | 27.x | Deploy reproducible |
| Web server | Nginx + Gunicorn | latest | Reverse proxy |

---

## Slide 10: Phân quyền & Module
| Role | Quyền truy cập | App |
|---|---|---|
| Customer | Đặt bàn, Xem menu, Order, Review, Lịch sử, Gợi ý ML | `restaurant`, `reservations`, `accounts` |
| Kitchen Staff | Màn hình bếp, In ticket | `restaurant` |
| Waiter | Quản lý bàn, Gọi món, Thanh toán | `restaurant`, `reservations` |
| Manager/Admin | Dashboard, Báo cáo, User mgmt, Settings | `dashboard`, `accounts`, `project2` |

**5 App chính:**
1. `accounts`
2. `restaurant`
3. `reservations`
4. `dashboard`
5. `project2`

---

## Slide 11: Module Khách hàng
- Trang chủ: Hero CTA đặt bàn, carousel món, gợi ý ML
- Menu: Category, tìm kiếm, hình ảnh, giá, mô tả, tag
- Đặt bàn: chọn ngày, giờ, số người, gợi ý bàn phù hợp
- Giỏ hàng + thanh toán: sửa số lượng, split bill, VNPAY sandbox/COD
- Lịch sử & review: xem đơn cũ, đánh giá món, yêu thích

---

## Slide 12: Module Nhà hàng — Bếp & Phục vụ
- Màn hình bếp realtime:
  - HTMX polling 2s
  - 3 cột: Mới → Đang nấu → Hoàn thành
  - Ticket gồm bàn số, món, số lượng, ghi chú
- Quản lý bàn:
  - Trạng thái: trống, có khách, cần dọn, bảo trì
  - Ghép/tách bàn, chuyển bàn
- Thanh toán & in:
  - In hóa đơn thermal
  - Split bill, áp voucher

---

## Slide 13: Module Quản lý
- Dashboard doanh thu: ngày/tuần/tháng
- Top món bán chạy theo thời gian
- Hiệu suất nhân viên
- Quản lý user, phân quyền, reset password
- Cài đặt nhà hàng: tên, logo, thông tin liên hệ, ca làm việc

---

## Slide 14: Hệ gợi ý ML
- Hybrid recommendation: Collaborative + Content-based
- Content-based: tag, category, mô tả món
- Collaborative: user-user similarity, overlap item
- Cold-start:
  - User mới: popularity + category diversity
  - Món mới: content similarity
- Pipeline:
  1. Collect interaction log
  2. Train offline với Celery
  3. Serve top-K recommendation
  4. Log impression và click

---

## Slide 15: Thiết kế CSDL
- Bảng chính:
  - User, CustomerProfile
  - MenuItem, Category
  - Table, Reservation
  - TableOrder, OrderItem
  - Review
- Quan hệ chính:
  - User 1-n Order/Review/Reservation
  - MenuItem 1-n OrderItem/Review
  - Category 1-n MenuItem
- Lưu ý kỹ thuật:
  - price thời điểm order
  - is_available cho menu item
  - unique review user-item

---

## Slide 16: CSDL kỹ thuật
- Soft delete: `is_deleted`, `deleted_at`
- Index tối ưu:
  - Order: `(user_id, created_at)`
  - OrderItem: `(order_id, menu_item_id)`
  - Reservation: `(table_id, shift_id, date)`
  - RecommendationLog: `(user_id, context, timestamp)`
- Migration: `makemigrations` / `migrate`
- Seed demo: `seed_demo` tạo menu, user, order, review

---

## Slide 17: Kết quả demo
- Dữ liệu seed:
  - 33 users
  - 52 menu items
  - 88 orders
- Hệ thống hoạt động:
  - User có lịch sử nhận recommendation
  - User mới nhận trending
- Dữ liệu chương 6:
  - 5 user cụ thể
  - orders + reviews cho từng user

---

## Slide 18: Test & đánh giá
- Test framework: `python manage.py test restaurant.recommender -v2`
- Test coverage: 19 test cases, 6 test classes
- Kiểm thử các phần:
  - Parse tags
  - Content-based
  - Collaborative
  - Scorer
  - Engine end-to-end
- UAT scenarios cơ bản

---

## Slide 19: Kết luận
- Đã xây dựng được hệ thống quản lý nhà hàng end-to-end
- Tích hợp recommender thông minh ngay trong ứng dụng
- Phân quyền rõ ràng cho customer, staff, manager
- Code base deployable với Docker, có test và seed data

---

## Slide 20: Hướng phát triển
- Mobile app native / PWA
- Multi-branch / chuỗi nhà hàng
- AI forecasting nguyên liệu
- Loyalty program, voucher, CRM
- Payment gateway mở rộng: MoMo, ZaloPay, VietQR

---

## Notes: Gợi ý nói cho từng slide
- **Slide 1:** Giới thiệu bản thân, đề tài, GVHD.
- **Slide 2:** Nêu thực trạng và vấn đề cụ thể của nhà hàng VN.
- **Slide 3:** So sánh giải pháp, nhấn vào lợi thế FourSeason.
- **Slide 4:** Mục tiêu và phạm vi thực hiện.
- **Slide 5:** Mục tiêu chính và đối tượng sử dụng.
- **Slide 6:** Mô tả nhanh homepage và điểm nổi bật.
- **Slide 7:** Giải pháp theo 3 trụ cột nghiệp vụ.
- **Slide 8:** Kiến trúc tổng thể và công nghệ nền.
- **Slide 9:** Tech stack, lý do chọn.
- **Slide 10:** Phân quyền, app chính.
- **Slide 11:** Chức năng khách hàng.
- **Slide 12:** Chức năng nhà hàng/bếp/phục vụ.
- **Slide 13:** Dashboard quản lý.
- **Slide 14:** Cấu trúc recommender và lợi ích.
- **Slide 15:** Thiết kế dữ liệu.
- **Slide 16:** Kỹ thuật CSDL và seed.
- **Slide 17:** Kết quả demo và số liệu.
- **Slide 18:** Test coverage và UAT.
- **Slide 19:** Kết luận chính.
- **Slide 20:** Hướng phát triển tương lai.


-> Dataset lớn (> 100,000 records) và ưu tiên hiệu năng

-> Ví dụ trên Facebook/Instagram feed dùng cursor-based vì dữ liệu luôn thay đổi và cần hiệu năng cao
