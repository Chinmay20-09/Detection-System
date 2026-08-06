# 🚀 Quick Start Guide - Run Your System in 5 Minutes

Follow these exact steps to see your fraud detection system in action.

## Step 1: Check Your Setup (2 minutes)

Open a PowerShell terminal and verify you have everything:

```powershell
# Check Python
python --version
# Expected: Python 3.8 or higher

# Check Node.js
node --version
npm --version
# Expected: Both installed

# Check you're in the right directory
cd c:\Detection-System
ls
# Expected: See ML model and other files
```

## Step 2: Start ML Service (1 minute)

Open Terminal 1:

```powershell
cd c:\Detection-System\ml-service
python service.py
```

✅ **Wait for this message:**
```
✅ Model loaded successfully
🚀 Starting ML Service on http://localhost:5000
```

If you see errors, check:
- Model exists: `ls model.pkl`
- Dependencies: `pip install -r requirements.txt`

## Step 3: Start Node.js Backend (1 minute)

Open Terminal 2:

```powershell
cd c:\Detection-System
npm start
```

✅ **Wait for these messages:**
```
🚀 Server running on http://localhost:3000
✅ ML Service connected and ready
```

If you see "ML Service connection failed", go back and check ML Service started.

## Step 4: Verify Everything Works (30 seconds)

Open Terminal 3:

```powershell
# Check system health
curl http://localhost:3000/status | jq

# Should show green ✅ for all components
```

Or just open browser and go to: `http://localhost:3000/status`

## Step 5: Send Your First Fraud Detection Request (30 seconds)

### Option A: Safe Transaction (Should Pass ✅)

Terminal 3:
```powershell
curl -X POST http://localhost:3000/api/realtime/ingest `
  -H "Content-Type: application/json" `
  -d '{
    "id": "TXN-SAFE-001",
    "amount": 50,
    "merchant": "Coffee Shop",
    "userId": "USR-123"
  }' | jq
```

Expected response:
```json
{
  "success": true,
  "action": "approve",
  "score": 0.15,
  "message": "Transaction approved - Low fraud risk"
}
```

### Option B: Suspicious Transaction (Should Alert 🟠)

Terminal 3:
```powershell
curl -X POST http://localhost:3000/api/realtime/ingest `
  -H "Content-Type: application/json" `
  -d '{
    "id": "TXN-ALERT-001",
    "amount": 5000,
    "merchant": "Unknown Online",
    "userId": "USR-123"
  }' | jq
```

Expected response:
```json
{
  "success": true,
  "action": "alert",
  "score": 0.65,
  "message": "Transaction flagged - Verify with customer"
}
```

### Option C: Highly Fraudulent (Should Block 🔴)

Terminal 3:
```powershell
curl -X POST http://localhost:3000/api/realtime/ingest `
  -H "Content-Type: application/json" `
  -d '{
    "id": "TXN-FRAUD-001",
    "amount": 15000,
    "merchant": "Overseas Unknown",
    "userId": "USR-NEW"
  }' | jq
```

Expected response:
```json
{
  "success": true,
  "action": "block",
  "score": 0.95,
  "message": "Transaction blocked - High fraud risk"
}
```

## Step 6: Monitor Real-Time Stats (Ongoing)

Terminal 3:
```powershell
# Watch queue in real-time
while ($true) {
    curl -s http://localhost:3000/api/realtime/stats | jq '.queue'
    Start-Sleep -Seconds 2
}
```

Or simpler:
```powershell
# Single check
curl http://localhost:3000/api/realtime/stats | jq
```

Look for:
- `queueLength` - How many pending transactions
- `totalProcessed` - Total processed so far
- `averageLatency` - How fast predictions are

## Step 7: Test Batch Processing (Optional)

Terminal 3:
```powershell
curl -X POST http://localhost:3000/api/realtime/batch-ingest `
  -H "Content-Type: application/json" `
  -d '[
    {"amount": 50, "merchant": "Store 1"},
    {"amount": 100, "merchant": "Store 2"},
    {"amount": 5000, "merchant": "Online"},
    {"amount": 20000, "merchant": "Overseas"}
  ]' | jq
```

## Step 8: Monitor Performance (Optional)

Terminal 3 - Watch these grow in real-time:

```powershell
# Backend statistics
curl http://localhost:3000/api/realtime/stats | jq '.queue.totalProcessed'

# ML Service statistics  
curl http://localhost:5000/stats | jq '.predictions_made'

# Queue depth
curl http://localhost:3000/api/realtime/queue-size | jq
```

## What You Should See

### When Everything Works ✅

Terminal 1 (ML Service):
```
[ML Service] Prediction request: Processing batch of 1
[ML Service] Prediction complete: fraud_score = 0.25
```

Terminal 2 (Backend):
```
Transaction: ID-123, Amount: $50, Risk: LOW ✅
Transaction: ID-124, Amount: $5000, Risk: MEDIUM ⚠️
Transaction: ID-125, Amount: $15000, Risk: HIGH 🔴
```

