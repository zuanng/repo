# 1. Phân trang Limit/Offset

# Cách hoạt động:

- Sử dụng hai tham số: `LIMIT` (số lượng bản ghi) và `OFFSET` (bỏ qua bao nhiêu bản ghi)

- Ví dụ câu lệnh query: <pre> ``` SELECT * FROM users LIMIT 10 OFFSET 20 ``` </pre>

# Ưu điểm:

- Linh hoạt: Có thể nhảy đến bất kỳ trang nào. 

# Nhược điểm:

- Hiệu năng kém với OFFSET lớn: Database phải đếm và bỏ qua nhiều bản ghi

- Khi có dữ liệu mới được thêm/xóa, có thể bị trùng lặp hoặc thiếu dữ liệu (khi xoá thì bản ghi đầu tiên của trang sau sẽ bị đẩy lên trang trước làm khi chuyển sang trang sau bị thiếu dữ liệu)

<pre> ```md | ID | Name | Created_at | |----|---------|------------| | 1 | A | 2024-01-01 | | 2 | B | 2024-01-02 | | 3 | C | 2024-01-03 | | 4 | D | 2024-01-04 | | 5 | E | 2024-01-05 | | 6 | F | 2024-01-06 | | 7 | G | 2024-01-07 | | 8 | H | 2024-01-08 | ``` </pre>

Khi user xem trang 1 với `LIMIT = 3 OFFSET = 0`, cùng lúc đó `id = 1` bị xoá. Khi đó sang trang 2 sẽ bắt đầu với `id = 5` làm `id = 4` biến mất.

- Tốn tài nguyên: OFFSET càng lớn càng chậm

-> Dataset nhỏ hoặc vừa (< 10,000 records)

-> Ví dụ trong Google Search dùng limit/offset vì người dùng cần nhảy đến trang cụ thể"

# 2. Phân trang dựa trên con trỏ (Cursor-based)

# Cách hoạt động:

- Sử dụng một giá trị duy nhất (cursor) để xác định vị trí hiện tại

- Thường dùng ID, timestamp, hoặc composite key

- Ví dụ query: 

    + ID: <pre> ``` SELECT * FROM users WHERE id > 1000 LIMIT 10 ``` </pre>

    + Timestamp: <pre> ``` SELECT * FROM posts WHERE created_at < '2025-09-09 17:00:00'; ``` </pre>

    + Composite key (khi trùng timestamp): <pre> ``` SELECT * FROM posts WHERE created_at < '2025-09-09 12:00:00' OR (created_at = '2025-09-09 12:00:00' AND id < 42); ``` </pre>

(Kết hợp `ORDER BY` để lọc được chính xác)

## Ưu điểm:

- Hiệu năng cao: Luôn nhanh bất kể vị trí và độ lớn dữ liệu vì Cursor trỏ đến một giá trị cụ thể, không phải vị trí

- Không bị ảnh hưởng bởi việc thêm/xóa dữ liệu (điều kiện WHERE cố định: `WHERE id > cursor_value` không thay đổi dù có thêm/xóa dữ liệu)

- Phù hợp với real-time data, lý tưởng cho feed, timeline 

## Nhược điểm:

- Phức tạp hơn so với Limit/Offset: Không thể nhảy trang, chỉ có thể next/previous và phụ thuộc vào thứ tự: cần có trường có thể sắp xếp duy nhất

-> Dataset lớn (> 100,000 records) và ưu tiên hiệu năng

-> Ví dụ trên Facebook/Instagram feed dùng cursor-based vì dữ liệu luôn thay đổi và cần hiệu năng cao
