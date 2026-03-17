# Vector Databases & pgvector — Complete Beginner's Guide

## What is a Vector Database?

A vector database is a special type of database designed to **store, index, and search vectors** (lists of numbers) efficiently.

### Why Can't We Just Use a Normal Database?

```
Normal Database (MySQL, PostgreSQL):
┌────────┬──────────┬───────────┐
│ id     │ name     │ age       │
├────────┼──────────┼───────────┤
│ 1      │ Rahim    │ 25        │
│ 2      │ Karim    │ 30        │
└────────┴──────────┴───────────┘

Search: SELECT * FROM users WHERE name = 'Rahim'
→ Exact match. Simple. Fast. ✅

But what if we want to search by MEANING?
Search: "Find documents SIMILAR to 'how to train a puppy'"
→ Normal DB can't do this ❌
   It can only match exact text, not MEANING
```

A vector database stores **embeddings** (vectors that represent meaning) and can find the **most similar vectors** lightning fast — even with millions of them.

```
Vector Database:
┌────────┬──────────────────────────┬──────────────────────────────┐
│ id     │ text                     │ vector (embedding)           │
├────────┼──────────────────────────┼──────────────────────────────┤
│ 1      │ "Dogs are loyal pets"    │ [0.12, -0.45, 0.78, ...]    │
│ 2      │ "Cats love to nap"       │ [0.09, -0.41, 0.65, ...]    │
│ 3      │ "Stock market crashed"   │ [-0.67, 0.22, -0.15, ...]   │
└────────┴──────────────────────────┴──────────────────────────────┘

Search: Find vectors most similar to embed("puppy training tips")
→ Returns row 1 (dogs) because meaning is closest ✅
```

---

## Types of Vector Databases

There are **3 categories** of vector databases:

### 1. Purpose-Built Vector Databases (Built from scratch for vectors)

| Database | Type | Notes |
|----------|------|-------|
| **Pinecone** | Cloud-only (managed) | Easiest to use, no infra management |
| **Weaviate** | Self-hosted or Cloud | Very flexible, has GraphQL API |
| **Qdrant** | Self-hosted or Cloud | Fast, written in Rust |
| **Milvus** | Self-hosted or Cloud | Designed for massive scale |
| **ChromaDB** | In-memory / local | Great for learning & prototyping |

### 2. Libraries (Not a database, just search in memory)

| Library | Notes |
|---------|-------|
| **FAISS** (Facebook) | Super fast, but no persistence by default |
| **Annoy** (Spotify) | Lightweight, good for read-heavy workloads |
| **ScaNN** (Google) | Optimized for Google-scale search |

### 3. Extensions on Existing Databases (Add vector search to a DB you already use)

| Extension | For | Notes |
|-----------|-----|-------|
| **pgvector** | PostgreSQL | Add vector search to your existing Postgres! |
| **MongoDB Atlas Vector** | MongoDB | Vector search built into MongoDB |
| **Elasticsearch kNN** | Elasticsearch | Vector search in Elastic |
| **Redis VSS** | Redis | Vector search in Redis |

---

## Why pgvector? Why Not Just Use Pinecone or ChromaDB?

This is the important question. Here's why pgvector is special:

### The Problem with Separate Vector Databases

```
Typical RAG Setup (Separate Vector DB):

┌──────────────┐     ┌──────────────┐
│ PostgreSQL   │     │ Pinecone     │
│              │     │              │
│ users table  │     │ vectors      │
│ orders table │     │ embeddings   │
│ products     │     │ metadata     │
│ payments     │     │              │
└──────┬───────┘     └──────┬───────┘
       │                     │
       │   Your App needs    │
       │   to talk to BOTH   │
       └──────────┬──────────┘
                  │
           ┌──────▼──────┐
           │  Your App   │
           │             │
           │ - Query Postgres for user info
           │ - Query Pinecone for similar docs
           │ - JOIN results manually in code 😫
           │ - Sync data between two systems 😫
           │ - Pay for two services 😫
           └─────────────┘
```

**Problems:**
1. **Two systems to manage** — double the infra, double the headaches
2. **Data sync issues** — what if Postgres and Pinecone get out of sync?
3. **Can't JOIN** — you can't easily combine vector search with regular SQL filters
4. **Extra cost** — paying for Pinecone + Postgres separately
5. **Extra latency** — two network round trips instead of one

### The pgvector Solution

