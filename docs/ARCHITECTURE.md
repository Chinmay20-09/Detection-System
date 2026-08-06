# ARCHITECTURE — Detection System

> Reference architecture for the fraud/anomaly detection platform.
> Generated during repository cleanup (August 2026). Companion docs: `SETUP_GUIDE.md`, `DEPENDENCIES.md`, `QUICK_START.md` (same folder).

## 1. Project Overview

The **Detection System** is a full-stack credit-card fraud detection platform. It trains an XGBoost classifier on PCA-style transaction features, serves predictions through a Python Flask API, ingests transactions in real time through a Node.js queue + WebSocket pipeline, and visualizes results in a Next.js dashboard.

Three runtime processes:

| Component | Tech | Port | Role |
|---|---|---|---|
| `frontend/` | Next.js 16 / React 19 / TS | 3001 (dev) | Monitoring dashboard |
| `backend/` | Node.js / Express | 3000 | REST API + real-time ingestion + WebSocket |
| `ml-service/` | Python / Flask / XGBoost | 5000 | Model loading + inference (single & batch) |

Offline components: `anomaly-detection/` (training/prediction scripts, model artifact) and `data-generator/` (synthetic transaction generation).

## 2. Directory Tree

```
Detection-System/
├── README.md               # Project overview & quick start (root entry point)
├── docs/                   # Documentation (this file, setup, deps, checklists)
├── scripts/                 # start.bat / start.ps1 / start.sh launchers
├── .env / .env.example     # Environment configuration + template
├── .gitignore
│
├── frontend/               # Next.js dashboard
│   ├── app/                # App router: layout.tsx, page.tsx, globals.css
│   ├── components/         # dashboard/ tabs + ui/ (Radix wrappers)
│   ├── hooks/              # use-coming-soon, use-mobile, use-toast
│   ├── lib/                # api.ts (REST client), utils.ts (cn helper)
│   ├── public/             # Icons used by layout metadata
│   ├── styles/             # (removed — duplicate of app/globals.css)
│   └── config files        # package.json, tsconfig, next.config, components.json
│
├── backend/                # Express API + real-time pipeline
│   ├── server.mjs          # Bootstrap: express, CORS, WS, ingestion manager
│   ├── routes/api.js       # REST: transactions/alerts/users/metrics/crawl/predict
│   ├── services/mlservice.js   # HTTP client → ML service (legacy/duplicate client)
│   ├── crawler/crawler.js  # Crawlee page crawler
│   └── realtime/
│       ├── ingestion.mjs   # TransactionQueue + StreamProcessor + Manager
│       ├── websocket.mjs   # WebSocket server + demo stream
│       ├── routes.mjs      # /api/realtime/* endpoints
│       ├── ml-client.mjs   # SafeMLServiceClient (primary ML client)
│       ├── config.mjs      # Queue/batch/threshold config
│       ├── client.mjs / client-browser.mjs  # WS client libraries
│       └── examples.mjs    # Usage examples
│
├── ml-service/             # Python inference service
│   ├── service.py          # ModelManager + Flask endpoints
│   ├── client.py           # Python client library
│   ├── config.py           # Env-driven configuration
│   ├── model.pkl           # Trained XGBClassifier artifact
│   ├── __main__.py         # python -m entry point
│   ├── examples.py         # 7 runnable scenarios
│   ├── Dockerfile / start.sh / requirements.txt / .env.example
│
├── anomaly-detection/      # ML training scripts (legacy module)
│   ├── train.py            # Train XGBoost + SHAP explainer, save model.pkl
│   ├── predict.py          # Legacy batch prediction script (⚠ needs repair)
│   └── model.pkl           # Duplicate model artifact
│
└── data-generator/         # Synthetic transaction generation
    ├── generator.py        # TransactionDataGenerator
    ├── api.py              # DataAPI facade for backend integration
    ├── config.py           # Generation params & thresholds
    └── __init__.py
```

## 3. Frontend (`frontend/`)

- **Routing/layout:** App Router; `app/layout.tsx` imports `./globals.css` (dark security theme, Tailwind 4 + tw-animate-css) and Vercel Analytics.
- **Dashboard:** `app/page.tsx` renders `DashboardSidebar` + `DashboardHeader` and switches between tabs (Overview, Transactions, Alerts, User Profiles, Fraud Chains, Settings).
- **Data access:** `lib/api.ts` — typed REST client (`getTransactions`, `getAlerts`, `getUserProfiles`, `getDashboardMetrics`, `updateAlertStatus`) hitting the backend at `/api/...` (via `NEXT_PUBLIC_API_URL`). Tabs consume these through React hooks with fallback data.
- **UI kit:** `components/ui/` — shadcn-style Radix wrappers; `hooks/` provide coming-soon and mobile detection utilities.
- **Theme:** `theme-provider.tsx` (next-themes) supports dark/light.

## 4. Backend (`backend/`)

