# SUMMARY — Detection-System Repository Audit

> **Audit date:** August 4, 2026
> **Scope:** Full repository analysis for integration into a production-style inference service
> **Constraint honored:** No source code modified. Only obsolete markdown documentation removed and this summary created.

---

## 1. Project Overview

### Purpose
A full-stack **credit-card fraud / anomaly detection system**. It combines a Python XGBoost model (trained on PCA-transformed credit-card transaction features), a real-time transaction ingestion pipeline, a REST/WebSocket backend, and a Next.js monitoring dashboard. The system detects fraudulent transactions in near real time and surfaces alerts, risk scores, and per-transaction "reasons."

### Current Architecture
The system is split into **four loosely-connected layers**, each running as its own process:

```
┌──────────────────────────────────────────────────────────────────────┐
│  Next.js Frontend (frontend/)                 :3001  (dev)            │
│  Dashboard — Overview / Transactions / Alerts / Users / Chains / ... │
└──────────────────────────┬───────────────────────────────────────────┘
                           │  HTTP (lib/api.ts)
┌──────────────────────────▼───────────────────────────────────────────┐
│  Node.js / Express Backend (backend/)        :3000                   │
│  ├─ routes/api.js           → REST endpoints (mock data + predict)   │
│  ├─ services/mlservice.js   → axios → ML service (duplicate client)  │
│  ├─ realtime/               → queue, stream processor, WebSocket     │
│  │   ├─ ingestion.mjs       → FIFO queue + batch stream processor    │
│  │   ├─ websocket.mjs       → WS server + demo stream               │
│  │   ├─ routes.mjs          → /api/realtime/* REST endpoints         │
│  │   └─ ml-client.mjs       → HTTP client for Python ML service      │
│  └─ crawler/crawler.js      → Crawlee-based URL crawler              │
└──────────────────────────┬───────────────────────────────────────────┘
                           │  HTTP (JSON, port 5000)
┌──────────────────────────▼───────────────────────────────────────────┐
│  Python ML Inference Service (ml-service/)   :5000                   │
│  ├─ service.py            → Flask app, ModelManager, /predict        │
│  ├─ client.py             → Python client library                    │
│  ├─ config.py             → env-driven configuration                 │
│  └─ model.pkl             → pickled XGBClassifier                    │
└──────────────────────────────────────────────────────────────────────┘

  Offline / batch components (not part of the live request path):
  ├─ anomaly-detection/       → train.py, predict.py, model.pkl
  └─ data-generator/          → synthetic credit-card data generation
```

### Technologies Used
| Layer | Technology |
|---|---|
| ML | Python, XGBoost, scikit-learn, pandas, numpy, SHAP |
| ML Serving | Flask, gunicorn, Docker |
| Backend | Node.js, Express, axios, crawlee, `ws` (undeclared) |
| Real-time | In-memory queue, WebSocket (`ws`), HTTP batch polling |
| Frontend | Next.js 16, React 19, TypeScript, Tailwind CSS 4, Radix UI, Recharts |
| Data | Synthetic generator (no real database; in-memory mock data) |

---

## 2. Directory Structure

### Root level
| Path | Purpose | Notes |
|---|---|---|
| `README.md` | Canonical project overview, quick start, endpoints | **Keep** |
| `SETUP_GUIDE.md` | Detailed setup + troubleshooting for the dashboard stack | Kept (user-facing) |
| `QUICK_START.md` | 5-minute run guide for the realtime/ML stack | Kept, but still references removed docs (see §9) |
| `DEPENDENCIES.md` | Full dependency listing | Referenced by README |
| `QUICK_REFERENCE.md` | Condensed dev cheat-sheet | Kept (borderline) |
| `COMING_SOON_GUIDE.md` / `COMING_SOON_EXAMPLES.md` | Docs for the "Coming Soon" popup feature in the UI | Kept — documents live code |
| `SUMMARY.md` | **This file** | New |
| `.env`, `.env.example` | Environment configuration + template | `.env` gitignored |
| `.gitignore` | Ignores node_modules, .venv, .env, build output, crawlee storage | |
| `start.bat` / `start.ps1` / `start.sh` | Platform launchers (backend + frontend) | `start.bat - Shortcut.lnk` is junk |
| `1NPDPRO5.DOC` | Unrelated binary `.doc` file | Obsolete/junk, not markdown |

