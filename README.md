# Stock Prediction Portal

A full‑stack web app for stock price prediction. The backend (Django REST Framework) fetches historical data from Yahoo Finance, runs an LSTM model (Keras/TensorFlow) to generate predictions and plots, and exposes a clean API. The frontend (React + Vite) provides a simple portal for user auth and running predictions.

This README includes everything you need to run the app locally on Windows with PowerShell and publish the project on GitHub.

## Features

- User registration and JWT-based authentication (access/refresh)
- Predict endpoint powered by a pre-trained Keras model
- Automatic data download with yfinance (10 years of history)
- Generated plots (closing price, 100/200 DMA, and final prediction) saved to Media and served via API
- React UI with protected routes and token refresh handling

## Tech stack

- Backend: Python, Django 5, Django REST Framework, SimpleJWT, django-cors-headers
- ML/DS: yfinance, NumPy, pandas, scikit-learn, matplotlib, Keras/TensorFlow
- Frontend: React 18, Vite 5, react-router, axios

## Repository layout

```
Stock-Prediction-Portal-model/
├─ backend-drf/                 # Django REST API
│  ├─ stock_prediction_main/    # Django project (settings, urls)
│  ├─ api/                      # API app (prediction endpoint)
│  ├─ accounts/                 # Auth: register + protected route
│  ├─ stock_prediction_model.keras  # Pre-trained LSTM model used by API
│  ├─ manage.py
│  └─ .env                      # SECRET_KEY and DEBUG
├─ frontend-react/              # React + Vite frontend
│  └─ src/
├─ Resources/                   # Not needed for runtime; notebooks, datasets
└─ requirements.txt             # Base Django deps (see below for full backend deps)
```

## Prerequisites

- Windows 10/11
- Python 3.10 or 3.11 (recommended)
- Node.js 18+ and npm
- Git (optional but recommended)
- VS Code (optional) with Python and ESLint extensions

## Quick start (local dev)

- Backend API runs on http://127.0.0.1:8000
- Frontend runs on http://localhost:5173

### 1) Backend (Django + ML)

From the repository root:

```powershell
# Go to the backend
cd backend-drf

# Create and activate a virtual environment
py -3 -m venv env
.\env\Scripts\Activate.ps1

# Upgrade packaging tools
python -m pip install --upgrade pip setuptools wheel

# Install base Django requirements from the repo root file
pip install -r ..\requirements.txt

# Install additional backend packages used by the prediction API
pip install djangorestframework-simplejwt django-cors-headers yfinance pandas numpy matplotlib scikit-learn tensorflow

# Create your .env (already present in this repo but shown here for clarity)
# backend-drf\.env
# SECRET_KEY=your-django-secret
# DEBUG=True

# Apply migrations and run
python manage.py migrate
python manage.py runserver
```

Notes
- The Keras model file `stock_prediction_model.keras` is already included under `backend-drf/` and is loaded by the API.
- Media files (plots) are written to `backend-drf/media/` and are automatically served by Django in development.

### 2) Frontend (React + Vite)

In a new terminal (keep the backend running):

```powershell
# From repo root
cd frontend-react

# Install packages
npm install

# Frontend environment: point to the backend base API
# Create a file named .env.local with this line:
# VITE_BACKEND_BASE_API=http://127.0.0.1:8000/api/v1

# Start the dev server
npm run dev
```

Open the URL shown by Vite (typically http://localhost:5173). The app will connect to the backend using the URL set in `VITE_BACKEND_BASE_API`.

## How to open and run in VS Code

- Open the repository folder in VS Code (File > Open Folder…)
- Recommended extensions: Python, Pylance, ESLint
- Open two terminals inside VS Code:
  1) Backend terminal: run the backend commands above
  2) Frontend terminal: run the frontend commands above
- Optionally create VS Code tasks/launch configs later for one-click runs

## Environment variables

Backend (`backend-drf/.env`)
- SECRET_KEY: Django secret key (required)
- DEBUG: True/False (True for local development)

Frontend (`frontend-react/.env.local`)
- VITE_BACKEND_BASE_API: Base API URL (e.g., http://127.0.0.1:8000/api/v1)

CORS is already configured to allow `http://localhost:5173` in development.


## Acknowledgements

- Yahoo Finance data via `yfinance`
- Django REST Framework and community packages
- Keras/TensorFlow for model inference