```
pgvector Setup (Everything in ONE database):

┌──────────────────────────────┐
│ PostgreSQL + pgvector        │
│                              │
│ users table                  │
│ orders table                 │
│ products table               │
│ documents table              │
│   ├── id                     │
│   ├── content (text)         │
│   ├── embedding (vector)  ←── pgvector column!
│   ├── category (text)        │
│   ├── created_at (timestamp) │
│   └── user_id (FK to users)  │
│                              │
│ Everything in ONE place! 🎉  │
└──────────────┬───────────────┘
               │
        ┌──────▼──────┐
        │  Your App   │
        │             │
        │ ONE query that does:
        │ - Vector similarity search
        │ - SQL filters (WHERE category = 'tech')
        │ - JOINs (with users, orders, etc.)
        │ - All in a single SQL query! 🎉
        └─────────────┘
```

### When to Use pgvector vs Others

| Scenario | Best Choice | Why |
|----------|-------------|-----|
| Already using PostgreSQL | **pgvector** | No new infra, just add an extension |
| Small-medium dataset (< 10M vectors) | **pgvector** | More than enough performance |
| Need SQL JOINs with vector search | **pgvector** | Only option that does this natively |
| Learning / prototyping | **ChromaDB** | Simplest setup, zero config |
| Massive scale (100M+ vectors) | **Pinecone / Milvus** | Built for extreme scale |
| Serverless / no infra management | **Pinecone** | Fully managed cloud service |
| Need the fastest possible search | **FAISS / Qdrant** | Optimized purely for speed |
| Already using MongoDB | **MongoDB Atlas Vector** | No new infra |

---

## pgvector — Full Deep Dive

### What is pgvector?

pgvector is an **open-source extension** for PostgreSQL that adds:
- A new `vector` data type (to store embeddings)
- Distance operators (to compare vectors)
- Indexing algorithms (to search fast)

It turns your regular PostgreSQL into a vector database — with **zero** additional infrastructure.

### Installation

#### Option 1: On Ubuntu/Debian

```bash
# Install PostgreSQL if you don't have it
sudo apt install postgresql postgresql-contrib

# Install pgvector
sudo apt install postgresql-16-pgvector
# (replace 16 with your PostgreSQL version)

# Or build from source:
cd /tmp
git clone --branch v0.8.0 https://github.com/pgvector/pgvector.git
cd pgvector
make
sudo make install
```

#### Option 2: Using Docker (Easiest)

```bash
# Pull the official pgvector Docker image
docker run --name pgvector-db \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -p 5432:5432 \
  -d pgvector/pgvector:pg16
```

#### Option 3: Cloud PostgreSQL (Already has pgvector)

Most cloud providers now include pgvector:
- **Supabase** — pgvector enabled by default
- **AWS RDS for PostgreSQL** — supports pgvector
- **Google Cloud SQL** — supports pgvector
- **Azure Database for PostgreSQL** — supports pgvector
- **Neon** — serverless Postgres with pgvector
- **Railway** — one-click Postgres with pgvector

### Enabling pgvector

```sql
-- Connect to your database and enable the extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Verify it's installed
SELECT * FROM pg_extension WHERE extname = 'vector';
```

---

## Core Concepts in pgvector

### 1. The `vector` Data Type

pgvector adds a new column type called `vector(n)` where `n` is the number of dimensions.

```sql
-- Create a table with a vector column
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(384)    -- 384 dimensions (matches all-MiniLM-L6-v2)
);
```

```
What this looks like in the table:

id | content                         | embedding
---|---------------------------------|----------------------------------
1  | "Dogs are loyal pets"           | [0.12,-0.45,0.78,...] (384 numbers)
2  | "Cats love to nap"              | [0.09,-0.41,0.65,...] (384 numbers)
3  | "Stock market crashed"          | [-0.67,0.22,-0.15,...] (384 numbers)
```

Common dimension sizes:
| Embedding Model | Dimensions | vector type |
|----------------|------------|-------------|
| all-MiniLM-L6-v2 | 384 | `vector(384)` |
| all-mpnet-base-v2 | 768 | `vector(768)` |
| text-embedding-3-small (OpenAI) | 1536 | `vector(1536)` |
| text-embedding-3-large (OpenAI) | 3072 | `vector(3072)` |

### 2. Distance Operators (How to Compare Vectors)

pgvector gives you **3 ways** to measure how similar two vectors are:

| Operator | Name | What It Measures | Range | Use When |
|----------|------|-----------------|-------|----------|
| `<->` | L2 (Euclidean) Distance | Straight-line distance between points | 0 to ∞ (0 = identical) | General purpose |
| `<=>` | Cosine Distance | Angle between vectors | 0 to 2 (0 = identical) | Text similarity (most common for RAG) |
| `<#>` | Inner Product (Negative) | Dot product (negated) | -∞ to ∞ (more negative = more similar) | When vectors are normalized |

#### Visual Explanation

