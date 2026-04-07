# 13. SQL VS NoSQL — COMPLETE COMPARISON

| Feature | SQL (PostgreSQL/MySQL) | NoSQL (MongoDB) |
|---------|----------------------|------------------|
| **Data Model** | Tables, rows, columns | Documents (JSON/BSON) |
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Relationships** | JOINs, foreign keys | Embedding or referencing |
| **Scaling** | Vertical (bigger server) | Horizontal (more servers, sharding) |
| **Transactions** | Full ACID | ACID per document (multi-doc since v4.0) |
| **Query Language** | SQL (standardized) | MongoDB Query Language (proprietary) |
| **Best for** | Structured data, complex queries, financial data | Unstructured/varied data, rapid prototyping, IoT |
| **Normalization** | Normalized (less duplication) | Denormalized (faster reads, more duplication) |
| **Joins** | Native, optimized | $lookup (slower, not native) |
| **Indexes** | B-Tree, GIN, GiST | B-Tree, Text, Geospatial, Hashed |
| **Consistency** | Strong consistency | Eventual consistency (configurable) |
| **Your usage** | Obenan (100+ models, 400+ migrations) | Zamulk, NowVPlay |

**Interview Q: When to use SQL?**
- Financial data (transactions, ACID required)
- Complex relationships (many JOINs)
- Data integrity is critical
- Reporting and analytics
- Structured, predictable data

**Interview Q: When to use NoSQL?**
- Rapidly changing schema
- High write throughput
- Geographic distribution
- Document-like data (profiles, catalogs, CMS)
- Real-time analytics

**Interview Q: Can you use both? (Your experience)**
Yes — at Obenan/Global Software Consulting, you built a **dual-database system**: PostgreSQL for structured business data (companies, users, subscriptions) + MongoDB for flexible data (reviews, metadata, logs). This is a common pattern called **polyglot persistence**.

