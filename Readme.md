````md
# Intelligent Test Case Search Platform  
## TestCases-RAG — Modular Edition v2.0

A **production-grade AI-powered backend platform** designed for uploading, enriching, indexing, and **semantically searching software test cases** using **LLM enrichment, vector search, and ranking experimentation**.

This edition refactors the original monolithic system into a **clean modular architecture** built for:

- Scalability  
- Observability  
- Ranking experimentation (A/B testing)  
- Maintainability  
- Real-world production workloads  

---

## Core Technology Stack

| Layer | Technology |
|------|-------------|
| API Framework | FastAPI |
| Database | MongoDB Atlas (Vector Search enabled) |
| Embeddings | SentenceTransformers – `all-MiniLM-L6-v2` |
| LLM Enrichment & Rerank | Google Gemini |
| Authentication | JWT with Role-Based Access Control |
| Ranking | Multi-signal weighted fusion + A/B testing |
| Caching | Query-level in-memory cache |
| Analytics | Audit logs + Metrics endpoints |

---

## Platform Capabilities

### AI Enrichment Pipeline
- Gemini extracts:
  - Key intent terms  
  - Relevant summary tokens  
  - Semantic boosters  
  - Search tags  

### Vector Search Engine
- SentenceTransformer generates:
  - Chunk embeddings  
  - Aggregate mean vectors  

- MongoDB Atlas `$vectorSearch` handles retrieval

### Ranking System
Signals used for scoring:
- Lexical overlap  
- Vector similarity  
- Feature relevance  
- Semantic closeness  
- Token density boosting  
- Popularity weighting  

### Secure Access Control

| Role | Capabilities |
|------|---------------|
| Viewer | Search only |
| Editor | Upload, update, delete test cases |
| Admin | Full system access, purge operations, and metrics access |

---

## System Setup

### Requirements

- Python 3.10+  
- MongoDB Atlas cluster with Vector Search enabled  
- Google Gemini API key

---

## Clone and Initialize Environment

```bash
git clone <your-repository-url>
cd TestCases-RAG

python -m venv .venv

# Linux / macOS
source .venv/bin/activate

# Windows
.venv\Scripts\activate
````

---

## Install Dependencies

```bash
pip install -r requirements.txt
```

### Required Packages

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

## Environment Variables

Create a `.env` file:

```env
GOOGLE_API_KEY=your_gemini_api_key
MONGO_CONNECTION_STRING=your_mongodb_uri

JWT_SECRET_KEY=change_this_in_production
```

---

## MongoDB Vector Search Configuration

Create a vector index on the `main_vector` field:

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

### Index Name

```
vector_index
```

---

## Running the Service

```bash
uvicorn app.main:app --reload
```

### Service URL

```
http://localhost:8000
```

### Swagger Docs

```
http://localhost:8000/docs
```

---

## Authentication

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

Response:

```json
{
  "access_token": "<JWT>",
  "token_type": "bearer"
}
```

### Authorization Header

```
Authorization: Bearer <ACCESS_TOKEN>
```

---

## Uploading Test Cases

### Endpoint

```http
POST /api/upload
```

Access: `editor` or `admin`

---

### Supported File Formats

* `.csv`
* `.xlsx`

---

### Mandatory Columns

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

## Data Ingestion Pipeline

```text
Upload File
     ↓
Gemini enrichment (summaries, keywords, tags)
     ↓
SentenceTransformer embeddings
     ↓
Vector aggregation
     ↓
MongoDB vector insertion and indexing
```

---

## Intelligent Search

### Endpoint

```http
POST /api/search
```

---

### Example Request

```json
{
  "query": "payment gateway retry failure",
  "feature": "Checkout",
  "tags": ["Regression"],
  "priority": "High",
  "platform": "Mobile",
  "ranking_variant": "B"
}
```

---

## Search Pipeline

```text
Query
   ↓
Embedding generation
   ↓
MongoDB Vector Search
   ↓
Local rank fusion
   ↓
Optional Gemini reranking
   ↓
Diversity filtering
   ↓
Top-K semantic matches
```

---

## Ranking Models

### Variant A — Baseline

```text
0.60 × Vector similarity
0.25 × Max cosine similarity
+ Token overlap boosts
```

---

### Variant B — Enhanced

```text
0.45 × Vector similarity
0.20 × Semantic similarity
0.12 × Keyword overlap
0.08 × Feature matching
0.05 × Token density
0.05 × Popularity weighting
```

Selection:

```json
"ranking_variant": "A" | "B"
```

---

## Updating Records

### Endpoint

```http
PUT /api/update/{document_id}
```

---

### Example Request

```json
{
  "feature": "Payments",
  "priority": "Critical",
  "tags": ["Smoke", "API"]
}
```

---

### Automated Actions

* Gemini re-enrichment if fields change
* Text re-embedding
* Vector recalculation

---

## Admin Operations

| Endpoint                            |
| ----------------------------------- |
| `GET /api/get-all`                  |
| `DELETE /api/testcase/{id}`         |
| `POST /api/delete-all?confirm=true` |
| `GET /api/metrics`                  |

---

## Metrics Example

```json
{
  "queries_today": 281,
  "top_features": [
    "Login",
    "Checkout"
  ]
}
```

---

## Audit Logging

Recorded fields:

* Timestamp
* Authenticated user
* API endpoint
* Payload / search request
* Ranking variant used
* Result count

MongoDB Collection:

```
api_audit_logs
```

---

## Why This Platform Matters

This architecture enables:

* Controlled ranking experiments via A/B testing
* Continuous improvement through relevance monitoring
* Full observability of user search behavior
* Stable production-scale workflows
* Clean debugging and scalability

---

## Development and Customization

| Area             | Path                        |
| ---------------- | --------------------------- |
| Ranking logic    | `app/services/ranking.py`   |
| Query expansion  | `app/services/expansion.py` |
| Gemini reranking | `app/services/rerank.py`    |
| Data schemas     | `app/models/schemas.py`     |
| API routes       | `app/routes/`               |
| Authentication   | `app/routes/auth.py`        |

---

## Final Notes

TestCases-RAG Modular Edition v2.0 provides a robust search backend combining:

* Deep semantic embeddings
* LLM enrichment workflows
* Ranking experimentation
* Enterprise-grade API design
