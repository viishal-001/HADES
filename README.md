# 🛡️ CAIS - Context-Aware Intent Shield

> **A mission-critical middleware protecting LLM-powered applications from prompt injections, jailbreaks, data leakage, and malicious intent.**

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Installation](#-installation)
- [Running the Application](#-running-the-application)
- [Web Interface Guide](#-web-interface-guide)
- [Technical Deep Dive](#-technical-deep-dive)
- [API Reference](#-api-reference)
- [Configuration](#-configuration)
- [Troubleshooting](#-troubleshooting)

---

## 🎯 Overview

**CAIS (Context-Aware Intent Shield)** is a security layer designed to sit between users and Large Language Models (LLMs). It intercepts user inputs in real-time, analyzes them for malicious intent, and enforces security policies *before* the input ever reaches the LLM.

### The Problem it Solves
- **Prompt Injection:** Attackers tricking LLMs into ignoring instructions.
- **Jailbreaks:** Bypassing safety filters (e.g., "DAN mode").
- **Data Leakage (DLP):** Users accidentally or maliciously extracting PII or API keys.
- **Legitimate Malice:** Distinguishing between a security analyst analyzing malware (Legitimate) and an attacker trying to execute it (Malicious).

---

## ✨ Key Features

### 1. 🛡️ Multi-Layered Defense
Combines regex heuristics, vector similarity search, and intent classification to detect sophisticated attacks.

### 2. ⚡ Ultra-Low Latency
Optimized for performance with **Result Caching** and efficient pipelining. Most requests are processed in **<50ms**.

### 3. 🧠 Context-Aware Analysis
Understands the *intent* behind the query. It allows "Analyze this malware" for a security tool but blocks "Write malware".

### 4. 🔒 Data Loss Prevention (DLP)
Automatically detects and blocks:
- **AWS Access Keys** (`AKIA...`)
- **API Tokens / Secrets**
- **PII (SSN, Email)**

### 5. 💻 Secure Code Review Simulation
Demonstrates how to protect an AI Code Review Agent from analyzing malicious code that contains injections.

### 6. 📊 Real-Time Metrics Dashboard
visualizes request counts, block rates, latency, and cache hit rates in the web interface.

---

## 🏗️ Architecture

The project is structured as a modular Python application with a loosely coupled frontend.

```
hack_project/
├── cais/                      # 🐍 Backend (Python Module)
│   ├── api/                   # FastAPI Server
│   │   └── server.py          # API Entrypoint
│   ├── core.py                # Main Protection Pipeline
│   ├── detection/             # Detectors
│   │   ├── heuristics.py      # Regex & Heuristic Logic (DLP here)
│   │   ├── vector_search.py   # Semantic Search (FAISS)
│   │   └── hybrid.py          # Combination Logic
│   ├── classification/        # Intent Classifiers
│   ├── mitigation/            # Action Engine (Block/Allow/Contain)
│   ├── session/               # Multi-turn Session Tracking
│   ├── sanitization/          # Input Normalization
│   └── performance.py         # Caching & Metrics
│
├── assets/                    # 🎨 Frontend Assets
│   ├── css/                   # Styles (style.css, chat.css)
│   └── js/                    # Logic (app.js - UI Controller)
│
├── index.html                 # 🖥️ Web Interface (Main Entry)
├── requirements.txt           # Python Dependencies
└── README.md                  # This File
```

---

## 🚀 Installation

### Prerequisites
- **Python 3.9+** installed.
- **Command Line / Terminal** access.

### 1. Setup Environment
```bash
# Navigate to project folder
cd <path_to_project>

# Create virtual environment (Optional but Recommended)
python -m venv venv
# Activate:
# Windows: venv\Scripts\activate
# Linux/Mac: source venv/bin/activate
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```
*Note: This installs FastAPI, Uvicorn, Torch, Transformers, FAISS, etc.*

---

## ▶️ Running the Application

You need to run **two** separate terminal processes (one for the Backend API, one for the Frontend UI).

### Terminal 1: Backend Server (API)
This starts the intelligent security engine.
```bash
python -m cais.api.server
```
*Wait until you see: `Uvicorn running on http://0.0.0.0:8000`*

### Terminal 2: Wellbeing Frontend (UI)
This serves the web interface to interact with CAIS.
```bash
python -m http.server 8080
```
*Wait until you see: `Serving HTTP on :: port 8080`*

### Access the App
Open your browser and navigate to:
👉 **http://localhost:8080**

---

## 🖥️ Web Interface Guide

The Web UI allows you to test CAIS capabilities interactively.

### 1. Console Mode
- **Security Scenario Selector:** Choose a use-case.
    - `🛡️ General Prompt Protection`: Basic jailbreak/injection testing.
    - `💻 Secure Code Review Agent`: Simulate protecting an AI coding assistant.
- **Input Area:** Type or paste text to analyze.
- **Preset Buttons:** Quickly load test payloads (Benign, Injection, PII Leak, etc.).
- **Analyze Input:** Sends text to the CAIS backend.
- **Result Panel:** Shows:
    - **Action:** (Allowed ✅, Blocked 🛡️, Contained 🔒)
    - **Analysis Details:** (Intent, Severity, Confidence)
    - **Defense Action:** (Sanitized output if applicable)

### 2. Data Loss Prevention (DLP) Test
1. Select **"Secure Code Review Agent"** (or General).
2. Paste a string containing an AWS Key (e.g., `AKIAIOSFODNN7EXAMPLE`).
3. Click **Analyze**.
4. Result: **BLOCKED**. Reason: "Sensitive Data Detected".

### 3. Chat Mode (Multi-turn)
- Click "Chat Mode" toggle.
- Simulate a conversation with a protected AI.
- CAIS tracks *Risk Score* across multiple turns. If you persistently try to jailbreak, the **Session Risk** increases until the session is **Locked**.

---

## 🔬 Technical Deep Dive

### The Protection Pipeline (`cais/core.py`)
1.  **Input Normalization:** Cleans invisible characters, homoglyphs, and normalizing unicode.
2.  **Emergency DLP Check:** (Fail-safe) Immediately scans for API keys and PII. Returns `BLOCK` instantly if found.
3.  **Heuristic Detection:** Scans for known attack patterns (regex) like "Ignore previous instructions", "DAN mode".
4.  **Vector Detection:** (Optional) Compares input semantic embedding against a database of known attacks using FAISS.
5.  **Intent Classification:** Determines if the user is attacking (`DIRECT_ATTACK`) or just asking a question (`LEGITIMATE_QUERY`).
6.  **Session Risk Scoring:** Updates the user's risk profile based on history.
7.  **Mitigation Engine:** Decides the final action (`BLOCK`, `ALLOW`, `SANITIZE`, `CONTAIN`).

### Performance (`cais/performance.py`)
- **LRU Cache:** Stores results of recent queries. Identical queries return in <1ms.
- **Latency Monitoring:** Tracks time spent in each component (Sanitization, Detection, etc.).

---

## 📚 API Reference

The backend provides a RESTful API (Swagger docs available at `http://localhost:8000/docs`).

### `POST /protect`
Main analysis endpoint.
- **Body:**
  ```json
  {
    "input": "User query string",
    "session_id": "unique_session_id",
    "context": "optional_context"
  }
  ```
- **Response:**
  ```json
  {
    "action": "block",
    "intent": "direct_attack",
    "confidence": 0.95,
    "reason": "Jailbreak pattern detected"
  }
  ```

### `GET /metrics`
Returns real-time system performance stats.
- **Response:**
  ```json
  {
    "performance": { "total_requests": 120, "avg_latency": 45.2 },
    "cache": { "hit_rate": 0.35 }
  }
  ```

### `GET /health`
System health check.

---

## 🔧 Configuration

You can adjust thresholds in `cais/models.py` or by passing a config object to `CAISMiddleware`.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `confidence_threshold` | 0.7 | Minimum confidence to classify an attack. |
| `session_risk_threshold` | 70.0 | Score at which a session is locked. |
| `max_input_length` | 8192 | Max characters to process. |
| `vector_detection_enabled` | True | Enable/Disable semantic search (Heavy). |

---

## ❓ Troubleshooting

**Q: The Analysis shows "Safe" for an attack!**
A: Check if you have recently restarted the backend. The system caches results. Restart the backend to clear the cache.

**Q: "Fetch Error" in the browser.**
A: Make sure the Backend Server (`python -m cais.api.server`) is running and accessible at `localhost:8000`.

**Q: Dependencies missing.**
A: Run `pip install -r requirements.txt`. Ensure you are in your virtual environment.

**Q: CSS/JS not updating.**
A: Hard reload your browser (`Ctrl+F5` or `Cmd+Shift+R`) to clear browser cache.

---

*Generated for CAIS Project.*
