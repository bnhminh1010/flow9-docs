# Flow9 Database Design

## Overview

- **Project**: Flow9 - Personal Life OS
- **Database**: MongoDB (Schema-less Document)
- **Purpose**: Quản lý tài chính cá nhân - Lương, Subscription, Thu chi

---

## Collections

### 1. users

```json
{
  "_id": ObjectId,
  "pin": "hashed_pin_string",
  "name": "Admin",
  "settings": {
    "currency": "VND",
    "timezone": "Asia/Ho_Chi_Minh"
  },
  "createdAt": Date,
  "updatedAt": Date
}
```

**Notes**:
- PIN được hash bằng bcrypt với rounds = 10
- Duy nhất 1 user admin

---

### 2. work_shifts

```json
{
  "_id": ObjectId,
  "userId": ObjectId (ref: users),
  "date": Date,
  "hours": Number,
  "shiftType": "day | night | holiday",
  "hourlyRate": Number,
  "dailySalary": Number,
  "isHoliday": Boolean,
  "notes": String,
  "createdAt": Date,
  "updatedAt": Date
}
```

**Index**: `{ userId: 1, date: -1 }`

**Salary Calculation**:
```
dailySalary = hours × hourlyRate × shiftMultiplier × (isHoliday ? holidayMultiplier : 1)
```

---

### 3. salary_config

```json
{
  "_id": ObjectId,
  "userId": ObjectId (ref: users),
  "baseHourlyRate": Number,
  "dayShiftMultiplier": Number,
  "nightShiftMultiplier": Number,
  "holidayMultiplier": Number,
  "overtimeMultiplier": Number,
  "allowances": [
    { "name": String, "amount": Number }
  ],
  "updatedAt": Date
}
```

**Default Values**:
- baseHourlyRate: 50000
- dayShiftMultiplier: 1.5
- nightShiftMultiplier: 2.0
- holidayMultiplier: 3.0
- overtimeMultiplier: 1.5

---

### 4. subscriptions

```json
{
  "_id": ObjectId,
  "userId": ObjectId (ref: users),
  "name": String,
  "amount": Number,
  "currency": "VND | USD",
  "billingCycle": "weekly | monthly | yearly",
  "nextBillingDate": Date,
  "category": String,
  "isActive": Boolean,
  "reminderDays": Number,
  "notified": Boolean,
  "lastNotifiedAt": Date,
  "paymentMethod": String,
  "notes": String,
  "paymentHistory": [
    {
      "date": Date,
      "amount": Number,
      "status": "paid | pending | failed",
      "transactionId": ObjectId
    }
  ],
  "createdAt": Date,
  "updatedAt": Date
}
```

**Index**: `{ userId: 1, isActive: 1, nextBillingDate: 1 }`

**Features**:
- Lưu lịch sử thanh toán (paymentHistory)
- Tự động nhắc nhở khi sắp hết hạn

---

### 5. transactions

```json
{
  "_id": ObjectId,
  "userId": ObjectId (ref: users),
  "type": "income | expense",
  "amount": Number,
  "category": String,
  "categoryId": ObjectId (ref: categories),
  "description": String,
  "date": Date,
  "tags": [String],
  "attachments": [String],
  "isRecurring": Boolean,
  "recurringConfig": {
    "frequency": "daily | weekly | monthly",
    "endDate": Date,
    "nextRunDate": Date
  },
  "createdAt": Date,
  "updatedAt": Date
}
```

**Index**: `{ userId: 1, date: -1 }`, `{ userId: 1, type: 1 }`

**Features**:
- Hỗ trợ recurring transactions (giao dịch lặp lại)
- NLP parsing: "Ăn trưa 50k" → { type: 'expense', amount: 50000, category: 'Ăn uống' }

---

### 6. categories

```json
{
  "_id": ObjectId,
  "userId": ObjectId (ref: users),
  "name": String,
  "type": "income | expense",
  "icon": String,
  "color": String,
  "isDefault": Boolean,
  "sortOrder": Number,
  "createdAt": Date
}
```

