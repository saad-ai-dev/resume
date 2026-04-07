# 37. NODE.JS — COMPLETE CONCEPTS & INTERVIEW QUESTIONS

This section covers **every Node.js concept** you could be asked in an interview — from fundamentals to advanced internals.

---

## 37.1 Node.js Fundamentals — The Basics

### What is Node.js?
Node.js is a **JavaScript runtime environment** built on Google Chrome's V8 JavaScript engine. It lets you run JavaScript outside the browser — on servers, desktops, IoT devices, etc.

**Key characteristics:**
- **Open source** — maintained by the OpenJS Foundation
- **Cross-platform** — runs on Windows, macOS, Linux
- **Single-threaded** — one thread for JavaScript execution
- **Non-blocking I/O** — doesn't wait for I/O operations to complete
- **Event-driven** — uses events and callbacks to handle async operations
- **Built on V8** — Google's high-performance JavaScript engine (written in C++)
- **Built on libuv** — C library that provides the event loop and async I/O

### Node.js vs Browser JavaScript

| Feature | Browser JS | Node.js |
|---------|-----------|---------|
| DOM access | Yes (`document`, `window`) | No |
| File system | No | Yes (`fs` module) |
| Network servers | No | Yes (`http`, `net`) |
| `window` object | Yes | No (`global` instead) |
| `require()` / `import` | `import` only | Both |
| Modules | ES Modules | CommonJS + ES Modules |
| Thread access | Web Workers | Worker Threads, Child Processes |
| Package manager | No (uses CDN) | npm, yarn, pnpm |

### Global Objects in Node.js

```javascript
// These are available everywhere without require()

__dirname   // Absolute path to current file's directory
__filename  // Absolute path to current file
process     // Info about the current Node.js process
global      // The global namespace object (like window in browser)
module      // Reference to the current module
exports     // Shorthand for module.exports
require()   // Function to import modules
console     // Console output methods
Buffer      // Handle binary data
setTimeout, setInterval, setImmediate  // Timers
```

```javascript
console.log(__dirname);  // /Users/saad/projects/api/src
console.log(__filename); // /Users/saad/projects/api/src/app.js

// process object
console.log(process.pid);          // Process ID
console.log(process.version);      // Node.js version (v20.11.0)
console.log(process.platform);     // 'darwin', 'linux', 'win32'
console.log(process.arch);         // 'x64', 'arm64'
console.log(process.uptime());     // Seconds since process started
console.log(process.memoryUsage()); // RAM usage
console.log(process.cwd());        // Current working directory
console.log(process.env.NODE_ENV); // Environment variables
process.exit(0);                   // Exit with success code
process.exit(1);                   // Exit with error code
```

---

## 37.2 Module System — CommonJS vs ES Modules

### CommonJS (CJS) — The Original Node.js Way

```javascript
// math.js — Exporting
function add(a, b) { return a + b; }
function multiply(a, b) { return a * b; }

module.exports = { add, multiply };
// OR
exports.add = add;
exports.multiply = multiply;

// app.js — Importing
const { add, multiply } = require('./math');
const fs = require('fs');           // Built-in module
const express = require('express'); // npm package
```

### ES Modules (ESM) — The Modern Way

```javascript
// math.mjs (or .js with "type": "module" in package.json)
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }
export default class Calculator { }

// app.mjs — Importing
import Calculator, { add, multiply } from './math.mjs';
import fs from 'fs';
import { readFile } from 'fs/promises';
```

### CommonJS vs ES Modules Comparison