### `anomaly-detection/` — the original ML module
| File | Purpose | Status |
|---|---|---|
| `train.py` | Trains XGBoost on generated or CSV data, SHAP explainer, saves `model.pkl` | **Contains unmerged git conflict markers — will not run** |
| `predict.py` | Loads model, runs batch predictions, prints per-user summary | **Broken — references undefined `X`; conflict markers** |
| `model.pkl` | Trained XGBClassifier | Duplicated in `ml-service/` |

### `data-generator/` — synthetic data source
| File | Purpose |
|---|---|
| `generator.py` | `TransactionDataGenerator` — log-normal amounts, PCA-style V1–V28 noise, optional `user_id` |
| `api.py` | `DataAPI` facade + singleton accessor for backend integration |
| `config.py` | Central config: data source, training/prediction params, thresholds |
| `__init__.py` | Package exports |

### `ml-service/` — the closest thing to a production inference service
| File | Purpose | Notes |
|---|---|---|
| `service.py` | `ModelManager` (load/cache/predict/stats) + Flask routes: `/health`, `/ready`, `/predict`, `/predict-batch`, `/stats`, `/stats/reset`, `/model-info` | **No module-level `app` → `gunicorn service:app` fails** |
| `client.py` | Python HTTP client (`MLServiceClient`, `PredictionResult`) | |
| `config.py` | Env-driven config (host, port, model path, features, thresholds) | |
| `requirements.txt` | Flask, numpy, pandas, sklearn, xgboost, requests, dotenv, gunicorn | Pinned but dated |
| `Dockerfile` | Python 3.10-slim, gunicorn CMD | Gunicorn CMD broken (see above) |
| `__main__.py` | `python -m ml_service` entry with dep/model checks | |
| `examples.py` | 7 runnable example/benchmark scripts | |
| `start.sh` | Bash launcher | Also uses broken gunicorn form |
| `model.pkl` | Copy of trained model | Duplicated artifact |
| `.env.example` | Service env template | |

### `backend/` — API + realtime ingestion
| Path | Purpose | Notes |
|---|---|---|
| `server.mjs` | Express bootstrap, wires realtime + ML client | |
| `routes/api.js` | Mock transactions/alerts/users + `/metrics`, `/crawl`, `/predict`, `/explain-fraud` | Mostly mock data |
| `services/mlservice.js` | Axios → `:5000/predict` with random-score fallback | **Duplicates `realtime/ml-client.mjs`; field mismatch (`risk` vs `risk_level`)** |
| `crawler/crawler.js` | Crawlee CheerioCrawler, max 5 requests | |
| `realtime/ingestion.mjs` | `TransactionQueue`, `StreamProcessor`, `RealtimeIngestionManager` | In-memory, event-driven |
| `realtime/websocket.mjs` | WS server, demo stream, live prediction loop | **Sends `{amount, frequency, ip_risk}` — not model features** |
| `realtime/routes.mjs` | `/api/realtime/*` endpoints | |
| `realtime/config.mjs` | Queue/batch/threshold configuration | Thresholds duplicated across layers |
| `realtime/ml-client.mjs` | `MLServiceClient` + `SafeMLServiceClient` | |
| `realtime/client.mjs` / `client-browser.mjs` | WS clients for Node/browser | |
| `realtime/examples.mjs` | Usage examples | |
| `package.json` | express, cors, crawlee, axios | **`ws` imported but not declared** |

### `frontend/` — Next.js dashboard
| Path | Purpose |
|---|---|
| `app/page.tsx`, `app/layout.tsx` | Dashboard shell + tabs |
| `components/dashboard/` | Header, sidebar, tab components |
| `components/ui/` | 40+ Radix UI wrappers (button, card, dialog, table, chart, …) |
| `components/theme-provider.tsx` | next-themes dark/light |
| `hooks/` | `use-coming-soon`, `use-mobile`, `use-toast` |
| `styles/globals.css` | Tailwind 4 theme |
| `components.json`, `tsconfig.json`, `next.config.mjs`, `package.json` | Config |

### Root-level `app/` — **stray duplicate**
`app/page.tsx` + `app/layout.tsx` mirror `frontend/app/` and import from `@/components/...`, which only resolves inside the frontend project. This is an orphaned copy outside any Next.js root and is not used by any build. **Safe to remove.**

