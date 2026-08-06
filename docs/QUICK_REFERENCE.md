# QUICK REFERENCE CARD

## 🚀 Start the System

**Windows (easiest)**
```bash
scripts\start.bat
```

**Any OS**
```bash
./scripts/start.ps1  # PowerShell
./scripts/start.sh   # macOS/Linux
```

**Manual**
```bash
# Terminal 1
cd backend && npm start

# Terminal 2
cd frontend && npm run dev -- -p 3001
```

---

## 🌐 Access Points

| Component | URL | Status |
|-----------|-----|--------|
| Dashboard | http://localhost:3001 | ✅ Running |
| Backend API | http://localhost:3000 | ✅ Running |
| API Docs | http://localhost:3000/api | ✅ Running |

---

## 📡 API Endpoints

```
GET  /api              → API info
GET  /api/metrics      → Dashboard metrics
GET  /api/transactions → All transactions
GET  /api/alerts       → All alerts  
GET  /api/users        → User profiles
POST /api/predict      → Fraud prediction
POST /api/explain-fraud → Fraud explanation
```

---

## 🛠 Common Commands

```bash
# Install dependencies
cd backend && npm install
cd ../frontend && npm install

# Start backend
cd backend && npm start

# Start frontend  
cd frontend && npm run dev

# Build for production
cd frontend && npm run build

# Check Node.js
node --version

# Kill process on port 3000
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

---

## 📁 Key Files

```
backend/server.mjs         ← Main server entry
backend/routes/api.js      ← API endpoints
frontend/lib/api.ts        ← API client
frontend/app/page.tsx      ← Dashboard
.env                       ← Environment config
scripts/start.*           ← Startup scripts
```

---

## ✅ Verification

- Backend running? `http://localhost:3000` → "Backend running 🚀"
- Frontend running? `http://localhost:3001` → Dashboard loads
- API working? `http://localhost:3000/api/metrics` → Returns JSON

---

## 🆘 Troubleshooting

| Problem | Solution |
|---------|----------|
| Port in use | `scripts/start.bat` auto-cleans ports |
| npm not found | Install Node.js v14+ |
| Module error | `npm install` in backend/frontend |
| API 404 | Restart backend server |

---

## 📦 Dependencies

**Backend**: express, cors, crawlee, axios
**Frontend**: react, next, tailwind, recharts, radix-ui
**Both**: TypeScript, Node.js 14+

---

## 🚢 Ready to Deploy!

✅ Both servers running
✅ All endpoints responding  
✅ Frontend loading data
✅ No errors in console
✅ Ready for `git push origin main`

---
