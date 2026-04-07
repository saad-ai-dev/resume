# 25. RAG ARCHITECTURE

## 25.1 What is RAG?
Retrieval-Augmented Generation combines a retrieval system with an LLM. Instead of the LLM relying only on training data, it retrieves relevant documents first, then generates answers based on them.

## 25.2 Why RAG?
- LLMs don't know your private data
- Training data has a cutoff date
- Fine-tuning is expensive and slow
- RAG gives you up-to-date, domain-specific answers

## 25.3 Architecture

```
┌────────────────────────────────────────────────────┐
│                    INDEXING PHASE                    │
│                                                      │
│  Documents → Chunk → Embed → Store in Vector DB     │
│                                                      │
│  "Company policy.pdf"                                │
│       ↓                                              │
│  [Chunk 1] [Chunk 2] [Chunk 3] ...                  │
│       ↓                                              │
│  [0.23, -0.15, ...] [0.45, 0.32, ...] ...          │ (embeddings)
│       ↓                                              │
│  ┌──────────────────┐                                │
│  │   Vector Database  │ (Pinecone, pgvector, Chroma) │
│  └──────────────────┘                                │
└────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│                    QUERY PHASE                       │
│                                                      │
│  User Query → Embed → Search Vector DB → Top K      │
│       ↓                                              │
│  "What is our refund policy?"                        │
│       ↓                                              │
│  [0.24, -0.14, ...]  (query embedding)              │
│       ↓                                              │
│  Vector DB returns top 5 most similar chunks         │
│       ↓                                              │
│  ┌───────────────────────────────┐                   │
│  │  PROMPT = System + Context     │                  │
│  │  + Retrieved Chunks + Query    │ → LLM → Answer   │
│  └───────────────────────────────┘                   │
└────────────────────────────────────────────────────┘
```

## 25.4 Implementation

```javascript
// 1. Chunking
function chunkText(text, chunkSize = 500, overlap = 50) {
  const chunks = [];
  for (let i = 0; i < text.length; i += chunkSize - overlap) {
    chunks.push(text.slice(i, i + chunkSize));
  }
  return chunks;
}

// 2. Generate embeddings
async function getEmbedding(text) {
  const response = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: text,
  });
  return response.data[0].embedding; // [0.23, -0.15, ...]
}

// 3. Store in pgvector (PostgreSQL)
// CREATE EXTENSION vector;
// CREATE TABLE documents (
//   id SERIAL PRIMARY KEY,
//   content TEXT,
//   embedding vector(1536),
//   metadata JSONB
// );

await db.query(
  'INSERT INTO documents (content, embedding, metadata) VALUES ($1, $2, $3)',
  [chunk, JSON.stringify(embedding), { source: 'policy.pdf', page: 3 }]
);

// 4. Search (similarity)
async function search(query, topK = 5) {
  const queryEmbedding = await getEmbedding(query);

  const results = await db.query(`
    SELECT content, metadata,
           1 - (embedding <=> $1::vector) as similarity
    FROM documents
    ORDER BY embedding <=> $1::vector
    LIMIT $2
  `, [JSON.stringify(queryEmbedding), topK]);

  return results.rows;
}

// 5. Generate answer with context
async function answerQuestion(question) {
  const relevantDocs = await search(question, 5);

  const context = relevantDocs.map(d => d.content).join('\n\n');

  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `Answer based ONLY on the provided context. If the answer is not in the context, say "I don't know."

Context:
${context}`
      },
      { role: 'user', content: question }
    ],
    temperature: 0,
  });

  return {
    answer: response.choices[0].message.content,
    sources: relevantDocs.map(d => d.metadata),
  };
}
```

## 25.5 Common RAG Problems & Solutions

| Problem | Solution |
|---------|----------|
| Bad retrieval | Better chunking strategy, metadata filtering, hybrid search |
| Hallucination | Lower temperature, "only answer from context" instruction |
| PDF parsing issues | Use specialized parsers (unstructured, LlamaParse) |
| Slow queries | Pre-filter by metadata, use approximate nearest neighbor |
| Outdated data | Scheduled re-indexing pipeline |
| Too much context | Reranking (Cohere Rerank), summarize before sending to LLM |