```
Euclidean Distance (<->):
  Measures the STRAIGHT LINE between two points

  A ●─────────────────● B     Distance = long (different)

  A ●──● C                    Distance = short (similar)


Cosine Distance (<=>):
  Measures the ANGLE between two vectors (ignores length)

        B
       ╱
      ╱  angle = small → similar
     ╱
  ──●──────────── A

        C
       ╱
      ╱
     ╱   angle = large → different
    ╱
  ──●──────────── A
```

**For text/RAG, always use Cosine Distance (`<=>`)** because it measures meaning similarity regardless of text length.

### 3. Basic SQL Operations

#### INSERT vectors

```sql
-- Insert a document with its embedding
INSERT INTO documents (content, embedding)
VALUES (
    'Python is a programming language',
    '[0.12, -0.45, 0.78, 0.33, ...]'    -- your 384-dimensional vector as a string
);
```

#### SELECT nearest vectors (Similarity Search!)

```sql
-- Find the 5 most similar documents to a query vector
SELECT
    id,
    content,
    embedding <=> '[0.11, -0.43, 0.80, ...]' AS distance    -- cosine distance
FROM documents
ORDER BY embedding <=> '[0.11, -0.43, 0.80, ...]'           -- sort by similarity
LIMIT 5;
```

```
Result:
id | content                              | distance
---|--------------------------------------|----------
1  | "Python is a programming language"   | 0.0523     ← most similar
4  | "JavaScript is used for web dev"     | 0.1872
7  | "Ruby was created in 1995"           | 0.2341
2  | "Machine learning uses data"         | 0.5692
9  | "The Eiffel Tower is in Paris"       | 0.8901     ← least similar
```

#### Combine Vector Search with SQL Filters

This is pgvector's **superpower** — you can mix vector search with regular SQL!

```sql
-- Find similar documents, but ONLY in the 'technology' category, created this year
SELECT
    d.id,
    d.content,
    d.embedding <=> '[0.11, -0.43, 0.80, ...]' AS distance,
    d.category,
    u.name AS author
FROM documents d
JOIN users u ON d.user_id = u.id
WHERE d.category = 'technology'                     -- SQL filter
  AND d.created_at >= '2025-01-01'                  -- SQL filter
ORDER BY d.embedding <=> '[0.11, -0.43, 0.80, ...]' -- vector similarity
LIMIT 5;
```

**You CANNOT do this with Pinecone or ChromaDB** — they don't support SQL JOINs.

---

## Indexing (Making Search FAST)

Without an index, pgvector has to compare your query against **every single vector** in the table. This is called a "sequential scan" and it's SLOW for large datasets.

```
Without Index:
Query vector → Compare with row 1 → Compare with row 2 → ... → Compare with row 1,000,000
Time: 😴😴😴 (seconds to minutes)

With Index:
Query vector → Smart data structure narrows it down → Compare with ~100 candidates
Time: ⚡ (milliseconds)
```

### Two Types of Indexes in pgvector

#### 1. IVFFlat (Inverted File with Flat Compression)

**How it works:** Divides all vectors into clusters (groups). During search, only looks at the nearest clusters.

```
Imagine a library with 1 million books:

Without index: Check EVERY book → 1,000,000 comparisons 😩

With IVFFlat:
  Step 1: Books are grouped into 100 sections by topic
  Step 2: Your query matches "Science" section
  Step 3: Only search 10,000 books in that section → 100x faster! 🚀
```

```sql
-- Create IVFFlat index
-- lists = number of clusters (rule of thumb: sqrt of row count)
-- For 1,000,000 rows → lists = 1000

CREATE INDEX ON documents
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);
```

```
How to choose the 'lists' value:
┌─────────────────┬───────────────┐
│ Number of rows  │ lists value   │
├─────────────────┼───────────────┤
│ < 10,000        │ Don't need    │
│                 │ an index yet  │
│ 10,000          │ 100           │
│ 100,000         │ 316           │
│ 1,000,000       │ 1000          │
│ 10,000,000      │ 3162          │
└─────────────────┴───────────────┘
Formula: lists = sqrt(number_of_rows)
```

**Important:** IVFFlat needs data to exist BEFORE creating the index. It clusters existing vectors. If you create the index on an empty table, it won't work well.

```sql
-- ❌ BAD: Create index on empty table, then insert data
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
INSERT INTO documents ...  -- index is useless

-- ✅ GOOD: Insert data first, then create index
INSERT INTO documents ...  -- add all your data
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

#### 2. HNSW (Hierarchical Navigable Small World)

**How it works:** Builds a multi-layer graph. Top layer = few nodes, far apart. Bottom layer = all nodes, close together. Search starts at the top and drills down.

```
HNSW Graph (simplified):

