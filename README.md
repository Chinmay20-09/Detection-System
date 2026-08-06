# Detection System - Fraud Detection Dashboard

A comprehensive full-stack fraud detection and anomaly detection system with a modern dashboard, backend API, and machine learning integration.

## 🌐 System Overview

- **Frontend**: Next.js 16.2.0 with React 19, TypeScript, and Tailwind CSS
- **Backend**: Express.js with ES modules and CORS support
- **ML/AI**: Python-based anomaly detection with SHAP explanations
- **Real-time**: Live data fetching with React hooks and API integration

## 📋 Features

### Dashboard
- **Overview Tab**: Real-time metrics, system health, and alert summary
- **Transactions Tab**: Transaction monitoring with risk scoring and filtering
- **Alerts Tab**: Alert management with severity levels and status tracking
- **User Profiles Tab**: User risk profiling and behavioral anomaly detection
- **Fraud Detection**: Automated fraud prediction with explainable reasons

### Backend API
- RESTful endpoints for transactions, alerts, users, and metrics
- Fraud prediction with risk scoring
- Fraud explanation with SHAP-based reasoning
- Web crawler integration (Crawlee)
- CORS-enabled for cross-origin requests

### Features
- Real-time data synchronization
- Fraud explanation dashboard
- Risk scoring algorithms
- Behavioral anomaly detection
- Transaction filtering and search
- Alert management
- User profile monitoring

## 🚀 Quick Start

### Prerequisites
- Node.js v14+ (tested with v18+)
- npm v6+
- Python 3.8+ (for ML components)
- Virtual environment (venv/virtualenv)

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/Komal-Sinha-tech/Detection-System.git
cd Detection-System
```

2. **Setup Python Virtual Environment**
```bash
# Create virtual environment
python -m venv .venv

# Activate virtual environment (Windows)
.\.venv\Scripts\Activate.ps1

# Activate virtual environment (macOS/Linux)
source .venv/bin/activate

# Install ML dependencies
pip install -r requirements.txt
```

3. **Setup Backend**
```bash
cd backend
npm install
cd ..
```

4. **Setup Frontend**
```bash
cd frontend
npm install
cd ..
```

## 📦 Installed Dependencies

### Python ML Stack ✅
- numpy (2.4.4) - Numerical computing
- pandas (3.0.2) - Data manipulation
- scikit-learn (1.8.0) - Machine learning algorithms
- shap (0.51.0) - Model explanations
- xgboost (3.2.0) - Gradient boosting
- matplotlib (3.10.8) - Data visualization

### Backend Stack ✅
- Express.js (4.22.1) - Web framework
- Crawlee (3.16.0) - Web scraping
- CORS (2.8.6) - Cross-origin support
- Axios (1.14.0) - HTTP client

### Frontend Stack ✅
- Next.js (16.2.0) - React framework
- React (19.2.4) - UI library
- TypeScript (5.7.3) - Type safety
- Tailwind CSS (4.2.2) - Styling
- Radix UI - Component library
- Recharts (2.15.0) - Data visualization
- React Hook Form - Form management
- Zod - Schema validation

## 🏃 Running the System

### Start Backend Server
```bash
cd backend
npm start          # Production
# OR
npm run dev        # Development with auto-reload (requires nodemon)
```
Backend runs on: http://localhost:5000

### Start Frontend Development Server
```bash
cd frontend
npm run dev
```
Frontend runs on: http://localhost:3000

### Train ML Model (Optional)
```bash
# Activate virtual environment first
.\.venv\Scripts\Activate.ps1

# Train anomaly detection model
cd anomaly-detection
python train.py
```

### Use ML Model for Predictions
```bash
# Activate virtual environment first
.\.venv\Scripts\Activate.ps1