| Feature | CommonJS (`require`) | ES Modules (`import`) |
|---------|---------------------|----------------------|
| Loading | Synchronous | Asynchronous |
| When parsed | Runtime | Compile time (static analysis) |
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Dynamic imports | `require(variable)` ✓ | `import(variable)` ✓ (async) |
| Tree shaking | No (can't analyze at build time) | Yes (dead code elimination) |
| File extension | `.js`, `.cjs` | `.mjs` or `.js` with `"type": "module"` |
| `__dirname` | Available | Not available (use `import.meta.url`) |
| Top-level await | No | Yes |
| Used by | Most existing Node.js code | New projects, frontend |
| Circular deps | Returns partial exports | Handled via live bindings |

```javascript
// Dynamic import (works in both)
const module = await import('./heavy-module.js');

// ESM equivalent of __dirname
import { fileURLToPath } from 'url';
import { dirname } from 'path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

### How require() Works Internally

```
require('./math')
    │
    ▼
1. RESOLVE: Find the file
   ./math → ./math.js → ./math/index.js → ./math.json → ./math.node
    │
    ▼
2. CHECK CACHE: Was this module already loaded?
   require.cache['/absolute/path/math.js']
   If yes → return cached module.exports (SINGLETON behavior)
    │
    ▼
3. LOAD: Read the file contents
    │
    ▼
4. WRAP: Node wraps your code in a function:
   (function(exports, require, module, __filename, __dirname) {
     // YOUR CODE HERE
   });
   This is WHY you have access to these 5 variables!
    │
    ▼
5. EXECUTE: Run the wrapped function
    │
    ▼
6. CACHE: Store module.exports in require.cache
    │
    ▼
7. RETURN: Return module.exports to the caller
```

**Interview Q: Why does require() return the same object every time?**
Because of step 2 — Node caches modules. The first `require()` loads and caches it. All subsequent `require()` calls return the cached version. This is why `module.exports = new Database()` creates a singleton.

---

## 37.3 The Process Object — Complete Guide

```javascript
// ─── ENVIRONMENT ───
process.env.NODE_ENV        // 'development', 'production', 'test'
process.env.PORT            // Custom env variables
process.env.DATABASE_URL    // Database connection string

// ─── PROCESS INFO ───
process.pid                 // Process ID
process.ppid                // Parent process ID
process.title               // Process name (shows in ps/top)
process.version             // Node.js version
process.versions            // { v8: '...', openssl: '...', ... }
process.platform            // 'darwin', 'linux', 'win32'
process.arch                // 'x64', 'arm64'
process.argv                // Command line arguments
process.execPath            // Path to Node.js binary

// ─── MEMORY ───
process.memoryUsage();
// {
//   rss: 30000000,          ← Resident Set Size (total memory allocated)
//   heapTotal: 18000000,    ← V8 heap total
//   heapUsed: 15000000,     ← V8 heap actually used
//   external: 500000,       ← C++ objects bound to JS
//   arrayBuffers: 100000    ← ArrayBuffer and SharedArrayBuffer
// }

// ─── CPU ───
process.cpuUsage();         // { user: microseconds, system: microseconds }

// ─── WORKING DIRECTORY ───
process.cwd()               // Current working directory
process.chdir('/new/path')  // Change working directory

// ─── COMMAND LINE ARGS ───
// node app.js --port 3000 --env production
process.argv[0]  // '/usr/bin/node'
process.argv[1]  // '/app/app.js'
process.argv[2]  // '--port'
process.argv[3]  // '3000'

// ─── EXIT ───
process.exit(0)             // Success
process.exit(1)             // Error
process.exitCode = 1;       // Set exit code without immediate exit

// ─── SIGNALS ───
process.on('SIGINT', () => {    // Ctrl+C
  console.log('Shutting down gracefully...');
  server.close(() => process.exit(0));
});

process.on('SIGTERM', () => {   // kill command
  console.log('SIGTERM received');
  process.exit(0);
});

// ─── STDIN/STDOUT/STDERR ───
process.stdout.write('Hello\n');  // Same as console.log but no newline
process.stderr.write('Error!\n'); // Write to stderr
process.stdin.on('data', (data) => { /* read user input */ });
```

---

## 37.4 File System (fs) Module — Complete Guide

### Synchronous vs Asynchronous vs Promise-based

```javascript
const fs = require('fs');
const fsPromises = require('fs/promises'); // OR fs.promises

// ═══ SYNCHRONOUS (blocks the event loop — avoid in production) ═══
const data = fs.readFileSync('file.txt', 'utf-8');

// ═══ CALLBACK-BASED (original async way) ═══
fs.readFile('file.txt', 'utf-8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// ═══ PROMISE-BASED (modern, recommended) ═══
const data = await fsPromises.readFile('file.txt', 'utf-8');

// ═══ STREAM-BASED (for large files) ═══
const stream = fs.createReadStream('huge-file.csv', 'utf-8');
stream.on('data', (chunk) => { /* process chunk */ });
```

### File Operations

```javascript
const fs = require('fs/promises');

// ─── READ ───
const content = await fs.readFile('data.json', 'utf-8');
const json = JSON.parse(content);

// ─── WRITE (overwrites) ───
await fs.writeFile('output.txt', 'Hello World', 'utf-8');
await fs.writeFile('data.json', JSON.stringify(data, null, 2));

// ─── APPEND ───
await fs.appendFile('log.txt', `${new Date().toISOString()} - Event occurred\n`);

// ─── DELETE ───
await fs.unlink('temp-file.txt');   // Delete file
await fs.rm('folder', { recursive: true, force: true }); // Delete folder

// ─── RENAME / MOVE ───
await fs.rename('old.txt', 'new.txt');
await fs.rename('file.txt', 'archive/file.txt'); // Move

// ─── COPY ───
await fs.copyFile('source.txt', 'dest.txt');

// ─── CHECK EXISTS ───
try {
  await fs.access('file.txt');
  console.log('File exists');
} catch {
  console.log('File does not exist');
}

// ─── FILE INFO ───
const stats = await fs.stat('file.txt');
console.log(stats.size);        // File size in bytes
console.log(stats.isFile());     // true
console.log(stats.isDirectory());// false
console.log(stats.birthtime);   // Creation date
console.log(stats.mtime);       // Last modified date
```

### Directory Operations

```javascript
// ─── CREATE DIRECTORY ───
await fs.mkdir('uploads');
await fs.mkdir('deep/nested/folder', { recursive: true }); // Create parents

// ─── READ DIRECTORY ───
const files = await fs.readdir('src');           // ['app.js', 'utils.js']
const filesWithTypes = await fs.readdir('src', { withFileTypes: true });
filesWithTypes.forEach(dirent => {
  console.log(dirent.name, dirent.isDirectory() ? 'DIR' : 'FILE');
});

// ─── WATCH FOR CHANGES ───
const watcher = fs.watch('src', { recursive: true }, (eventType, filename) => {
  console.log(`${eventType}: ${filename}`);
});
```

---

## 37.5 Path Module — Complete Guide

```javascript
const path = require('path');

// ─── JOIN PATHS (cross-platform) ───
path.join('/users', 'saad', 'projects', 'api');
// Linux:   '/users/saad/projects/api'
// Windows: '\users\saad\projects\api'

// ─── RESOLVE (absolute path) ───
path.resolve('src', 'app.js');  // '/Users/saad/project/src/app.js'

// ─── PARSE PATH ───
const parsed = path.parse('/home/saad/project/app.js');
// {
//   root: '/',
//   dir: '/home/saad/project',
//   base: 'app.js',
//   name: 'app',
//   ext: '.js'
// }

// ─── GET PARTS ───
path.dirname('/home/saad/app.js');   // '/home/saad'
path.basename('/home/saad/app.js');  // 'app.js'
path.basename('/home/saad/app.js', '.js'); // 'app'
path.extname('app.js');              // '.js'
path.extname('app.test.js');         // '.js'

// ─── NORMALIZE (clean up messy paths) ───
path.normalize('/users//saad/../saad/./project');
// '/users/saad/project'

// ─── IS ABSOLUTE ───
path.isAbsolute('/home/saad');  // true
path.isAbsolute('./src');       // false

// ─── RELATIVE PATH ───
path.relative('/users/saad/project', '/users/saad/docs');
// '../docs'
```

---

## 37.6 HTTP Module — Building Servers Without Express

```javascript
const http = require('http');

// ─── BASIC SERVER ───
const server = http.createServer((req, res) => {
  // Request info
  console.log(req.method);     // 'GET', 'POST', etc.
  console.log(req.url);        // '/api/users?page=1'
  console.log(req.headers);    // { 'content-type': '...', ... }

  // Parse URL
  const url = new URL(req.url, `http://${req.headers.host}`);
  const pathname = url.pathname;   // '/api/users'
  const page = url.searchParams.get('page'); // '1'

  // Routing
  if (req.method === 'GET' && pathname === '/api/users') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ users: [] }));
  }
  else if (req.method === 'POST' && pathname === '/api/users') {
    let body = '';
    req.on('data', (chunk) => { body += chunk; });
    req.on('end', () => {
      const user = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ created: user }));
    });
  }
  else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});

// ─── MAKE HTTP REQUEST (client) ───
const https = require('https');

https.get('https://api.example.com/data', (res) => {
  let data = '';
  res.on('data', (chunk) => { data += chunk; });
  res.on('end', () => {
    console.log(JSON.parse(data));
  });
}).on('error', (err) => {
  console.error('Error:', err);
});
```

**Interview Q: Why use Express instead of the http module?**
The http module is low-level — you manually parse URLs, handle routing, parse bodies, etc. Express adds routing, middleware, body parsing, error handling, and templating on top of http. For production APIs, Express (or NestJS) saves massive time.

---

## 37.7 Child Processes — Running External Commands

Node.js can spawn child processes to run shell commands, scripts, or even other Node.js files.

### Four Methods

```javascript
const { exec, execSync, spawn, fork } = require('child_process');

// ═══ exec — Run a shell command (buffered output) ═══
// Good for: short commands, small output
exec('ls -la', (error, stdout, stderr) => {
  if (error) { console.error(`Error: ${error.message}`); return; }
  console.log(stdout);
});

// Promise version
const { promisify } = require('util');
const execAsync = promisify(exec);
const { stdout } = await execAsync('git log --oneline -5');

// ═══ execSync — Synchronous version (blocks event loop) ═══
const result = execSync('npm --version').toString().trim();

// ═══ spawn — Run a command (streamed output) ═══
// Good for: long-running processes, large output
const child = spawn('ffmpeg', ['-i', 'input.mp4', '-o', 'output.mp4']);

child.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});
child.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});
child.on('close', (code) => {
  console.log(`Process exited with code ${code}`);
});

// ═══ fork — Spawn a new Node.js process with IPC channel ═══
// Good for: CPU-intensive work in a separate Node.js process
// parent.js
const child = fork('./worker.js');
child.send({ task: 'processData', data: largeArray });
child.on('message', (result) => {
  console.log('Worker result:', result);
});

// worker.js
process.on('message', (msg) => {
  const result = heavyComputation(msg.data);
  process.send(result); // Send back to parent
});
```

### exec vs spawn vs fork

| Feature | `exec` | `spawn` | `fork` |
|---------|--------|---------|--------|
| Output | Buffered (all at once) | Streamed (chunk by chunk) | Streamed + IPC |
| Shell | Yes (runs in shell) | No (direct command) | No (runs Node.js file) |
| Best for | Short commands | Long processes, large output | Node.js worker processes |
| Max output | ~1MB default | Unlimited | Unlimited |
| IPC channel | No | No | Yes (`send()`/`on('message')`) |

---

## 37.8 Crypto Module

```javascript
const crypto = require('crypto');

