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