Layer 3 (top):     A ──────────────── D          (few nodes, big jumps)
                   │                  │
Layer 2:           A ─── B ────────── D          (more nodes, medium jumps)
                   │     │            │
Layer 1:           A ─ B ─ C ── D ─── E          (even more nodes)
                   │   │   │    │     │
Layer 0 (bottom):  A-B-C-D-E-F-G-H-I-J          (ALL nodes, small jumps)

Search: Start at top layer → Jump to rough area → Drill down → Find exact match
```

```sql
-- Create HNSW index
-- m = connections per node (higher = more accurate but more memory)
-- ef_construction = build quality (higher = better index but slower to build)

CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);
```

### IVFFlat vs HNSW — Which to Use?

| Feature | IVFFlat | HNSW |
|---------|---------|------|
| **Search Speed** | Fast | Faster |
| **Search Accuracy** | Good (95%+) | Better (99%+) |
| **Build Speed** | Fast | Slower |
| **Memory Usage** | Lower | Higher |
| **Works on empty table?** | ❌ No (needs data first) | ✅ Yes |
| **Insertions after index** | Needs periodic reindex | Handles well |
| **Best for** | Static data, cost-sensitive | Dynamic data, accuracy-critical |

**Recommendation:**
- **Use HNSW** if you can afford the memory — it's better in almost every way
- **Use IVFFlat** if you're memory-constrained or have very large datasets (10M+ vectors)

### Index Distance Operations

You must match the index operator to the distance type you'll use in queries:

| Distance Type | Index Operator | SQL Operator |
|---------------|---------------|--------------|
| L2 (Euclidean) | `vector_l2_ops` | `<->` |
| Cosine | `vector_cosine_ops` | `<=>` |
| Inner Product | `vector_ip_ops` | `<#>` |

```sql
-- For cosine similarity (most common for text/RAG)
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);

-- For euclidean distance
CREATE INDEX ON documents USING hnsw (embedding vector_l2_ops);
```

### Tuning Search Accuracy vs Speed

After creating the index, you can tune search behavior per query:

```sql
-- For IVFFlat: 'probes' = how many clusters to search
-- Higher probes = more accurate but slower
SET ivfflat.probes = 10;         -- default is 1
-- Rule of thumb: probes = sqrt(lists)

-- For HNSW: 'ef_search' = size of candidate list during search
-- Higher ef_search = more accurate but slower
SET hnsw.ef_search = 100;        -- default is 40
```

```
Accuracy vs Speed tradeoff:

probes/ef_search LOW  → ⚡ Fast, 😕 might miss some results
probes/ef_search HIGH → 🐢 Slower, 🎯 finds the best results

          Speed ◄──────────────────────► Accuracy

IVFFlat:  probes=1     probes=10    probes=100
HNSW:     ef_search=10 ef_search=100 ef_search=500
```

---

## Complete Working Example: RAG with pgvector + Python

### Setup

```bash
pip install psycopg2-binary sentence-transformers openai
```

### Step-by-Step Code