// ─── HASH (one-way, irreversible) ───
const hash = crypto.createHash('sha256')
  .update('password123')
  .digest('hex');
// 'ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f'

// ─── HMAC (hash with secret key) ───
const hmac = crypto.createHmac('sha256', 'my-secret')
  .update('data to sign')
  .digest('hex');

// ─── RANDOM VALUES ───
const randomBytes = crypto.randomBytes(32).toString('hex');  // Random string
const uuid = crypto.randomUUID();  // 'a1b2c3d4-e5f6-...'

// ─── ENCRYPT / DECRYPT (AES-256) ───
const algorithm = 'aes-256-cbc';
const key = crypto.randomBytes(32);
const iv = crypto.randomBytes(16);

function encrypt(text) {
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  let encrypted = cipher.update(text, 'utf-8', 'hex');
  encrypted += cipher.final('hex');
  return { iv: iv.toString('hex'), data: encrypted };
}

function decrypt(encrypted) {
  const decipher = crypto.createDecipheriv(algorithm, key,
    Buffer.from(encrypted.iv, 'hex'));
  let decrypted = decipher.update(encrypted.data, 'hex', 'utf-8');
  decrypted += decipher.final('utf-8');
  return decrypted;
}

const encrypted = encrypt('secret data');
const original = decrypt(encrypted); // 'secret data'

// ─── COMPARE (timing-safe) ───
// Prevents timing attacks on token/password comparison
const a = Buffer.from('token123');
const b = Buffer.from('token123');
crypto.timingSafeEqual(a, b); // true (constant-time comparison)
```

---

## 37.9 URL & Query String Parsing

```javascript
// ═══ Modern URL API (recommended) ═══
const url = new URL('https://api.example.com:8080/api/users?page=1&sort=name#section');

url.protocol   // 'https:'
url.hostname   // 'api.example.com'
url.port       // '8080'
url.pathname   // '/api/users'
url.search     // '?page=1&sort=name'
url.hash       // '#section'
url.origin     // 'https://api.example.com:8080'

// Search params
url.searchParams.get('page');      // '1'
url.searchParams.get('sort');      // 'name'
url.searchParams.has('page');      // true
url.searchParams.set('limit', 20);
url.searchParams.delete('sort');
url.searchParams.toString();       // 'page=1&limit=20'

// Build URL
const apiUrl = new URL('/api/reviews', 'https://myapp.com');
apiUrl.searchParams.set('locationId', '123');
apiUrl.searchParams.set('status', 'active');
console.log(apiUrl.toString());
// 'https://myapp.com/api/reviews?locationId=123&status=active'

// ═══ Legacy querystring module ═══
const qs = require('querystring');
qs.parse('page=1&sort=name');         // { page: '1', sort: 'name' }
qs.stringify({ page: 1, sort: 'name' }); // 'page=1&sort=name'
```

---

## 37.10 OS Module

```javascript
const os = require('os');

os.platform()     // 'darwin', 'linux', 'win32'
os.arch()         // 'x64', 'arm64'
os.cpus()         // Array of CPU core info
os.cpus().length  // Number of CPU cores (used for clustering)
os.totalmem()     // Total RAM in bytes
os.freemem()      // Free RAM in bytes
os.hostname()     // Computer name
os.homedir()      // User home directory
os.tmpdir()       // Temp directory
os.uptime()       // System uptime in seconds
os.networkInterfaces()  // Network interface info (IP addresses)
os.type()         // 'Darwin', 'Linux', 'Windows_NT'
os.release()      // OS version

// Memory in human-readable format
console.log(`Total: ${(os.totalmem() / 1024 / 1024 / 1024).toFixed(2)} GB`);
console.log(`Free: ${(os.freemem() / 1024 / 1024 / 1024).toFixed(2)} GB`);
```

---

## 37.11 Streams — Complete Guide

Streams are collections of data that might not be available all at once. They let you process data **piece by piece** without loading everything into memory.

### Four Types of Streams

```
Readable  →  data flows OUT  →  fs.createReadStream, HTTP request, process.stdin
Writable  →  data flows IN   →  fs.createWriteStream, HTTP response, process.stdout
Duplex    →  both directions  →  TCP socket, WebSocket
Transform →  modify data      →  zlib.createGzip, crypto.createCipher
```

### Stream Events

```javascript
// Readable stream events
readable.on('data', (chunk) => { });    // New data available
readable.on('end', () => { });          // No more data
readable.on('error', (err) => { });     // Error occurred
readable.on('close', () => { });        // Stream closed
readable.on('readable', () => { });     // Data available to read

// Writable stream events
writable.on('drain', () => { });        // Ready for more data
writable.on('finish', () => { });       // All data flushed
writable.on('error', (err) => { });     // Error occurred
writable.on('close', () => { });        // Stream closed
```

### Practical Examples

```javascript
const fs = require('fs');
const { Transform, pipeline } = require('stream');
const { promisify } = require('util');
const zlib = require('zlib');
const pipelineAsync = promisify(pipeline);

// ─── READ LARGE FILE ───
const readStream = fs.createReadStream('huge-data.csv', {
  encoding: 'utf-8',
  highWaterMark: 64 * 1024,  // 64KB chunks (default is 16KB)
});

let lineCount = 0;
readStream.on('data', (chunk) => {
  lineCount += chunk.split('\n').length;
});
readStream.on('end', () => {
  console.log(`Total lines: ${lineCount}`);
});

// ─── PIPE (connect readable to writable) ───
fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));

// ─── TRANSFORM STREAM (modify data as it flows) ───
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

fs.createReadStream('input.txt')
  .pipe(upperCaseTransform)
  .pipe(fs.createWriteStream('output.txt'));

// ─── COMPRESS FILE ───
await pipelineAsync(
  fs.createReadStream('large-file.json'),
  zlib.createGzip(),
  fs.createWriteStream('large-file.json.gz')
);

// ─── DECOMPRESS FILE ───
await pipelineAsync(
  fs.createReadStream('large-file.json.gz'),
  zlib.createGunzip(),
  fs.createWriteStream('large-file.json')
);

// ─── HTTP STREAMING RESPONSE ───
app.get('/download', (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename=data.csv');

  const readStream = fs.createReadStream('data.csv');
  readStream.pipe(res); // Stream directly to HTTP response
});

// ─── CUSTOM READABLE STREAM ───
const { Readable } = require('stream');

class CounterStream extends Readable {
  constructor(max) {
    super();
    this.max = max;
    this.current = 0;
  }

  _read() {
    if (this.current <= this.max) {
      this.push(`${this.current}\n`);
      this.current++;
    } else {
      this.push(null); // Signal end of stream
    }
  }
}

const counter = new CounterStream(1000000);
counter.pipe(fs.createWriteStream('numbers.txt'));
```

### Why Use pipeline() Instead of pipe()?

```javascript
// BAD — pipe() doesn't handle errors properly
readStream.pipe(transform).pipe(writeStream);
// If transform throws an error, readStream is NOT destroyed → memory leak!

// GOOD — pipeline() properly destroys all streams on error
const { pipeline } = require('stream');

pipeline(
  readStream,
  transform,
  writeStream,
  (err) => {
    if (err) console.error('Pipeline failed:', err);
    else console.log('Pipeline succeeded');
  }
);
```

---

## 37.12 Error Handling — Complete Guide

### Types of Errors in Node.js

```javascript
// ─── 1. STANDARD ERRORS ───
new Error('Something went wrong');
new TypeError('Expected string, got number');
new RangeError('Value out of range');
new ReferenceError('Variable not defined');
new SyntaxError('Invalid JSON');

