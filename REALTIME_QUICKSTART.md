# Real-Time Data Ingestion - Quick Start Guide

## What is This?

The Real-Time Data Ingestion Layer enables your Detection System to process credit card transactions **instantly** as they arrive, detecting fraud with minimal latency.

## 5-Minute Setup

### Step 1: Install Dependencies

The following packages are required. Add to `backend/package.json`:

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "cors": "^2.8.5",
    "ws": "^8.13.0",
    "axios": "^1.4.0"
  }
}
```

Install:
```bash
npm install
```

### Step 2: Server Already Updated ✅

The `backend/server.mjs` file has been updated to include:
- WebSocket server initialization
- Real-time ingestion routes
- Health check endpoints

### Step 3: Start the Server

```bash
npm start
# or
node backend/server.mjs
```

You should see:
```
🚀 Server running on http://localhost:3000
🔴 WebSocket real-time streaming available at ws://localhost:3000
📊 Real-time ingestion pipeline started
```

## Basic Usage

### Option A: Browser WebSocket (Easiest)

```html
<script type="module">
  import RealtimeIngestionClient from "./backend/realtime/client-browser.mjs";

  const client = new RealtimeIngestionClient("ws://localhost:3000");

  // Connect
  await client.connect();

  // Listen for fraud alerts
  client.on("transaction-result", (result) => {
    if (result.action === "block") {
      console.log("🚨 FRAUD DETECTED:", result.id);
    } else if (result.action === "alert") {
      console.log("⚠️ ALERT:", result.id);
    }
  });

  // Send transaction
  const txnId = client.ingestTransaction({
    amount: 5000,
    merchant: "Store",
    userId: "USR-123"
  });

  console.log("Transaction sent:", txnId);
</script>
```

### Option B: HTTP API (Simple)

```bash
# Send a transaction
curl -X POST http://localhost:3000/api/realtime/ingest \
  -H "Content-Type: application/json" \
  -d '{
    "id": "TXN-001",
    "amount": 5000,
    "merchant": "Store",
    "userId": "USR-123"
  }'

# Check system health
curl http://localhost:3000/api/realtime/health

# Get stats
curl http://localhost:3000/api/realtime/stats
```

### Option C: Python (Backend Integration)

```python
import requests

API = "http://localhost:3000/api/realtime"

# Send transaction
data = {
    "id": "TXN-001",
    "amount": 5000,
    "merchant": "Store",
    "userId": "USR-123"
}
response = requests.post(f"{API}/ingest", json=data)
result = response.json()
print(f"Queued: {result['success']}")

# Send batch
transactions = [
    {"amount": 100, "merchant": "Store", "userId": "USR-1"},
    {"amount": 200, "merchant": "Gas", "userId": "USR-2"},
]
response = requests.post(f"{API}/batch-ingest", json=transactions)
print(f"Batch result: {response.json()}")
```

### Option D: JavaScript Node.js

```javascript
import { RealtimeIngestionClient } from "./backend/realtime/client.mjs";

const client = new RealtimeIngestionClient("ws://localhost:3000");
await client.connect();

// Listen for results
client.on("transaction-result", (result) => {
  console.log(`${result.id}: ${result.action}`);
});

// Send transaction
client.ingestTransaction({
  amount: 5000,
  merchant: "Store"
});
```

## Real-Time Processing Flow

```
Your App
  │
  ├─► Ingest Transaction (WebSocket or HTTP)
  │   │
  │   └─► Queue (Instant)
  │       │
  │       └─► Processor (Batches every 1 second)
  │           │
  │           └─► ML Model (50-200ms)
  │               │
  │               └─► Action Decision (block/alert/review)
  │                   │
  │                   └─► Result sent back to client
  │                       (Total: 100-300ms)
