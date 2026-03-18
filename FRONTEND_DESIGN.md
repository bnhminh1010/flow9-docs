# Flow9 Frontend Design System

> **Reference**: Design from `docs/design_test_2/` - Actual App UI

---

## 1. Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Icons**: Material Symbols Outlined (Google Fonts)
- **Charts**: Recharts
- **State**: React Context + useState/useReducer
- **Font**: Inter

---

## 2. Color Palette

### Primary Colors
```css
--color-primary: #0D9488;        /* Teal - Main brand */
--color-primary-light: #0d968b;  /* Teal - Same as primary */
```

### Background Colors
```css
--color-bg-primary: #09090B;     /* Zinc 950 - Main background */
--color-bg-secondary: #18181B;    /* Zinc 900 - Cards */
--color-bg-tertiary: #27272A;    /* Zinc 800 - Inputs */
```

### Border Colors
```css
--color-border: #3F3F46;         /* Zinc 700 */
```

### Text Colors
```css
--color-text-primary: #FAFAFA;   /* Zinc 50 */
--color-text-secondary: #A1A1AA;  /* Zinc 400 */
--color-text-muted: #71717A;     /* Zinc 500 */
```

### Accent Colors
```css
--color-success: #22C55E;
--color-warning: #F59E0B;        /* Amber - Subscriptions due */
--color-danger: #EF4444;
--color-info: #3B82F6;
```

---

## 3. Layout Structure

### 3.1 Main App Layout
```
┌──────────────────────────────────────────────────────────┐
│ Sidebar (240px fixed)    │  Main Content Area           │
│ ┌──────────────────────┐ │ ┌──────────────────────────┐│
│ │ Logo + Brand         │ │ │ Header (64px)            ││
│ ├──────────────────────┤ │ │ Title + [Notif] [+Add]   ││
│ │ Navigation           │ │ ├──────────────────────────┤│
│ │ - Dashboard (active) │ │ │                          ││
│ │ - Ledger             │ │ │ Page Content             ││
│ │ - Subscriptions      │ │ │ (Cards, Charts, Lists)   ││
│ │ - Payroll            │ │ │                          ││
│ │ - Settings           │ │ │                          ││
│ ├──────────────────────┤ │ │                          ││
│ │ User Info           │ │ │                          ││
│ └──────────────────────┘ │ └──────────────────────────┘│
└──────────────────────────────────────────────────────────┘
```

### 3.2 Sidebar Design
- Width: 240px
- Background: `bg-zinc-950`
- Border right: `border-zinc-700`
- Logo: Icon (48px) + Brand name
- Nav items: Icon + Label, hover states
- Active item: `bg-primary/10 text-primary`
- User section at bottom: Avatar + Name + Plan

### 3.3 Header Design
- Height: 64px
- Background: `bg-zinc-950/80 backdrop-blur-md`
- Border bottom: `border-zinc-700`
- Content: Page title (left) + Actions (right)
- Actions: Notification icon + Add Transaction button

---

## 4. Page Designs

### 4.1 Dashboard Page

```
┌──────────────────────────────────────────────────────────┐
│ Dashboard Overview                        March 2026     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────┐ ┌────────────────┐ ┌──────────────┐  │
│  │ Lương          │ │ Chi tiêu      │ │ Subs         │  │
│  │ 15,000,000 VND │ │ 4,250,000 VND │ │ 890,000 VND │  │
│  │ ▲ +12%         │ │ ▼ -5%         │ │ 3 due soon   │  │
│  └────────────────┘ └────────────────┘ └──────────────┘  │
│                                                          │
│  ┌─────────────────────────┐ ┌───────────────────────┐  │
│  │ 7-day Trend             │ │ Top Categories        │  │
│  │ [Line Chart]            │ │ [Pie Chart]           │  │
│  │                         │ │ Food 65%              │  │
│  │                         │ │ Rent 20%              │  │
│  │                         │ │ Others 15%            │  │
│  └─────────────────────────┘ └───────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────────┐│
│  │ Recent Transactions                    View All →   ││
│  │ ────────────────────────────────────────────────────││
│  │ 🍔 Ăn trưa         -50,000   12:30 Today          ││
│  │ 💼 Lương tháng   +15,000,000 09:00 Today          ││
│  │ 🛒 Mua sữa        -45,000   18:20 Yesterday       ││
│  └──────────────────────────────────────────────────────┘│
│                                                          │
└──────────────────────────────────────────────────────────┘
```

