# 10. REST VS GRAPHQL — COMPARISON

| Feature | REST | GraphQL |
|---------|------|---------|
| **Data fetching** | Fixed endpoints, fixed data | Client specifies exact fields |
| **Over-fetching** | Common (get all fields) | Never (get only what you ask) |
| **Under-fetching** | Common (need multiple requests) | Never (get everything in one query) |
| **Endpoints** | Multiple (/users, /posts, etc.) | Single (/graphql) |
| **Versioning** | /v1, /v2 | Schema evolution, no versioning needed |
| **Caching** | Easy (HTTP caching) | Harder (single endpoint, POST requests) |
| **File upload** | Native | Not native (needs multipart spec) |
| **Learning curve** | Lower | Higher |
| **Error handling** | HTTP status codes | Always 200, errors in response body |
| **Best for** | Simple CRUD, public APIs, caching | Complex UIs, mobile apps, nested data |
| **Tooling** | Postman, curl | Apollo, GraphQL Playground |

**Interview Q: When to use REST vs GraphQL?**
- **REST:** Simple CRUD, public APIs, microservices, when caching matters
- **GraphQL:** Complex nested data, mobile apps (bandwidth sensitive), rapidly changing frontend needs

