# BẢN TÓM TẮT DỰ ÁN (PROJECT BRIEF)

Tên dự án: Personal Life OS (Hệ sinh thái quản lý cá nhân)
Mô hình: Single-user Web Application (Ứng dụng web dành cho 1 người dùng duy nhất)
Tech Stack yêu cầu: Next.js (Frontend) - Node.js (Backend) - MongoDB (Database)

---

## 1. Mục tiêu cốt lõi

Tạo ra một công cụ nội bộ để số hóa 3 mảng tài chính cá nhân:
- Tính toán tự động lương đi làm
- Kiểm soát các khoản tiền bị trừ tự động hàng tháng
- Theo dõi thu chi thường nhật

---

## 2. Ba Module chính Dev cần code

### Module A: Golden Gate Payroll (Tính lương & Ca làm)

- **UI**: Xây dựng UI dạng Lịch (Calendar) để chọn ngày và nhập số giờ làm
- **Logic Backend**: Viết hàm nhận đầu vào là (số giờ, loại ca, ngày Lễ/Tết) → nhân với hệ số cấu hình sẵn → trả về số tiền lương ngày. Tự động cộng dồn thành lương tháng và lưu vào database

### Module B: Subscription Tracker (Trạm gác Gói cước)

- **UI**: Form nhập thông tin gói cước (Tên dịch vụ, Số tiền, Ngày gia hạn tiếp theo)
- **Logic Backend**: Phải setup được Cron Job (tác vụ chạy ngầm) trên Node.js. Mỗi ngày quét database 1 lần. Nếu phát hiện các gói như YouTube Premium hay Apple Music còn đúng 2 ngày nữa là đến hạn thanh toán, API phải bắn ra cảnh báo (Notification) ngay lập tức

### Module C: Smart Ledger (Sổ Thu Chi)

- **UI**: Tạo một thanh nhập liệu cực nhanh (Ví dụ gõ: "Ăn trưa 50k"). Xây dựng Dashboard hiển thị biểu đồ
- **Logic Backend**: Dùng Regex hoặc logic bóc tách chuỗi để tự nhận diện số tiền và danh mục. Viết các câu lệnh Aggregation trong MongoDB để nhóm tổng tiền tiêu theo tháng, sau đó trả data về cho Frontend vẽ biểu đồ (Chart.js/Recharts)

---

## 3. Yêu cầu kỹ thuật bắt buộc (NFRs)

- **Database Schema-less**: Bắt buộc dùng cấu trúc Document của MongoDB để sau này muốn thêm field (ví dụ: thêm loại phụ cấp mới vào lương) thì không phải sửa lại toàn bộ bảng
- **Responsive**: UI phải hiển thị tốt dạng bảng lưới trên màn hình laptop ngang, nhưng phải tự động co lại thành dạng danh sách dọc (list view) mượt mà, dễ bấm khi mở trên điện thoại
- **Auth (Bảo mật)**: Không làm luồng Đăng ký (Register) phức tạp. Cấp cứng 1 tài khoản Admin duy nhất (hoặc bảo vệ bằng mã PIN/JWT token) để khóa người lạ

---

## 4. Tech Stack đã chọn

- **TypeScript**: Có
- **Tailwind CSS**: Có
- **Authentication**: PIN code (4-6 số)