---

## 3. Model Pipeline

### Dataset loading
- `anomaly-detection/train.py` sets `DATA_SOURCE = 'generator'` and calls `get_data_from_generator(source='generator', n_samples=50000, fraud_ratio=0.001, random_seed=42)`.
- The `data-generator` synthesizes transactions: `Time` (0–172800s), `Amount` (log-normal, clipped), `V1–V28` (standard normal; 30% of fraud rows get shifted distributions), and binary `Class`.
- A `csv` branch exists but the real CreditCard CSV path (`G:\KAAM\Hackathon\archive\creditcard.csv`) is hardcoded to one developer's machine.

### Preprocessing
- Minimal: `X = df.drop('Class', axis=1)`, stratified 80/20 `train_test_split`.
- No scaling, no feature engineering, no outlier handling.
- Class imbalance is handled only via `scale_pos_weight = non_fraud / fraud`.

### Training
- `XGBClassifier(n_estimators=200, max_depth=6, learning_rate=0.1, scale_pos_weight=..., eval_metric='logloss', use_label_encoder=False)`.
- No cross-validation, no hyperparameter search, no early stopping, no train/validation/test holdout discipline (train/test only).

### Evaluation
- `confusion_matrix` + `classification_report` printed at a **custom threshold of 0.3**.
- No metrics are persisted anywhere; the "production-ready" claims in the old docs were not backed by saved artifacts.

### SHAP explainability
- `shap.TreeExplainer(model)` built at train time.
- `predict_with_reason()` computes SHAP values per sample and maps high-contribution features to human-readable reasons via a small `FEATURE_MAP`.
- **The SHAP explainer is created in the training script and used in the training script only — it is not shipped with the model** and is not exposed by the inference service.

### Model saving
- `pickle.dump(model, open("model.pkl", "wb"))` in the training script's working directory.
- The same artifact is manually copied to `ml-service/model.pkl` (documented in `ml-service/README.md`).
- No model registry, versioning, or metadata (feature list is re-derived by hand in `service.py`).

### Prediction flow
1. `ml-service/service.py` loads `model.pkl` at startup (`ModelManager.load_model`).
2. `_prepare_features()` builds a 30-dim vector `[V1..V28, Time, Amount]`, defaulting missing keys to `0.0`.
3. `predict_proba` → fraud score → `risk_level` buckets (0.2/0.4/0.6/0.8) → `is_fraud = score > 0.5`.
4. `/predict` and `/predict-batch` expose single/batch scoring over HTTP.
5. Backend realtime pipeline batches queued transactions and calls the service; results are cached in memory and broadcast over WebSocket.

---

## 4. Current Problems

### Critical
1. **Committed git merge conflicts** — `anomaly-detection/train.py` (lines 5, 25) and `predict.py` (line 24) contain raw `<<<<<<< HEAD` / `=======` / `>>>>>>>` markers **already committed to the working tree**. Both scripts are syntactically broken and cannot run. This is a release-blocking defect.
2. **Broken inference script** — `predict.py` uses `X.columns` before `X` is ever defined and mixes the `user_id` merge-conflict branch with generator output. It is un-runnable and conceptually confused (selects one synthetic user, then drops `Class`/`user_id`).
3. **Broken production entry point** — `ml-service/Dockerfile` and `start.sh` invoke `gunicorn ... service:app`, but `service.py` defines **no module-level `app`**. `app` only exists as `MLServiceServer.app` after instantiation. Gunicorn deployment fails.
4. **Undeclared runtime dependency** — `backend/realtime/websocket.mjs` imports `ws`, but `backend/package.json` does not list it. Startup depends on a transitive install and fails on a clean install.

