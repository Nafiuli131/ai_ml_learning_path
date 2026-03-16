# RAG (Retrieval-Augmented Generation) - Complete Beginner's Guide

## What is RAG?

RAG stands for **Retrieval-Augmented Generation**. Let's break that down:

- **Retrieval** = Finding/fetching relevant information from your own data
- **Augmented** = Adding that information to the LLM's input
- **Generation** = LLM generates an answer using that extra context

### The Problem RAG Solves

LLMs like ChatGPT are trained on public internet data up to a cutoff date. They **don't know** about:
- Your company's internal documents
- Your personal notes
- Recent events after their training cutoff
- Private databases, PDFs, or knowledge bases

**Without RAG:**
```
User: "What is our company's refund policy?"
LLM:  "I don't have access to your company's policies." (useless answer)
```

**With RAG:**
```
User: "What is our company's refund policy?"
RAG System: *searches your company docs* -> finds refund_policy.pdf
LLM + Context: "According to your policy, refunds are available within 30 days..." (accurate answer)
```

### RAG = Give the LLM a "cheat sheet" before it answers

Think of it like an open-book exam:
- Without RAG = LLM answers from memory only (can hallucinate)
- With RAG = LLM gets relevant pages from the book, then answers (more accurate)

---

## How RAG Works - The Big Picture

```
┌─────────────────────────────────────────────────────────┐
│                    RAG PIPELINE                         │
│                                                         │
│  STEP 1: INGESTION (One-time setup)                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────────────┐   │
│  │ Documents │───>│ Chunking │───>│ Embedding Model  │   │
│  │ (PDF,txt) │    │ (split)  │    │ (text -> vector) │   │
│  └──────────┘    └──────────┘    └────────┬─────────┘   │
│                                           │             │
│                                  ┌────────▼─────────┐   │
│                                  │  Vector Database  │   │
│                                  │  (store vectors)  │   │
│                                  └────────┬─────────┘   │
│                                           │             │
│  STEP 2: RETRIEVAL (Every query)          │             │
│  ┌──────────┐    ┌──────────┐    ┌────────▼─────────┐   │
│  │ User     │───>│ Embed    │───>│ Semantic Search   │   │
│  │ Question │    │ Question │    │ (find similar)    │   │
│  └──────────┘    └──────────┘    └────────┬─────────┘   │
│                                           │             │
│  STEP 3: GENERATION                       │             │
│  ┌──────────────────────────────┐         │             │
│  │ LLM receives:                │<────────┘             │
│  │  - User's question           │                       │
│  │  - Retrieved relevant chunks │                       │
│  │  - System prompt             │                       │
│  │                              │                       │
│  │ LLM generates final answer   │                       │
│  └──────────────────────────────┘                       │
└─────────────────────────────────────────────────────────┘
```

---

## Wait — How Does the LLM Know to Use RAG?

This is a very important question beginners have. The answer is:

### The LLM Does NOT Decide. YOUR CODE Decides.

The LLM is **dumb by itself**. It doesn't know your PDFs exist. It doesn't go looking for documents. **You (the developer) build a system around the LLM** that fetches the right data and feeds it in.

Think of it like this:

```
WITHOUT RAG (normal chatbot):
┌──────────┐                    ┌─────────┐
│   User   │───"What is our ──>│   LLM   │──> "I don't know your policy"
│          │   refund policy?"  │         │
└──────────┘                    └─────────┘
The LLM only sees the question. That's it. It has no access to your documents.


WITH RAG (your code does the work):
┌──────────┐     ┌───────────────────────────────────┐     ┌─────────┐
│   User   │──>  │  YOUR RAG CODE (middleware):      │──>  │   LLM   │
│          │     │                                    │     │         │
│ "What is │     │  1. Take user's question           │     │ Receives│
│  our     │     │  2. Search vector DB for matches   │     │ question│
│  refund  │     │  3. Find: "refund within 30 days"  │     │    +    │
│  policy?"│     │  4. Stuff it into the prompt       │     │ context │
│          │     │  5. Send BOTH to LLM               │     │    ↓    │
└──────────┘     └───────────────────────────────────┘     │ "Refund │
                                                           │  within │
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^           │ 30 days"│
                 THIS IS YOUR CODE, NOT THE LLM            └─────────┘
```

### The LLM Just Sees a Really Good Prompt

When you use RAG, the LLM doesn't "go to the PDF." Instead, **your code** searches the vector database, finds relevant chunks, and **pastes them into the prompt** before sending it to the LLM.

Here's what the LLM actually receives:

```
WITHOUT RAG — what the LLM sees:
┌─────────────────────────────────────────────┐
│ User: What is our refund policy?            │
└─────────────────────────────────────────────┘
LLM thinks: "I have no idea what this company's policy is..." → bad answer

WITH RAG — what the LLM sees:
┌─────────────────────────────────────────────┐
│ System: Answer using ONLY the context below.│
│                                             │
│ Context:                                    │
│ - "Refund policy: customers may return      │
│    items within 30 days for a full refund.  │
│    Items must be in original packaging."    │
│                                             │
│ User: What is our refund policy?            │
└─────────────────────────────────────────────┘
LLM thinks: "The context says 30 days, original packaging..." → great answer!
```

**The LLM doesn't know RAG exists.** It just sees a longer prompt with useful context. Your code did all the work behind the scenes.

### So Who Decides WHEN to Use RAG?

There are 3 common approaches:

#### Approach 1: Always Use RAG (Simplest — Most Common)

Every user question goes through the vector search, no matter what.

```python
# EVERY question triggers a search
def answer(user_question):
    chunks = vector_db.search(user_question, top_k=3)   # always search
    context = "\n".join(chunks)
    prompt = f"Context: {context}\n\nQuestion: {user_question}"
    return llm.generate(prompt)
```

**Pros:** Simple, never misses relevant context
**Cons:** Wastes a search call on questions like "hi" or "thanks"

#### Approach 2: Router / Classifier (Medium Complexity)

Use a small LLM or classifier to decide: "Does this question need RAG or not?"