#### Metric Card Design
```tsx
<div class="p-6 rounded-xl bg-zinc-900 border border-zinc-700">
  <div class="flex justify-between items-start mb-4">
    <span class="text-slate-400 text-sm font-medium">Label</span>
    <div class="p-2 rounded-lg bg-{color}/10">
      <Icon className="text-{color}" />
    </div>
  </div>
  <div class="space-y-1">
    <h3 class="text-2xl font-bold">{value}</h3>
    <div class="flex items-center gap-1 text-{color} text-sm font-medium">
      <Icon arrow />
      <span>{percentage}%</span>
      <span className="text-slate-500 font-normal">vs last month</span>
    </div>
  </div>
</div>
```

#### Transaction Row Design
```tsx
<div class="p-4 flex items-center justify-between hover:bg-zinc-800/50">
  <div class="flex items-center gap-4">
    <div class="size-10 rounded-lg bg-zinc-800 flex items-center justify-center">
      <Icon className="text-slate-400" />
    </div>
    <div>
      <p class="text-sm font-bold">Description</p>
      <p class="text-[10px] text-slate-500">Time • Category</p>
    </div>
  </div>
  <div class="text-right">
    <p class="text-sm font-bold text-{color}">{amount} VND</p>
    <p class="text-[10px] text-slate-500">Method</p>
  </div>
</div>
```

---

### 4.2 Ledger Page (with Quick Input)

```
┌──────────────────────────────────────────────────────────┐
│ Ledger                                            [≡]   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────────────────────────────────────────┐│
│  │ [+ Add Transaction]              [Search] [Filter] ││
│  └─────────────────────────────────────────────────────┘│
│                                                          │
│  ┌─────────────────────────────────────────────────────┐│
│  │ 💳 Quick: "Ăn trưa 50k"               [Enter]     ││
│  │                                                     ││
│  │ Suggestions:                                        ││
│  │ "Mua [item] [price]" → Mua sắm                    ││
│  │ "[food] [price]" → Ăn uống                        ││
│  └─────────────────────────────────────────────────────┘│
│                                                          │
│  Today - Tuesday, 18/3                 Total: -150,000 │
│  ─────────────────────────────────────────────────────  │
│  🍔 Ăn trưa                      -50,000   12:30      │
│  🛒 Mua sữa                       -45,000   08:00      │
│  💼 Lương tháng                +15,000,000  09:00      │
│                                                          │
│  Yesterday - Monday, 17/3              Total: -890,000 │
│  ─────────────────────────────────────────────────────  │
│  🚗 Xăng                          -200,000  17:00      │
│  💡 Điện                          -150,000  15:00      │
│  🎬 Netflix                        -79,000   14:00      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### 4.3 Subscriptions Page

```
┌──────────────────────────────────────────────────────────┐
│ Subscriptions                                  [+ Add]  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────────────────────────────────────────┐ │
│  │ [Search...]           [Grid ▦] [List ☰]          │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
│  Total monthly: 890,000 VND                             │
│                                                          │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐          │
│  │ Netflix    │ │ Spotify    │ │ YouTube    │          │
│  │ ██████░░   │ │ █████░░    │ │ ██████░░   │          │
│  │ 79,000/mo  │ │ 29,000/mo  │ │ 79,000/mo  │          │
│  │ 2 days     │ │ 5 days     │ │ 8 days     │          │
│  └────────────┘ └────────────┘ └────────────┘          │
│                                                          │
│  █████░░ = 10-segment countdown indicator               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

#### Subscription Card
```tsx
<div class="p-4 rounded-xl bg-zinc-900 border border-zinc-700">
  <div class="flex justify-between items-start">
    <Icon className="text-2xl" />
    <button class="text-slate-400">⋮</button>
  </div>
  <div class="mt-4">
    <div class="flex gap-1 mb-2">
      {/* 10 segments */}
      {[...Array(10)].map((_, i) => (
        <div className={`h-1 flex-1 rounded ${i < 7 ? 'bg-primary' : 'bg-zinc-700'}`} />
      ))}
    </div>
    <p className="text-xs text-slate-500">{daysLeft} days</p>
  </div>
  <div className="mt-4">
    <p className="font-bold">{amount}/mo</p>
  </div>
</div>
```

---

### 4.4 Payroll Page

