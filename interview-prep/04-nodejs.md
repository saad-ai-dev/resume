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