```python
def answer(user_question):
    # Step 1: Ask a small/cheap LLM to classify
    needs_rag = classifier.predict(
        f"Does this question require searching documents? "
        f"Question: '{user_question}' "
        f"Answer YES or NO."
    )

    if needs_rag == "YES":
        chunks = vector_db.search(user_question, top_k=3)
        context = "\n".join(chunks)
        prompt = f"Context: {context}\n\nQuestion: {user_question}"
    else:
        prompt = user_question   # no context needed

    return llm.generate(prompt)
```

```
"What is our refund policy?"  → YES → search docs → answer with context
"Hi, how are you?"            → NO  → just chat normally
"Summarize yesterday's report"→ YES → search docs → answer with context
"What is 2 + 2?"              → NO  → just answer directly
```

#### Approach 3: Agentic RAG (Advanced — Tool Use)

Give the LLM a **tool** it can choose to call. The LLM itself decides when to search.

```python
tools = [
    {
        "name": "search_company_docs",
        "description": "Search internal company documents for information",
        "parameters": {"query": "the search query"}
    }
]

# LLM decides: "I need to search for refund policy" → calls the tool
# LLM decides: "2+2 is just math, no tool needed" → answers directly
response = llm.chat(
    messages=[{"role": "user", "content": user_question}],
    tools=tools
)
```

This is how ChatGPT with "Browse" or "Code Interpreter" works — the LLM decides which tool to use.

### Summary: Who Does What?

```
┌────────────────────────────────────────────────────────────┐
│  Component        │  Responsibility                        │
├────────────────────────────────────────────────────────────┤
│  YOUR CODE        │  Takes user question                   │
│  (RAG pipeline)   │  Searches vector DB                    │
│                   │  Finds relevant chunks                 │
│                   │  Builds the final prompt               │
│                   │  Sends everything to LLM               │
├────────────────────────────────────────────────────────────┤
│  VECTOR DATABASE  │  Stores all chunk embeddings           │
│  (ChromaDB etc.)  │  Returns similar chunks when searched  │
├────────────────────────────────────────────────────────────┤
│  LLM              │  Receives the ready-made prompt        │
│  (GPT/Claude)     │  Reads the context your code provided  │
│                   │  Generates an answer based on context   │
│                   │  Has NO idea RAG exists (in basic RAG) │
└────────────────────────────────────────────────────────────┘
```

**Key takeaway: The LLM is the "brain" but YOUR CODE is the "hands and eyes." Your code finds the documents, your code builds the prompt, and the LLM just reads what it's given.**

---

## What If the Answer is NOT in the PDF / Documents?

This is a very real problem. Your vector database will **ALWAYS return something** — even if nothing is actually relevant. It just returns the "least bad" match.

### The Problem: Vector Search Always Returns Results

```
Your documents are about: Company policies, HR rules, product info

User asks: "What is the capital of France?"

Vector DB: "Hmm... nothing matches well, but here's the closest chunk I have..."
Returns:   "Our company headquarters is in San Francisco." (similarity: 0.25 — LOW)

LLM receives this context and might say:
❌ "Based on the documents, the capital is San Francisco" (WRONG — hallucination!)
```

The vector DB doesn't say "I found nothing." It always returns the top-K results, even if they are garbage.

### Solution 1: Similarity Score Threshold (Most Common)

Check the similarity score. If it's too low, **don't use the context**.

```python
def answer_with_rag(user_question):
    results = collection.query(
        query_texts=[user_question],
        n_results=3,
        include=["documents", "distances"]  # get similarity scores too
    )

    chunks = results['documents'][0]
    scores = results['distances'][0]

    THRESHOLD = 0.5  # only use chunks with similarity > 0.5

    # Filter: keep only relevant chunks
    relevant_chunks = [
        chunk for chunk, score in zip(chunks, scores)
        if score >= THRESHOLD
    ]

    if relevant_chunks:
        # Good matches found — use RAG
        context = "\n".join(relevant_chunks)
        prompt = f"Context:\n{context}\n\nQuestion: {user_question}"
    else:
        # No good matches — tell the LLM there's no context
        prompt = (
            f"The user asked: {user_question}\n\n"
            f"No relevant information was found in our documents. "
            f"Politely tell the user you don't have that information."
        )

    return llm.generate(prompt)
```

```
"What is our refund policy?"   → score 0.92 → ✅ USE context → accurate answer
"Capital of France?"           → score 0.18 → ❌ SKIP context → "Sorry, I don't have that info"
```

### Solution 2: System Prompt Instructions (Simple but Less Reliable)

Tell the LLM in the system prompt: "If the context doesn't help, say you don't know."

```python
system_prompt = """You are a helpful assistant that answers questions using ONLY
the provided context.

IMPORTANT RULES:
- If the context does NOT contain the answer, say: "I don't have that information
  in my knowledge base."
- Do NOT make up answers.
- Do NOT use your general knowledge — ONLY use the context provided.
- If you're unsure, say you don't know.
"""

prompt = f"""Context:
{context}

Question: {user_question}

Answer (remember: ONLY use the context above, say "I don't know" if the context
doesn't contain the answer):"""
```

**Pros:** Very easy to implement
**Cons:** LLMs don't always follow instructions — they may still hallucinate

### Solution 3: Combine Both (Recommended for Production)

Use threshold filtering **AND** system prompt instructions together.

```python
def answer_with_rag(user_question):
    results = collection.query(query_texts=[user_question], n_results=3)
    chunks = results['documents'][0]
    scores = results['distances'][0]

    THRESHOLD = 0.5
    relevant_chunks = [c for c, s in zip(chunks, scores) if s >= THRESHOLD]

    system_prompt = (
        "Answer using ONLY the provided context. "
        "If the context doesn't contain the answer, say 'I don't have that information.'"
    )

    if relevant_chunks:
        context = "\n".join(relevant_chunks)
        user_prompt = f"Context:\n{context}\n\nQuestion: {user_question}"
    else:
        user_prompt = (
            f"No relevant context found.\n\n"
            f"Question: {user_question}\n\n"
            f"Respond that this question is outside your knowledge base."
        )

    return llm.generate(system_prompt=system_prompt, user_prompt=user_prompt)
```

### Solution 4: Fallback to General LLM (Optional)

If RAG has no answer, let the LLM answer from its own knowledge — but **tell the user** it's not from the documents.

