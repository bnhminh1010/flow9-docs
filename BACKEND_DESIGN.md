**Kế Hoạch Backend (Revised) – Flow9**

**Tóm tắt**
- Chốt kiến trúc backend theo Express + TypeScript + Mongoose, phục vụ single‑user.
- Thống nhất auth bằng **JWT trong HttpOnly cookie**, bootstrap admin từ ENV, và notification **lưu DB + SSE realtime**.
- Làm rõ schema chuẩn, chuẩn hóa API, validation, pagination, timezone, và cron idempotency.

**Thay đổi/chuẩn hóa chính (API & Schema)**
1. **Auth (public API)**
   - `POST /api/auth/login`  
     Body: `{ pin: string }` → Set `token` cookie (HttpOnly, SameSite=Lax, Secure on prod) + `{ user: { id, name } }`
   - `POST /api/auth/change-pin`  
     Body: `{ oldPin, newPin }`
   - `GET /api/auth/verify`  
     Returns `{ valid: true, userId }` nếu token hợp lệ
   - `POST /api/auth/logout`  
     Clear cookie (optional but recommended)

2. **Collections (canonical schema)**
   - `users`: `{ pinHash, name, settings: { currency, timezone }, createdAt, updatedAt }`
   - `work_shifts`: `{ userId, date, hours, shiftType, hourlyRate, dailySalary, isHoliday, notes, createdAt, updatedAt }`
   - `salary_config`: `{ userId, baseHourlyRate, dayShiftMultiplier, nightShiftMultiplier, holidayMultiplier, overtimeMultiplier?, allowances?, updatedAt }`
   - `subscriptions`: `{ userId, name, amount, currency, billingCycle, nextBillingDate, category?, isActive, reminderDays, notified, lastNotifiedAt?, paymentHistory?, createdAt, updatedAt }`
   - `transactions`: `{ userId, type, amount, currency?, category, categoryId?, description?, date, tags?, attachments?, isRecurring?, recurringConfig?, createdAt, updatedAt }`
   - `categories`: `{ userId, name, type, icon?, color?, isDefault?, sortOrder?, createdAt }`
   - `notifications`: `{ userId, type, title, message, data?, readAt?, createdAt }`
   - `audit_logs` giữ như thiết kế hiện tại (optional, có thể hoãn v1)

3. **Endpoints (public API chuẩn hóa)**
   - Payroll: `/api/shifts` (GET by `year`,`month`, pagination optional), `/api/salary/config`, `/api/salary/summary`
   - Subscriptions: `/api/subscriptions`, `/api/subscriptions/upcoming`
   - Ledger: `/api/transactions` (GET with filters), `/api/transactions/aggregate`, `/api/categories`
   - Notifications: `GET /api/notifications` (list), `GET /api/notifications/stream` (SSE)
   - Cron (internal): `POST /api/cron/check-subscriptions` (guarded bằng secret header)

4. **Data & Time rules**
   - Lưu `Date` theo **UTC** trong DB, query theo **timezone của user** (default `Asia/Ho_Chi_Minh`).
   - `amount` lưu **integer ở đơn vị nhỏ nhất theo currency** (VND: đồng, USD: cents).

5. **Validation & Error format**
   - Dùng `zod` cho request validation.
   - Response thống nhất:
     - Success: `{ success: true, data: ... }`
     - Error: `{ success: false, error: { code, message } }`

6. **Cron & Notification idempotency**
   - Cron chạy 09:00 mỗi ngày theo timezone user.
   - Query subscriptions với `isActive=true`, `nextBillingDate`, `notified=false`.
   - Dùng `findOneAndUpdate`/`updateMany` có điều kiện để tránh double‑send khi chạy lại.
   - Tạo record `notifications` + SSE push realtime.

**Test Plan**
1. Auth
   - Login đúng PIN → set cookie và `verify` thành công.
   - Login sai PIN → 401 + rate limit kích hoạt.
   - Change PIN → login bằng PIN mới thành công.
2. Payroll
   - Tạo shift → dailySalary đúng công thức.
   - Summary theo tháng chính xác.
3. Subscriptions
   - CRUD + upcoming trả `daysUntil`.
   - Cron chạy lại không tạo duplicate notification.
4. Ledger
   - NLP parse “Ăn trưa 50k” → amount 50000, category đúng.
   - Aggregate theo tháng & category đúng.
5. Notifications
   - SSE stream nhận notification mới.
   - `GET /api/notifications` trả lịch sử đầy đủ.

**Assumptions & Defaults**
- Auth dùng HttpOnly cookie (SameSite=Lax; Secure ở prod).
- Admin bootstrap từ ENV (`ADMIN_PIN`, `ADMIN_NAME`) nếu DB chưa có user.
- Notification lưu DB + SSE.
- `zod` cho validation, `node-cron` cho schedule, `luxon` cho timezone.
- Pagination mặc định `page=1`, `limit=20` cho list endpoints.
