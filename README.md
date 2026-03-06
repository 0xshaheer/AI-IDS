# AI-IDS
AI‑Powered Intrusion Detection System (IDS) leveraging the NSL‑KDD dataset. Implements preprocessing, feature encoding, and scaling, trains a Random Forest model, and evaluates performance. Includes real‑time packet capture with Scapy and a Flask dashboard for monitoring alerts and visualizing IDS activity.
---

## 📂 Project Layout

```
AI_IDS/
├── data/
│   ├── train.csv
│   └── test.csv
├── models/
│   ├── random_forest.pkl
│   └── scaler.pkl
├── src/
│   ├── __init__.py
│   ├── preprocess.py
│   ├── train_model.py
│   ├── live_capture.py
│   ├── dashboard.py
├── requirements.txt
├── README.md
```

---

## 📝 File Contents

### `requirements.txt`
```text
pandas
scikit-learn
flask
matplotlib
scapy
joblib
```

---

### `src/preprocess.py`
```python
import pandas as pd
from sklearn.preprocessing import LabelEncoder, StandardScaler

def preprocess_dataset(file_path):
    data = pd.read_csv(file_path)

    target_col = 'labels'  # confirmed from your dataset

    # Encode categorical features
    for col in ['protocol_type', 'service', 'flag']:
        if col in data.columns:
            encoder = LabelEncoder()
            data[col] = encoder.fit_transform(data[col])

    # Encode labels (Normal vs Attack types)
    label_encoder = LabelEncoder()
    data[target_col] = label_encoder.fit_transform(data[target_col])

    X = data.drop(target_col, axis=1)
    y = data[target_col]

    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X)

    return X_scaled, y, scaler
```

---

### `src/train_model.py`
```python
from src.preprocess import preprocess_dataset
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, confusion_matrix
import joblib

# Load and preprocess dataset
X, y, scaler = preprocess_dataset("data/train.csv")

# Split dataset
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Train model
model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Save model and scaler
joblib.dump(model, "models/random_forest.pkl")
joblib.dump(scaler, "models/scaler.pkl")

# Evaluate
y_pred = model.predict(X_test)
print(confusion_matrix(y_test, y_pred))
print(classification_report(y_test, y_pred))
```

---

### `src/live_capture.py`
```python
from scapy.all import sniff
import joblib
import pandas as pd

# Load trained model and scaler
model = joblib.load("models/random_forest.pkl")
scaler = joblib.load("models/scaler.pkl")

def process_packet(packet):
    # Example: extract simple features (expand later)
    features = {
        "src_bytes": len(packet),
        "dst_bytes": 0,  # placeholder
        "protocol_type": 1 if packet.haslayer("TCP") else 0
    }
    df = pd.DataFrame([features])
    X_scaled = scaler.transform(df)
    prediction = model.predict(X_scaled)
    print(f"Packet classified as: {prediction[0]}")

# Capture packets
sniff(prn=process_packet, count=10)
```

---

### `src/dashboard.py`
```python
from flask import Flask, render_template
import joblib

app = Flask(__name__)

@app.route("/")
def home():
    return "<h1>AI IDS Dashboard</h1><p>Model is running...</p>"

if __name__ == "__main__":
    app.run(debug=True)
```

---

### `README.md`
```markdown
# AI-Powered Intrusion Detection System (IDS)

## Overview
This project implements an AI-powered IDS using the NSL-KDD dataset. It preprocesses network traffic data, trains a Random Forest model, and provides real-time detection with a dashboard.

## Structure
- `src/preprocess.py`: Preprocessing pipeline
- `src/train_model.py`: Model training and evaluation
- `src/live_capture.py`: Real-time packet capture
- `src/dashboard.py`: Flask dashboard for alerts
- `data/`: Contains NSL-KDD train/test datasets
- `models/`: Stores trained models

## Run Training
```bash
cd AI_IDS
python -m src.train_model
```

## Run Dashboard
```bash
python -m src.dashboard
```

## Run Live Capture
```bash
sudo python -m src.live_capture
```