- **Entry:** `server.mjs` — Express + CORS + JSON body parsing; mounts `/api` (routes/api.js) and `/api/realtime` (routes.mjs); initializes the `SafeMLServiceClient` and the singleton ingestion manager; starts WebSocket + demo stream.
- **REST API** (`routes/api.js`): mock-backed endpoints for transactions, alerts (with PATCH status update), users, dashboard metrics; `/crawl` (Crawlee); `/predict` and `/explain-fraud` delegate to `services/mlservice.js` (HTTP → ML service with random-score fallback).
- **Real-time ingestion** (`realtime/`):
  - `TransactionQueue` — in-memory FIFO (default cap 10,000) with stats and overflow events.
  - `StreamProcessor` — batches dequeue (default 10 txns / 1s), fans out to the ML client with `Promise.allSettled`, caches results (last 1,000), computes per-txn `action` (block/alert/review/approve from score).
  - `RealtimeIngestionManager` — orchestrates queue + processor + WS client registry.
  - `websocket.mjs` — accepts WS connections, routes message types (`ingest`, `batch-ingest`, `query-stats`, …), broadcasts results, and runs a demo stream that periodically POSTs a synthetic transaction to the ML service and pushes `fraud-update` payloads.
- **ML client:** `realtime/ml-client.mjs` (`MLServiceClient` + `SafeMLServiceClient` with retry-on-init) is the primary client. `services/mlservice.js` is a legacy duplicate used only by `/predict` routes.

## 5. ML Service (`ml-service/`)

- **`service.py`:** `ModelManager` loads `model.pkl` (pickled `XGBClassifier`) at startup and exposes:
  - `GET /health`, `GET /ready` — liveness/model readiness
  - `POST /predict` — single transaction JSON → `{score, risk_level, is_fraud, metadata}`
  - `POST /predict-batch` — list of transactions → `{predictions: [...]}`
  - `GET /stats`, `POST /stats/reset` — in-process counters
  - `GET /model-info` — model metadata + expected feature list
- **Feature contract:** 30 features — `V1..V28`, `Time`, `Amount`. Missing keys default to `0.0` (lenient but risky; see notes).
- **Config:** `config.py` reads env (`ML_SERVICE_HOST/PORT`, `MODEL_PATH`, `LOG_LEVEL`, …).
- **Packaging:** `Dockerfile` (python:3.10-slim, gunicorn) and `start.sh`. ⚠ Known issue: gunicorn is invoked as `service:app`, but `service.py` currently exposes no module-level `app` — must be fixed before container/WSGI deployment.

## 6. Real-Time Pipeline (end-to-end)

```
Client (HTTP/WS) ──► backend/ingestion.mjs (queue)
                          │  batch every 1s / 10 txns
                          ▼
                    backend/realtime/ml-client.mjs
                          │  POST /predict-batch
                          ▼
                    ml-service/service.py (XGBoost)
                          │  score, risk_level, is_fraud
                          ▼
                    StreamProcessor: action = f(score)
                          │
              ┌───────────┴────────────┐
              ▼                        ▼
        WS broadcast            result cache (last 1000)
        (transaction-result)    GET /api/realtime/results/:id
```

1. Transaction arrives via `POST /api/realtime/ingest` (or WS `ingest`).
2. Enqueued into `TransactionQueue` (FIFO, 10k cap).
3. `StreamProcessor` dequeues a batch and calls the ML service.
4. ML service returns per-transaction score; processor maps score → action.
5. Result is cached and broadcast to connected WebSocket clients.
6. `GET /api/realtime/results/:id` serves cached results; `/api/realtime/stats` and `/health` expose pipeline telemetry.

## 7. Data Flow (offline / training)

1. `data-generator/generator.py` synthesizes transactions (`Time`, `Amount`, `V1–V28`, `Class`, optional `user_id`).
2. `anomaly-detection/train.py` (via `data-generator` or CSV) trains an XGBoost classifier (80/20 stratified split, `scale_pos_weight` for imbalance), builds a SHAP explainer, and writes `model.pkl`.
3. The artifact is copied to `ml-service/model.pkl` for serving (duplication — see cleanup notes).
4. `ml-service/service.py` loads it and serves inference over HTTP.

## 8. API Communication Summary

| Consumer → Provider | Protocol | Endpoint |
|---|---|---|
| Frontend → Backend | HTTP/JSON | `/api/transactions`, `/api/alerts`, `/api/users`, `/api/metrics` |
| Frontend → Backend | HTTP/JSON | `/api/predict`, `/api/explain-fraud`, `/api/crawl` |
| Client → Backend | HTTP/JSON | `/api/realtime/ingest`, `/batch-ingest`, `/stats`, `/health`, `/results/:id` |
| Client → Backend | WebSocket | `ws://host:3000` (ingest, batch-ingest, query-stats, …) |
| Backend → ML Service | HTTP/JSON | `POST :5000/predict`, `POST :5000/predict-batch`, `GET :5000/health` etc. |
| ML Service ↔ Python client | HTTP/JSON | `ml-service/client.py` → same endpoints |

## 9. Deployment Overview

- **Local dev:** `scripts/start.bat` / `scripts/start.ps1` / `scripts/start.sh` launch backend (`npm start`) and frontend (`npm run dev -p 3001`). ML service runs separately: `cd ml-service && python service.py`.
- **Containerized:** `ml-service/Dockerfile` (gunicorn) for the inference tier; backend/frontend follow standard Node/Next container patterns (not yet provided).
- **Production gaps (documented, not addressed here):** no auth/rate limiting, in-memory state only (no DB/queue persistence), unversioned model artifact, and the `service:app` gunicorn mismatch noted in §5. See `CLEANUP_REPORT.md` for the full technical-debt list.
