# 🔷 TypeScript — Complete Interview Guide

> **Part 4 of Ahmed's shadow interview prep.**
> Referenced directly from the [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) (v5.x / v6.0).
> Every section shows the **JS bug it would have caused → the TS fix** so you understand *why* each feature exists.

---

## 📑 Table of Contents

1.  [What is TypeScript & Why Does It Exist?](#1-what-is-typescript--why-does-it-exist)
2.  [Silent Bugs TypeScript Catches](#2-silent-bugs-typescript-catches)
3.  [Type Inference — TS Reads Your Mind](#3-type-inference--ts-reads-your-mind)
4.  [Primitive Types & Basic Annotations](#4-primitive-types--basic-annotations)
5.  [Special Types: any, unknown, never, void](#5-special-types-any-unknown-never-void)
6.  [Interfaces](#6-interfaces)
7.  [Type Aliases](#7-type-aliases)
8.  [Interface vs Type Alias — The Real Difference](#8-interface-vs-type-alias--the-real-difference)
9.  [Union & Intersection Types](#9-union--intersection-types)
10. [Narrowing & Type Guards](#10-narrowing--type-guards)
11. [Literal Types & as const](#11-literal-types--as-const)
12. [Functions in TypeScript](#12-functions-in-typescript)
13. [Generics — Write Once, Type Everything](#13-generics--write-once-type-everything)
14. [Classes in TypeScript](#14-classes-in-typescript)
15. [Utility Types — The Built-in Toolkit](#15-utility-types--the-built-in-toolkit)
16. [Type Manipulation — Advanced](#16-type-manipulation--advanced)
17. [Enums](#17-enums)
18. [Modules & Declaration Files](#18-modules--declaration-files)
19. [tsconfig.json — Compiler Options](#19-tsconfigjson--compiler-options)
20. [TypeScript in React (TSX)](#20-typescript-in-react-tsx)
21. [Common Pitfalls & Anti-Patterns](#21-common-pitfalls--anti-patterns)
22. [Interview Questions & Answers](#22-interview-questions--answers)

---

## 1. What is TypeScript & Why Does It Exist?

TypeScript is a **statically typed superset of JavaScript** developed by Microsoft. It compiles down to plain JavaScript, so it runs everywhere JS runs — browsers, Node.js, Deno, Bun.

```
TypeScript (.ts) ──► tsc (compiler) ──► JavaScript (.js)
```

The core idea is simple: **catch bugs at compile time, not at runtime**. JavaScript is dynamic and flexible — great for quick scripts, but as apps grow, that flexibility becomes a source of bugs, especially in teams.

### What TypeScript Adds to JavaScript

| Feature | JavaScript | TypeScript |
|---|---|---|
| Type annotations | No | Yes — `name: string` |
| Compile-time error checking | No | Yes |
| Interfaces & type aliases | No | Yes |
| Generics | No | Yes |
| Enums | No (only objects) | Yes |
| Access modifiers | No | Yes — `private`, `protected`, `readonly` |
| Autocomplete / IntelliSense | Guesswork | Precise, context-aware |
| Refactoring safety | Breaks silently | Compiler finds every broken callsite |
| `null`/`undefined` safety | No | Yes — `strictNullChecks` |

> **Key insight:** TypeScript is erased at runtime. Types are **only** for the compiler and your editor. The output JS has zero TypeScript in it.

### 📖 Resources
- [typescriptlang.org — The TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [typescriptlang.org — TS for JS Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [TypeScript Playground](https://www.typescriptlang.org/play/)

---

## 2. Silent Bugs TypeScript Catches

This is the best way to understand TypeScript's value: see the exact bugs it prevents in real code.

### Bug 1 — Typo on a Property Name
```javascript
// JS — silently returns undefined, then crashes
const user = { firstName: "Ahmed", age: 25 };
console.log(user.fistName.toUpperCase()); // TypeError at runtime!
//                ^^^^^^^^ typo: fistName vs firstName
```

```typescript
// TS — caught immediately at the editor level, before you even run
const user = { firstName: "Ahmed", age: 25 };
console.log(user.fistName.toUpperCase());
//               ^^^^^^^^
// Error: Property 'fistName' does not exist. Did you mean 'firstName'?
```

---

### Bug 2 — Calling a Method on null
```javascript
// JS — crashes when user is null (e.g., not logged in, DB miss)
function greetUser(user) {
  return user.name.toUpperCase(); // TypeError: Cannot read properties of null
}
greetUser(null); // boom in production
```

```typescript
// TS — forces you to handle null before using the value
function greetUser(user: { name: string } | null): string {
  return user.name.toUpperCase();
  //     ^^^^ Error: 'user' is possibly 'null'
}

// Fixed version — TypeScript accepts this:
function greetUser(user: { name: string } | null): string {
  if (!user) return "Guest";
  return user.name.toUpperCase(); // TS knows: user is not null here
}
```

---

### Bug 3 — Wrong Argument Type
```javascript
// JS — passes a string where a number is expected, NaN in disguise
function addTax(price, taxRate) {
  return price * (1 + taxRate);
}
addTax("100", 0.2); // NaN — "100" * 1.2 = NaN. No warning. Bugs in invoices.
```

```typescript
// TS — error right at the call site
function addTax(price: number, taxRate: number): number {
  return price * (1 + taxRate);
}
addTax("100", 0.2);
//     ^^^^^ Error: Argument of type 'string' is not assignable to 'number'
```

---

### Bug 4 — Accessing a Field That Doesn't Exist on All Union Members
```javascript
// JS — works for Circle, silently returns NaN for Rectangle
function getArea(shape) {
  return Math.PI * shape.radius * shape.radius; // radius is undefined for rect
}
getArea({ width: 10, height: 5 }); // NaN — no warning whatsoever
```

```typescript
// TS — you must handle all cases correctly
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number };

function getArea(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2; // TS knows: radius exists here
  }
  return shape.width * shape.height;   // TS knows: width/height exist here
}
```

---

### Bug 5 — Refactoring Breaks Callsites Silently
```javascript
// JS — rename a field, nothing tells you what broke
function createUser(name, email) { return { name, email }; }
// Refactor to fullName:
function createUser(fullName, email) { return { fullName, email }; }
// Every reader: user.name → now silently undefined. Good luck finding it.
```

```typescript
// TS — every broken callsite is flagged instantly
interface User { fullName: string; email: string; }

function createUser(fullName: string, email: string): User {
  return { fullName, email };
}

const u = createUser("Ahmed", "ahmed@dev.com");
console.log(u.name); // Error: Property 'name' does not exist on type 'User'
// TS tells you every single place that was using .name
```

---

### Bug 6 — Async Function With Missing Return Path
```javascript
// JS — if id is negative, returns undefined silently
async function getUser(id) {
  if (id > 0) {
    return await db.find(id);
  }
  // forgot the else! caller gets undefined
}
const user = await getUser(-1);
user.email; // TypeError: Cannot read properties of undefined
```

```typescript
// TS — forces all code paths to return the declared type
async function getUser(id: number): Promise<User> {
  if (id > 0) {
    return await db.find(id);
  }
  // Error: A function returning 'Promise<User>' must have a return statement
  throw new Error(`Invalid id: ${id}`); // forced to handle it
}
```

---

## 3. Type Inference — TS Reads Your Mind

You don't need to annotate everything. TypeScript **infers** types from context automatically.

```typescript
// TypeScript infers all of these — no annotation needed
let name    = "Ahmed";          // inferred: string
let age     = 25;               // inferred: number
let isAdmin = true;             // inferred: boolean
let scores  = [98, 87, 91];    // inferred: number[]
let tags    = ["ts", "react"]; // inferred: string[]

// Object — all fields inferred
const user = {
  name: "Ahmed",     // string
  age: 25,           // number
  active: true,      // boolean
};
// user: { name: string; age: number; active: boolean }

// Function return type inferred from the return statement
function double(n: number) {
  return n * 2; // return type inferred: number
}

// Chained array methods — fully typed
const doubled = [1, 2, 3].map(n => n * 2); // number[]
const names   = users.map(u => u.name);     // string[]
```

### When to Annotate vs When to Let TS Infer

```typescript
// Let TS infer — it's smarter than redundant annotations
const doubled = [1, 2, 3].map(n => n * 2); // no need for: number[]

// Annotate function PARAMETERS — TS can't infer these
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Annotate when the inferred type is too wide
let status = "active"; // inferred: string (too wide — reassignable to "banana")
let status: "active" | "inactive" = "active"; // precise literal union

// Annotate variables that start as null/empty
let currentUser: User | null = null; // can't infer from null alone
let items: string[] = [];            // can't infer element type from []
```

### 📖 Resources
- [typescriptlang.org — Type Inference](https://www.typescriptlang.org/docs/handbook/type-inference.html)

---

## 4. Primitive Types & Basic Annotations

```typescript
// Core primitives — always lowercase
let name:    string  = "Ahmed";
let age:     number  = 25;
let isAdmin: boolean = false;

// Arrays
let scores:  number[]  = [98, 87, 91];
let tags:    string[]  = ["ts", "node"];
let matrix:  number[][] = [[1, 2], [3, 4]]; // 2D array
let scores2: Array<number> = [98, 87]; // generic syntax — identical

// Tuples — fixed-length, fixed-type, ordered
let point:     [number, number]  = [10, 20];
let entry:     [string, number]  = ["Ahmed", 25];
let httpPair:  [number, string]  = [200, "OK"];

point[0]; // number
point[2]; // Error: Tuple type has no element at index '2'

// Named tuple elements (TS 4.0+)
type RGB = [red: number, green: number, blue: number];
const crimson: RGB = [220, 20, 60];

// Optional object fields
function printUser(user: {
  name: string;
  age: number;
  email?: string; // optional — can be omitted or undefined
}) {
  console.log(user.email?.toUpperCase()); // optional chaining is safe
}

// null and undefined
let maybeNull:      string | null      = null;
let maybeUndefined: number | undefined = undefined;

// BigInt and Symbol
let big: bigint = 100n;
let sym: symbol = Symbol("id");
```

### 📖 Resources
- [typescriptlang.org — Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

---

## 5. Special Types: any, unknown, never, void

These four come up constantly in interviews. Know them cold.

### `any` — The Escape Hatch (avoid it)
```typescript
// any = disable TypeScript entirely for this value
let value: any = "hello";
value = 42;            // no error
value = { x: 1 };     // no error
value.foo.bar.baz;     // no error — but crashes at runtime!
value();               // no error — but crashes!

// The danger: any is contagious
function parseData(raw: any) {
  return raw.user.name; // TypeScript won't warn you. This is as unsafe as plain JS.
}
```

### `unknown` — The Safe `any`
```typescript
// unknown = could be anything, but you MUST check before using
let value: unknown = fetchSomething();

value.toUpperCase(); // Error: Object is of type 'unknown'
value();             // Error: Cannot invoke an object of type 'unknown'

// Must narrow it before use
if (typeof value === "string") {
  value.toUpperCase(); // safe — TS knows it's a string here
}

// Catch blocks — TS 4.0+ defaults error to unknown
try {
  riskyOp();
} catch (err: unknown) {
  if (err instanceof Error) {
    console.error(err.message); // safe
  }
}

// Use unknown for: API responses, JSON.parse results, catch blocks, user input
function processInput(input: unknown): string {
  if (typeof input === "string") return input.toUpperCase();
  if (typeof input === "number") return input.toFixed(2);
  throw new Error("Unsupported type");
}
```

### `never` — Values That Can Never Exist
```typescript
// never appears in two contexts:
// 1. Functions that never return
function throwError(msg: string): never {
  throw new Error(msg); // always throws — never returns
}

function infiniteLoop(): never {
  while (true) {} // never exits
}

// 2. Exhaustive union checking — the most powerful use
type Shape = "circle" | "square" | "triangle";

function assertNever(x: never): never {
  throw new Error(`Unexpected: ${x}`);
}

function describe(shape: Shape): string {
  switch (shape) {
    case "circle":   return "round";
    case "square":   return "4 sides";
    case "triangle": return "3 sides";
    default:
      return assertNever(shape);
      // If you add "hexagon" to Shape and forget a case here,
      // TS error: 'hexagon' is not assignable to 'never'
      // The compiler forces you to handle every case.
  }
}
```

### `void` — Functions Returning Nothing Meaningful
```typescript
// void = this function completes but doesn't return a useful value
function logMessage(msg: string): void {
  console.log(msg);
  // no return, or just: return;
}

// void vs never:
// void — function runs to completion, just returns nothing
// never — function never reaches its end (throws / infinite loops)
```

### 📖 Resources
- [typescriptlang.org — The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)

---

## 6. Interfaces

Interfaces define the **shape** of an object. They're the primary tool for describing structured data.

```typescript
// Basic interface
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;             // optional — can be omitted
  readonly createdAt: Date; // cannot be changed after creation
}

const user: User = {
  id: 1,
  name: "Ahmed",
  email: "ahmed@kickoff.dev",
  createdAt: new Date(),
};

user.email = "new@email.com"; // fine
user.createdAt = new Date();  // Error: Cannot assign to 'createdAt' — it's readonly

// Extending — an interface can inherit from another
interface Admin extends User {
  role: "super" | "moderator";
  permissions: string[];
}

// Multiple inheritance
interface Timestamped { createdAt: Date; updatedAt: Date; }
interface SoftDeletable { deletedAt: Date | null; }

interface Post extends Timestamped, SoftDeletable {
  id: number;
  title: string;
  body: string;
}

// Index signature — for objects with dynamic keys
interface Headers {
  [key: string]: string;
}
const headers: Headers = {
  "Content-Type": "application/json",
  "Authorization": "Bearer xyz",
};

// Function shape in interface
interface Comparator<T> {
  (a: T, b: T): number;
}
const compareNumbers: Comparator<number> = (a, b) => a - b;

// Contract for a class
interface Serializable {
  serialize(): string;
  deserialize(data: string): void;
}

class UserModel implements Serializable {
  serialize() { return JSON.stringify(this); }
  deserialize(data: string) { Object.assign(this, JSON.parse(data)); }
}
```

---

## 7. Type Aliases

Type aliases give a name to **any** type — not just objects. They're more flexible than interfaces.

```typescript
// Object type (looks like interface)
type User = {
  id: number;
  name: string;
  email: string;
};

// Primitive alias
type UserId   = number;
type Callback = () => void;

// Union types — this is where type aliases shine
type Status = "active" | "inactive" | "banned";
type ID     = string | number;
type MaybeUser = User | null | undefined;

// Intersection types
type AdminUser = User & { role: "admin"; permissions: string[] };

// Tuple type
type Coordinate = [latitude: number, longitude: number];

// Function type
type AsyncHandler  = (req: Request, res: Response) => Promise<void>;
type Predicate<T>  = (value: T) => boolean;

// Generic type alias — extremely common
type ApiResponse<T> =
  | { status: "success"; data: T;    error: null }
  | { status: "error";   data: null; error: { code: string; message: string } };

// Usage:
async function fetchUser(id: number): Promise<ApiResponse<User>> {
  try {
    const user = await db.getUser(id);
    return { status: "success", data: user, error: null };
  } catch (e) {
    return { status: "error", data: null, error: { code: "NOT_FOUND", message: String(e) } };
  }
}

// Conditional types (can only be expressed as type, not interface)
type NonNullable<T> = T extends null | undefined ? never : T;
type Flatten<T>     = T extends Array<infer Item> ? Item : T;
```

---

## 8. Interface vs Type Alias — The Real Difference

This is one of the most common interview questions.

| Capability | `interface` | `type` |
|---|---|---|
| Object shapes | Yes | Yes |
| Primitive aliases | No | Yes — `type Age = number` |
| Union types | No | Yes — `type A = B \| C` |
| Intersection types | via `extends` | Yes — `type A = B & C` |
| Tuples | No | Yes |
| Computed/conditional types | No | Yes |
| Declaration merging | Yes — auto-merges | No — Error |
| `implements` in class | Yes | Yes (for object types) |

### The Critical Difference — Declaration Merging

```typescript
// Interface — multiple declarations MERGE automatically
// Perfect for augmenting third-party library types
interface Window {
  myPlugin: { version: string };
}
interface Window {
  analytics: { track(event: string): void };
}
// Result: Window now has BOTH properties — this is how @types packages extend globals

// Type alias — CANNOT be redeclared
type Config = { debug: boolean };
type Config = { verbose: boolean }; // Error: Duplicate identifier 'Config'
```

### Rule of Thumb

```typescript
// Use INTERFACE when:
// - Describing object shapes that others will implement or extend
// - Writing library types that consumers may need to augment
// - Class contracts (implements)
interface UserRepository {
  findById(id: string): Promise<User>;
  save(user: User): Promise<void>;
}

// Use TYPE when:
// - Union / intersection types
// - Primitives, tuples, function signatures
// - Computed / mapped / conditional types
// - Anything an interface literally cannot express
type EventName = "click" | "focus" | "blur";
type Handler<E> = (event: E) => void;
type MaybeArray<T> = T | T[];
```

---

## 9. Union & Intersection Types

### Union Types (`|`) — "Either This OR That"

```typescript
type StringOrNumber = string | number;
type Status = "active" | "inactive" | "pending";

function formatId(id: string | number): string {
  if (typeof id === "string") return id.padStart(8, "0");
  return id.toString();
}

// Discriminated unions — the most powerful real-world pattern
// Each member has a unique literal 'type' or 'kind' field
type ApiEvent =
  | { type: "REQUEST_STARTED";  url: string }
  | { type: "REQUEST_SUCCESS";  data: unknown; statusCode: number }
  | { type: "REQUEST_FAILED";   error: string; statusCode: number }
  | { type: "REQUEST_CANCELLED" };

function handleEvent(event: ApiEvent): void {
  switch (event.type) {
    case "REQUEST_STARTED":
      console.log(`Fetching: ${event.url}`);          // url exists here
      break;
    case "REQUEST_SUCCESS":
      console.log(`${event.statusCode}:`, event.data); // data + statusCode exist here
      break;
    case "REQUEST_FAILED":
      console.error(`${event.statusCode}: ${event.error}`);
      break;
    case "REQUEST_CANCELLED":
      console.log("Cancelled");
      break;
  }
}

// React-style typed reducer
type CounterAction =
  | { type: "INCREMENT" }
  | { type: "DECREMENT" }
  | { type: "RESET" }
  | { type: "SET"; payload: number }; // payload only on SET

function counterReducer(state: number, action: CounterAction): number {
  switch (action.type) {
    case "INCREMENT": return state + 1;
    case "DECREMENT": return state - 1;
    case "RESET":     return 0;
    case "SET":       return action.payload; // TS: payload available only here
  }
}
```

### Intersection Types (`&`) — "This AND That"

```typescript
// Intersection = combine ALL fields of both types
type WithTimestamps<T> = T & {
  createdAt: Date;
  updatedAt: Date;
};

type WithPagination<T> = T & {
  page: number;
  perPage: number;
  total: number;
};

// Usage: compose types without rewriting fields
type UserWithTimestamps  = WithTimestamps<User>;
type PaginatedUsers      = WithPagination<{ users: User[] }>;
type AdminUserTimestamped = WithTimestamps<User & { role: "admin" }>;

// Merging external interfaces
interface GeoData     { lat: number; lng: number; city: string }
interface BusinessData { revenue: number; employees: number }
type BusinessLocation  = GeoData & BusinessData;

const hq: BusinessLocation = {
  lat: 30.04, lng: 31.23, city: "Cairo",
  revenue: 1_000_000, employees: 50,
};
```

---

## 10. Narrowing & Type Guards

Narrowing is how TypeScript **refines** a wide type to a more specific one based on your code's control flow.

```typescript
// typeof — for primitives
function process(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // value: string
  }
  return value.toFixed(2);     // value: number
}

// instanceof — for class instances
function formatError(error: unknown): string {
  if (error instanceof Error)   return error.message;
  if (typeof error === "string") return error;
  return "An unknown error occurred";
}

// 'in' operator — check if a property exists on an object
type Cat = { meow(): void };
type Dog = { bark(): void };

function makeSound(animal: Cat | Dog) {
  if ("meow" in animal) {
    animal.meow(); // animal: Cat
  } else {
    animal.bark(); // animal: Dog
  }
}

// Discriminated union narrowing — the cleanest pattern
type Shape =
  | { kind: "circle";   radius: number }
  | { kind: "rect";     width: number; height: number }
  | { kind: "triangle"; base: number;  height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":   return Math.PI * shape.radius ** 2;
    case "rect":     return shape.width * shape.height;
    case "triangle": return 0.5 * shape.base * shape.height;
  }
}

// Custom type predicates — 'value is Type' return annotation
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "name" in value &&
    "email" in value
  );
}

function processResponse(data: unknown): User {
  if (isUser(data)) {
    return data; // TypeScript: data is User here
  }
  throw new Error("Invalid user data shape");
}

// Assertion functions (TS 3.7+) — narrow by throwing
function assertIsString(val: unknown): asserts val is string {
  if (typeof val !== "string") throw new Error(`Expected string, got ${typeof val}`);
}

function processInput(raw: unknown) {
  assertIsString(raw);
  raw.toUpperCase(); // TS knows: raw is string here
}
```

### 📖 Resources
- [typescriptlang.org — Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

---

## 11. Literal Types & `as const`

### Literal Types — Exact Values as Types

```typescript
// A type that only allows specific exact values
type Direction   = "north" | "south" | "east" | "west";
type DiceRoll    = 1 | 2 | 3 | 4 | 5 | 6;
type HttpMethod  = "GET" | "POST" | "PUT" | "PATCH" | "DELETE";
type StatusCode  = 200 | 201 | 400 | 401 | 403 | 404 | 500;

function move(direction: Direction, steps: number) { /* ... */ }
move("north", 5);      // OK
move("diagonal", 5);   // Error: "diagonal" is not a valid Direction

// Config type with literal fields
type Config = {
  env:      "development" | "staging" | "production";
  logLevel: "debug" | "info" | "warn" | "error";
  port:     number;
};
```

### `as const` — Lock Values to Literal Types

```typescript
// Without as const — TypeScript widens types
const env = "production"; // type: string (too wide)
const config = {
  env: "production", // type: string (not "production"!)
  port: 3000,        // type: number (not 3000)
};

// With as const — TypeScript keeps exact literal types
const env = "production" as const;    // type: "production"
const config = {
  env: "production",
  port: 3000,
} as const;
// config.env: "production", config.port: 3000 — exact literal types
// AND every field is readonly!

config.env = "staging"; // Error: Cannot assign to 'env' — readonly

// Most powerful use: derive a union type from an array
const ROLES = ["admin", "user", "moderator"] as const;
type Role = typeof ROLES[number]; // "admin" | "user" | "moderator"
// Without as const, this would be: string

const HTTP_METHODS = ["GET", "POST", "PUT", "DELETE"] as const;
type HttpMethod = typeof HTTP_METHODS[number]; // "GET" | "POST" | "PUT" | "DELETE"

// Practical: permission map typed from the source of truth
const PERMISSIONS = {
  admin:     ["read", "write", "delete", "manage"],
  user:      ["read"],
  moderator: ["read", "write"],
} as const;

type Role = keyof typeof PERMISSIONS;           // "admin" | "user" | "moderator"
type Permission = typeof PERMISSIONS[Role][number]; // "read" | "write" | "delete" | "manage"
```

---

## 12. Functions in TypeScript

```typescript
// Annotations for function parameters and return types
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Optional parameters (must come after required ones)
function greet(name: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${name}!`;
}

// Default parameters (type inferred from default)
function paginate(page = 1, limit = 20) {
  // page: number, limit: number — inferred from defaults
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}
sum(1, 2, 3, 4, 5); // 15

// Function overloads — same function name, different valid signatures
function getUser(id: number): User;
function getUser(email: string): User;
function getUser(idOrEmail: number | string): User {
  if (typeof idOrEmail === "number") return db.findById(idOrEmail);
  return db.findByEmail(idOrEmail);
}

// Type-safe callbacks
type EventCallback<T> = (event: T, index: number) => void;

function processEvents<T>(events: T[], callback: EventCallback<T>): void {
  events.forEach((event, i) => callback(event, i));
}

// Readonly parameters — prevent callers from mutating your array
function processUsers(users: readonly User[]): User[] {
  users.push({ id: 1, name: "test" }); // Error: readonly — protected!
  return users.filter(u => u.active);
}

// 'this' parameter (compile-time annotation only — disappears in output)
interface Button {
  label: string;
  onClick(this: Button, event: MouseEvent): void;
}
```

### 📖 Resources
- [typescriptlang.org — More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)

---

## 13. Generics — Write Once, Type Everything

Generics are type parameters — they let you write one function/class/interface that works with **any type** while remaining fully type-safe.

```typescript
// Without generics — lose type info
function identity(value: any): any { return value; }
const result = identity("hello"); // result: any — useless

// With generics — T is filled in at call time
function identity<T>(value: T): T { return value; }
const str = identity("hello"); // result: string
const num = identity(42);      // result: number
// TypeScript infers T — no need to pass it manually
```

### Generic Functions

```typescript
// First element of an array, typed correctly
function first<T>(arr: T[]): T | undefined { return arr[0]; }

first([1, 2, 3]);       // number | undefined
first(["a", "b"]);      // string | undefined
first([]);               // undefined

// Swap two values
function swap<T>(a: T, b: T): [T, T] { return [b, a]; }
swap(1, 2);     // [number, number]
swap("a", "b"); // [string, string]

// Zip two arrays
function zip<A, B>(a: A[], b: B[]): [A, B][] {
  return a.map((item, i) => [item, b[i]]);
}
zip([1, 2], ["a", "b"]); // [number, string][]

// Constraint — T must have a .length property
function logLength<T extends { length: number }>(value: T): T {
  console.log(`Length: ${value.length}`);
  return value;
}
logLength("hello");   // string has length — OK
logLength([1, 2, 3]); // array has length — OK
logLength(42);        // Error: number doesn't have length

// Generic with keyof constraint — type-safe property access
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]; // return type matches the actual property type
}

const user = { id: 1, name: "Ahmed", email: "a@b.com" };
getProperty(user, "name");   // string
getProperty(user, "id");     // number
getProperty(user, "phone");  // Error: 'phone' is not a key of typeof user
```

### Generic Interfaces & Classes

```typescript
// Generic repository interface — classic enterprise pattern
interface Repository<T, ID> {
  findById(id: ID): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: ID): Promise<void>;
}

class UserRepository implements Repository<User, string> {
  async findById(id: string): Promise<User | null> { /* db query */ return null; }
  async findAll(): Promise<User[]> { return []; }
  async save(user: User): Promise<User> { return user; }
  async delete(id: string): Promise<void> { /* */ }
}

// Generic Stack — type-safe data structure
class Stack<T> {
  private items: T[] = [];

  push(item: T): void { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
  peek(): T | undefined { return this.items[this.items.length - 1]; }
  isEmpty(): boolean { return this.items.length === 0; }
  get size(): number { return this.items.length; }
}

const numStack = new Stack<number>();
numStack.push(1);
numStack.push("two"); // Error: string not assignable to number

// Generic API wrapper
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
  timestamp: Date;
}

async function get<T>(url: string): Promise<ApiResponse<T>> {
  const res = await fetch(url);
  return res.json();
}

const { data: users } = await get<User[]>("/api/users");
users[0].name; // string — fully typed end-to-end
```

### 📖 Resources
- [typescriptlang.org — Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)

---

## 14. Classes in TypeScript

TypeScript adds access modifiers, readonly, abstract classes, and more to JS classes.

```typescript
class UserService {
  // Access modifiers
  public    id: number;      // accessible anywhere (default)
  private   password: string; // only inside this class
  protected role: string;     // this class and subclasses
  readonly  createdAt: Date;  // set once, never changed

  // Shorthand constructor — declares AND assigns fields in one line
  constructor(
    public  name:  string,
    private email: string,
    public  readonly id: string,
  ) {
    this.createdAt = new Date();
  }

  // Getter
  get displayName(): string {
    return `${this.name} <${this.email}>`;
  }

  // Setter with validation
  set userName(value: string) {
    if (value.length < 2) throw new Error("Name too short");
    this.name = value;
  }

  // Static — belongs to the class, not instances
  static create(name: string, email: string): UserService {
    return new UserService(name, email, crypto.randomUUID());
  }
}

// Inheritance
class AdminService extends UserService {
  constructor(name: string, email: string, public permissions: string[]) {
    super(name, email); // must call super() first
  }

  override get displayName(): string { // override keyword (TS 4.3+)
    return `[ADMIN] ${super.displayName}`;
  }
}

// Abstract classes — define structure without implementation
abstract class BaseRepository<T, ID> {
  abstract findById(id: ID): Promise<T | null>;
  abstract save(entity: T): Promise<T>;

  // Concrete shared behavior all subclasses get for free
  async findOrFail(id: ID): Promise<T> {
    const result = await this.findById(id);
    if (!result) throw new Error(`Entity ${String(id)} not found`);
    return result;
  }
}

class PostRepository extends BaseRepository<Post, string> {
  async findById(id: string): Promise<Post | null> { /* ... */ return null; }
  async save(post: Post): Promise<Post> { return post; }
}

// Implementing multiple interfaces
interface Printable  { print(): void }
interface Exportable { exportToCSV(): string }

class Report implements Printable, Exportable {
  print()         { console.log("Printing..."); }
  exportToCSV()   { return "col1,col2\n1,2"; }
}

// Private class fields — truly private, not just a convention (TS 4.3+)
class BankAccount {
  #balance = 0; // not accessible outside, even with type casting

  deposit(amount: number) { this.#balance += amount; }
  get balance(): number   { return this.#balance; }
}
```

### 📖 Resources
- [typescriptlang.org — Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)

---

## 15. Utility Types — The Built-in Toolkit

TypeScript ships with powerful built-in generic types that transform other types into new ones.

```typescript
// Our base type for all examples
interface User {
  id:        number;
  name:      string;
  email:     string;
  password:  string;
  age:       number;
  role:      "admin" | "user";
  createdAt: Date;
}
```

### Picking & Omitting Fields

```typescript
// Partial<T> — make ALL fields optional
type UpdateUserDto = Partial<User>;
// { id?: number; name?: string; email?: string; ... }

function updateUser(id: number, updates: Partial<User>) { /* */ }
updateUser(1, { name: "Ali" });           // just name — OK
updateUser(1, { name: "Ali", age: 26 }); // subset — OK

// Required<T> — make ALL fields required (reverse of Partial)
type StrictUser = Required<User>;

// Readonly<T> — make ALL fields readonly
type FrozenUser = Readonly<User>;
const frozen: FrozenUser = { id: 1, name: "Ahmed", /* ... */ } as FrozenUser;
frozen.name = "Ali"; // Error: Cannot assign to 'name' — readonly

// Pick<T, Keys> — keep only specified keys
type UserPreview = Pick<User, "id" | "name" | "role">;
// { id: number; name: string; role: "admin" | "user" }

// Omit<T, Keys> — remove specified keys
type PublicUser = Omit<User, "password" | "createdAt">;
// { id, name, email, age, role } — safe to return from API
```

### Set Operations on Union Types

```typescript
// Exclude<T, U> — remove U from union T
type AllEvents  = "click" | "focus" | "blur" | "mouseenter" | "mouseleave";
type MouseOnly  = Exclude<AllEvents, "focus" | "blur">;
// "click" | "mouseenter" | "mouseleave"

// Extract<T, U> — keep only U from union T
type FocusEvents = Extract<AllEvents, "focus" | "blur" | "scroll">;
// "focus" | "blur" (scroll wasn't in AllEvents)

// NonNullable<T> — remove null and undefined
type MaybeUser    = User | null | undefined;
type DefiniteUser = NonNullable<MaybeUser>; // User
```

### Function & Return Types

```typescript
// ReturnType<T> — get the return type of a function type
function getUser() { return { id: 1, name: "Ahmed", email: "a@b.com" }; }
type UserShape = ReturnType<typeof getUser>;
// { id: number; name: string; email: string }

// Super useful: get the type of a factory function's output
// without access to the original interface
import { createStore } from "redux";
type StoreType = ReturnType<typeof createStore>;

// Parameters<T> — get parameter types as a tuple
function makeRequest(url: string, method: string, body?: object): void {}
type RequestParams = Parameters<typeof makeRequest>;
// [url: string, method: string, body?: object]

// Awaited<T> — unwrap Promise types (TS 4.5+)
type AsyncUser      = Promise<User>;
type SyncUser       = Awaited<AsyncUser>; // User
type DeepNested     = Awaited<Promise<Promise<string>>>; // string — fully unwrapped
```

### Record — Typed Dictionaries

```typescript
// Record<Keys, Value> — typed key-value map
type UserMap = Record<string, User>;
const cache: UserMap = { "user-1": { id: 1, name: "Ahmed", /* ... */ } as User };

// Ensure all keys of a union are handled
type OrderStatus = "pending" | "processing" | "shipped" | "delivered" | "cancelled";
const statusLabels: Record<OrderStatus, string> = {
  pending:    "Awaiting Payment",
  processing: "Preparing Order",
  shipped:    "On The Way",
  delivered:  "Delivered",
  cancelled:  "Cancelled",
  // if you miss one key, TypeScript errors — exhaustiveness enforced
};
```

### 📖 Resources
- [typescriptlang.org — Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

---

## 16. Type Manipulation — Advanced

These are the power tools. They let you create types from other types programmatically.

### `keyof` — Get All Keys of a Type

```typescript
interface User { id: number; name: string; email: string; }

type UserKeys = keyof User; // "id" | "name" | "email"

// Type-safe generic property accessor
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]; // return type is T[K] — exact type of that field
}

const user = { id: 1, name: "Ahmed", email: "a@b.com" };
getProperty(user, "name");   // string
getProperty(user, "id");     // number
getProperty(user, "phone");  // Error: 'phone' is not keyof User

// Type-safe sort function
function sortBy<T>(arr: T[], key: keyof T): T[] {
  return [...arr].sort((a, b) => (a[key] > b[key] ? 1 : -1));
}
sortBy(users, "name");  // OK
sortBy(users, "age");   // OK
sortBy(users, "xyz");   // Error: compile-time catch
```

### Indexed Access Types — `T[K]`

```typescript
type User = { id: number; name: string; address: { city: string; zip: string } };

type UserId    = User["id"];             // number
type UserCity  = User["address"]["city"]; // string — nested access

// Get array element type
type Colors    = ["red", "green", "blue"];
type ColorType = Colors[number]; // "red" | "green" | "blue"

// Extract a deeply nested type from a large API type
type ApiResponse = {
  users: { id: number; posts: { id: number; title: string }[] }[];
};
type Post = ApiResponse["users"][number]["posts"][number];
// { id: number; title: string }
```

### Mapped Types — Transform Every Property

```typescript
// Syntax: { [K in keyof T]: NewType }

// Build Partial<T> yourself
type MyPartial<T> = { [K in keyof T]?: T[K] };

// Build Readonly<T> yourself
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };

// Remove readonly from all fields
type Mutable<T> = { -readonly [K in keyof T]: T[K] };

// Remove optionality from all fields
type MyRequired<T> = { [K in keyof T]-?: T[K] };

// Nullable version of any type
type Nullable<T> = { [K in keyof T]: T[K] | null };
type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; ... }

// Real-world: form state generated from a model
type FormState<T> = {
  [K in keyof T]: {
    value:   T[K];
    error:   string | null;
    touched: boolean;
  }
};
type UserFormState = FormState<Pick<User, "name" | "email">>;
// { name: { value: string; error: string | null; touched: boolean }; email: { ... } }
```

### Conditional Types — Types with Logic

```typescript
// T extends U ? TypeIfTrue : TypeIfFalse
type IsArray<T> = T extends any[] ? true : false;
type A = IsArray<string[]>; // true
type B = IsArray<string>;   // false

// Flatten an array type
type Flatten<T> = T extends Array<infer Item> ? Item : T;
type Str = Flatten<string[]>; // string
type Num = Flatten<number[]>; // number
type Unchanged = Flatten<boolean>; // boolean

// Unwrap Promise (recursive)
type UnwrapPromise<T> = T extends Promise<infer Inner> ? UnwrapPromise<Inner> : T;
type A2 = UnwrapPromise<Promise<number>>;                  // number
type B2 = UnwrapPromise<Promise<Promise<Promise<string>>>>; // string
```

### Template Literal Types — String Types With Logic

```typescript
type EventName = "click" | "focus" | "blur";

// Capitalize each member and prefix with 'on'
type HandlerName = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// CSS property combinator
type Side     = "Top" | "Right" | "Bottom" | "Left";
type SpaceProp = "margin" | "padding";
type CSSProps  = `${SpaceProp}${Side}`;
// "marginTop" | "marginRight" | ... | "paddingLeft"

// Type-safe event emitter key pattern
type DOMEvent = "click" | "keydown" | "mouseenter";
type ListenerKey = `on${Capitalize<DOMEvent>}`;
// "onClick" | "onKeydown" | "onMouseenter"
```

### 📖 Resources
- [typescriptlang.org — Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [typescriptlang.org — Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [typescriptlang.org — Template Literal Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)

---

## 17. Enums

```typescript
// Numeric enum — values are 0, 1, 2... by default
enum Direction { North, South, East, West }
Direction.North; // 0
Direction[0];    // "North" — reverse mapping exists for numeric enums

// Explicit numeric values
enum StatusCode {
  OK          = 200,
  Created     = 201,
  BadRequest  = 400,
  Unauthorized= 401,
  NotFound    = 404,
  ServerError = 500,
}

// String enum — most common in practice, debuggable in logs & JSON
enum UserRole {
  Admin     = "ADMIN",
  User      = "USER",
  Moderator = "MODERATOR",
}
UserRole.Admin; // "ADMIN"
// No reverse mapping for string enums

// Const enum — erased at compile time, values inlined
// Zero runtime overhead — no JS object generated
const enum LogLevel {
  Debug = "DEBUG",
  Info  = "INFO",
  Warn  = "WARN",
  Error = "ERROR",
}
console.log(LogLevel.Debug); // compiled to: console.log("DEBUG")

// When to prefer union types over enums
// Many modern TS devs use this instead — simpler, no JS object, works in .js files too
type UserRole  = "ADMIN" | "USER" | "MODERATOR";
type LogLevel  = "DEBUG" | "INFO" | "WARN" | "ERROR";

// Use actual enums when:
// - You need numeric enums with reverse mapping
// - You're working in OOP-heavy codebases (Angular, Java-adjacent)
// - You want const enums for zero-cost literals
```

### 📖 Resources
- [typescriptlang.org — Enums](https://www.typescriptlang.org/docs/handbook/enums.html)

---

## 18. Modules & Declaration Files

### `.d.ts` — Declaration Files

```typescript
// .d.ts files describe types for JavaScript libraries (no implementation)
// Example: if you use a plain JS utility lib without types
declare function formatDate(date: Date, format: string): string;
declare const VERSION: string;

// Most popular libraries have types on DefinitelyTyped:
npm install --save-dev @types/node
npm install --save-dev @types/express
npm install --save-dev @types/lodash

// Modern libraries (axios, zod, prisma) ship their own types — no @types needed
// Check package.json for "types" or "typings" field
```

### Module Augmentation — Extend Third-Party Types

```typescript
// Add a 'user' field to Express's Request type globally
// In: src/types/express.d.ts
import "express";

declare module "express" {
  interface Request {
    user?:     { id: string; role: "admin" | "user" };
    requestId: string;
  }
}

// Now anywhere in your Express app:
app.use((req, res, next) => {
  req.user = { id: "1", role: "admin" }; // TypeScript knows this field exists
  next();
});

// This pattern uses interface declaration merging — only works with interface, not type
```

---

## 19. tsconfig.json — Compiler Options

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "lib": ["ES2022", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true,

    // Strictness — ALWAYS enable in new projects
    "strict": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "noUncheckedIndexedAccess": true,

    // Extra safety catches
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,

    // Module resolution
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Why `strict: true` Is Non-Negotiable

```typescript
// Without strictNullChecks (the default WITHOUT strict!):
function getUser(id: number): User {
  return null; // No error! A landmine in disguise.
}

// With strictNullChecks:
function getUser(id: number): User {
  return null; // Error: 'null' is not assignable to 'User'
}
function getUser(id: number): User | null {
  return null; // Honest — caller knows to check
}

// Without noImplicitAny:
function process(data) { // data is silently 'any' — no protection at all
  data.whatever.you.want();
}

// With noImplicitAny:
function process(data) { // Error: 'data' implicitly has 'any' type
}
function process(data: unknown) { // must declare type explicitly
}
```

### 📖 Resources
- [typescriptlang.org — TSConfig Reference](https://www.typescriptlang.org/tsconfig/)

---

## 20. TypeScript in React (TSX)

```tsx
import { useState, useEffect, useRef, forwardRef } from "react";

// Props interface — always explicit
interface ButtonProps {
  label:     string;
  onClick:   () => void;
  variant?:  "primary" | "secondary" | "danger";
  disabled?: boolean;
  children?: React.ReactNode; // any valid JSX
}

const Button = ({ label, onClick, variant = "primary", disabled }: ButtonProps) => (
  <button onClick={onClick} disabled={disabled} className={`btn-${variant}`}>
    {label}
  </button>
);

// useState — inferred from initial value
const [count, setCount] = useState(0);               // number
const [name, setName]   = useState("");              // string
const [user, setUser]   = useState<User | null>(null); // explicit when initial is null

// useRef — typed with the DOM element
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus(); // HTMLInputElement | null — optional chain

// Event handlers — typed by the JSX element
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

// Generic component — works with any item type
interface ListProps<T> {
  items:        T[];
  renderItem:   (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, i) => (
        <li key={keyExtractor(item)}>{renderItem(item, i)}</li>
      ))}
    </ul>
  );
}

// Usage — T inferred as User
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>;

// Custom hook with generic return
function useLocalStorage<T>(key: string, initial: T): [T, (v: T) => void] {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initial;
  });
  const set = (v: T) => {
    setValue(v);
    localStorage.setItem(key, JSON.stringify(v));
  };
  return [value, set];
}

const [theme, setTheme] = useLocalStorage<"dark" | "light">("theme", "light");
```

---

## 21. Common Pitfalls & Anti-Patterns

```typescript
// 1. Using 'any' when 'unknown' is the right choice
function handleResponse(data: any) {
  data.user.profile.name; // TypeScript doesn't warn — crashes at runtime
}
// Fix:
function handleResponse(data: unknown) {
  if (isUser(data)) { data.name; } // must narrow first
}

// 2. Type assertions without validation ('as' overrides TypeScript blindly)
const user = response.data as User; // TS trusts you completely — dangerous
// Fix: use a schema validator
import { z } from "zod";
const UserSchema = z.object({ id: z.number(), name: z.string() });
const user = UserSchema.parse(response.data); // throws if invalid, typed if valid

// 3. Non-null assertion without checking
const el = document.querySelector("#email")!; // trusting it exists
el.addEventListener("input", handler);         // crashes if element is absent
// Fix:
const el = document.querySelector("#email");
if (el instanceof HTMLInputElement) {
  el.addEventListener("input", handler); // safe
}

// 4. Using @ts-ignore to silence a real bug
// @ts-ignore
const broken = wrongType.nonExistent; // ignores the error — hides the bug

// Use @ts-expect-error ONLY in tests for intentional error checking
// @ts-expect-error: testing invalid input path
expect(() => processUser(null)).toThrow();

// 5. Return type 'object' or 'any' — completely loses type info
function getConfig(): object { /* ... */ } // useless — tells you nothing
// Fix:
function getConfig(): AppConfig { /* ... */ }

// 6. Mutating state when readonly is expected
function process(items: readonly string[]) {
  (items as string[]).push("hack"); // bypasses readonly with cast
}
// Just don't. If you need to mutate, accept mutable array or create a copy.

// 7. Forgetting undefined in optional chain results
interface Config { server?: { port: number } }
const port = config.server?.port; // port: number | undefined
const doubled = port * 2; // Error: 'port' is possibly 'undefined'
// Fix:
const doubled = (port ?? 3000) * 2; // provide default

// 8. Using 'Function' type — too loose
function run(fn: Function) { fn(1, 2, 3); } // accepts anything callable
// Fix: specify the exact signature
function run(fn: (a: number, b: number) => number) { fn(1, 2); }

// 9. Not using 'satisfies' — losing literal types
const theme = { primary: "#3b82f6" } as ThemeConfig; // loses "#3b82f6" literal
// Fix: (TS 4.9+)
const theme = { primary: "#3b82f6" } satisfies ThemeConfig;
// TS checks it matches ThemeConfig AND keeps "#3b82f6" literal type

// 10. Writing duplicate types instead of deriving them
// Bad — if User changes, UpdateUserDto drifts out of sync
type UpdateUserDto = { name?: string; email?: string; age?: number };
// Good — always in sync with User
type UpdateUserDto = Partial<Pick<User, "name" | "email" | "age">>;
```

---

## 22. Interview Questions & Answers

### Fundamentals

**Q: What is TypeScript and why use it over JavaScript?**

TypeScript is a statically typed superset of JavaScript. Its primary value is catching type errors at compile time before they reach production. Secondary benefits: full autocomplete and IntelliSense in editors, safe refactoring (compiler flags every broken callsite), self-documenting code through types, and a much better developer experience on large teams. The cost is a build step and some learning curve; the payoff is fewer runtime bugs and faster development on anything past a small script.

**Q: Does TypeScript have any runtime overhead?**

Zero. TypeScript is entirely a compile-time tool. Types, interfaces, and generics are completely erased during compilation. The output is plain JavaScript — identical to what you'd write by hand, just with more confidence. This is why you can't do `typeof MyInterface` at runtime — it doesn't exist there.

**Q: What does `"strict": true` do in tsconfig?**

It enables a bundle of strict checks: `strictNullChecks` (null/undefined are not assignable to other types), `noImplicitAny` (can't have accidental any), `strictFunctionTypes`, `strictPropertyInitialization`, `noImplicitThis`, and more. The biggest one is `strictNullChecks` — without it, `null` can be silently assigned to any type, hiding entire classes of bugs.

---

### Types

**Q: What's the difference between `interface` and `type`?**

Both describe object shapes. The key practical differences: `interface` supports declaration merging (multiple `interface Window` declarations auto-combine), making it ideal for library types consumers can augment. `type` supports union types, intersection of arbitrary types, tuples, conditional types, and mapped types — things `interface` literally cannot express. Rule of thumb: `interface` for object shapes and class contracts; `type` for everything else.

**Q: `any` vs `unknown` — explain the difference and when to use each.**

`any` disables all TypeScript checking for a value — it propagates unsafely and tells TS to trust you completely. `unknown` says "this could be anything, but you must check the type before using it." Use `unknown` for: values from APIs, `JSON.parse`, catch blocks, and any external input. You must narrow `unknown` before calling methods on it, which is the whole point. Never use `any` except when migrating a JS file or as a very intentional suppression.

**Q: What is `never` and name two use cases.**

`never` is the type of values that can never exist. Use case 1: functions that always throw or loop forever — their return type is `never` because they never reach the end. Use case 2: exhaustive union checking — after you've narrowed every member of a union, the remaining type is `never`. If you pass that `never` to a function expecting `never`, adding a new union member without handling it becomes a compile-time error, not a runtime surprise.

---

### Generics

**Q: What are generics and why are they needed?**

Generics are type parameters — variables for types. Without them, you'd either write duplicate functions for each type or use `any` and lose type information. With generics, one function works with any type while preserving what type went in. Example: `function identity<T>(value: T): T` returns the same type that was passed. Without generics: the function either only works for one type (duplication) or accepts and returns `any` (useless typing).

**Q: What is a generic constraint?**

`T extends SomeType` restricts what types T can be. Example: `function getLength<T extends { length: number }>(val: T)` — you can only call this with arrays, strings, or anything with a `.length`. Without the constraint, TypeScript would error when you try to access `.length` on an unconstrained `T`.

---

### Utility Types

**Q: Name 5 utility types and their practical use cases.**

`Partial<T>` — PATCH endpoints (DTO where all fields are optional for updates). `Omit<T, K>` — strip `password` before returning user data from an API. `Pick<T, K>` — select only the fields a component's prop type needs. `ReturnType<T>` — get the shape of what a function returns without manually importing its return type. `Record<K, V>` — typed lookup map like `Record<OrderStatus, string>` where every status must have a label.

---

### Advanced

**Q: What is narrowing and what are the mechanisms?**

Narrowing is TypeScript refining a wider union type to a more specific member based on control flow. Mechanisms: `typeof` checks (primitives), `instanceof` checks (classes), `in` operator (property existence), truthiness checks (`if (val)`), discriminated union switch statements (checking a shared literal `kind` field), custom type predicates (`value is Type` return annotation), and assertion functions (`asserts value is Type`).

**Q: What is a mapped type and how would you build `Partial<T>` from scratch?**

A mapped type iterates over a union of keys and transforms each one. `type MyPartial<T> = { [K in keyof T]?: T[K] }`. The `[K in keyof T]` part loops over every key in T, `?` makes it optional, and `T[K]` preserves the original value type. Modifiers like `readonly` can be added, and `-?` or `-readonly` remove optionality or readonly.

**Q: What is the `satisfies` operator?**

`satisfies` (TS 4.9+) validates a value against a type while preserving the most specific literal types. With `as Config`, TypeScript treats the value as `Config` and loses any narrower types. With `satisfies Config`, it checks the shape matches `Config` but keeps literal types for inference. Most useful for config objects and palette maps where you want validation without losing specificity.

**Q: What is declaration merging and when would you use it?**

Declaration merging is when TypeScript automatically combines multiple `interface` declarations with the same name into one. The main real-world use is augmenting third-party library types — for example, adding a `user` property to Express's `Request` interface. You create a `.d.ts` file with `declare module "express" { interface Request { user?: MyUser } }` and it merges globally without modifying the library.

---

## 📖 Master Resource List

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) — start here
- [Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [Utility Types Reference](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [Type Manipulation Overview](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
- [TSConfig Reference](https://www.typescriptlang.org/tsconfig/)
- [TypeScript Playground](https://www.typescriptlang.org/play/)
- [TypeScript Cheat Sheets](https://www.typescriptlang.org/cheatsheets/)
- [Matt Pocock's Total TypeScript](https://www.totaltypescript.com/) — deep advanced content
- [type-challenges on GitHub](https://github.com/type-challenges/type-challenges) — hands-on practice

---

*TypeScript is just JavaScript that tells you when you're wrong before your users do. Go ace it, Ahmed. 🔷*

> **Legend:** `✅` = correct pattern | `❌` = avoid / bug | `🐛` = bug it prevents
