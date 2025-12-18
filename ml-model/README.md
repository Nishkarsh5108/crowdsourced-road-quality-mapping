# Road Condition Classification & Real-Time Sender

A machine-learning powered pipeline that reads mobile sensor data, predicts road conditions (Good/Bad) using a trained CNN model, and sends live updates to a backend API with GPS coordinates.

This repository contains:
- A real-time prediction loop (`run_model.py`)
- A network sender module (`ml_sender.py`)
- The sensor dataset (`combined.csv`)
- A trained PyTorch CNN model (`best_roadcnn.pth`)
- A Jupyter notebook (`main.ipynb`)
- An optional scaler (`scaler.pkl`)

---

## 🚀 Features

### 1. Real-Time Window-Based ML Inference
- Uses accelerometer + gyroscope values: `ax ay az wx wy wz`
- Processes data in windows of length **10**
- Standardizes data using StandardScaler
- CNN model predicts: **Good** or **Bad**

### 2. GPS-Attached JSON Payloads

Each prediction is sent to backend as:

```

{
"latitude": <float>,
"longitude": <float>,
"road_condition": "Good" | "Bad"
}

```

Target backend endpoint:

```

[http://10.149.131.154:5000/update](http://10.149.131.154:5000/update)

```

---

## 📁 Repository Structure

```

.
├── data/
│   └── combined.csv
│
├── src/
│   ├── run_model.py
│   ├── ml_sender.py
│   └── best_roadcnn.pth
│
├── scaler.pkl
├── main.ipynb
└── README.md

```

---

## 🧠 Model Architecture (RoadConditionCNN)

- Conv1D → ReLU → MaxPool  
- Conv1D → ReLU → MaxPool  
- Flatten → Linear → ReLU → Dropout → Output layer  

Expected input shape:

```

(1, 6 features, sequence_length=10)

```

---

## ⚙️ How the Pipeline Works

### 1. Data Loading
- Loads `combined.csv`
- Ensures required columns:
  - `ax ay az wx wy wz`
  - `latitude longitude`
  - `roadCondition`

### 2. Fit Scaler
Scaler is fitted on dataset features to keep normalization consistent.

### 3. Load CNN Model
- Loads `best_roadcnn.pth`
- Moves to CPU/GPU
- Switches to eval mode  
If model fails → automatic **Simulation Mode** using CSV labels.

### 4. Prediction Loop
For each window of 10 rows:
1. Slice window  
2. Scale features  
3. Predict Good/Bad  
4. Use GPS from last row  
5. Send JSON to backend  
6. Sleep 1 second  
7. Move forward by 10 rows  
8. Restart when CSV ends  

---

## 🛰️ Network Sender (`ml_sender.py`)

Sends JSON to backend using:

```

send_prediction(latitude, longitude, prediction)

```

Handles:
- timeouts  
- connection errors  
- invalid responses  
- success logs with timestamps  

---

## 🔧 Configuration

Main config in `run_model.py`:

```

SEQUENCE_LENGTH = 10
SEND_INTERVAL_SECONDS = 1
DATA_FILE = '..\data\combined.csv'
MODEL_FILE = '..\src\best_roadcnn.pth'
FEATURES = ['ax','ay','az','wx','wy','wz']
BACKEND_URL = "[http://10.149.131.154:5000/update](http://10.149.131.154:5000/update)"

```

---

## ▶️ How to Run

### Install dependencies:

```

pip install torch pandas scikit-learn requests numpy

```

### Run main loop:

```

python run_model.py

```

### Test only the sender:

```

python ml_sender.py

```

---

## 🧪 Simulation Mode

If model fails:
- No crash  
- Uses majority label of window (`roadCondition`)  
- Still sends correct GPS + condition  

---

## 📊 Dataset Info

`combined.csv` contains:
- accelerometer + gyroscope values  
- latitude, longitude  
- ground truth labels  

---

## 🧱 Possible Extensions

- temporal smoothing  
- ONNX / TorchScript export  
- real-time dashboard  
- battery optimization  

---

## 🤝 Contributing

Pull requests welcome.

---