**Default Categories**:

| Type | Name | Icon | Color |
|------|------|------|-------|
| expense | Ăn uống | 🍔 | #ef4444 |
| expense | Mua sắm | 🛒 | #f97316 |
| expense | Đi lại | 🚗 | #eab308 |
| expense | Điện/nước | 💡 | #22c55e |
| expense | Giải trí | 🎬 | #8b5cf6 |
| income | Lương | 💰 | #10b981 |
| income | Thưởng | 🎁 | #06b6d4 |
| income | Khác | 💵 | #6366f1 |

---

### 7. audit_logs

```json
{
  "_id": ObjectId,
  "userId": ObjectId (ref: users),
  "action": "create | update | delete",
  "collection": String,
  "documentId": ObjectId,
  "changes": {
    "before": Object,
    "after": Object
  },
  "ipAddress": String,
  "userAgent": String,
  "createdAt": Date
}
```

**Purpose**: Lưu lịch sử thay đổi (who changed what)

---

## Relationships

```
users (1)
    │
    ├── work_shifts (N)
    ├── salary_config (N)
    ├── subscriptions (N)
    ├── transactions (N)
    ├── categories (N)
    ├── audit_logs (N)
    └── notifications (real-time via SSE)
```

---

## Aggregation Queries

### Module C - Smart Ledger

```javascript
// Tổng chi theo tháng
db.transactions.aggregate([
  { $match: { userId: ObjectId, type: 'expense' } },
  { $group: { 
    _id: { year: { $year: "$date" }, month: { $month: "$date" } }, 
    total: { $sum: "$amount" } 
  }},
  { $sort: { "_id.year": -1, "_id.month": -1 } }
])

// Chi theo danh mục (tháng hiện tại)
db.transactions.aggregate([
  { $match: { 
    userId: ObjectId, 
    type: 'expense',
    date: { $gte: startOfMonth, $lte: endOfMonth }
  }},
  { $group: { 
    _id: "$category", 
    total: { $sum: "$amount" },
    count: { $sum: 1 }
  }},
  { $sort: { total: -1 } }
])

// Thu vs Chi theo ngày (7 ngày gần nhất)
db.transactions.aggregate([
  { $match: { 
    userId: ObjectId,
    date: { $gte: sevenDaysAgo }
  }},
  { $group: { 
    _id: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
    income: { $sum: { $cond: [{ $eq: ["$type", "income"] }, "$amount", 0] } },
    expense: { $sum: { $cond: [{ $eq: ["$type", "expense"] }, "$amount", 0] } }
  }},
  { $sort: { "_id": 1 } }
])
```

### Module A - Payroll

```javascript
// Tổng lương tháng
db.workShifts.aggregate([
  { $match: { 
    userId: ObjectId,
    date: { $gte: startOfMonth, $lte: endOfMonth }
  }},
  { $group: { 
    _id: null, 
    totalSalary: { $sum: "$dailySalary" },
    totalHours: { $sum: "$hours" },
    totalShifts: { $sum: 1 }
  }}
])

// Lương theo loại ca
db.workShifts.aggregate([
  { $match: { 
    userId: ObjectId,
    date: { $gte: startOfMonth, $lte: endOfMonth }
  }},
  { $group: { 
    _id: "$shiftType", 
    total: { $sum: "$dailySalary" },
    count: { $sum: 1 }
  }}
])
```

### Module B - Subscriptions

```json
// Tổng chi subscription hàng tháng
db.subscriptions.aggregate([
  { $match: { userId: ObjectId, isActive: true } },
  { $addFields: {
    monthlyAmount: {
      $switch: {
        branches: [
          { case: { $eq: ["$billingCycle", "weekly"] }, then: { $multiply: ["$amount", 4.33] } },
          { case: { $eq: ["$billingCycle", "yearly"] }, then: { $divide: ["$amount", 12] } }
        ],
        default: "$amount"
      }
    }
  }},
  { $group: { _id: null, total: { $sum: "$monthlyAmount" } } }
])
```

