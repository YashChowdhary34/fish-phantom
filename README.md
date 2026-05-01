# 🐟 Fish Phantom — Phishing URL Detection

A machine learning–powered web application that detects phishing websites in real time by analyzing URL and page-content features. The model is trained on a labeled dataset of over 11,000 websites and achieves **97.4% accuracy** on held-out test data.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Dataset](#dataset)
- [Model](#model)
- [Feature Engineering](#feature-engineering)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [Environment Variables](#environment-variables)
- [Usage](#usage)
- [Performance](#performance)
- [License](#license)

---

## Overview

Phishing attacks remain one of the most prevalent cybersecurity threats, deceiving users into revealing sensitive credentials through fraudulent websites. **Fish Phantom** combats this by extracting 30 structural, behavioral, and content-based features from any URL and running them through a trained **Gradient Boosting Classifier** to determine whether a site is legitimate or a phishing attempt.

---

## Features

- **Real-time URL analysis** — submit any URL and receive an instant classification
- **30-feature extraction engine** — covers URL structure, domain metadata, HTML content, and external service signals
- **Gradient Boosting model** — high accuracy ensemble method with tuned hyperparameters
- **React frontend** — clean, responsive UI built with Vite
- **Python backend** — lightweight server handling feature extraction and model inference

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite 6, ESLint |
| Backend | Python 3.8, Flask / Gunicorn |
| ML Model | scikit-learn (Gradient Boosting Classifier) |
| Data Processing | pandas, NumPy |
| Feature Extraction | BeautifulSoup4, requests, python-whois, googlesearch-python |
| Visualization (notebook) | matplotlib, seaborn |

---

## Project Structure

```
fish-phantom/
├── client/                         # React + Vite frontend
│   ├── public/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── App.css
│   │   └── main.jsx
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
│
├── server/                         # Python backend
│   ├── feature-extraction.py       # 30-feature URL analysis class
│   ├── models/
│   │   ├── model.pkl               # Serialized Gradient Boosting model
│   │   └── phishing-url-detection-model.ipynb  # Training notebook
│   └── .env
│
├── data/
│   ├── phishing.csv                # Main training dataset (11,054 samples, 30 features)
│   └── url.csv                     # Sample URLs for quick testing
│
├── tests/
│   └── Accuracy.txt                # Recorded model accuracy metrics
│
├── requirements.txt
├── environment.yml
└── .gitignore
```

---

## Dataset

The training dataset (`data/phishing.csv`) is sourced from [Kaggle — Phishing Website Detector](https://www.kaggle.com/eswarchandt/phishing-website-detector).

| Property | Value |
|---|---|
| Total samples | 11,054 |
| Independent features | 30 |
| Target classes | `1` (legitimate), `-1` (phishing) |
| Feature encoding | Integer (`1`, `0`, `-1`) |
| Missing values | None |
| Outliers | None |

The dataset is well-balanced with no pre-processing required for class imbalance.

---

## Model

**Algorithm:** Gradient Boosting Classifier (`sklearn.ensemble.GradientBoostingClassifier`)

**Hyperparameters:**
| Parameter | Value |
|---|---|
| `max_depth` | 4 |
| `learning_rate` | 0.7 |

**Training split:** 80% train / 20% test (random state = 42)

The model is serialized to `server/models/model.pkl` using Python's `pickle` module.

---

## Feature Engineering

The `FeatureExtraction` class in `server/feature-extraction.py` computes 30 features for any given URL. Features are grouped into three categories:

### URL-Based Features

| # | Feature | Description |
|---|---|---|
| 1 | `UsingIP` | Detects if the URL uses a raw IP address instead of a domain name |
| 2 | `LongURL` | Flags URLs longer than 54 characters (>75 = phishing) |
| 3 | `ShortURL` | Detects use of known URL shortening services (bit.ly, tinyurl, etc.) |
| 4 | `Symbol@` | Checks for `@` symbol in the URL, which redirects browsers |
| 5 | `Redirecting//` | Checks for `//` appearing after position 6 in the URL |
| 6 | `PrefixSuffix-` | Flags hyphen usage in the domain name |
| 7 | `SubDomains` | Counts the number of dots to assess subdomain depth |
| 8 | `HTTPS` | Verifies use of the HTTPS scheme |
| 9 | `DomainRegLen` | Checks domain registration length via WHOIS (< 12 months = suspicious) |
| 10 | `Favicon` | Detects favicons loaded from external domains |

### Page-Content Features

| # | Feature | Description |
|---|---|---|
| 11 | `NonStdPort` | Flags non-standard ports in the domain |
| 12 | `HTTPSDomainURL` | Detects `https` string appearing inside the domain token |
| 13 | `RequestURL` | Ratio of external resource URLs (images, audio, iframes) |
| 14 | `AnchorURL` | Ratio of anchor tags pointing to external or null locations |
| 15 | `LinksInScriptTags` | Ratio of external links in `<script>` and `<link>` tags |
| 16 | `ServerFormHandler` | Checks whether form actions point to the same domain |
| 17 | `InfoEmail` | Detects `mailto:` usage in page content |
| 18 | `AbnormalURL` | Compares response content against WHOIS data |
| 19 | `WebsiteForwarding` | Counts HTTP redirects (> 4 redirects = phishing) |
| 20 | `StatusBarCust` | Detects `onmouseover` JavaScript used to spoof the status bar |

### External / Behavioral Features

| # | Feature | Description |
|---|---|---|
| 21 | `DisableRightClick` | Detects JavaScript that disables right-click context menus |
| 22 | `UsingPopupWindow` | Checks for unauthorized popup (`alert()`) usage |
| 23 | `IframeRedirection` | Detects use of invisible `<iframe>` elements |
| 24 | `AgeofDomain` | Flags domains registered less than 6 months ago |
| 25 | `DNSRecording` | Verifies DNS record age via WHOIS |
| 26 | `WebsiteTraffic` | Checks Alexa rank (rank > 100,000 = suspicious) |
| 27 | `PageRank` | Queries global page rank via CheckPageRank |
| 28 | `GoogleIndex` | Verifies whether the URL is indexed by Google |
| 29 | `LinksPointingToPage` | Counts inbound links found in page source |
| 30 | `StatsReport` | Matches against known phishing domain/IP blocklists |

All features return one of three values: `1` (legitimate), `0` (suspicious), `-1` (phishing).

---

## Getting Started

### Prerequisites

- Python 3.8
- Node.js 18+ and npm
- Conda (recommended) or a Python virtual environment

### Backend Setup

**Option 1 — Conda (recommended):**

```bash
conda env create -f environment.yml
conda activate fish-phantom
```

**Option 2 — pip:**

```bash
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

**Start the server:**

```bash
cd server
# Using Gunicorn (production)
gunicorn app:app

# Or using Flask development server
python app.py
```

### Frontend Setup

```bash
cd client
npm install
npm run dev
```

The frontend will be available at `http://localhost:5173` by default.

---

## Environment Variables

Both the client and server contain `.env` files. Create or update them before running the application.

**`server/.env`**
```
# Add your server configuration here
FLASK_ENV=development
PORT=5000
```

**`client/.env`**
```
# Backend API base URL
VITE_API_URL=http://localhost:5000
```

> **Note:** Neither `.env` file is committed to source control. Refer to the `.gitignore` for details.

---

## Usage

1. Launch the backend server (see [Backend Setup](#backend-setup)).
2. Launch the frontend (see [Frontend Setup](#frontend-setup)).
3. Open `http://localhost:5173` in your browser.
4. Paste any URL into the input field and submit.
5. The application will extract the 30 features, run them through the model, and display whether the URL is **Legitimate** or **Phishing**.

To test the feature extraction directly:

```python
from server.feature_extraction import FeatureExtraction

fe = FeatureExtraction("https://example.com")
print(fe.getFeaturesList())
```

---

## Performance

| Metric | Score |
|---|---|
| Training Accuracy | **98.9%** |
| Test Accuracy | **97.4%** |
| Algorithm | Gradient Boosting Classifier |

Hyperparameter tuning was conducted over `max_depth` (1–9) and `learning_rate` (0.1–0.9), with `max_depth=4` and `learning_rate=0.7` yielding the best generalization performance on the 20% holdout set.

---

## License

This project is open source. See the repository root for license details.
