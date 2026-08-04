# Data Generator Integration Guide

## Overview

The Detection System now includes a data generator module that creates synthetic transaction data. This eliminates dependency on static CSV files and provides unlimited, customizable data for training and prediction.

## What Was Added

### 1. New Folder: `data-generator/`

Located at: `Detection-System/data-generator/`

**Files:**
- `generator.py` - Main data generation module
- `config.py` - Centralized configuration
- `api.py` - API interface for backend integration
- `__init__.py` - Python package initialization
- `README.md` - Comprehensive documentation

### 2. Updated Files

#### `anomaly-detection/train.py`
- Now imports `get_data_from_generator` from the data generator
- Added `DATA_SOURCE` configuration variable (default: `'generator'`)
- Can switch between generated and CSV data by changing one variable
- Default: Generates 50,000 training transactions with 0.1% fraud ratio

#### `anomaly-detection/predict.py`
- Now imports `get_data_from_generator` from the data generator
- Generates data with user IDs for multi-user scenarios
- Default: Generates 10,000 prediction transactions with 10 unique users
- Handles both generated data and static CSV fallback

## How to Use

### Option 1: Use Generated Data (Default)

No changes needed! The system is configured to use the data generator by default.

To train the model with generated data:
```bash
cd Detection-System/anomaly-detection
python train.py
```

To run prediction with generated data:
```bash
cd Detection-System/anomaly-detection
python predict.py
```

### Option 2: Switch to CSV Data

Edit the `DATA_SOURCE` variable in the scripts:

**In `train.py`:**
```python
DATA_SOURCE = 'csv'  # Change from 'generator' to 'csv'
```

**In `predict.py`:**
```python
DATA_SOURCE = 'csv'  # Change from 'generator' to 'csv'
```

### Option 3: Customize Generation Parameters

Edit the `data-generator/config.py` file:

```python
TRAINING = {
    'data_source': 'generator',
    'n_samples': 100000,  # Increase sample size
    'fraud_ratio': 0.002,  # Increase fraud rate
    'random_seed': 42,
}

PREDICTION = {
    'data_source': 'generator',
    'n_samples': 20000,
    'fraud_ratio': 0.0015,
    'use_user_ids': True,
    'n_users': 50,  # More users
    'random_seed': 123,
}
```

## API for Backend Integration

The `data-generator/api.py` provides an easy interface for the backend:

```python
from data_generator.api import get_data_api

api = get_data_api()

# Get training data
train_data = api.get_training_data()

# Get prediction data
pred_data = api.get_prediction_data()

# Get data stats
stats = api.get_data_stats(train_data)
print(f"Total transactions: {stats['total_transactions']}")
print(f"Fraud count: {stats['fraud_count']}")

# Generate batch stream for real-time processing
batch_stream = api.get_batch_stream(batch_size=100)
batch = next(batch_stream)

# Get user-specific transactions
user_data = api.generate_user_transactions(n_samples=500)

# Export to CSV if needed
api.export_to_csv(train_data, 'exported_data.csv')
```

## Data Structure

Generated data includes these columns:

| Column | Type | Description |
|--------|------|-------------|
| Time | float | Transaction time (0-172800 seconds) |
| Amount | float | Transaction amount in dollars |
| V1-V28 | float | PCA-transformed feature vectors |
| Class | int | Target: 0=legitimate, 1=fraud |
| user_id | int | User identifier (optional) |

## Configuration Points

### 1. Data Source
Change where data comes from:
```python
DATA_SOURCE = 'generator'  # or 'csv'
```

### 2. Sample Size
Control how much data to generate:
```python
n_samples = 50000  # or any number
```

### 3. Fraud Ratio
Set the proportion of fraudulent transactions:
```python
fraud_ratio = 0.001  # 0.1% fraud
fraud_ratio = 0.002  # 0.2% fraud
```

### 4. Random Seed
Enable reproducible generation:
```python
random_seed = 42  # Produces same data every time
random_seed = None  # Different data each time
```

## Advantages

✅ **Unlimited Data** - Generate as much as needed  
✅ **Realistic Patterns** - Simulates real transaction behavior  
✅ **Customizable** - Adjust fraud ratio, volume, user distribution  
✅ **Reproducible** - Same seed = same data  
✅ **No Privacy Issues** - Fully synthetic data  
✅ **Fast** - Instant generation vs. loading from disk  
✅ **Scalable** - Can generate in batches for streaming  

## Troubleshooting

### "ModuleNotFoundError: No module named 'generator'"

Make sure you're running scripts from the correct directory or the path is correctly set in the imports.

### Different Results Each Time

Set `random_seed` to a fixed value in the configuration or method calls.

### Memory Issues with Large Datasets

Use batch generation instead:
```python
from data_generator.generator import TransactionDataGenerator

gen = TransactionDataGenerator()
batch_stream = gen.generate_batch(batch_size=10000)

for i in range(100):  # Process 100 batches
    batch = next(batch_stream)
    # Process batch without keeping all data in memory
```

## Next Steps

1. **Test the Generator**
   ```bash
   python data-generator/generator.py
   ```

2. **Run Training with Generated Data**
   ```bash
   python anomaly-detection/train.py
   ```

3. **Run Prediction**
   ```bash
   python anomaly-detection/predict.py
   ```

4. **Integrate with Backend** (mlservice.js)
   - Update `backend/services/mlservice.js` to call Python scripts
   - Use the data API for on-demand data generation

## Questions or Issues?

Refer to:
- `data-generator/README.md` - Detailed technical documentation
- `data-generator/config.py` - All configuration options
- `data-generator/api.py` - Backend integration examples
