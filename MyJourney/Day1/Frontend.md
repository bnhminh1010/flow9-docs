# Frontend Progress - Day 1

## Overview

Built the complete Next.js 14 frontend for Flow9 Personal Life OS with full API integration.

## Tech Stack

- **Framework**: Next.js 14 (App Router)
- **Styling**: Tailwind CSS with dark theme (Zinc)
- **Language**: TypeScript
- **Icons**: Lucide React

## Project Structure

```
flow9-frontend/
├── src/
│   ├── app/
│   │   ├── layout.tsx           # Root layout with AuthProvider
│   │   ├── page.tsx            # Redirects to /login or /dashboard
│   │   ├── login/page.tsx      # PIN login (6 digits)
│   │   └── dashboard/
│   │       ├── layout.tsx       # Sidebar navigation
│   │       ├── page.tsx        # Dashboard overview
│   │       ├── ledger/page.tsx # Quick input + transactions
│   │       ├── payroll/page.tsx # Calendar + shifts
│   │       ├── subscriptions/page.tsx
│   │       └── settings/page.tsx
│   ├── components/
│   │   └── CategoryIcon.tsx    # Brand logos & category icons
│   ├── lib/api.ts              # API client
│   ├── hooks/useAuth.tsx       # Auth context
│   ├── types/index.ts          # TypeScript types
│   └── app/globals.css         # Tailwind + custom styles
├── .env.local
├── package.json
├── README.md
└── tsconfig.json
```

## Features

### 1. Login Page (`/login`)
- 6-digit PIN keypad
- Auto-login when PIN complete
- First-time creates admin account

### 2. Dashboard (`/dashboard`)
- Monthly stats: Payroll, Income, Expense, Net Flow
- Savings rate with progress bar
- Expense breakdown by category
- 7-day activity chart
- Comparison with last month
- Subscriptions alert
- Auto-refresh on focus

### 3. Ledger (`/dashboard/ledger`)
- NLP quick input: `"Ăn trưa 50k"` or `"+ Lương 15tr"`
- Filter by type: All / + Income / - Expense
- Filter by category
- Search transactions
- Edit/Delete transactions
- Stats: Income, Expense, Net Flow
- Payment history

### 4. Payroll (`/dashboard/payroll`)
- Calendar view of shifts
- Add/Edit/Delete shifts
- Shift types: Day, Night, Holiday
- Salary calculation by shift type
- Monthly summary

### 5. Subscriptions (`/dashboard/subscriptions`)
- 100+ brand logos (Netflix, Spotify, YouTube, ChatGPT, etc.)
- Quick link to service website
- Payment history tracking
- Mark as paid (auto-update next billing date)
- Monthly/Yearly totals
- Stats: Monthly Burn, Yearly Burn, Total Paid

### 6. Settings (`/dashboard/settings`)
- Change PIN (6 digits)
- Export data to CSV
- Import data from CSV
- System information

## Brand Logos

Supports 100+ brands:
- **Streaming**: Netflix, Spotify, YouTube, Disney+, HBO, etc.
- **Gaming**: Xbox, PlayStation, Steam, Nintendo, EA Play
- **Cloud**: Google Drive, iCloud, Dropbox, OneDrive
- **AI**: ChatGPT, Claude, Gemini, Midjourney
- **Software**: Adobe, Figma, Notion, Slack, Zoom
- **Education**: Udemy, Coursera, MasterClass
- **Hosting**: AWS, Vercel, Netlify, DigitalOcean
- **Shopping**: Amazon, Shopee, Lazada

## API Integration

- Centralized API client in `/src/lib/api.ts`
- JWT via httpOnly cookies
- Type-safe API responses
- Credentials included in requests

## Environment Variables

```env
NEXT_PUBLIC_API_URL=http://localhost:3001
```

For production:
```env
NEXT_PUBLIC_API_URL=https://your-backend.railway.app
```

## Commands

```bash
cd flow9-frontend
npm install      # Install dependencies
npm run dev      # Start development server on port 3000
npm run build    # Build production
npm start        # Run production
```

## Deploy on Vercel

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Or use Vercel dashboard
# 1. Import GitHub repo
# 2. Set NEXT_PUBLIC_API_URL
# 3. Deploy
```

## Notes

- First-time login creates admin user with the PIN entered
- Frontend runs on port 3000, backend on port 3001
- PIN is 6 digits (changed from 4-6)
- Auto-refresh dashboard when tab is focused