```python
import psycopg2
from sentence_transformers import SentenceTransformer
import openai
import os

# ============================================
# STEP 1: Connect to PostgreSQL
# ============================================
conn = psycopg2.connect(
    host="localhost",
    port=5432,
    dbname="mydb",
    user="postgres",
    password="mysecretpassword"
)
conn.autocommit = True
cur = conn.cursor()

# ============================================
# STEP 2: Enable pgvector and create table
# ============================================
cur.execute("CREATE EXTENSION IF NOT EXISTS vector;")

cur.execute("""
    CREATE TABLE IF NOT EXISTS documents (
        id SERIAL PRIMARY KEY,
        content TEXT NOT NULL,
        category TEXT,
        embedding vector(384)      -- 384 dims for all-MiniLM-L6-v2
    );
""")
print("✅ Table created")

# ============================================
# STEP 3: Load embedding model
# ============================================
model = SentenceTransformer('all-MiniLM-L6-v2')

# ============================================
# STEP 4: Insert documents with embeddings
# ============================================
documents = [
    ("Python was created by Guido van Rossum in 1991.", "programming"),
    ("JavaScript was created by Brendan Eich in 1995.", "programming"),
    ("Machine learning uses data to learn patterns.", "ai"),
    ("Deep learning is a subset of machine learning.", "ai"),
    ("The Eiffel Tower was built in 1889 in Paris.", "history"),
    ("The Great Wall of China is over 13,000 miles long.", "history"),
    ("PostgreSQL is an open-source relational database.", "programming"),
    ("Neural networks are inspired by the human brain.", "ai"),
    ("Django is a Python web framework.", "programming"),
    ("The refund policy allows returns within 30 days.", "policy"),
]

for content, category in documents:
    # Generate embedding for each document
    embedding = model.encode(content).tolist()

    cur.execute(
        """INSERT INTO documents (content, category, embedding)
           VALUES (%s, %s, %s::vector)""",
        (content, category, str(embedding))
    )

print(f"✅ Inserted {len(documents)} documents")

# ============================================
# STEP 5: Create HNSW index for fast search
# ============================================
cur.execute("""
    CREATE INDEX IF NOT EXISTS documents_embedding_idx
    ON documents
    USING hnsw (embedding vector_cosine_ops);
""")
print("✅ HNSW index created")

# ============================================
# STEP 6: Semantic search function
# ============================================
def search(query, top_k=3, category_filter=None):
    """Search for documents similar to the query."""
    # Convert query to vector
    query_embedding = model.encode(query).tolist()

    if category_filter:
        # Vector search + SQL filter combined!
        cur.execute("""
            SELECT id, content, category,
                   embedding <=> %s::vector AS distance
            FROM documents
            WHERE category = %s
            ORDER BY embedding <=> %s::vector
            LIMIT %s;
        """, (str(query_embedding), category_filter, str(query_embedding), top_k))
    else:
        cur.execute("""
            SELECT id, content, category,
                   embedding <=> %s::vector AS distance
            FROM documents
            ORDER BY embedding <=> %s::vector
            LIMIT %s;
        """, (str(query_embedding), str(query_embedding), top_k))

    results = cur.fetchall()
    return results

# ============================================
# STEP 7: Test it!
# ============================================

# Search 1: General search
print("\n🔍 Query: 'Who created Python?'")
results = search("Who created Python?")
for id, content, category, distance in results:
    print(f"  [{category}] {content} (distance: {distance:.4f})")

# Search 2: With category filter
print("\n🔍 Query: 'learning from data' (AI category only)")
results = search("learning from data", category_filter="ai")
for id, content, category, distance in results:
    print(f"  [{category}] {content} (distance: {distance:.4f})")

# Search 3: Policy search
print("\n🔍 Query: 'Can I return my product?'")
results = search("Can I return my product?")
for id, content, category, distance in results:
    print(f"  [{category}] {content} (distance: {distance:.4f})")
```

**Expected Output:**

```
✅ Table created
✅ Inserted 10 documents
✅ HNSW index created

🔍 Query: 'Who created Python?'
  [programming] Python was created by Guido van Rossum in 1991. (distance: 0.1523)
  [programming] JavaScript was created by Brendan Eich in 1995. (distance: 0.4218)
  [programming] Django is a Python web framework. (distance: 0.5102)

🔍 Query: 'learning from data' (AI category only)
  [ai] Machine learning uses data to learn patterns. (distance: 0.2891)
  [ai] Deep learning is a subset of machine learning. (distance: 0.4523)
  [ai] Neural networks are inspired by the human brain. (distance: 0.5871)

🔍 Query: 'Can I return my product?'
  [policy] The refund policy allows returns within 30 days. (distance: 0.3214)
  [history] The Great Wall of China is over 13,000 miles long. (distance: 0.8901)
  [history] The Eiffel Tower was built in 1889 in Paris. (distance: 0.9102)
```

---

## Complete RAG Pipeline with pgvector + OpenAI

Now let's combine pgvector with an LLM to build a full RAG system:

