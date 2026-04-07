# 33. SYSTEM DESIGN PATTERNS

## 33.1 Multi-Tenant Architecture (Your Obenan Experience)

```
┌──────────────────────────────────────────────┐
│                  Shared Application            │
├──────────────────────────────────────────────┤
│  Company A Data │ Company B Data │ Company C  │
│  ┌────────────┐ │ ┌────────────┐ │            │
│  │ Users      │ │ │ Users      │ │            │
│  │ Locations  │ │ │ Locations  │ │            │
│  │ Reviews    │ │ │ Reviews    │ │            │
│  └────────────┘ │ └────────────┘ │            │
├──────────────────────────────────────────────┤
│              Shared Database                   │
│  WHERE companyId = :companyId  (row-level)    │
└──────────────────────────────────────────────┘
```

**Isolation Strategies:**
1. **Separate databases** per tenant (most isolated, most expensive)
2. **Separate schemas** per tenant (good isolation, moderate cost)
3. **Shared database, row-level** isolation (least isolated, cheapest) ← Your approach

## 33.2 Microservices vs Monolith

| Feature | Monolith | Microservices |
|---------|----------|---------------|
| Deployment | One unit | Independent services |
| Scaling | Scale everything | Scale individual services |
| Complexity | Simple initially | Complex (networking, orchestration) |
| Data | Shared database | Each service owns its data |
| Team | One team | Multiple small teams |
| Best for | Startups, small teams | Large teams, high scale |

## 33.3 Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                   // 100 requests per window
  message: { error: 'Too many requests, try again later' },
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', limiter);
```

## 33.4 API Pagination Patterns

```javascript
// Offset-based (simple, but slow on large datasets)
GET /api/reviews?page=3&limit=20
// SELECT * FROM reviews LIMIT 20 OFFSET 40

// Cursor-based (better performance, used by Facebook/Twitter)
GET /api/reviews?cursor=eyJpZCI6MTAwfQ&limit=20
// SELECT * FROM reviews WHERE id > 100 LIMIT 20
```

