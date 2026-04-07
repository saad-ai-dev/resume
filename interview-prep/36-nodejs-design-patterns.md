# 36. NODE.JS DESIGN PATTERNS

## 36.1 Why Design Patterns Matter
Design patterns are proven, reusable solutions to common software design problems. In interviews, knowing these shows you write **maintainable, scalable, professional code** — not just code that works.

---

## 36.2 Creational Patterns

### Singleton Pattern
Ensures a class has only **one instance** throughout the application.

```javascript
// database.js — Only ONE connection pool for the entire app
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }
    this.pool = new Pool({
      host: process.env.DB_HOST,
      database: process.env.DB_NAME,
      max: 20,
    });
    Database.instance = this;
  }

  query(sql, params) {
    return this.pool.query(sql, params);
  }
}

module.exports = new Database();

// Usage — both get the SAME instance
const db1 = require('./database');
const db2 = require('./database');
console.log(db1 === db2); // true
```

**When to use:** Database connections, Redis clients, Logger instances, Config managers.

**Node.js shortcut:** `module.exports = new Instance()` — Node caches modules, so `require()` always returns the same object. This is a natural singleton.

**Your experience:** Sequelize instance shared across 100+ models at Obenan.

---

### Factory Pattern
Creates objects **without specifying the exact class**. A function decides which object to create based on input.

```javascript
// notification-factory.js
class EmailNotification {
  send(to, message) {
    return sendGrid.send({ to, subject: message.title, html: message.body });
  }
}

class PushNotification {
  send(to, message) {
    return firebase.messaging().send({ token: to, notification: message });
  }
}

class SMSNotification {
  send(to, message) {
    return twilio.messages.create({ to, body: message.body });
  }
}

class SlackNotification {
  send(to, message) {
    return slack.postMessage({ channel: to, text: message.body });
  }
}

// The Factory — caller doesn't need to know which class to use
function createNotification(type) {
  const notifications = {
    email: EmailNotification,
    push: PushNotification,
    sms: SMSNotification,
    slack: SlackNotification,
  };

  const NotificationClass = notifications[type];
  if (!NotificationClass) throw new Error(`Unknown notification type: ${type}`);
  return new NotificationClass();
}

// Usage
const notifier = createNotification('email');
await notifier.send('saad@email.com', { title: 'New Review', body: '...' });

const pushNotifier = createNotification('push');
await pushNotifier.send(deviceToken, { title: 'Match Started', body: '...' });
```

**When to use:** Multiple similar objects with different implementations, payment processors (Stripe vs PayPal), notification channels, database adapters.

**Your experience:** Different notification types at NowVPlay (match invitations, booking confirmations).

---

### Builder Pattern
Constructs complex objects **step by step**. Useful when an object has many optional parameters.

```javascript
class QueryBuilder {
  constructor(table) {
    this.table = table;
    this._select = '*';
    this._where = [];
    this._orderBy = null;
    this._limit = null;
    this._offset = null;
    this._joins = [];
  }

  select(...fields) {
    this._select = fields.join(', ');
    return this; // Return this for chaining
  }

  where(condition, value) {
    this._where.push({ condition, value });
    return this;
  }

  join(table, on) {
    this._joins.push(`JOIN ${table} ON ${on}`);
    return this;
  }

  orderBy(field, direction = 'ASC') {
    this._orderBy = `${field} ${direction}`;
    return this;
  }

  limit(n) {
    this._limit = n;
    return this;
  }

  offset(n) {
    this._offset = n;
    return this;
  }

  build() {
    let sql = `SELECT ${this._select} FROM ${this.table}`;
    if (this._joins.length) sql += ' ' + this._joins.join(' ');
    if (this._where.length) {
      sql += ' WHERE ' + this._where.map(w => w.condition).join(' AND ');
    }
    if (this._orderBy) sql += ` ORDER BY ${this._orderBy}`;
    if (this._limit) sql += ` LIMIT ${this._limit}`;
    if (this._offset) sql += ` OFFSET ${this._offset}`;
    return { sql, values: this._where.map(w => w.value) };
  }
}

// Usage — clean, readable, flexible
const query = new QueryBuilder('reviews')
  .select('id', 'content', 'rating', 'created_at')
  .join('locations', 'locations.id = reviews.location_id')
  .where('company_id = $1', companyId)
  .where('rating >= $2', 4)
  .orderBy('created_at', 'DESC')
  .limit(20)
  .offset(0)
  .build();

const results = await db.query(query.sql, query.values);
```

