# 8. REST APIs

## 8.1 What is REST?
REST (Representational State Transfer) is an **architectural style** for designing networked applications. It uses HTTP methods to perform CRUD operations on resources.

## 8.2 HTTP Methods

| Method | Action | Idempotent | Safe | Body |
|--------|--------|-----------|------|------|
| GET | Read | Yes | Yes | No |
| POST | Create | No | No | Yes |
| PUT | Replace entirely | Yes | No | Yes |
| PATCH | Update partially | Yes | No | Yes |
| DELETE | Remove | Yes | No | Optional |

**Idempotent:** Making the same request multiple times produces the same result.
**Safe:** Doesn't modify server state.

## 8.3 HTTP Status Codes

| Code | Meaning | When to use |
|------|---------|-------------|
| 200 | OK | Successful GET, PUT, PATCH |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input, validation errors |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource (e.g., email already exists) |
| 422 | Unprocessable Entity | Valid syntax but semantic errors |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Unhandled server error |

## 8.4 REST Best Practices

```
# Good API Design
GET    /api/v1/users              ← List users
GET    /api/v1/users/123          ← Get user 123
POST   /api/v1/users              ← Create user
PUT    /api/v1/users/123          ← Replace user 123
PATCH  /api/v1/users/123          ← Update user 123
DELETE /api/v1/users/123          ← Delete user 123
GET    /api/v1/users/123/posts    ← Get user 123's posts

# Bad API Design
GET    /api/getUser/123           ← Verbs in URL
POST   /api/deleteUser            ← Using POST for delete
GET    /api/v1/User               ← Singular (should be plural)
```

**Key Principles:**
1. Use **nouns** (not verbs) in URLs
2. Use **plural** resource names
3. **Version** your API (v1, v2)
4. Use proper **status codes**
5. Support **pagination** (`?page=1&limit=20`)
6. Support **filtering** (`?status=active&role=admin`)
7. Support **sorting** (`?sort=createdAt&order=desc`)
8. Return **consistent response format**

```json
{
  "success": true,
  "data": { },
  "meta": { "page": 1, "limit": 20, "total": 150 },
  "message": "Users retrieved successfully"
}
```