```python
import psycopg2
from sentence_transformers import SentenceTransformer
from openai import OpenAI
import os

# Setup
embed_model = SentenceTransformer('all-MiniLM-L6-v2')
llm_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

conn = psycopg2.connect(
    host="localhost", dbname="mydb",
    user="postgres", password="mysecretpassword"
)
conn.autocommit = True
cur = conn.cursor()

# ============================================
# The RAG function
# ============================================
def ask_rag(question, top_k=3, similarity_threshold=0.5):
    """
    Full RAG pipeline:
    1. Embed the question
    2. Search pgvector for relevant chunks
    3. Filter by similarity threshold
    4. Send context + question to LLM
    5. Return answer with sources
    """

    # Step 1: Embed the question
    query_vector = embed_model.encode(question).tolist()

    # Step 2: Search pgvector
    cur.execute("""
        SELECT id, content, category,
               1 - (embedding <=> %s::vector) AS similarity
        FROM documents
        ORDER BY embedding <=> %s::vector
        LIMIT %s;
    """, (str(query_vector), str(query_vector), top_k))
    # Note: 1 - cosine_distance = cosine_similarity (so higher = more similar)

    results = cur.fetchall()

    # Step 3: Filter by threshold
    relevant = [(id, content, cat, sim) for id, content, cat, sim in results if sim >= similarity_threshold]

    if not relevant:
        return {
            "answer": "I don't have enough information in my knowledge base to answer that.",
            "sources": [],
            "confidence": "low"
        }

    # Step 4: Build prompt with context
    context = "\n".join([f"- {content}" for _, content, _, _ in relevant])

    system_prompt = """You are a helpful assistant. Answer the question using ONLY
the provided context. If the context doesn't contain the answer, say
"I don't have that information." Do NOT make up answers."""

    user_prompt = f"""Context:
{context}

Question: {question}

Answer:"""

    # Step 5: Send to LLM
    response = llm_client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ],
        temperature=0
    )

    answer = response.choices[0].message.content

    return {
        "answer": answer,
        "sources": [{"content": content, "similarity": f"{sim:.2f}"} for _, content, _, sim in relevant],
        "confidence": "high" if relevant[0][3] > 0.7 else "medium"
    }

# ============================================
# Use it!
# ============================================
result = ask_rag("What programming language was created by Guido?")

print(f"Answer: {result['answer']}")
print(f"Confidence: {result['confidence']}")
print(f"Sources:")
for source in result['sources']:
    print(f"  - [{source['similarity']}] {source['content']}")

# Output:
# Answer: Python was created by Guido van Rossum in 1991.
# Confidence: high
# Sources:
#   - [0.85] Python was created by Guido van Rossum in 1991.
#   - [0.58] JavaScript was created by Brendan Eich in 1995.
```

---

## Loading Real Documents (PDFs, Text Files) into pgvector

In the real world, you don't manually type documents. You load them from files:

```python
import psycopg2
from sentence_transformers import SentenceTransformer
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader, TextLoader
import os

# Setup
model = SentenceTransformer('all-MiniLM-L6-v2')
conn = psycopg2.connect(host="localhost", dbname="mydb", user="postgres", password="pass")
conn.autocommit = True
cur = conn.cursor()

# Create table
cur.execute("""
    CREATE TABLE IF NOT EXISTS knowledge_base (
        id SERIAL PRIMARY KEY,
        content TEXT NOT NULL,
        source_file TEXT,
        page_number INTEGER,
        chunk_index INTEGER,
        embedding vector(384)
    );
""")

# ============================================
# Function to load and store a document
# ============================================
def ingest_document(file_path):
    """Load a document, chunk it, embed it, and store in pgvector."""

    # Step 1: Load the file
    if file_path.endswith('.pdf'):
        loader = PyPDFLoader(file_path)
    elif file_path.endswith('.txt'):
        loader = TextLoader(file_path)
    else:
        raise ValueError(f"Unsupported file type: {file_path}")

    documents = loader.load()
    print(f"📄 Loaded {len(documents)} page(s) from {file_path}")

    # Step 2: Chunk the document
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,
        chunk_overlap=50
    )
    chunks = splitter.split_documents(documents)
    print(f"✂️  Split into {len(chunks)} chunks")

    # Step 3: Embed and store each chunk
    for i, chunk in enumerate(chunks):
        embedding = model.encode(chunk.page_content).tolist()

        cur.execute("""
            INSERT INTO knowledge_base (content, source_file, page_number, chunk_index, embedding)
            VALUES (%s, %s, %s, %s, %s::vector)
        """, (
            chunk.page_content,
            os.path.basename(file_path),
            chunk.metadata.get('page', 0),
            i,
            str(embedding)
        ))

    print(f"✅ Stored {len(chunks)} chunks in pgvector")

# ============================================
# Ingest multiple files
# ============================================
ingest_document("company_policy.pdf")
ingest_document("product_manual.pdf")
ingest_document("faq.txt")

# Create index AFTER inserting all data (especially if using IVFFlat)
cur.execute("""
    CREATE INDEX IF NOT EXISTS kb_embedding_idx
    ON knowledge_base
    USING hnsw (embedding vector_cosine_ops);
""")
print("✅ Index created")
```

---

## pgvector with ORMs (SQLAlchemy / Django)

### Using SQLAlchemy

```bash
pip install sqlalchemy pgvector
```

