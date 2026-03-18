# Flow9 - Day 1: Backend Setup & Database Design

## Ngày: 2026-03-18

---

## 🎯 Mục tiêu ngày hôm nay

Thiết kế và xây dựng phần backend cho Flow9 - Personal Life OS

---

## 📋 Những gì đã làm

### 1. Thiết kế Database (MongoDB)

Tạo 7 collections theo yêu cầu:

| Collection | Mục đích |
|------------|-----------|
| `users` | Tài khoản admin (PIN hash) |
| `work_shifts` | Ca làm việc |
| `salary_config` | Cấu hình lương |
| `subscriptions` | Gói cước (có paymentHistory) |
| `transactions` | Thu chi (có recurring) |
| `categories` | Danh mục |
| `audit_logs` | Lịch sử thay đổi |

---

### 2. Code Backend (Express + TypeScript)

Cấu trúc project:

```
flow9-backend/
├── src/
│   ├── config/
│   │   └── database.ts        # MongoDB connection
│   ├── models/                 # 7 Mongoose schemas
│   │   ├── User.ts
│   │   ├── WorkShift.ts
│   │   ├── SalaryConfig.ts
│   │   ├── Subscription.ts
│   │   ├── Transaction.ts
│   │   ├── Category.ts
│   │   └── AuditLog.ts
│   ├── routes/                 # API routes
│   │   ├── auth.ts
│   │   ├── shifts.ts
│   │   ├── salary.ts
│   │   ├── subscriptions.ts
│   │   ├── transactions.ts
│   │   ├── categories.ts
│   │   └── notifications.ts
│   ├── services/
│   │   ├── nlpParser.ts       # NLP parsing
│   │   └── salaryCalculator.ts
│   ├── middleware/
│   │   └── auth.ts            # JWT middleware
│   ├── cron/
│   │   └── index.ts           # Cron jobs
│   └── index.ts               # Entry point
├── .env                       # Environment variables
├── package.json
└── tsconfig.json
```

---

### 3. API Endpoints

| Module | Method | Endpoint | Mô tả |
|--------|--------|----------|--------|
| Auth | POST | `/api/auth/login` | Login với PIN 6 số |
| Auth | POST | `/api/auth/verify` | Verify JWT token |
| Auth | POST | `/api/auth/change-pin` | Đổi PIN |
| Shifts | GET | `/api/shifts` | Lấy ca làm |
| Shifts | POST | `/api/shifts` | Tạo ca làm |
| Shifts | PUT | `/api/shifts/:id` | Cập nhật ca làm |
| Shifts | DELETE | `/api/shifts/:id` | Xóa ca làm |
| Salary | GET | `/api/salary/config` | Lấy cấu hình lương |
| Salary | PUT | `/api/salary/config` | Cập nhật cấu hình lương |
| Salary | GET | `/api/salary/summary` | Tổng lương tháng |
| Subscriptions | GET | `/api/subscriptions` | Lấy subscriptions |
| Subscriptions | GET | `/api/subscriptions/upcoming` | Subscriptions sắp đến hạn |
| Subscriptions | POST | `/api/subscriptions` | Tạo subscription |
| Subscriptions | PUT | `/api/subscriptions/:id` | Cập nhật subscription |
| Subscriptions | POST | `/api/subscriptions/:id/pay` | Đánh dấu đã thanh toán |
| Subscriptions | DELETE | `/api/subscriptions/:id` | Xóa subscription |
| Transactions | GET | `/api/transactions` | Lấy giao dịch |
| Transactions | POST | `/api/transactions` | Tạo giao dịch (NLP) |
| Transactions | PUT | `/api/transactions/:id` | Cập nhật giao dịch |
| Transactions | DELETE | `/api/transactions/:id` | Xóa giao dịch |
| Transactions | GET | `/api/transactions/aggregate` | Thống kê chi tiêu |
| Categories | GET | `/api/categories` | Lấy danh mục |
| Categories | POST | `/api/categories` | Tạo danh mục |
| Categories | DELETE | `/api/categories/:id` | Xóa danh mục |
| Notifications | GET | `/api/notifications/stream` | SSE notification |

---

### 4. NLP Parser

Parse input tự nhiên thành transaction:

```
# Chi tiêu (tự nhận diện)
Ăn trưa 50k
Mua sắm 200k
Xăng 100k

# Thu nhập (explicit)
+ Lương 15tr
+ Thưởng 2tr
- Mua sắm 500k

# Shorthand
k = nghìn (50k = 50,000)
tr = triệu (1.5tr = 1,500,000)
```

---

### 5. Cron Jobs

| Job | Schedule | Task |
|-----|----------|------|
| Subscription Checker | Daily 9:00 AM | Kiểm tra subscription sắp hết hạn |
| Recurring Transactions | Daily 1:00 AM | Tạo transaction từ recurring config |

---

## 🔧 Bug Fixes

### 1. PIN Length - 6 digits
- Thay đổi từ 4-6 digits → chỉ 6 digits
- Cập nhật cả frontend và backend validation

### 2. ObjectId Aggregate Query
- Fix vấn đề aggregate query không tìm thấy transactions
- Chuyển đổi userId string sang ObjectId trong MongoDB aggregation pipeline

---

## 📁 Files đã tạo

```
flow9-backend/
├── src/
│   ├── config/database.ts
│   ├── models/ (7 files)
│   ├── routes/ (7 files)
│   ├── services/ (2 files)
│   ├── middleware/auth.ts
│   ├── cron/index.ts
│   └── index.ts
├── .env
├── README.md
├── package.json
└── tsconfig.json
```

---

## ✅ Kết quả

- Backend compile và chạy thành công
- Server chạy trên port 3001
- MongoDB Atlas kết nối thành công
- API tích hợp với frontend hoàn chỉnh
- PIN 6 digits, NLP parser hoạt động
- Subscriptions có payment history và mark as paid

---

## 🚀 Deploy

### Backend Options
- **Railway** (Khuyến nghị)
- **Render**
- **VPS**

### Environment Variables cần thiết
```env
PORT=3001
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/flow9
JWT_SECRET=your-secret-key
FRONTEND_URL=https://your-frontend.vercel.app
```
