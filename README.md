# ThreatFusion Analyzer

Multi-source threat intelligence tool that analyzes URLs and file hashes for malicious activity using **VirusTotal**, **URLScan.io**, and **AI-powered reasoning**, orchestrated through a **Multi-stage Classification Pipeline (MCP)**.

## Architecture

### MCP Pipeline

Every analysis flows through 5 tracked stages with real-time status and timing:

```
┌──────────┐    ┌────────────┐    ┌─────────┐    ┌──────────┐    ┌─────────┐
│ Collect  │───▶│ Correlate  │───▶│  Score  │───▶│ Classify │───▶│ Explain │
│          │    │            │    │         │    │          │    │         │
│ VT +     │    │ Cross-ref  │    │ Weighted│    │ Safe /   │    │ Gemini  │
│ URLScan  │    │ findings   │    │ scoring │    │ Suspicious│   │ AI or   │
│ (async)  │    │ & insights │    │ engine  │    │ Malicious│    │ Rules   │
└──────────┘    └────────────┘    └─────────┘    └──────────┘    └─────────┘
```

### AI Agent

- Uses **Google Gemini 1.5 Flash** for intelligent threat explanations
- Produces structured output: reasoning, recommendations, confidence level
- Falls back to rule-based analysis when no Gemini key is configured

## Project Structure

```
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                 # FastAPI entry point
│   │   ├── config.py               # Environment / API key config
│   │   ├── models.py               # Pydantic request/response models
│   │   ├── routes.py               # REST endpoints (/analyze, /health)
│   │   ├── validators.py           # Input validation helpers
│   │   └── services/
│   │       ├── __init__.py
│   │       ├── virustotal_service.py   # VirusTotal v3 API integration
│   │       ├── urlscan_service.py      # URLScan.io API integration
│   │       ├── scoring_service.py      # Weighted scoring engine
│   │       ├── correlation_service.py  # Cross-source correlation
│   │       ├── ai_agent.py             # Gemini AI threat explainer
│   │       └── pipeline.py             # MCP pipeline orchestrator
│   ├── .env
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── main.jsx
│   │   ├── App.jsx                 # Main dashboard
│   │   ├── api.js                  # API client
│   │   ├── index.css               # Tailwind CSS v4
│   │   └── components/
│   │       ├── InputBox.jsx            # URL/hash input with validation
│   │       ├── ScoreIndicator.jsx      # Circular score + classification
│   │       ├── ResultCard.jsx          # Per-API result details
│   │       ├── PipelineView.jsx        # Pipeline stage tracker
│   │       ├── CorrelationInsights.jsx # Cross-source insight cards
│   │       └── AIAnalysisCard.jsx      # AI reasoning & recommendations
│   ├── package.json
│   └── vite.config.js
└── README.md
```

## Quick Start

### 1. Configure API keys

Create `backend/.env`:

```env
VIRUSTOTAL_API_KEY=your_key_here
URLSCAN_API_KEY=your_key_here
GEMINI_API_KEY=your_key_here          # optional — enables AI explanations
```

**Get free keys:**
- VirusTotal: https://www.virustotal.com → Profile → API key
- URLScan.io: https://urlscan.io → Settings → API Keys
- Gemini: https://aistudio.google.com/apikey

If keys are missing, the app returns mock data for demonstration.

### 2. Backend

```bash
cd backend
pip install -r requirements.txt
python3 -m uvicorn app.main:app --reload --port 8000
```

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

Open **http://localhost:5173** — the Vite dev server proxies API calls to the backend.

## API Endpoints

| Method | Path       | Description                                   |
|--------|-----------|-----------------------------------------------|
| POST   | /analyze  | Run full MCP pipeline on a URL or file hash   |
| GET    | /health   | Health check with API key status               |

### POST /analyze

**Request:**

```json
{
  "value": "https://example.com",
  "input_type": "url"
}
```

**Response:**

```json
{
  "risk_score": 25,
  "classification": "Safe",
  "explanation": "...",
  "api_results": [...],
  "pipeline_stages": [
    { "name": "Collect", "status": "completed", "duration_ms": 1200 },
    { "name": "Correlate", "status": "completed", "duration_ms": 5 },
    { "name": "Score", "status": "completed", "duration_ms": 1 },
    { "name": "Classify", "status": "completed", "duration_ms": 0 },
    { "name": "Explain", "status": "completed", "duration_ms": 800 }
  ],
  "correlation_insights": [
    { "type": "agreement", "severity": "info", "message": "..." }
  ],
  "ai_analysis": {
    "reasoning": "...",
    "recommendations": ["...", "..."],
    "confidence": "high",
    "model_used": "gemini-1.5-flash"
  },
  "mock_data": false
}
```

## Scoring Logic

- **VirusTotal weight:** 70%
- **URLScan.io weight:** 30%
- Detection ratio maps to 0–80 points
- Negative reputation adds up to 20 bonus points
- Final score clamped to 0–100

| Score Range | Classification |
|------------|----------------|
| 0–30       | Safe           |
| 31–70      | Suspicious     |
| 71–100     | Malicious      |

## Correlation Engine

Cross-references findings across sources to surface:

- **Multi-source agreement** — both APIs flag as malicious (high confidence)
- **Source disagreements** — one flags, other doesn't (requires attention)
- **Phishing indicators** — category-based detection from URLScan
- **Reputation analysis** — negative VT community scores
- **High detection rates** — >50% of engines flagging

## Tech Stack

| Layer    | Technology                     |
|----------|-------------------------------|
| Backend  | FastAPI, Python 3.9+, httpx   |
| Frontend | React 18, Tailwind CSS v4, Vite |
| AI       | Google Gemini 1.5 Flash       |
| APIs     | VirusTotal v3, URLScan.io v1  |