### High
5. **Hardcoded developer paths** — `G:\KAAM\Hackathon\archive\creditcard.csv` in `predict.py` and `data-generator/config.py`.
6. **Training/inference coupling** — training logic (SHAP, reason mapping, thresholds) lives inside `train.py`; prediction logic (thresholds, actions) is re-implemented independently in `predict.py`, `service.py`, and `backend/realtime/config.mjs`. Changing a threshold requires editing four places.
7. **Duplicate ML clients** — `backend/services/mlservice.js` (axios, returns `{risk, score}`) and `backend/realtime/ml-client.mjs` (returns `{score, risk_level, is_fraud}`). Two inconsistent client contracts for the same service.
8. **Feature-space mismatch in the realtime path** — the WebSocket demo stream sends `{amount, frequency, ip_risk}` and the REST `/predict` docs show `{amount, merchant}`. None of these are model features (`V1–V28, Time, Amount`); they silently map to zero vectors, so every realtime prediction is effectively computed from the same all-zero input.
9. **Inconsistent thresholds** — 0.3 (train), 0.05/0.15 (predict.py), 0.5 (service.py `is_fraud`), 0.2/0.4/0.6/0.8 (risk buckets), 0.3/0.5/0.8 (realtime config). No single source of truth.

### Medium
10. **Documentation drift** — README and deleted docs claim "production ready," ports are documented inconsistently (backend on 3000 vs 5000 in different sections), and the realtime docs describe behaviors the code does not implement.
11. **No tests** — no unit/integration tests for Python or Node; the only "verification" is `examples.py` / `examples.mjs` against a live server.
12. **In-memory everything** — mock data in routes, in-memory queue, in-memory result cache. No persistence, no distributed queue, no audit trail.
13. **Stats bug** — `ModelManager.total_inference_time` is never incremented, so `/stats` latency numbers are always ~0.
14. **Model duplication** — two `model.pkl` copies with no checksum/version linkage; they can drift silently.
15. **Silent ML degradation** — `mlservice.js` and the ingestion manager fall back to `Math.random()` scores when the ML service is down; dashboards then show plausible-looking but meaningless numbers.
16. **Frontend/backend contract mismatch** — dashboard tabs expect `{riskScore, riskLevel}` shapes while the realtime pipeline emits `{score, risk_level}`; the frontend consumes mostly mock shapes.

### Low / hygiene
17. Duplicate `globals.css` and layout files between root `app/` and `frontend/app/`.
18. `start.bat - Shortcut.lnk`, `1NPDPRO5.DOC` junk files at root.
19. `README.md` claims an MIT license but no `LICENSE` file exists.

---

## 5. Deployment Readiness — what blocks an API service

| Blocker | Where | Impact |
|---|---|---|
| Merge-conflict markers in committed Python | `anomaly-detection/*` | Training/inference scripts cannot run |
| No module-level Flask `app` for gunicorn | `ml-service/service.py` + Dockerfile | Container/WSGI deployment fails out of the box |
| `ws` missing from `package.json` | `backend/` | Clean backend install fails at import |
| No auth, rate limiting, or TLS story | Flask + Express | Exposing `:5000`/`:3000` publicly is unsafe |
| No input schema validation | `/predict`, `/ingest` | Bad/malicious payloads reach the model or crash handlers |
| No persistence | all layers | Restart loses queue, results, alerts |
| No tests / CI | repo | No safety net for refactors |
| Model artifacts unversioned | `model.pkl` × 2 | Cannot roll back or audit which model served what |
| Docs claim readiness the code does not deliver | README + removed docs | Misleading operator expectations |

**Verdict:** the realtime ingestion layer and the Flask service are *promising skeletons*, but the system as committed is **not deployable** without first fixing the four critical blockers above.

---

## 6. Recommended Architecture

A clean, production-style layout for the ML portion (what the user asked to design toward):

```
project/
├── train.py            # Training CLI (data → model artifact + metrics)
├── predict.py          # Batch/offline inference CLI (CSV in → JSON/CSV out)
├── api.py              # FastAPI/Flask serving layer (loads model, exposes /predict)
├── model.pkl           # Single canonical model artifact (or model/ dir with version)
├── utils.py            # Shared preprocessing, feature schema, thresholds, logging
├── config.py           # Env-driven config (paths, thresholds, model metadata)   [recommended add]
├── requirements.txt    # Pinned runtime deps (split dev/test separately)
└── tests/              # Unit tests for preprocessing + inference                [recommended add]
```