// ─── 2. SYSTEM ERRORS ───
// Errors from the OS (file not found, permission denied, network error)
// err.code: 'ENOENT' (file not found), 'EACCES' (permission), 'ECONNREFUSED' (connection)

// ─── 3. CUSTOM ERRORS ───
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'AppError';
    this.statusCode = statusCode;
    this.isOperational = true; // Known error vs unexpected crash
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource) {
    super(`${resource} not found`, 404);
    this.name = 'NotFoundError';
  }
}

class ValidationError extends AppError {
  constructor(field, message) {
    super(`Validation failed: ${field} - ${message}`, 400);
    this.name = 'ValidationError';
    this.field = field;
  }
}

// Usage
throw new NotFoundError('User');
throw new ValidationError('email', 'must be a valid email');
```

### Error Handling Patterns

```javascript
// ─── 1. TRY/CATCH (async/await) — RECOMMENDED ───
async function getUser(id) {
  try {
    const user = await db.users.findByPk(id);
    if (!user) throw new NotFoundError('User');
    return user;
  } catch (error) {
    if (error.isOperational) throw error; // Re-throw known errors
    console.error('Unexpected error:', error);
    throw new AppError('Internal server error', 500);
  }
}

// ─── 2. ERROR-FIRST CALLBACKS (legacy pattern) ───
fs.readFile('data.txt', (err, data) => {
  if (err) {
    console.error('Failed to read:', err);
    return;
  }
  console.log(data);
});

// ─── 3. PROMISE .catch() ───
fetchData()
  .then(data => processData(data))
  .catch(err => console.error('Failed:', err));

// ─── 4. EXPRESS ERROR MIDDLEWARE ───
// Catch async errors in Express
function asyncHandler(fn) {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await getUser(req.params.id); // If this throws, it goes to error middleware
  res.json(user);
}));

// Error middleware (MUST have 4 parameters)
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.isOperational ? err.message : 'Internal server error';

  // Log unexpected errors
  if (!err.isOperational) {
    console.error('UNEXPECTED ERROR:', err);
    Sentry.captureException(err);
  }

  res.status(statusCode).json({
    success: false,
    error: message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack }),
  });
});

// ─── 5. GLOBAL HANDLERS (last resort) ───
process.on('uncaughtException', (err) => {
  console.error('UNCAUGHT EXCEPTION:', err);
  // MUST exit — application state is corrupted
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('UNHANDLED REJECTION:', reason);
  // In Node.js 15+, this also causes exit by default
});
```

### Operational vs Programming Errors

| Type | Example | Handling |
|------|---------|----------|
| **Operational** | File not found, API timeout, invalid input, DB down | Handle gracefully (return error to user) |
| **Programming** | TypeError, null reference, undefined variable | Fix the code (crash is acceptable) |

---

## 37.13 Memory Management & Garbage Collection

### How V8 Manages Memory

```
┌─────────────────────────────────────┐
│              V8 HEAP                 │
├─────────────────────────────────────┤
│  New Space (Young Generation)        │  ← Short-lived objects
│  ┌──────────┐ ┌──────────┐          │     Scavenging GC (fast, frequent)
│  │ Semi-     │ │ Semi-     │         │
│  │ space A   │ │ space B   │         │
│  └──────────┘ └──────────┘          │
├─────────────────────────────────────┤
│  Old Space (Old Generation)          │  ← Long-lived objects
│  ┌────────────────────────────┐     │     Mark-Sweep-Compact GC (slow, rare)
│  │                            │     │
│  │  Objects that survived      │     │
│  │  multiple GC cycles         │     │
│  └────────────────────────────┘     │
├─────────────────────────────────────┤
│  Large Object Space                  │  ← Objects > 1MB
│  Code Space                          │  ← JIT compiled code
│  Map Space                           │  ← Hidden classes / shapes
└─────────────────────────────────────┘
```

### Common Memory Leaks and How to Fix Them

```javascript
// ─── LEAK 1: Global variables ───
// BAD — accumulates forever
const cache = {};
app.get('/data/:id', (req, res) => {
  cache[req.params.id] = fetchData(req.params.id); // Never cleaned up!
});

// GOOD — use LRU cache with max size
const LRU = require('lru-cache');
const cache = new LRU({ max: 500 }); // Max 500 entries

// ─── LEAK 2: Event listeners not removed ───
// BAD — adds new listener on every request
app.get('/stream', (req, res) => {
  emitter.on('data', (data) => res.write(data));
  // Never removed! Listeners accumulate.
});

// GOOD — remove listener when done
app.get('/stream', (req, res) => {
  const handler = (data) => res.write(data);
  emitter.on('data', handler);
  req.on('close', () => emitter.removeListener('data', handler));
});

// ─── LEAK 3: Closures holding references ───
// BAD — closure holds reference to huge array
function processData() {
  const hugeArray = new Array(1000000).fill('data');
  return function() {
    return hugeArray.length; // Closure prevents GC of hugeArray
  };
}

// GOOD — extract what you need
function processData() {
  const hugeArray = new Array(1000000).fill('data');
  const length = hugeArray.length;
  return function() {
    return length; // Only holds the number, not the array
  };
}

// ─── LEAK 4: Timers not cleared ───
// BAD
setInterval(() => {
  // This runs forever even if not needed
}, 1000);

// GOOD
const timer = setInterval(() => { ... }, 1000);
// Clear when done
clearInterval(timer);

// ─── DETECTING MEMORY LEAKS ───
// Log memory usage periodically
setInterval(() => {
  const used = process.memoryUsage();
  console.log(`Heap: ${(used.heapUsed / 1024 / 1024).toFixed(2)} MB`);
}, 10000);

// V8 flags for debugging
// node --max-old-space-size=4096 app.js  ← Increase heap to 4GB
// node --inspect app.js                  ← Enable Chrome DevTools
// node --expose-gc app.js                ← Allow manual GC (global.gc())
```

---

## 37.14 Performance Optimization

### Measuring Performance

```javascript
// ─── console.time ───
console.time('db-query');
const users = await db.users.findAll();
console.timeEnd('db-query'); // db-query: 45.123ms

// ─── Performance hooks ───
const { performance, PerformanceObserver } = require('perf_hooks');

performance.mark('start');
await heavyOperation();
performance.mark('end');
performance.measure('Heavy Op', 'start', 'end');

const obs = new PerformanceObserver((list) => {
  const entry = list.getEntries()[0];
  console.log(`${entry.name}: ${entry.duration.toFixed(2)}ms`);
});
obs.observe({ entryTypes: ['measure'] });

// ─── process.hrtime.bigint() — Nanosecond precision ───
const start = process.hrtime.bigint();
await someOperation();
const end = process.hrtime.bigint();
console.log(`Duration: ${Number(end - start) / 1_000_000}ms`);
```

### Optimization Techniques

```javascript
// ─── 1. USE STREAMS FOR LARGE DATA ───
// BAD — loads entire file into memory
const data = await fs.readFile('1GB-file.csv', 'utf-8');
const lines = data.split('\n').map(parse);

// GOOD — process line by line
const readline = require('readline');
const rl = readline.createInterface({
  input: fs.createReadStream('1GB-file.csv'),
});
for await (const line of rl) {
  processLine(line);
}