```python
if relevant_chunks:
    answer = llm.generate(f"Context: {context}\nQuestion: {user_question}")
    source = "📄 From your documents"
else:
    answer = llm.generate(f"Answer from general knowledge: {user_question}")
    source = "⚠️ From general AI knowledge (not from your documents)"

print(f"{source}\n{answer}")
```

```
User: "What is our refund policy?"
📄 From your documents
→ "Refunds are available within 30 days of purchase."

User: "What is the capital of France?"
⚠️ From general AI knowledge (not from your documents)
→ "The capital of France is Paris."
```

### What Happens in Each Scenario — Quick Reference

| Scenario | What Happens | Best Approach |
|----------|-------------|---------------|
| Question matches docs well (score > 0.7) | RAG works perfectly | Use the context, generate answer |
| Question partially matches (score 0.4-0.7) | RAG might work, risky | Use context cautiously + strong system prompt |
| Question doesn't match at all (score < 0.4) | RAG returns garbage | Skip context, say "I don't know" or fallback |
| Question is about something docs don't cover | Vector DB returns unrelated chunks | Threshold filter catches this |
| User asks a general question ("What is AI?") | Might match too many chunks loosely | Router decides: skip RAG, answer directly |

### The Golden Rule

```
RAG is NOT about answering every question.
RAG is about answering questions FROM YOUR DATA accurately.

For everything else → say "I don't know" or fallback gracefully.
```

---

## STEP 1: Document Chunking Strategy

### What is Chunking?

Chunking = **Splitting large documents into smaller pieces (chunks)**

### Why Do We Need Chunking?

1. **LLMs have token limits** - You can't feed a 500-page PDF into an LLM at once
2. **Precision** - Smaller chunks = more precise retrieval (you find exactly the relevant paragraph, not the whole book)
3. **Cost** - Sending less text to the LLM = fewer tokens = lower API cost
4. **Speed** - Less text = faster processing

### Chunking Strategies (From Simple to Advanced)

---

### 1. Fixed-Size Chunking (Simplest)

Split text into chunks of a fixed number of characters or tokens.

```
Document: "The cat sat on the mat. The dog ran in the park. The bird flew over the tree."

Chunk size: 30 characters
Chunk 1: "The cat sat on the mat. The d"   <-- cuts mid-word!
Chunk 2: "og ran in the park. The bird "
Chunk 3: "flew over the tree."
```

**Problem:** Cuts words and sentences in the middle!

**Fix: Add Overlap**

```
Chunk size: 30 characters, Overlap: 10 characters

Chunk 1: "The cat sat on the mat. The d"
Chunk 2: " the mat. The dog ran in the "    <-- overlaps with chunk 1
Chunk 3: "n in the park. The bird flew "    <-- overlaps with chunk 2
Chunk 4: "bird flew over the tree."
```

Overlap ensures no information is lost at chunk boundaries.

```python
# Simple fixed-size chunking with overlap
def fixed_size_chunk(text, chunk_size=500, overlap=50):
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start = end - overlap  # move back by overlap amount
    return chunks
```

**Pros:** Simple, fast, predictable chunk sizes
**Cons:** Ignores document structure, can break mid-sentence

---

### 2. Sentence-Based Chunking

Split on sentence boundaries instead of character count.

```python
import nltk
nltk.download('punkt')

def sentence_chunk(text, max_sentences=5):
    sentences = nltk.sent_tokenize(text)
    chunks = []
    for i in range(0, len(sentences), max_sentences):
        chunk = " ".join(sentences[i:i + max_sentences])
        chunks.append(chunk)
    return chunks
```

**Example:**
```
Document: "Machine learning is a subset of AI. It uses data to learn patterns.
           Deep learning uses neural networks. It requires lots of data.
           NLP processes human language. It powers chatbots and translators."

Chunk 1 (2 sentences): "Machine learning is a subset of AI. It uses data to learn patterns."
Chunk 2 (2 sentences): "Deep learning uses neural networks. It requires lots of data."
Chunk 3 (2 sentences): "NLP processes human language. It powers chatbots and translators."
```

**Pros:** Never breaks mid-sentence, more meaningful chunks
**Cons:** Chunk sizes can vary a lot

---

### 3. Recursive Character Text Splitting (Most Popular - Used by LangChain)

This is the **most commonly used** strategy. It tries to split by the largest meaningful unit first, then falls back to smaller splits.

**Split hierarchy:**
1. First try to split by `\n\n` (paragraphs)
2. If still too big, split by `\n` (lines)
3. If still too big, split by `. ` (sentences)
4. If still too big, split by ` ` (words)
5. Last resort: split by character

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,       # max characters per chunk
    chunk_overlap=200,     # overlap between chunks
    separators=["\n\n", "\n", ". ", " ", ""]  # split hierarchy
)

chunks = splitter.split_text(your_document_text)
```

**Example:**
```
Document:
"# Introduction

Machine learning is a field of AI.
It helps computers learn from data.

# Deep Learning

Neural networks are the backbone of deep learning.
They consist of layers of neurons."

Split by "\n\n" first:
Chunk 1: "# Introduction\nMachine learning is a field of AI.\nIt helps computers learn from data."
Chunk 2: "# Deep Learning\nNeural networks are the backbone of deep learning.\nThey consist of layers of neurons."
```

**Pros:** Respects document structure, configurable, good default
**Cons:** Still doesn't understand the actual meaning/topics

---

### 4. Markdown / HTML Header-Based Chunking

Split based on document structure (headers, sections).

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

headers_to_split_on = [
    ("#", "Header 1"),
    ("##", "Header 2"),
    ("###", "Header 3"),
]

splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
chunks = splitter.split_text(markdown_text)
```

**Example:**
```markdown
# Animals
## Dogs
Dogs are loyal pets. They love humans.
## Cats
Cats are independent. They love naps.
# Plants
## Trees
Trees provide oxygen.
```

**Result:**
```
Chunk 1: {content: "Dogs are loyal pets. They love humans.", metadata: {Header 1: "Animals", Header 2: "Dogs"}}
Chunk 2: {content: "Cats are independent. They love naps.", metadata: {Header 1: "Animals", Header 2: "Cats"}}
Chunk 3: {content: "Trees provide oxygen.", metadata: {Header 1: "Plants", Header 2: "Trees"}}
```

