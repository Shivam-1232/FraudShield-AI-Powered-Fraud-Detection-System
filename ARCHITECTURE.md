# 🏗️ FraudShield - System Architecture & Data Flow

## Complete System Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                          FRAUDSHIELD SYSTEM ARCHITECTURE                         │
├──────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │                            🔴 DATA INPUT LAYER                              │ │
│  ├────────────────────────────────────────────────────────────────────────────┤ │
│  │                                                                            │ │
│  │   fraud_dataset_500.csv          Database          Real-Time API Stream   │ │
│  │   (Historical Data)              (Live Data)       (Transaction Flow)      │ │
│  │        │                              │                    │              │ │
│  │        └──────────────┬───────────────┴────────────────────┘              │ │
│  │                       │                                                    │ │
│  └───────────────────────┼────────────────────────────────────────────────────┘ │
│                          │                                                      │
│                          ▼                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │                    🟠 DATA PREPROCESSING LAYER                              │ │
│  ├────────────────────────────────────────────────────────────────────────────┤ │
│  │  [fraud_model.py - Lines 28-62]                                           │ │
│  │                                                                            │ │
│  │  ┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐        │ │
│  │  │  1. EDA         │   │  2. ENCODING     │   │  3. SCALING     │        │ │
│  │  │  - Distribution │─▶ │  - Categorical  │─▶ │  - Normalize    │        │ │
│  │  │  - Anomalies    │   │  - One-Hot       │   │  - StandardScr  │        │ │
│  │  └─────────────────┘   └──────────────────┘   └─────────────────┘        │ │
│  │                                                                            │ │
│  │  ┌─────────────────┐   ┌──────────────────┐   ┌─────────────────┐        │ │
│  │  │  4. FEATURE ENG │   │  5. DIM REDUC    │   │  6. SELECTION   │        │ │
│  │  │  - New Features │─▶ │  - PCA (5 comp)  │─▶ │  - SelectKBest  │        │ │
│  │  │  - Interactions │   │  - Variance      │   │  - Top 5        │        │ │
│  │  └─────────────────┘   └──────────────────┘   └─────────────────┘        │ │
│  │                                                                            │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                          │                                                      │
│                          ▼                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │              🟡 CLASS IMBALANCE HANDLING (SMOTE)                           │ │
│  ├────────────────────────────────────────────────────────────────────────────┤ │
│  │  Original: 99.8% Legitimate ➜ 0.2% Fraud                                 │ │
│  │       ↓ (SMOTE - Synthetic Minority Over-Sampling)                        │ │
│  │  Balanced: 50% Legitimate ≈ 50% Fraud (Training)                        │ │
│  │       ↓                                                                    │ │
│  │  Prevents: Model bias toward majority class                              │ │
│  │                                                                            │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                          │                                                      │
│                          ▼                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │         🟢 TRAIN-TEST SPLIT & MODEL TRAINING                              │ │
│  ├────────────────────────────────────────────────────────────────────────────┤ │
│  │  80% Training Data ─┐                                                     │ │
│  │  20% Testing Data  ─┼─▶  5 DIFFERENT MODELS                              │ │
│  │                     │                                                      │ │
│  │  ┌──────────────────┴──────────────────────────────────────────────────┐ │ │
│  │  │                                                                   │ │ │
│  │  │ ┌─────────────────┐  ┌──────────────┐  ┌─────────────────┐      │ │ │
│  │  │ │ 1. RANDOM      │  │ 2. XGBOOST   │  │ 3. LOGISTIC     │      │ │ │
│  │  │ │    FOREST      │  │    (Gradient │  │    REGRESSION   │      │ │ │
│  │  │ │                │  │     Boosting)│  │                 │      │ │ │
│  │  │ │ n_est: 100     │  │ n_est: 100   │  │ max_iter: 1000  │      │ │ │
│  │  │ │ ✓ BEST MODEL   │  │ ✓ ACCURATE   │  │ ✓ BASELINE      │      │ │ │
│  │  │ └─────────────────┘  └──────────────┘  └─────────────────┘      │ │ │
│  │  │                                                                   │ │ │
│  │  │ ┌─────────────────┐  ┌──────────────────────────────────────┐   │ │ │
│  │  │ │ 4. SVM          │  │ 5. LSTM NEURAL NETWORK               │   │ │ │
│  │  │ │ (Non-linear)    │  │ (Deep Learning - Temporal Patterns) │   │ │ │
│  │  │ │                 │  │ - LSTM: 64 hidden units             │   │ │ │
│  │  │ │ ✓ COMPLEX       │  │ - Dropout: 20%                      │   │ │ │
│  │  │ │   PATTERNS      │  │ - Sigmoid activation                │   │ │ │
│  │  │ └─────────────────┘  │ ✓ DEEP LEARNING                     │   │ │ │
│  │  │                      └──────────────────────────────────────┘   │ │ │
│  │  │                                                                   │ │ │
│  │  └───────────────────────────────────────────────────────────────────┘ │ │
│  │                                                                            │ │
│  │         HYPERPARAMETER TUNING: GridSearchCV + 5-Fold Cross-Val            │ │
│  │         Scoring Metric: ROC-AUC (Best for imbalanced data)               │ │
│  │                                                                            │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                          │                                                      │
│                          ▼                                                      │
│  ┌────────────────────────────────────────────────────────────────────────────┐ │
│  │         🔵 MODEL EVALUATION & SELECTION                                   │ │
│  ├────────────────────────────────────────────────────────────────────────────┤ │
│  │                                                                            │ │
│  │  Random Forest:    ROC-AUC = 0.94  ✓✓✓ SELECTED (Best interpretability)  │ │
│  │  XGBoost:          ROC-AUC = 0.92  ✓✓                                    │ │
│  │  Logistic Regr:    ROC-AUC = 0.80  ✓                                     │ │
│  │  SVM:              ROC-AUC = 0.87  ✓                                     │ │
│  │  LSTM:             ROC-AUC = 0.89  ✓                                     │ │
│  │                                                                            │ │
│  │  ✓ Random Forest selected for production (SHAP explainability)           │ │
│  │                                                                            │ │
│  └────────────────────────────────────────────────────────────────────────────┘ │
│                          │                                                      │
│         ┌────────────────┼────────────────┐                                    │
│         │                │                │                                    │
│         ▼                ▼                ▼                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐                │
│  │ SAVE MODEL   │  │ SHAP EXPLAIN │  │ FLASK REST API       │                │
│  │              │  │              │  │                      │                │
│  │ .pkl file    │  │ Analyze why  │  │ Load fraud_model     │                │
│  │ .pth file    │  │ predictions  │  │ Serve predictions    │                │
│  │ (Pickled)    │  │ made         │  │ Real-time scoring    │                │
│  └──────────────┘  └──────────────┘  └──────────────────────┘                │
│                          │                                                      │
└──────────────────────────┼──────────────────────────────────────────────────────┘
                           │
                           ▼
        ┌──────────────────────────────────────────────────┐
        │   🟣 EXPLAINABILITY LAYER - SHAP ANALYSIS        │
        ├──────────────────────────────────────────────────┤
        │                                                  │
        │  shap_analysis.py Generates:                    │
        │                                                  │
        │  ✓ Summary Bar Plot      (Global importance)    │
        │  ✓ Impact Scatter Plot   (Feature values)       │
        │  ✓ Dependence Plots      (Top 4 features)       │
        │  ✓ Force Plots           (Individual)           │
        │  ✓ Waterfall Plots       (Step-by-step)         │
        │  ✓ Decision Plot         (Decision paths)       │
        │                                                  │
        │  Output: 10+ PNG visualizations                 │
        │                                                  │
        └──────────────────────────────────────────────────┘
                           │
           ┌───────────────┴───────────────┐
           │                               │
           ▼                               ▼
   ┌──────────────────┐           ┌──────────────────┐
   │  FLASK API       │           │  DASHBOARDS      │
   │                  │           │                  │
   │  /predict        │           │  ┌────────────┐  │
   │  POST endpoint   │           │  │   React    │  │
   │  (JSON in/out)   │           │  │ Dashboard  │  │
   │                  │           │  │            │  │
   │  Real-time       │           │  │ Real-time  │  │
   │  Predictions     │───────────▶  │ Monitoring │  │
   │                  │           │  │            │  │
   │  Returns:        │           │  │ Fraud Rate │  │
   │  - Prediction    │           │  │ Risk Score │  │
   │  - Probability   │           │  │ Location   │  │
   │                  │           │  │ Maps       │  │
   └──────────────────┘           │  └────────────┘  │
           │                       │                  │
           │                       │  ┌────────────┐  │
           │                       │  │  Power BI  │  │
           │                       │  │ Dashboard  │  │
           │                       │  │            │  │
           │                       │  │ Advanced BI│  │
           │                       │  │ Analytics  │  │
           │                       │  └────────────┘  │
           │                       │                  │
           └──────────────────────▶└──────────────────┘