// ─── 2. CLUSTER MODULE FOR CPU UTILIZATION ───
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const numCPUs = os.cpus().length;
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork(); // One worker per CPU core
  }
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, restarting...`);
    cluster.fork(); // Auto-restart
  });
} else {
  app.listen(3000);
}

// ─── 3. WORKER THREADS FOR CPU-INTENSIVE TASKS ───
const { Worker, isMainThread, workerData, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename, {
    workerData: { numbers: [1, 2, 3, 4, 5] }
  });
  worker.on('message', (result) => console.log('Sum:', result));
} else {
  const sum = workerData.numbers.reduce((a, b) => a + b, 0);
  parentPort.postMessage(sum);
}

// ─── 4. CONNECTION POOLING ───
// BAD — new connection per query
async function getUser(id) {
  const client = new Client(); // New connection every time!
  await client.connect();
  const result = await client.query('SELECT * FROM users WHERE id = $1', [id]);
  client.end();
  return result.rows[0];
}

// GOOD — reuse connections from pool
const { Pool } = require('pg');
const pool = new Pool({ max: 20 }); // 20 reusable connections

async function getUser(id) {
  const result = await pool.query('SELECT * FROM users WHERE id = $1', [id]);
  return result.rows[0];
}

// ─── 5. CACHING WITH REDIS ───
// Already covered in Section 15

// ─── 6. COMPRESSION ───
const compression = require('compression');
app.use(compression()); // Gzip all responses

// ─── 7. AVOID SYNC OPERATIONS ───
// BAD — blocks event loop
const data = fs.readFileSync('file.txt');
const hash = crypto.pbkdf2Sync(password, salt, 100000, 64, 'sha512');

// GOOD — async version
const data = await fs.promises.readFile('file.txt');
const hash = await util.promisify(crypto.pbkdf2)(password, salt, 100000, 64, 'sha512');
```

---

## 37.15 PM2 — Production Process Manager

```bash
# Install
npm install -g pm2

# Start application
pm2 start app.js
pm2 start app.js --name "api-server"
pm2 start app.js -i max          # Cluster mode (all CPUs)
pm2 start app.js -i 4            # 4 instances

# Management
pm2 list                          # List all processes
pm2 stop all                      # Stop all
pm2 restart all                   # Restart all
pm2 delete all                    # Remove all from list
pm2 monit                         # Real-time monitoring dashboard

# Logs
pm2 logs                          # View all logs
pm2 logs api-server               # View specific app logs
pm2 flush                         # Clear all logs

# Zero-downtime reload
pm2 reload all                    # Graceful reload (no downtime)

# Startup (auto-start on server reboot)
pm2 startup                       # Generate startup script
pm2 save                          # Save current process list

# Ecosystem file (pm2.config.js)
module.exports = {
  apps: [{
    name: 'api',
    script: './dist/main.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
    max_memory_restart: '1G',
    error_file: './logs/error.log',
    out_file: './logs/output.log',
  }]
};
```

---

## 37.16 Security Best Practices for Node.js

```javascript
// ─── 1. NEVER USE eval() OR Function() ───
// BAD — allows code injection
eval(userInput);
new Function(userInput)();

// ─── 2. VALIDATE ALL INPUT ───
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).max(128).required(),
  age: Joi.number().integer().min(0).max(150),
});

const { error, value } = schema.validate(req.body);
if (error) return res.status(400).json({ error: error.message });

// ─── 3. PREVENT SQL INJECTION ───
// BAD
db.query(`SELECT * FROM users WHERE email = '${email}'`);

// GOOD — parameterized queries
db.query('SELECT * FROM users WHERE email = $1', [email]);

// ─── 4. PREVENT NOSQL INJECTION ───
// BAD — user could send { "$gt": "" } as email
db.users.find({ email: req.body.email });

// GOOD — validate type
if (typeof req.body.email !== 'string') throw new Error('Invalid email');

// ─── 5. SECURITY HEADERS ───
const helmet = require('helmet');
app.use(helmet()); // Sets 15+ security headers

// ─── 6. RATE LIMITING ───
const rateLimit = require('express-rate-limit');
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }));

// ─── 7. CORS ───
const cors = require('cors');
app.use(cors({
  origin: ['https://myapp.com'], // Not '*' in production!
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true,
}));

// ─── 8. SANITIZE OUTPUT (prevent XSS) ───
const xss = require('xss');
const cleanInput = xss(userInput);

// ─── 9. USE HTTPS ───
// In production, always terminate TLS at the load balancer (Nginx/ALB)

// ─── 10. KEEP DEPENDENCIES UPDATED ───
// npm audit                    ← Check for vulnerabilities
// npm audit fix                ← Auto-fix vulnerabilities
// Use Dependabot or Snyk for automated updates

// ─── 11. DON'T EXPOSE STACK TRACES IN PRODUCTION ───
if (process.env.NODE_ENV === 'production') {
  app.use((err, req, res, next) => {
    res.status(500).json({ error: 'Internal server error' });
    // Don't send err.stack to client!
  });
}

// ─── 12. ENVIRONMENT VARIABLES ───
// NEVER hardcode secrets in code
// Use .env files (with dotenv) or AWS Secrets Manager / Azure Key Vault
require('dotenv').config();
const secret = process.env.JWT_SECRET;
```

---

## 37.17 Debugging Node.js

```bash
# ─── Chrome DevTools ───
node --inspect app.js                 # Start with debugger
node --inspect-brk app.js             # Break on first line
# Open chrome://inspect in Chrome

# ─── VS Code Debugging ───
# Create .vscode/launch.json:
{
  "type": "node",
  "request": "launch",
  "name": "Debug App",
  "program": "${workspaceFolder}/src/app.js",
  "env": { "NODE_ENV": "development" }
}

# ─── Built-in Debugger ───
node inspect app.js                   # CLI debugger
# Commands: cont (c), next (n), step (s), out (o), repl, watch('expr')
```

```javascript
// ─── Debug Logging ───
const debug = require('debug');
const log = debug('app:server');
const dbLog = debug('app:database');

log('Server starting on port 3000');    // Only shows if DEBUG=app:server
dbLog('Connected to database');          // Only shows if DEBUG=app:database

// Run with: DEBUG=app:* node app.js    (show all app logs)
// Run with: DEBUG=app:server node app.js (only server logs)
```

---

## 37.18 Environment Variables & Configuration

```javascript
// ─── dotenv (development) ───
// .env file (NEVER commit to git!)
// NODE_ENV=development
// PORT=3000
// DATABASE_URL=postgres://user:pass@localhost:5432/mydb
// JWT_SECRET=super-secret-key
// REDIS_URL=redis://localhost:6379

require('dotenv').config();

// ─── Config module pattern ───
// config.js
const config = {
  development: {
    port: 3000,
    db: process.env.DATABASE_URL || 'postgres://localhost:5432/dev',
    redis: process.env.REDIS_URL || 'redis://localhost:6379',
    jwtSecret: process.env.JWT_SECRET || 'dev-secret',
    logLevel: 'debug',
  },
  production: {
    port: process.env.PORT || 8080,
    db: process.env.DATABASE_URL,  // MUST be set in production
    redis: process.env.REDIS_URL,
    jwtSecret: process.env.JWT_SECRET,
    logLevel: 'error',
  },
  test: {
    port: 3001,
    db: 'postgres://localhost:5432/test',
    redis: 'redis://localhost:6379/1',
    jwtSecret: 'test-secret',
    logLevel: 'silent',
  },
};

module.exports = config[process.env.NODE_ENV || 'development'];

// Usage
const config = require('./config');
app.listen(config.port);
```