Terminal 3 (Your Testing):
```
response: {
  "success": true,
  "action": "approve",
  ✅ "score": 0.15,
  "id": "TXN-SAFE-001"
}
```

### If Something Goes Wrong ❌

**ML Service won't start:**
```powershell
# Make sure model exists
ls ml-service\model.pkl

# Install dependencies
pip install -r ml-service\requirements.txt

# Try again
python ml-service\service.py
```

**Backend can't find ML Service:**
```powershell
# Check ML Service is running
curl http://localhost:5000/health

# Check backend logs for connection retries
# Wait 10 seconds and restart backend
npm start
```

**Predictions failing:**
```powershell
# Test ML Service directly
curl http://localhost:5000/health | jq

# Test prediction directly
curl -X POST http://localhost:5000/predict `
  -H "Content-Type: application/json" `
  -d '{"amount": 100}' | jq
```

## Performance Baseline

After 10-20 transactions, you should see:

| Metric | Expected |
|---------|----------|
| Average latency | 50-150ms |
| Throughput | 100+ txn/sec |
| Queue depth | 0-2 (mostly empty) |
| Success rate | 100% |

## Next: Load Testing (Optional)

Send many transactions at once:

```powershell
# Send 50 transactions in rapid fire
for ($i = 0; $i -lt 50; $i++) {
    curl -X POST http://localhost:3000/api/realtime/ingest `
      -H "Content-Type: application/json" `
      -d "{
        \"amount\": $(Get-Random -Minimum 10 -Maximum 10000),
        \"merchant\": \"Store-$i\"
      }" | jq '.action' 2>$null
}

# Then check stats
curl http://localhost:3000/api/realtime/stats | jq
```

Expected: All 50 should process within seconds, queue never exceeds 20.

## Next: Real-Time WebSocket (Optional)

Create a file `test-websocket.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Fraud Detection - Real-Time</title>
</head>
<body>
    <h1>Real-Time Transactions</h1>
    <div id="results"></div>
    
    <script>
        const ws = new WebSocket("ws://localhost:3000");
        const resultsDiv = document.getElementById("results");
        
        ws.onmessage = (event) => {
            const result = JSON.parse(event.data);
            if (result.type === "transaction-result") {
                const actions = {
                    "approve": "✅",
                    "review": "⚠️",
                    "alert": "🟠",
                    "block": "🔴"
                };
                resultsDiv.innerHTML += `
                    <p>${actions[result.action]} 
                       ${result.id}: $${result.amount} 
                       (Risk: ${(result.score * 100).toFixed(0)}%)</p>
                `;
            }
        };
    </script>
</body>
</html>
```

Open in browser and it will show transactions in real-time!

## Stopping Everything

When you're done:

```powershell
# Terminal 1 & 2: Press Ctrl+C
# Or kill specific processes:
Get-Process | Where-Object {$_.ProcessName -like "python" -or $_.ProcessName -like "node"} | Stop-Process
```

## Success! 🎉

If you got here and saw responses, your fraud detection system is working!

**What you've verified:**
- ✅ ML Service loads models and makes predictions
- ✅ Backend processes transactions in real-time
- ✅ Latency is under 300ms
- ✅ System can handle multiple transactions
- ✅ Fraud scoring works correctly

## What Happens Behind The Scenes

1. **You send transaction** → HTTP request to backend
2. **Backend receives** → Adds to ingestion queue
3. **Queue processes** → Batches transactions every 1 second
4. **ML Service predicts** → Returns fraud probability (0-1)
5. **Backend determines action**:
   - Score 0-0.3: ✅ APPROVE
   - Score 0.3-0.5: ⚠️ REVIEW  
   - Score 0.5-0.8: 🟠 ALERT
   - Score 0.8-1.0: 🔴 BLOCK
6. **Result sent back** → Response with action and risk score

## Common Questions

**Q: Can I change the fraud thresholds?**
A: Yes! Edit `backend/realtime/config.mjs` (lines with `THRESHOLDS`)

**Q: Can I add more fields to transactions?**
A: Yes! The ML model expects 30 features. Add what you want, unknown fields are ignored.

**Q: Can I retrain the model?**
A: Yes! Run `python anomaly-detection/train.py` with your data, copy `model.pkl` to `ml-service/`

**Q: How do I deploy this to production?**
A: See `DEPLOYMENT_CHECKLIST.md`

**Q: Can I use this with my database?**
A: Yes! Modify the backend routes in `backend/routes/api.js` to fetch/store data

## You're All Set! 🚀

Your fraud detection system is running. Next steps:

1. Experiment with different transaction amounts
2. Load test with 100+ transactions
3. Read `DEPLOYMENT_CHECKLIST.md` for deployment options
4. Configure thresholds for your use case
5. Connect to your real data source

**Questions?** Check:
- `SETUP_GUIDE.md` - Setup & running guide
- `../ml-service/README.md` - ML Service documentation
- `../backend/realtime/README.md` - Real-time ingestion docs
- Source code comments for implementation details

---

**Made with ❤️ for real-time fraud detection**
