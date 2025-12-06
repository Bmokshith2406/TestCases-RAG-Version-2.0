# TestCases-RAG-Version-2.0
## Intelligent Test Case Search Platform – Modular Edition

---

## Overview

This project is a production-grade backend platform for uploading, enriching, indexing, deduplicating, and semantically searching software test cases using:

- FastAPI for APIs  
- MongoDB Atlas for persistence and vector search  
- SentenceTransformers (all-MiniLM-L6-v2) for embeddings  
- Google Gemini for enrichment, query expansion, and reranking  
- JWT Authentication with role-based access  
- Advanced ranking heuristics with A/B testing  
- Automatic duplicate detection and suppression  
- Query caching  
- Audit logging and search metrics  

This refactor modularizes the original single-file application into clean service layers for easier debugging, scaling, and experimentation workflows.

---

## Project Structure

```

app/
├── main.py                # App startup and lifespan orchestration
│
├── core/                  # Global configuration and security
│   ├── config.py          # Environment variables and constants
│   ├── logging.py         # Structured logging
│   ├── cache.py           # In-memory query caching
│   ├── security.py        # JWT and password hashing
│   └── analytics.py      # Audit logging & metrics
│
├── db/
│   └── mongo.py           # MongoDB connection and helpers
│
├── models/
│   ├── schemas.py         # Pydantic DTO schemas
│   └── users.py           # MongoDB user CRUD helpers
│
├── services/
│   ├── embeddings.py        # SentenceTransformer lifecycle and batching
│   ├── keywords.py          # Keyword extraction
│   ├── enrichment.py        # Gemini test-case enrichment
│   ├── expansion.py         # Gemini query expansion
│   ├── rerank.py            # Gemini reranking logic
│   ├── ranking.py           # Multi-signal scoring + A/B ranking
│
│   # Deduplication Layer
│   ├── dedupe_search_helper.py   # Candidate retrieval (vector + fuzzy)
│   ├── dedupe_summary.py         # Summary normalization & fingerprinting
│   └── dedupe_verifier.py        # Duplicate verification thresholds
│
├── routes/
│   ├── auth.py             # Login and registration APIs
│   ├── upload.py           # CSV/XLSX ingestion, dedupe, enrichment pipeline
│   ├── search.py           # Hybrid vector search + ranking
│   ├── update.py           # Test case updates and reprocessing
│   └── admin.py            # Admin maintenance and metrics APIs
│
└── middleware/             # Optional global middleware

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

Your `requirements.txt` should include:

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

## MongoDB Vector Search Requirements

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

Index Name:

```
vector_index
```

---

## Running the Application

Start the backend:

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

## Authentication and User Roles

### Register

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

POST `/auth/login` (x-www-form-urlencoded)

Response:

```json
{
  "access_token": "JWT_TOKEN",
  "token_type": "bearer"
}
```

---

### Usage

Add to request headers:

```
Authorization: Bearer YOUR_TOKEN
```

---

### Role Permissions

| Role   | Permissions                                  |
| ------ | -------------------------------------------- |
| viewer | Search only                                  |
| editor | Upload, update, delete individual test cases |
| admin  | Full control, bulk delete, metrics           |

---

---

## Uploading Test Cases

POST `/api/upload`
Authorization required: `editor` or `admin`

---

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

---

## Deduplication System (Auto-Dedupe)

Before any test case is stored or enriched, a **multi-layer duplicate detection pipeline** ensures that already indexed or semantically equivalent records are not added again.

---

### Dedupe Modules

| Module                    | Responsibility                                                         |
| ------------------------- | ---------------------------------------------------------------------- |
| `dedupe_search_helper.py` | Vector KNN and fuzzy text matching to retrieve duplicate candidates    |
| `dedupe_summary.py`       | Content normalization + semantic fingerprint generation                |
| `dedupe_verifier.py`      | Final similarity evaluation and threshold-based duplicate confirmation |

---

### Dedupe Workflow

```
Incoming Test Case
      ↓
Text Normalization & Summary Fingerprinting
      ↓
Candidate Retrieval
(Vector Search + Fuzzy Matching)
      ↓
Semantic Similarity Verification
      ↓
Threshold Filtering
      ↓
Duplicate Decision
```

---

### Detection Signals

* Embedding cosine similarity
* Keyword overlap ratio
* Summary text similarity
* Test step semantic alignment
* Feature name correlation

---

### Behavior

* Confirmed duplicates are **skipped**
* Duplicate metadata is logged
* Only truly unique test cases proceed to enrichment and storage

---

### Result

This ensures:

* No index contamination
* Reduced storage usage
* Higher ranking precision
* Improved search relevance

---

---

## Upload Processing Pipeline

```
1. File ingestion
2. Content normalization
3. Multi-layer duplicate detection
4. Gemini enrichment
5. Keyword extraction
6. SentenceTransformer embedding
7. Mean vector generation
8. MongoDB insert & vector indexing
```

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
MongoDB Vector Search
   →
Local Rank Fusion
   →
Optional Gemini Re-ranking
   →
Diversity Filtering
   →
Final TOP-K Results
```

---

### Ranking Signals

---

**Ranking A – Baseline**

```
0.60 × Vector similarity
0.25 × Max cosine similarity
+ Token match boosts
```

---

**Ranking B – Enhanced**

```
0.45 × Vector similarity
0.20 × Semantic similarity
0.12 × Keyword overlap
0.08 × Feature name match
0.05 × Token density
0.05 × Popularity weighting
```

---

Specify:

```json
"ranking_variant": "A" | "B"
```

---

---

## Updating Records

PUT `/api/update/{doc_id}`

```json
{
  "feature": "Payments",
  "priority": "Critical",
  "tags": ["Smoke", "API"]
}
```

---

### Automatic Triggers

* Re-enrichment using Gemini (if needed)
* Re-embedding
* Vector recalculation

---

---

## Admin APIs

### Get All Test Cases

GET `/api/get-all`

---

### Delete all Records

POST `/api/delete-all?confirm=true`
Admin only

---

### Delete Individual Record

DELETE `/api/testcase/{id}`

---

### Metrics

GET `/api/metrics`

```json
{
  "queries_today": 281,
  "top_features": ["Login", "Checkout"]
}
```

---

## Audit Logging

Each search request logs:

* Timestamp
* Endpoint
* User
* Filters and query payload
* Ranking variant
* Result count

Mongo collection:

```
api_audit_logs
```

---

---

## Development Workflow

---

### Ranking & Signals

```
app/services/ranking.py
```

---

### LLM Strategy

```
app/services/expansion.py
app/services/rerank.py
```

---

### Data Models

```
app/models/schemas.py
```

---

### API Wiring

```
app/routes/
```

---

---

## Version

```
TestCases-RAG-Version-2.0
```

---

Built for scalable, deduplicated test knowledge discovery using AI-powered semantic search and ranking.

```
```