---

## NLP Parsing (Smart Ledger)

### Regex Patterns

```typescript
// Pattern tách số tiền
const amountPattern = /(\d+(?:\.\d+)?)\s*(k|tr|nghìn|triệu)?/i;

// Danh mục chi tiêu
const expenseCategories = [
  { keywords: ['ăn', 'trưa', 'sáng', 'tối', 'café', 'coffee', 'uống'], category: 'Ăn uống' },
  { keywords: ['mua', 'shopee', 'lazada', 'tiki', 'sắm'], category: 'Mua sắm' },
  { keywords: ['xăng', 'ô tô', 'xe', 'grab', 'gojek', 'be', 'đi lại'], category: 'Đi lại' },
  { keywords: ['điện', 'nước', 'wifi', 'internet', 'phone', 'mobile'], category: 'Điện/nước' },
  { keywords: ['phim', 'netflix', 'youtube', 'spotify', 'game', 'giải trí'], category: 'Giải trí' },
];

// Danh mục thu nhập
const incomeCategories = [
  { keywords: ['lương', 'tháng', 'salary'], category: 'Lương' },
  { keywords: ['thưởng', 'bonus'], category: 'Thưởng' },
  { keywords: ['bán'], category: 'Khác' },
];
```

### Examples

| Input | Output |
|-------|--------|
| "Ăn trưa 50k" | `{ type: 'expense', amount: 50000, category: 'Ăn uống' }` |
| "Mua sữa 45k" | `{ type: 'expense', amount: 45000, category: 'Mua sắm' }` |
| "Lương tháng 15tr" | `{ type: 'income', amount: 15000000, category: 'Lương' }` |
| "Thưởng Tết 5tr" | `{ type: 'income', amount: 5000000, category: 'Thưởng' }` |
| "Xăng 200k" | `{ type: 'expense', amount: 200000, category: 'Đi lại' }` |

---

## Cron Jobs

### 1. Subscription Reminder (Daily - 9:00 AM)

```typescript
// Tìm subscriptions sắp hết hạn
const subscriptions = await Subscription.find({
  userId,
  isActive: true,
  nextBillingDate: {
    $lte: new Date(Date.now() + reminderDays * 24 * 60 * 60 * 1000)
  },
  notified: false
});

// Gửi notification via SSE
// Update notified = true
```

### 2. Recurring Transactions (Daily - 1:00 AM)

```typescript
// Tìm transactions sắp chạy
const transactions = await Transaction.find({
  userId,
  isRecurring: true,
  'recurringConfig.nextRunDate': { $lte: new Date() }
});

// Với mỗi transaction:
// 1. Tạo bản copy với date = nextRunDate
// 2. Tính nextRunDate mới
// 3. Nếu nextRunDate > endDate → isRecurring = false
```

---

## Real-time Notifications (SSE)

```typescript
// Backend - Event Stream
app.get('/api/notifications/stream', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  // Add client to active connections
  const clientId = Date.now();
  clients.push({ id: clientId, res });
  
  // Cleanup on close
  req.on('close', () => {
    clients = clients.filter(c => c.id !== clientId);
  });
});

// Send notification
function sendNotification(userId, notification) {
  clients
    .filter(c => c.userId === userId)
    .forEach(c => c.res.write(`data: ${JSON.stringify(notification)}\n\n`));
}
```

---

## Indexes Summary

| Collection | Index | Purpose |
|------------|-------|---------|
| users | { pin: 1 } | Login lookup |
| work_shifts | { userId: 1, date: -1 } | Query by user + date range |
| salary_config | { userId: 1 } | Get user config |
| subscriptions | { userId: 1, isActive: 1, nextBillingDate: 1 } | Active + upcoming |
| transactions | { userId: 1, date: -1 } | Query by user + date range |
| transactions | { userId: 1, type: 1 } | Filter by type |
| categories | { userId: 1, type: 1 } | Get by type |
| audit_logs | { userId: 1, createdAt: -1 } | Recent changes |

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-03-18 | Initial design |