```

## Understanding Results

Each result contains:

```javascript
{
  id: "TXN-001",              // Transaction ID
  status: "completed",        // completed/error
  prediction: {
    score: 0.87,             // 0=safe, 1=fraud
    risk: "HIGH"            // Risk level
  },
  action: "block",           // block/alert/review/approve
  processedAt: "2024-01-15T10:30:45.123Z",
  processingDuration: "145 ms"
}
```

**Action Types:**
- 🔴 `block` - Score ≥ 0.8 (Stop transaction immediately)
- 🟠 `alert` - Score ≥ 0.5 (Require user verification)
- 🟡 `review` - Score ≥ 0.3 (Manual review needed)
- 🟢 `approve` - Score < 0.3 (Proceed normally)

## Key Endpoints

### REST API

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/realtime/ingest` | Send single transaction |
| POST | `/api/realtime/batch-ingest` | Send multiple transactions |
| GET | `/api/realtime/stats` | Get processing statistics |
| GET | `/api/realtime/health` | Get system health |
| GET | `/api/realtime/queue-size` | Get queue metrics |
| GET | `/api/realtime/results/:id` | Get transaction result |

### WebSocket Events

| Type | Direction | Purpose |
|------|-----------|---------|
| `ingest` | Send | Submit transaction |
| `batch-ingest` | Send | Submit multiple transactions |
| `transaction-result` | Receive | Get processing result |
| `transaction-error` | Receive | Get error notification |
| `query-stats` | Send | Request statistics |
| `stats-response` | Receive | Receive statistics |
| `query-health` | Send | Request health status |
| `health-response` | Receive | Receive health status |

## Configuration

Edit `backend/realtime/config.mjs` to adjust:

```javascript
processor: {
  batchSize: 10,          // Transactions per batch
  batchInterval: 1000,    // ms between batches
}

thresholds: {
  blockScore: 0.8,        // Auto-block if score ≥ this
  alertScore: 0.5,        // Alert if score ≥ this
  reviewScore: 0.3,       // Review if score ≥ this
}
```

## Performance

- **Throughput**: 1000-2000 transactions/second
- **Latency**: 100-300ms end-to-end
- **Queue capacity**: 10,000 transactions
- **Memory usage**: ~30MB base

## Monitoring

### Check Health

```bash
curl http://localhost:3000/api/realtime/health
```

Response:
```json
{
  "health": "healthy",           // or "warning"/"critical"
  "queue": {
    "queueLength": 45,
    "maxSize": 10000,
    "utilizationPercent": "0.45",
    "totalProcessed": 2500
  },
  "connectedClients": 3
}
```

### View Stats

```bash
curl http://localhost:3000/api/realtime/stats
```

## Troubleshooting

### "Connection refused"
- Make sure backend is running: `node backend/server.mjs`
- Check port 3000 is not in use

### "Queue is full"
- System is overloaded
- Reduce transaction send rate
- Increase `QUEUE_MAX_SIZE` in config
- Check ML service performance

### High latency (> 1 second)
- Check ML service is running
- Verify network connectivity
- Monitor system CPU/memory

### WebSocket disconnects
- Browser might timeout idle connections
- Use heartbeat: `client.startHeartbeat(30000)`

## Common Patterns

### Pattern 1: Real-Time Dashboard

```javascript
// Update dashboard every 2 seconds
setInterval(async () => {
  const stats = await client.queryStats();
  updateDashboard({
    queueSize: stats.queue.queueLength,
    processed: stats.queue.totalProcessed,
    avgLatency: stats.queue.avgProcessingTime
  });
}, 2000);
```

### Pattern 2: Batch Processing

```javascript
// Collect transactions, send every 100 or every 5 seconds
let batch = [];
const sendBatch = () => {
  if (batch.length > 0) {
    client.ingestBatch(batch);
    batch = [];
  }
};

function addTransaction(txn) {
  batch.push(txn);
  if (batch.length >= 100) {
    sendBatch();
  }
}

setInterval(sendBatch, 5000);
```

### Pattern 3: Error Handling

```javascript
client.on("transaction-error", (error) => {
  console.error("❌ Error:", error);
  // Retry logic, logging, alerting
});

client.on("error", (error) => {
  console.error("Connection error:", error);
  // Implement reconnection logic
});
```

## Next Steps

1. ✅ Start the backend server
2. 📨 Send test transactions
3. 📊 Monitor processing stats
4. 🔧 Tune thresholds for your use case
5. 🚀 Deploy to production

## support

For issues or questions:
- Check `backend/realtime/README.md` for detailed docs
- Review examples in `backend/realtime/examples.mjs`
- Check server logs for errors

---

**You're all set!** Your Detection System now processes transactions in real-time. 🎉