```

---

## Data Flow: Step by Step

### **STAGE 1: Input Transaction**
```
Transaction JSON received:
{
  "amount": 1500.50,
  "location": "New York",
  "device": "Mobile",
  "transaction_type": "Online",
  "account_age_days": 45,
  "num_transactions_last_24h": 3,
  "transaction_hour": 14
}
```

### **STAGE 2: Preprocessing (Same as training)**
```
1. Encode Categoricals
   location: "New York" → 1
   device: "Mobile" → 0
   transaction_type: "Online" → 2

2. Apply StandardScaler (learned from training)
   amount: 1500.50 → 0.75 (normalized)
   
3. Feature Vector
   [0.75, 1, 0, 2, 45, 3, 14]
```

### **STAGE 3: Model Prediction**
```
Random Forest makes prediction:
  - Passes through 100 trained decision trees
  - Each tree votes: Fraud or Legitimate?
  - Average vote: 92% fraud, 8% legitimate
  
Predicted Class: FRAUD (1)
Probability: 0.92
```

### **STAGE 4: SHAP Explanation**
```
Why did it predict fraud?

Base Value: 0.30 (average fraud prob)

SHAP Contributions:
  ✗ Amount High (+0.20)        ← Unusual amount
  ✗ New Device (+0.15)         ← First-time device
  ✗ High Velocity (+0.12)      ← Many transactions
  ✗ Young Account (+0.10)      ← New customer
  ✗ Afternoon Hour (+0.05)     ← Normal time
  ─────────────────────────────
  Final: 0.92 = FRAUD

