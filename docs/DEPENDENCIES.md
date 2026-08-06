# DEPENDENCIES.md

Complete list of all dependencies used in the Detection System project.

## Frontend Dependencies (Next.js/React)

### Core Framework
- **next** (16.2.0) - React framework with server-side rendering
- **react** (^19) - JavaScript library for building user interfaces
- **react-dom** (^19) - React package for DOM manipulation

### UI Component Libraries
- **@radix-ui/react-accordion** (1.2.12) - Accordion component
- **@radix-ui/react-alert-dialog** (1.1.15) - Alert dialog component
- **@radix-ui/react-aspect-ratio** (1.1.8) - Aspect ratio wrapper
- **@radix-ui/react-avatar** (1.1.11) - Avatar component
- **@radix-ui/react-checkbox** (1.3.3) - Checkbox component
- **@radix-ui/react-collapsible** (1.1.12) - Collapsible component
- **@radix-ui/react-context-menu** (2.2.16) - Context menu component
- **@radix-ui/react-dialog** (1.1.15) - Dialog/Modal component
- **@radix-ui/react-dropdown-menu** (2.1.16) - Dropdown menu component
- **@radix-ui/react-hover-card** (1.1.15) - Hover card component
- **@radix-ui/react-label** (2.1.8) - Form label component
- **@radix-ui/react-menubar** (1.1.16) - Menubar component
- **@radix-ui/react-navigation-menu** (1.2.14) - Navigation menu component
- **@radix-ui/react-popover** (1.1.15) - Popover component
- **@radix-ui/react-progress** (1.1.8) - Progress bar component
- **@radix-ui/react-radio-group** (1.3.8) - Radio group component
- **@radix-ui/react-scroll-area** (1.2.10) - Scrollable area component
- **@radix-ui/react-select** (2.2.6) - Select dropdown component
- **@radix-ui/react-separator** (1.1.8) - Separator component
- **@radix-ui/react-slider** (1.3.6) - Slider component
- **@radix-ui/react-slot** (1.2.4) - Slot component for composition
- **@radix-ui/react-switch** (1.2.6) - Toggle switch component
- **@radix-ui/react-tabs** (1.1.13) - Tabs component
- **@radix-ui/react-toast** (1.2.15) - Toast notification component
- **@radix-ui/react-toggle** (1.1.10) - Toggle button component
- **@radix-ui/react-toggle-group** (1.1.11) - Toggle group component
- **@radix-ui/react-tooltip** (1.2.8) - Tooltip component

### Form Management
- **react-hook-form** (^7.54.1) - Performant form library
- **@hookform/resolvers** (^3.9.1) - Schema validators for react-hook-form
- **zod** (^3.24.1) - TypeScript-first schema validation library

### Data Visualization
- **recharts** (2.15.0) - React charting library built on D3.js

### Styling & CSS
- **tailwindcss** (^4.2.0) - Utility-first CSS framework
- **autoprefixer** (^10.4.20) - PostCSS plugin to parse CSS and add vendor prefixes
- **@tailwindcss/postcss** (^4.2.0) - Tailwind CSS core
- **class-variance-authority** (^0.7.1) - Type-safe variant composition library
- **clsx** (^2.1.1) - Utility for constructing className strings
- **tailwind-merge** (^3.3.1) - Merge Tailwind CSS classes without conflicts

### Icons
- **lucide-react** (^0.564.0) - Beautiful React icon library

### Notifications
- **sonner** (^1.7.1) - Toast notifications library

### Theme Management
- **next-themes** (^0.4.6) - Theme provider for Next.js

### Date & Time
- **date-fns** (4.1.0) - Modern JavaScript date utility library
- **react-day-picker** (9.13.2) - Flexible date picker component

### UI Utilities
- **cmdk** (1.1.1) - Command palette component
- **embla-carousel-react** (8.6.0) - Carousel component library
- **input-otp** (1.4.2) - OTP input component
- **react-resizable-panels** (^2.1.7) - Resizable panel component
- **vaul** (^1.1.2) - Drawer component

### Analytics
- **@vercel/analytics** (1.6.1) - Vercel Web Analytics

### Development Dependencies
- **typescript** (5.7.3) - TypeScript compiler
- **@types/node** (^22) - TypeScript definitions for Node.js
- **@types/react** (^19) - TypeScript definitions for React
- **@types/react-dom** (^19) - TypeScript definitions for React DOM
- **postcss** (^8.5) - CSS transformation tool
- **tw-animate-css** (1.3.3) - Tailwind animation utilities

---

## Backend Dependencies (Node.js/Express)

### Core Framework
- **express** (^4.19.2) - Web application framework
- **cors** (^2.8.5) - Cross-origin resource sharing middleware

### Web Scraping & Crawling
- **crawlee** (^3.16.0) - Web scraping and automation tool

### HTTP Client
- **axios** (^1.14.0) - Promise-based HTTP client

### Development Dependencies
- **nodemon** - Monitor for changes and restart application (optional, used for development)

---

## Python Dependencies (Anomaly Detection - Optional)

Located in `anomaly-detection/` directory

### Machine Learning & Data Science
- **numpy** - Numerical computing library
- **pandas** - Data manipulation and analysis
- **scikit-learn** - Machine learning library
- **tensorflow** or **pytorch** - Deep learning frameworks

### ML Explainability
- **shap** - SHAP (SHapley Additive exPlanations) for model interpretability

### Utilities
- **flask** or **fastapi** - Web framework for ML service API

---

## Environment Variables Configuration

All environment variables are configured in `.env`, `.env.local`, or `.env.backend` files:

### Backend (.env or backend/.env)
```
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
```
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_ANALYTICS_ENABLED=true
NEXT_PUBLIC_APP_NAME=Detection System Dashboard
NEXT_PUBLIC_APP_VERSION=1.0.0
```

---

## Installation Instructions

### Install Frontend Dependencies
```bash
cd frontend
npm install
```

### Install Backend Dependencies
```bash
cd backend
npm install
```

### Install Python Dependencies (Optional)
```bash
cd anomaly-detection
pip install -r requirements.txt
```

---

## Running the Application

### Terminal 1 - Backend Server
```bash
cd backend
npm start          # or npm run dev with nodemon
# Runs on http://localhost:3000
```

### Terminal 2 - Frontend Development Server
```bash
cd frontend
npm run dev
# Runs on http://localhost:3001
```

### Terminal 3 - ML Service (Optional)
```bash
cd anomaly-detection
python train.py
# Runs on http://localhost:5000
```

---

## API Endpoints Available

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/transactions` | Get all transactions |
| GET | `/api/alerts` | Get all alerts |
| GET | `/api/users` | Get user profiles |
| GET | `/api/metrics` | Get dashboard metrics |
| POST | `/api/predict` | Predict fraud for a transaction |
| POST | `/api/explain-fraud` | Get fraud explanation with SHAP |
| PATCH | `/api/alerts/:id` | Update alert status |
| GET | `/api/crawl` | Crawl a URL |

---

## Troubleshooting

### Port Already in Use
```bash
# Find process using port 3000/3001
netstat -ano | findstr :3000
# Kill process
taskkill /PID <PID> /F
```

### Dependencies Not Installed
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

### TypeScript Errors
```bash
# Rebuild TypeScript
npm run build
```

---

## Version Information

- **Node.js**: v14+ (tested with v18+)
- **npm**: v6+
- **Python**: v3.8+ (for ML components)
- **Next.js**: 16.2.0
- **React**: 19
- **Express**: ^4.19.2
- **TypeScript**: 5.7.3

---
