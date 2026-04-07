# 38. JAVASCRIPT & NODE.JS — ADVANCED INTERVIEW PRACTICE

## The 8 Golden Rules Framework

Memorize these. Apply them to EVERY execution-order question.

```
┌────────────────────────────────────────────────────────────┐
│  RULE 1: Sync code ALWAYS runs first — top to bottom       │
│  RULE 2: Microtasks drain completely before ANY macrotask   │
│  RULE 3: process.nextTick > Promise.then — always           │
│  RULE 4: setTimeout(0) ≠ "run immediately"                  │
│  RULE 5: Promise executor is sync, .then/.catch are async   │
│  RULE 6: Function arguments are evaluated synchronously     │
│  RULE 7: After EACH macrotask, drain all microtasks again   │
│  RULE 8: await pauses the function, not the program         │
└────────────────────────────────────────────────────────────┘
```

---

## 38.1 Advanced Execution Order Challenges

### Challenge 1: The Classic Trap — for loop with var

**What:** Demonstrates why `var` in a loop with `setTimeout` produces unexpected results due to function scoping and closure.
**Why:** This is the single most asked closure + async question in JavaScript interviews — getting it wrong is an instant red flag.

```javascript
// BEFORE (the bug)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

**Your prediction before reading the answer:** ___

**Output:**
```
3
3
3
```

**Step-by-step reasoning (applying Golden Rules):**
1. **Rule 1 (sync first):** The `for` loop runs entirely synchronously — i goes 0→1→2→3, loop ends
2. **Rule 4 (setTimeout ≠ immediate):** All 3 `setTimeout` callbacks are REGISTERED, not executed
3. `var` is function-scoped — there's only ONE `i` variable shared by all 3 callbacks
4. By the time callbacks run (Rule 2 — after sync), `i` is already `3`
5. All 3 callbacks print the SAME `i` → 3, 3, 3

**AFTER (3 different fixes):**

```javascript
// Fix 1: Use let (block-scoped — creates new i per iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// Output: 0, 1, 2

// Fix 2: IIFE (creates new scope per iteration)
for (var i = 0; i < 3; i++) {
  ((j) => {
    setTimeout(() => console.log(j), 0);
  })(i);
}
// Output: 0, 1, 2

// Fix 3: setTimeout's third argument (passes value, not reference)
for (var i = 0; i < 3; i++) {
  setTimeout((j) => console.log(j), 0, i);
}
// Output: 0, 1, 2
```

**Gotcha:** Fix 3 is the least known — `setTimeout(fn, delay, ...args)` passes extra args to the callback. Mentioning this in an interview shows deep knowledge.

---

### Challenge 2: Promise Executor Side Effects

**What:** Shows that code inside `new Promise(executor)` runs synchronously, including side effects like console.log and variable mutations.
**Why:** Interviewers use this to catch people who think everything inside a Promise is asynchronous — only `.then()` callbacks are async.

```javascript
console.log('A');

const p = new Promise((resolve) => {
  console.log('B');
  resolve('C');
  console.log('D');  // Does this run?
});

p.then((val) => console.log(val));

console.log('E');
```

**Your prediction:** ___

**Output:**
```
A    ← Rule 1: sync
B    ← Rule 5: Promise executor is SYNC
D    ← Rule 5: resolve() doesn't stop execution! Code after resolve still runs
E    ← Rule 1: sync continues outside promise
C    ← Rule 2: .then() is microtask, runs after all sync
```

**Common misconception:** `resolve()` does NOT return or break. Code after `resolve()` still executes synchronously. The promise is fulfilled, but execution continues.

**Best practice:**
```javascript
// If you want to stop after resolve, use return
const p = new Promise((resolve) => {
  console.log('B');
  return resolve('C');  // return prevents further execution
  console.log('D');     // This will NOT run
});
```

---

### Challenge 3: Multiple await in Sequence vs Parallel

**What:** Demonstrates the performance difference between sequential `await` (slow) and `Promise.all` (fast) for independent async operations.
**Why:** This is a real production bug that costs companies seconds of response time — interviewers want to see if you know the pattern.

```javascript
// BEFORE: Sequential (SLOW — 3 seconds total)
async function getDataSlow() {
  const users = await fetchUsers();     // 1 sec
  const posts = await fetchPosts();     // 1 sec
  const comments = await fetchComments(); // 1 sec
  return { users, posts, comments };
}