**When to use:** Complex query construction, HTTP request builders, configuration objects, email builders.

---

## 36.3 Structural Patterns

### Module Pattern
Encapsulates private state and exposes a **public API**. This is the foundation of Node.js itself.

```javascript
// auth-module.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

// Private — not exported
const SECRET = process.env.JWT_SECRET;
const SALT_ROUNDS = 12;

async function hashPassword(password) {
  return bcrypt.hash(password, SALT_ROUNDS);
}

// Public API — only these are accessible
module.exports = {
  async register(email, password) {
    const hash = await hashPassword(password);
    return db.users.create({ email, password: hash });
  },

  async login(email, password) {
    const user = await db.users.findOne({ where: { email } });
    if (!user) throw new Error('User not found');

    const valid = await bcrypt.compare(password, user.password);
    if (!valid) throw new Error('Invalid password');

    const token = jwt.sign({ userId: user.id, role: user.role }, SECRET, {
      expiresIn: '15m',
    });
    return { user, token };
  },

  verifyToken(token) {
    return jwt.verify(token, SECRET);
  },
};

// Usage
const auth = require('./auth-module');
const { user, token } = await auth.login('saad@email.com', 'password123');
// auth.hashPassword() → NOT accessible (private)
// auth.SECRET → NOT accessible (private)
```

**When to use:** Every Node.js file is essentially a module. Use this pattern to keep internals private and expose a clean API.

---

### Adapter Pattern
Wraps an incompatible interface to make it **work with your existing code**. Like a power adapter for different plug types.

```javascript
// You have code that expects this interface:
// { send(to, subject, body) }

// But SendGrid has a different API:
class SendGridAdapter {
  constructor(apiKey) {
    this.client = require('@sendgrid/mail');
    this.client.setApiKey(apiKey);
  }

  async send(to, subject, body) {
    return this.client.send({
      to,
      from: 'noreply@app.com',
      subject,
      html: body,
    });
  }
}

// And Mailgun has yet another API:
class MailgunAdapter {
  constructor(apiKey, domain) {
    this.client = require('mailgun-js')({ apiKey, domain });
  }

  async send(to, subject, body) {
    return this.client.messages().send({
      from: 'noreply@app.com',
      to,
      subject,
      html: body,
    });
  }
}

// Your code works with ANY email provider — just swap the adapter
const emailService = new SendGridAdapter(process.env.SENDGRID_KEY);
// OR
const emailService = new MailgunAdapter(process.env.MAILGUN_KEY, 'app.com');

// Same interface regardless of provider
await emailService.send('user@email.com', 'Welcome!', '<h1>Hello</h1>');
```

**When to use:** Switching between third-party services (email, payment, storage), API version migrations, legacy code integration.

**Your experience:** Switching between AI providers (OpenAI, Claude, Gemini) at Obenan — each has a different API but your code needs a uniform interface.

---

### Proxy Pattern
A wrapper that **controls access** to another object. Adds functionality without changing the original.

