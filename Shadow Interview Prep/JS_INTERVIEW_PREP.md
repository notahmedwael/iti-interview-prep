# 🟨 JavaScript Interview Prep — Senior Engineer's Guide

> **From Ahmed's shadow interview prep — compiled by a senior SWE.**
> Covers everything from JS basics to advanced internals. Use this as a living checklist.

---

## 📑 Table of Contents

1. [The Big Picture — How JS Works](#1-the-big-picture--how-js-works)
2. [Types & Type Coercion](#2-types--type-coercion)
3. [Variables: var / let / const](#3-variables-var--let--const)
4. [Scope & Closures](#4-scope--closures)
5. [The `this` Keyword](#5-the-this-keyword)
6. [Prototypes & Inheritance](#6-prototypes--inheritance)
7. [Functions — All the Forms](#7-functions--all-the-forms)
8. [Arrays & Iteration](#8-arrays--iteration)
9. [Objects — Deep Dive](#9-objects--deep-dive)
10. [Asynchronous JavaScript](#10-asynchronous-javascript)
11. [ES6+ Modern Features](#11-es6-modern-features)
12. [Error Handling](#12-error-handling)
13. [The Weird JS Stuff ⚠️](#13-the-weird-js-stuff-%EF%B8%8F)
14. [Common Interview Pitfalls 🪤](#14-common-interview-pitfalls-)
15. [Advanced Topics 🚀](#15-advanced-topics-)
16. [Coding Patterns to Know](#16-coding-patterns-to-know)
17. [Quick-Fire Q&A Cheatsheet](#17-quick-fire-qa-cheatsheet)

---

## 1. The Big Picture — How JS Works

### Core Concepts
- **Single-threaded** — one call stack, one thing at a time.
- **Interpreted / JIT compiled** — V8 compiles to machine code at runtime.
- **Event-driven** — the Event Loop is the heart of async JS.

### The Event Loop
```
Call Stack → Web APIs → Callback Queue (Macro) → Microtask Queue → Call Stack
```
- **Microtasks** (Promises, `queueMicrotask`, MutationObserver) run *before* the next macro task.
- **Macro tasks** (setTimeout, setInterval, I/O, UI render) run after the microtask queue is empty.

```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// Output: 1, 4, 3, 2  ← Promise fires before setTimeout!
```

### 📖 Resources
- [javascript.info — Event Loop](https://javascript.info/event-loop)
- [MDN — Concurrency model and the event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [Jake Archibald's In The Loop (video)](https://www.youtube.com/watch?v=cCOL7MC4Pl0)
- [Loupe (visual tool)](http://latentflip.com/loupe/)

---

## 2. Types & Type Coercion

### Primitive Types (7)
```
string, number, bigint, boolean, undefined, null, symbol
```
Everything else is an **object** (including arrays, functions, dates).

### `typeof` quirks
```js
typeof null        // "object"  ← famous bug, kept for legacy
typeof function(){} // "function" ← special case
typeof NaN          // "number"  ← NaN is technically a number
typeof undefined    // "undefined"
typeof []           // "object"
```

### Equality: `==` vs `===`
- `===` — strict equality, no coercion.
- `==` — abstract equality, triggers coercion rules (the source of many bugs).

```js
0 == false    // true  (false coerces to 0)
"" == false   // true
null == undefined // true (special rule)
null == 0     // false (null only == undefined)
NaN == NaN    // false (NaN is not equal to anything, including itself)
```

### Type Coercion Rules
```js
// String + anything = string concatenation
"5" + 3       // "53"
"5" - 3       // 2  (- triggers numeric conversion)
+"5"          // 5  (unary + converts to number)
+""           // 0
+null         // 0
+undefined    // NaN
+[]           // 0
+{}           // NaN
```

### Checking Types Reliably
```js
Array.isArray([])             // true (best for arrays)
Object.prototype.toString.call(new Date()) // "[object Date]"
Number.isNaN(NaN)             // true (safer than global isNaN)
Number.isFinite(Infinity)     // false
```

### 📖 Resources
- [javascript.info — Data Types](https://javascript.info/types)
- [javascript.info — Type Conversions](https://javascript.info/type-conversions)
- [javascript.info — Comparisons](https://javascript.info/comparison)
- [MDN — Equality comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)
- [The Abstract Equality Comparison Algorithm (spec)](https://tc39.es/ecma262/#sec-abstract-equality-comparison)

---

## 3. Variables: var / let / const

| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | function | block | block |
| Hoisted | yes (as `undefined`) | yes (TDZ) | yes (TDZ) |
| Re-declare | yes | no | no |
| Re-assign | yes | yes | no |
| Global property | yes (`window.x`) | no | no |

### Hoisting
```js
console.log(a); // undefined (var is hoisted, initialized to undefined)
var a = 5;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 5;
```

### Temporal Dead Zone (TDZ)
`let` and `const` are hoisted but NOT initialized — accessing them before declaration throws a `ReferenceError`. The gap between the start of the block and the declaration is the TDZ.

### `const` ≠ immutable
```js
const obj = { name: "Ahmed" };
obj.name = "Ali"; // ✅ works — the reference is constant, not the value
obj = {};         // ❌ TypeError — can't reassign the binding
```
Use `Object.freeze()` for shallow immutability.

### 📖 Resources
- [javascript.info — Variables](https://javascript.info/variables)
- [javascript.info — The old "var"](https://javascript.info/var)
- [MDN — let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)

---

## 4. Scope & Closures

### Scope Chain
When JS looks up a variable, it starts in the current scope and walks **up the chain** until it finds it or hits global scope.

### Closures
A closure is a function that **remembers its outer lexical environment** even after the outer function has returned.

```js
function makeCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    get: () => count,
  };
}

const counter = makeCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.get();       // 2 — `count` lives on in the closure
```

### Classic Closure Trap (var in loops)
```js
// ❌ Bug: all callbacks share the same `i`
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3
}

// ✅ Fix 1: use let (block-scoped per iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2
}

// ✅ Fix 2: IIFE to capture i
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i);
}
```

### Module Pattern (practical closure)
```js
const BankAccount = (() => {
  let balance = 0; // private
  return {
    deposit: (n) => (balance += n),
    getBalance: () => balance,
  };
})();
```

### 📖 Resources
- [javascript.info — Closure](https://javascript.info/closure)
- [MDN — Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [You Don't Know JS — Scope & Closures](https://github.com/getify/You-Dont-Know-JS/blob/2nd-ed/scope-closures/README.md)

---

## 5. The `this` Keyword

`this` is determined by **how a function is called**, not where it is defined (except arrow functions).

| Call type | `this` value |
|---|---|
| Regular function (non-strict) | `window` / `global` |
| Regular function (strict mode) | `undefined` |
| Method call (`obj.fn()`) | `obj` |
| `new` call | newly created object |
| Arrow function | lexical `this` (inherited from enclosing scope) |
| `call/apply/bind` | explicitly set |

```js
const obj = {
  name: "Ahmed",
  greet() {
    console.log(this.name); // "Ahmed" — method call
  },
  greetArrow: () => {
    console.log(this.name); // undefined — arrow captures outer `this` (global/undefined)
  }
};

const fn = obj.greet;
fn(); // undefined — lost context!
fn.call(obj); // "Ahmed" — explicitly bound
```

### `bind`, `call`, `apply`
```js
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const user = { name: "Ahmed" };
greet.call(user, "Hello", "!");    // args spread
greet.apply(user, ["Hello", "!"]); // args as array
const boundGreet = greet.bind(user, "Hello"); // returns new function
boundGreet("!");
```

### 📖 Resources
- [javascript.info — Object methods, "this"](https://javascript.info/object-methods)
- [javascript.info — Arrow functions revisited](https://javascript.info/arrow-functions)
- [MDN — this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)

---

## 6. Prototypes & Inheritance

### Prototype Chain
Every object has an internal `[[Prototype]]` link. When you access a property, JS walks up the chain until it finds it or hits `null`.

```js
const animal = { breathes: true };
const dog = Object.create(animal);
dog.bark = () => "woof";

dog.breathes; // true — found on animal via prototype chain
dog.hasOwnProperty("breathes"); // false — it's inherited
```

### `class` is syntactic sugar
```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} makes a sound`; }
}

class Dog extends Animal {
  speak() { return `${this.name} barks`; }
}

const d = new Dog("Rex");
d.speak();          // "Rex barks"
d instanceof Dog;   // true
d instanceof Animal; // true
```

### How `new` Works
1. Create a new empty object `{}`
2. Set its `[[Prototype]]` to `Constructor.prototype`
3. Call the constructor with `this` = new object
4. Return the object (unless constructor returns an object explicitly)

### 📖 Resources
- [javascript.info — Prototypal inheritance](https://javascript.info/prototype-inheritance)
- [javascript.info — Classes](https://javascript.info/classes)
- [MDN — Object prototypes](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)

---

## 7. Functions — All the Forms

```js
// Function declaration (hoisted fully)
function add(a, b) { return a + b; }

// Function expression (not hoisted)
const add = function(a, b) { return a + b; };

// Arrow function (no own `this`, `arguments`)
const add = (a, b) => a + b;

// Named function expression (name only accessible inside)
const factorial = function fact(n) { return n <= 1 ? 1 : n * fact(n - 1); };

// IIFE
(function() { /* runs immediately */ })();
(() => { /* arrow IIFE */ })();

// Generator function
function* range(start, end) {
  for (let i = start; i < end; i++) yield i;
}
[...range(0, 3)]; // [0, 1, 2]

// Async function
async function fetchData(url) {
  const res = await fetch(url);
  return res.json();
}
```

### `arguments` vs rest params
```js
function legacy() {
  console.log(arguments); // array-like, not real array
}

function modern(...args) {
  console.log(args); // real array ✅
}
```

### Default Parameters
```js
function greet(name = "World") { return `Hello, ${name}`; }
// Default is evaluated at call time, not definition time
```

### 📖 Resources
- [javascript.info — Functions](https://javascript.info/function-basics)
- [javascript.info — Arrow functions](https://javascript.info/arrow-functions-basics)
- [javascript.info — Generators](https://javascript.info/generators)

---

## 8. Arrays & Iteration

### Must-Know Array Methods
```js
const nums = [1, 2, 3, 4, 5];

nums.map(x => x * 2);          // [2,4,6,8,10] — transform
nums.filter(x => x % 2 === 0); // [2,4] — keep matches
nums.reduce((acc, x) => acc + x, 0); // 15 — fold
nums.find(x => x > 3);         // 4 — first match
nums.findIndex(x => x > 3);    // 3
nums.some(x => x > 4);         // true — any match?
nums.every(x => x > 0);        // true — all match?
nums.flat();                   // flatten one level
nums.flatMap(x => [x, x * 2]); // map then flatten
[...new Set(nums)];             // deduplicate
```

### Mutating vs Non-Mutating
```js
// Mutating (changes original array)
arr.push(), arr.pop(), arr.shift(), arr.unshift()
arr.splice(), arr.sort(), arr.reverse(), arr.fill()

// Non-mutating (returns new array)
arr.map(), arr.filter(), arr.reduce()
arr.slice(), arr.concat(), arr.flat(), arr.flatMap()
[...arr].reverse() // non-mutating reverse
```

### Sorting Gotcha
```js
[10, 9, 2, 1, 100].sort(); // [1, 10, 100, 2, 9] — lexicographic ❌
[10, 9, 2, 1, 100].sort((a, b) => a - b); // [1, 2, 9, 10, 100] ✅
```

### 📖 Resources
- [javascript.info — Arrays](https://javascript.info/array)
- [javascript.info — Array methods](https://javascript.info/array-methods)
- [MDN — Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)

---

## 9. Objects — Deep Dive

### Property Descriptors
```js
Object.defineProperty(obj, "key", {
  value: 42,
  writable: false,    // can't change the value
  enumerable: false,  // won't show in for...in or Object.keys()
  configurable: false // can't delete or reconfigure
});
```

### Object Methods
```js
Object.keys(obj)    // own enumerable keys
Object.values(obj)  // own enumerable values
Object.entries(obj) // [key, value] pairs
Object.assign({}, a, b) // shallow merge (mutates first arg)
Object.freeze(obj)  // shallow immutability
Object.create(proto) // create with specific prototype
Object.fromEntries([["a", 1], ["b", 2]]) // {a:1, b:2}
```

### Spread vs `Object.assign`
```js
const merged = { ...a, ...b }; // same as Object.assign({}, a, b)
// Both are SHALLOW copies — nested objects are still references!
```

### Deep Clone
```js
// ✅ Modern (handles most cases)
structuredClone(obj);

// ✅ Simple hack (no functions, dates, undefined)
JSON.parse(JSON.stringify(obj));

// ✅ Lodash
_.cloneDeep(obj);
```

### Optional Chaining & Nullish Coalescing
```js
const city = user?.address?.city;         // undefined if any is null/undefined
const name = user?.name ?? "Anonymous";   // fallback only on null/undefined
const len = arr?.length ?? 0;
```

### 📖 Resources
- [javascript.info — Objects](https://javascript.info/object)
- [javascript.info — Property flags](https://javascript.info/property-descriptors)
- [MDN — Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

---

## 10. Asynchronous JavaScript

### Callback Hell → Promises → async/await

```js
// Callbacks — pyramid of doom
getData(function(a) {
  getMore(a, function(b) {
    getEvenMore(b, function(c) { /* ... */ });
  });
});

// Promises — chainable
getData()
  .then(a => getMore(a))
  .then(b => getEvenMore(b))
  .catch(err => console.error(err))
  .finally(() => cleanup());

// async/await — reads like sync code
async function run() {
  try {
    const a = await getData();
    const b = await getMore(a);
    const c = await getEvenMore(b);
  } catch (err) {
    console.error(err);
  }
}
```

### Promise Combinators
```js
// All must resolve, or it rejects on first error
await Promise.all([p1, p2, p3]);

// Resolves/rejects with the first settled promise
await Promise.race([p1, p2, p3]);

// Waits for all to settle (never rejects)
await Promise.allSettled([p1, p2, p3]);
// Returns: [{status: "fulfilled", value}, {status: "rejected", reason}]

// First to RESOLVE (ignores rejections unless all fail)
await Promise.any([p1, p2, p3]);
```

### Parallel vs Sequential
```js
// ❌ Sequential (slow — each waits for the previous)
const a = await fetch(url1);
const b = await fetch(url2);

// ✅ Parallel (fast — fire both at once)
const [a, b] = await Promise.all([fetch(url1), fetch(url2)]);
```

### Creating Promises
```js
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const fetchWithTimeout = (url, timeout) =>
  Promise.race([fetch(url), delay(timeout).then(() => { throw new Error("Timeout"); })]);
```

### 📖 Resources
- [javascript.info — Promises](https://javascript.info/promise-basics)
- [javascript.info — async/await](https://javascript.info/async-await)
- [MDN — Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [javascript.info — Promise API](https://javascript.info/promise-api)

---

## 11. ES6+ Modern Features

### Destructuring
```js
// Array
const [first, second, ...rest] = [1, 2, 3, 4];

// Object
const { name, age = 25, address: { city } = {} } = user;

// Function params
function render({ title, body = "", isPublished = false }) { /* ... */ }
```

### Spread & Rest
```js
// Spread — expand into individual elements
const merged = [...arr1, ...arr2];
const copy = { ...obj };

// Rest — collect remaining into array
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
```

### Template Literals
```js
const html = `
  <div class="${cls}">
    ${items.map(i => `<li>${i}</li>`).join('')}
  </div>
`;

// Tagged templates
function sql(strings, ...values) { /* sanitize values */ }
sql`SELECT * FROM users WHERE id = ${userId}`;
```

### Symbols
```js
const id = Symbol("id"); // unique, not enumerable by default
obj[id] = 42;
// Symbols are invisible to for...in, Object.keys(), JSON.stringify()
```

### Map & Set
```js
// Map — keyed by anything (vs Object which only accepts strings/Symbols)
const map = new Map();
map.set(objKey, "value"); // object as key!
map.get(objKey);
map.has(objKey);
[...map.entries()];

// Set — unique values
const unique = new Set([1, 2, 2, 3]); // {1, 2, 3}
[...unique]; // [1, 2, 3]
```

### WeakMap & WeakRef
- `WeakMap` — keys must be objects, not enumerable, allows GC if key is unreferenced.
- Use case: storing private metadata without preventing garbage collection.

### 📖 Resources
- [javascript.info — Destructuring](https://javascript.info/destructuring-assignment)
- [javascript.info — Map and Set](https://javascript.info/map-set)
- [javascript.info — Symbol](https://javascript.info/symbol)
- [javascript.info — Proxy and Reflect](https://javascript.info/proxy)

---

## 12. Error Handling

```js
// Sync
try {
  riskyOp();
} catch (err) {
  if (err instanceof TypeError) { /* specific */ }
  else throw err; // re-throw unknowns!
} finally {
  cleanup(); // always runs
}

// Async
async function run() {
  try {
    await riskyAsync();
  } catch (err) {
    // handles both sync throws AND rejected promises
  }
}

// Custom errors
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}
throw new ValidationError("Invalid email", "email");
```

### Unhandled Rejections
```js
// Always handle promise rejections!
process.on("unhandledRejection", (err) => { /* log and crash gracefully */ });
window.addEventListener("unhandledrejection", (event) => { /* browser */ });
```

### 📖 Resources
- [javascript.info — Error handling](https://javascript.info/try-catch)
- [javascript.info — Custom errors](https://javascript.info/custom-errors)

---

## 13. The Weird JS Stuff ⚠️

These come up in interviews to test deep understanding. Know them cold.

```js
// 1. typeof null is "object" — historical bug, never fixed
typeof null === "object"; // true

// 2. NaN is not equal to itself
NaN === NaN; // false
Number.isNaN(NaN); // true (the safe check)

// 3. Array + Array
[] + [];  // ""  (both coerce to "")
[] + {};  // "[object Object]"
{} + [];  // 0  (when {} is parsed as a block, +[] = 0)
({}) + []; // "[object Object]" (force {} as expression)

// 4. parseInt surprises
parseInt("09");    // 9 (modern engines) — always pass radix!
parseInt("0x10");  // 16 (hex!)
["1", "2", "3"].map(parseInt); // [1, NaN, NaN] !!! 
// Because map passes (value, index, array) → parseInt("1",0), parseInt("2",1), parseInt("3",2)
// Fix:
["1", "2", "3"].map(Number); // [1, 2, 3]

// 5. Floating point
0.1 + 0.2 === 0.3; // false! (0.30000000000000004)
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON; // true — correct comparison

// 6. Loose equality surprises
false == "0"; // true
false == 0;   // true
"0" == 0;     // true
// But: false == "0" == 0, yet "0" !== false... transitivity broken!

// 7. typeof function
typeof class {} === "function"; // true
typeof (() => {}) === "function"; // true

// 8. delete operator
const obj = { a: 1 };
delete obj.a; // true
obj.a; // undefined

// 9. void operator
void 0 === undefined; // true
void "anything" === undefined; // always undefined

// 10. Arguments object is not an array
function f() {
  return Array.from(arguments).map(x => x * 2); // convert first
}

// 11. Automatic semicolon insertion (ASI)
function foo() {
  return    // ASI inserts ; here!
  { value: 42 };
}
foo(); // undefined (not {value: 42})

// 12. Label statements
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) break outer; // breaks the outer loop
  }
}

// 13. Comma operator
const x = (1, 2, 3); // x = 3 (evaluates all, returns last)
```

### 📖 Resources
- [javascript.info — "Weird" JS behaviors](https://javascript.info/javascript-specials)
- [wtfjs — Collection of JS quirks](https://github.com/denysdovhan/wtfjs)
- [Gary Bernhardt's WAT Talk](https://www.destroyallsoftware.com/talks/wat)

---

## 14. Common Interview Pitfalls 🪤

### 1. Mutating vs Copying
```js
// ❌ Accidentally mutates original
function addItem(arr, item) {
  arr.push(item);  // mutates!
  return arr;
}

// ✅ Returns new array
function addItem(arr, item) {
  return [...arr, item];
}
```

### 2. `this` context lost in callbacks
```js
class Timer {
  constructor() { this.count = 0; }
  
  // ❌ `this` is lost inside setTimeout callback
  start() { setTimeout(function() { this.count++; }, 100); }
  
  // ✅ Arrow function inherits `this`
  start() { setTimeout(() => { this.count++; }, 100); }
}
```

### 3. Async in loops
```js
// ❌ forEach doesn't await
arr.forEach(async (item) => { await process(item); }); // runs in parallel, not sequence

// ✅ Sequential
for (const item of arr) { await process(item); }

// ✅ Parallel with all
await Promise.all(arr.map(item => process(item)));
```

### 4. Reference vs Value
```js
// Primitives — copied by value
let a = 5, b = a;
b = 10; // a is still 5

// Objects — copied by reference
let obj1 = { x: 1 };
let obj2 = obj1;
obj2.x = 99; // obj1.x is also 99!
```

### 5. `sort` mutates in place
```js
const original = [3, 1, 2];
const sorted = original.sort(); // mutates original!
// ✅ const sorted = [...original].sort();
```

### 6. Closure in Class Fields
```js
class Button {
  // ❌ Each call to handleClick creates a new function (bad for listeners)
  handleClick = () => { /* ... */ };
  
  // ✅ Defined on prototype (shared)
  handleClick() { /* ... */ }
}
```

### 7. Promise executor runs synchronously
```js
new Promise((resolve) => {
  console.log("A"); // runs immediately, synchronously
  resolve();
}).then(() => console.log("C")); // microtask, after current sync
console.log("B");
// Output: A, B, C
```

### 8. Object as Map key problem
```js
const map = {};
const key1 = { id: 1 };
const key2 = { id: 2 };
map[key1] = "first";
map[key2] = "second";
console.log(map); // { "[object Object]": "second" } — both coerce to same string!
// ✅ Use Map instead
```

---

## 15. Advanced Topics 🚀

### Currying
```js
const curry = fn => {
  const arity = fn.length;
  return function curried(...args) {
    if (args.length >= arity) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
};

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3); // 6
add(1, 2)(3); // 6
add(1)(2, 3); // 6
```

### Memoization
```js
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveFn = memoize((n) => { /* heavy computation */ });
```

### Debounce & Throttle
```js
// Debounce — delay execution until after calls stop
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Throttle — ensure fn runs at most once per interval
function throttle(fn, limit) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}
```

### Proxy & Reflect
```js
const handler = {
  get(target, key) {
    return key in target ? target[key] : `Property "${key}" not found`;
  },
  set(target, key, value) {
    if (typeof value !== "number") throw new TypeError("Only numbers!");
    target[key] = value;
    return true; // must return true!
  }
};

const proxy = new Proxy({}, handler);
proxy.age = 25;
proxy.name; // 'Property "name" not found'
```

### Iterators & Iterables
```js
// Custom iterable
const range = {
  from: 1, to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { value: undefined, done: true };
      }
    };
  }
};

[...range]; // [1, 2, 3, 4, 5]
for (const n of range) { /* ... */ }
```

### async Iterators
```js
async function* paginate(url) {
  let page = 1;
  while (true) {
    const data = await fetch(`${url}?page=${page++}`).then(r => r.json());
    if (!data.length) return;
    yield data;
  }
}

for await (const page of paginate("/api/items")) {
  render(page);
}
```

### Observability — Proxy-based reactive
```js
function reactive(obj, onChange) {
  return new Proxy(obj, {
    set(target, key, value) {
      const old = target[key];
      target[key] = value;
      onChange(key, value, old);
      return true;
    }
  });
}
```

### 📖 Resources
- [javascript.info — Proxy and Reflect](https://javascript.info/proxy)
- [javascript.info — Iterables](https://javascript.info/iterable)
- [javascript.info — Generators / async iteration](https://javascript.info/async-iterators-generators)
- [javascript.info — Currying](https://javascript.info/currying-partials)
- [Addy Osmani — Learning Patterns](https://www.patterns.dev/vanilla/)

---

## 16. Coding Patterns to Know

### Deep Clone (custom)
```js
function deepClone(obj, seen = new Map()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (seen.has(obj)) return seen.get(obj); // handle circular refs
  const clone = Array.isArray(obj) ? [] : {};
  seen.set(obj, clone);
  for (const key of Reflect.ownKeys(obj)) {
    clone[key] = deepClone(obj[key], seen);
  }
  return clone;
}
```

### Event Emitter
```js
class EventEmitter {
  #listeners = new Map();
  on(event, fn) {
    if (!this.#listeners.has(event)) this.#listeners.set(event, []);
    this.#listeners.get(event).push(fn);
    return this;
  }
  off(event, fn) {
    this.#listeners.set(event, (this.#listeners.get(event) || []).filter(f => f !== fn));
  }
  emit(event, ...args) {
    (this.#listeners.get(event) || []).forEach(fn => fn(...args));
  }
}
```

### Pipeline / Compose
```js
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

const transform = pipe(
  str => str.trim(),
  str => str.toLowerCase(),
  str => str.replace(/ /g, "-")
);

transform("  Hello World  "); // "hello-world"
```

### Retry with backoff
```js
async function retry(fn, retries = 3, delay = 300) {
  for (let i = 0; i <= retries; i++) {
    try {
      return await fn();
    } catch (err) {
      if (i === retries) throw err;
      await new Promise(r => setTimeout(r, delay * 2 ** i)); // exponential backoff
    }
  }
}
```

---

## 17. Quick-Fire Q&A Cheatsheet

| Question | Answer |
|---|---|
| `null == undefined`? | `true` (special case in spec) |
| `null === undefined`? | `false` |
| `typeof NaN`? | `"number"` |
| What is the TDZ? | Period between entering scope and let/const declaration — accessing throws ReferenceError |
| `[] == ![]`? | `true` — `![]` is `false`, then `[] == false` → `"" == 0` → `0 == 0` |
| Is `class` a new paradigm? | No, it's syntactic sugar over prototypal inheritance |
| `Promise.all` vs `allSettled`? | `all` rejects on first failure; `allSettled` waits for all to settle |
| Arrow function vs regular? | Arrow has no own `this`, `arguments`, can't be `new`-called, no prototype |
| What does `Object.freeze` do? | Prevents adding, removing, or changing properties (shallow) |
| How does GC work in JS? | Mark-and-sweep: mark all reachable objects, sweep the rest |
| What is a Symbol? | A unique, non-enumerable primitive, often used as object keys |
| `for...in` vs `for...of`? | `for...in` iterates keys (including inherited); `for...of` iterates values of iterables |
| What is `use strict`? | Opt-in to strict mode: disables sloppy behaviors, `this` is undefined in functions |
| Microtask vs Macrotask? | Microtasks (Promises) run before next macrotask (setTimeout); higher priority |
| What is `arguments`? | Array-like object in regular functions containing all arguments; not available in arrows |

---

## 🎯 Interview Day Tips (from a senior SWE)

1. **Think out loud** — interviewers value your reasoning more than your final answer.
2. **Start with edge cases** — what if the input is null? An empty array? Negative?
3. **Talk about tradeoffs** — time vs space, readability vs performance.
4. **Use modern syntax** — show you know ES6+, but explain what it does.
5. **When stuck** — ask a clarifying question, not for a hint. "Can I assume the array is sorted?" etc.
6. **Don't panic on weird JS questions** — they're testing curiosity, not memorization. Say "I know this is a coercion quirk, let me trace through it."
7. **Know when to use `Map` over `Object`** — size property, any key type, insertion order guaranteed.
8. **Performance vocabulary** — "this runs in O(n) time and O(1) space" impresses.

---

*Good luck, Ahmed. You've got this. 🚀*

> **Legend:** 📖 = Read this | ⚠️ = Watch out | ✅ = Do this | ❌ = Avoid this
