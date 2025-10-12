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

## API overview

Base URL: `http://127.0.0.1:8000/api/v1/`

- POST `register/`
  - Body: `{ "username": "alice", "email": "alice@example.com", "password": "yourpassword" }`
  - Creates a user account.

- POST `token/`
  - Body: `{ "username": "alice", "password": "yourpassword" }`
  - Returns `{ access, refresh }` for JWT auth.

- POST `token/refresh/`
  - Body: `{ "refresh": "<refresh-token>" }`
  - Returns a new `{ access }`.

- GET `protected-view/`
  - Header: `Authorization: Bearer <access-token>`
  - Returns a short confirmation payload.

- POST `predict/`
  - Header: `Authorization: Bearer <access-token>` (enforced by default auth settings)
  - Body: `{ "ticker": "AAPL" }`
  - Downloads up to 10 years of daily data for the ticker, runs the model, saves plots, and returns:
    ```json
    {
      "status": "success",
      "plot_img": "/media/AAPL_plot.png",
      "plot_100_dma": "/media/AAPL_100_dma.png",
      "plot_200_dma": "/media/AAPL_200_dma.png",
      "plot_prediction": "/media/AAPL_final_prediction.png",
      "mse": 0.0,
      "rmse": 0.0,
      "r2": 0.0
    }
    ```

Tip: You can open the image URLs in the browser directly, e.g. `http://127.0.0.1:8000/media/AAPL_plot.png`.

## How it works (high level)

1) The frontend authenticates users and stores tokens in `localStorage`.
2) An axios interceptor automatically sends `Authorization: Bearer <access>` and refreshes tokens when needed.
3) For predictions, the backend:
   - Pulls historical data with `yfinance.download()`
   - Builds test sequences, loads `stock_prediction_model.keras`
   - Predicts and inverse-transforms values
   - Generates and saves plots under `MEDIA_ROOT`
   - Responds with plot URLs and metrics (MSE, RMSE, R²)

## Troubleshooting

- TensorFlow/Keras install issues on Windows
  - Try: `pip install --upgrade pip setuptools wheel`
  - Ensure Python 3.10/3.11 and a recent `pip`
  - For CPU-only, plain `pip install tensorflow` is fine; GPU requires extra setup

- CORS errors in the browser
  - Confirm `CORS_ALLOWED_ORIGINS` in `backend-drf/stock_prediction_main/settings.py` includes your frontend origin (default: `http://localhost:5173`).

- 401 Unauthorized from the API
  - Make sure you logged in via `token/` and have a valid access token
  - The frontend refresh flow uses `/token/refresh/` automatically

- Empty data from yfinance
  - Verify the ticker symbol (e.g., `AAPL`, `TSLA`, `MSFT`)
  - Some tickers may not have 10 years of data; the API returns an error if empty

- File paths for the model
  - The backend loads `stock_prediction_model.keras` from the project root (`backend-drf/`). Always run `manage.py` commands from `backend-drf/` so relative paths resolve.

## Scripts reference

Backend
- `python manage.py migrate`
- `python manage.py createsuperuser`
- `python manage.py runserver`

Frontend
- `npm run dev` – start dev server
- `npm run build` – production build
- `npm run preview` – preview production build

## Production notes (brief)

- Configure environment variables securely; set `DEBUG=False` and add your domain to `ALLOWED_HOSTS`
- Serve Django via a WSGI server (e.g., gunicorn/uvicorn + reverse proxy) and a proper static/media setup
- Build the frontend and serve static assets via a CDN or web server

## License

No license is declared in this repository. If you plan to open-source, add a LICENSE file (e.g., MIT).

## Acknowledgements

- Yahoo Finance data via `yfinance`
- Django REST Framework and community packages
- Keras/TensorFlow for model inference