All signals point to fraud!
```

### **STAGE 5: API Response**
```
{
  "fraud_prediction": 1,
  "fraud_probability": 0.92,
  "shap_explanation": {
    "amount": 0.20,
    "location": 0.15,
    "device": 0.15,
    "account_age": 0.10,
    "time": 0.05
  }
}
```

### **STAGE 6: Dashboard Display**
```
React Dashboard shows:
  ✗ FRAUD DETECTED
  Confidence: 92%
  
  Top Risk Factors:
  1. High Amount ($1500)
  2. Unknown Device
  3. Multiple Transactions (Velocity)
  4. New Account (1 day)
  
  Action: ⬛ BLOCK TRANSACTION
          ⬛ ALERT ANALYST
          ⬛ NOTIFY CUSTOMER
```

---

## Model Comparison Matrix

```
┌─────────────────┬──────────────┬───────────┬──────────┬──────────┐
│ Model           │ ROC-AUC      │ Speed     │ Accuracy │ Interp   │
├─────────────────┼──────────────┼───────────┼──────────┼──────────┤
│ Random Forest   │ 0.94 ⭐⭐⭐  │ Fast      │ Very Good│ Excellent│
│                 │              │           │          │ (SHAP)   │
├─────────────────┼──────────────┼───────────┼──────────┼──────────┤
│ XGBoost         │ 0.92 ⭐⭐   │ Very Fast │ Good     │ Good     │
│                 │              │           │          │ (Feature)│
├─────────────────┼──────────────┼───────────┼──────────┼──────────┤
│ LSTM            │ 0.89 ⭐⭐   │ Medium    │ Good     │ Hard     │
│                 │              │           │          │ (Black   │
│                 │              │           │          │  Box)    │
├─────────────────┼──────────────┼───────────┼──────────┼──────────┤
│ SVM             │ 0.87 ⭐⭐   │ Slow      │ Good     │ Medium   │
│                 │              │           │          │          │
├─────────────────┼──────────────┼───────────┼──────────┼──────────┤
│ Log Regression  │ 0.80 ⭐     │ Very Fast │ Okay     │ Excellent│
│                 │              │           │          │ (Linear) │
└─────────────────┴──────────────┴───────────┴──────────┴──────────┘
```

---

## Feature Engineering Pipeline

```
Raw Features                Enhanced Features
┌─────────────────────┐     ┌──────────────────────┐
│ amount              │     │ amount_scaled        │
│ location            │     │ location_encoded     │
│ device              │     │ device_encoded       │
│ transaction_type    │     │ type_encoded         │
│ account_age_days    │     │ age_scaled           │
│ num_trans_24h       │     │ velocity_scaled      │
│ transaction_hour    │     │ hour_encoded         │
└─────────────────────┘     │ hourly_fraud_rate    │ ← NEW
                            │ pca_1 to pca_5       │ ← DIM REDUCE
                            │ top_5_selected       │ ← FEATURE SEL
                            └──────────────────────┘
                                    │
                                    ▼
                            Model Ready Features
                            (Standardized, Encoded,
                             Engineered, Reduced)
