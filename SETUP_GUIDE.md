# Detection System - Setup & Running Guide

## 📋 Quick Setup & Run

### For Windows Users

#### Method 1: Batch Script (Easiest)
```bash
start.bat
```

#### Method 2: PowerShell Script
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
.\start.ps1
```

#### Method 3: Manual (Terminal 1 - Backend)
```bash
cd backend
npm install  # (only if node_modules doesn't exist)
npm start
```

#### Method 3: Manual (Terminal 2 - Frontend)
```bash
cd frontend
npm install  # (only if node_modules doesn't exist)
npm run dev -- -p 3001
```

### For macOS/Linux Users

#### Method 1: Shell Script
```bash
chmod +x start.sh
./start.sh
```

#### Method 2: Manual (Terminal 1 - Backend)
```bash
cd backend
npm install  # (only if node_modules doesn't exist)
npm start
```

#### Method 2: Manual (Terminal 2 - Frontend)
```bash
cd frontend
npm install  # (only if node_modules doesn't exist)
npm run dev -- -p 3001
```

---

## ✅ Verification Checklist

After starting the servers, verify everything is working:

### 1. Backend Server
- [ ] Terminal shows: `🚀 Server running on http://localhost:3000`
- [ ] Visit: `http://localhost:3000` - Should show "Backend running 🚀"
- [ ] Visit: `http://localhost:3000/api` - Should show JSON with endpoints

### 2. Frontend Server
- [ ] Terminal shows: `✓ Ready in XXXms`
- [ ] Terminal shows: `- Local: http://localhost:3001`
- [ ] No error messages in console

### 3. API Endpoints
Test these in your browser or with curl:
- [ ] `http://localhost:3000/api` - API info
- [ ] `http://localhost:3000/api/metrics` - Dashboard metrics
- [ ] `http://localhost:3000/api/transactions` - Transactions list
- [ ] `http://localhost:3000/api/alerts` - Alerts list
- [ ] `http://localhost:3000/api/users` - User profiles

### 4. Frontend Dashboard
- [ ] Open: `http://localhost:3001`
- [ ] Dashboard loads without errors
- [ ] Tabs visible: Overview, Transactions, Alerts, Users, Fraud Chains, Risk Scoring, Settings
- [ ] Data from backend displays correctly

---

## 🔧 Troubleshooting

### Port Already in Use
**Problem**: `Error: listen EADDRINUSE: address already in use :::3000`

**Solution (Windows)**:
```powershell
# Find process on port 3000
netstat -ano | findstr :3000

# Kill process (replace XXXX with PID)
taskkill /PID XXXX /F
```

**Solution (Mac/Linux)**:
```bash
# Find process on port 3000
lsof -i :3000

# Kill process (replace XXXX with PID)
kill -9 XXXX
```

### npm: command not found
**Problem**: npm is not installed

**Solution**: Install Node.js v14+ from https://nodejs.org/

### Frontend shows error
**Problem**: Turbopack or Next.js error

**Solution**:
```bash
# Clear Next.js cache
cd frontend
rm -rf .next
npm run dev -- -p 3001
```

### Backend returns 404 for /api
**Problem**: API routes not loading

**Solution**: Restart backend server
```bash
cd backend
npm start
```

### Modules not found
**Problem**: `Cannot find module 'express'`

**Solution**: Install dependencies
```bash
cd backend
npm install

cd ../frontend
npm install
```

---

## 📚 API Documentation

### Base URL
```
http://localhost:3000/api
```

### Endpoints

#### Get Dashboard Metrics
```
GET /metrics
```
Returns: `{ totalTransactions, blockedTransactions, openAlerts, systemHealth, ... }`

#### Get All Transactions
```
GET /transactions
```
Response: Array of transaction objects with risk scores

#### Get All Alerts
```
GET /alerts
```
Response: Array of alert objects with severity levels

#### Get User Profiles
```
GET /users
```
Response: Array of user profiles with anomaly detection

#### Predict Fraud Risk
```
POST /predict
Body: { transaction data }
```
Response: Risk prediction with score and level

#### Explain Fraud Reasons
```
POST /explain-fraud
Body: { transactionData }
```
Response: List of fraud reasons with confidence

#### Update Alert Status
```
PATCH /alerts/:id
Body: { status: "resolved", ... }
```
Response: Updated alert object

