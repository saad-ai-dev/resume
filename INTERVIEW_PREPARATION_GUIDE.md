# COMPLETE INTERVIEW PREPARATION GUIDE
## Saad Sohail — Senior Full-Stack Engineer | AI & LLM Integration
### PhD-Level Deep Dive for Every Resume Topic

---

# TABLE OF CONTENTS

1. [JavaScript Deep Dive](#1-javascript-deep-dive)
2. [TypeScript Deep Dive](#2-typescript-deep-dive)
3. [JavaScript vs TypeScript — Complete Comparison](#3-javascript-vs-typescript--complete-comparison)
4. [Node.js — Runtime & Architecture](#4-nodejs--runtime--architecture)
5. [Express.js](#5-expressjs)
6. [NestJS](#6-nestjs)
7. [Express.js vs NestJS — Comparison](#7-expressjs-vs-nestjs--comparison)
8. [REST APIs](#8-rest-apis)
9. [GraphQL](#9-graphql)
10. [REST vs GraphQL — Comparison](#10-rest-vs-graphql--comparison)
11. [SQL Databases (PostgreSQL, MySQL)](#11-sql-databases-postgresql-mysql)
12. [NoSQL Databases (MongoDB)](#12-nosql-databases-mongodb)
13. [SQL vs NoSQL — Complete Comparison](#13-sql-vs-nosql--complete-comparison)
14. [PostgreSQL vs MySQL — Comparison](#14-postgresql-vs-mysql--comparison)
15. [Redis](#15-redis)
16. [Sequelize ORM](#16-sequelize-orm)
17. [Authentication & Security (JWT, Passport.js, OAuth, RBAC)](#17-authentication--security)
18. [Socket.IO — Real-Time Communication](#18-socketio--real-time-communication)
19. [BullMQ — Job Queues & Background Processing](#19-bullmq--job-queues--background-processing)
20. [AWS Cloud Services](#20-aws-cloud-services)
21. [Docker](#21-docker)
22. [Nginx](#22-nginx)
23. [CI/CD (CodePipeline, GitHub Actions, Elastic Beanstalk)](#23-cicd)
24. [AI & LLMs (OpenAI, Claude, Gemini)](#24-ai--llms)
25. [RAG Architecture](#25-rag-architecture)
26. [Fine-Tuning LLMs](#26-fine-tuning-llms)
27. [Prompt Engineering](#27-prompt-engineering)
28. [Stripe Payment Integration](#28-stripe-payment-integration)
29. [SendGrid Email Integration](#29-sendgrid-email-integration)
30. [Testing (Mocha, Chai)](#30-testing-mocha-chai)
31. [Git, GitHub & Branching Strategies](#31-git-github--branching-strategies)
32. [Monitoring (Sentry, CodeClimate)](#32-monitoring-sentry-codeclimate)
33. [System Design Patterns](#33-system-design-patterns)
34. [Python Basics](#34-python-basics)
35. [Behavioral & Soft Skill Questions](#35-behavioral--soft-skill-questions)
36. [Node.js Design Patterns](#36-nodejs-design-patterns)

---

# 1. JAVASCRIPT DEEP DIVE

## 1.1 What is JavaScript?
JavaScript is a **single-threaded, non-blocking, asynchronous, interpreted** programming language. It was created by Brendan Eich in 1995 at Netscape. Originally designed for browsers, it now runs everywhere — browsers, servers (Node.js), mobile (React Native), desktop (Electron).

## 1.2 Core Concepts

### Event Loop
The event loop is the heart of JavaScript's asynchronous behavior.

```
┌───────────────────────────┐
│        Call Stack          │  ← Executes synchronous code
└─────────┬─────────────────┘
          │
┌─────────▼─────────────────┐
│      Web APIs / Node APIs  │  ← setTimeout, fetch, I/O
└─────────┬─────────────────┘
          │
┌─────────▼─────────────────┐
│    Callback Queue          │  ← Macrotasks (setTimeout, setInterval)
│    Microtask Queue         │  ← Microtasks (Promises, queueMicrotask)
└─────────┬─────────────────┘
          │
┌─────────▼─────────────────┐
│      Event Loop            │  ← Checks: Stack empty? → Pick next task
└───────────────────────────┘
```

**Key Rule:** Microtasks (Promises) ALWAYS execute before Macrotasks (setTimeout).

```javascript
console.log('1');                          // Sync → runs first
setTimeout(() => console.log('2'), 0);     // Macrotask → runs last
Promise.resolve().then(() => console.log('3')); // Microtask → runs second
// Output: 1, 3, 2
```

### Closures
A closure is a function that remembers the variables from its outer scope even after the outer function has returned.

```javascript
function createCounter() {
  let count = 0; // This variable is "closed over"
  return {
    increment: () => ++count,
    getCount: () => count
  };
}
const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.getCount();  // 2
```

**Interview Q: Why are closures useful?**
- Data privacy (encapsulation without classes)
- Function factories
- Callbacks that remember state
- Module pattern

### Hoisting
JavaScript moves **declarations** (not initializations) to the top of their scope before execution.

```javascript
console.log(a); // undefined (var is hoisted, but value isn't)
var a = 5;

console.log(b); // ReferenceError (let/const are in "Temporal Dead Zone")
let b = 10;

greet(); // Works! Function declarations are fully hoisted
function greet() { console.log('Hello'); }

greet2(); // TypeError: greet2 is not a function
var greet2 = function() { console.log('Hello'); };
```

### var vs let vs const

| Feature | `var` | `let` | `const` |
|---------|-------|-------|---------|
| Scope | Function | Block | Block |
| Hoisted | Yes (undefined) | Yes (TDZ) | Yes (TDZ) |
| Re-declare | Yes | No | No |
| Re-assign | Yes | Yes | No |
| Use case | Legacy code | Mutable variables | Constants, objects |

**Rule:** Always use `const` by default. Use `let` only when you need to reassign. Never use `var`.

### Prototypal Inheritance
JavaScript doesn't have classical inheritance — it uses **prototypes**.

```javascript
const animal = {
  speak() { return `${this.name} makes a sound`; }
};

const dog = Object.create(animal);
dog.name = 'Rex';
dog.speak(); // "Rex makes a sound"

// The prototype chain:
// dog → animal → Object.prototype → null
```

### `this` Keyword

| Context | `this` refers to |
|---------|-----------------|
| Global | `window` (browser) or `global` (Node.js) |
| Object method | The object that owns the method |
| Arrow function | Inherits `this` from parent scope (lexical) |
| `new` keyword | The newly created object |
| `call/apply/bind` | Whatever you explicitly pass |
| Event handler | The DOM element that triggered the event |

```javascript
const obj = {
  name: 'Saad',
  greet: function() { return this.name; },        // "Saad" ✓
  greetArrow: () => { return this.name; }          // undefined ✗ (inherits outer this)
};
```

### Promises & Async/Await

```javascript
// Promise
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    db.find(id, (err, user) => {
      if (err) reject(err);
      else resolve(user);
    });
  });
}

// Async/Await (syntactic sugar over Promises)
async function getUser(id) {
  try {
    const user = await fetchUser(id);
    return user;
  } catch (error) {
    console.error('Failed:', error);
  }
}

// Parallel execution
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts()
]);

// Promise methods
Promise.all([p1, p2])      // Resolves when ALL resolve, rejects if ANY rejects
Promise.allSettled([p1,p2]) // Waits for ALL to settle (resolve or reject)
Promise.race([p1, p2])     // Resolves/rejects with the FIRST to settle
Promise.any([p1, p2])      // Resolves with the FIRST to resolve (ignores rejections)
```

### Destructuring, Spread & Rest

```javascript
// Object destructuring
const { name, age, city = 'Lahore' } = user;

// Array destructuring
const [first, , third] = [1, 2, 3]; // first=1, third=3

// Spread (expand)
const merged = { ...defaults, ...overrides };
const combined = [...arr1, ...arr2];

// Rest (collect)
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
```

### Map, Filter, Reduce

```javascript
const users = [
  { name: 'Ali', age: 25, active: true },
  { name: 'Sara', age: 30, active: false },
  { name: 'Saad', age: 28, active: true }
];

// Map — transform each element
const names = users.map(u => u.name); // ['Ali', 'Sara', 'Saad']

// Filter — keep elements that pass a test
const active = users.filter(u => u.active); // [Ali, Saad]

// Reduce — accumulate into a single value
const totalAge = users.reduce((sum, u) => sum + u.age, 0); // 83

// Chaining
const activeNames = users
  .filter(u => u.active)
  .map(u => u.name); // ['Ali', 'Saad']
```

### WeakMap & WeakSet
```javascript
// WeakMap — keys must be objects, garbage collected when no other reference
const cache = new WeakMap();
let obj = { data: 'heavy' };
cache.set(obj, 'cached result');
obj = null; // The entry in WeakMap is now garbage collected

// Use case: storing metadata about objects without memory leaks
```

### Event Delegation
```javascript
// Instead of adding listeners to each button...
document.getElementById('parent').addEventListener('click', (e) => {
  if (e.target.matches('.btn')) {
    handleClick(e.target);
  }
});
// One listener on the parent handles all child clicks
```

## 1.3 Common JavaScript Interview Questions

**Q: What is the difference between == and ===?**
- `==` (loose equality) — converts types before comparing: `"5" == 5` → true
- `===` (strict equality) — no type conversion: `"5" === 5` → false
- **Always use ===**

**Q: What is a pure function?**
A function that: (1) always returns the same output for the same input, (2) has no side effects (doesn't modify external state).

**Q: What is debouncing vs throttling?**
- **Debounce:** Wait until the user STOPS doing something, then execute. (Search input — wait until user stops typing)
- **Throttle:** Execute at most once every X milliseconds. (Scroll events — fire handler every 200ms max)

**Q: What is the difference between null and undefined?**
- `undefined` — variable declared but not assigned; function parameter not passed
- `null` — explicitly assigned to represent "no value"
- `typeof undefined` → "undefined"; `typeof null` → "object" (JS bug since 1995)

**Q: Explain call(), apply(), bind()**
```javascript
function greet(greeting) { return `${greeting}, ${this.name}`; }
const user = { name: 'Saad' };

greet.call(user, 'Hello');    // "Hello, Saad" — call with args
greet.apply(user, ['Hello']); // "Hello, Saad" — call with array of args
const bound = greet.bind(user); // Returns new function with `this` bound
bound('Hello');               // "Hello, Saad"
```

**Q: What is optional chaining and nullish coalescing?**
```javascript
const city = user?.address?.city;       // undefined if any part is null/undefined
const name = user.name ?? 'Anonymous';  // 'Anonymous' only if null or undefined (not 0 or '')
```

---

# 2. TYPESCRIPT DEEP DIVE

## 2.1 What is TypeScript?
TypeScript is a **statically-typed superset of JavaScript** developed by Microsoft. It compiles to plain JavaScript and adds type safety, interfaces, generics, enums, and better tooling.

## 2.2 Core Concepts

### Basic Types
```typescript
// Primitives
let name: string = 'Saad';
let age: number = 28;
let active: boolean = true;

// Arrays
let ids: number[] = [1, 2, 3];
let names: Array<string> = ['a', 'b'];

// Tuples
let pair: [string, number] = ['Saad', 28];

// Enums
enum Status { Active = 'ACTIVE', Inactive = 'INACTIVE' }
let s: Status = Status.Active;

// Any vs Unknown
let a: any = 5;      // Disables type checking — AVOID
let u: unknown = 5;  // Safe — must type-check before using

// Void, Never
function log(msg: string): void { console.log(msg); }
function throwError(msg: string): never { throw new Error(msg); }
```

### Interfaces vs Types

```typescript
// Interface — describes object shape, extendable
interface User {
  id: number;
  name: string;
  email?: string;         // optional
  readonly createdAt: Date; // cannot be modified
}

// Extending interfaces
interface Admin extends User {
  role: 'admin' | 'superadmin';
  permissions: string[];
}

// Type alias — more flexible, can represent unions, tuples, primitives
type ID = string | number;
type Status = 'active' | 'inactive' | 'banned';
type Pair = [string, number];

// Intersection types
type AdminUser = User & { role: string };
```

| Feature | `interface` | `type` |
|---------|------------|--------|
| Object shape | Yes | Yes |
| Extend/inherit | Yes (extends) | Yes (& intersection) |
| Union types | No | Yes |
| Tuple types | No | Yes |
| Declaration merging | Yes | No |
| Use case | Object contracts, APIs | Complex types, unions |

### Generics
Generics let you write reusable code that works with multiple types.

```typescript
// Generic function
function getFirst<T>(arr: T[]): T {
  return arr[0];
}
getFirst<number>([1, 2, 3]);  // 1
getFirst<string>(['a', 'b']); // 'a'

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

type UserResponse = ApiResponse<User>;
type PostResponse = ApiResponse<Post[]>;

// Generic constraints
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Generic class
class Repository<T extends { id: number }> {
  private items: T[] = [];

  add(item: T): void { this.items.push(item); }
  findById(id: number): T | undefined {
    return this.items.find(item => item.id === id);
  }
}
```

### Utility Types
```typescript
// Partial — all properties become optional
type UpdateUser = Partial<User>;

// Required — all properties become required
type FullUser = Required<User>;

// Pick — select specific properties
type UserPreview = Pick<User, 'id' | 'name'>;

// Omit — exclude specific properties
type UserWithoutEmail = Omit<User, 'email'>;

// Record — define key-value pairs
type UserMap = Record<string, User>;

// Readonly — all properties become readonly
type FrozenUser = Readonly<User>;

// ReturnType — extract function return type
type Result = ReturnType<typeof fetchUser>;

// Exclude / Extract — for union types
type NonNull = Exclude<string | null | undefined, null | undefined>; // string
```

### Type Guards
```typescript
// typeof guard
function process(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase(); // TypeScript knows it's string here
  }
  return value.toFixed(2); // TypeScript knows it's number here
}

// instanceof guard
function handleError(error: Error | string) {
  if (error instanceof Error) {
    return error.message;
  }
  return error;
}

// Custom type guard
function isUser(obj: any): obj is User {
  return obj && typeof obj.name === 'string' && typeof obj.id === 'number';
}
```

### Decorators (Used Heavily in NestJS)
```typescript
// Class decorator
function Logger(target: Function) {
  console.log(`Class ${target.name} created`);
}

@Logger
class UserService { }

// Method decorator
function Log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${key} with`, args);
    return original.apply(this, args);
  };
}

class Service {
  @Log
  getData(id: number) { /* ... */ }
}
```

---

# 3. JAVASCRIPT VS TYPESCRIPT — COMPLETE COMPARISON

| Feature | JavaScript | TypeScript |
|---------|-----------|------------|
| **Typing** | Dynamic (runtime) | Static (compile-time) |
| **Compilation** | Interpreted directly | Compiles to JavaScript |
| **Error detection** | At runtime | At compile time |
| **Interfaces** | No | Yes |
| **Generics** | No | Yes |
| **Enums** | No | Yes |
| **Decorators** | No (proposal) | Yes |
| **Tooling/IDE** | Basic autocomplete | Rich autocomplete, refactoring |
| **Learning curve** | Lower | Higher |
| **Ecosystem** | All npm packages | Most have @types/* |
| **Build step** | Optional | Required (tsc) |
| **Best for** | Scripts, prototypes | Large apps, teams, APIs |

**Interview Q: When would you choose TypeScript over JavaScript?**
- Team projects (multiple developers)
- Large codebases (50+ files)
- API contracts (backend services)
- When refactoring safety matters
- Production applications

**Interview Q: When is JavaScript fine?**
- Quick prototypes
- Small scripts
- Legacy projects without TypeScript setup

---

# 4. NODE.JS — RUNTIME & ARCHITECTURE

## 4.1 What is Node.js?
Node.js is a **JavaScript runtime built on Chrome's V8 engine**. It allows running JavaScript on the server. Created by Ryan Dahl in 2009.

## 4.2 Architecture

```
┌─────────────────────────────────────┐
│           Your Node.js Code          │
├─────────────────────────────────────┤
│         Node.js Bindings (C++)       │
├──────────────┬──────────────────────┤
│    V8 Engine │    libuv             │
│  (JS → Machine│  (Event Loop,       │
│    Code)     │   Thread Pool,       │
│              │   Async I/O)         │
├──────────────┴──────────────────────┤
│         Operating System             │
└─────────────────────────────────────┘
```

**Key Points:**
- **Single-threaded** for JavaScript execution
- **Non-blocking I/O** via libuv's event loop
- **Thread pool** (default 4 threads) handles heavy I/O (file system, DNS, crypto)
- **V8** compiles JS to machine code (JIT compilation)

## 4.3 Event Loop in Node.js (Detailed)

```
   ┌───────────────────────────┐
┌─>│         Timers             │  ← setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │     Pending Callbacks      │  ← I/O callbacks deferred to next loop
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │       Idle, Prepare        │  ← Internal use only
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │          Poll              │  ← Retrieve new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │          Check             │  ← setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────▼─────────────┐
│  │      Close Callbacks       │  ← socket.on('close')
│  └─────────────┬─────────────┘
│                 │
└─────────────────┘  (loop back)

Between each phase: process all microtasks (Promises, process.nextTick)
```

## 4.4 Key Node.js Concepts

### Streams
```javascript
const fs = require('fs');

// Reading a 2GB file
// BAD — loads entire file into memory
const data = fs.readFileSync('huge.csv');

// GOOD — streams it chunk by chunk
const stream = fs.createReadStream('huge.csv');
stream.on('data', (chunk) => { /* process chunk */ });
stream.on('end', () => { /* done */ });

// Types of streams:
// Readable  — fs.createReadStream, HTTP request
// Writable  — fs.createWriteStream, HTTP response
// Duplex    — TCP socket (read + write)
// Transform — zlib.createGzip (modify data as it passes through)
```

### Cluster Module
```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  // Fork one worker per CPU core
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  // Each worker runs the server
  const app = require('./app');
  app.listen(3000);
}
```

### Worker Threads
```javascript
const { Worker } = require('worker_threads');

// Offload CPU-intensive work to a separate thread
const worker = new Worker('./heavy-computation.js', {
  workerData: { numbers: [1, 2, 3, 4, 5] }
});

worker.on('message', (result) => {
  console.log('Result:', result);
});
```

### Buffer
```javascript
// Buffers handle binary data (images, files, network packets)
const buf = Buffer.from('Hello', 'utf-8');
console.log(buf);         // <Buffer 48 65 6c 6c 6f>
console.log(buf.toString()); // 'Hello'
```

## 4.5 Common Node.js Interview Questions

**Q: Is Node.js single-threaded?**
Yes for JavaScript execution. But libuv's thread pool handles I/O operations in background threads. So Node.js is single-threaded for your code, but multi-threaded under the hood for I/O.

**Q: What is the difference between process.nextTick() and setImmediate()?**
- `process.nextTick()` — executes BEFORE the event loop continues (before any I/O)
- `setImmediate()` — executes in the "check" phase of the event loop (after I/O)
- `process.nextTick` has higher priority

**Q: How do you handle uncaught exceptions?**
```javascript
process.on('uncaughtException', (err) => {
  console.error('Uncaught:', err);
  process.exit(1); // MUST exit — state is corrupted
});

process.on('unhandledRejection', (reason) => {
  console.error('Unhandled Promise:', reason);
});
```

**Q: What is middleware in Node.js?**
A function that sits between the request and response, processing the request before it reaches the route handler. Middleware can modify req/res, end the request, or call `next()` to pass to the next middleware.

---

## 4.6 Event Loop Deep Dive — Complete Walkthrough

This is the **#1 most asked Node.js interview question**. Let's break it down so you can explain it to anyone.

### How Node.js Executes Code — The Full Picture

When Node.js runs your file, it goes through these phases in order:

```
┌─────────────────────────────────────────────────────────┐
│  PHASE 1: SYNCHRONOUS CODE                               │
│  ─────────────────────────                               │
│  Node.js reads your file TOP to BOTTOM.                  │
│  Everything that is NOT a callback runs immediately      │
│  on the Call Stack. This includes:                        │
│    • console.log()                                        │
│    • variable declarations                                │
│    • function definitions                                 │
│    • Promise EXECUTORS (the function inside new Promise)  │
│    • Arguments to functions (evaluated before the call)   │
│                                                           │
│  During this phase, async callbacks are REGISTERED        │
│  (placed in their respective queues) but NOT executed.    │
└───────────────────────────┬─────────────────────────────┘
                            │ Call Stack is now empty
                            ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 2: MICROTASK QUEUES (run between EVERY phase)     │
│  ──────────────────────────                               │
│  Two sub-queues, in this exact priority order:            │
│                                                           │
│  2a. process.nextTick() queue  ← HIGHEST priority         │
│      Runs ALL nextTick callbacks first.                    │
│      If a nextTick adds another nextTick, that runs too   │
│      before moving to Promises.                            │
│                                                           │
│  2b. Promise microtask queue   ← Second priority           │
│      Runs ALL resolved Promise .then()/.catch() callbacks. │
│      Includes: Promise.resolve().then(),                   │
│      async/await continuations, queueMicrotask()           │
└───────────────────────────┬─────────────────────────────┘
                            │ All microtasks drained
                            ▼
┌─────────────────────────────────────────────────────────┐
│  PHASE 3: MACROTASK QUEUES (Event Loop phases)            │
│  ──────────────────────────                               │
│  The event loop now processes one macrotask at a time:     │
│                                                           │
│  3a. Timers phase        → setTimeout, setInterval         │
│      ↓ (drain microtasks again)                            │
│  3b. Pending I/O phase   → completed I/O callbacks         │
│      ↓ (drain microtasks again)                            │
│  3c. Poll phase          → new I/O events                  │
│      ↓ (drain microtasks again)                            │
│  3d. Check phase         → setImmediate                    │
│      ↓ (drain microtasks again)                            │
│  3e. Close phase         → socket.on('close')              │
│                                                           │
│  After each macrotask: ALL microtasks are drained again.  │
│  Then the loop repeats.                                    │
└─────────────────────────────────────────────────────────┘
```

### The Priority Ladder (memorize this)

```
HIGHEST ──► Synchronous code (Call Stack)
         ──► process.nextTick()
         ──► Promise .then() / .catch() / .finally()
         ──► setTimeout(fn, 0) / setInterval
LOWEST  ──► setImmediate()
```

**Rule:** Between EVERY phase of the event loop, Node.js completely drains ALL microtasks (nextTick first, then Promises) before moving to the next macrotask.

---

### Practical Example — Line by Line Execution

```javascript
console.log("1");

console.log("2");

process.nextTick(() => {
  console.log("process defined");
});

setTimeout(function () {
  console.log("3");
}, 0);

var promise = new Promise((resolve, reject) => {
  resolve("Promises-resolved");
});

promise.then(function (successMessage) {
  console.log(successMessage);
},
function (errorMessage) {
  console.log(errorMessage);
});

Promise.resolve(console.log("exception"));

console.log("4");
```

### Step-by-Step Execution

#### PHASE 1: Synchronous Execution (Call Stack)

Node reads the file top to bottom. Everything synchronous runs immediately.

```
STEP 1: console.log("1")
├── Type: Synchronous
├── Where: Call Stack (runs immediately)
├── Action: Prints to console
└── Output: "1"

STEP 2: console.log("2")
├── Type: Synchronous
├── Where: Call Stack (runs immediately)
├── Action: Prints to console
└── Output: "2"

STEP 3: process.nextTick(() => { console.log("process defined") })
├── Type: Asynchronous registration
├── Where: The callback is REGISTERED in the nextTick queue
├── Action: Does NOT execute the callback now
├── Output: (nothing yet)
└── Queue state:
    nextTick queue: [() => console.log("process defined")]
    Promise queue:  []
    Timer queue:    []

STEP 4: setTimeout(function() { console.log("3") }, 0)
├── Type: Asynchronous registration
├── Where: The callback is REGISTERED in the Timer (macrotask) queue
├── Action: Does NOT execute the callback now
│   Note: 0ms doesn't mean "run now" — it means "run as soon as
│         possible AFTER all sync code and microtasks are done"
├── Output: (nothing yet)
└── Queue state:
    nextTick queue: [() => console.log("process defined")]
    Promise queue:  []
    Timer queue:    [() => console.log("3")]

STEP 5: var promise = new Promise((resolve, reject) => {
          resolve("Promises-resolved");
        })
├── Type: SYNCHRONOUS! (this is the KEY trick)
├── Where: Call Stack
├── Action: The Promise EXECUTOR function runs IMMEDIATELY
│   ├── resolve("Promises-resolved") is called synchronously
│   └── This SCHEDULES the .then() callback as a microtask
│       (but .then() hasn't been attached yet at this point)
├── Output: (nothing — resolve doesn't print anything)
└── The promise is now in "fulfilled" state with value "Promises-resolved"

    WHY IS THE EXECUTOR SYNCHRONOUS?
    ─────────────────────────────────
    new Promise((resolve, reject) => {
      // THIS CODE runs on the Call Stack, just like console.log()
      // Only the .then()/.catch() CALLBACKS are async
    })

STEP 6: promise.then(function(successMessage) {
          console.log(successMessage);
        })
├── Type: Asynchronous registration
├── Where: Since the promise is ALREADY resolved, the .then()
│          callback is immediately SCHEDULED in the Promise microtask queue
├── Action: Does NOT execute the callback now
├── Output: (nothing yet)
└── Queue state:
    nextTick queue: [() => console.log("process defined")]
    Promise queue:  [() => console.log("Promises-resolved")]
    Timer queue:    [() => console.log("3")]

STEP 7: Promise.resolve(console.log("exception"))
├── Type: SYNCHRONOUS! (another KEY trick)
├── Where: Call Stack
├── Action: console.log("exception") is an ARGUMENT to Promise.resolve()
│   ├── In JavaScript, arguments are evaluated BEFORE the function is called
│   ├── So console.log("exception") runs FIRST (synchronous)
│   ├── It prints "exception" and returns undefined
│   └── Then Promise.resolve(undefined) creates a resolved promise
│       (but nobody calls .then() on it, so nothing is queued)
├── Output: "exception"
│
│   DETAILED BREAKDOWN:
│   ─────────────────
│   Promise.resolve(console.log("exception"))
│                   ^^^^^^^^^^^^^^^^^^^^^^^^
│                   This is evaluated FIRST!
│                   It's just a function call argument.
│
│   Equivalent to:
│   const result = console.log("exception");  // prints "exception", returns undefined
│   Promise.resolve(result);                  // Promise.resolve(undefined) — no .then()
│
└── Queue state: (unchanged)
    nextTick queue: [() => console.log("process defined")]
    Promise queue:  [() => console.log("Promises-resolved")]
    Timer queue:    [() => console.log("3")]

STEP 8: console.log("4")
├── Type: Synchronous
├── Where: Call Stack (runs immediately)
├── Action: Prints to console
└── Output: "4"
```

**After Phase 1, the output so far is:**
```
1
2
exception
4
```

**The Call Stack is now EMPTY. Node.js checks the microtask queues.**

---

#### PHASE 2: Microtask Queues

##### Phase 2a: process.nextTick() queue (highest priority)

```
STEP 9: Execute nextTick callback
├── Callback: () => console.log("process defined")
├── Where: Call Stack (pulled from nextTick queue)
├── Action: Prints to console
└── Output: "process defined"

nextTick queue is now EMPTY → move to Promise queue
```

##### Phase 2b: Promise microtask queue

```
STEP 10: Execute Promise .then() callback
├── Callback: (successMessage) => console.log(successMessage)
├── Argument: successMessage = "Promises-resolved"
├── Where: Call Stack (pulled from Promise queue)
├── Action: Prints to console
└── Output: "Promises-resolved"

Promise queue is now EMPTY → move to macrotask queues
```

**After Phase 2, the output so far is:**
```
1
2
exception
4
process defined
Promises-resolved
```

---

#### PHASE 3: Macrotask Queue (Timers phase)

```
STEP 11: Execute setTimeout callback
├── Callback: () => console.log("3")
├── Where: Call Stack (pulled from Timer queue)
├── Action: Prints to console
└── Output: "3"

Timer queue is now EMPTY → event loop checks for more work → none → EXIT
```

---

### FINAL OUTPUT

```
1                    ← Step 1:  Sync (Call Stack)
2                    ← Step 2:  Sync (Call Stack)
exception            ← Step 7:  Sync (argument evaluated immediately)
4                    ← Step 8:  Sync (Call Stack)
process defined      ← Step 9:  Microtask (process.nextTick — highest async priority)
Promises-resolved    ← Step 10: Microtask (Promise .then — second async priority)
3                    ← Step 11: Macrotask (setTimeout — lowest priority)
```

### Visual Timeline

```
TIME ───────────────────────────────────────────────────────────►

Call Stack:   [log "1"] [log "2"] [register] [register] [Promise executor] [register] [log "exception"] [log "4"]
                 │         │                                                               │               │
                 ▼         ▼                                                               ▼               ▼
Output:        "1"       "2"                                                          "exception"         "4"

── Call Stack empty ──

nextTick:     [log "process defined"]
                      │
                      ▼
Output:         "process defined"

Promise:      [log "Promises-resolved"]
                      │
                      ▼
Output:         "Promises-resolved"

Timer:        [log "3"]
                 │
                 ▼
Output:         "3"
```

---

### More Tricky Examples

#### Example 2: Nested Microtasks

```javascript
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve()
  .then(() => {
    console.log('promise 1');
    process.nextTick(() => console.log('nextTick inside promise'));
  })
  .then(() => console.log('promise 2'));

process.nextTick(() => {
  console.log('nextTick 1');
  Promise.resolve().then(() => console.log('promise inside nextTick'));
});

console.log('end');
```

**Output:**
```
start                      ← Sync
end                        ← Sync
nextTick 1                 ← nextTick queue (highest microtask priority)
promise inside nextTick    ← Promise created inside nextTick runs in same microtask drain
promise 1                  ← Promise queue
nextTick inside promise    ← nextTick registered inside promise — runs before next promise
promise 2                  ← Chained .then() runs after nextTick is drained
timeout                    ← Macrotask (setTimeout) runs LAST
```

**Key insight:** After EACH microtask executes, Node checks if new nextTick callbacks were added. nextTick ALWAYS gets priority over Promise .then() — even if the nextTick was registered inside a Promise.

#### Example 3: setTimeout vs setImmediate

```javascript
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

**Output:** UNPREDICTABLE! Could be either order.
- If the event loop enters the Timer phase before 1ms has passed → `immediate` first
- If 1ms has passed → `timeout` first

```javascript
// But inside an I/O callback, setImmediate ALWAYS runs first:
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// Output: ALWAYS "immediate" then "timeout"
// Because after I/O, the Check phase (setImmediate) comes before Timers
```

#### Example 4: async/await is just Promises

```javascript
async function main() {
  console.log('1');

  const result = await Promise.resolve('2');
  console.log(result);

  console.log('3');
}

console.log('A');
main();
console.log('B');
```

**Output:**
```
A     ← Sync
1     ← Sync (inside main, before await)
B     ← Sync (main() paused at await, execution continues outside)
2     ← Microtask (await resolved, continuation runs as Promise .then())
3     ← Sync (after await, but inside the microtask continuation)
```

**Key insight:** `await` pauses the async function and returns control to the caller. Everything after `await` runs as a microtask (like `.then()`).

---

### Cheat Sheet — What Goes Where?

| Code | Queue | Priority |
|------|-------|----------|
| `console.log()` | Call Stack (sync) | Immediate — runs NOW |
| `new Promise(executor)` | Call Stack (sync) | Immediate — executor runs NOW |
| `Promise.resolve(arg)` | arg on Call Stack (sync) | arg evaluated NOW |
| `process.nextTick(cb)` | nextTick microtask queue | Highest async priority |
| `.then(cb)` / `.catch(cb)` | Promise microtask queue | After all nextTicks |
| `await` continuation | Promise microtask queue | After all nextTicks |
| `queueMicrotask(cb)` | Promise microtask queue | After all nextTicks |
| `setTimeout(cb, 0)` | Timer macrotask queue | After ALL microtasks |
| `setInterval(cb, n)` | Timer macrotask queue | After ALL microtasks |
| `setImmediate(cb)` | Check macrotask queue | After Timer phase |
| `I/O callbacks` | Pending I/O macrotask queue | After Timer phase |

### The Golden Rules

1. **Sync code ALWAYS runs first** — all of it, top to bottom
2. **Microtasks drain completely** before ANY macrotask runs
3. **process.nextTick > Promise.then** — always
4. **setTimeout(fn, 0) !== "run immediately"** — it means "run after everything else"
5. **Promise executor is synchronous** — only callbacks (.then/.catch) are async
6. **Function arguments are evaluated synchronously** — `fn(console.log("x"))` prints "x" immediately
7. **After EACH macrotask**, all microtasks are drained again before the next macrotask
8. **await pauses the function**, not the program — code outside continues

---

# 5. EXPRESS.JS

## 5.1 What is Express.js?
Express is a **minimal, unopinionated web framework** for Node.js. It provides routing, middleware, and HTTP utilities.

## 5.2 Core Concepts

### Middleware Chain
```javascript
const express = require('express');
const app = express();

// Middleware executes in order
app.use(express.json());                    // 1. Parse JSON body
app.use(cors());                            // 2. Enable CORS
app.use(morgan('dev'));                     // 3. Log requests
app.use('/api', authMiddleware);            // 4. Authenticate /api routes

// Custom middleware
function authMiddleware(req, res, next) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'No token' });

  try {
    const decoded = jwt.verify(token, SECRET);
    req.user = decoded;
    next(); // Pass to next middleware/route
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Error-handling middleware (4 params)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong' });
});
```

### Routing
```javascript
const router = express.Router();

router.get('/users', getUsers);
router.get('/users/:id', getUserById);
router.post('/users', validateBody, createUser);
router.put('/users/:id', updateUser);
router.delete('/users/:id', deleteUser);

// Route parameters
router.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
});

// Query parameters: GET /users?page=1&limit=10
router.get('/users', (req, res) => {
  const { page = 1, limit = 10 } = req.query;
});

app.use('/api/v1', router);
```

---

# 6. NESTJS

## 6.1 What is NestJS?
NestJS is an **opinionated, Angular-inspired Node.js framework** built with TypeScript. It uses decorators, dependency injection, and a modular architecture.

## 6.2 Core Architecture

```
┌─────────────────────────────────────┐
│              Module                   │
│  ┌─────────┐ ┌──────────┐ ┌──────┐ │
│  │Controller│ │  Service  │ │Guard │ │
│  │(Routes)  │→│(Business  │ │(Auth)│ │
│  │          │ │ Logic)    │ │      │ │
│  └─────────┘ └──────────┘ └──────┘ │
│              ┌──────────┐           │
│              │   Pipe    │           │
│              │(Validate) │           │
│              └──────────┘           │
└─────────────────────────────────────┘
```

### Module
```typescript
@Module({
  imports: [DatabaseModule, AuthModule],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Make available to other modules
})
export class UsersModule {}
```

### Controller
```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {} // DI

  @Get()
  findAll(@Query('page') page: number) {
    return this.usersService.findAll(page);
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return this.usersService.findOne(id);
  }

  @Post()
  @UseGuards(AuthGuard)
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

### Service
```typescript
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepo: Repository<User>,
  ) {}

  async findAll(page: number): Promise<User[]> {
    return this.usersRepo.find({
      skip: (page - 1) * 10,
      take: 10,
    });
  }

  async findOne(id: number): Promise<User> {
    const user = await this.usersRepo.findOne({ where: { id } });
    if (!user) throw new NotFoundException(`User #${id} not found`);
    return user;
  }
}
```

### Guards (Authentication)
```typescript
@Injectable()
export class JwtAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const token = request.headers.authorization?.split(' ')[1];
    if (!token) throw new UnauthorizedException();

    try {
      const payload = jwt.verify(token, SECRET);
      request.user = payload;
      return true;
    } catch {
      throw new UnauthorizedException();
    }
  }
}
```

### Pipes (Validation)
```typescript
// DTO with class-validator
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsStrongPassword()
  password: string;
}

// NestJS automatically validates using the DTO
@Post()
@UsePipes(new ValidationPipe({ whitelist: true }))
create(@Body() dto: CreateUserDto) { }
```

---

# 7. EXPRESS.JS VS NESTJS — COMPARISON

| Feature | Express.js | NestJS |
|---------|-----------|--------|
| **Architecture** | Unopinionated, flexible | Opinionated, modular (Angular-style) |
| **Language** | JavaScript | TypeScript (first-class) |
| **Learning curve** | Easy | Steeper |
| **Structure** | You decide | Modules → Controllers → Services |
| **Dependency Injection** | Manual | Built-in |
| **Validation** | Manual (Joi, express-validator) | Built-in (class-validator + Pipes) |
| **Testing** | Manual setup | Built-in testing utilities |
| **Decorators** | No | Yes (@Get, @Post, @Injectable) |
| **Middleware** | Simple function | Guards, Interceptors, Pipes, Filters |
| **Best for** | Small-medium APIs, prototypes | Large enterprise apps, microservices |
| **Performance** | Slightly faster (less overhead) | Slightly slower (abstraction layer) |
| **Your experience** | Zamulk, NowVPlay | GoTo API, AT-Function-GPT |

**Interview Q: When would you choose NestJS over Express?**
- Large team (enforced structure)
- Enterprise project with long lifespan
- Microservices architecture needed
- Strong TypeScript requirement
- Complex dependency injection needs

---

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

---

# 9. GRAPHQL

## 9.1 What is GraphQL?
GraphQL is a **query language for APIs** developed by Facebook (2015). Clients specify exactly what data they need — no more, no less.

## 9.2 Core Concepts

```graphql
# Schema Definition
type User {
  id: ID!
  name: String!
  email: String
  posts: [Post!]!
}

type Post {
  id: ID!
  title: String!
  author: User!
}

type Query {
  user(id: ID!): User
  users(page: Int, limit: Int): [User!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}

# Client Query
query {
  user(id: "123") {
    name
    email
    posts {
      title
    }
  }
}
```

## 9.3 Key Concepts
- **Query** — Read data (like GET)
- **Mutation** — Write data (like POST/PUT/DELETE)
- **Subscription** — Real-time data (WebSocket)
- **Resolver** — Function that returns data for a field
- **Schema** — Contract between client and server

---

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

---

# 11. SQL DATABASES (PostgreSQL, MySQL)

## 11.1 Core SQL Concepts

### Data Types (PostgreSQL)
```sql
-- Numbers
INTEGER, BIGINT, DECIMAL(10,2), FLOAT, SERIAL (auto-increment)

-- Strings
VARCHAR(255), TEXT, CHAR(10)

-- Date/Time
DATE, TIMESTAMP, TIMESTAMPTZ, INTERVAL

-- Boolean
BOOLEAN

-- JSON (PostgreSQL special)
JSON, JSONB (binary, indexable, faster queries)

-- Arrays (PostgreSQL special)
INTEGER[], TEXT[]

-- UUID
UUID
```

### CRUD Operations
```sql
-- Create
INSERT INTO users (name, email, age) VALUES ('Saad', 'saad@email.com', 28);

-- Read
SELECT * FROM users WHERE age > 25 ORDER BY name ASC LIMIT 10 OFFSET 0;

-- Update
UPDATE users SET name = 'Saad Sohail' WHERE id = 1;

-- Delete
DELETE FROM users WHERE id = 1;
```

### JOINs (Critical Interview Topic)

```sql
-- INNER JOIN — only matching rows from both tables
SELECT u.name, p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id;

-- LEFT JOIN — all rows from left table + matching from right
SELECT u.name, p.title
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
-- Users without posts will show with NULL for title

-- RIGHT JOIN — all rows from right table + matching from left
SELECT u.name, p.title
FROM users u
RIGHT JOIN posts p ON u.id = p.user_id;

-- FULL OUTER JOIN — all rows from both tables
SELECT u.name, p.title
FROM users u
FULL OUTER JOIN posts p ON u.id = p.user_id;
```

```
INNER JOIN:     LEFT JOIN:      RIGHT JOIN:     FULL OUTER JOIN:
  ┌───┐           ┌───┐           ┌───┐           ┌───┐
  │ A │∩│ B │     █ A █∩│ B │     │ A │∩█ B █     █ A █∩█ B █
  └───┘           └───┘           └───┘           └───┘
Only matching   All A +          All B +         All from
rows            matching B       matching A      both tables
```

### Indexes
```sql
-- B-Tree index (default, most common)
CREATE INDEX idx_users_email ON users(email);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Composite index
CREATE INDEX idx_users_name_age ON users(name, age);
-- Follows "leftmost prefix" rule: this index works for
-- queries on (name), (name, age), but NOT (age) alone

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- GIN index (for JSONB, arrays, full-text search)
CREATE INDEX idx_user_data ON users USING GIN(metadata);
```

**Interview Q: When NOT to add indexes?**
- Small tables (< 1000 rows)
- Columns with low cardinality (boolean, status with 2-3 values)
- Tables with heavy write operations (indexes slow down INSERT/UPDATE)
- Columns rarely used in WHERE/JOIN/ORDER BY

### Transactions & ACID
```sql
BEGIN;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

-- If anything fails between BEGIN and COMMIT, nothing is saved
COMMIT;  -- or ROLLBACK; to undo
```

**ACID Properties:**
- **Atomicity** — All or nothing. Either all operations succeed or none do.
- **Consistency** — Database moves from one valid state to another.
- **Isolation** — Concurrent transactions don't interfere with each other.
- **Durability** — Once committed, data survives crashes.

### Aggregate Functions
```sql
SELECT
  department,
  COUNT(*) as total,
  AVG(salary) as avg_salary,
  MAX(salary) as max_salary,
  MIN(salary) as min_salary,
  SUM(salary) as total_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000
ORDER BY avg_salary DESC;
```

### Subqueries vs JOINs
```sql
-- Subquery (slower for large datasets)
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE total > 1000);

-- JOIN equivalent (usually faster)
SELECT DISTINCT u.*
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.total > 1000;
```

### Window Functions (Advanced)
```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT
  name,
  department,
  salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank
FROM employees;

-- Running total
SELECT
  date,
  amount,
  SUM(amount) OVER (ORDER BY date) as running_total
FROM transactions;
```

### Normalization

| Form | Rule | Example |
|------|------|---------|
| **1NF** | No repeating groups, atomic values | Split "skills: JS, Python" into separate rows |
| **2NF** | 1NF + no partial dependencies | Remove columns that depend on part of composite key |
| **3NF** | 2NF + no transitive dependencies | If A→B and B→C, don't store C in same table |

---

# 12. NoSQL DATABASES (MongoDB)

## 12.1 Core Concepts

### Document Structure
```javascript
// MongoDB stores data as BSON (Binary JSON) documents
{
  _id: ObjectId("507f191e810c19729de860ea"),
  name: "Saad",
  email: "saad@email.com",
  age: 28,
  address: {                          // Embedded document
    city: "Lahore",
    country: "Pakistan"
  },
  skills: ["Node.js", "MongoDB"],     // Array
  posts: [                            // Array of embedded documents
    { title: "First Post", likes: 10 },
    { title: "Second Post", likes: 25 }
  ],
  createdAt: ISODate("2024-01-15")
}
```

### CRUD Operations
```javascript
// Create
db.users.insertOne({ name: "Saad", age: 28 });
db.users.insertMany([{ name: "Ali" }, { name: "Sara" }]);

// Read
db.users.find({ age: { $gt: 25 } }).sort({ name: 1 }).limit(10);
db.users.findOne({ email: "saad@email.com" });

// Update
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { name: "Saad Sohail" }, $inc: { age: 1 } }
);

// Delete
db.users.deleteOne({ _id: ObjectId("...") });
```

### Query Operators
```javascript
// Comparison
$eq, $ne, $gt, $gte, $lt, $lte, $in, $nin

// Logical
$and, $or, $not, $nor

// Element
$exists, $type

// Array
$all, $elemMatch, $size

// Example
db.users.find({
  $and: [
    { age: { $gte: 25, $lte: 35 } },
    { skills: { $in: ["Node.js", "React"] } },
    { "address.city": { $exists: true } }
  ]
});
```

### Aggregation Pipeline
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },           // Filter
  { $group: {                                      // Group
    _id: "$customerId",
    totalSpent: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }},
  { $sort: { totalSpent: -1 } },                  // Sort
  { $limit: 10 },                                  // Limit
  { $lookup: {                                     // Join
    from: "customers",
    localField: "_id",
    foreignField: "_id",
    as: "customerInfo"
  }},
  { $project: {                                    // Select fields
    totalSpent: 1,
    orderCount: 1,
    customerName: { $arrayElemAt: ["$customerInfo.name", 0] }
  }}
]);
```

### Indexes in MongoDB
```javascript
// Single field
db.users.createIndex({ email: 1 });       // 1 = ascending, -1 = descending

// Compound
db.users.createIndex({ name: 1, age: -1 });

// Unique
db.users.createIndex({ email: 1 }, { unique: true });

// Text (full-text search)
db.posts.createIndex({ title: "text", body: "text" });
db.posts.find({ $text: { $search: "nodejs tutorial" } });

// Geospatial (2dsphere) — used at Zamulk for property search
db.properties.createIndex({ location: "2dsphere" });
db.properties.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [74.35, 31.52] },
      $maxDistance: 5000 // meters
    }
  }
});

// TTL (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

### Embedding vs Referencing

| Strategy | When to Use | Example |
|----------|-------------|---------|
| **Embed** | 1-to-few, data accessed together, rarely changes | User + address |
| **Reference** | 1-to-many, large/growing arrays, data shared across docs | User + posts |

```javascript
// Embedded (denormalized)
{
  name: "Saad",
  address: { city: "Lahore", zip: "54000" }  // Lives inside user
}

// Referenced (normalized)
// users collection
{ _id: 1, name: "Saad" }
// posts collection
{ _id: 101, title: "Hello", authorId: 1 }    // References user
```

---

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

---

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

---

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

---

# 16. SEQUELIZE ORM

## 16.1 What is Sequelize?
Sequelize is a **Promise-based Node.js ORM** for SQL databases (PostgreSQL, MySQL, SQLite, MSSQL). You used it at Obenan with 100+ models and 400+ migrations.

## 16.2 Core Concepts

### Model Definition
```javascript
const { DataTypes } = require('sequelize');

module.exports = (sequelize) => {
  const User = sequelize.define('User', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    name: {
      type: DataTypes.STRING(100),
      allowNull: false,
      validate: { len: [2, 100] },
    },
    email: {
      type: DataTypes.STRING,
      unique: true,
      validate: { isEmail: true },
    },
    role: {
      type: DataTypes.ENUM('user', 'admin', 'superadmin'),
      defaultValue: 'user',
    },
    metadata: {
      type: DataTypes.JSONB, // PostgreSQL
    },
  }, {
    tableName: 'users',
    timestamps: true,      // createdAt, updatedAt
    paranoid: true,        // soft delete (deletedAt)
    indexes: [
      { fields: ['email'], unique: true },
      { fields: ['role'] },
    ],
  });

  return User;
};
```

### Associations
```javascript
// One-to-Many
User.hasMany(Post, { foreignKey: 'userId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'userId', as: 'author' });

// Many-to-Many
User.belongsToMany(Role, { through: 'user_roles' });
Role.belongsToMany(User, { through: 'user_roles' });

// One-to-One
User.hasOne(Profile, { foreignKey: 'userId' });
Profile.belongsTo(User, { foreignKey: 'userId' });
```

### Queries
```javascript
// Find with associations (eager loading)
const user = await User.findOne({
  where: { id: userId },
  include: [
    { model: Post, as: 'posts', where: { status: 'published' }, required: false },
    { model: Profile, as: 'profile' }
  ],
  attributes: ['id', 'name', 'email'],
});

// Pagination
const { count, rows } = await User.findAndCountAll({
  where: { role: 'admin' },
  limit: 10,
  offset: (page - 1) * 10,
  order: [['createdAt', 'DESC']],
});

// Raw query (when ORM is too limiting)
const results = await sequelize.query(
  'SELECT * FROM users WHERE age > :age',
  { replacements: { age: 25 }, type: QueryTypes.SELECT }
);

// Transactions
const t = await sequelize.transaction();
try {
  await User.create({ name: 'Saad' }, { transaction: t });
  await Profile.create({ userId: user.id }, { transaction: t });
  await t.commit();
} catch (error) {
  await t.rollback();
}
```

### Migrations
```javascript
// npx sequelize migration:generate --name add-users-table
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('users', {
      id: { type: Sequelize.UUID, primaryKey: true },
      name: { type: Sequelize.STRING, allowNull: false },
      email: { type: Sequelize.STRING, unique: true },
      createdAt: { type: Sequelize.DATE },
      updatedAt: { type: Sequelize.DATE },
    });
    await queryInterface.addIndex('users', ['email']);
  },
  down: async (queryInterface) => {
    await queryInterface.dropTable('users');
  },
};
```

### Multi-Tenant Scoping (Your Obenan Experience)
```javascript
// Scope queries by company, user, and location
const reviews = await Review.findAll({
  where: {
    companyId: req.user.companyId,    // Tenant isolation
    locationId: req.params.locationId, // Location scoping
  },
  include: [{ model: Location, where: { companyId: req.user.companyId } }],
});
```

---

# 17. AUTHENTICATION & SECURITY

## 17.1 JWT (JSON Web Tokens)

### Structure
```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9.signature
```

```javascript
// Header
{ "alg": "HS256", "typ": "JWT" }

// Payload
{ "userId": 1, "role": "admin", "iat": 1700000000, "exp": 1700003600 }

// Signature
HMACSHA256(base64(header) + "." + base64(payload), secret)
```

### Implementation
```javascript
const jwt = require('jsonwebtoken');

// Generate
function generateTokens(user) {
  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: '15m' }
  );

  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_SECRET,
    { expiresIn: '7d' }
  );

  return { accessToken, refreshToken };
}

// Verify
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'No token provided' });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

### Access Token vs Refresh Token

| Feature | Access Token | Refresh Token |
|---------|-------------|---------------|
| Lifespan | Short (15 min) | Long (7-30 days) |
| Stored | Memory / HTTP header | HttpOnly cookie / DB |
| Purpose | Authenticate API requests | Get new access tokens |
| Contains | userId, role, permissions | userId only |

## 17.2 OAuth 2.0

```
┌────────┐     1. Redirect to Google     ┌──────────┐
│  User  │ ──────────────────────────────>│  Google  │
│(Browser)│<──────────────────────────────│  OAuth   │
│        │     2. User logs in & consents │          │
│        │     3. Redirect with auth code │          │
└────┬───┘                                └──────────┘
     │ 4. Send auth code to your server       │
┌────▼───┐     5. Exchange code for tokens ┌──┴───────┐
│  Your  │ ──────────────────────────────>│  Google  │
│ Server │<──────────────────────────────│  Token   │
│        │     6. Access token + user info │ Endpoint │
└────────┘                                └──────────┘
```

**Your Experience:** Google My Business OAuth for location sync, reviews, and insights.

## 17.3 Passport.js
```javascript
const passport = require('passport');
const { Strategy: JwtStrategy, ExtractJwt } = require('passport-jwt');

passport.use(new JwtStrategy({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
}, async (payload, done) => {
  try {
    const user = await User.findByPk(payload.userId);
    if (user) return done(null, user);
    return done(null, false);
  } catch (err) {
    return done(err, false);
  }
}));

// Protect route
app.get('/api/profile', passport.authenticate('jwt', { session: false }), handler);
```

## 17.4 RBAC (Role-Based Access Control)

```javascript
// Role hierarchy
const permissions = {
  superadmin: ['read', 'write', 'delete', 'manage_users', 'manage_billing'],
  admin:      ['read', 'write', 'delete', 'manage_users'],
  manager:    ['read', 'write', 'delete'],
  user:       ['read', 'write'],
  viewer:     ['read'],
};

// Middleware
function authorize(...allowedRoles) {
  return (req, res, next) => {
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// Usage
router.delete('/users/:id', authorize('admin', 'superadmin'), deleteUser);
```

## 17.5 Security Best Practices

| Threat | Prevention |
|--------|-----------|
| **SQL Injection** | Parameterized queries, ORMs |
| **XSS** | Sanitize input, Content-Security-Policy header |
| **CSRF** | CSRF tokens, SameSite cookies |
| **Brute Force** | Rate limiting, account lockout |
| **Password Storage** | bcrypt with salt (never plain text) |
| **Sensitive Data** | HTTPS only, encrypt at rest |
| **JWT Theft** | Short expiry, HttpOnly cookies, refresh rotation |

```javascript
const bcrypt = require('bcrypt');

// Hash password
const hash = await bcrypt.hash(password, 12); // 12 salt rounds

// Verify
const isMatch = await bcrypt.compare(password, hash);
```

---

# 18. SOCKET.IO — REAL-TIME COMMUNICATION

## 18.1 What is Socket.IO?
Socket.IO enables **real-time, bidirectional, event-based communication** between client and server. Built on WebSocket protocol with fallbacks.

## 18.2 Architecture

```
┌─────────┐    WebSocket     ┌─────────┐
│ Client 1 │ <──────────────> │         │
├─────────┤                   │ Socket  │
│ Client 2 │ <──────────────> │   .IO   │
├─────────┤                   │ Server  │
│ Client 3 │ <──────────────> │         │
└─────────┘                   └─────────┘
```

## 18.3 Implementation

```javascript
// Server
const io = require('socket.io')(server, {
  cors: { origin: '*' },
});

io.on('connection', (socket) => {
  console.log('Connected:', socket.id);

  // Join a room (e.g., match room, chat room)
  socket.on('joinRoom', (roomId) => {
    socket.join(roomId);
    socket.to(roomId).emit('userJoined', { userId: socket.userId });
  });

  // Send message to room
  socket.on('sendMessage', ({ roomId, message }) => {
    io.to(roomId).emit('newMessage', {
      from: socket.userId,
      message,
      timestamp: Date.now(),
    });
  });

  // Typing indicator
  socket.on('typing', (roomId) => {
    socket.to(roomId).emit('userTyping', socket.userId);
  });

  socket.on('disconnect', () => {
    console.log('Disconnected:', socket.id);
  });
});

// Emit to specific user
io.to(socketId).emit('notification', data);

// Emit to all users in a room
io.to(roomId).emit('event', data);

// Broadcast to all except sender
socket.broadcast.emit('event', data);
```

### Scaling Socket.IO with Redis Adapter
```javascript
const { createAdapter } = require('@socket.io/redis-adapter');
const { createClient } = require('redis');

const pubClient = createClient({ url: 'redis://localhost:6379' });
const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));
// Now Socket.IO works across multiple server instances
```

## 18.4 Your Real-World Usage

| Project | Socket.IO Use Case |
|---------|-------------------|
| **Zamulk** | Live chat, audio/video calls between buyers/sellers/agents, real-time bidding in auction room |
| **NowVPlay** | Instant match invitations, booking confirmations, live activity updates |
| **US Ride** | Real-time ride tracking, driver-rider matching |

---

# 19. BULLMQ — JOB QUEUES & BACKGROUND PROCESSING

## 19.1 What is BullMQ?
BullMQ is a **Redis-backed job queue** for Node.js. Used for background processing, scheduled tasks, and async workflows.

## 19.2 Why Use Job Queues?
- Heavy tasks that would block the API response
- Scheduled/recurring tasks (cron-like)
- Retry failed operations
- Rate-limiting external API calls
- Decoupling services

## 19.3 Implementation

```javascript
const { Queue, Worker } = require('bullmq');
const Redis = require('ioredis');

const connection = new Redis({ host: 'localhost', port: 6379 });

// Create queue
const reviewQueue = new Queue('review-processing', { connection });

// Add jobs
await reviewQueue.add('fetch-reviews', {
  locationId: 123,
  companyId: 456,
}, {
  attempts: 3,                    // Retry 3 times on failure
  backoff: { type: 'exponential', delay: 5000 }, // 5s, 10s, 20s
  removeOnComplete: 100,         // Keep last 100 completed jobs
  removeOnFail: 50,              // Keep last 50 failed jobs
});

// Scheduled job (every hour)
await reviewQueue.add('sync-reviews', { all: true }, {
  repeat: { pattern: '0 * * * *' }, // cron pattern
});

// Worker (processes jobs)
const worker = new Worker('review-processing', async (job) => {
  console.log(`Processing ${job.name} with data:`, job.data);

  switch (job.name) {
    case 'fetch-reviews':
      await fetchReviewsFromGoogle(job.data.locationId);
      break;
    case 'analyze-sentiment':
      await analyzeSentiment(job.data.reviewId);
      break;
    case 'send-notification':
      await sendEmail(job.data);
      break;
  }
}, {
  connection,
  concurrency: 5, // Process 5 jobs in parallel
});

// Events
worker.on('completed', (job) => console.log(`Job ${job.id} completed`));
worker.on('failed', (job, err) => console.error(`Job ${job.id} failed:`, err));
```

## 19.4 Your BullMQ Usage at Obenan/Global Software Consulting
- Review fetching from Google Business
- Sentiment analysis pipeline
- Report generation
- Email notification queues
- Scheduled data sync jobs

---

# 20. AWS CLOUD SERVICES

## 20.1 Services You've Used

### EC2 (Elastic Compute Cloud)
```
Virtual servers in the cloud.

Instance Types:
- t3.micro  → 2 vCPU, 1 GB RAM (dev/staging)
- t3.medium → 2 vCPU, 4 GB RAM (small production)
- c5.xlarge → 4 vCPU, 8 GB RAM (compute-intensive)

Key Concepts:
- AMI (Amazon Machine Image) → server template
- Security Groups → virtual firewall (inbound/outbound rules)
- Key Pairs → SSH access
- Elastic IP → static public IP
```

### S3 (Simple Storage Service)
```
Object storage for files, backups, static assets.

Key Concepts:
- Bucket → container for objects
- Object → file + metadata
- Pre-signed URLs → temporary secure file access
- Storage classes → Standard, IA (Infrequent Access), Glacier (archive)
- Lifecycle policies → auto-move old data to cheaper storage

Use Cases (Your Experience):
- Data backup and archival
- Disaster recovery
- Static asset hosting
- User file uploads
```

### Lambda (Serverless Functions)
```
Run code without managing servers. Pay per execution.

Triggers: API Gateway, S3, SQS, CloudWatch Events
Limits: 15 min timeout, 10 GB memory, 50 MB deploy package

Your Usage:
- Serverless sentiment analysis
- Review data processing
```

### VPC (Virtual Private Cloud)
```
Isolated network environment in AWS.

Components:
- Subnets → public (internet-facing) and private (internal only)
- Internet Gateway → connects VPC to internet
- NAT Gateway → lets private subnets access internet (outbound only)
- Route Tables → rules for traffic routing
- Network ACLs → stateless firewall at subnet level
- Security Groups → stateful firewall at instance level

Typical Architecture:
┌────────────────── VPC (10.0.0.0/16) ──────────────────┐
│  ┌─── Public Subnet ───┐  ┌─── Private Subnet ───┐    │
│  │  Load Balancer       │  │  App Servers (EC2)   │    │
│  │  NAT Gateway         │  │  Database (RDS)      │    │
│  │  Bastion Host        │  │  Redis (ElastiCache) │    │
│  └──────────────────────┘  └──────────────────────┘    │
│           ↕ Internet Gateway                            │
└─────────────────────────────────────────────────────────┘
```

### ELB (Elastic Load Balancer)
```
Distributes traffic across multiple targets.

Types:
- ALB (Application LB) → HTTP/HTTPS, path-based routing
- NLB (Network LB) → TCP/UDP, ultra-low latency
- CLB (Classic LB) → Legacy

Health Checks:
- Sends requests to /health endpoint
- Removes unhealthy instances from rotation
- Re-adds when healthy again
```

### CodePipeline + CodeDeploy (CI/CD)
```
Pipeline: Source → Build → Test → Deploy

Source:  GitHub/CodeCommit (trigger on push)
Build:   CodeBuild (npm install, npm test, npm build)
Deploy:  CodeDeploy (rolling, blue/green, all-at-once)

Deployment Strategies:
- All-at-once: Fast but risky (downtime)
- Rolling: Deploy to subset at a time
- Blue/Green: Two environments, switch traffic
- Canary: Deploy to 10% → monitor → deploy to 100%
```

## 20.2 Common AWS Interview Questions

**Q: How would you design a highly available architecture?**
- Multi-AZ deployment (instances in multiple Availability Zones)
- Auto Scaling Group (scale based on CPU/memory/requests)
- Load Balancer distributing traffic
- RDS Multi-AZ for database failover
- S3 for static assets (11 nines durability)

**Q: How do you secure an AWS infrastructure?**
- VPC with private subnets for app/DB
- Security Groups (least privilege)
- IAM roles (not access keys) for service-to-service
- Encryption at rest (KMS) and in transit (TLS)
- CloudTrail for audit logging

---

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

---

# 22. NGINX

## 22.1 What is Nginx?
Nginx is a **reverse proxy, load balancer, and web server**. Used to route traffic to your Node.js app.

## 22.2 Configuration

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name myapp.com;

    # Redirect HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name myapp.com;

    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    # API proxy
    location /api {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # WebSocket proxy (Socket.IO)
    location /socket.io {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # Static files
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }

    # Gzip compression
    gzip on;
    gzip_types text/css application/json application/javascript;
}
```

---

# 23. CI/CD

## 23.1 What is CI/CD?
- **CI (Continuous Integration):** Automatically build and test code on every push
- **CD (Continuous Delivery):** Automatically deploy tested code to staging/production

## 23.2 GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run lint
      - run: npm test

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to EC2
        run: |
          ssh -i ${{ secrets.SSH_KEY }} ec2-user@${{ secrets.EC2_HOST }} "
            cd /app && git pull && npm ci && pm2 restart all
          "
```

## 23.3 Branch Protection Rules
```
- Require pull request reviews (1-2 approvers)
- Require status checks to pass (tests, lint)
- Require branches to be up to date
- No direct pushes to main
- Require linear history (no merge commits)
```

---

# 24. AI & LLMs

## 24.1 What is an LLM?
Large Language Model — a neural network trained on massive text data that can generate, understand, and reason about text. Examples: GPT-4, Claude, Gemini.

## 24.2 OpenAI API

```javascript
const OpenAI = require('openai');
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// Chat Completion
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [
    { role: 'system', content: 'You are a helpful assistant.' },
    { role: 'user', content: 'Analyze this review for sentiment.' },
  ],
  temperature: 0,       // 0 = deterministic, 1 = creative
  max_tokens: 500,
  response_format: { type: 'json_object' }, // Force JSON output
});

const answer = response.choices[0].message.content;

// Streaming
const stream = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'Tell me a story' }],
  stream: true,
});

for await (const chunk of stream) {
  const text = chunk.choices[0]?.delta?.content || '';
  process.stdout.write(text); // Real-time output
}

// Function Calling (Tool Use)
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages: [{ role: 'user', content: 'What is the weather in Lahore?' }],
  tools: [{
    type: 'function',
    function: {
      name: 'getWeather',
      description: 'Get current weather for a city',
      parameters: {
        type: 'object',
        properties: {
          city: { type: 'string', description: 'City name' },
        },
        required: ['city'],
      },
    },
  }],
});
```

## 24.3 Claude API (Anthropic)

```javascript
const Anthropic = require('@anthropic-ai/sdk');
const anthropic = new Anthropic({ apiKey: process.env.CLAUDE_API_KEY });

const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  system: 'You are a sentiment analysis expert.',
  messages: [
    { role: 'user', content: 'Analyze this review: "Great food but slow service"' },
  ],
});

console.log(response.content[0].text);

// Tool Use
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 1024,
  tools: [{
    name: 'search_reviews',
    description: 'Search reviews for a business location',
    input_schema: {
      type: 'object',
      properties: {
        locationId: { type: 'string' },
        sentiment: { type: 'string', enum: ['positive', 'negative', 'neutral'] },
      },
      required: ['locationId'],
    },
  }],
  messages: [{ role: 'user', content: 'Find negative reviews for location 123' }],
});
```

## 24.4 Model Comparison

| Feature | GPT-4o | Claude Opus | Claude Sonnet | Claude Haiku | GPT-4o Mini |
|---------|--------|-------------|---------------|-------------|-------------|
| **Quality** | Excellent | Excellent | Very Good | Good | Good |
| **Speed** | Fast | Slower | Fast | Very Fast | Very Fast |
| **Cost** | $$$ | $$$$ | $$ | $ | $ |
| **Context** | 128K | 200K | 200K | 200K | 128K |
| **Best for** | General tasks | Complex reasoning | Balanced | Quick tasks | Cheap, fast |
| **Coding** | Excellent | Excellent | Very Good | Good | Good |

**Interview Q: How do you choose which model to use?**
- **Simple classification** → Haiku or GPT-4o Mini (cheapest)
- **Complex reasoning** → Opus or GPT-4o
- **Production chatbot** → Sonnet or GPT-4o (balance of quality/cost)
- **Cost-sensitive high-volume** → Haiku + caching
- **Safety-critical** → Claude (better guardrails)

---

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

---

# 26. FINE-TUNING LLMs

## 26.1 What is Fine-Tuning?
Taking a pre-trained model and training it further on your specific data to improve performance for a particular task.

## 26.2 When to Fine-Tune vs RAG

| Criteria | Fine-Tuning | RAG |
|----------|------------|-----|
| Need latest data | No (static after training) | Yes (dynamic retrieval) |
| Custom behavior/tone | Yes | Somewhat (via prompting) |
| Cost | High (training + hosting) | Low (just API calls) |
| Speed to deploy | Slow (hours/days) | Fast (minutes) |
| Private data | Baked into model | Retrieved at query time |
| Best for | Style, format, specialized tasks | Knowledge retrieval, Q&A |

**Rule of thumb:** Try RAG first. Fine-tune only when RAG isn't enough.

## 26.3 Fine-Tuning with OpenAI

```javascript
// Training data format (JSONL)
{"messages": [{"role": "system", "content": "You classify reviews."}, {"role": "user", "content": "Great food!"}, {"role": "assistant", "content": "{\"sentiment\": \"positive\", \"score\": 0.95}"}]}
{"messages": [{"role": "system", "content": "You classify reviews."}, {"role": "user", "content": "Terrible service"}, {"role": "assistant", "content": "{\"sentiment\": \"negative\", \"score\": 0.9}"}]}

// Upload training file
const file = await openai.files.create({
  file: fs.createReadStream('training.jsonl'),
  purpose: 'fine-tune',
});

// Create fine-tuning job
const job = await openai.fineTuning.jobs.create({
  training_file: file.id,
  model: 'gpt-4o-mini-2024-07-18',
  hyperparameters: { n_epochs: 3 },
});

// Use fine-tuned model
const response = await openai.chat.completions.create({
  model: 'ft:gpt-4o-mini-2024-07-18:org:custom-name:id',
  messages: [{ role: 'user', content: 'Review: Not bad at all!' }],
});
```

---

# 27. PROMPT ENGINEERING

## 27.1 Key Techniques

### System Prompts
```
You are a senior review analyst for a multi-location business management platform.
Your task is to analyze customer reviews and extract:
1. Overall sentiment (positive/negative/neutral)
2. Key topics mentioned
3. Action items for the business owner

Always respond in valid JSON format.
```

### Few-Shot Prompting
```
Classify the following reviews:

Review: "Amazing pizza, will come back!"
Classification: {"sentiment": "positive", "category": "food_quality"}

Review: "Waited 45 minutes for a table"
Classification: {"sentiment": "negative", "category": "wait_time"}

Review: "Decent ambiance but pricey"
Classification:
```

### Chain-of-Thought
```
Analyze this business review step by step:
1. First, identify the overall sentiment
2. Then, list specific aspects mentioned (food, service, price, etc.)
3. For each aspect, determine if it's positive or negative
4. Finally, provide an overall score from 1-5

Review: "The food was incredible but the waiter was rude and it took forever to get our check."
```

### Structured Output
```
Respond in this exact JSON format:
{
  "sentiment": "positive" | "negative" | "neutral",
  "confidence": 0.0 to 1.0,
  "topics": ["string"],
  "summary": "one sentence summary",
  "actionItems": ["string"]
}
```

## 27.2 Temperature Guide

| Temperature | Behavior | Use Case |
|-------------|----------|----------|
| 0.0 | Deterministic, consistent | Classification, extraction, code |
| 0.3 | Mostly consistent, slight variation | Summarization, analysis |
| 0.7 | Creative, varied | Content generation, brainstorming |
| 1.0 | Very creative, unpredictable | Poetry, creative writing |

## 27.3 Prompt Injection Prevention
```javascript
// Never put user input directly in system prompt
// BAD
const systemPrompt = `You help with ${userInput}`;

// GOOD — separate user input from instructions
const messages = [
  { role: 'system', content: 'You are a review analyzer. Only analyze reviews.' },
  { role: 'user', content: `Analyze this review: "${sanitizedInput}"` },
];

// Input validation
function sanitizeForLLM(input) {
  // Remove attempts to override system prompt
  return input
    .replace(/ignore previous instructions/gi, '')
    .replace(/system:/gi, '')
    .slice(0, 5000); // Limit length
}
```

---

# 28. STRIPE PAYMENT INTEGRATION

## 28.1 Core Concepts

```
Customer → Subscription → Invoice → Payment

Key Objects:
- Customer: The person paying
- Product: What you sell (e.g., "Pro Plan")
- Price: How much ($30/month, $300/year)
- Subscription: Recurring billing
- PaymentIntent: One-time payment
- Invoice: Bill generated for subscription
```

## 28.2 Implementation

```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

// Create customer
const customer = await stripe.customers.create({
  email: 'saad@email.com',
  name: 'Saad Sohail',
  metadata: { userId: '123' },
});

// Create subscription
const subscription = await stripe.subscriptions.create({
  customer: customer.id,
  items: [{ price: 'price_xxx' }], // Monthly price ID
  payment_behavior: 'default_incomplete',
  expand: ['latest_invoice.payment_intent'],
});

// Webhook handling (CRITICAL for production)
app.post('/webhooks/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];
  const event = stripe.webhooks.constructEvent(req.body, sig, WEBHOOK_SECRET);

  switch (event.type) {
    case 'invoice.paid':
      // Activate subscription
      await activateSubscription(event.data.object);
      break;
    case 'invoice.payment_failed':
      // Notify user, retry
      await handlePaymentFailure(event.data.object);
      break;
    case 'customer.subscription.deleted':
      // Deactivate access
      await deactivateSubscription(event.data.object);
      break;
  }

  res.json({ received: true });
});
```

---

# 29. SENDGRID EMAIL INTEGRATION

```javascript
const sgMail = require('@sendgrid/mail');
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

// Send email
await sgMail.send({
  to: 'user@email.com',
  from: 'noreply@obenan.com',
  subject: 'New Review Alert',
  templateId: 'd-xxxxx',           // SendGrid dynamic template
  dynamicTemplateData: {
    name: 'Saad',
    reviewCount: 5,
    locationName: 'Main Street Branch',
  },
});

// Webhook verification (verify events are from SendGrid)
app.post('/webhooks/sendgrid', (req, res) => {
  const signature = req.headers['x-twilio-email-event-webhook-signature'];
  const timestamp = req.headers['x-twilio-email-event-webhook-timestamp'];

  // Verify signature...

  for (const event of req.body) {
    switch (event.event) {
      case 'delivered': /* track delivery */ break;
      case 'bounce':    /* handle bounce */  break;
      case 'open':      /* track opens */    break;
      case 'click':     /* track clicks */   break;
    }
  }

  res.sendStatus(200);
});
```

---

# 30. TESTING (MOCHA, CHAI)

## 30.1 Testing Pyramid

```
         ┌──────┐
         │  E2E  │  Few (slow, expensive)
        ┌┴──────┴┐
        │ Integr. │  Some
       ┌┴────────┴┐
       │   Unit    │  Many (fast, cheap)
       └──────────┘
```

## 30.2 Mocha + Chai

```javascript
const { expect } = require('chai');
const chaiHttp = require('chai-http');
const app = require('../app');

chai.use(chaiHttp);

describe('Users API', () => {
  // Setup
  before(async () => { await seedDatabase(); });
  after(async () => { await cleanDatabase(); });

  describe('GET /api/users', () => {
    it('should return all users', async () => {
      const res = await chai.request(app).get('/api/users');

      expect(res).to.have.status(200);
      expect(res.body).to.be.an('array');
      expect(res.body).to.have.length.greaterThan(0);
      expect(res.body[0]).to.have.property('name');
      expect(res.body[0]).to.have.property('email');
    });

    it('should support pagination', async () => {
      const res = await chai.request(app)
        .get('/api/users')
        .query({ page: 1, limit: 5 });

      expect(res.body).to.have.length.at.most(5);
    });
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const res = await chai.request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${token}`)
        .send({ name: 'Test User', email: 'test@email.com' });

      expect(res).to.have.status(201);
      expect(res.body.name).to.equal('Test User');
    });

    it('should return 400 for invalid data', async () => {
      const res = await chai.request(app)
        .post('/api/users')
        .set('Authorization', `Bearer ${token}`)
        .send({ name: '' }); // Invalid

      expect(res).to.have.status(400);
      expect(res.body).to.have.property('error');
    });

    it('should return 401 without auth token', async () => {
      const res = await chai.request(app)
        .post('/api/users')
        .send({ name: 'Test' });

      expect(res).to.have.status(401);
    });
  });
});
```

## 30.3 Testing Best Practices
- Test behavior, not implementation
- One assertion per test (ideally)
- Use descriptive test names
- Arrange → Act → Assert pattern
- Mock external services (don't call real APIs in tests)
- Test edge cases and error paths

---

# 31. GIT, GITHUB & BRANCHING STRATEGIES

## 31.1 Git Flow
```
main ─────────────────────────────────────────────
  │                              ↑
  └── develop ─────────────────────────────────────
       │           ↑          ↑
       └── feature/login ─────┘
       └── feature/reviews ───┘
       └── hotfix/security ───────────── merge to main
```

## 31.2 Common Commands
```bash
# Branching
git checkout -b feature/new-api
git push -u origin feature/new-api

# Rebase (clean history)
git checkout feature/new-api
git rebase main

# Squash commits
git rebase -i HEAD~3  # Squash last 3 commits

# Stash (save work temporarily)
git stash
git stash pop

# Cherry-pick (take specific commit)
git cherry-pick abc123

# Undo last commit (keep changes)
git reset --soft HEAD~1

# View changes
git log --oneline --graph
git diff --staged
```

## 31.3 PR Best Practices
- Small, focused PRs (< 400 lines)
- Descriptive title and description
- Link to Jira ticket
- Include screenshots for UI changes
- Request specific reviewers
- Address all comments before merge

---

# 32. MONITORING (SENTRY, CODECLIMATE)

## 32.1 Sentry (Error Monitoring)
```javascript
const Sentry = require('@sentry/node');

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1, // 10% of transactions
});

// Auto-captures unhandled errors
// Manual capture:
try {
  riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { module: 'payments' },
    extra: { userId: '123' },
  });
}
```

## 32.2 CodeClimate (Code Quality)
```
Metrics tracked:
- Maintainability (A-F grade)
- Code duplication
- Complexity (cyclomatic)
- Test coverage percentage
- Code smells
```

---

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

---

# 34. PYTHON BASICS

## 34.1 Key Concepts (For AI/ML Context)

```python
# Variables (no type declaration needed)
name = "Saad"
age = 28
skills = ["Node.js", "Python", "AWS"]

# Functions
def analyze_sentiment(text: str) -> dict:
    """Analyze sentiment of text."""
    result = model.predict(text)
    return {"sentiment": result.label, "score": result.score}

# List comprehensions
positive = [r for r in reviews if r.sentiment == "positive"]

# Dictionary
user = {"name": "Saad", "age": 28}

# Classes
class ReviewAnalyzer:
    def __init__(self, model_name: str):
        self.model = load_model(model_name)

    def analyze(self, text: str) -> str:
        return self.model.predict(text)

# Async (asyncio)
import asyncio

async def fetch_reviews(location_id: int):
    async with aiohttp.ClientSession() as session:
        async with session.get(f"/api/reviews/{location_id}") as resp:
            return await resp.json()

# Virtual environments
# python -m venv venv
# source venv/bin/activate
# pip install -r requirements.txt
```

---

# 35. BEHAVIORAL & SOFT SKILL QUESTIONS

## 35.1 Common Questions & Answer Frameworks

**Use STAR method: Situation → Task → Action → Result**

### Q: Tell me about yourself
> "I'm a Senior Full-Stack Engineer with 6+ years of experience building production-grade systems. I currently work at Global Software Consulting where I architect AI-powered SaaS platforms using Node.js, PostgreSQL, MongoDB, and AWS. I've integrated LLMs like GPT-4 and Claude into production systems, built RAG architectures, and managed infrastructure serving thousands of businesses. I'm passionate about clean architecture, scalable systems, and shipping quality software."

### Q: Describe a challenging technical problem you solved
> **Situation:** At Obenan, we had a SaaS platform serving thousands of businesses, and review fetching from Google was bottlenecking our system.
> **Task:** I needed to process thousands of review fetches without blocking the main application.
> **Action:** I designed an asynchronous job processing pipeline using BullMQ and Redis. Jobs were queued with retry logic, exponential backoff, and concurrency limits. I also added a sentiment analysis step using GPT-4 that ran as a separate job in the pipeline.
> **Result:** Review processing became fully async with zero impact on API response times. The system handled 10x more reviews with automatic retry on failures.

### Q: How do you handle disagreements with team members?
> Focus on data and outcomes, not opinions. Propose A/B testing or prototyping when possible. Ultimately defer to whoever owns the decision, but ensure concerns are documented.

### Q: How do you stay updated with new technologies?
> I follow tech blogs, participate in open-source, experiment with new tools in side projects, and actively use AI-assisted development (Claude Code, Codex) to explore cutting-edge patterns.

### Q: What's your biggest weakness?
> "I sometimes over-engineer solutions — anticipating future requirements that may never come. I've learned to focus on shipping the simplest solution first and iterating based on real user feedback."

### Q: Why should we hire you?
> "I bring a rare combination of deep backend expertise, hands-on AI/LLM integration experience, and proven ability to architect production systems at scale. I've built systems handling thousands of businesses with multi-tenant isolation, integrated multiple AI providers, and managed full AWS infrastructure. I don't just write code — I design systems that ship, scale, and maintain."

---

# QUICK REFERENCE CHEAT SHEET

## One-Liner Definitions

| Term | Definition |
|------|-----------|
| **REST** | Architectural style using HTTP methods for CRUD on resources |
| **GraphQL** | Query language where clients specify exactly what data they need |
| **JWT** | Encoded token containing user claims, signed with a secret |
| **OAuth** | Authorization protocol — lets apps access user data without passwords |
| **RBAC** | Access control based on user roles (admin, user, viewer) |
| **ACID** | Atomicity, Consistency, Isolation, Durability — DB transaction guarantees |
| **ORM** | Maps database tables to code objects (Sequelize, TypeORM) |
| **Middleware** | Function between request and response that processes/modifies data |
| **WebSocket** | Full-duplex communication channel over a single TCP connection |
| **Redis** | In-memory key-value store for caching, queues, pub/sub |
| **Docker** | Containerization platform — packages app + dependencies together |
| **CI/CD** | Automated build, test, and deployment pipeline |
| **RAG** | Feed retrieved documents to an LLM for accurate, grounded answers |
| **Fine-tuning** | Further training a pre-trained model on your specific data |
| **Embedding** | Converting text to a numerical vector for similarity search |
| **Vector DB** | Database optimized for storing and searching embeddings |
| **Multi-tenant** | Single app serving multiple isolated customers |
| **Microservices** | Architecture where each feature is an independent service |
| **Load Balancer** | Distributes incoming traffic across multiple servers |
| **Reverse Proxy** | Server (Nginx) that forwards requests to backend servers |

---

## Your Resume Projects — Quick Reference for Interviews

| Project | Stack | Key Achievement |
|---------|-------|----------------|
| **Obenan/Global Software Consulting** | Node.js, PostgreSQL, MongoDB, AWS, BullMQ, Stripe | 100+ models, 400+ migrations, multi-tenant SaaS, AI sentiment analysis |
| **GoTo API** | NestJS, TypeScript, Next.js | AI transcription with AWS Transcribe + GPT-4o, webhook processing |
| **AT-Function-GPT** | NestJS, TypeScript, Next.js 14 | Automated ticket analysis with GPT-4o-mini, Azure Key Vault |
| **NowVPlay** | Node.js, MongoDB, Socket.IO, AWS | Real-time sports platform, content moderation, CI/CD |
| **Zamulk** | Node.js, MongoDB, Socket.IO | Real estate portal (100+ cities), geospatial search, live bidding |
| **US Ride** | Node.js, Socket.IO, Stripe | Real-time ride tracking, peer-to-peer payments |

---

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

---

*Last Updated: April 2026*
*Prepared for: Saad Sohail — Senior Full-Stack Engineer*
*Total Topics: 36 | Total Pages: ~60+ when printed*