Why each file exists:
- **`train.py`** — the *only* place that touches training data, fits the model, computes evaluation metrics, and writes a versioned artifact. Running it must be side-effect free for serving.
- **`predict.py`** — a thin CLI: read CSV → run `utils.py` preprocessing → load model → emit predictions as JSON/CSV. It never trains and never hardcodes data paths (paths come from CLI/env).
- **`api.py`** — the serving entry point. Loads the model once at startup, exposes `/health`, `/predict`, `/predict-batch`. Owns request validation and error mapping. Run via gunicorn/uvicorn with a real WSGI `app` object.
- **`model.pkl`** — the single artifact; train.py writes it, api.py and predict.py read it. Nothing else creates a copy.
- **`utils.py`** — the single source of truth for the feature list (`V1..V28, Time, Amount`), the `_prepare_features` transformation, and risk-bucket thresholds. Both `predict.py` and `api.py` import it, eliminating the 4-way threshold duplication.
- **`requirements.txt`** — pinned versions for reproducible deploys.
- **`config.py` / `tests/`** — recommended additions: env-driven configuration and regression tests for the data→prediction contract.

This mirrors what `ml-service/` already attempts, but adds: a real `app` for WSGI, shared utils so training/prediction/serving can't drift, and a single model artifact.

---

## 7. Integration Plan (external workflow system, e.g. AxisFlow)

Required contract: **CSV uploaded → trained model loaded → predictions generated → results returned as JSON.**

1. **Expose a batch endpoint in `api.py`**
   `POST /predict-batch` accepts `{ "features": [row...] }` or a file upload, and returns
   ```json
   { "predictions": [ { "transaction_id": "...", "score": 0.87,
                        "risk_level": "HIGH", "is_fraud": 1 } ] }
   ```
   The service already has this endpoint (`ml-service/service.py`), so the fastest path is to fix the blockers and keep it.

2. **AxisFlow upload step** → POST the CSV (or its rows) to `/predict-batch`; the service validates the header against the canonical feature list in `utils.py` (columns missing → 422 with a clear message; unknown extra columns → ignored).

3. **Model loading** → server loads `model.pkl` once at startup (`ModelManager`), so each request pays only inference cost. Add a `/model-info` check in the workflow to confirm model version before batch runs.

4. **Prediction** → row-wise feature alignment (column order independent — align by name), `predict_proba`, risk bucketing.

5. **Return JSON** → one JSON document with per-row `transaction_id`, `score`, `risk_level`, `is_fraud`, plus a summary block (`total`, `flagged`, `blocked`, `avg_score`) so AxisFlow can branch downstream.

6. **Operational glue** — the workflow should poll `/health` before submitting, set `MODEL_PATH` via env, and treat 4xx (bad input) vs 5xx (service failure) differently. For larger CSVs, chunk rows to stay under the configured batch limit.

7. **Post-integration** — the same endpoint is already what the Node backend's `SafeMLServiceClient` calls, so AxisFlow can reuse the identical contract.

---

## 8. Refactoring Priority

Highest → lowest:

1. **Resolve the merge conflicts** in `anomaly-detection/train.py` and `predict.py` (or delete `predict.py` once the service covers its role). Blocking everything else.
2. **Expose a real module-level `app`** in `ml-service/service.py` so gunicorn/Docker work; add `ws` to `backend/package.json`.
3. **Create `utils.py` (shared feature schema + thresholds)** and route `predict.py`, `service.py`, and `backend/realtime/config.mjs` through it — kill the duplicated thresholds and the `risk` vs `risk_level` field mismatch.
4. **Delete the stray root `app/`** and the duplicate `backend/services/mlservice.js` client (keep `realtime/ml-client.mjs`).
5. **Fix the feature-space mismatch** in `websocket.mjs` demo stream / `/predict` docs so realtime requests actually carry model features.
6. **Make paths configurable** — remove `G:\KAAM\...`; use env/CLI for `MODEL_PATH`, CSV input, output dirs.
7. **Fix the `/stats` latency bug** (`total_inference_time` never updated) and remove the random-score silent fallback (or mark it explicitly).
8. **Add input validation + auth/rate limiting** to the Flask service before it is exposed beyond localhost.
9. **Introduce tests** (`pytest` for Python, `node --test` for backend) around the predict contract.
10. **Version the model artifact** (e.g. `models/xgboost-v1.pkl` + metadata JSON) and persist results (DB or at least append-only JSONL) to make the pipeline auditable.
11. **Consolidate docs** — one README per component is enough; point QUICK_START's "see also" references at the surviving READMEs.

---

## 9. Markdown Cleanup — What Was Removed and Why