// AFTER: Parallel (FAST — 1 second total)
async function getDataFast() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),     // 1 sec ─┐
    fetchPosts(),     // 1 sec ─┤ All run simultaneously
    fetchComments()   // 1 sec ─┘
  ]);
  return { users, posts, comments };
}
```

**Step-by-step reasoning (applying Rule 8):**
- In `getDataSlow`: Each `await` pauses the function, waits for completion, THEN moves to next line
- In `getDataFast`: All 3 promises START immediately (sync), then `await Promise.all` waits for all to finish

**When to use which:**
| Pattern | Use When |
|---------|----------|
| Sequential `await` | Each call depends on the previous result |
| `Promise.all` | Calls are independent (no data dependency) |
| `Promise.allSettled` | You need all results even if some fail |
| `Promise.race` | You want the fastest response (e.g., timeout pattern) |

**Performance implication:** On an API serving 1000 requests/sec, sequential awaits on 3 database queries (100ms each) = 300ms per request. Parallel = 100ms. That's 3x throughput improvement.

---

### Challenge 4: Microtask Starvation

**What:** A scenario where continuously adding microtasks prevents macrotasks (like setTimeout) from ever executing, effectively freezing the event loop.
**Why:** Understanding this explains why `process.nextTick` can starve I/O, and why you should prefer `setImmediate` in Node.js for yielding control.

```javascript
// DANGER: This will STARVE the event loop
function recursiveNextTick() {
  process.nextTick(() => {
    console.log('nextTick');
    recursiveNextTick(); // Adds another nextTick BEFORE any macrotask runs
  });
}

setTimeout(() => console.log('I will NEVER run'), 0);
recursiveNextTick();
```

**Why it starves:** Rule 3 says nextTick runs before macrotasks. Rule 2 says microtasks drain completely. Since each nextTick adds another nextTick, the microtask queue NEVER empties → setTimeout callback NEVER executes.

**Fix — use setImmediate instead:**
```javascript
function recursiveImmediate() {
  setImmediate(() => {
    console.log('immediate');
    recursiveImmediate();
  });
}

setTimeout(() => console.log('I WILL run!'), 0);
recursiveImmediate();
// setTimeout callback runs between setImmediate iterations
```

**Real-world impact:** At Obenan, if BullMQ job processing used recursive nextTick instead of proper queue management, it could starve the HTTP server from processing new requests.

---

### Challenge 5: this + Arrow Functions in Classes

**What:** Shows how `this` binding differs between regular methods and arrow functions, especially when methods are passed as callbacks.
**Why:** This is a daily source of bugs in Express/NestJS handlers and event listeners — interviewers test this to see if you can debug real-world issues.

```javascript
class UserService {
  constructor(name) {
    this.name = name;
  }

  // Regular method — `this` depends on HOW it's called
  greet() {
    return `Hello, ${this.name}`;
  }

  // Arrow method — `this` is ALWAYS the instance (lexical binding)
  greetArrow = () => {
    return `Hello, ${this.name}`;
  }
}

const service = new UserService('Saad');

// Direct call — both work
service.greet();      // "Hello, Saad" ✓
service.greetArrow(); // "Hello, Saad" ✓

// Passed as callback — only arrow works!
const greetFn = service.greet;
const greetArrowFn = service.greetArrow;

greetFn();           // "Hello, undefined" ✗ (this = global/undefined)
greetArrowFn();      // "Hello, Saad" ✓ (this = always instance)

// Fix for regular methods:
const greetBound = service.greet.bind(service);
greetBound();        // "Hello, Saad" ✓
```

**Express.js real-world example:**
```javascript
// BUG: `this` is lost when Express calls the method
app.get('/users', userController.getAll);  // this = undefined!

// FIX 1: Arrow function
app.get('/users', (req, res) => userController.getAll(req, res));

// FIX 2: Bind in constructor
class UserController {
  constructor() {
    this.getAll = this.getAll.bind(this);
  }
}

