# 15. REDIS

## 15.1 What is Redis?
Redis is an **in-memory key-value data store** used for caching, session storage, message brokering (pub/sub), rate limiting, and job queues.

## 15.2 Data Structures

```bash
# String
SET user:1:name "Saad"
GET user:1:name

# Hash (like an object)
HSET user:1 name "Saad" age 28 city "Lahore"
HGETALL user:1

# List (ordered)
LPUSH queue "job1" "job2"
RPOP queue

# Set (unique values)
SADD skills "Node.js" "React" "Node.js"  # Only 2 stored
SMEMBERS skills

# Sorted Set (ranked)
ZADD leaderboard 100 "player1" 200 "player2"
ZRANGE leaderboard 0 -1 WITHSCORES

# TTL (auto-expire)
SET session:abc "data" EX 3600  # Expires in 1 hour
```

## 15.3 Use Cases from Your Resume

| Use Case | How You Used It |
|----------|----------------|
| **BullMQ queues** | Redis as the backing store for job queues |
| **Caching** | Cache API responses, database queries |
| **Session storage** | Store user sessions (faster than DB) |
| **Rate limiting** | Track request counts per user/IP |
| **Pub/Sub** | Real-time event broadcasting |

## 15.4 Caching Strategies

| Strategy | Description |
|----------|-------------|
| **Cache-Aside** | App checks cache first → if miss, read DB → write to cache |
| **Write-Through** | Write to cache AND DB simultaneously |
| **Write-Behind** | Write to cache, async write to DB later |
| **Read-Through** | Cache handles DB reads automatically |

```javascript
// Cache-Aside pattern (most common)
async function getUser(id) {
  // 1. Check cache
  const cached = await redis.get(`user:${id}`);
  if (cached) return JSON.parse(cached);

  // 2. Cache miss — read from DB
  const user = await db.users.findByPk(id);

  // 3. Write to cache with TTL
  await redis.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);

  return user;
}
```