Removed (8 files — all obsolete generated summaries, dated status snapshots, or direct duplicates of in-component READMEs):

| File | Reason |
|---|---|
| `PROJECT_STATUS.md` | Dated one-time deployment status snapshot ("Generated: April 4, 2026") — obsolete |
| `IMPLEMENTATION_COMPLETE.md` | Generated completion/celebration summary — obsolete |
| `COMPLETION_CHECKLIST.md` | Generated "what's built / what's next" planning checklist — obsolete |
| `SYSTEM_READY.md` | Generated mega-guide duplicating README + component READMEs; a status doc, not operational reference |
| `ML_INFERENCE_SETUP.md` | Duplicates `ml-service/README.md` |
| `REALTIME_QUICKSTART.md` | Duplicates `backend/realtime/README.md` |
| `REALTIME_SETUP_COMPLETE.md` | Duplicates `backend/realtime/README.md` |
| `DATA_GENERATOR_INTEGRATION.md` | Duplicates `data-generator/README.md` |

Kept: `README.md` (canonical), `DEPENDENCIES.md` (referenced by README), `SETUP_GUIDE.md`, `QUICK_START.md`, `DEPLOYMENT_CHECKLIST.md`, `QUICK_REFERENCE.md`, `COMING_SOON_GUIDE.md`, `COMING_SOON_EXAMPLES.md`, and all in-component READMEs (`ml-service/`, `backend/realtime/`, `data-generator/`). No `LICENSE`/`CONTRIBUTING`/CI-referenced docs exist to preserve.

> ⚠️ `QUICK_START.md` still references `SYSTEM_READY.md` and `REALTIME_SETUP_COMPLETE.md` in its "Questions?" footer. Those links now dangle; updating them to the surviving READMEs is a 2-line doc fix.

---

## 10. Files Safe To Remove

- `app/` (root-level) — orphaned duplicate of `frontend/app/`; imports resolve nowhere.
- `anomaly-detection/predict.py` — broken (undefined `X`, conflict markers) and functionally replaced by `ml-service`; only remove after the service covers its per-user batch role.
- `backend/services/mlservice.js` — superseded by `backend/realtime/ml-client.mjs` (two conflicting clients for one service).
- `anomaly-detection/model.pkl` **or** `ml-service/model.pkl` — keep exactly one canonical artifact (suggest keeping it inside `ml-service/` and pointing training to write there, or move to a shared `models/` dir).
- `start.bat - Shortcut.lnk` — Windows shortcut junk.
- `1NPDPRO5.DOC` — unrelated legacy binary document.
- `data-generator/api.py` `generate_user_transactions` / `export_to_csv` — currently unused by any caller (verify before removing).

---

## 11. Final Verdict

| Dimension | Score /10 | Rationale |
|---|---|---|
| **Code quality** | 3 | Merge conflicts committed to source, undefined-variable bug, duplicate clients, undocumented fallbacks. The realtime layer is reasonably structured, which keeps it from a 1–2. |
| **ML quality** | 4 | Sound baseline approach (XGBoost + `scale_pos_weight` + SHAP), but trained on synthetic data, no CV/hyperparameter tuning, no persisted metrics, no shipped explainer, thresholds scattered. |
| **Maintainability** | 3 | Four sources of truth for thresholds, doc/code drift, no tests, no lint/CI, hand-copied model artifacts. |
| **Scalability** | 5 | Stateless Flask service + batch processing are horizontally scalable in principle; in-memory queue/cache and single-machine assumption cap real scale; gunicorn config broken today. |
| **Hackathon readiness** | 8 | Impressive demo surface: polished dashboard, WebSocket streaming, health/stats endpoints, generous docs. The demo path works and looks good. |
| **Production readiness** | 2 | Blockers 1–4 in §5 prevent even a clean local boot; no auth, persistence, tests, or model versioning. Claims of "production ready" in the (now removed) docs were aspirational. |

**Overall:** A strong hackathon/demo system with a promising realtime architecture, wrapped in documentation that overstates its maturity. With the four critical blockers fixed (§5), a shared feature/threshold module (§8.3), and a real WSGI entry point, the `ml-service/` layer is genuinely close to production-style inference serving — and the AxisFlow integration path (§7) is short.

---

*Generated by repository audit — no source code was modified.*