```python
from sqlalchemy import create_engine, Column, Integer, String, Text
from sqlalchemy.orm import declarative_base, Session
from pgvector.sqlalchemy import Vector

Base = declarative_base()
engine = create_engine("postgresql://postgres:password@localhost/mydb")

# Define model with vector column
class Document(Base):
    __tablename__ = "documents"

    id = Column(Integer, primary_key=True)
    content = Column(Text, nullable=False)
    category = Column(String(50))
    embedding = Column(Vector(384))     # pgvector column!

Base.metadata.create_all(engine)

# Insert
with Session(engine) as session:
    doc = Document(
        content="Python is awesome",
        category="programming",
        embedding=model.encode("Python is awesome").tolist()
    )
    session.add(doc)
    session.commit()

# Search (find similar documents)
with Session(engine) as session:
    query_vector = model.encode("What programming language should I learn?").tolist()

    results = (
        session.query(
            Document,
            Document.embedding.cosine_distance(query_vector).label("distance")
        )
        .order_by(Document.embedding.cosine_distance(query_vector))
        .limit(5)
        .all()
    )

    for doc, distance in results:
        print(f"[{doc.category}] {doc.content} (distance: {distance:.4f})")
```

### Using Django

```bash
pip install django pgvector
```

```python
# models.py
from django.db import models
from pgvector.django import VectorField, HnswIndex

class Document(models.Model):
    content = models.TextField()
    category = models.CharField(max_length=50)
    embedding = VectorField(dimensions=384)

    class Meta:
        indexes = [
            HnswIndex(
                name="doc_embedding_idx",
                fields=["embedding"],
                m=16,
                ef_construction=64,
                opclasses=["vector_cosine_ops"],
            ),
        ]

# views.py
from pgvector.django import CosineDistance

def search_documents(query_vector):
    results = (
        Document.objects
        .annotate(distance=CosineDistance("embedding", query_vector))
        .order_by("distance")[:5]
    )
    return results
```

---

## Performance Tips & Best Practices

### 1. Batch Insert (Don't Insert One by One)

```python
# ❌ BAD: Insert one at a time (slow for large datasets)
for chunk in chunks:
    cur.execute("INSERT INTO documents ...")

# ✅ GOOD: Batch insert with execute_values
from psycopg2.extras import execute_values

data = []
for chunk in chunks:
    embedding = model.encode(chunk).tolist()
    data.append((chunk, str(embedding)))

execute_values(
    cur,
    "INSERT INTO documents (content, embedding) VALUES %s",
    data,
    template="(%s, %s::vector)"
)
```

### 2. Use COPY for Very Large Datasets

```python
# For millions of vectors, COPY is the fastest way to load data
import csv
import io

# Prepare data in memory
buffer = io.StringIO()
writer = csv.writer(buffer, delimiter='\t')
for content, embedding in data:
    writer.writerow([content, str(embedding)])

buffer.seek(0)

# COPY into PostgreSQL (fastest bulk load method)
cur.copy_from(buffer, 'documents', columns=('content', 'embedding'), sep='\t')
```

### 3. Right-Size Your Vectors

```
More dimensions = More accurate BUT more storage + slower search

384 dimensions  → 384 × 4 bytes = 1.5 KB per vector
1536 dimensions → 1536 × 4 bytes = 6 KB per vector
3072 dimensions → 3072 × 4 bytes = 12 KB per vector

1 million documents:
  384 dims  → ~1.5 GB storage
  1536 dims → ~6 GB storage
  3072 dims → ~12 GB storage
```

Start with `all-MiniLM-L6-v2` (384 dims) for prototyping. Upgrade to larger models only if search quality needs improvement.

### 4. Tune Your PostgreSQL for Vector Workloads

```sql
-- Increase working memory for sorting large vector results
SET work_mem = '256MB';

-- Increase maintenance memory for building indexes
SET maintenance_work_mem = '512MB';

-- Allow parallel query execution
SET max_parallel_workers_per_gather = 4;

-- For HNSW: Tune ef_search per query
SET hnsw.ef_search = 100;    -- default 40, increase for better accuracy

-- For IVFFlat: Tune probes per query
SET ivfflat.probes = 10;     -- default 1, increase for better accuracy
```

### 5. Partial Indexes (Index Only What You Need)

```sql
-- Only index documents in the 'active' category
-- Saves storage and speeds up searches on that category
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WHERE category = 'active';
```

---

## Monitoring & Debugging

### Check Index Usage

```sql
-- Is PostgreSQL actually using your index?
EXPLAIN ANALYZE
SELECT id, content, embedding <=> '[0.1, 0.2, ...]'::vector AS distance
FROM documents
ORDER BY embedding <=> '[0.1, 0.2, ...]'::vector
LIMIT 5;

-- Look for "Index Scan using documents_embedding_idx" in the output
-- If you see "Seq Scan" instead, the index is NOT being used!
```

### Common Reasons Index Isn't Used