---

## 37.19 package.json — Complete Guide

```json
{
  "name": "obenan-api",
  "version": "2.5.0",
  "description": "AI-powered business management SaaS platform",
  "main": "src/index.js",
  "scripts": {
    "start": "node dist/main.js",
    "dev": "nodemon src/index.js",
    "build": "tsc",
    "test": "mocha --recursive tests/",
    "test:coverage": "nyc mocha --recursive tests/",
    "lint": "eslint src/",
    "lint:fix": "eslint src/ --fix",
    "migrate": "sequelize db:migrate",
    "migrate:undo": "sequelize db:migrate:undo",
    "seed": "sequelize db:seed:all"
  },
  "engines": {
    "node": ">=20.0.0"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "mocha": "^10.0.0",
    "nodemon": "^3.0.0"
  }
}
```

### Dependency Versioning (Semver)

```
"express": "4.18.2"     → EXACT version only
"express": "~4.18.2"    → Patch updates (4.18.x) — tilde
"express": "^4.18.2"    → Minor + patch updates (4.x.x) — caret (DEFAULT)
"express": "*"           → Any version (DANGEROUS)
"express": ">=4.0.0"    → Version 4 or higher

Semver: MAJOR.MINOR.PATCH
- MAJOR: Breaking changes (4.x → 5.x)
- MINOR: New features, backwards compatible (4.18 → 4.19)
- PATCH: Bug fixes (4.18.1 → 4.18.2)
```

### npm vs yarn vs pnpm

| Feature | npm | yarn | pnpm |
|---------|-----|------|------|
| Lock file | `package-lock.json` | `yarn.lock` | `pnpm-lock.yaml` |
| Speed | Moderate | Fast | Fastest |
| Disk space | Each project copies deps | Same | Shared store (hardlinks) |
| Workspaces | Yes | Yes | Yes |
| Security | `npm audit` | `yarn audit` | `pnpm audit` |

---

## 37.20 Timers — Complete Reference

```javascript
// ─── setTimeout — Execute ONCE after delay ───
const timer = setTimeout(() => {
  console.log('Runs after 2 seconds');
}, 2000);
clearTimeout(timer); // Cancel

// ─── setInterval — Execute REPEATEDLY ───
const interval = setInterval(() => {
  console.log('Runs every 3 seconds');
}, 3000);
clearInterval(interval); // Cancel

// ─── setImmediate — Execute after I/O events ───
setImmediate(() => {
  console.log('Runs in the Check phase of event loop');
});

// ─── process.nextTick — Execute before I/O ───
process.nextTick(() => {
  console.log('Runs before any I/O or timer callbacks');
});

// ─── queueMicrotask — Standard microtask ───
queueMicrotask(() => {
  console.log('Runs as a microtask (like Promise.then)');
});

// ─── AbortController (cancel async operations) ───
const controller = new AbortController();
const { signal } = controller;

setTimeout(() => controller.abort(), 5000); // Cancel after 5s

try {
  const response = await fetch(url, { signal });
  const data = await response.json();
} catch (err) {
  if (err.name === 'AbortError') {
    console.log('Request was cancelled');
  }
}
```

---

## 37.21 Buffer & Binary Data

```javascript
const buf = Buffer.from('Hello World', 'utf-8');

// ─── CREATE ───
Buffer.alloc(10);                    // 10 bytes, filled with zeros
Buffer.alloc(10, 1);                // 10 bytes, filled with 1
Buffer.from([72, 101, 108, 108, 111]); // From byte array
Buffer.from('Hello', 'utf-8');       // From string
Buffer.from('48656c6c6f', 'hex');    // From hex

// ─── READ ───
buf.toString('utf-8');               // 'Hello World'
buf.toString('hex');                 // '48656c6c6f20576f726c64'
buf.toString('base64');              // 'SGVsbG8gV29ybGQ='
buf.length;                          // 11 bytes
buf[0];                              // 72 (ASCII for 'H')

// ─── COMPARE ───
buf.equals(Buffer.from('Hello World')); // true
Buffer.compare(buf1, buf2);              // -1, 0, or 1

// ─── CONCAT ───
const combined = Buffer.concat([buf1, buf2, buf3]);

// ─── SLICE ───
const slice = buf.slice(0, 5);       // First 5 bytes
// Note: slice shares memory with original (not a copy)

// ─── COMMON USE CASES ───
// File I/O returns Buffers
const fileBuffer = fs.readFileSync('image.png');
console.log(fileBuffer.length); // File size in bytes

// Converting between encodings
const base64 = Buffer.from('Hello').toString('base64');      // 'SGVsbG8='
const original = Buffer.from('SGVsbG8=', 'base64').toString(); // 'Hello'

// HTTP request bodies
app.post('/upload', (req, res) => {
  const chunks = [];
  req.on('data', (chunk) => chunks.push(chunk));
  req.on('end', () => {
    const body = Buffer.concat(chunks);
    // body is a Buffer containing the raw request body
  });
});
```

---

## 37.22 EventEmitter — Complete Guide

```javascript
const EventEmitter = require('events');

class OrderService extends EventEmitter {
  async placeOrder(orderData) {
    const order = await db.orders.create(orderData);
    this.emit('order:placed', order);
    return order;
  }

  async cancelOrder(orderId) {
    const order = await db.orders.update(orderId, { status: 'cancelled' });
    this.emit('order:cancelled', order);
    return order;
  }
}

const orderService = new OrderService();

// ─── on() — Listen for event (persistent) ───
orderService.on('order:placed', (order) => {
  console.log('New order:', order.id);
});

// ─── once() — Listen only ONCE ───
orderService.once('order:placed', (order) => {
  console.log('First order ever:', order.id);
});

// ─── Remove listener ───
const handler = (order) => console.log(order);
orderService.on('order:placed', handler);
orderService.removeListener('order:placed', handler); // OR .off()
orderService.removeAllListeners('order:placed');       // Remove all for event
orderService.removeAllListeners();                     // Remove ALL listeners

// ─── Get listener count ───
orderService.listenerCount('order:placed');  // number

// ─── Set max listeners (default: 10) ───
orderService.setMaxListeners(20);

// ─── Error event (special — crashes if no listener!) ───
orderService.on('error', (err) => {
  console.error('Order service error:', err);
});
// If you emit 'error' without a listener, Node crashes!
```

---

## 37.23 Util Module — Useful Utilities

```javascript
const util = require('util');

// ─── promisify — Convert callback functions to promises ───
const readFile = util.promisify(fs.readFile);
const data = await readFile('file.txt', 'utf-8');

const pbkdf2 = util.promisify(crypto.pbkdf2);
const hash = await pbkdf2(password, salt, 100000, 64, 'sha512');

// ─── callbackify — Convert promise to callback (rare) ───
const callbackFn = util.callbackify(asyncFunction);

// ─── types — Type checking ───
util.types.isPromise(new Promise(() => {}));  // true
util.types.isDate(new Date());                // true
util.types.isRegExp(/abc/);                   // true
util.types.isAsyncFunction(async () => {});   // true

// ─── inspect — Pretty print objects ───
console.log(util.inspect(complexObject, {
  depth: 5,      // How deep to recurse
  colors: true,  // Colorized output
  compact: false // Expanded format
}));

// ─── format — printf-style formatting ───
util.format('Hello %s, you are %d years old', 'Saad', 28);
// 'Hello Saad, you are 28 years old'

// ─── deprecate — Mark function as deprecated ───
const oldFunction = util.deprecate(() => {
  // old implementation
}, 'oldFunction() is deprecated. Use newFunction() instead.');
```