// FIX 3: Arrow class method (cleanest)
class UserController {
  getAll = async (req, res) => {
    const users = await this.userService.findAll();
    res.json(users);
  }
}
```

---

### Challenge 6: Async Error Handling Gotchas

**What:** Common mistakes in async error handling — unhandled rejections, missing try/catch, and the difference between thrown errors and rejected promises.
**Why:** Unhandled promise rejections crash Node.js 15+ processes; interviewers want to know you write production-safe async code.

```javascript
// GOTCHA 1: forEach doesn't await
async function processItems(items) {
  items.forEach(async (item) => {    // ✗ forEach ignores the returned promise
    await processItem(item);
  });
  console.log('Done!');  // Runs BEFORE any item is processed!
}

// FIX: Use for...of
async function processItems(items) {
  for (const item of items) {
    await processItem(item);
  }
  console.log('Done!');  // Runs AFTER all items processed ✓
}

// FIX (parallel): Use Promise.all with map
async function processItems(items) {
  await Promise.all(items.map(item => processItem(item)));
  console.log('Done!');  // Runs AFTER all items processed (in parallel) ✓
}
```

```javascript
// GOTCHA 2: try/catch doesn't catch without await
async function broken() {
  try {
    riskyAsyncFunction();          // ✗ No await! Promise floats unhandled
  } catch (error) {
    console.log('Never catches');   // This never runs
  }
}

async function fixed() {
  try {
    await riskyAsyncFunction();    // ✓ await lets try/catch work
  } catch (error) {
    console.log('Caught!', error); // This catches the rejection
  }
}
```

```javascript
// GOTCHA 3: Returning inside try/catch with finally
async function tricky() {
  try {
    return 'try';
  } finally {
    return 'finally';  // ⚠️ This OVERRIDES the try return!
  }
}
console.log(await tricky()); // "finally" — NOT "try"!
```

**Production safety net:**
```javascript
// Always add these in your entry file (app.js / server.js)
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
  // Log to Sentry/Winston, then gracefully shutdown
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  process.exit(1);  // MUST exit — state is corrupted
});
```

---

### Challenge 7: Deep Event Loop — Interleaving All Queue Types

**What:** A complex scenario mixing sync, nextTick, Promise, setTimeout, setImmediate, and I/O to test complete event loop understanding.
**Why:** This is a "final boss" level question — if you can trace this correctly, you can handle any event loop question thrown at you.

```javascript
const fs = require('fs');

console.log('1');

setTimeout(() => {
  console.log('2');
  Promise.resolve().then(() => console.log('3'));
}, 0);

setImmediate(() => {
  console.log('4');
  process.nextTick(() => console.log('5'));
});

fs.readFile(__filename, () => {
  console.log('6');
  setTimeout(() => console.log('7'), 0);
  setImmediate(() => console.log('8'));
  process.nextTick(() => console.log('9'));
});

Promise.resolve().then(() => console.log('10'));
process.nextTick(() => console.log('11'));

console.log('12');
```

**Step-by-step analysis:**

**Phase 1 — Sync (Rule 1):**
```
Output: 1, 12
Queues registered:
  Timer:     [() => log('2') + promise]
  Check:     [() => log('4') + nextTick]
  I/O:       [readFile callback]
  nextTick:  [() => log('11')]
  Promise:   [() => log('10')]
```

**Phase 2 — Microtasks (Rule 2 + Rule 3):**
```
nextTick first (Rule 3): 11
Promise next: 10
Output so far: 1, 12, 11, 10
```

**Phase 3 — Macrotasks (event loop phases):**
```
Timers phase:
  Output: 2
  (new Promise microtask registered)
  Drain microtasks → Output: 3
  Output so far: 1, 12, 11, 10, 2, 3

I/O Poll phase (when readFile completes):
  Output: 6
  (new timer, setImmediate, nextTick registered)
  Drain microtasks → nextTick → Output: 9
  Output so far: 1, 12, 11, 10, 2, 3, 6, 9

