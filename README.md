# Crowdsourced Road Quality Mapping System 🚗🛣️

An intelligent, community-driven solution to detect, map, and analyze road conditions (potholes, bad roads) using smartphone sensors, IoT hardware, and Machine Learning.

## 📖 Table of Contents
- [Project Overview](#-project-overview)
- [System Architecture](#-system-architecture)
- [Repository Structure](#-repository-structure)
- [Key Components](#-key-components)
  - [1. Data Collection App](#1-data-collection-app)
  - [2. Machine Learning Model](#2-machine-learning-model)
  - [3. Backend Server](#3-backend-server)
  - [4. Navigation & Map App](#4-navigation--map-app)
  - [5. IoT Hardware (Prototype)](#5-iot-hardware-prototype)
- [Getting Started](#-getting-started)
- [Workflow & Logic](#-workflow--logic)
- [Technologies Used](#-technologies-used)
- [Future Scope](#-future-scope)

---

## 🚀 Project Overview

**Problem:** Potholes and uneven road surfaces are a major cause of accidents and vehicle damage. Authorities often lack real-time data to identify and repair these hazards efficiently.

**Solution:** A comprehensive system that collects road vibration data using vehicle-mounted sensors (or smartphone sensors), classifies the road quality (Good vs. Bad) using a **Convolutional Neural Network (CNN)**, and updates a live map for users to avoid hazardous routes.

**Core Features:**
- **Automated Detection:** ML model analyzes accelerometer & gyroscope data to detect anomalies.
- **Crowdsourcing:** Users contribute data simply by driving.
- **Real-time Mapping:** Visualization of hazardous zones on a map (Green = Good, Red = Bad).
- **Dual-Mode Data Gathering:** Supports both a dedicated Android app and a custom hardware module.

---

## 🏗 System Architecture

The project follows a cyclic data flow:
1.  **Data Acquisition:** Sensors (Smartphone or ESP32+MPU6050) collect kinematic data (`ax, ay, az, wx, wy, wz`) and GPS coordinates.
2.  **Edge/Cloud Processing:** Kinematic data constitutes a time-series window (e.g., 1 second).
3.  **Classification:** The CNN model processes the window to classify road condition.
4.  **Database/Server:** Results are stored/relayed by the backend.
5.  **User Feedback:** The Map App fetches these results and overlays road quality on the user's map.

---

## 📂 Repository Structure

```
crowdsourced-road-quality-mapping/
├── backend/                # Flask server & Inference logic
│   ├── data/               # Contains CSVs for simulation
│   ├── model/              # Trained PyTorch models (.pth)
│   ├── server.py           # API endpoints (Flask)
│   └── inference.py        # Real-time inference simulation logic
├── data-collection-app/    # Android App for training data generation
│   ├── app/src/.../MainActivity.kt  # Sensor collection logic
│   └── ...
├── map-app/                # Android User App for visualization
│   ├── app/src/.../MainActivity.kt  # OSM Map & API polling
│   └── ...
└── ml-model/               # Model training resources
    ├── src/                # Training notebooks & scripts
    └── ...
```

---

## 🧩 Key Components

### 1. Data Collection App
**Purpose:** Collect labeled training data.
- **Features:**
    - Reads Accelerometer & Gyroscope at high frequency.
    - Captures GPS Location (Lat, Lon, Speed).
    - **User Interface:** Real-time graphs for sensors; "Good Road" / "Bad Road" toggle button for manual labeling.
    - **Output:** Generates a CSV file (`timestamp, ax, ay, az, wx, wy, wz, lat, lon, speed, label`).
- **Location:** `data-collection-app/`

### 2. Machine Learning Model
**Purpose:** Classify 1-second time windows of sensor data.
- **Type:** 1D Convolutional Neural Network (CNN).
- **Architecture:**
    - Input: 10 time-steps x 6 features (Ax, Ay, Az, Wx, Wy, Wz).
    - Layers: 2x Conv1D + ReLU + MaxPool blocks -> Flatten -> Dense -> Dropout -> Output (Softmax).
- **Training:** Trained on collected CSV data.
- **Location:** `ml-model/` (contains training notebooks) & `backend/model/` (contains `.pth` weights).

### 3. Backend Server
**Purpose:** Acts as the bridge between model inference and the User App.
- **Tech Stack:** Python, Flask.
- **Operation:**
    - Runs a background thread (`inference.py`) that reads from a data source or stream.
    - Scales the data using `StandardScaler`.
    - Runs the PyTorch model inference.
    - Exposes an endpoint `/get_latest` for the Map App to consume.
- **Location:** `backend/`

### 4. Navigation & Map App
**Purpose:** The end-user interface.
- **Tech Stack:** Android (Kotlin), OSMDroid (OpenStreetMap).
- **Features:**
    - Displays a live map.
    - Polls the backend server every few seconds.
    - Plot markers: **Green** for Good Road, **Red** for Bad Road/Pothole.
- **Location:** `map-app/`

### 5. IoT Hardware (Prototype)
*Planned/Prototype Stage*
- **Components:** ESP32 MCU, MPU6050 (6-axis IMU), Neo M8N (GPS).
- **Function:** Independent "black box" that sits in the car chassis, collecting and transmitting data via Bluetooth/WiFi, removing the need for the user's phone to be rigidly mounted.

---

## 🛠 Getting Started

### Prerequisites
- **Python 3.8+** (for Backend)
- **Android Studio** (for Apps)
- **Physical Android Device** (Recommended for sensors/GPS testing)

### Step 1: Run the Backend
The backend mimics a live stream by reading a combined CSV file and running inference.

1. Navigate to the backend folder:
   ```bash
   cd backend
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the server:
   ```bash
   python server.py
   ```
   *The server will start on port 5000 and begin printing inference logs.*

### Step 2: Run the Map App
1. Open the **`map-app`** folder in Android Studio.
2. Open `MainActivity.kt` and update the `SERVER_URL` with your computer's local IP address:
   ```kotlin
   // Example:
   private val SERVER_URL = "http://192.168.1.X:5000/get_latest"
   ```
3. Build and Run on an Android Emulator or Device.
4. You should see markers appearing on the map corresponding to the inference results.

### Step 3: Collect New Data (Optional)
If you want to train your own model:
1. Open **`data-collection-app`** in Android Studio.
2. Run on a physical device.
3. Rigidly mount the phone in your car (e.g., center console).
4. Start recording and toggle "Good/Bad" as you drive.
5. Export the CSV and use it to retrain the model in `ml-model/`.

---

## 🔄 Workflow & Logic

1. **Input:** The `inference.py` script reads a window of 10 rows (approx 1 second) from the input CSV.
2. **Preprocessing:** Data is normalized using a pre-fitted `scaler.pkl`.
3. **Inference:** The loaded CNN assumes the input shape `(Batch, Features, Time)` and outputs a class prediction (0 or 1).
4. **Broadcast:** The prediction + GPS coordinates are stored in memory.
5. **Consumption:** The Android Map App requests this data via HTTP GET and renders the visual indicator.

---

## 💻 Technologies Used
- **Languages:** Kotlin (Android), Python (ML/Backend).
- **Libraries/Frameworks:**
    - **Android:** Jetpack Compose, XML Views, MPAndroidChart, OSMDroid, Volley.
    - **ML/Data:** PyTorch, Pandas, Scikit-learn, NumPy.
    - **Server:** Flask.
    - **Hardware (Firmware):** C++ (Arduino/ESP-IDF).

---

## 🔮 Future Scope
- **Real-time Hardware Integration:** Fully replacing the CSV simulation with live UDP/WebSocket data streams from the ESP32.
- **Advanced Navigation:** Routing algorithms that actively avoid "Red" zones.
- **Geospatial Database:** Storing hazards in a persistent DB (PostgreSQL/PostGIS) instead of ephemeral memory.

---
*Created for the Pothole Detection & Road Quality Mapping Group Project.*