```javascript
// Caching proxy for database queries
class CachingProxy {
  constructor(database, redis) {
    this.db = database;
    this.redis = redis;
    this.ttl = 3600; // 1 hour
  }

  async query(key, queryFn) {
    // 1. Check cache first
    const cached = await this.redis.get(key);
    if (cached) {
      console.log(`Cache HIT: ${key}`);
      return JSON.parse(cached);
    }

    // 2. Cache miss — execute real query
    console.log(`Cache MISS: ${key}`);
    const result = await queryFn();

    // 3. Store in cache
    await this.redis.set(key, JSON.stringify(result), 'EX', this.ttl);

    return result;
  }

  async invalidate(pattern) {
    const keys = await this.redis.keys(pattern);
    if (keys.length) await this.redis.del(...keys);
  }
}

// Usage
const cache = new CachingProxy(db, redis);

const reviews = await cache.query(
  `reviews:location:${locationId}`,
  () => db.reviews.findAll({ where: { locationId } })
);

// After update, invalidate cache
await cache.invalidate(`reviews:location:${locationId}`);
```

**When to use:** Caching, logging, access control, lazy loading, rate limiting.

---

### Decorator Pattern
**Adds behavior** to an object dynamically without modifying it. In Node.js, this is often done with higher-order functions.

```javascript
// Base function
async function fetchReviews(locationId) {
  return db.reviews.findAll({ where: { locationId } });
}

// Decorator: Add logging
function withLogging(fn, label) {
  return async function(...args) {
    console.log(`[${label}] Called with:`, args);
    const start = Date.now();
    const result = await fn(...args);
    console.log(`[${label}] Completed in ${Date.now() - start}ms`);
    return result;
  };
}

// Decorator: Add retry logic
function withRetry(fn, maxRetries = 3) {
  return async function(...args) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await fn(...args);
      } catch (error) {
        if (attempt === maxRetries) throw error;
        console.log(`Attempt ${attempt} failed, retrying...`);
        await new Promise(r => setTimeout(r, 1000 * attempt));
      }
    }
  };
}

// Decorator: Add caching
function withCache(fn, redis, ttl = 3600) {
  return async function(...args) {
    const key = `cache:${fn.name}:${JSON.stringify(args)}`;
    const cached = await redis.get(key);
    if (cached) return JSON.parse(cached);

    const result = await fn(...args);
    await redis.set(key, JSON.stringify(result), 'EX', ttl);
    return result;
  };
}

// Stack decorators — each adds a layer of behavior
const enhancedFetch = withLogging(
  withRetry(
    withCache(fetchReviews, redis),
    3
  ),
  'ReviewFetch'
);

const reviews = await enhancedFetch(locationId);
// Logs → Retries on failure → Checks cache → Fetches from DB
```

**When to use:** Adding cross-cutting concerns (logging, caching, retry, auth, validation) without modifying original functions.

---

## 36.4 Behavioral Patterns

### Observer Pattern (EventEmitter)
Objects **subscribe to events** and get notified when something happens. Node.js EventEmitter is the built-in implementation.

```javascript
const EventEmitter = require('events');

class ReviewService extends EventEmitter {
  async createReview(data) {
    const review = await db.reviews.create(data);

    // Emit event — let listeners handle side effects
    this.emit('review:created', review);

    return review;
  }

  async deleteReview(id) {
    const review = await db.reviews.findByPk(id);
    await review.destroy();

    this.emit('review:deleted', review);

    return review;
  }
}

const reviewService = new ReviewService();

// Listener 1: Send notification
reviewService.on('review:created', async (review) => {
  await notificationService.send(review.ownerId, 'New review received!');
});

// Listener 2: Update analytics
reviewService.on('review:created', async (review) => {
  await analyticsService.trackReview(review);
});

// Listener 3: Run sentiment analysis
reviewService.on('review:created', async (review) => {
  await sentimentQueue.add('analyze', { reviewId: review.id });
});

// Listener 4: Invalidate cache
reviewService.on('review:created', async (review) => {
  await redis.del(`reviews:location:${review.locationId}`);
});

// Listener 5: Log the event
reviewService.on('review:deleted', async (review) => {
  logger.info(`Review ${review.id} deleted by ${review.deletedBy}`);
});
```

