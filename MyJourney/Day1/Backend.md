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

**Tài liệu**: `flow9-docs/DATABASE_DESIGN.md`

---

### 2. Thiết kế Frontend UI

Tạo design system dựa trên mẫu từ `design_test_2`:

- **Color palette**: Teal (#0D9488) + Zinc dark theme
- **Layout**: Sidebar 240px + Main content
- **Components**: Cards, Charts, Forms, Quick Input
- **Pages**: Login, Dashboard, Payroll, Subscriptions, Ledger

**Tài liệu**: `flow9-docs/FRONTEND_DESIGN.md`

---

### 3. Thiết kế Backend API

Tạo API specification với:

- 7 modules chính
- 25+ endpoints
- 3 core services (NLP Parser, Salary Calculator, Subscription Reminder)
- 2 Cron Jobs (Subscription checker, Recurring transactions)
- SSE cho real-time notifications

**Tài liệu**: `flow9-docs/BACKEND_DESIGN.md`

---

### 4. Code Backend (Express + TypeScript)

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

### 5. Kết nối MongoDB Atlas

- Sử dụng MongoDB Atlas với connection string SRV
- Cấu hình trong `.env`:
  ```
  MONGODB_URI=mongodb+srv://pata10102004_db_user:Binhminh%401010@flow9.0tdh9i3.mongodb.net/?appName=flow9
  ```

---

### 6. API Endpoints đã tạo

| Module | Method | Endpoint |
|--------|--------|----------|
| Auth | POST | `/api/auth/login` |
| Auth | POST | `/api/auth/verify` |
| Auth | POST | `/api/auth/change-pin` |
| Shifts | GET/POST/PUT/DELETE | `/api/shifts` |
| Salary | GET | `/api/salary/config`, `/api/salary/summary` |
| Subscriptions | CRUD | `/api/subscriptions` |
| Transactions | CRUD + Aggregate | `/api/transactions` |
| Categories | GET/POST/DELETE | `/api/categories` |
| Notifications | GET | `/api/notifications/stream` (SSE) |

---

### 7. Core Services

#### NLP Parser
- Parse input như "Ăn trưa 50k" → `{ type: 'expense', amount: 50000, category: 'Ăn uống' }`
- Hỗ trợ: k, tr, triệu
- Tự động nhận diện danh mục

#### Salary Calculator
- Công thức: `hours × hourlyRate × shiftMultiplier × holidayMultiplier`
- Tính tổng lương tháng theo loại ca

---

### 8. Cron Jobs

| Job | Schedule | Task |
|-----|----------|------|
| Subscription Checker | Daily 9:00 AM | Kiểm tra subscription sắp hết hạn |
| Recurring Transactions | Daily 1:00 AM | Tạo transaction từ recurring config |

---

## ✅ Kết quả

- Backend compile thành công
- Server chạy trên port 3001
- MongoDB Atlas kết nối thành công
- Health check endpoint hoạt động: `GET /health`

---

## 🔜 Ngày mai (Day 2)

- Setup Frontend (Next.js)
- Tích hợp Frontend với Backend API
- Login page với PIN
- Dashboard với charts

---

## 📁 Files đã tạo

```
flow9-docs/
├── PROJECT_BRIEF.md         # Yêu cầu dự án
├── TECH_SPEC.md             # Tech stack, API
├── DATABASE_DESIGN.md       # 7 MongoDB collections
├── FRONTEND_DESIGN.md       # UI/UX design
├── BACKEND_DESIGN.md        # API design
└── MyJourney/
    └── Day1.md             # File này

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
├── package.json
└── tsconfig.json

flow9-frontend/              # Đã có từ trước (Next.js + Tailwind)
```