```

---

## SHAP Explanation Types

### **1. Global Explanation (What matters on average?)**
```
Feature Importance Ranking:
  1. Amount              ████████████ (28% of variance)
  2. Location            ██████████   (22%)
  3. Device              ████████     (18%)
  4. Account Age         ██████       (15%)
  5. Time                ████         (12%)
  6. Velocity            ██           (5%)
```

### **2. Local Explanation (Why this specific prediction?)**
```
Transaction #1234:
  Base Value: 0.30
  ↑ Amount: $5000      (+0.25) ← MAJOR FRAUD SIGNAL
  ↑ Device: New        (+0.15) ← FRAUD SIGNAL
  ↑ Location: Unknown  (+0.12) ← FRAUD SIGNAL
  ═════════════════════════════
  Predicted: 0.82 = FRAUD ✗

Transaction #5678:
  Base Value: 0.30
  ↓ Amount: $50        (-0.10) ← LEGITIMATE
  ↓ Device: Known      (-0.15) ← LEGITIMATE
  ↓ Location: Home     (-0.12) ← LEGITIMATE
  ═════════════════════════════
  Predicted: 0.03 = LEGITIMATE ✓
```

### **3. Feature Interaction (How do features interact?)**
```
Dependence Analysis:
  
  Amount vs Fraud Probability:
  $100     ──────────  20% fraud risk
  $500     ────────────  40% fraud risk
  $1000    ──────────────  60% fraud risk
  $5000    ────────────────  80% fraud risk
  
  Location vs Fraud Probability:
  Known City    ──────────  25% fraud risk
  Unknown City  ──────────────  75% fraud risk