| Problem | Fix |
|---------|-----|
| Table too small (< 10K rows) | PostgreSQL decides seq scan is faster — that's OK |
| Wrong operator | Use `<=>` for cosine, `<->` for L2 — must match index |
| LIMIT too high | Index works best with small LIMIT (< 100). High LIMIT may trigger seq scan |
| Missing `ORDER BY` | Must ORDER BY the distance expression for index to kick in |

### Check Table and Index Size

```sql
-- Table size
SELECT pg_size_pretty(pg_total_relation_size('documents'));

-- Index size
SELECT pg_size_pretty(pg_relation_size('documents_embedding_idx'));

-- Row count
SELECT count(*) FROM documents;
```

---

## pgvector vs Other Vector Databases — Final Comparison

| Feature | pgvector | Pinecone | ChromaDB | Qdrant |
|---------|----------|----------|----------|--------|
| **SQL Support** | ✅ Full SQL | ❌ | ❌ | ❌ |
| **JOINs** | ✅ | ❌ | ❌ | ❌ |
| **Transactions (ACID)** | ✅ | ❌ | ❌ | ❌ |
| **Existing infra** | ✅ (if using Postgres) | ❌ New service | ❌ New service | ❌ New service |
| **Free** | ✅ Open source | ❌ (free tier limited) | ✅ Open source | ✅ Open source |
| **Scale (100M+ vectors)** | ⚠️ Needs tuning | ✅ Built for it | ❌ | ✅ |
| **Search speed at scale** | Good | Great | Slow | Great |
| **Managed hosting** | ✅ (Supabase, RDS, etc.) | ✅ Only | ❌ | ✅ |
| **Ease of use** | Medium (need SQL) | Easy | Easiest | Easy |
| **Backup/Recovery** | ✅ (pg_dump) | ✅ (managed) | ❌ Manual | ✅ |
| **Best for** | Apps already using Postgres | Fully managed, zero ops | Learning/prototyping | High perf self-hosted |

---

## Quick Reference: pgvector Cheat Sheet

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create table with vector column
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding vector(384)
);

-- Insert vector
INSERT INTO items (content, embedding) VALUES ('hello', '[0.1, 0.2, ...]');

-- Cosine similarity search (most common for text)
SELECT * FROM items ORDER BY embedding <=> '[0.1, 0.2, ...]' LIMIT 5;

-- Euclidean distance search
SELECT * FROM items ORDER BY embedding <-> '[0.1, 0.2, ...]' LIMIT 5;

-- Inner product search
SELECT * FROM items ORDER BY embedding <#> '[0.1, 0.2, ...]' LIMIT 5;

-- Get actual similarity score (1 - cosine_distance)
SELECT content, 1 - (embedding <=> '[0.1, ...]'::vector) AS similarity FROM items ORDER BY embedding <=> '[0.1, ...]'::vector LIMIT 5;

-- Create HNSW index (recommended)
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);

-- Create IVFFlat index (lighter on memory)
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- Tune search accuracy
SET hnsw.ef_search = 100;
SET ivfflat.probes = 10;

-- Combined vector + SQL query (pgvector superpower!)
SELECT * FROM items
WHERE category = 'tech' AND created_at > '2025-01-01'
ORDER BY embedding <=> '[0.1, ...]'::vector
LIMIT 5;

-- Check index usage
EXPLAIN ANALYZE SELECT * FROM items ORDER BY embedding <=> '[0.1, ...]'::vector LIMIT 5;
```

---

## Summary

```
pgvector = PostgreSQL + Vector Search in ONE database

Why pgvector?
├── You already use PostgreSQL → Just add an extension
├── SQL + Vector search → Combined in one query
├── JOINs → Link vector results with your regular tables
├── ACID → Transactions, consistency, reliability
├── Free → Open source, no vendor lock-in
└── Cloud-ready → Supabase, AWS RDS, Google Cloud, Azure

How it works:
1. Enable extension: CREATE EXTENSION vector;
2. Add vector column: embedding vector(384)
3. Insert embeddings: INSERT INTO ... VALUES (..., '[0.1, 0.2, ...]')
4. Search: SELECT * FROM ... ORDER BY embedding <=> query_vector LIMIT 5
5. Add index: CREATE INDEX ... USING hnsw (embedding vector_cosine_ops)
6. Combine with SQL: WHERE category = 'x' ORDER BY embedding <=> ...

When to use pgvector:
✅ Already using PostgreSQL
✅ Need SQL JOINs + vector search
✅ Dataset < 10M vectors
✅ Want one database, not two
✅ Need ACID transactions

When NOT to use pgvector:
❌ 100M+ vectors (consider Pinecone/Milvus)
❌ Need absolute fastest search speed (consider Qdrant/FAISS)
❌ Don't use PostgreSQL and don't want to start
```
