# RAG (Retrieval-Augmented Generation) Architecture

> **In one sentence:** RAG connects an LLM to your own data — so it can answer questions using _your_ documents, not just what it was trained on.

---

## 🧠 The Core Idea

A standard LLM (like GPT-4 or Claude) has two big limitations:

|Problem|Why it matters|
|---|---|
|Knowledge cutoff|It doesn't know anything after its training date|
|No access to private data|It can't read your company docs, PDFs, databases|
|Hallucination|It sometimes makes up plausible-sounding but wrong answers|

**RAG solves all three** by adding a _retrieval step_ before generation — the model looks up relevant facts first, then answers using that retrieved context.

---

## 🗺️ The Big Picture (Two Phases)

RAG has two distinct phases that happen at different times:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PHASE 1: INDEXING           (done once, offline)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Raw Documents (PDFs, Docs, Websites...)
              ↓
        Split into chunks
              ↓
      Convert to vectors (embeddings)
              ↓
       Store in Vector Database ✅


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PHASE 2: RETRIEVAL + GENERATION   (every query)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  User asks a question
              ↓
     Convert question → vector
              ↓
     Search Vector DB for similar vectors
              ↓
     Retrieve Top-K matching chunks
              ↓
     LLM reads: question + retrieved chunks
              ↓
         Final Answer ✅
```

---

## 📥 Phase 1: Indexing (Offline)

This runs **before** any user interacts with the system. Think of it as building the library.

### Step 1 — Chunking

Long documents are split into smaller pieces so the retrieval system can find _precisely_ relevant sections.

```
Document: "React is a JavaScript library for building user interfaces.
           It was created by Facebook in 2013. Components are the
           building blocks of React applications..."

   ↓  Split into chunks  ↓

Chunk 1: "React is a JavaScript library for building user interfaces."
Chunk 2: "It was created by Facebook in 2013."
Chunk 3: "Components are the building blocks of React applications."
```

> **Why not use the whole document?**  
> A 50-page PDF has only ~3 relevant sentences for any given question. Chunking lets you fetch just those 3 sentences, not all 50 pages.

---

### Step 2 — Embedding (Text → Vector)

Each chunk is passed through an **embedding model**, which converts text into a list of numbers (a _vector_). This vector captures the _meaning_ of the text.

```
"React is a JavaScript library"  →  [0.23, 0.91, 0.11, 0.54, ...]
"It is used for building UIs"    →  [0.21, 0.89, 0.13, 0.51, ...]
"Python is great for ML"         →  [0.78, 0.12, 0.88, 0.03, ...]
```

Notice the React sentences produce _similar_ vectors (similar numbers), while the Python sentence produces a very different vector. This is what makes semantic search possible.

---

### Step 3 — Store in Vector Database

The vectors (along with the original text) are stored in a **vector database** (e.g., Pinecone, FAISS, Weaviate). The database is optimized for one specific operation: _"find the vectors most similar to this query vector."_

---

## 📤 Phase 2: Retrieval + Generation (Online)

This runs **every time** a user asks a question.

### Step 1 — Embed the Question

The user's question is converted to a vector using the _same_ embedding model used during indexing.

```
"What is React used for?"  →  [0.22, 0.90, 0.12, 0.53, ...]
```

### Step 2 — Similarity Search

The vector database compares this query vector against all stored vectors and returns the **Top-K most similar** chunks (typically K = 3 to 5).

```
Query vector: [0.22, 0.90, 0.12, 0.53, ...]

Most similar stored vectors:
  → "React is a JavaScript library"     (similarity: 0.97) ✅
  → "It is used for building UIs"       (similarity: 0.94) ✅
  → "React components are reusable"     (similarity: 0.91) ✅
  → "Python is great for ML"            (similarity: 0.21) ❌ (not retrieved)
```

> **Why cosine similarity and not SQL `LIKE '%React%'`?**  
> SQL keyword search would miss "What is a frontend library?" even though it means the same thing.  
> Vector similarity finds results by _meaning_, not exact words.

### Step 3 — LLM Generates the Answer

The retrieved chunks are inserted into the prompt alongside the user's question:

```
Prompt sent to LLM:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Context:
  [1] "React is a JavaScript library for building user interfaces."
  [2] "It is used for building UI components."
  [3] "React components are reusable pieces of UI."

Question: "What is React used for?"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