**Pros:** Preserves document hierarchy, great for structured docs
**Cons:** Only works with structured documents (Markdown, HTML)

---

### 5. Semantic Chunking (Most Advanced)

Split based on **meaning changes** in the text, not structure. Uses embeddings to detect when the topic shifts.

```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
semantic_splitter = SemanticChunker(embeddings, breakpoint_threshold_type="percentile")

chunks = semantic_splitter.split_text(your_text)
```

**How it works:**
1. Split text into sentences
2. Generate embeddings for each sentence
3. Compare similarity between consecutive sentences
4. When similarity drops significantly = topic change = chunk boundary

```
Sentence 1: "Python is a programming language."        ─┐
Sentence 2: "It was created by Guido van Rossum."       │ Similar → Same chunk
Sentence 3: "Python is used for web development."       ─┘
                                                        ← BIG meaning shift detected
Sentence 4: "The weather in Tokyo is hot in summer."   ─┐
Sentence 5: "Japan has four distinct seasons."          ─┘ Similar → Same chunk
```

**Pros:** Chunks are topically coherent, best retrieval quality
**Cons:** Slower (requires embedding each sentence), more complex, costs money

---

### Chunking Strategy Comparison Table

| Strategy | Complexity | Quality | Speed | Best For |
|----------|-----------|---------|-------|----------|
| Fixed-Size | Very Low | Low | Very Fast | Quick prototyping |
| Sentence-Based | Low | Medium | Fast | Simple documents |
| Recursive (LangChain) | Medium | Good | Fast | General purpose (recommended default) |
| Header-Based | Medium | Good | Fast | Structured docs (Markdown, HTML) |
| Semantic | High | Best | Slow | When retrieval quality is critical |

### Chunk Size Guidelines

| Chunk Size | Use Case |
|-----------|----------|
| 100-200 tokens | Short, precise answers (FAQ, definitions) |
| 200-500 tokens | General purpose (most common, recommended) |
| 500-1000 tokens | When context/narrative matters (stories, reports) |
| 1000+ tokens | Long-form analysis, legal documents |

**Rule of Thumb:** Start with **500 tokens, 50-100 token overlap** using Recursive splitting. Adjust based on results.

---

## STEP 2: Embedding Models (Converting Text to Vectors)

### First, What is a Vector?

Before we talk about embeddings, let's understand **vectors** — because this is the foundation of everything in RAG.

#### A Vector is Just a List of Numbers

That's it. Really. A vector is nothing more than an **ordered list of numbers**.

```
A single number:          5
A vector (list of numbers): [5, 3, 8]
```

#### Real Life Examples of Vectors

**Example 1: Your Location on a Map**

Your location can be described by 2 numbers: latitude and longitude.

```
Your location = [23.8103, 90.4125]    ← This is a vector! (2 numbers = 2 dimensions)
                  ↑          ↑
               latitude   longitude
```

**Example 2: A Color (RGB)**

Every color on your screen is described by 3 numbers: Red, Green, Blue.

```
Red    = [255, 0, 0]       ← vector with 3 dimensions
Green  = [0, 255, 0]       ← vector with 3 dimensions
Yellow = [255, 255, 0]     ← vector with 3 dimensions
White  = [255, 255, 255]   ← vector with 3 dimensions
```

**Example 3: A Student's Profile**

```
Student = [age, height_cm, weight_kg, gpa]
Rahim   = [20,  170,       65,        3.5]    ← vector with 4 dimensions
Karim   = [21,  175,       70,        3.8]    ← vector with 4 dimensions
```

#### Why Do We Use Vectors?

Because vectors let us **measure how similar two things are** using math!

```
Red    = [255, 0, 0]
Orange = [255, 165, 0]
Blue   = [0, 0, 255]

Question: Is Red more similar to Orange or Blue?

Distance(Red, Orange) = small   → SIMILAR (both have high Red value)
Distance(Red, Blue)   = large   → DIFFERENT
```

We can compare ANY two things — as long as they are represented as vectors (lists of numbers).

#### Vectors in RAG: Representing Meaning as Numbers

In RAG, we convert **text into vectors** where each number represents some aspect of meaning:

```
"I love cats" → [0.12, -0.45, 0.78, 0.33, ... 0.91]    (hundreds of numbers!)
```

Each number captures something about the meaning:
- Maybe number 1 represents "how much about animals"
- Maybe number 2 represents "how much about emotions"
- Maybe number 3 represents "how positive the sentiment is"
- ... and so on for hundreds of dimensions

The key insight: **texts with similar meanings produce similar vectors** (similar lists of numbers).

```
"I love cats"    → [0.12, -0.45, 0.78, ...]
"I adore kittens"→ [0.11, -0.43, 0.80, ...]   ← Numbers are very close! (similar meaning)
"Stock market"   → [-0.67, 0.22, -0.15, ...]   ← Numbers are very different! (different meaning)
```

#### Quick Summary

| Concept | What It Is | Example |
|---------|-----------|---------|
| **Scalar** | A single number | `5` |
| **Vector** | A list of numbers | `[5, 3, 8]` |
| **2D Vector** | A point on a flat surface | `[x, y]` = map location |
| **3D Vector** | A point in space | `[x, y, z]` = RGB color |
| **Embedding Vector** | A point in "meaning space" | `[0.12, -0.45, ...]` = text meaning |
| **Dimensions** | How many numbers in the vector | 384, 768, 1536 are common in AI |

**Bottom line: A vector is just a list of numbers. In RAG, we turn text into these lists so that a computer can compare meanings mathematically.**

---

### What is an Embedding?

An embedding is a way to convert text into a **list of numbers (vector)** that captures the **meaning** of that text.

```
"I love dogs"  →  [0.12, -0.45, 0.78, 0.33, ..., 0.91]   (1536 numbers)
"I adore puppies" → [0.11, -0.43, 0.80, 0.31, ..., 0.89]  (1536 numbers)  ← very similar!
"The stock market crashed" → [-0.67, 0.22, -0.15, 0.88, ..., -0.34]        ← very different!
```

### Why Do We Need Embeddings?

Computers can't understand text directly. But they **CAN** compare numbers. By converting text to numbers (vectors), we can:
- **Measure similarity** between texts
- **Search** for relevant content
- **Cluster** similar documents together