```
┌──────────────────────────────────────────────────────────┐
│ Payroll                                         [+ Add] │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Tháng 3/2026                         Tổng: 15,000,000  │
│                                                          │
│  ┌──────────────────────────┐ ┌───────────────────────┐│
│  │     Calendar            │ │ Chi tiết ca làm       ││
│  │ [Mar 2026]             │ │                       ││
│  │ T2 T3 T4 T5 T6 T7 CN  │ │ Ngày: 18/3            ││
│  │ ●  ●  ●  ●  ●  ●  ●   │ │ Ca: day (8h)          ││
│  │ ●  ●  ●  ○  ●  ●  ●   │ │ Lương: 600,000       ││
│  │ ●  ●  ●  ●  ●  ○  ●   │ │ [Edit] [Delete]       ││
│  │                        │ │                       ││
│  │ ● = có làm            │ │ ───────────────────── ││
│  │ ○ = nghỉ              │ │ Tổng:                ││
│  └──────────────────────────┘ │ • Day: 20 ca         ││
│                              │ • Night: 5 ca        ││
│                              │ • Holiday: 1 ca      ││
│                              └───────────────────────┘│
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

### 4.5 Login Page (PIN)

```
┌────────────────────────────────────────┐
│                                        │
│         [Wallet Icon]                  │
│         Flow9 Finance                 │
│                                        │
│     ┌───┬───┬───┐                     │
│     │ 1 │ 2 │ 3 │                     │
│     ├───┼───┼───┤                     │
│     │ 4 │ 5 │ 6 │                     │
│     ├───┼───┼───┤                     │
│     │ 7 │ 8 │ 9 │                     │
│     ├───┼───┼───┤                     │
│     │   │ 0 │   │                     │
│     └───┴───┴───┘                     │
│                                        │
│     ● ○ ○ ○ ○ ○                       │
│                                        │
│     [Forgot PIN?]                      │
│                                        │
└────────────────────────────────────────┘
```

---

## 5. Components Specification

### 5.1 Button Variants

```tsx
// Primary
<button class="bg-primary hover:bg-primary/90 text-white px-4 py-2 rounded-lg text-sm font-medium">
  Add Transaction
</button>

// Ghost
<button class="text-slate-400 hover:text-slate-100 px-3 py-2">
  View All
</button>

// Icon
<button class="p-2 text-slate-400 hover:text-slate-100">
  <Icon />
</button>
```

### 5.2 Card Component

```tsx
<div class="p-6 rounded-xl bg-zinc-900 border border-zinc-700">
  {/* Content */}
</div>
```

### 5.3 Input Component

```tsx
<input 
  class="bg-zinc-800 border-zinc-700 rounded-lg px-4 py-2 outline-none focus:border-primary"
  placeholder="Search..."
/>
```

### 5.4 Chart Containers

```tsx
// Line Chart
<div class="p-6 rounded-xl bg-zinc-900 border border-zinc-700">
  <h4 class="font-bold mb-6">Title</h4>
  <div class="h-64">
    <Recharts.LineChart ... />
  </div>
</div>

// Pie Chart
<div class="p-6 rounded-xl bg-zinc-900 border border-zinc-700">
  <h4 class="font-bold mb-6">Title</h4>
  <div class="flex items-center justify-center h-48">
    <Recharts.PieChart ... />
  </div>
</div>
```

---

## 6. Responsive Behavior

| Breakpoint | Sidebar | Content |
|------------|---------|---------|
| Desktop (>1024px) | Fixed 240px | Full |
| Tablet (768-1024px) | Collapsed (icons only) | 2 columns |
| Mobile (<768px) | Hidden (hamburger) | Full, stacked |

---

## 7. File Structure

```
src/
├── app/
│   ├── (auth)/
│   │   └── login/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx       # Sidebar + Header
│   │   ├── page.tsx         # Dashboard
│   │   ├── ledger/
│   │   │   └── page.tsx
│   │   ├── subscriptions/
│   │   │   └── page.tsx
│   │   ├── payroll/
│   │   │   └── page.tsx
│   │   └── settings/
│   │       └── page.tsx
│   └── api/
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Input.tsx
│   │   └── Modal.tsx
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   └── Header.tsx
│   └── modules/
│       ├── MetricCard.tsx
│       ├── TransactionRow.tsx
│       ├── SubscriptionCard.tsx
│       └── Charts/
└── lib/
```

---

## 8. Implementation Priority

1. **Setup**: Tailwind config + Layout (Sidebar + Header)
2. **Dashboard**: Metric cards + Charts + Transaction list
3. **Ledger**: Quick input + Transaction list
4. **Subscriptions**: Grid view + Card + Countdown
5. **Payroll**: Calendar + Shift management
6. **Login**: PIN input
7. **Settings**: Config forms

---

## 9. Key Design Decisions

- **Dark mode only**: `#09090B` background
- **Material Symbols icons**: Consistent, familiar
- **10-segment countdown**: Visual billing indicator
- **Quick Input**: Text field + NLP parsing (not just button)
- **Card-based layout**: Clear information hierarchy
- **Trend indicators**: Show vs previous month