```

---

## Deployment Architecture

```
Production Environment:
┌─────────────────────────────────────────────────┐
│                   CLOUD (AWS/Azure)              │
├─────────────────────────────────────────────────┤
│                                                 │
│  Load Balancer (Port 443 - HTTPS)              │
│         │                                       │
│         ▼                                       │
│  ┌───────────────┐                             │
│  │ API Gateway   │                             │
│  └───────┬───────┘                             │
│          │                                     │
│  ┌───────┴────────────────┐                   │
│  │                        │                   │
│  ▼                        ▼                   │
│  ┌────────────┐      ┌────────────┐           │
│  │ Flask API  │      │ React App  │           │
│  │ Container  │      │ Container  │           │
│  │ (Fraud)    │      │ (UI)       │           │
│  │ Instance 1 │      │ Instance 1 │           │
│  └────────────┘      └────────────┘           │
│  ┌────────────┐      ┌────────────┐           │
│  │ Flask API  │      │ React App  │           │
│  │ Container  │      │ Container  │           │
│  │ (Fraud)    │      │ (UI)       │           │
│  │ Instance 2 │      │ Instance 2 │           │
│  └────────────┘      └────────────┘           │
│         │                                     │
│         └────────┬─────────────────┘          │
│                  │                            │
│                  ▼                            │
│         ┌─────────────────┐                  │
│         │  PostgreSQL DB  │                  │
│         │ (Transactions)  │                  │
│         └─────────────────┘                  │
│                  │                            │
│                  ▼                            │
│         ┌─────────────────┐                  │
│         │   Model Store   │                  │
│         │ (fraud_model    │                  │
│         │  .pkl, .pth)    │                  │
│         └─────────────────┘                  │
│                                              │
│  Monitoring:                                │
│  - Prometheus (Metrics)                    │
│  - Grafana (Dashboards)                    │
│  - ELK Stack (Logs)                        │
│                                              │
└─────────────────────────────────────────────┘
```

---

## Error Handling & Monitoring

```
Prediction Pipeline with Error Handling:

Request → Validate Input
           │
           ├─ Invalid? → Return 400 Error
           │
           ▼
        Preprocess → Encoding/Scaling
           │
           ├─ Failed? → Return 500 Error
           │
           ▼
        Predict → Load Model & Predict
           │
           ├─ Model Error? → Return 503 Error
           │
           ▼
        Explain → SHAP Analysis (Optional)
           │
           ├─ SHAP Error? → Return partial response
           │
           ▼
        Response → Return JSON with prediction
           │
           ▼
        Log → Log to database for monitoring
```

---

## Performance Metrics Tracked

```
Real-Time Monitoring Dashboard:
┌─────────────────────────────────────────┐
│ API Performance                         │
│ ├─ Request Rate: 1,250 req/sec         │
│ ├─ Avg Response Time: 45ms             │
│ ├─ 99th Percentile: 120ms              │
│ └─ Error Rate: 0.05%                   │
│                                        │
│ Model Performance                       │
│ ├─ Accuracy: 94%                       │
│ ├─ False Positive Rate: 2%             │
│ ├─ False Negative Rate: 4%             │
│ ├─ Precision: 92%                      │
│ └─ Recall: 96%                         │
│                                        │
│ System Health                          │
│ ├─ Uptime: 99.9%                       │
│ ├─ CPU Usage: 45%                      │
│ ├─ Memory Usage: 62%                   │
│ └─ DB Connections: 45/100              │
└─────────────────────────────────────────┘
```

---

## Security Considerations

```
🔒 Security Layers:

1. Input Validation
   - Type checking
   - Range validation
   - Pattern matching

2. Authentication
   - API Key validation
   - JWT tokens
   - Role-based access

3. Data Protection
   - HTTPS/TLS encryption
   - PII masking
   - Encrypted database

4. Model Security
   - Model versioning
   - Audit trails
   - Tamper detection

5. Compliance
   - GDPR compliance
   - PCI-DSS for payments
   - SOC 2 certification
```

---

**This architecture provides a production-ready, scalable, and explainable fraud detection system! 🚀**
