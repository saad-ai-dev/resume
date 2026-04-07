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

