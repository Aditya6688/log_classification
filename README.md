## 🔎 Hybrid Log Classification Framework

A hybrid system that combines rule-based matching, lightweight ML embeddings, and LLM reasoning to classify logs of varying complexity. It is fast and deterministic for simple cases, while remaining robust when data is noisy or under‑labeled.

### 📑 Table of Contents
- **Overview**
- **Key Approaches**
- **Project Structure**
- **Setup**
- **Environment variables**
- **Run the API server**
- **API usage**
- **Programmatic usage (Python)**
- **Training the ML model**
- **Troubleshooting**

## 🌟 Overview
- **Simple & predictable logs**: handled with regex rules.
- **Structured but complex logs**: handled with sentence embeddings + a small classifier.
- **Ambiguous or under‑labeled logs**: handled with an LLM fallback.

Combining these techniques provides end‑to‑end adaptability for real‑world logging systems.

## ⚙️ Key Approaches
- **Regex rules**: fast, deterministic for repetitive patterns.
- **Sentence‑Transformers + classifier**: captures semantics for trained categories.
- **LLM fallback**: classifies difficult edge cases or long‑tail events.

## 📂 Project Structure
```
project-nlp-log-classification/
  ├─ classify.py                 # Orchestrates regex → ML → LLM pipeline
  ├─ processor_regex.py          # Rule-based classifier
  ├─ processor_bert.py           # Embeddings + classifier (requires trained model)
  ├─ processor_llm.py            # Groq LLM-based classifier (uses .env)
  ├─ server.py                   # FastAPI server exposing /classify/
  ├─ resources/
  │   ├─ test.csv               # Sample input
  │   └─ output.csv             # Output written by the API
  ├─ training/
  │   ├─ dataset/
  │   │   └─ synthetic_logs.csv
  │   └─ log_classification.ipynb
  ├─ requirements.txt
  └─ models/                     # Optional; place trained model here
      └─ log_classifier.joblib   # Expected by processor_bert.py
```

## 🚀 Setup
1) Python 3.10+

2) Install dependencies
```bash
pip install -r requirements.txt
```

## 🔐 Environment variables
LLM classification uses Groq. Provide your API key in a `.env` file at the project root:
```
GROQ_API_KEY=your_key_here
```

## ▶️ Run the API server
Start the FastAPI server (hot‑reload in development):
```bash
uvicorn server:app --reload
```

Interactive docs:
- Swagger UI: `http://127.0.0.1:8000/docs`
- ReDoc: `http://127.0.0.1:8000/redoc`

## 📡 API usage
- **Endpoint**: `POST /classify/`
- **Body**: `multipart/form-data` with a `file` field containing a CSV.
- **CSV columns required**: `source`, `log_message`
- **Response**: Returns a CSV with an added `target_label` column. A copy is saved as `resources/output.csv`.

Example with curl:
```bash
curl -X POST "http://127.0.0.1:8000/classify/" \
  -H "accept: text/csv" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@resources/test.csv" \
  --output output.csv
```

Input CSV example:
```csv
source,log_message
app1,Connection timed out
app2,User login successful
```

Output CSV example:
```csv
source,log_message,target_label
app1,Connection timed out,NetworkError
app2,User login successful,AuthSuccess
```

## 🐍 Programmatic usage (Python)
```python
from classify import classify, classify_csv

# Classify a list of (source, log_message) tuples
labels = classify([
    ("ModernCRM", "File data_6957.csv uploaded successfully by user User265."),
    ("LegacyCRM", "The 'ReportGenerator' module will be retired in version 4.0."),
])
print(labels)

# Classify a CSV and write a new CSV with target_label
output_path = classify_csv("resources/test.csv")
print("Wrote:", output_path)
```

Routing logic (see `classify.py`):
- **LegacyCRM** sources: routed to the LLM classifier.
- Others: try regex first, then fall back to the ML classifier.

## 🎯 Training the ML model (optional)
`processor_bert.py` expects a trained classifier at `models/log_classifier.joblib`.

- Use the notebook: `training/log_classification.ipynb`
- Dataset example: `training/dataset/synthetic_logs.csv`
- After training, save the classifier to `models/log_classifier.joblib` and ensure the `models/` folder exists.

Without this file, the ML step will raise an error on import. You can rely on regex + LLM only, or adjust the code to skip the ML step.

## 🛠️ Troubleshooting
- File must be a CSV with headers `source`, `log_message`.
- Model file missing: create `models/` and place `log_classifier.joblib` there, or modify `processor_bert.py` to skip ML.
- LLM key missing: add `GROQ_API_KEY` to `.env` for LLM-based classification.
- uvicorn not found: `pip install uvicorn`.

---

Built for clarity, speed, and flexibility across diverse log streams.