Check phase:
  Output: 4 (from original setImmediate)
  Drain microtasks → nextTick → Output: 5
  Output: 8 (from I/O callback's setImmediate)
  Output so far: 1, 12, 11, 10, 2, 3, 6, 9, 4, 5, 8

Next loop — Timers phase:
  Output: 7 (from I/O callback's setTimeout)
```

**Final output:**
```
1     ← Sync
12    ← Sync
11    ← nextTick (highest microtask)
10    ← Promise.then (second microtask)
2     ← setTimeout (timers phase)
3     ← Promise.then (microtask drain after timer)
6     ← I/O callback (poll phase — when file read completes)
9     ← nextTick (microtask drain after I/O)
4     ← setImmediate (check phase)
5     ← nextTick (microtask drain after setImmediate)
8     ← setImmediate from I/O (check phase)
7     ← setTimeout from I/O (next loop timers phase)
```

**Note:** The exact ordering of 2/4 and 6 may vary depending on when the file read completes. The I/O callback runs when the poll phase detects the file read is done.

---

### Challenge 8: Closure + Async = Interview Gold

**What:** Practical closure patterns used in real Node.js applications — rate limiters, memoization, and retry logic.
**Why:** Interviewers love questions that combine closures with async because it tests two concepts simultaneously and mirrors real production code.

```javascript
// Real-world: Rate limiter using closures
function createRateLimiter(maxRequests, windowMs) {
  let requests = 0;
  let windowStart = Date.now();

  return async function limiter(req, res, next) {
    const now = Date.now();
    if (now - windowStart > windowMs) {
      requests = 0;
      windowStart = now;
    }
    if (requests >= maxRequests) {
      return res.status(429).json({ error: 'Rate limit exceeded' });
    }
    requests++;  // Closure "remembers" requests count between calls
    next();
  };
}

// Usage in Express
app.use(createRateLimiter(100, 15 * 60 * 1000)); // 100 req per 15 min
```

```javascript
// Real-world: Memoize async function results
function memoizeAsync(fn) {
  const cache = new Map();  // Closed over — persists between calls

  return async function (...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);

    const result = await fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Usage
const getUser = memoizeAsync(async (id) => {
  const user = await db.users.findByPk(id);
  return user;
});

await getUser(1); // DB query
await getUser(1); // Cache hit — no DB query!
```

```javascript
// Real-world: Retry with exponential backoff
function withRetry(fn, maxRetries = 3) {
  let attempts = 0;  // Closed over

  return async function (...args) {
    while (attempts < maxRetries) {
      try {
        return await fn(...args);
      } catch (error) {
        attempts++;
        if (attempts >= maxRetries) throw error;
        const delay = Math.pow(2, attempts) * 1000; // 2s, 4s, 8s
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  };
}

// Usage at Obenan — retry Google API calls
const fetchReviews = withRetry(async (locationId) => {
  return await googleAPI.reviews.list({ locationId });
}, 3);
```

---

## 38.2 Node.js Design Principles in Practice

### The Module Pattern — Before & After

```javascript
// BEFORE: Global state pollution
let connection;
function connect(url) { connection = createConnection(url); }
function query(sql) { return connection.execute(sql); }

// AFTER: Encapsulated module (closure-based)
function createDatabase(url) {
  const connection = createConnection(url);  // Private

  return {
    query: (sql) => connection.execute(sql),       // Public
    close: () => connection.end(),                  // Public
    // connection is NOT exposed — data privacy via closure
  };
}

const db = createDatabase('postgres://localhost/mydb');
db.query('SELECT * FROM users');
db.connection; // undefined — properly encapsulated
```

### Event Emitter Pattern — Your BullMQ Workers

```javascript
const EventEmitter = require('events');

class ReviewProcessor extends EventEmitter {
  async process(review) {
    this.emit('start', review.id);

    try {
      const sentiment = await analyzeSentiment(review.text);
      this.emit('analyzed', { reviewId: review.id, sentiment });

      await saveResult(review.id, sentiment);
      this.emit('complete', review.id);
    } catch (error) {
      this.emit('error', { reviewId: review.id, error });
    }
  }
}

// Usage — decoupled listeners
const processor = new ReviewProcessor();
processor.on('start', (id) => logger.info(`Processing review ${id}`));
processor.on('analyzed', ({ sentiment }) => metrics.track(sentiment));
processor.on('error', ({ error }) => sentry.captureException(error));
processor.on('complete', (id) => logger.info(`Done: ${id}`));

await processor.process(review);
```

### Streams Pipeline — Processing Large Files

```javascript
const { pipeline } = require('stream/promises');
const fs = require('fs');
const { Transform } = require('stream');

// Transform stream — process each chunk
const csvToJson = new Transform({
  transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n');
    const jsonLines = lines
      .filter(line => line.trim())
      .map(line => {
        const [name, email, city] = line.split(',');
        return JSON.stringify({ name, email, city }) + '\n';
      })
      .join('');
    callback(null, jsonLines);
  }
});

// Pipeline — handles errors and backpressure automatically
await pipeline(
  fs.createReadStream('users.csv'),      // 2GB file
  csvToJson,                              // Transform chunk by chunk
  fs.createWriteStream('users.json')     // Write chunk by chunk
);
// Memory usage: ~64KB (chunk size) instead of 2GB!
```

---

## 38.3 Common Misconceptions & Gotchas

| Misconception | Reality |
|--------------|---------|
| `setTimeout(fn, 0)` runs immediately | It runs after ALL sync code and microtasks (Rule 4) |
| `async` functions always run async | Code BEFORE the first `await` runs synchronously (Rule 8) |
| `Promise.resolve()` is always async | `Promise.resolve(syncValue)` resolves sync, but `.then()` callback is always async |
| `forEach` works with `async/await` | `forEach` ignores returned promises — use `for...of` or `Promise.all(map)` |
| Node.js is single-threaded | JS execution is single-threaded; libuv uses a thread pool for I/O |
| `const` means immutable | `const` prevents reassignment; object properties can still change |
| Arrow functions are just shorter syntax | Arrow functions have NO `this`, NO `arguments`, NO `prototype`, can't be constructors |
| `typeof null === 'null'` | `typeof null === 'object'` — a 30-year-old JS bug |
| `0.1 + 0.2 === 0.3` | `false` — IEEE 754 floating point: `0.1 + 0.2 === 0.30000000000000004` |
| `[] == false` is false | `[] == false` is `true`! (type coercion: [] → "" → 0, false → 0, 0 == 0) |

---

## 38.4 Performance Comparison Patterns

### JSON.parse vs Structured Data

```javascript
// Comparing 1M iterations
const obj = { name: 'Saad', role: 'Senior Engineer', skills: ['Node', 'AWS'] };

// Spread clone — ~50ms for 1M ops
const clone1 = { ...obj };

// JSON round-trip — ~800ms for 1M ops (16x slower)
const clone2 = JSON.parse(JSON.stringify(obj));

// structuredClone — ~200ms for 1M ops (handles circular refs)
const clone3 = structuredClone(obj);
```

**When to use which:**
- Spread: Shallow clone, flat objects (fastest)
- structuredClone: Deep clone with dates, maps, sets, circular refs
- JSON round-trip: When you need a string anyway; loses functions, undefined, dates

### Map vs Object for Lookups

```javascript
// Object — slower for frequent add/delete
const objMap = {};
objMap['key1'] = 'value1';  // String keys only
delete objMap['key1'];       // Slow — deoptimizes V8 hidden classes

// Map — faster for frequent add/delete, any key type
const map = new Map();
map.set('key1', 'value1');   // Any type as key (objects, numbers, etc.)
map.delete('key1');           // Fast — designed for this

// Performance:
// 1M inserts: Object ~120ms, Map ~80ms
// 1M lookups: Object ~20ms, Map ~25ms (slightly slower)
// 1M deletes: Object ~300ms, Map ~50ms (6x faster)
```

**Rule of thumb:** Use `Map` when keys are dynamic or non-string. Use plain objects for static config/schemas.

---

## 38.5 Evaluation Platforms for Practice

### Coding Challenge Platforms

| Platform | Best For | Difficulty | JS/Node.js Support |
|----------|----------|-----------|-------------------|
| **LeetCode** | Algorithm problems, FAANG prep | Medium-Hard | Full JS/TS support |
| **HackerRank** | Skill assessments, certificates | Easy-Hard | JS + Node.js challenges |
| **CodeSignal** | Company-specific assessments | Medium | JS coding challenges |
| **Codewars** | Kata-style practice, community | Easy-Hard | Excellent JS support |
| **InterviewBit** | Structured interview prep | Medium | JS interview questions |
| **Edabit** | Beginner-friendly challenges | Easy-Medium | JS challenges by difficulty |
| **JSchallenger** | JavaScript-specific practice | Easy-Medium | Pure JS focus |

### Project-Based Practice

| Platform | Best For |
|----------|----------|
| **Frontend Mentor** | Full-stack JavaScript projects with real designs |
| **Dev.to** | Community challenges and code reviews |
| **GitHub** | Open source Node.js projects for real-world experience |
| **Exercism** | Mentored JavaScript/TypeScript tracks |

### Recommended Practice Path

```
Week 1-2: JSchallenger + Edabit (warm up fundamentals)
    ↓
Week 3-4: Codewars (6 kyu → 4 kyu katas)
    ↓
Week 5-6: LeetCode Easy → Medium (focus on arrays, strings, hashmaps)
    ↓
Week 7-8: HackerRank Node.js challenges + InterviewBit
    ↓
Ongoing: GitHub open source + Frontend Mentor projects
```

### What to Practice on Each Platform

**LeetCode — Focus on these patterns:**
- Two Pointers (string reversal, palindrome)
- Hash Map (frequency count, two sum)
- Stack/Queue (valid parentheses, BFS)
- Sliding Window (max subarray)
- Recursion + Memoization (Fibonacci, grid paths)

**Codewars — Focus on these kata:**
- Array manipulation (flatten, chunk, zip)
- String parsing (decode morse, roman numerals)
- Functional programming (compose, curry, pipe)
- Async patterns (promise sequencing, retry logic)

**HackerRank — Take these assessments:**
- JavaScript (Intermediate) Certificate
- Node.js (Intermediate) Certificate
- Problem Solving (Basic) Certificate

---

## 38.6 Quick Self-Test — Predict the Output

Try predicting BEFORE reading the answers. Apply the 8 Golden Rules.

### Test 1
```javascript
console.log(typeof typeof 1);
```
<details>
<summary>Answer</summary>

`"string"` — `typeof 1` returns the string `"number"`, then `typeof "number"` returns `"string"`.
</details>

### Test 2
```javascript
console.log(0.1 + 0.2 === 0.3);
console.log(0.1 + 0.2);
```
<details>
<summary>Answer</summary>

`false` then `0.30000000000000004` — IEEE 754 floating point precision issue.
Fix: `Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON`
</details>

### Test 3
```javascript
const a = {};
const b = { key: 'b' };
const c = { key: 'c' };

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```
<details>
<summary>Answer</summary>

`456` — Object keys are strings. `b.toString()` and `c.toString()` both return `"[object Object]"`, so `a[b]` and `a[c]` write to the same key.
</details>

### Test 4
```javascript
async function foo() {
  console.log('1');
  await bar();
  console.log('2');
}

async function bar() {
  console.log('3');
}

console.log('A');
foo();
console.log('B');
```
<details>
<summary>Answer</summary>

```
A  ← Sync
1  ← Sync (foo runs, before await)  — Rule 8
3  ← Sync (bar runs entirely, before await returns) — Rule 8
B  ← Sync (foo paused at await, control returns to caller) — Rule 8
2  ← Microtask (await continuation) — Rule 2
```
</details>

### Test 5
```javascript
console.log([] + []);
console.log([] + {});
console.log({} + []);
```
<details>
<summary>Answer</summary>

```
""              ← Both arrays become "", "" + "" = ""
"[object Object]" ← [] becomes "", {} becomes "[object Object]"
"[object Object]" ← In expression context, {} is object, [] becomes ""
```
Note: In some consoles, `{} + []` may output `0` because `{}` is parsed as an empty block, not an object.
</details>

### Test 6
```javascript
let x = 1;
const fn = () => {
  console.log(x);
  let x = 2;
};
fn();
```
<details>
<summary>Answer</summary>

`ReferenceError: Cannot access 'x' before initialization` — The inner `let x` creates a new binding in the block scope. Due to hoisting, `x` exists but is in the Temporal Dead Zone until the `let x = 2` line executes. The outer `x = 1` is NOT accessible because the inner `x` shadows it.
</details>

### Test 7
```javascript
Promise.resolve(1)
  .then(x => x + 1)
  .then(x => { throw new Error(x) })
  .then(x => console.log('then:', x))
  .catch(x => console.log('catch:', x.message))
  .then(x => console.log('after catch:', x));
```
<details>
<summary>Answer</summary>

```
catch: 2
after catch: undefined
```
- `.then(x => x + 1)` → resolves with 2
- `.then(x => { throw new Error(x) })` → rejects with Error("2")
- `.then(x => ...)` → SKIPPED (rejected promises skip .then)
- `.catch(x => ...)` → catches, prints "catch: 2", returns undefined
- `.then(x => ...)` → runs because catch returned (no throw), x = undefined
</details>