### How Embeddings Work (Intuition)

Think of each number in the vector as a "meaning dimension":

```
Dimension 1: How much about animals? (high for "dog", low for "stock market")
Dimension 2: How much about emotions? (high for "love", low for "the")
Dimension 3: How much about finance? (low for "dog", high for "stock market")
... and so on for hundreds/thousands of dimensions
```

Real embeddings don't have such clean interpretations, but the principle is the same - similar meanings produce similar numbers.

### Visual Example: 2D Embedding Space

Imagine we simplify to just 2 dimensions:

```
                  ▲ Animal-ness
                  │
            dog ● │    ● cat
                  │
          puppy ● │         ● kitten
                  │
    ──────────────┼──────────────────► Finance-ness
                  │
                  │         ● stock
                  │
                  │    ● bond     ● market
                  │
```

In reality, embeddings have **hundreds to thousands** of dimensions, capturing incredibly subtle meaning differences.

### Popular Embedding Models

#### 1. OpenAI Embeddings

```python
from openai import OpenAI

client = OpenAI(api_key="your-key")

response = client.embeddings.create(
    model="text-embedding-3-small",   # or "text-embedding-3-large"
    input="I love programming in Python"
)

vector = response.data[0].embedding
print(f"Vector length: {len(vector)}")     # 1536 dimensions
print(f"First 5 values: {vector[:5]}")     # [0.012, -0.034, 0.078, ...]
```

| Model | Dimensions | Cost | Quality |
|-------|-----------|------|---------|
| text-embedding-3-small | 1536 | $0.02/1M tokens | Good |
| text-embedding-3-large | 3072 | $0.13/1M tokens | Best |
| text-embedding-ada-002 | 1536 | $0.10/1M tokens | Good (older) |

#### 2. Open-Source: Sentence Transformers (FREE, runs locally)

```python
from sentence_transformers import SentenceTransformer

# Download model (runs on your machine, no API key needed!)
model = SentenceTransformer('all-MiniLM-L6-v2')

sentences = [
    "I love dogs",
    "I adore puppies",
    "The stock market crashed"
]

embeddings = model.encode(sentences)

print(f"Shape: {embeddings.shape}")  # (3, 384) → 3 sentences, 384 dimensions each
```

| Model | Dimensions | Speed | Quality |
|-------|-----------|-------|---------|
| all-MiniLM-L6-v2 | 384 | Very Fast | Good |
| all-mpnet-base-v2 | 768 | Medium | Better |
| bge-large-en-v1.5 | 1024 | Slower | Great |
| e5-large-v2 | 1024 | Slower | Great |

#### 3. Cohere Embeddings

```python
import cohere

co = cohere.Client("your-api-key")

response = co.embed(
    texts=["I love dogs", "The stock market crashed"],
    model="embed-english-v3.0",
    input_type="search_document"   # or "search_query" for queries
)

embeddings = response.embeddings  # list of vectors
```

#### 4. Google Vertex AI Embeddings

```python
from vertexai.language_models import TextEmbeddingModel

model = TextEmbeddingModel.from_pretrained("textembedding-gecko@003")
embeddings = model.get_embeddings(["I love dogs"])
vector = embeddings[0].values  # 768 dimensions
```

### Embedding Model Comparison

| Model | Provider | Dimensions | Free? | Best For |
|-------|----------|-----------|-------|----------|
| text-embedding-3-small | OpenAI | 1536 | No | Production apps |
| text-embedding-3-large | OpenAI | 3072 | No | Highest quality |
| all-MiniLM-L6-v2 | HuggingFace | 384 | Yes | Learning/prototyping |
| all-mpnet-base-v2 | HuggingFace | 768 | Yes | Good free option |
| bge-large-en-v1.5 | BAAI | 1024 | Yes | Best free option |
| embed-english-v3.0 | Cohere | 1024 | No | Multilingual |

### How to Measure Similarity Between Vectors

The most common method is **Cosine Similarity**:

```
                    A · B
Cosine Similarity = ─────────
                    |A| × |B|

Result ranges from:
  -1 (completely opposite meaning)
   0 (no relationship)
  +1 (identical meaning)
```

```python
import numpy as np

def cosine_similarity(vec1, vec2):
    dot_product = np.dot(vec1, vec2)
    magnitude = np.linalg.norm(vec1) * np.linalg.norm(vec2)
    return dot_product / magnitude

# Example
embedding_dog = model.encode("I love dogs")
embedding_puppy = model.encode("I adore puppies")
embedding_stock = model.encode("The stock market crashed")

print(cosine_similarity(embedding_dog, embedding_puppy))   # ~0.92 (very similar!)
print(cosine_similarity(embedding_dog, embedding_stock))    # ~0.15 (not similar)
```

### Complete Embedding Pipeline

```python
from sentence_transformers import SentenceTransformer
import numpy as np

# 1. Load model
model = SentenceTransformer('all-MiniLM-L6-v2')

# 2. Your document chunks (from chunking step)
chunks = [
    "Python is a programming language created by Guido van Rossum.",
    "JavaScript is used for web development and runs in browsers.",
    "Machine learning uses algorithms to learn patterns from data.",
    "The Eiffel Tower is located in Paris, France.",
]

# 3. Convert all chunks to vectors
chunk_embeddings = model.encode(chunks)
print(f"Embedded {len(chunks)} chunks into vectors of size {chunk_embeddings.shape[1]}")

# 4. Convert a question to a vector
question = "What programming language did Guido create?"
question_embedding = model.encode(question)

# 5. Find most similar chunk
similarities = [
    np.dot(question_embedding, chunk_emb) /
    (np.linalg.norm(question_embedding) * np.linalg.norm(chunk_emb))
    for chunk_emb in chunk_embeddings
]

best_match_index = np.argmax(similarities)
print(f"Best match: {chunks[best_match_index]}")
print(f"Similarity: {similarities[best_match_index]:.4f}")

# Output:
# Best match: "Python is a programming language created by Guido van Rossum."
# Similarity: 0.7823
```

---

## STEP 3: Semantic Search (Finding Relevant Chunks)

### What is Semantic Search?

**Keyword Search (Traditional):**
```
Query: "How to fix a broken heart"
Finds: Documents containing exact words "fix", "broken", "heart"
Result: Might return plumbing repair manuals mentioning "broken" pipes ❌
```

