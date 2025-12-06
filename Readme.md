# TestCases-RAG-Version-2.0
## Intelligent Test Case Search Platform – Modular Edition

---

## Overview

This project is a production-grade backend platform for uploading, enriching, indexing, and semantically searching software test cases using:

- FastAPI for APIs  
- MongoDB Atlas for persistence and vector search  
- SentenceTransformers (all-MiniLM-L6-v2) for embeddings  
- Google Gemini for enrichment, query expansion, and reranking  
- JWT Authentication with role-based access  
- Advanced ranking heuristics and A/B testing  
- Search caching  
- Audit logging and metrics

This refactor modularizes the original single-file application into clean service layers for easier debugging, scaling, and experimentation workflows.

---

## Project Structure

```

app/
├── main.py                # App startup and lifespan orchestration
│
├── core/                  # Global configuration and security
│   ├── config.py          # Env and constants
│   ├── logging.py         # Structured logging
│   ├── cache.py           # In-memory query caching
│   ├── security.py        # JWT and password hashing
│   └── analytics.py      # Audit logging
│
├── db/
│   └── mongo.py           # MongoDB connection and helpers
│
├── models/
│   ├── schemas.py         # Pydantic DTO schemas
│   └── users.py           # Mongo user CRUD helpers
│
├── services/
│   ├── embeddings.py     # SentenceTransformer lifecycle and batching
│   ├── keywords.py       # Keyword extraction and fallback summaries
│   ├── enrichment.py     # Gemini test-case enrichment
│   ├── expansion.py      # Gemini query expansion
│   ├── rerank.py          # Gemini reranking
│   └── ranking.py         # Multi-signal scoring and A/B logic
│
├── routes/
│   ├── auth.py            # Login and register APIs
│   ├── upload.py          # CSV/XLSX ingestion and enrichment pipeline
│   ├── search.py          # Hybrid vector search with ranking
│   ├── update.py          # Test case updates and reprocessing
│   └── admin.py           # Admin maintenance and metrics APIs
│
└── middleware/            # Optional global middleware (future work)

```

---

## Setup and Installation

### Python Version

```

Python 3.10+

````

---

### Clone and Setup Virtual Environment

```bash
git clone <your-repository>
cd <your-repository>

python -m venv .venv
source .venv/bin/activate      # macOS/Linux
.venv\Scripts\activate         # Windows
````

---

### Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Required Packages

`requirements.txt` should include:

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

## Environment Variables

Create a `.env` file:

```env
GOOGLE_API_KEY=your-google-api-key
MONGO_CONNECTION_STRING=your-mongodb-uri
JWT_SECRET_KEY=change-me-in-production
```

---

## MongoDB Requirements

Create a vector search index on the `main_vector` field:

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

Index name:

```
vector_index
```

The name must match exactly.

---

## Running the Application

Start the backend server:

```bash
uvicorn app.main:app --reload
```

---

### API Access

| Resource   | URL                                                        |
| ---------- | ---------------------------------------------------------- |
| API Base   | [http://localhost:8000](http://localhost:8000)             |
| Swagger UI | [http://localhost:8000/docs](http://localhost:8000/docs)   |
| ReDoc      | [http://localhost:8000/redoc](http://localhost:8000/redoc) |

---

---

## Authentication and User Roles

### Create Account

POST `/auth/register`

```json
{
  "username": "admin",
  "password": "test123",
  "role": "admin"
}
```

---

### Login

POST `/auth/login`

Form-encoded

Returns:

```json
{
  "access_token": "JWT_TOKEN",
  "token_type": "bearer"
}
```

---

### Use Token

Add header:

```
Authorization: Bearer YOUR_TOKEN
```

---

### Role Permissions

| Role   | Permissions                                  |
| ------ | -------------------------------------------- |
| viewer | Search only                                  |
| editor | Upload, update, delete individual test cases |
| admin  | Full control, delete all data, metrics       |

---

---

## Uploading Test Cases

POST `/api/upload`
Authorization required: editor or admin

### Accepted Files

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

### Processing Pipeline

1. File ingestion
2. Gemini summary generation
3. Keyword extraction
4. SentenceTransformer embedding
5. Mean vector creation
6. MongoDB insert and indexing

---

---

## Searching Test Cases

POST `/api/search`

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

### Search Pipeline

```
User Query
   →
Embedding
   →
MongoDB vector search
   →
Local rank fusion
   →
Optional Gemini reranking
   →
Diversity filtering
   →
Final TOP-K results
```

---

### Scoring Signals

---

**Ranking A — Baseline**

```
0.60 × Vector similarity
0.25 × Max cosine similarity
+ Token match boosts
```

---

**Ranking B — Enhanced**

```
0.45 × Vector similarity
0.20 × Semantic similarity
0.12 × Keyword overlap
0.08 × Feature name match
0.05 × Token density
0.05 × Popularity weighting
```

---

Specify ranking mode:

```json
"ranking_variant": "A" | "B"
```

---

---

## Updating Records

PUT `/api/update/{doc_id}`

Example:

```json
{
  "feature": "Payments",
  "priority": "Critical",
  "tags": ["Smoke", "API"]
}
```

---

### Automatic Triggers

* Gemini re-enrichment if required
* Re-embedding
* Main vector recalculation

---

---

## Admin APIs

### Get All Test Cases

GET `/api/get-all`

---

### Delete All Test Cases

POST `/api/delete-all?confirm=true`
Admin only

---

### Delete Single Test Case

DELETE `/api/testcase/{id}`

---

### Metrics

GET `/api/metrics`

Returns:

```json
{
  "queries_today": 281,
  "top_features": ["Login", "Checkout"]
}
```

---

---

## Audit Logging

Every search request records:

* Timestamp
* Endpoint
* User
* Request payload
* Ranking variant
* Result count

Mongo collection:

```
api_audit_logs
```

---

## Why Audit Logging Matters

* Quality monitoring
* Ranking experimentation feedback
* Popular query discovery
* Search UX improvements

---

---

## Development Workflow

---

### Ranking Updates

```
app/services/ranking.py
```

---

### LLM Experiments

```
app/services/expansion.py
app/services/rerank.py
```

---

### Schema Updates

```
app/models/schemas.py
```

---

### Route Wiring

```
app/routes/
```

---

---

## Version

```
TestCases-RAG-Version-2.0
```