# Run predictions
cd anomaly-detection
python predict.py
```

## 📊 System Architecture

### Frontend (Next.js)
- Dashboard with real-time data
- Multi-tab interface (Overview, Transactions, Alerts, Users, Risk Scoring)
- Interactive charts and data tables
- Alert management system

### Backend (Express.js)
- RESTful API endpoints
- Fraud prediction engine
- SHAP-based fraud explanations
- Web crawler integration
- Real-time data synchronization

### ML Engine (Python)
- Anomaly detection models
- Fraud classification
- SHAP explainability
- Risk scoring algorithms

## 🔗 API Endpoints

### Transactions
- `GET /api/transactions` - Get all transactions
- `POST /api/fraud-prediction` - Predict fraud with explanations

### Alerts
- `GET /api/alerts` - Get all alerts
- `PUT /api/alerts/:id` - Update alert status

### Users
- `GET /api/users` - Get user profiles
- `GET /api/users/:id` - Get user details

### Metrics
- `GET /api/metrics` - Get dashboard metrics

## 🧪 Verification Checklist

✅ **Backend Dependencies**: All packages installed
✅ **Frontend Dependencies**: All packages installed  
✅ **Python Environment**: ML packages installed
- scikit-learn, pandas, numpy, shap, xgboost, matplotlib

✅ **Models**: Pre-trained model available at `anomaly-detection/model.pkl`

2. **Install dependencies**

**Backend:**
```bash
cd backend
npm install
```

**Frontend:**
```bash
cd frontend
npm install
```

**Python (Optional):**
```bash
cd anomaly-detection
pip install -r requirements.txt
```

3. **Configure environment variables**

Copy `.env.example` to `.env` and update values:
```bash
cp .env.example .env
```

See `.env` file for all available configuration options.

### Running the System

**Terminal 1 - Backend Server:**
```bash
cd backend
npm start
# Runs on http://localhost:3000
```

**Terminal 2 - Frontend Development Server:**
```bash
cd frontend
npm run dev
# Runs on http://localhost:3001
```

**Terminal 3 - ML Service (Optional):**
```bash
cd anomaly-detection
python train.py
# Runs on http://localhost:5000
```

Access the dashboard at **http://localhost:3001**

## 📁 Project Structure

```
Detection-System/
├── frontend/              # Next.js frontend application
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   │   ├── dashboard/    # Dashboard components
│   │   ├── ui/          # UI components (Radix)
│   │   └── theme-provider.tsx
│   ├── lib/              # Utilities and API client
│   └── package.json
│
├── backend/               # Express.js backend server
│   ├── routes/           # API routes
│   ├── services/         # Business logic
│   ├── crawler/          # Web crawler
│   ├── storage/          # Data storage
│   ├── server.mjs        # Express server
│   └── package.json
│
├── anomaly-detection/     # Python ML module
│   └── train.py          # Training script
│
├── .env                   # Environment configuration
├── .env.example           # Example env file
├── docs/                  # Documentation (setup guide, dependencies, checklists)
└── README.md             # This file
```

## 🔌 API Endpoints

All endpoints are under `/api/:

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | API info and available endpoints |
| GET | `/transactions` | Get all transactions |
| GET | `/transactions/:id` | Get specific transaction |
| GET | `/alerts` | Get all alerts |
| GET | `/alerts/:id` | Get specific alert |
| PATCH | `/alerts/:id` | Update alert status |
| GET | `/users` | Get user profiles |
| GET | `/users/:id` | Get specific user |
| GET | `/metrics` | Get dashboard metrics |
| GET | `/crawl?url=<URL>` | Crawl a webpage |
| POST | `/predict` | Predict fraud for transaction |
| POST | `/explain-fraud` | Get fraud explanation |

## 🛠️ Environment Variables

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

### Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_ANALYTICS_ENABLED=true
NEXT_PUBLIC_APP_NAME=Detection System Dashboard
NEXT_PUBLIC_APP_VERSION=1.0.0
```

## 📦 Dependencies

### Frontend
- **UI Framework**: React 19, Next.js 16.2.0
- **Components**: Radix UI (25+ components)
- **Styling**: Tailwind CSS 4.2.0
- **Forms**: React Hook Form with Zod validation
- **Charts**: Recharts for data visualization
- **Icons**: Lucide React
- **Notifications**: Sonner
- **Utilities**: Date-fns, clsx, tailwind-merge

### Backend
- **Framework**: Express.js 4.19.2
- **Web Scraping**: Crawlee 3.16.0
- **HTTP Client**: Axios 1.14.0
- **CORS**: cors 2.8.5

See [docs/DEPENDENCIES.md](docs/DEPENDENCIES.md) for complete list.

## 🔐 Security Features

- CORS protection
- Environment variable isolation
- Type-safe API interfaces (TypeScript)
- Secure form validation (Zod)
- Server-side API routes

## 🧪 Testing

To test the API manually:

```bash
# Get API info
curl http://localhost:3000/api

# Get metrics
curl http://localhost:3000/api/metrics

# Get transactions
curl http://localhost:3000/api/transactions