**Semantic Search (Meaning-Based):**
```
Query: "How to fix a broken heart"
Finds: Documents about emotional healing, moving on after breakups
Result: Relevant emotional advice ✅
```

Semantic search understands **meaning**, not just keywords.

### How Semantic Search Works in RAG

```
Step 1: User asks a question
        "What causes diabetes?"

Step 2: Convert question to vector
        [0.23, -0.11, 0.45, ...] (embedding)

Step 3: Compare with ALL stored chunk vectors
        Chunk 1 vector: [0.21, -0.09, 0.44, ...] → similarity = 0.95 ← HIGH!
        Chunk 2 vector: [-0.56, 0.78, -0.12, ...] → similarity = 0.12
        Chunk 3 vector: [0.19, -0.15, 0.41, ...] → similarity = 0.88 ← HIGH!
        Chunk 4 vector: [0.67, 0.34, -0.89, ...] → similarity = 0.05

Step 4: Return top-K most similar chunks
        Top 2: Chunk 1 (0.95), Chunk 3 (0.88)
```

### Vector Databases (Where We Store Embeddings)

You need a special database to efficiently store and search through millions of vectors. Regular databases (MySQL, PostgreSQL) are not designed for this.

#### Popular Vector Databases

| Database | Type | Free Tier? | Best For |
|----------|------|-----------|----------|
| ChromaDB | In-memory/local | Yes (open source) | Learning, prototyping |
| FAISS | In-memory library | Yes (open source) | Fast local search |
| Pinecone | Cloud service | Yes (limited) | Production, managed |
| Weaviate | Self-hosted/Cloud | Yes (open source) | Production, flexible |
| Qdrant | Self-hosted/Cloud | Yes (open source) | Production, fast |
| pgvector | PostgreSQL extension | Yes (open source) | If you already use PostgreSQL |
| Milvus | Self-hosted/Cloud | Yes (open source) | Large-scale production |

---

### Using ChromaDB (Easiest to Start)

```bash
pip install chromadb
```

```python
import chromadb
from sentence_transformers import SentenceTransformer

# 1. Create a ChromaDB client (stores data locally)
client = chromadb.Client()

# 2. Create a collection (like a table in a database)
collection = client.create_collection(
    name="my_documents",
    metadata={"hnsw:space": "cosine"}  # use cosine similarity
)

# 3. Add your chunks
chunks = [
    "Python was created by Guido van Rossum in 1991.",
    "JavaScript was created by Brendan Eich in 1995.",
    "Machine learning helps computers learn from data without being explicitly programmed.",
    "The Eiffel Tower was built in 1889 for the World's Fair in Paris.",
    "Deep learning is a subset of machine learning using neural networks.",
]

# Add documents with IDs
collection.add(
    documents=chunks,
    ids=[f"chunk_{i}" for i in range(len(chunks))]
)

# 4. Search!
results = collection.query(
    query_texts=["Who created Python?"],
    n_results=2  # return top 2 matches
)

print(results['documents'])
# [['Python was created by Guido van Rossum in 1991.',
#   'JavaScript was created by Brendan Eich in 1995.']]
```

### Using FAISS (Facebook AI Similarity Search)

```bash
pip install faiss-cpu sentence-transformers
```

```python
import faiss
import numpy as np
from sentence_transformers import SentenceTransformer

# 1. Load embedding model
model = SentenceTransformer('all-MiniLM-L6-v2')

# 2. Your chunks
chunks = [
    "Python was created by Guido van Rossum in 1991.",
    "JavaScript was created by Brendan Eich in 1995.",
    "Machine learning helps computers learn from data.",
    "The Eiffel Tower is in Paris, France.",
    "Deep learning uses neural networks with many layers.",
]

# 3. Create embeddings
embeddings = model.encode(chunks)
dimension = embeddings.shape[1]  # 384 for MiniLM

# 4. Create FAISS index
index = faiss.IndexFlatL2(dimension)  # L2 (Euclidean) distance
index.add(embeddings.astype('float32'))

print(f"Total vectors in index: {index.ntotal}")  # 5

# 5. Search
query = "Who invented Python programming?"
query_vector = model.encode([query]).astype('float32')

k = 2  # top 2 results
distances, indices = index.search(query_vector, k)

print("Top results:")
for i, idx in enumerate(indices[0]):
    print(f"  {i+1}. {chunks[idx]} (distance: {distances[0][i]:.4f})")

# Output:
# Top results:
#   1. Python was created by Guido van Rossum in 1991. (distance: 0.4521)
#   2. JavaScript was created by Brendan Eich in 1995. (distance: 0.8234)
```

### Using Pinecone (Cloud-Based, Production Ready)

```bash
pip install pinecone-client
```

```python
from pinecone import Pinecone, ServerlessSpec

# 1. Initialize
pc = Pinecone(api_key="your-pinecone-api-key")

# 2. Create index
pc.create_index(
    name="my-rag-index",
    dimension=384,  # must match your embedding model's output size
    metric="cosine",
    spec=ServerlessSpec(cloud="aws", region="us-east-1")
)

index = pc.Index("my-rag-index")

# 3. Upsert vectors
vectors = [
    {"id": "chunk_0", "values": embedding_0.tolist(), "metadata": {"text": chunks[0]}},
    {"id": "chunk_1", "values": embedding_1.tolist(), "metadata": {"text": chunks[1]}},
    # ... more vectors
]
index.upsert(vectors=vectors)

# 4. Query
results = index.query(
    vector=query_embedding.tolist(),
    top_k=3,
    include_metadata=True
)

for match in results['matches']:
    print(f"Score: {match['score']:.4f} | Text: {match['metadata']['text']}")
```

---

### Improving Semantic Search Quality

#### 1. Hybrid Search (Keyword + Semantic)

Combine traditional keyword search with semantic search for better results.

```
Query: "Python 3.12 release date"

Keyword Search: Finds docs containing "Python", "3.12", "release" (good for exact terms)
Semantic Search: Finds docs about Python version releases (good for meaning)
Hybrid: Combines both scores → Best results!
```

