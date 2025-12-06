# TestCases-RAG-Version-2.0
## Intelligent Test Case Search Platform – Modular Edition

---

## 🔍 Overview

This project is a **production-grade backend platform** for uploading, enriching, indexing, and semantically searching software test cases using:

- ✅ **FastAPI** for APIs  
- ✅ **MongoDB Atlas** for persistence & vector search  
- ✅ **SentenceTransformers (all-MiniLM-L6-v2)** for embeddings  
- ✅ **Google Gemini** for enrichment, query expansion, and reranking  
- ✅ **JWT Authentication** with role-based access  
- ✅ **Advanced ranking heuristics + A/B testing**  
- ✅ **Search caching**  
- ✅ **Audit logging + metrics**

This refactor modularizes the original single-file application into clean layers to enable easier **debugging, scaling, and experimentation workflows**.

---

## 📂 Project Structure

```

app/
├── main.py                # App startup + lifespan orchestration
│
├── core/                  # Global configuration & security
│   ├── config.py          # Env + constants
│   ├── logging.py         # Structured logging
│   ├── cache.py           # In-memory query caching
│   ├── security.py        # JWT + password hashing
│   └── analytics.py      # Audit logging
│
├── db/
│   └── mongo.py           # MongoDB connection + helpers
│
├── models/
│   ├── schemas.py         # Pydantic DTO schemas
│   └── users.py           # Mongo user CRUD helpers
│
├── services/
│   ├── embeddings.py     # SentenceTransformer lifecycle + batching
│   ├── keywords.py       # Keyword extraction & fallback summaries
│   ├── enrichment.py     # Gemini test-case enrichment
│   ├── expansion.py      # Gemini query expansion
│   ├── rerank.py          # Gemini reranking
│   └── ranking.py         # Multi-signal scoring + A/B logic
│
├── routes/
│   ├── auth.py            # Login / Register APIs
│   ├── upload.py          # CSV/XLSX ingestion + enrichment + embeddings
│   ├── search.py          # Hybrid vector + heuristic ranking search
│   ├── update.py          # Test case updates + reprocessing
│   └── admin.py           # Admin maintenance + metrics APIs
│
└── middleware/            # Optional global middleware (future work)

```

---

## ⚙️ Setup & Installation

### ✅ 1. Python Version

```

Python 3.10+

````

---

### ✅ 2. Clone & Setup Virtual Environment

```bash
git clone <your-repository>
cd <your-repository>

python -m venv .venv
source .venv/bin/activate      # macOS/Linux
.venv\Scripts\activate         # Windows
````

---

### ✅ 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## 📦 Required Packages

Your **`requirements.txt`** should include:

```txt
fastapi
uvicorn
motor
pymongo
sentence-transformers
numpy
pandas
python-dotenv
python-jose
passlib[bcrypt]==1.7.4
bcrypt==3.2.2
openpyxl
google-generativeai
python-multipart
```

---

## 🔑 Environment Variables

Create a **`.env`** file in the root:

```env
GOOGLE_API_KEY=your-google-api-key
MONGO_CONNECTION_STRING=your-mongodb-uri
JWT_SECRET_KEY=change-me-in-production
```

---

## ✅ MongoDB Requirements

You must create a **Vector Search Index** in MongoDB Atlas using the following configuration on the field `main_vector`:

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

### 🔍 Index Name

```
vector_index
```

> ⚠️ The name must match exactly.

---

## ▶️ Running the Application

Start the FastAPI backend:

```bash
uvicorn app.main:app --reload
```

---

### 🌐 Access Endpoints