LLM Answer:
"React is a JavaScript library primarily used for building
 user interfaces. It lets you create reusable UI components,
 making it easier to build and maintain complex web applications."
```

---

## ⚙️ Key Components

### 🔷 Embedding Model

Converts text → vectors. Both the documents (during indexing) and the query (at runtime) must use the **same** embedding model.

Popular options: `text-embedding-ada-002` (OpenAI), `all-MiniLM-L6-v2` (open source)

---

### 🔷 Vector Database

Stores vectors and performs fast similarity search. Not a replacement for a regular database — used _alongside_ one.

|Feature|Regular SQL DB|Vector DB|
|---|---|---|
|Search type|Exact keyword match|Semantic / meaning-based|
|Query|`WHERE text LIKE '%X%'`|"Find meaning similar to X"|
|Use case|Structured data lookups|Unstructured text retrieval|
|Examples|PostgreSQL, MySQL|Pinecone, FAISS, Weaviate|

---

### 🔷 LLM (Large Language Model)

Reads the retrieved context and generates a natural language answer.

**What the LLM contributes:**

- Reasoning over the retrieved facts
- Summarizing and combining information
- Handling ambiguous or complex questions
- Producing fluent, human-readable output

**What the LLM does NOT do in RAG:**

- Memorize your documents (that's the vector DB's job)
- Decide what's relevant (that's the retrieval step's job)

---

## 🤝 Why You Need BOTH

```
                ┌──────────────────────────────────────┐
                │         Without Vector DB             │
                │                                       │
                │  LLM alone:                           │
                │  ✗ Doesn't know your private data     │
                │  ✗ Knowledge cutoff                   │
                │  ✗ Hallucinates facts                 │
                └──────────────────────────────────────┘

                ┌──────────────────────────────────────┐
                │           Without LLM                 │
                │                                       │
                │  Vector DB alone:                     │
                │  ✗ Only returns raw text chunks       │
                │  ✗ Can't synthesize or explain        │
                │  ✗ No natural language output         │
                └──────────────────────────────────────┘

                ┌──────────────────────────────────────┐
                │              RAG ✅                   │
                │                                       │
                │  Vector DB  → finds relevant facts    │
                │  LLM        → understands & explains  │
                │                                       │
                │  = Accurate answers from YOUR data    │
                └──────────────────────────────────────┘
```

---

## 🛒 Real Example: E-Commerce Chatbot

**Indexing (one-time setup):**

All product descriptions are chunked, embedded, and stored in the vector DB.

**At query time:**

```
User: "Best budget phone under $300?"
         ↓
Embed query → search vector DB
         ↓
Retrieved chunks:
  - "iPhone SE: Starting at $279, 4.7-inch display..."
  - "Samsung Galaxy A35: $269, 6.6-inch display, 5000mAh..."
         ↓
LLM generates:
  "Based on your $300 budget, the Samsung Galaxy A35 ($269)
   and iPhone SE ($279) are both excellent options. The A35
   offers a larger screen and bigger battery, while the SE
   gives you the iOS ecosystem..."
```

---

## 💡 The Analogy That Makes It Click

|Component|Analogy|
|---|---|
|**Embedding Model**|A translator that converts any question or document into a universal "meaning language"|
|**Vector Database**|A library index — tells you _which shelf_ has the relevant books|
|**LLM**|A knowledgeable librarian — reads the relevant books and _explains_ the answer to you|

> Without the index, the librarian has to read every book to answer your question (slow, prone to mistakes).  
> Without the librarian, the index just points you to a shelf but can't explain anything.  
> **Together: fast, accurate, and articulate.**

---

## 🔑 Glossary

|Term|Plain English|
|---|---|
|**Embedding**|A list of numbers that represents the _meaning_ of a piece of text|
|**Vector**|Another word for embedding — just a list of numbers|
|**Similarity Search**|Finding vectors (meanings) that are closest to the query vector|
|**Top-K Retrieval**|Returning the K most similar results (e.g., top 3, top 5)|
|**Cosine Similarity**|A math formula for measuring how "close" two vectors are in meaning|
|**Chunking**|Splitting large documents into smaller, focused pieces|
|**Context Window**|The maximum amount of text an LLM can read at once|

---

_RAG is the standard architecture for building AI systems that need to work with your own data — from internal knowledge bases and customer support bots to document Q&A tools and code assistants._