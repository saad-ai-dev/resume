# 21. DOCKER

## 21.1 What is Docker?
Docker packages applications into **containers** — lightweight, portable, self-sufficient environments that run anywhere.

## 21.2 Key Concepts

```
Image  → Blueprint (recipe)
Container → Running instance (the dish)
Dockerfile → Instructions to build image
Docker Compose → Define multi-container apps
```

### Dockerfile
```dockerfile
# Multi-stage build (smaller final image)
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

EXPOSE 3000
USER node
CMD ["node", "dist/main.js"]
```

### Docker Compose
```yaml
version: '3.8'
services:
  api:
    build: .
    ports: ["3000:3000"]
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on: [db, redis]

  db:
    image: postgres:15-alpine
    volumes: ["pgdata:/var/lib/postgresql/data"]
    environment:
      POSTGRES_DB: mydb
      POSTGRES_PASSWORD: pass

  redis:
    image: redis:7-alpine

  mongo:
    image: mongo:7
    volumes: ["mongodata:/data/db"]

volumes:
  pgdata:
  mongodata:
```

### Common Commands
```bash
docker build -t myapp .           # Build image
docker run -p 3000:3000 myapp     # Run container
docker ps                         # List running containers
docker logs <container>           # View logs
docker exec -it <container> sh    # Shell into container
docker-compose up -d              # Start all services
docker-compose down               # Stop all services
```