**Why this is powerful:**
- `createReview()` only handles creating the review
- Side effects (notifications, analytics, AI, cache) are **decoupled**
- Adding a new side effect = just add another listener
- No need to modify the original function

**Your experience:** Socket.IO events at Zamulk/NowVPlay, BullMQ job events at Obenan.

---

### Strategy Pattern
Defines a **family of algorithms** and makes them interchangeable. The caller picks which strategy to use at runtime.

```javascript
// Sentiment analysis strategies — different AI providers
const strategies = {
  openai: {
    async analyze(text) {
      const response = await openai.chat.completions.create({
        model: 'gpt-4o-mini',
        messages: [
          { role: 'system', content: 'Classify sentiment as positive/negative/neutral. Return JSON.' },
          { role: 'user', content: text },
        ],
        temperature: 0,
        response_format: { type: 'json_object' },
      });
      return JSON.parse(response.choices[0].message.content);
    },
  },

  claude: {
    async analyze(text) {
      const response = await anthropic.messages.create({
        model: 'claude-haiku-4-5-20251001',
        max_tokens: 100,
        messages: [{ role: 'user', content: `Classify sentiment: "${text}". Return JSON.` }],
      });
      return JSON.parse(response.content[0].text);
    },
  },

  local: {
    analyze(text) {
      // Simple keyword-based fallback (no API cost)
      const positive = ['great', 'amazing', 'excellent', 'love', 'best'];
      const negative = ['terrible', 'awful', 'worst', 'hate', 'bad'];
      const words = text.toLowerCase().split(/\s+/);
      const posCount = words.filter(w => positive.includes(w)).length;
      const negCount = words.filter(w => negative.includes(w)).length;
      if (posCount > negCount) return { sentiment: 'positive' };
      if (negCount > posCount) return { sentiment: 'negative' };
      return { sentiment: 'neutral' };
    },
  },
};

// Service picks strategy based on config or context
class SentimentService {
  constructor(strategyName = 'openai') {
    this.strategy = strategies[strategyName];
    if (!this.strategy) throw new Error(`Unknown strategy: ${strategyName}`);
  }

  setStrategy(strategyName) {
    this.strategy = strategies[strategyName];
  }

  async analyze(text) {
    return this.strategy.analyze(text);
  }
}

// Usage
const sentiment = new SentimentService('openai');
await sentiment.analyze('Great food!'); // Uses OpenAI

sentiment.setStrategy('claude');
await sentiment.analyze('Terrible service'); // Now uses Claude

sentiment.setStrategy('local');
sentiment.analyze('Best pizza ever'); // Free, no API call
```

**When to use:** Multiple algorithms for the same task, A/B testing different approaches, switching providers, cost optimization (cheap model for simple tasks, expensive for complex).

**Your experience:** Switching between GPT-4, Claude, and Gemini for sentiment analysis at Obenan.

---

### Middleware/Chain of Responsibility Pattern
Passes a request through a **chain of handlers**. Each handler can process the request, modify it, or pass it along. This is the core of Express.js.

```javascript
// Express middleware IS the Chain of Responsibility pattern
// Each middleware is a "handler" in the chain

// Handler 1: Parse body
app.use(express.json());

// Handler 2: Log request
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // Pass to next handler
});

// Handler 3: Authenticate
app.use('/api', (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token' });
  req.user = jwt.verify(token, SECRET);
  next();
});

// Handler 4: Authorize role
function requireRole(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// Handler 5: Validate input
function validateBody(schema) {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    if (error) return res.status(400).json({ error: error.message });
    next();
  };
}

// Handler 6: Rate limit
const apiLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });

// Chain them together
router.post('/reviews',
  apiLimiter,                              // Rate limit
  requireRole('admin', 'manager'),         // Authorization
  validateBody(reviewSchema),              // Validation
  async (req, res) => {                    // Final handler
    const review = await reviewService.create(req.body);
    res.status(201).json(review);
  }
);

// Request flows: Rate Limit → Auth → Role Check → Validate → Handler
```

