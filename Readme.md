```md
# Intelligent Test Case Search Platform – Modular Edition  
## TestCases-RAG Version 2.0

---

## 🔍 Overview

This project is a **production-grade backend platform** for uploading, enriching, indexing, and semantically searching software test cases using modern AI and vector search technologies.

### Key Technologies

- ✅ **FastAPI** – High-performance API framework  
- ✅ **MongoDB Atlas** – Persistence + Vector Search  
- ✅ **SentenceTransformers (all-MiniLM-L6-v2)** – Embedding generation  
- ✅ **Google Gemini** – Enrichment, query expansion & reranking  
- ✅ **JWT Authentication** – Secure role-based access  
- ✅ **Advanced Ranking** – Multi-signal scoring + A/B testing  
- ✅ **Search Caching** – Faster repeat queries  
- ✅ **Audit Logging + Metrics** – Observability & experimentation  

This version refactors the original monolithic *TestCases-RAG* implementation into a **fully modular architecture** to support improved debugging, scalability, and experimentation workflows.


---

## ⚙️ Setup & Installation

### ✅ 1. Requirements

- Python **3.10+**
- MongoDB Atlas with Vector Search enabled
- Google Gemini API key

---

### ✅ 2. Clone & Setup Virtual Environment

```bash
git clone <your-repository>
cd <your-repository>

python -m venv .venv
source .venv/bin/activate     # macOS / Linux
.venv\Scripts\activate        # Windows
````

---

### ✅ 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 📦 Required Packages

```
fastapi
uvicorn
motor
pymongo
sentence-transformers
numpy
pandas
python-dotenv
python-jose
passlib[bcrypt]
google-generativeai
```

---

## 🔑 Environment Variables

Create a `.env` file:

```
GOOGLE_API_KEY=your-google-api-key
MONGO_CONNECTION_STRING=your-mongodb-uri

JWT_SECRET_KEY=change-me-in-prod
```

---

## ✅ MongoDB Vector Index

Create a **Vector Search Index** on the `main_vector` field:

```json
{
  "fields": [
    {
      "type": "vector",
      "path": "main_vector",
      "numDimensions": 384,
      "similarity": "cosine",
      "quantization": "none"
    }
  ]
}
```

Index name must be:

```
vector_index
```

---

## ▶️ Running the Application

```bash
uvicorn app.main:app --reload
```

### API Base URL

```
http://localhost:8000
```

### Swagger Docs

```
http://localhost:8000/docs
```

---

## 🔐 Authentication

### Register

```http
POST /auth/register
```

```json
{
  "username": "admin",
  "password": "test123",
  "role": "admin"
}
```

---

### Login

```http
POST /auth/login
```

(Form URL-encoded body)

```json
{
  "access_token": "...",
  "token_type": "bearer"
}
```

---

### Auth Header

```
Authorization: Bearer <YOUR_TOKEN>
```

---

## 🧑‍💼 Role Permissions

| Role   | Permissions                                  |
| ------ | -------------------------------------------- |
| viewer | Search only                                  |
| editor | Upload, update, delete individual test cases |
| admin  | Full control + delete-all + metrics access   |

---

## 📤 Uploading Test Cases

### Endpoint

```http
POST /api/upload
```

**Access:** `editor` or `admin`

---

### Accepted Formats

* `.csv`
* `.xlsx`

---

### Required Columns

* Test Case ID
* Feature
* Test Case Description
* Pre-requisites
* Test Step
* Expected Result
* Step No.

---

### Optional Columns

* Tags (comma-separated)
* Priority
* Platform

---

## 🔄 Ingestion Flow

```
Upload
   ↓
Gemini enrichment + keyword tagging
   ↓
SentenceTransformer embeddings
   ↓
Mean-vector aggregation
   ↓
MongoDB insert/index
```

---

## 🔎 Searching Test Cases

### Endpoint

```http
POST /api/search
```

### Example Request

```json
{
  "query": "payment failure",
  "feature": "Checkout",
  "tags": ["Regression"],
  "priority": "High",
  "platform": "Mobile",
  "ranking_variant": "B"
}
```

---

## 🔬 Search Pipeline

```
Query
   ↓
Embedding
   ↓
MongoDB $vectorSearch
   ↓
Local signal fusion ranker
   ↓
(Gemini reranking — optional)
   ↓
Diversity filtering
   ↓
Top-K results
```

---

## 📊 Ranking Variants

### Variant A — Baseline

```
0.60 * Vector similarity
0.25 * Max cosine similarity
+ Token match boosts
```

---

### Variant B — Enhanced

```
0.45 * Vector similarity
0.20 * Semantic similarity
0.12 * Keyword overlap
0.08 * Feature name match
0.05 * Token density
0.05 * Popularity weighting
```

Set using:

```
"ranking_variant": "A" | "B"
```

---

## 🔄 Updating Records

```http
PUT /api/update/{doc_id}
```

### Example Payload

```json
{
  "feature": "Payments",
  "priority": "Critical",
  "tags": ["Smoke","API"]
}
```

Automatically triggers:

* Gemini re-enrichment (if needed)
* Re-embedding
* Vector recalculation

---

## 👮 Admin APIs

| Endpoint                            |
| ----------------------------------- |
| `GET /api/get-all`                  |
| `POST /api/delete-all?confirm=true` |
| `DELETE /api/testcase/{id}`         |
| `GET /api/metrics`                  |

---

## 📈 Metrics Example

```json
{
  "queries_today": 281,
  "top_features": ["Login", "Checkout"]
}
```

---

## 🧾 Audit Logging

All searches record:

* Timestamp
* API endpoint
* User
* Request payload
* Ranking variant
* Result count

Stored in MongoDB collection:

```
api_audit_logs
```

---

## ✅ Why This Matters

Audit + metrics enable:

* Search quality monitoring
* Ranking experiments (A/B testing)
* Query behavior insights
* UX optimization

---

## 🧠 Development Workflow

* Ranking logic → `app/services/ranking.py`
* Gemini experimentation →
  `app/services/expansion.py`
  `app/services/rerank.py`
* Data schemas → `app/models/schemas.py`
* API wiring → `app/routes/`



```
```
