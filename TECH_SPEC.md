# Personal Life OS - Technical Specification

## 1. Tech Stack

- **Frontend**: Next.js 14 (App Router) + TypeScript + Tailwind CSS
- **Backend**: Node.js/Express + TypeScript
- **Database**: MongoDB (Mongoose ODM)
- **Charts**: Recharts
- **Authentication**: JWT + PIN code

---

## 2. Database Collections (MongoDB)

### users
```json
{
  "_id": "ObjectId",
  "pin": "string (hashed bcrypt)",
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

### work_shifts
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "date": "Date",
  "hours": "number",
  "shiftType": "day | night | holiday",
  "hourlyRate": "number",
  "dailySalary": "number",
  "notes": "string",
  "createdAt": "Date"
}
```

### salary_config
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "baseHourlyRate": "number",
  "dayShiftMultiplier": "number",
  "nightShiftMultiplier": "number",
  "holidayMultiplier": "number",
  "updatedAt": "Date"
}
```

### subscriptions
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "name": "string",
  "amount": "number",
  "currency": "VND | USD",
  "billingCycle": "monthly | yearly | weekly",
  "nextBillingDate": "Date",
  "category": "string",
  "isActive": "boolean",
  "reminderDays": "number",
  "notified": "boolean",
  "createdAt": "Date"
}
```

### transactions
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "type": "income | expense",
  "amount": "number",
  "category": "string",
  "description": "string",
  "date": "Date",
  "createdAt": "Date"
}
```

### categories
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "name": "string",
  "type": "income | expense",
  "icon": "string",
  "color": "string"
}
```

---

## 3. API Endpoints

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/verify-pin` | Xác thực PIN, trả về JWT |
| POST | `/api/auth/change-pin` | Đổi PIN |

### Module A: Payroll
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/shifts` | Lấy danh sách ca làm (theo tháng) |
| POST | `/api/shifts` | Tạo ca làm mới |
| PUT | `/api/shifts/:id` | Cập nhật ca làm |
| DELETE | `/api/shifts/:id` | Xóa ca làm |
| GET | `/api/salary/summary` | Tổng lương tháng |
| GET | `/api/salary/config` | Lấy cấu hình lương |
| PUT | `/api/salary/config` | Cập nhật cấu hình lương |

### Module B: Subscriptions
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/subscriptions` | Lấy danh sách subscriptions |
| POST | `/api/subscriptions` | Tạo subscription mới |
| PUT | `/api/subscriptions/:id` | Cập nhật subscription |
| DELETE | `/api/subscriptions/:id` | Xóa subscription |
| GET | `/api/subscriptions/upcoming` | Lấy các subscription sắp đến hạn |

### Module C: Smart Ledger
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/transactions` | Lấy danh sách giao dịch |
| POST | `/api/transactions` | Tạo giao dịch (hỗ trợ NLP) |
| GET | `/api/transactions/aggregate` | Thống kê theo tháng/danh mục |
| GET | `/api/categories` | Lấy danh mục |
| POST | `/api/categories` | Tạo danh mục mới |

### Cron Job
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/cron/check-subscriptions` | Chạy mỗi ngày, kiểm tra subscription sắp hết hạn |

---

## 4. NLP Parsing Logic (Module C)

### Regex Patterns
```typescript
// Pattern để tách số tiền
const amountPattern = /(\d+(?:\.\d+)?)\s*(k|tr|nghìn|triệu)?/i;

// Danh mục chi tiêu
const expenseCategories = ['ăn', 'uống', 'đi lại', 'xăng', 'mua', 'tiêu', 'bill', 'điện', 'nước', 'internet', 'shopee', 'lazada'];

// Danh mục thu nhập
const incomeCategories = ['lương', 'thưởng', 'tiền', 'nhận', 'bán', 'lãi'];
```

### Examples
| Input | Output |
|-------|--------|
| "Ăn trưa 50k" | `{ type: 'expense', amount: 50000, category: 'Ăn uống' }` |
| "Mua sữa 45k" | `{ type: 'expense', amount: 45000, category: 'Mua sắm' }` |
| "Lương tháng 15tr" | `{ type: 'income', amount: 15000000, category: 'Lương' }` |
| "Thưởng Tết 5tr" | `{ type: 'income', amount: 5000000, category: 'Thưởng' }` |

---

## 5. Salary Calculation Logic (Module A)

### Công thức
```
dailySalary = hours × baseHourlyRate × shiftMultiplier × holidayMultiplier
```

### Example
```
Ca ngày (8 giờ): 8 × 50000 × 1.5 = 600,000 VND
Ca đêm (8 giờ): 8 × 50000 × 2.0 = 800,000 VND
Ngày lễ (8 giờ): 8 × 50000 × 3.0 = 1,200,000 VND
```

### Default Config
```json
{
  "baseHourlyRate": 50000,
  "dayShiftMultiplier": 1.5,
  "nightShiftMultiplier": 2.0,
  "holidayMultiplier": 3.0
}
```

---

## 6. Cron Job Logic (Module B)

### Timing
- Chạy mỗi ngày vào lúc **9:00 AM**

### Algorithm
1. Query subscriptions với điều kiện:
   - `isActive = true`
   - `nextBillingDate - today <= reminderDays`
   - `notified = false`

2. Với mỗi subscription tìm được:
   - Gửi notification (lưu vào notifications collection hoặc return realtime)
   - Set `notified = true`

3. Reset `notified = false` khi đến ngày billing mới

---

## 7. Authentication Flow

1. User mở app → Redirect to `/login`
2. Nhập 4-6 số PIN
3. Server verify PIN (bcrypt compare) → Return JWT token
4. Token lưu trong httpOnly cookie, expire sau 7 ngày
5. Tất cả API requests require valid token (middleware check)

### Session Flow
```
/login (POST PIN)
    ↓