**When to use:** Request processing pipelines, validation chains, logging/monitoring chains, error handling.

**Your experience:** Every Express/NestJS app you've built uses this pattern.

---

### Repository Pattern
Abstracts data access into a **separate layer**. Your business logic never talks directly to the database.

```javascript
// user-repository.js
class UserRepository {
  constructor(model) {
    this.model = model;
  }

  async findById(id) {
    return this.model.findByPk(id);
  }

  async findByEmail(email) {
    return this.model.findOne({ where: { email } });
  }

  async findAll({ page = 1, limit = 20, role, companyId }) {
    const where = {};
    if (role) where.role = role;
    if (companyId) where.companyId = companyId;

    return this.model.findAndCountAll({
      where,
      limit,
      offset: (page - 1) * limit,
      order: [['createdAt', 'DESC']],
    });
  }

  async create(data) {
    return this.model.create(data);
  }

  async update(id, data) {
    const user = await this.findById(id);
    if (!user) throw new NotFoundException('User not found');
    return user.update(data);
  }

  async delete(id) {
    const user = await this.findById(id);
    if (!user) throw new NotFoundException('User not found');
    return user.destroy(); // Soft delete if paranoid: true
  }
}

// user-service.js — Business logic uses repository, never touches DB directly
class UserService {
  constructor(userRepo, emailService) {
    this.userRepo = userRepo;
    this.emailService = emailService;
  }

  async registerUser(data) {
    const existing = await this.userRepo.findByEmail(data.email);
    if (existing) throw new ConflictException('Email already registered');

    const user = await this.userRepo.create({
      ...data,
      password: await bcrypt.hash(data.password, 12),
    });

    await this.emailService.sendWelcome(user.email, user.name);
    return user;
  }
}

// If you switch from Sequelize to TypeORM or even MongoDB,
// only the Repository changes — Service stays the same
```

**When to use:** Any app with database access. Keeps business logic clean, makes testing easier (mock the repository), makes DB migrations possible.

**Your experience:** 100+ Sequelize models at Obenan — repository pattern keeps each model's queries organized.

---

## 36.5 Concurrency Patterns

### Pub/Sub Pattern
Publishers send messages to a **channel**, subscribers receive them. Decouples senders from receivers.

```javascript
// Using Redis Pub/Sub
const Redis = require('ioredis');
const publisher = new Redis();
const subscriber = new Redis();

// Publisher (e.g., API server that receives a new review)
async function publishReviewEvent(review) {
  await publisher.publish('reviews', JSON.stringify({
    event: 'new_review',
    data: review,
    timestamp: Date.now(),
  }));
}

// Subscriber 1: Notification service
subscriber.subscribe('reviews');
subscriber.on('message', (channel, message) => {
  const { event, data } = JSON.parse(message);
  if (event === 'new_review') {
    sendNotification(data.ownerId, `New ${data.rating}-star review!`);
  }
});

// Subscriber 2: Analytics service (separate process)
subscriber2.subscribe('reviews');
subscriber2.on('message', (channel, message) => {
  const { event, data } = JSON.parse(message);
  if (event === 'new_review') {
    updateDashboardMetrics(data.locationId);
  }
});
```

**Difference from Observer:**
- **Observer:** Same process, in-memory events (EventEmitter)
- **Pub/Sub:** Cross-process, distributed via a broker (Redis, RabbitMQ, Kafka)

**Your experience:** Redis Pub/Sub for Socket.IO scaling across multiple server instances.

---

### Circuit Breaker Pattern
**Prevents cascading failures** when an external service is down. Like an electrical circuit breaker.

