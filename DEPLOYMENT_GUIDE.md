# Flow9 Deployment Guide

## Overview

This guide covers deploying Flow9 to production with:
- **Frontend**: Vercel
- **Backend**: Railway (recommended) or Render
- **Database**: MongoDB Atlas (already configured)

---

## Prerequisites

1. MongoDB Atlas account with a cluster
2. GitHub account
3. Vercel account
4. Railway account (for backend)

---

## Step 1: Update Environment Files

### Backend `.env`

```env
PORT=3001
MONGODB_URI=mongodb+srv://YOUR_USER:YOUR_PASSWORD@YOUR_CLUSTER.mongodb.net/flow9
JWT_SECRET=super-secure-random-string-change-this
FRONTEND_URL=https://flow9-frontend.vercel.app
```

### Frontend `.env.local` (create this file)

```env
NEXT_PUBLIC_API_URL=https://flow9-backend.railway.app
```

---

## Step 2: Deploy Backend to Railway

### Option A: Railway (Recommended)

1. **Go to** [railway.app](https://railway.app)

2. **Login** with GitHub

3. **New Project** → **Deploy from GitHub repo**

4. **Select** your `flow9-backend` repo

5. **Configure Environment Variables**:
   - Click "Add Variable"
   - Add each variable from `.env`:
     - `MONGODB_URI`
     - `JWT_SECRET`
     - `FRONTEND_URL`
     - `PORT=3001`

6. **Wait for deployment** to complete

7. **Get your backend URL**:
   - In project settings → Domains
   - Format: `https://flow9-backend.up.railway.app`

### Option B: Render

1. Go to [render.com](https://render.com)

2. **New** → **Web Service**

3. Connect GitHub repo

4. Configure:
   - **Root Directory**: (leave empty)
   - **Build Command**: `npm install && npm run build`
   - **Start Command**: `npm start`

5. **Add Environment Variables** from `.env`

6. **Deploy**

---

## Step 3: Update Frontend

Update `flow9-frontend/.env.local`:

```env
NEXT_PUBLIC_API_URL=https://flow9-backend.railway.app
```

---

## Step 4: Deploy Frontend to Vercel

### Option A: Vercel CLI

```bash
cd flow9-frontend

# Install Vercel
npm i -g vercel

# Deploy
vercel

# Follow prompts:
# - Set up and deploy? Y
# - Which scope? Your account
# - Link to existing project? N
# - Project name? flow9
# - Directory? ./
# - Override settings? N

# For production
vercel --prod
```

### Option B: Vercel Dashboard

1. Go to [vercel.com](https://vercel.com)

2. **Add New Project**

3. **Import** `flow9-frontend` repo

4. **Framework Preset**: Next.js

5. **Environment Variables**:
   - `NEXT_PUBLIC_API_URL` = `https://flow9-backend.railway.app`

6. **Deploy**

---

## Step 5: Configure CORS on Backend

Make sure your backend `.env` includes:

```env
FRONTEND_URL=https://flow9-frontend.vercel.app
```

The backend CORS is already configured to allow this origin.

---

## Step 6: Verify Deployment

1. Open your Vercel URL (e.g., `https://flow9-frontend.vercel.app`)

2. Login with your 6-digit PIN

3. Test:
   - Add a transaction in Ledger
   - Check Dashboard updates
   - Add a subscription

---

## Troubleshooting

### CORS Error
- Make sure `FRONTEND_URL` in backend `.env` matches your Vercel URL exactly
- Include `https://` prefix

### API Not Found
- Verify `NEXT_PUBLIC_API_URL` in Vercel environment variables
- Check backend is running on Railway

### Database Connection
- Verify `MONGODB_URI` is correct
- Check MongoDB Atlas network whitelist allows Railway/Vercel IPs

---

## Useful Commands

### Railway
```bash
# View logs
railway logs

# Open shell
railway run bash

# Restart
railway up
```

### Vercel
```bash
# View logs
vercel logs

# Redeploy
vercel --prod
```

---

## Cost Summary

| Service | Free Tier | Cost |
|---------|-----------|------|
| Vercel | 100GB bandwidth | Free |
| Railway | 500 hours/month | Free |
| MongoDB Atlas | 512MB | Free |

**Total: FREE** (for small personal use)
