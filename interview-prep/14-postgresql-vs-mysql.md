# 14. POSTGRESQL VS MYSQL — COMPARISON

| Feature | PostgreSQL | MySQL |
|---------|-----------|-------|
| **JSONB support** | Excellent (indexable, queryable) | Basic JSON |
| **Full-text search** | Built-in (tsvector) | Built-in (less powerful) |
| **Arrays** | Native support | No |
| **CTEs (WITH)** | Full support | Added in 8.0 |
| **Window functions** | Full support | Added in 8.0 |
| **UPSERT** | ON CONFLICT DO UPDATE | ON DUPLICATE KEY UPDATE |
| **Replication** | Streaming, logical | Master-slave, group |
| **Performance** | Better for complex queries | Better for simple reads |
| **Extensions** | PostGIS, pgvector, pg_trgm | Limited |
| **License** | PostgreSQL License (very permissive) | GPL (Oracle owned) |