| Resource   | URL                                                        |
| ---------- | ---------------------------------------------------------- |
| API Base   | [http://localhost:8000](http://localhost:8000)             |
| Swagger UI | [http://localhost:8000/docs](http://localhost:8000/docs)   |
| ReDoc      | [http://localhost:8000/redoc](http://localhost:8000/redoc) |

---

---

## 🔐 Authentication & User Roles

### Create Account

**POST** `/auth/register`

```json
{
  "username": "admin",
  "password": "test123",
  "role": "admin"
}
```

---

### Login

**POST** `/auth/login`

*(Form-encoded)*

Returns:

```json
{
  "access_token": "JWT_TOKEN",
  "token_type": "bearer"
}
```

---

### Use Token

Add to API headers:

```
Authorization: Bearer YOUR_TOKEN
```

---

### 👥 Role Permissions

| Role   | Permissions                                   |
| ------ | --------------------------------------------- |
| viewer | Search only                                   |
| editor | Upload, update, delete individual test cases  |
| admin  | Full control, delete all data, metrics access |

---

---

## 📤 Uploading Test Cases

**POST** `/api/upload`
(**editor or admin role required**)

### Accepted File Types

* `.csv`
* `.xlsx`

### ✅ Required Columns

* Test Case ID
* Feature
* Test Case Description
* Pre-requisites
* Test Step
* Expected Result
* Step No.

### 🧩 Optional Columns

* Tags *(comma-separated)*
* Priority
* Platform

---

### 📌 Processing Flow

1. File ingestion
2. Gemini summary + keyword extraction
3. Batched SentenceTransformer embeddings
4. Mean vector generation
5. MongoDB insert + indexing

---

---

## 🔍 Searching Test Cases

**POST** `/api/search`

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

### 🧠 Search Pipeline

```
User Query
   ↓
Embedding
   ↓
MongoDB $vectorSearch
   ↓
Local signal-fusion ranking
   ↓
Optional Gemini re-ranking
   ↓
Diversity filtering
   ↓
Final TOP-K results
```

---

### 📊 Scoring Signals

---

#### Ranking Variant A — **Baseline**

```
0.60 × Vector similarity  
0.25 × Max cosine similarity  
+ Token match boosts
```

---

#### Ranking Variant B — **Enhanced**

```
0.45 × Vector similarity
0.20 × Semantic similarity
0.12 × Keyword overlap
0.08 × Feature name match
0.05 × Token density
0.05 × Popularity weighting
```

---

Use:

```json
"ranking_variant": "A" | "B"
```

---

---

## 🔄 Updating Records

**PUT** `/api/update/{doc_id}`

### Partial Update Example

```json
{
  "feature": "Payments",
  "priority": "Critical",
  "tags": ["Smoke", "API"]
}
```

---

### 🔁 Triggers Automatically

* Gemini re-enrichment (if needed)
* Re-embedding
* Main vector recalculation

---

---

## 👮 Admin APIs

### Get All Test Cases

**GET** `/api/get-all`

---

### Delete ALL Data

**POST** `/api/delete-all?confirm=true`
*(admin only)*

---

### Delete Single Test Case

**DELETE** `/api/testcase/{id}`

---

### Metrics

**GET** `/api/metrics`

Returns:

```json
{
  "queries_today": 281,
  "top_features": ["Login", "Checkout"]
}
```

---

---

## 🧾 Audit Logging

Every search request records:

* Timestamp
* Endpoint
* User
* Request payload
* Ranking variant
* Result count

### Mongo Collection

```
api_audit_logs
```

---

### ✅ Why Audit Logging Matters

* Quality monitoring
* Ranking experimentation feedback
* Popular query tracking
* Search UX optimization

---

---

## 🧠 Development Workflow

### Recommended Flow

#### Ranking & Ranking Experiments

```text
app/services/ranking.py
```

---

#### LLM Strategy Experiments

```text
app/services/expansion.py
app/services/rerank.py
```

---

#### Data Schema Updates

```text
app/models/schemas.py
```

---

#### Route Wiring Only

```text
app/routes/
```

---

---

## 🚀 Version

```
TestCases-RAG-Version-2.0
```

---

✅ Built for scalable testing knowledge discovery with AI-powered semantic search and ranking.

```
```