```python
# Hybrid search concept (simplified)
def hybrid_search(query, chunks, keyword_weight=0.3, semantic_weight=0.7):
    keyword_scores = bm25_search(query, chunks)      # traditional keyword scoring
    semantic_scores = semantic_search(query, chunks)  # embedding-based scoring

    # Combine scores
    final_scores = (keyword_weight * keyword_scores) + (semantic_weight * semantic_scores)

    # Return top results
    top_indices = np.argsort(final_scores)[::-1][:k]
    return [chunks[i] for i in top_indices]
```

#### 2. Re-Ranking

After initial retrieval, use a more powerful model to re-rank results.

```
Step 1: Retrieve top 20 chunks (fast, rough ranking)
Step 2: Re-rank those 20 with a cross-encoder model (slow, precise ranking)
Step 3: Return top 5 after re-ranking
```

```python
from sentence_transformers import CrossEncoder

# Cross-encoder is more accurate but slower than bi-encoder
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')

# Initial retrieval gives us candidate chunks
candidates = ["chunk 1 text", "chunk 2 text", "chunk 3 text", ...]

# Re-rank
query = "What is machine learning?"
pairs = [[query, chunk] for chunk in candidates]
scores = reranker.predict(pairs)

# Sort by re-ranker score
ranked_results = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)
```

#### 3. Metadata Filtering

Filter search results using metadata before or after semantic search.

```python
# In ChromaDB
results = collection.query(
    query_texts=["machine learning basics"],
    n_results=5,
    where={
        "category": "education",     # only search educational docs
        "year": {"$gte": 2023}       # only docs from 2023 or later
    }
)
```

#### 4. Query Transformation

Improve the query before searching.

```python
# HyDE: Hypothetical Document Embeddings
# Instead of embedding the question, generate a hypothetical answer and embed THAT

question = "What causes diabetes?"

# Ask LLM to generate a hypothetical answer
hypothetical_answer = llm.generate(
    f"Write a short paragraph answering: {question}"
)
# "Diabetes is caused by the body's inability to produce or use insulin properly..."

# Embed the hypothetical answer instead of the question
search_vector = model.encode(hypothetical_answer)
# This often matches better because the answer looks like the stored documents!
```

---

## STEP 4: Putting It All Together - Complete RAG Pipeline

### Full Working Example with LangChain

```bash
pip install langchain langchain-openai chromadb
```

```python
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import TextLoader, PyPDFLoader
from langchain.chains import RetrievalQA
from langchain.prompts import PromptTemplate

# ============================================
# STEP 1: Load Documents
# ============================================
# From a text file
loader = TextLoader("my_document.txt")
documents = loader.load()

# Or from a PDF
# loader = PyPDFLoader("my_document.pdf")
# documents = loader.load()

print(f"Loaded {len(documents)} document(s)")

# ============================================
# STEP 2: Chunk the Documents
# ============================================
splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " ", ""]
)

chunks = splitter.split_documents(documents)
print(f"Split into {len(chunks)} chunks")

# ============================================
# STEP 3: Create Embeddings & Store in Vector DB
# ============================================
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./my_vectordb"  # saves to disk
)

print(f"Stored {vectorstore._collection.count()} vectors")

# ============================================
# STEP 4: Create Retriever
# ============================================
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}  # return top 3 chunks
)

# ============================================
# STEP 5: Create RAG Chain
# ============================================
llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt_template = PromptTemplate(
    template="""Use the following context to answer the question.
If you don't know the answer based on the context, say "I don't know."

Context:
{context}

Question: {question}

Answer:""",
    input_variables=["context", "question"]
)

rag_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",  # stuff = put all chunks into one prompt
    retriever=retriever,
    chain_type_kwargs={"prompt": prompt_template},
    return_source_documents=True
)

# ============================================
# STEP 6: Ask Questions!
# ============================================
result = rag_chain.invoke({"query": "What is the main topic of the document?"})

print("Answer:", result['result'])
print("\nSources:")
for doc in result['source_documents']:
    print(f"  - {doc.page_content[:100]}...")
```

### Full Working Example WITHOUT LangChain (Pure Python)

```python
import openai
import chromadb
import os

# ============================================
# Setup
# ============================================
openai_client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
chroma_client = chromadb.PersistentClient(path="./my_vectordb")

# ============================================
# STEP 1: Prepare chunks (simplified - from a list of texts)
# ============================================
documents = [
    "Our company was founded in 2020 by Jane Smith.",
    "The refund policy allows returns within 30 days of purchase.",
    "Our main office is located in San Francisco, California.",
    "Employees get 20 days of paid vacation per year.",
    "The company revenue was $5 million in 2024.",
    "Our tech stack includes Python, React, and PostgreSQL.",
    "Customer support is available 24/7 via chat and email.",
]

# ============================================
# STEP 2: Store in ChromaDB (it handles embeddings automatically)
# ============================================
collection = chroma_client.get_or_create_collection(
    name="company_docs",
    metadata={"hnsw:space": "cosine"}
)

collection.add(
    documents=documents,
    ids=[f"doc_{i}" for i in range(len(documents))]
)

# ============================================
# STEP 3: Function to do RAG
# ============================================
def ask_with_rag(question, n_results=3):
    # 3a. Search for relevant chunks
    results = collection.query(
        query_texts=[question],
        n_results=n_results
    )

    relevant_chunks = results['documents'][0]

    # 3b. Build prompt with context
    context = "\n".join([f"- {chunk}" for chunk in relevant_chunks])

    prompt = f"""Based on the following context, answer the question.
If the answer is not in the context, say "I don't have that information."

Context:
{context}

Question: {question}

Answer:"""

    # 3c. Send to LLM
    response = openai_client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": "You are a helpful assistant that answers based on provided context."},
            {"role": "user", "content": prompt}
        ],
        temperature=0
    )

    answer = response.choices[0].message.content

    return {
        "answer": answer,
        "sources": relevant_chunks
    }

# ============================================
# STEP 4: Use it!
# ============================================
result = ask_with_rag("What is the refund policy?")
print(f"Answer: {result['answer']}")
print(f"Sources: {result['sources']}")

# Output:
# Answer: The refund policy allows returns within 30 days of purchase.
# Sources: ['The refund policy allows returns within 30 days of purchase.',
#           'Customer support is available 24/7 via chat and email.',
#           'Our company was founded in 2020 by Jane Smith.']
```