# Get alerts
curl http://localhost:3000/api/alerts
```

## 📊 Data Models

### Transaction
```json
{
  "id": "TXN-001",
  "userId": "USR-4521",
  "amount": 12500,
  "merchant": "Wire Transfer",
  "category": "Transfer",
  "riskScore": 92,
  "riskLevel": "critical",
  "status": "blocked",
  "timestamp": "2024-01-15 14:32:18",
  "location": "Lagos, Nigeria",
  "device": "Unknown Device",
  "factors": ["New location", "Large amount", ...]
}
```

### Alert
```json
{
  "id": "ALT-001",
  "title": "Suspicious Wire Transfer",
  "description": "Large international wire transfer...",
  "severity": "critical",
  "status": "open",
  "userId": "USR-4521",
  "transactionId": "TXN-001",
  "timestamp": "2024-01-15 14:32:18"
}
```

### User Profile
```json
{
  "id": "USR-4521",
  "name": "John Doe",
  "email": "john@example.com",
  "riskScore": 75,
  "location": "New York, NY",
  "accountAge": "2 years",
  "transactionCount": 234,
  "avgTransaction": 450,
  "anomalyCount": 2,
  "status": "normal"
}
```

## 🚀 Deployment

### Frontend (Vercel)
```bash
npm run build
npm start
```

### Backend (Docker/Railway/Heroku)
```bash
# Dockerfile example
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

## 📝 License

MIT License - feel free to use for personal or commercial projects.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📞 Support

For issues or questions:
- Open an issue on GitHub
- Check [docs/DEPENDENCIES.md](docs/DEPENDENCIES.md) for detailed setup guide

## 👨‍💻 Author

**Komal Sinha**
- GitHub: [@Komal-Sinha-tech](https://github.com/Komal-Sinha-tech)

## 🙏 Acknowledgments

- Radix UI for component system
- Next.js for React framework
- Tailwind CSS for styling
- Crawlee for web scraping
- SHAP for model explainability

## ⚙️ Troubleshooting

### Backend Issues

**Error: Port 3000 already in use**
```bash
# Change port in .env file
PORT=3001

# Or kill the process using port 3000
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows
```

**Error: CORS issues**
- Ensure `CORS_ORIGIN` in .env matches your frontend URL
- Check frontend `.env.local` for correct `NEXT_PUBLIC_API_URL`

**Error: ML service not connecting**
- Make sure Python ML service is running (separate terminal)
- Check `ML_SERVICE_URL` in backend .env

### Frontend Issues

**Error: Module not found**
```bash
# Clear cache and reinstall
rm -rf node_modules .next
npm install
```

**Error: Port 3000 already in use**
```bash
# The dev server will automatically use port 3001 or 3002
npm run dev
```

### Python/ML Issues

**Error: Module not found (numpy, pandas, etc.)**
```bash
# Activate virtual environment and reinstall
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

**Error: Model file not found**
```bash
# Make sure model.pkl exists in anomaly-detection/
# If missing, retrain the model:
cd anomaly-detection
python train.py
```

## 🔍 System Status Check (Updated April 4, 2026)

### ✅ All Systems Operational

**Backend Status**
- Express.js: ✅ Ready (v4.22.1)
- Dependencies: ✅ All installed
- Routes: ✅ Configured
- ML Integration: ✅ Available
- Test: `npm start` in backend directory

**Frontend Status**
- Next.js: ✅ Ready (v16.2.0)
- React: ✅ Installed (v19.2.4)
- UI Components: ✅ All available (Radix UI)
- TypeScript: ✅ Configured (v5.7.3)
- Test: `npm run dev` in frontend directory

**Python ML Stack**
- scikit-learn: ✅ v1.8.0
- Pandas: ✅ v3.0.2
- NumPy: ✅ v2.4.4
- SHAP: ✅ v0.51.0
- XGBoost: ✅ v3.2.0
- Model: ✅ `/anomaly-detection/model.pkl`

**Environment**
- Node.js: ✅ Available
- Python: ✅ v3.8+
- Virtual Environment: ✅ Configured (.venv)

### 📋 Next Steps

1. **Start the backend**: `cd backend && npm start`
2. **Start the frontend**: `cd frontend && npm run dev`
3. **Visit dashboard**: http://localhost:3000
4. **View documentation**: See [docs/DEPENDENCIES.md](docs/DEPENDENCIES.md)

### 🎯 Features Ready to Use

- ✅ Transaction monitoring and risk scoring
- ✅ Fraud detection and prediction
- ✅ Alert management system
- ✅ User profile analysis
- ✅ Real-time dashboard with charts
- ✅ API endpoints for integration
- ✅ SHAP-based fraud explanations
- ✅ Web crawler integration

---

**Last Updated**: April 4, 2026
**Status**: Production Ready ✅