---

## 37.24 Graceful Shutdown

When your server receives a shutdown signal, you should clean up resources before exiting.

```javascript
const server = app.listen(3000);

async function gracefulShutdown(signal) {
  console.log(`${signal} received. Starting graceful shutdown...`);

  // 1. Stop accepting new connections
  server.close(() => {
    console.log('HTTP server closed');
  });

  // 2. Wait for existing requests to finish (with timeout)
  const forceShutdownTimeout = setTimeout(() => {
    console.error('Forcing shutdown after timeout');
    process.exit(1);
  }, 30000); // 30 seconds max

  try {
    // 3. Close database connections
    await sequelize.close();
    console.log('Database connections closed');

    // 4. Close Redis connections
    await redis.quit();
    console.log('Redis connection closed');

    // 5. Wait for BullMQ workers to finish current jobs
    await worker.close();
    console.log('Job workers stopped');

    clearTimeout(forceShutdownTimeout);
    console.log('Graceful shutdown complete');
    process.exit(0);
  } catch (err) {
    console.error('Error during shutdown:', err);
    process.exit(1);
  }
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));
```

---

## 37.25 REPL & Console Methods

```javascript
// ─── Console methods ───
console.log('Normal output');
console.error('Error output — goes to stderr');
console.warn('Warning');

console.table([
  { name: 'Saad', age: 28 },
  { name: 'Ali', age: 25 },
]);
// Prints a formatted table!

console.time('operation');
await heavyWork();
console.timeEnd('operation');  // operation: 145.23ms

console.count('api-call');     // api-call: 1
console.count('api-call');     // api-call: 2
console.countReset('api-call');

console.dir(object, { depth: 3, colors: true }); // Inspect object

console.group('Server');
console.log('Port: 3000');
console.log('Env: production');
console.groupEnd();

console.assert(1 === 2, 'This will print because assertion failed');

console.trace('Where am I?'); // Prints stack trace
```

---

## 37.26 Zlib — Compression

```javascript
const zlib = require('zlib');
const { promisify } = require('util');
const gzip = promisify(zlib.gzip);
const gunzip = promisify(zlib.gunzip);

// ─── Compress/Decompress strings ───
const compressed = await gzip('Hello World');
console.log(compressed.length); // Much smaller for large data

const original = (await gunzip(compressed)).toString();
console.log(original); // 'Hello World'

// ─── Express compression middleware ───
const compression = require('compression');
app.use(compression({
  level: 6,        // 1-9 (1=fastest, 9=best compression)
  threshold: 1024, // Don't compress responses smaller than 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  },
}));
```

---

## 37.27 Complete Node.js Interview Questions & Answers

### Fundamentals

**Q: What is Node.js and why was it created?**
> Node.js is a JavaScript runtime built on Chrome's V8 engine, created by Ryan Dahl in 2009. He was frustrated with how Apache HTTP Server handled concurrent connections — it created a new thread per connection, which was inefficient. Node.js solved this with a single-threaded, event-driven, non-blocking I/O model that can handle thousands of concurrent connections efficiently.

**Q: What is the difference between Node.js and JavaScript?**
> JavaScript is a programming language. Node.js is a runtime environment that lets you execute JavaScript outside the browser. Node.js adds APIs for file system, networking, OS access, etc. that don't exist in browser JavaScript.

**Q: What is V8?**
> V8 is Google's open-source JavaScript engine written in C++. It compiles JavaScript directly to machine code using JIT (Just-In-Time) compilation, making it extremely fast. It powers both Chrome and Node.js.

**Q: What is libuv?**
> libuv is a C library that provides Node.js with its event loop, async I/O, thread pool, and cross-platform abstractions. It handles file system operations, DNS resolution, network I/O, child processes, and timers. The thread pool (default 4 threads, max 1024) handles operations that can't be done asynchronously at the OS level.

**Q: Is Node.js truly single-threaded?**
> Node.js runs your JavaScript code on a single thread, but it's NOT truly single-threaded. Under the hood, libuv maintains a thread pool (default 4 threads) for blocking operations like file I/O, DNS lookups, and crypto operations. Also, you can create additional threads using Worker Threads or child processes.

**Q: When should you NOT use Node.js?**
> - CPU-intensive tasks (image/video processing, complex calculations, ML training)
> - Applications that require heavy multi-threading
> - Simple CRUD apps that don't need real-time features (any language works fine)
>
> However, CPU-intensive tasks can be handled with Worker Threads, child processes, or offloading to services like AWS Lambda.

### Async & Event Loop

**Q: What is the difference between sync and async code?**
> Synchronous code blocks the execution — the program waits for each line to complete before moving to the next. Asynchronous code doesn't wait — it registers a callback and continues executing. When the async operation completes, the callback is placed in the event loop queue and executed when the call stack is empty.

**Q: What is callback hell and how do you avoid it?**
```javascript
// Callback hell (pyramid of doom)
getUser(id, (err, user) => {
  getOrders(user.id, (err, orders) => {
    getOrderDetails(orders[0].id, (err, details) => {
      getProduct(details.productId, (err, product) => {
        // 4 levels deep!
      });
    });
  });
});

// Solution 1: Promises
getUser(id)
  .then(user => getOrders(user.id))
  .then(orders => getOrderDetails(orders[0].id))
  .then(details => getProduct(details.productId))
  .catch(err => console.error(err));

// Solution 2: async/await (BEST)
try {
  const user = await getUser(id);
  const orders = await getOrders(user.id);
  const details = await getOrderDetails(orders[0].id);
  const product = await getProduct(details.productId);
} catch (err) {
  console.error(err);
}
```

**Q: What is the difference between Promise.all(), Promise.allSettled(), Promise.race(), and Promise.any()?**
```javascript
const p1 = Promise.resolve('A');
const p2 = Promise.reject('B');
const p3 = Promise.resolve('C');

// Promise.all — FAILS if ANY promise rejects
await Promise.all([p1, p2, p3]);
// ❌ Rejects with 'B'

// Promise.allSettled — Waits for ALL to complete (never rejects)
await Promise.allSettled([p1, p2, p3]);
// ✓ [{ status: 'fulfilled', value: 'A' },
//    { status: 'rejected', reason: 'B' },
//    { status: 'fulfilled', value: 'C' }]

// Promise.race — First to SETTLE (resolve or reject) wins
await Promise.race([p1, p2, p3]);
// ✓ 'A' (p1 resolved first)

// Promise.any — First to RESOLVE wins (ignores rejections)
await Promise.any([p2, p3, p1]);
// ✓ 'C' (first resolved — p2 rejected, so it's ignored)
// Only rejects if ALL promises reject (AggregateError)
```

**Q: What happens if you don't handle a Promise rejection?**
> In Node.js 15+, unhandled promise rejections terminate the process. In earlier versions, it logs a warning. Always use try/catch with async/await or .catch() with Promises. Add `process.on('unhandledRejection')` as a safety net.

### Modules & Packages

**Q: What is the difference between dependencies and devDependencies?**
> - `dependencies` — needed at runtime in production (express, sequelize, jsonwebtoken)
> - `devDependencies` — only needed during development (mocha, eslint, nodemon, typescript)
> - `npm install --production` only installs dependencies, not devDependencies

**Q: What is npx?**
> npx runs a package command without installing it globally. It checks `node_modules/.bin` first, then downloads temporarily if not found.
> - `npx sequelize migration:generate` — runs sequelize CLI without global install
> - `npx create-react-app my-app` — downloads, runs, and cleans up