#### Web Crawler
```
GET /crawl?url=https://example.com
```
Response: Crawled page data

---

## 🚀 Project Structure

```
Detection-System/
├── backend/                    # Express.js API
│   ├── server.mjs             # Main server file
│   ├── routes/
│   │   └── api.js             # API endpoints
│   ├── services/
│   │   └── mlservice.js       # ML predictions
│   ├── crawler/
│   │   └── crawler.js         # Web crawler
│   ├── package.json
│   └── .env                   # Backend config
├── frontend/                   # Next.js Dashboard
│   ├── app/
│   │   ├── page.tsx           # Main page
│   │   └── layout.tsx         # Layout
│   ├── components/
│   │   └── dashboard/         # Dashboard components
│   ├── lib/
│   │   └── api.ts             # API client
│   ├── package.json
│   └── .env.local             # Frontend config
├── anomaly-detection/         # Python ML (optional)
│   └── train.py               # ML training script
├── .env                       # Root environment
├── start.bat                  # Windows batch launcher
├── start.ps1                  # PowerShell launcher
├── start.sh                   # Unix shell launcher
├── DEPENDENCIES.md            # All dependencies list
├── SETUP_GUIDE.md            # This file
└── README.md                 # Project overview
```

---

## 📦 Environment Variables

### Backend (.env or backend/.env)
```env
PORT=3000
HOST=localhost
NODE_ENV=development
CORS_ORIGIN=http://localhost:3001
ML_SERVICE_URL=http://localhost:5000
ML_SERVICE_ENABLED=false
CRAWLEE_STORAGE_DIR=./storage
CRAWLEE_HEADLESS=true
```

### Frontend (.env.local or frontend/.env.local)
```env
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_ANALYTICS_ENABLED=true
NEXT_PUBLIC_APP_NAME=Detection System Dashboard
NEXT_PUBLIC_APP_VERSION=1.0.0
```

---

## 🔄 Development Workflow

### Install Latest Dependencies
```bash
# Backend
cd backend
npm install --latest

# Frontend
cd ../frontend
npm install --latest
```

### Update Dependencies
```bash
# Check for updates
npm outdated

# Update all packages
npm update
```

### Development Mode with Auto-Reload

**Backend** (with nodemon):
```bash
cd backend
npm run dev  # if configured in package.json
```

**Frontend** (automatic with Next.js):
```bash
cd frontend
npm run dev
```

---

## 🚢 Deployment Checklist

Before pushing to main:

- [ ] Both servers run without errors
- [ ] All API endpoints respond with 200 status
- [ ] Frontend loads without console errors
- [ ] Dashboard displays real data from backend
- [ ] Dependencies installed and working
- [ ] Environment variables configured
- [ ] .env files added to .gitignore
- [ ] No console warnings or errors
- [ ] Git status is clean

---

## 📝 Commands Reference

### Backend Commands
```bash
npm start          # Start production server
npm run dev        # Start with nodemon (if configured)
npm install        # Install dependencies
npm list          # List installed packages
```

### Frontend Commands
```bash
npm run dev        # Start development server
npm run build      # Build for production
npm start          # Start production server
npm run lint       # Run ESLint
```

### Git Commands
```bash
git status        # Check status
git add .         # Stage all changes
git commit -m "message"  # Commit changes
git push          # Push to main
git pull          # Pull latest changes
```

---

## 🆘 Getting Help

If you encounter issues:

1. Check the Troubleshooting section above
2. Review error messages in terminal carefully
3. Check if ports 3000 and 3001 are free
4. Ensure Node.js v14+ is installed
5. Clear node_modules and reinstall:
   ```bash
   rm -rf node_modules package-lock.json
   npm install
   ```
6. Check DEPENDENCIES.md for version conflicts

---

## ✅ Final Checklist Before Push

- [ ] `start.bat` works on Windows
- [ ] `start.ps1` works on Windows
- [ ] `start.sh` works on macOS/Linux
- [ ] Backend starts without errors
- [ ] Frontend starts without errors  
- [ ] All API endpoints respond
- [ ] Dashboard loads and displays data
- [ ] .env files are in .gitignore
- [ ] node_modules is in .gitignore
- [ ] README.md is up to date
- [ ] DEPENDENCIES.md is accurate
- [ ] All files are committed
- [ ] Ready for production deployment

---

Happy coding! 🚀