```javascript
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.failureThreshold = options.failureThreshold || 5;
    this.resetTimeout = options.resetTimeout || 30000; // 30 seconds
    this.state = 'CLOSED';  // CLOSED = normal, OPEN = blocked, HALF_OPEN = testing
    this.failureCount = 0;
    this.lastFailureTime = null;
  }

  async execute(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF_OPEN'; // Try one request
      } else {
        throw new Error('Circuit breaker is OPEN — service unavailable');
      }
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      console.warn('Circuit breaker OPENED — too many failures');
    }
  }
}

// Usage — protect calls to external APIs
const openaiBreaker = new CircuitBreaker(
  (text) => openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [{ role: 'user', content: text }],
  }),
  { failureThreshold: 3, resetTimeout: 60000 }
);

try {
  const result = await openaiBreaker.execute('Analyze this review');
} catch (error) {
  // Fallback to local analysis or different provider
  const result = localSentimentAnalysis(text);
}
```

```
States:
CLOSED ──(failure threshold reached)──> OPEN
  ↑                                       │
  │                                       │ (timeout expires)
  │                                       ↓
  └────(success)──── HALF_OPEN ──(failure)──> OPEN
```

**When to use:** External API calls (OpenAI, Google, Stripe), database connections, any unreliable dependency.

---

## 36.6 Design Patterns Comparison Table

| Pattern | Category | Problem It Solves | Your Resume Example |
|---------|----------|-------------------|-------------------|
| **Singleton** | Creational | Only one instance needed | DB connection, Redis client |
| **Factory** | Creational | Create objects without specifying class | Notification types at NowVPlay |
| **Builder** | Creational | Complex object construction | Query builders, email builders |
| **Module** | Structural | Encapsulate private state | Every Node.js file |
| **Adapter** | Structural | Incompatible interfaces | Switching AI providers (OpenAI/Claude/Gemini) |
| **Proxy** | Structural | Control access, add caching | Redis caching layer |
| **Decorator** | Structural | Add behavior dynamically | Logging, retry, caching wrappers |
| **Observer** | Behavioral | React to events | Socket.IO events, BullMQ job events |
| **Strategy** | Behavioral | Swap algorithms at runtime | AI model selection for sentiment |
| **Middleware** | Behavioral | Chain of request handlers | Express/NestJS middleware |
| **Repository** | Behavioral | Abstract data access | Sequelize models at Obenan |
| **Pub/Sub** | Concurrency | Cross-process communication | Redis Pub/Sub for Socket.IO scaling |
| **Circuit Breaker** | Concurrency | Prevent cascading failures | External API protection |

## 36.7 Common Interview Questions

**Q: What design patterns have you used in production?**
> "In my work at Obenan, I've used several patterns daily:
> - **Repository pattern** for abstracting 100+ Sequelize models
> - **Factory pattern** for creating different notification types
> - **Strategy pattern** for switching between OpenAI, Claude, and Gemini for sentiment analysis
> - **Observer/EventEmitter** for decoupling review creation from side effects like notifications and analytics
> - **Middleware chain** in Express/NestJS for auth, validation, and rate limiting
> - **Singleton** for database and Redis connections
> - **Pub/Sub** via Redis for scaling Socket.IO across multiple instances"

**Q: What's the difference between Factory and Builder?**
- **Factory:** Creates a complete object in one step based on type → `createNotification('email')`
- **Builder:** Constructs a complex object step by step → `new QueryBuilder().select().where().limit().build()`

**Q: What's the difference between Adapter and Proxy?**
- **Adapter:** Changes the interface to make incompatible things work together (different shape)
- **Proxy:** Same interface but adds extra behavior like caching, logging, or access control (same shape)

**Q: When would you NOT use a design pattern?**
- When simple code solves the problem
- When adding a pattern makes the code harder to understand
- When there's only one implementation (no need for Strategy with one algorithm)
- "The best code is the simplest code that works"