Verify PIN → Generate JWT
    ↓
Set-Cookie: token=xxx; HttpOnly; Max-Age=604800
    ↓
Redirect to / (dashboard)
    ↓
All subsequent requests include cookie
```

---

## 8. Responsive Design

### Breakpoints
| Breakpoint | Width | Layout |
|------------|-------|--------|
| Mobile | < 640px | List view, hamburger menu |
| Tablet | 640px - 1024px | Grid 2 columns |
| Desktop | > 1024px | Grid 3-4 columns, full sidebar |

### Grid to List Transition
- **Desktop**: Cards hiển thị dạng lưới (grid)
- **Mobile**: Tự động chuyển thành danh sách dọc (list view)
- Sử dụng Tailwind: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4`

---

## 9. Project Structure

```
PersonalWorld/
├── frontend/                    # Next.js App
│   ├── src/
│   │   ├── app/               # App Router pages
│   │   │   ├── (auth)/        # Auth pages
│   │   │   │   └── login/     # Login page
│   │   │   ├── (dashboard)/   # Protected pages
│   │   │   │   ├── page.tsx   # Dashboard
│   │   │   │   ├── payroll/   # Module A
│   │   │   │   ├── subscriptions/  # Module B
│   │   │   │   ├── ledger/    # Module C
│   │   │   │   └── settings/  # Settings
│   │   │   ├── api/           # API routes (proxies to backend)
│   │   │   └── layout.tsx    # Root layout
│   │   ├── components/       # Reusable components
│   │   │   ├── ui/           # Base UI components
│   │   │   └── modules/      # Module-specific components
│   │   ├── lib/              # Utilities, API client
│   │   ├── hooks/            # Custom React hooks
│   │   └── types/           # TypeScript types
│   ├── package.json
│   ├── tailwind.config.ts
│   └── tsconfig.json
├── backend/                    # Express API
│   ├── src/
│   │   ├── controllers/      # Route handlers
│   │   ├── models/           # Mongoose schemas
│   │   ├── routes/          # API routes
│   │   ├── services/         # Business logic
│   │   ├── middleware/       # Auth, validation
│   │   ├── cron/            # Cron jobs
│   │   └── index.ts        # Entry point
│   ├── package.json
│   └── tsconfig.json
├── docs/                      # Documentation
│   ├── PROJECT_BRIEF.md
│   └── TECH_SPEC.md
├── SPEC.md                   # Main specification
└── README.md
```

---

## 10. Implementation Priority

### Phase 1: Setup + Auth
1. Khởi tạo Next.js + Express projects
2. Setup MongoDB connection
3. Implement PIN authentication
4. Tạo protected routes

### Phase 2: Module C - Smart Ledger (MVP)
1. Tạo transactions collection
2. Implement NLP parsing service
3. CRUD transactions API
4. Dashboard với quick input
5. Transaction list view

### Phase 3: Module A - Payroll
1. Tạo work_shifts + salary_config collections
2. Calendar UI component
3. CRUD shifts API
4. Salary calculation service
5. Monthly summary

### Phase 4: Module B - Subscriptions
1. Tạo subscriptions collection
2. CRUD subscriptions API
3. Grid/List view UI
4. Implement Cron job
5. Notification system

### Phase 5: Polish
1. Thêm charts (Recharts)
2. Responsive refinement
3. Performance optimization
4. Error handling

---

## 11. Dependencies

### Frontend
```json
{
  "next": "^14.0.0",
  "react": "^18.2.0",
  "recharts": "^2.10.0",
  "clsx": "^2.0.0",
  "lucide-react": "^0.300.0"
}
```

### Backend
```json
{
  "express": "^4.18.0",
  "mongoose": "^8.0.0",
  "bcryptjs": "^2.4.3",
  "jsonwebtoken": "^9.0.0",
  "node-cron": "^3.0.0",
  "cors": "^2.8.5",
  "dotenv": "^16.0.0"
}
```