**Q: What is the difference between module.exports and exports?**
```javascript
// exports is a SHORTHAND for module.exports
// They initially point to the same object

exports.add = (a, b) => a + b;     // ✓ Works — adding property to the object
module.exports.add = (a, b) => a + b; // ✓ Same thing

exports = { add: (a, b) => a + b };   // ✗ BROKEN — reassigns exports reference
module.exports = { add: (a, b) => a + b }; // ✓ Works — replaces the whole export

// Rule: If you're exporting a single thing (class, function),
// use module.exports. For multiple things, use exports.property.
```

### Performance & Scaling

**Q: How do you scale a Node.js application?**
> 1. **Vertical scaling** — More CPU/RAM (limited)
> 2. **Cluster module** — One process per CPU core
> 3. **PM2** — Process manager with cluster mode, auto-restart
> 4. **Load balancer** — Nginx/ALB distributes traffic across instances
> 5. **Horizontal scaling** — Multiple servers behind a load balancer
> 6. **Microservices** — Split into independent services
> 7. **Caching** — Redis to reduce database load
> 8. **CDN** — Serve static assets from edge locations
> 9. **Worker threads** — Offload CPU-intensive tasks
> 10. **Message queues** — BullMQ/RabbitMQ for async processing

**Q: How do you handle CPU-intensive tasks in Node.js?**
> 1. **Worker Threads** — offload to separate thread
> 2. **Child processes** — fork a separate process
> 3. **Message queue** — BullMQ processes job in background worker
> 4. **External service** — AWS Lambda, dedicated microservice
> 5. **Cluster mode** — distribute across CPU cores
>
> At Obenan, I used BullMQ with Redis for CPU-intensive sentiment analysis — the API queues the job and returns immediately, while a background worker processes it.

**Q: What is the thread pool in Node.js?**
> libuv maintains a thread pool (default 4 threads) for operations that can't be done asynchronously at the OS level:
> - File system operations (fs.readFile, fs.writeFile)
> - DNS lookups (dns.lookup)
> - Crypto operations (crypto.pbkdf2, crypto.randomBytes)
> - Zlib compression
>
> You can increase the pool size: `process.env.UV_THREADPOOL_SIZE = 8` (max 1024)

### Security

**Q: How do you prevent common security vulnerabilities in Node.js?**
> Already covered in section 37.16 — SQL injection, NoSQL injection, XSS, CSRF, rate limiting, helmet, CORS, input validation, env variables.

**Q: How do you store passwords securely?**
> Never store plain text. Use bcrypt with a salt factor of 10-12:
> ```javascript
> const hash = await bcrypt.hash(password, 12);
> const isValid = await bcrypt.compare(input, hash);
> ```
> bcrypt is intentionally slow — it's designed to resist brute force attacks.

**Q: What is the helmet package?**
> helmet sets security-related HTTP headers:
> - `X-Content-Type-Options: nosniff` — prevents MIME sniffing
> - `X-Frame-Options: DENY` — prevents clickjacking
> - `Strict-Transport-Security` — forces HTTPS
> - `X-XSS-Protection` — enables browser XSS filter
> - `Content-Security-Policy` — controls resource loading

### Real-World Architecture

**Q: How do you structure a large Node.js project?**
```
src/
├── config/          # Configuration files
│   ├── database.js
│   ├── redis.js
│   └── index.js
├── controllers/     # Route handlers (thin — call services)
│   └── users.controller.js
├── services/        # Business logic
│   └── users.service.js
├── repositories/    # Data access layer
│   └── users.repository.js
├── models/          # Database models
│   └── user.model.js
├── middleware/       # Express middleware
│   ├── auth.js
│   ├── validate.js
│   └── errorHandler.js
├── routes/          # Route definitions
│   └── users.routes.js
├── utils/           # Helper functions
│   ├── logger.js
│   └── errors.js
├── jobs/            # BullMQ job processors
│   └── reviewProcessor.js
├── migrations/      # Database migrations
├── seeders/         # Seed data
├── tests/           # Tests
│   ├── unit/
│   └── integration/
├── app.js           # Express app setup
└── index.js         # Entry point (starts server)
```

**Q: Explain the flow of a request through your API.**
> ```
> Client → Nginx (reverse proxy) → Load Balancer (ELB)
>   → Node.js Server
>     → CORS middleware
>     → Body parser (express.json)
>     → Rate limiter
>     → Auth middleware (JWT verify)
>     → Route handler
>       → Controller (extracts params, calls service)
>         → Service (business logic, validation)
>           → Repository (database query via Sequelize)
>             → PostgreSQL / MongoDB
>           ← Data returned
>         ← Formatted response
>       ← JSON response
>     → Error middleware (if error thrown)
>   ← HTTP Response
> ← Client receives response
> ```

**Q: How do you handle logging in production?**
```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }));
}

// Usage
logger.info('Server started', { port: 3000 });
logger.error('Database connection failed', { error: err.message });
logger.warn('Rate limit exceeded', { ip: req.ip });
```

**Q: How do you handle database connection failures?**
```javascript
async function connectWithRetry(maxRetries = 5) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      await sequelize.authenticate();
      console.log('Database connected');
      return;
    } catch (err) {
      console.error(`DB connection attempt ${attempt} failed:`, err.message);
      if (attempt === maxRetries) {
        console.error('Max retries reached. Exiting.');
        process.exit(1);
      }
      await new Promise(r => setTimeout(r, 5000 * attempt)); // Exponential backoff
    }
  }
}
```

### Quick-Fire Questions (One-Liner Answers)

| Question | Answer |
|----------|--------|
| What is Node.js? | JavaScript runtime built on V8 engine |
| Who created Node.js? | Ryan Dahl in 2009 |
| Is Node.js a framework? | No, it's a runtime environment |
| Is Node.js single-threaded? | Yes for JS code, no under the hood (libuv thread pool) |
| What is npm? | Node Package Manager — installs and manages dependencies |
| What is package-lock.json? | Locks exact dependency versions for reproducible installs |
| What is the event loop? | Mechanism that handles async operations via task queues |
| What is middleware? | Function that processes requests before the route handler |
| What is a callback? | Function passed as argument, called when async op completes |
| What is a stream? | Data processed piece by piece without loading all into memory |
| What is a Buffer? | Raw binary data (like a byte array) |
| What is REPL? | Read-Eval-Print-Loop — interactive Node.js shell (`node` command) |
| What is process.env? | Object containing environment variables |
| What is __dirname? | Absolute path to current file's directory |
| What are CommonJS modules? | Node.js module system using `require()` and `module.exports` |
| What is the purpose of .env files? | Store environment variables locally (never commit to git) |
| What does npm init do? | Creates a new package.json file |
| What does npm ci do? | Clean install — deletes node_modules and installs from lock file |
| What is nodemon? | Auto-restarts Node.js on file changes (dev tool) |
| What is PM2? | Production process manager with clustering and monitoring |
| What is cors? | Cross-Origin Resource Sharing — controls which domains can access your API |
| What is helmet? | Middleware that sets security HTTP headers |
| What is morgan? | HTTP request logger middleware for Express |
| What is dotenv? | Loads .env file variables into process.env |
| What is ESLint? | JavaScript/TypeScript linter — enforces code style rules |
| What is Prettier? | Code formatter — auto-formats code style |
| What is n or nvm? | Node Version Manager — switch between Node.js versions |

---

*Last Updated: April 2026*
*Prepared for: Saad Sohail — Senior Full-Stack Engineer*
*Total Topics: 37 | Total Pages: ~80+ when printed*