---

## Advanced RAG Patterns

### 1. Multi-Query RAG

Generate multiple versions of the user's question to catch different relevant chunks.

```python
def multi_query_rag(question):
    # Generate alternative questions using LLM
    prompt = f"""Generate 3 different versions of this question to help find relevant information:
    Original: {question}

    Return only the 3 questions, one per line."""

    response = llm.generate(prompt)
    alternative_questions = response.strip().split("\n")

    # Search with each question
    all_chunks = set()
    for q in [question] + alternative_questions:
        results = collection.query(query_texts=[q], n_results=3)
        for chunk in results['documents'][0]:
            all_chunks.add(chunk)

    # Use all unique chunks as context
    return list(all_chunks)
```

### 2. Parent-Child Chunking

Store small chunks for precise retrieval, but return larger parent chunks for context.

```
Parent Chunk (full section):
"Machine learning is a field of AI. It uses algorithms to learn patterns
from data. Common types include supervised learning, unsupervised learning,
and reinforcement learning. Supervised learning uses labeled data..."

Child Chunks (for retrieval):
  Child 1: "Machine learning is a field of AI."
  Child 2: "It uses algorithms to learn patterns from data."
  Child 3: "Common types include supervised, unsupervised, and reinforcement learning."

Search matches Child 3 → Return the FULL parent chunk to the LLM
```

### 3. Self-Querying RAG

Let the LLM convert natural language to structured filters.

```
User: "Show me machine learning papers from 2024"

LLM extracts:
  - Semantic query: "machine learning papers"
  - Filters: {year: 2024, type: "paper"}

Search: semantic_search("machine learning papers", where={year: 2024, type: "paper"})
```

---

## Common RAG Mistakes & How to Fix Them

| Mistake | Problem | Fix |
|---------|---------|-----|
| Chunks too large | Retrieves irrelevant text along with relevant text | Use smaller chunks (200-500 tokens) |
| Chunks too small | Loses context, chunks are meaningless on their own | Use larger chunks or parent-child strategy |
| No overlap | Information at chunk boundaries is lost | Add 10-20% overlap between chunks |
| Wrong embedding model | Poor similarity matching | Use a model trained for your language/domain |
| Too few results (k too low) | Misses relevant information | Increase k to 5-10, use re-ranking |
| Too many results (k too high) | Fills context with noise | Decrease k, use re-ranking |
| No metadata | Can't filter by date, source, category | Always store metadata with chunks |
| Not evaluating | Don't know if RAG is working well | Use RAGAS or manual evaluation |

---

## RAG Evaluation

How do you know if your RAG system is working well?

### Key Metrics

| Metric | What It Measures | Good Score |
|--------|-----------------|------------|
| **Context Precision** | Are the retrieved chunks relevant? | > 0.8 |
| **Context Recall** | Did we find ALL relevant chunks? | > 0.7 |
| **Answer Faithfulness** | Is the answer based on the context (not hallucinated)? | > 0.9 |
| **Answer Relevancy** | Does the answer actually address the question? | > 0.8 |

### Using RAGAS for Evaluation

```bash
pip install ragas
```

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision, context_recall
from datasets import Dataset

# Prepare evaluation data
eval_data = {
    "question": ["What is our refund policy?"],
    "answer": ["Returns are allowed within 30 days."],
    "contexts": [["The refund policy allows returns within 30 days of purchase."]],
    "ground_truth": ["Customers can return items within 30 days."]
}

dataset = Dataset.from_dict(eval_data)

results = evaluate(
    dataset,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall]
)

print(results)
# {'faithfulness': 0.95, 'answer_relevancy': 0.92, 'context_precision': 1.0, 'context_recall': 0.88}
```

---

## Quick Reference: RAG Tech Stack Recommendations

### For Learning / Prototyping
| Component | Recommendation |
|-----------|---------------|
| Embedding | all-MiniLM-L6-v2 (free, local) |
| Vector DB | ChromaDB (simple, no setup) |
| Chunking | RecursiveCharacterTextSplitter |
| LLM | GPT-4 / Claude |
| Framework | LangChain or plain Python |

### For Production
| Component | Recommendation |
|-----------|---------------|
| Embedding | text-embedding-3-large (OpenAI) or bge-large-en-v1.5 |
| Vector DB | Pinecone / Qdrant / Weaviate |
| Chunking | Semantic chunking + parent-child |
| LLM | GPT-4 / Claude |
| Search | Hybrid (keyword + semantic) + re-ranking |
| Evaluation | RAGAS + manual review |
| Framework | LangChain / LlamaIndex |

---

## Summary: RAG in One Picture

```
YOUR DOCUMENTS          CHUNKING              EMBEDDING            VECTOR DB
┌──────────┐         ┌──────────┐         ┌──────────┐        ┌──────────────┐
│ PDF      │         │ Chunk 1  │────────>│ [0.1,..] │───────>│              │
│ TXT      │──SPLIT─>│ Chunk 2  │────────>│ [0.3,..] │───────>│  ChromaDB /  │
│ HTML     │         │ Chunk 3  │────────>│ [0.7,..] │───────>│  Pinecone /  │
│ Markdown │         │ ...      │────────>│ [0.2,..] │───────>│  FAISS       │
└──────────┘         └──────────┘         └──────────┘        └──────┬───────┘
                                                                     │
USER QUESTION                                                        │
┌──────────────┐     ┌──────────┐     ┌──────────────┐              │
│ "What is..." │────>│ [0.4,..] │────>│ Find similar │<─────────────┘
└──────────────┘     └──────────┘     │ vectors      │
                     (embed query)    └──────┬───────┘
                                             │
                                     ┌───────▼────────┐
                                     │ Top K chunks:  │
                                     │ - Chunk 2      │
                                     │ - Chunk 5      │
                                     │ - Chunk 1      │
                                     └───────┬────────┘
                                             │
                                     ┌───────▼────────┐
                                     │ LLM receives:  │
                                     │ Question +     │──────> FINAL ANSWER
                                     │ Relevant chunks│
                                     └────────────────┘
```

**RAG = Chunk your docs + Embed them + Store vectors + Search by meaning + Let LLM answer with context**
