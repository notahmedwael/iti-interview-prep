# 🟢 Node.js · Express · MongoDB · Mongoose — Senior Engineer's Interview Guide

> **Part 2 of Ahmed's shadow interview prep.**
> Covers Node internals, Express patterns, MongoDB design, Mongoose ODM, and the full async story.

---

## 📑 Table of Contents

1. [What is Node.js?](#1-what-is-nodejs)
2. [The Node.js Event Loop — Deep Dive](#2-the-nodejs-event-loop--deep-dive)
3. [Browser Event Loop vs Node Event Loop](#3-browser-event-loop-vs-node-event-loop)
4. [Don't Block the Event Loop ⚡](#4-dont-block-the-event-loop-)
5. [Node.js Core Modules](#5-nodejs-core-modules)
6. [Streams & Buffers](#6-streams--buffers)
7. [Worker Threads & Child Processes](#7-worker-threads--child-processes)
8. [Express.js — Fundamentals](#8-expressjs--fundamentals)
9. [Express — Middleware Deep Dive](#9-express--middleware-deep-dive)
10. [Express — Routing & Controllers](#10-express--routing--controllers)
11. [Express — Error Handling](#11-express--error-handling)
12. [REST API Design Patterns](#12-rest-api-design-patterns)
13. [Authentication & Security](#13-authentication--security)
14. [ORM vs ODM vs Query Builders — The Full Picture](#14-orm-vs-odm-vs-query-builders--the-full-picture)
15. [MongoDB — Core Concepts](#15-mongodb--core-concepts)
16. [MongoDB — Queries & Aggregation](#16-mongodb--queries--aggregation)
17. [MongoDB — Indexing & Performance](#17-mongodb--indexing--performance)
18. [Mongoose — ODM Deep Dive](#18-mongoose--odm-deep-dive)
19. [Mongoose — Schema Design Patterns](#19-mongoose--schema-design-patterns)
20. [Common Pitfalls 🪤](#20-common-pitfalls-)
21. [Quick-Fire Q&A Cheatsheet](#21-quick-fire-qa-cheatsheet)

---

## 1. What is Node.js?

Node.js is a **JavaScript runtime built on Chrome's V8 engine**. It lets you run JavaScript on the server side. Key characteristics:

- **Non-blocking I/O** — I/O calls (disk, network, DB) don't stall the process; a callback/promise is called when the work is done.
- **Single-threaded** — one main thread runs your JS code, but I/O is offloaded to the OS or libuv's thread pool.
- **Event-driven** — everything revolves around the Event Loop.
- **Built on libuv** — a C library that handles async I/O, the event loop, thread pool, and OS abstractions.

```
Your JS Code
     │
     ▼
   V8 Engine (compile + execute JS)
     │
     ▼
Node.js Bindings (C++ bridge)
     │
     ▼
libuv (Event Loop + Thread Pool + OS async I/O)
```

### 📖 Resources
- [Node.js official docs](https://nodejs.org/en/docs)
- [nodejs.org — About Node.js](https://nodejs.org/en/about)
- [The Art of Node (great intro)](https://github.com/maxogden/art-of-node)

---

## 2. The Node.js Event Loop — Deep Dive

This is the most important concept in Node. The event loop is what allows Node to perform non-blocking I/O despite being single-threaded.

### The 6 Phases (libuv)

```
   ┌─────────────────────────────┐
   │           timers            │  ← setTimeout / setInterval callbacks
   └──────────────┬──────────────┘
                  │
   ┌──────────────▼──────────────┐
   │     pending callbacks       │  ← I/O errors deferred from previous tick
   └──────────────┬──────────────┘
                  │
   ┌──────────────▼──────────────┐
   │       idle, prepare         │  ← internal use only
   └──────────────┬──────────────┘
                  │
   ┌──────────────▼──────────────┐
   │           poll              │  ← fetch new I/O events, execute I/O callbacks
   └──────────────┬──────────────┘
                  │
   ┌──────────────▼──────────────┐
   │           check             │  ← setImmediate callbacks
   └──────────────┬──────────────┘
                  │
   ┌──────────────▼──────────────┐
   │      close callbacks        │  ← socket.on('close', ...) etc.
   └─────────────────────────────┘
                  │
   (loop again or exit if nothing pending)
```

### Microtasks — Highest Priority
Between **every phase** (and between every callback within a phase in Node 11+), Node drains:
1. `process.nextTick()` queue — runs first (even before Promises!)
2. Promise microtask queue (`.then` callbacks)

```js
setTimeout(() => console.log("setTimeout"), 0);
setImmediate(() => console.log("setImmediate"));
Promise.resolve().then(() => console.log("Promise"));
process.nextTick(() => console.log("nextTick"));

// Output:
// nextTick       ← highest priority, runs before everything
// Promise        ← microtask queue
// setTimeout     ← timers phase (order may vary vs setImmediate)
// setImmediate   ← check phase
```

### `setTimeout(fn, 0)` vs `setImmediate()`
```js
// Inside I/O callback — setImmediate ALWAYS fires first
fs.readFile("file.txt", () => {
  setTimeout(() => console.log("timeout"), 0);
  setImmediate(() => console.log("immediate"));
  // Output: immediate, timeout — guaranteed in I/O context
});

// At top level — order is NOT guaranteed (depends on system timer resolution)
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
// Could be either order
```

### The Poll Phase — The Heart of I/O
The poll phase:
1. Calculates how long to block and poll for I/O
2. Processes events in the poll queue (file reads, network responses)
3. If the poll queue is empty, checks if `setImmediate` is scheduled → moves to check phase
4. Otherwise waits for I/O (up to a timer threshold)

### `process.nextTick` — Use Carefully
`nextTick` runs before I/O, before Promises, before everything else queued. Overusing it can **starve I/O**.

```js
// ❌ Recursive nextTick — starves everything!
function badRecursion() {
  process.nextTick(badRecursion);
}

// ✅ Use setImmediate for recursive async patterns
function goodRecursion() {
  setImmediate(goodRecursion);
}
```

### 📖 Resources
- [Node.js — The Node.js Event Loop, Timers, and process.nextTick()](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick)
- [nodejs.dev — Understanding the Event Loop](https://nodejs.dev/en/learn/the-nodejs-event-loop/)
- [libuv documentation](https://docs.libuv.org/en/v1.x/design.html)
- [IBM Developer — Understanding the Node.js event loop](https://developer.ibm.com/tutorials/learn-nodejs-the-event-loop/)

---

## 3. Browser Event Loop vs Node Event Loop

Both implement the Event Loop concept, but their implementations, APIs, and priorities differ significantly.

### Side-by-Side Comparison

| Feature | Browser | Node.js |
|---|---|---|
| Spec | HTML Living Standard (WHATWG) | libuv (custom, not spec-based) |
| Microtask sources | `Promise`, `queueMicrotask`, `MutationObserver` | `Promise`, `process.nextTick`, `queueMicrotask` |
| `process.nextTick` | ❌ Not available | ✅ Runs before all other microtasks |
| `setImmediate` | ❌ Not available (IE had it) | ✅ Check phase, after I/O |
| Macrotask sources | `setTimeout`, `setInterval`, UI events, XHR, `fetch` | `setTimeout`, `setInterval`, I/O, `setImmediate` |
| Rendering step | ✅ Yes (between tasks, rAF) | ❌ No (no DOM, no painting) |
| `requestAnimationFrame` | ✅ Before render | ❌ Not available |
| `MutationObserver` | ✅ DOM change microtask | ❌ Not available |
| I/O thread pool | ❌ Delegated to browser C++ | ✅ libuv (default 4 threads) |
| Event loop phases | Tasks → Microtasks → Render | 6 explicit phases + microtask checkpoints |
| Microtask priority | All microtasks equal | `nextTick` queue drains BEFORE Promise queue |

### The Render Step (Browser Only)
The browser event loop has an extra step that Node doesn't:
```
Task → Microtasks → [requestAnimationFrame] → [Render/Paint] → Task → ...
```
This is why heavy JS blocks rendering — the render step never gets a chance to run while the call stack is busy.

### Microtask Priority Difference — Critical!
```js
// In Node.js:
Promise.resolve().then(() => console.log("Promise"));
process.nextTick(() => console.log("nextTick"));
// Output: nextTick, Promise  ← nextTick has higher priority than Promise

// In Browser:
// process.nextTick doesn't exist.
// All microtasks (Promise, queueMicrotask) are equal priority, FIFO order.
Promise.resolve().then(() => console.log("Promise 1"));
queueMicrotask(() => console.log("microtask"));
// Output: Promise 1, microtask  ← FIFO within microtask queue
```

### Concurrency Model Difference
- **Browser**: Multiple tabs/workers each get their own event loop. Web Workers run in separate threads.
- **Node.js**: One event loop per process. Parallelism via `cluster`, `child_process`, or `worker_threads`.

### 📖 Resources
- [Jake Archibald — Tasks, microtasks, queues and schedules (browser-focused)](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
- [Node.js Event Loop vs Browser Event Loop (blog)](https://blog.insiderattack.net/event-loop-and-the-big-picture-nodejs-event-loop-part-1-1cb67a182810)
- [HTML spec — Event Loop processing model](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)

---

## 4. Don't Block the Event Loop ⚡

> **"Node.js uses a small number of threads to handle many clients. In Node.js there are two types of threads: one Event Loop (the main loop), and a pool of k Workers in a Worker Pool."**
> — Node.js official guide

This is the single most important operational concern in Node.js. The event loop runs your JS. If your JS takes too long, **every other client waits**.

### What Blocks the Event Loop?

```js
// ❌ 1. Synchronous CPU-heavy computation
function sumToN(n) {
  let sum = 0;
  for (let i = 0; i <= n; i++) sum += i; // blocks for large n
  return sum;
}

// ❌ 2. Synchronous file I/O
const data = fs.readFileSync("big-file.txt"); // blocks!

// ❌ 3. JSON parsing large payloads
const obj = JSON.parse(bigJsonString); // > 1MB gets slow

// ❌ 4. Regex catastrophic backtracking
/^(a+)+$/.test("aaaaaaaaaaaab"); // can run for seconds

// ❌ 5. Synchronous crypto
crypto.pbkdf2Sync(password, salt, 100000, 64, "sha512"); // blocks!
```

### Time-to-Completion (TTC) Budget
The Node.js docs recommend keeping each callback's execution under **~100ms** so the loop can service other requests.

### Solutions

```js
// ✅ 1. Use async versions always
fs.readFile("big-file.txt", (err, data) => { /* non-blocking */ });
await fs.promises.readFile("big-file.txt");

// ✅ 2. Partition long-running CPU work (setImmediate chunking)
function processLargeArray(arr, chunkSize = 1000) {
  return new Promise(resolve => {
    const results = [];
    let i = 0;
    function processChunk() {
      const end = Math.min(i + chunkSize, arr.length);
      while (i < end) results.push(heavyWork(arr[i++]));
      if (i < arr.length) setImmediate(processChunk); // yield to event loop
      else resolve(results);
    }
    processChunk();
  });
}

// ✅ 3. Offload CPU work to Worker Threads
const { Worker } = require("worker_threads");
const worker = new Worker("./heavy-task.js", { workerData: { n: 1e9 } });
worker.on("message", (result) => console.log(result));

// ✅ 4. Use async crypto
await crypto.promises.pbkdf2(password, salt, 100000, 64, "sha512");
// or the callback form:
crypto.pbkdf2(password, salt, 100000, 64, "sha512", (err, key) => { /* */ });

// ✅ 5. Stream large files instead of loading into memory
const readable = fs.createReadStream("big-file.txt");
readable.pipe(res); // streams chunks, never loads whole file
```

### The Worker Pool (libuv thread pool)
Node offloads these to its internal thread pool (default: 4 threads):
- `fs` file I/O (on most OSes — Linux O_DIRECT, macOS, Windows)
- DNS lookups (`dns.lookup`)
- Synchronous crypto (`crypto.pbkdf2`, `crypto.randomBytes`)
- Zlib compression
- Custom native addons (C++ bindings)

```js
// Increase thread pool for I/O-heavy apps
process.env.UV_THREADPOOL_SIZE = 16; // must be set before any I/O
```

### Monitoring the Event Loop Lag
```js
// DIY lag measurement
let lastCheck = Date.now();
setInterval(() => {
  const now = Date.now();
  const lag = now - lastCheck - 1000;
  if (lag > 50) console.warn(`Event loop lag: ${lag}ms`);
  lastCheck = now;
}, 1000);

// Production: use clinic.js or 0x profiler
// npm install -g clinic
// clinic doctor -- node app.js
```

### 📖 Resources — **READ THESE FIRST**
- [**Node.js Official — Don't Block the Event Loop (Worker Pool)**](https://nodejs.org/en/docs/guides/dont-block-the-event-loop) ← 🔴 must read
- [Node.js — Event Loop Timers](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick)
- [clinic.js — Profiling tool](https://clinicjs.org/)
- [0x — Flame graph profiler](https://github.com/davidmarkclements/0x)

---

## 5. Node.js Core Modules

### `fs` — File System
```js
const fs = require("fs");
const fsp = require("fs").promises; // promise-based (Node 10+)

// Async (non-blocking ✅)
await fsp.readFile("file.txt", "utf8");
await fsp.writeFile("out.txt", data, "utf8");
await fsp.appendFile("log.txt", "entry\n");
await fsp.mkdir("dir", { recursive: true });
const files = await fsp.readdir("./");
await fsp.rename("old.txt", "new.txt");
await fsp.unlink("file.txt"); // delete

// Sync (blocking ❌ — only use at startup)
const config = JSON.parse(fs.readFileSync("config.json", "utf8"));

// File watching
fs.watch("file.txt", (event, filename) => { /* 'change' or 'rename' */ });
```

### `path` — Path Utilities
```js
const path = require("path");

path.join("/users", "ahmed", "docs");    // "/users/ahmed/docs"
path.resolve("../config/db.js");          // absolute path
path.dirname("/users/ahmed/file.txt");    // "/users/ahmed"
path.basename("/users/ahmed/file.txt");   // "file.txt"
path.extname("file.txt");                 // ".txt"
path.parse("/users/ahmed/file.txt");      // {root, dir, base, name, ext}
```

### `os` — Operating System Info
```js
const os = require("os");

os.cpus().length;  // number of CPU cores
os.totalmem();     // total RAM in bytes
os.freemem();
os.platform();     // 'darwin', 'linux', 'win32'
os.hostname();
os.tmpdir();
```

### `events` — EventEmitter
```js
const { EventEmitter } = require("events");

class Database extends EventEmitter {
  connect() {
    // ...
    this.emit("connected", { host: "localhost" });
  }
}

const db = new Database();
db.on("connected", (info) => console.log("DB connected", info));
db.once("connected", () => { /* fires only once */ });
db.off("connected", handler); // remove listener
db.emit("connected", { host: "localhost" }); // manual emit

// Avoid memory leak — default max 10 listeners
db.setMaxListeners(20);
```

### `http` / `https` — HTTP Servers
```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(JSON.stringify({ ok: true }));
});

server.listen(3000, () => console.log("Server running"));
```

### `crypto` — Cryptography
```js
const crypto = require("crypto");

// Hashing
crypto.createHash("sha256").update("data").digest("hex");

// HMAC
crypto.createHmac("sha256", "secret").update("data").digest("hex");

// Random bytes (async ✅)
const token = (await crypto.promises.randomBytes(32)).toString("hex");

// Async pbkdf2 ✅
const key = await crypto.promises.pbkdf2(password, salt, 100000, 64, "sha512");
```

### 📖 Resources
- [Node.js API docs](https://nodejs.org/api/)
- [javascript.info — Node.js](https://javascript.info/nodejs)

---

## 6. Streams & Buffers

### Why Streams?
Without streams, loading a 1GB file means putting 1GB in RAM. With streams, you process it chunk by chunk.

### 4 Types of Streams
```
Readable  — source of data (fs.createReadStream, HTTP req body)
Writable  — destination (fs.createWriteStream, HTTP res)
Duplex    — both (TCP socket)
Transform — duplex that transforms data (zlib, crypto cipher)
```

### Basic Stream Usage
```js
const fs = require("fs");
const zlib = require("zlib");

// Stream pipeline (handles backpressure + cleanup automatically)
const { pipeline } = require("stream/promises");

await pipeline(
  fs.createReadStream("input.txt"),
  zlib.createGzip(),
  fs.createWriteStream("output.txt.gz")
);

// Manual pipe (less safe — no automatic cleanup on error)
fs.createReadStream("input.txt")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("output.txt.gz"));
```

### Custom Transform Stream
```js
const { Transform } = require("stream");

const upperCase = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

process.stdin.pipe(upperCase).pipe(process.stdout);
```

### Backpressure
When a writable stream can't keep up with a readable, data piles up in memory. `pipe()` and `pipeline()` handle this automatically — they pause the readable when the writable buffer fills.

### Buffers
```js
// Buffer — raw binary data (like a typed array but for Node's I/O)
const buf = Buffer.from("hello", "utf8");
buf.toString("hex");    // hex string
buf.toString("base64"); // base64 string
buf.length;             // byte length

Buffer.alloc(16);               // zero-filled, safe
Buffer.allocUnsafe(16);         // faster but uninitialized (may contain old data!)
Buffer.concat([buf1, buf2]);
```

### 📖 Resources
- [Node.js — Stream documentation](https://nodejs.org/api/stream.html)
- [nodejs.dev — Streams](https://nodejs.dev/en/learn/nodejs-streams/)
- [Node.js streams — Everything you need to know](https://www.freecodecamp.org/news/node-js-streams-everything-you-need-to-know-c9141306be93/)

---

## 7. Worker Threads & Child Processes

### The Problem: CPU-Bound Work
Node is single-threaded. A `for` loop from 0 to 1 billion blocks all other requests.

### Three Solutions

#### 1. Worker Threads (same process, shared memory)
```js
// main.js
const { Worker, isMainThread, parentPort, workerData } = require("worker_threads");

if (isMainThread) {
  const worker = new Worker(__filename, { workerData: { n: 1e9 } });
  worker.on("message", result => console.log("Result:", result));
  worker.on("error", err => console.error(err));
  worker.on("exit", code => { if (code !== 0) console.error("Worker exited", code); });
} else {
  // This runs in the worker thread
  let sum = 0;
  for (let i = 0; i <= workerData.n; i++) sum += i;
  parentPort.postMessage(sum);
}
```

#### 2. Child Process (`child_process.fork`)
```js
// Spawns a completely separate Node.js process
const { fork } = require("child_process");

const child = fork("./worker.js");
child.send({ task: "heavy-computation", data: largeDataset });
child.on("message", (result) => console.log("Done:", result));
child.on("exit", (code) => console.log("Child exited:", code));

// worker.js
process.on("message", ({ task, data }) => {
  const result = doHeavyWork(data);
  process.send(result);
});
```

#### 3. Cluster Module (HTTP load balancing)
```js
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isPrimary) {
  for (let i = 0; i < numCPUs; i++) cluster.fork();
  cluster.on("exit", (worker) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // respawn
  });
} else {
  http.createServer((req, res) => res.end("Hello")).listen(3000);
  console.log(`Worker ${process.pid} started`);
}
```

### When to Use What

| Need | Solution |
|---|---|
| CPU-heavy computation, share memory | Worker Threads |
| Separate process isolation, run any script | Child Process (`fork`, `exec`, `spawn`) |
| Scale HTTP across CPU cores | Cluster module |
| Production HTTP scaling | PM2 cluster mode |

### 📖 Resources
- [Node.js — Worker Threads](https://nodejs.org/api/worker_threads.html)
- [Node.js — Child Processes](https://nodejs.org/api/child_process.html)
- [PM2 — Process manager](https://pm2.keymetrics.io/)

---

## 8. Express.js — Fundamentals

Express is a minimal, unopinionated web framework for Node.js.

### Minimal Server
```js
const express = require("express");
const app = express();

app.use(express.json());           // parse JSON bodies
app.use(express.urlencoded({ extended: true })); // parse form data

app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok", uptime: process.uptime() });
});

app.listen(3000, () => console.log("Server on port 3000"));
```

### Request Object (`req`)
```js
req.params      // route params: /users/:id → req.params.id
req.query       // query string: ?page=2&limit=10 → req.query.page
req.body        // parsed body (needs express.json() middleware)
req.headers     // HTTP headers
req.cookies     // (needs cookie-parser)
req.ip          // client IP
req.method      // GET, POST, etc.
req.path        // /api/users
req.originalUrl // /api/users?page=2
req.get("Authorization") // get specific header
```

### Response Object (`res`)
```js
res.status(200).json({ data })        // JSON response
res.status(201).json({ created: id }) // created
res.status(204).send()                // no content
res.status(400).json({ error: "msg" }) // bad request
res.sendFile(path.join(__dirname, "index.html"))
res.redirect(301, "/new-path")
res.set("X-Custom-Header", "value")
res.cookie("token", value, { httpOnly: true, secure: true })
```

### 📖 Resources
- [Express.js official docs](https://expressjs.com/)
- [Express — Getting Started](https://expressjs.com/en/starter/installing.html)

---

## 9. Express — Middleware Deep Dive

Middleware is the **core design pattern** of Express. Every middleware is a function with signature `(req, res, next)`.

### Middleware Stack
```
Request → [middleware 1] → [middleware 2] → [route handler] → [error handler] → Response
```

```js
// Application-level middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // ← MUST call next() or the request hangs!
});

// Route-level middleware
app.get("/admin", requireAuth, (req, res) => res.json({ secret: true }));

// Multiple middleware on one route
app.post("/users", validateBody, checkPermissions, createUser);
```

### Built-in Middleware
```js
app.use(express.json({ limit: "10mb" }));        // parse JSON
app.use(express.urlencoded({ extended: true }));  // parse form data
app.use(express.static("public"));                // serve static files
```

### Popular Third-Party Middleware
```js
const cors = require("cors");
const helmet = require("helmet");
const morgan = require("morgan");
const rateLimit = require("express-rate-limit");
const cookieParser = require("cookie-parser");
const compression = require("compression");

app.use(cors({ origin: "https://yourapp.com", credentials: true }));
app.use(helmet()); // sets security HTTP headers
app.use(morgan("combined")); // HTTP request logger
app.use(cookieParser());
app.use(compression()); // gzip responses

// Rate limiting
const limiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });
app.use("/api/", limiter);
```

### Writing Custom Middleware
```js
// Authentication middleware
function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "No token" });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid token" });
  }
}

// Async middleware — wrap to catch errors
function asyncMiddleware(fn) {
  return (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);
}

// Or use express-async-errors package to auto-wrap
```

### Middleware Order Matters
```js
app.use(express.json());  // ← must be before routes that need req.body
app.use(cors());          // ← must be before routes
// routes...
app.use(errorHandler);    // ← error handler MUST be last, with 4 args
```

### 📖 Resources
- [Express — Using middleware](https://expressjs.com/en/guide/using-middleware.html)
- [Express — Writing middleware](https://expressjs.com/en/guide/writing-middleware.html)

---

## 10. Express — Routing & Controllers

### Router Module Pattern
```js
// routes/users.js
const router = require("express").Router();

router.get("/", getUsers);
router.post("/", validateUser, createUser);
router.get("/:id", getUserById);
router.put("/:id", requireAuth, updateUser);
router.delete("/:id", requireAuth, deleteUser);

module.exports = router;

// app.js
app.use("/api/v1/users", require("./routes/users"));
app.use("/api/v1/posts", require("./routes/posts"));
```

### Controller Pattern
```js
// controllers/userController.js
const User = require("../models/User");

exports.getUsers = async (req, res, next) => {
  try {
    const { page = 1, limit = 20, sort = "-createdAt" } = req.query;
    const users = await User.find()
      .sort(sort)
      .skip((page - 1) * limit)
      .limit(Number(limit))
      .lean(); // plain JS objects, faster than Mongoose docs
    const total = await User.countDocuments();
    res.json({ data: users, total, page: Number(page), pages: Math.ceil(total / limit) });
  } catch (err) {
    next(err); // pass to error handler
  }
};
```

### Route Parameter Middleware
```js
// Runs whenever :userId appears in any route in this router
router.param("userId", async (req, res, next, id) => {
  const user = await User.findById(id);
  if (!user) return res.status(404).json({ error: "User not found" });
  req.foundUser = user;
  next();
});
```

---

## 11. Express — Error Handling

### Error Handler (4-argument middleware)
```js
// Must have exactly 4 params for Express to treat as error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  const status = err.statusCode || err.status || 500;
  res.status(status).json({
    error: {
      message: err.message || "Internal Server Error",
      ...(process.env.NODE_ENV === "development" && { stack: err.stack }),
    }
  });
});
```

### Custom Error Classes
```js
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true; // vs programming errors
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(resource = "Resource") {
    super(`${resource} not found`, 404);
  }
}

class ValidationError extends AppError {
  constructor(message) { super(message, 400); }
}

// Usage in controller:
if (!user) throw new NotFoundError("User");
```

### Async Error Handling
```js
// Option 1: express-async-errors (patch Express globally)
require("express-async-errors");
// After this, thrown errors in async handlers automatically go to error middleware

// Option 2: Manual wrapper
const catchAsync = (fn) => (req, res, next) =>
  fn(req, res, next).catch(next);

router.get("/:id", catchAsync(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) throw new NotFoundError("User");
  res.json(user);
}));
```

### 📖 Resources
- [Express — Error handling guide](https://expressjs.com/en/guide/error-handling.html)

---

## 12. REST API Design Patterns

### HTTP Method Semantics
```
GET     /users           → list users (200)
POST    /users           → create user (201)
GET     /users/:id       → get one user (200)
PUT     /users/:id       → replace user (200)
PATCH   /users/:id       → partial update (200)
DELETE  /users/:id       → delete user (204)

GET     /users/:id/posts → nested resource (user's posts)
POST    /users/:id/posts → create post for user
```

### Standard Response Shape
```json
// Success list
{ "data": [...], "total": 100, "page": 1, "pages": 5 }

// Success single
{ "data": { "id": "...", "name": "Ahmed" } }

// Error
{ "error": { "code": "VALIDATION_ERROR", "message": "Email is invalid", "field": "email" } }
```

### Versioning
```js
app.use("/api/v1", v1Router); // URL versioning — most common
// Alternatives: header versioning (Accept: application/vnd.api.v1+json)
```

### Pagination, Filtering, Sorting
```js
// GET /users?page=2&limit=20&sort=-createdAt&status=active&name=ahmed

const buildQuery = (queryParams) => {
  const { page = 1, limit = 20, sort = "-createdAt", ...filters } = queryParams;
  return {
    filter: filters,
    options: {
      skip: (page - 1) * limit,
      limit: Number(limit),
      sort,
    }
  };
};
```

### 📖 Resources
- [REST API Design — Best Practices](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)

---

## 13. Authentication & Security

### JWT Flow
```js
const jwt = require("jsonwebtoken");

// Sign (on login)
const token = jwt.sign(
  { userId: user._id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: "15m" } // short-lived access token
);

// Verify (in middleware)
const decoded = jwt.verify(token, process.env.JWT_SECRET);
// throws JsonWebTokenError, TokenExpiredError on failure

// Refresh token pattern:
// - Access token: 15min, in memory or Authorization header
// - Refresh token: 7 days, httpOnly cookie, stored in DB for revocation
```

### Password Hashing
```js
const bcrypt = require("bcrypt");
const SALT_ROUNDS = 12; // higher = slower = more secure

// Hash on signup
const hashed = await bcrypt.hash(plainPassword, SALT_ROUNDS);

// Compare on login
const match = await bcrypt.compare(plainPassword, hashed); // true/false
// Never compare strings directly! Timing attacks.
```

### Security Checklist
```js
app.use(helmet());  // X-XSS-Protection, noSniff, hide X-Powered-By, etc.

// CORS — be specific, not wildcard in production
app.use(cors({ origin: process.env.ALLOWED_ORIGINS.split(",") }));

// Rate limiting
const authLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 10 });
app.use("/api/auth", authLimiter);

// Validate and sanitize input
const { body, validationResult } = require("express-validator");
app.post("/users", [
  body("email").isEmail().normalizeEmail(),
  body("password").isLength({ min: 8 }),
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
});

// Environment variables — never hardcode secrets
process.env.JWT_SECRET;
process.env.DB_URI;
// Use dotenv locally, proper secrets manager in production
```

### 📖 Resources
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [jwt.io — JWT debugger and docs](https://jwt.io/)
- [helmet.js docs](https://helmetjs.github.io/)

---

## 14. ORM vs ODM vs Query Builders — The Full Picture

This is a commonly confused area. Here's the clear breakdown.

### The Taxonomy

```
Raw Driver     → talk directly to DB (pg, mongodb, mysql2)
Query Builder  → programmatic SQL/query construction (Knex.js, Kysely)
ORM            → Object-Relational Mapper — maps tables to classes (Sequelize, TypeORM, Prisma)
ODM            → Object-Document Mapper — maps documents to classes (Mongoose)
```

### Comparison Table

| | Raw Driver | Query Builder | ORM | ODM |
|---|---|---|---|---|
| Database type | Any | Relational (usually) | Relational | Document (NoSQL) |
| Abstraction level | Low | Medium | High | High |
| SQL/query control | Full | Full | Partial | N/A |
| Schema definition | In DB | In DB | In code | In code |
| Migrations | Manual | Yes (Knex) | Yes | ❌ (Mongo is schemaless) |
| Relations | Manual | Manual | Automatic (`hasMany` etc) | Manual (`populate`) |
| Validation | Manual | Manual | Partial | Yes (Mongoose validators) |
| Performance | Fastest | Fast | Overhead | Overhead |
| Examples | `mongodb`, `pg` | `Knex.js`, `Kysely` | `Sequelize`, `TypeORM`, `Prisma` | `Mongoose` |

### When to Use What

```
Raw Driver    → performance-critical, you know the DB deeply, or unusual queries
Query Builder → need full SQL control with some DX improvements (great for complex reports)
ORM           → standard CRUD app with a relational DB, team benefits from abstraction
ODM (Mongoose) → MongoDB + need schema validation, middleware hooks, and model structure
```

### ORM Examples (Relational — SQL)

```js
// Sequelize (most popular for Node + SQL)
const user = await User.findOne({ where: { email }, include: ["Posts"] });

// TypeORM (TypeScript-first)
const user = await userRepository.findOne({ where: { email }, relations: ["posts"] });

// Prisma (newest, most modern DX, auto-generates types)
const user = await prisma.user.findUnique({ where: { email }, include: { posts: true } });
```

### ODM Example (Document — MongoDB)

```js
// Mongoose
const user = await User.findOne({ email }).populate("posts");
```

### The N+1 Problem (applies to both ORM and ODM)
```js
// ❌ N+1: 1 query to get users, then N queries to get each user's posts
const users = await User.find();
for (const user of users) {
  user.posts = await Post.find({ userId: user._id }); // N queries!
}

// ✅ Fix with populate (Mongoose) or include (Sequelize)
const users = await User.find().populate("posts"); // 2 queries
```

### 📖 Resources
- [Prisma — ORM vs query builder vs raw SQL](https://www.prisma.io/dataguide/types/relational/comparing-sql-query-builders-and-orms)
- [Sequelize docs](https://sequelize.org/)
- [Prisma docs](https://www.prisma.io/docs)
- [TypeORM docs](https://typeorm.io/)

---

## 15. MongoDB — Core Concepts

### Document Model
MongoDB stores data as **BSON** (Binary JSON) documents in **collections** (analogous to tables).

```js
// A document
{
  _id: ObjectId("64f1a2b3c4d5e6f7a8b9c0d1"), // auto-generated 12-byte unique ID
  name: "Ahmed",
  email: "ahmed@example.com",
  address: {           // embedded document (denormalized)
    city: "Cairo",
    country: "EG"
  },
  tags: ["developer", "football"], // array
  createdAt: ISODate("2024-01-01T00:00:00Z")
}
```

### SQL → MongoDB Mental Model

| SQL | MongoDB |
|---|---|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| Primary Key | `_id` |
| JOIN | `$lookup` (or embedded docs) |
| Index | Index |
| `SELECT` | `find()` |
| `INSERT` | `insertOne()` / `insertMany()` |
| `UPDATE` | `updateOne()` / `updateMany()` |
| `DELETE` | `deleteOne()` / `deleteMany()` |
| `WHERE` | Query filter object |
| `GROUP BY` | `$group` stage (aggregation) |

### Embedding vs Referencing — Critical Design Decision

```js
// Embedding (denormalized) — store related data in same document
// ✅ When: data is always accessed together, 1:1 or 1:few relationship
// ❌ When: data grows unbounded, data is shared across many documents
{
  _id: ObjectId("..."),
  title: "Post title",
  comments: [  // embedded — retrieved with the post always
    { author: "Ahmed", text: "Great post", date: ISODate("...") }
  ]
}

// Referencing (normalized) — store _id and look up separately
// ✅ When: data is large, shared, or accessed independently
// ❌ When: you always need both pieces — requires extra query
{
  _id: ObjectId("..."),
  title: "Post title",
  authorId: ObjectId("user_id_here") // reference to users collection
}
```

### ObjectId
```js
const { ObjectId } = require("mongodb"); // or mongoose.Types.ObjectId

ObjectId("64f1a2b3c4d5e6f7a8b9c0d1"); // from string
new ObjectId(); // generate new
id.toString(); // to string
id.getTimestamp(); // creation time embedded in ObjectId!
```

### 📖 Resources
- [MongoDB University (free courses)](https://university.mongodb.com/)
- [MongoDB manual](https://www.mongodb.com/docs/manual/)
- [mongodb.com — Data modeling intro](https://www.mongodb.com/docs/manual/core/data-modeling-introduction/)
- [MongoDB — Embedded vs Referenced](https://www.mongodb.com/docs/manual/data-modeling/concepts/embedding-vs-references/)

---

## 16. MongoDB — Queries & Aggregation

### CRUD Operations
```js
const db = client.db("mydb");
const users = db.collection("users");

// Create
await users.insertOne({ name: "Ahmed", age: 25 });
await users.insertMany([{ name: "Ali" }, { name: "Sara" }]);

// Read
await users.findOne({ email: "ahmed@example.com" });
await users.find({ age: { $gte: 18 } }).toArray();
await users.find({}).sort({ name: 1 }).skip(20).limit(10).toArray();

// Update
await users.updateOne(
  { _id: id },
  { $set: { age: 26 }, $push: { tags: "new-tag" } }
);
await users.updateMany({ active: false }, { $set: { deleted: true } });
await users.findOneAndUpdate({ _id: id }, { $set: { name: "New" } }, { returnDocument: "after" });

// Delete
await users.deleteOne({ _id: id });
await users.deleteMany({ createdAt: { $lt: cutoffDate } });
```

### Query Operators
```js
// Comparison
{ age: { $eq: 25, $ne: 30, $gt: 18, $gte: 18, $lt: 65, $lte: 64 } }
{ status: { $in: ["active", "pending"] } }
{ status: { $nin: ["deleted", "banned"] } }

// Logical
{ $and: [{ age: { $gte: 18 } }, { active: true }] }
{ $or: [{ city: "Cairo" }, { city: "Alex" }] }
{ $not: { age: { $gte: 18 } } }
{ $nor: [{ status: "deleted" }, { banned: true }] }

// Array
{ tags: "javascript" }                           // array contains this value
{ tags: { $all: ["node", "express"] } }          // array contains ALL
{ tags: { $size: 3 } }                           // array has exactly 3 elements
{ "comments.author": "Ahmed" }                   // query nested array field

// Element
{ field: { $exists: true } }
{ field: { $type: "string" } }

// Regex
{ name: { $regex: /^ahmed/i } }
```

### Update Operators
```js
$set: { field: value }            // set field value
$unset: { field: "" }             // remove field
$inc: { age: 1, score: -5 }      // increment/decrement
$push: { tags: "new" }            // add to array
$pull: { tags: "old" }            // remove from array
$addToSet: { tags: "unique" }     // add only if not present (set semantics)
$pop: { arr: 1 }                  // remove last (1) or first (-1)
$rename: { oldName: "newName" }   // rename field
```

### Aggregation Pipeline
```js
// Most powerful feature in MongoDB — chain stages to transform data
const result = await db.collection("orders").aggregate([
  // Stage 1: filter
  { $match: { status: "completed", createdAt: { $gte: new Date("2024-01-01") } } },

  // Stage 2: join (like SQL JOIN)
  { $lookup: {
    from: "users",
    localField: "userId",
    foreignField: "_id",
    as: "user"
  }},
  { $unwind: "$user" }, // flatten the user array

  // Stage 3: group
  { $group: {
    _id: "$user.city",
    totalRevenue: { $sum: "$amount" },
    orderCount: { $count: {} },
    avgOrder: { $avg: "$amount" }
  }},

  // Stage 4: sort
  { $sort: { totalRevenue: -1 } },

  // Stage 5: shape output
  { $project: {
    city: "$_id",
    totalRevenue: 1,
    orderCount: 1,
    avgOrder: { $round: ["$avgOrder", 2] },
    _id: 0
  }},

  // Stage 6: limit
  { $limit: 10 }
]).toArray();
```

### Common Aggregation Stages
```
$match      → filter (put early to use indexes!)
$project    → reshape documents (include/exclude/compute fields)
$group      → group and aggregate
$sort       → sort results
$limit      → limit number of results
$skip       → skip results (for pagination)
$lookup     → join with another collection
$unwind     → flatten array field into separate documents
$addFields  → add computed fields without removing others
$count      → count documents into a field
$facet      → run multiple pipelines in parallel
$bucket     → categorize into ranges
$out        → write results to a collection
```

### 📖 Resources
- [MongoDB — Aggregation pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/)
- [MongoDB — Query operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [Studio 3T aggregation tutorial](https://studio3t.com/knowledge-base/articles/mongodb-aggregation-framework/)

---

## 17. MongoDB — Indexing & Performance

### Why Indexes?
Without an index, MongoDB does a **collection scan** — reads every document. With an index, it jumps directly to matching documents.

### Index Types
```js
// Single field
db.collection.createIndex({ email: 1 });        // 1 = ascending, -1 = descending

// Compound (order matters for queries!)
db.collection.createIndex({ city: 1, age: -1 });

// Unique index
db.collection.createIndex({ email: 1 }, { unique: true });

// Sparse index (only index docs where field exists)
db.collection.createIndex({ phone: 1 }, { sparse: true });

// TTL index (auto-delete documents after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Text index (full-text search)
db.posts.createIndex({ title: "text", body: "text" });

// Wildcard index (for dynamic schemas)
db.collection.createIndex({ "attributes.$**": 1 });
```

### The ESR Rule (Compound Indexes)
For compound indexes, field order: **E**quality → **S**ort → **R**ange

```js
// Query: find users in Cairo aged 18-30, sorted by name
db.users.createIndex({ city: 1, name: 1, age: 1 });
// city (equality) → name (sort) → age (range)
```

### `explain()` — Analyze Queries
```js
const plan = await db.collection("users")
  .find({ email: "ahmed@example.com" })
  .explain("executionStats");

// Look for:
plan.executionStats.executionStages.stage;
// "IXSCAN" = index scan ✅
// "COLLSCAN" = collection scan ❌ (means missing index)
plan.executionStats.totalDocsExamined; // should be close to nReturned
plan.executionStats.totalKeysExamined;
```

### Performance Tips
```js
// ✅ Project only needed fields
db.users.find({ active: true }, { name: 1, email: 1 });

// ✅ Use lean() in Mongoose (returns plain objects)
await User.find().lean();

// ✅ Covered queries — index contains all queried + projected fields
// No need to fetch actual documents!

// ❌ Avoid large $in arrays (> few hundred items gets slow)

// ❌ Avoid $where or JavaScript expressions in queries — can't use index

// ✅ Use $match early in aggregation pipelines to filter before processing
```

### 📖 Resources
- [MongoDB — Indexes](https://www.mongodb.com/docs/manual/indexes/)
- [MongoDB — Explain results](https://www.mongodb.com/docs/manual/reference/explain-results/)
- [MongoDB — Index strategies](https://www.mongodb.com/docs/manual/applications/indexes/)

---

## 18. Mongoose — ODM Deep Dive

### Why Mongoose?
The native MongoDB driver gives you power but no structure. Mongoose adds:
- Schema validation
- Pre/post middleware hooks
- Virtual fields
- `populate()` for references
- Instance & static methods
- Type casting

### Connection
```js
const mongoose = require("mongoose");

mongoose.connect(process.env.MONGODB_URI, {
  // These defaults are fine in Mongoose 6+
});

mongoose.connection.on("connected", () => console.log("MongoDB connected"));
mongoose.connection.on("error", (err) => console.error("MongoDB error:", err));
mongoose.connection.on("disconnected", () => console.log("MongoDB disconnected"));

// Graceful shutdown
process.on("SIGINT", async () => {
  await mongoose.connection.close();
  process.exit(0);
});
```

### Schema Definition
```js
const mongoose = require("mongoose");
const { Schema } = mongoose;

const userSchema = new Schema({
  name: {
    type: String,
    required: [true, "Name is required"],
    trim: true,
    minlength: 2,
    maxlength: 100,
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, "Invalid email format"],
  },
  age: {
    type: Number,
    min: [0, "Age must be positive"],
    max: 150,
  },
  role: {
    type: String,
    enum: ["user", "admin", "moderator"],
    default: "user",
  },
  tags: [String],                          // array of strings
  address: {                               // embedded subdocument (inline)
    city: String,
    country: String,
  },
  profilePic: Buffer,
  active: { type: Boolean, default: true },
  lastLogin: Date,
  metadata: Schema.Types.Mixed,            // any type — use sparingly
}, {
  timestamps: true,    // auto-adds createdAt and updatedAt
  toJSON: { virtuals: true },  // include virtuals when converting to JSON
  toObject: { virtuals: true },
});
```

### Middleware (Hooks)
```js
// Pre-save: hash password before saving
userSchema.pre("save", async function(next) {
  if (!this.isModified("password")) return next(); // only hash if changed
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Post-save
userSchema.post("save", function(doc, next) {
  console.log(`User ${doc._id} saved`);
  next();
});

// Pre-find: auto-filter deleted users
userSchema.pre(/^find/, function(next) { // matches find, findOne, findById...
  this.where({ active: { $ne: false } });
  next();
});

// Pre-remove
userSchema.pre("deleteOne", { document: true }, async function(next) {
  await Post.deleteMany({ author: this._id }); // cascade delete
  next();
});
```

### Instance Methods & Statics
```js
// Instance method — called on a document
userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Static method — called on the Model
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Usage
const user = await User.findByEmail("ahmed@example.com");
const match = await user.comparePassword("plaintext");
```

### Virtuals
```js
// Computed property — not stored in DB
userSchema.virtual("fullName").get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Virtual populate — reference without storing IDs
userSchema.virtual("posts", {
  ref: "Post",
  localField: "_id",
  foreignField: "author",
});

// Populate it
const user = await User.findById(id).populate("posts");
```

### Querying with Mongoose
```js
// All return Mongoose Query objects (chainable)
User.find({ active: true })
  .where("age").gte(18)
  .select("name email age")   // projection
  .sort("-createdAt")          // - prefix = descending
  .skip(20)
  .limit(10)
  .populate("posts", "title createdAt") // populate with field selection
  .lean()                      // return plain JS objects (much faster!)
  .exec();                     // execute

// Short forms
await User.findById(id);
await User.findOne({ email });
await User.findByIdAndUpdate(id, { $set: { name } }, { new: true, runValidators: true });
await User.findByIdAndDelete(id);
await User.countDocuments({ active: true });
await User.exists({ email });  // returns { _id: ... } or null (cheap check)
```

### Transactions
```js
// Use sessions for multi-document atomicity
const session = await mongoose.startSession();
session.startTransaction();

try {
  await User.create([{ name: "Ahmed" }], { session });
  await Order.create([{ userId: user._id }], { session });
  await session.commitTransaction();
} catch (err) {
  await session.abortTransaction();
  throw err;
} finally {
  session.endSession();
}
```

### 📖 Resources
- [Mongoose official docs](https://mongoosejs.com/docs/)
- [Mongoose — Schemas](https://mongoosejs.com/docs/guide.html)
- [Mongoose — Middleware](https://mongoosejs.com/docs/middleware.html)
- [Mongoose — Populate](https://mongoosejs.com/docs/populate.html)
- [Mongoose — Queries](https://mongoosejs.com/docs/queries.html)

---

## 19. Mongoose — Schema Design Patterns

### One-to-One (Embed)
```js
// ✅ Embed when always accessed together
const userSchema = new Schema({
  name: String,
  profile: {    // embedded
    bio: String,
    avatar: String,
    website: String,
  }
});
```

### One-to-Few (Embed Array)
```js
// ✅ Embed when array is small and bounded
const postSchema = new Schema({
  title: String,
  comments: [{     // max ~100 comments — fine to embed
    author: { type: Schema.Types.ObjectId, ref: "User" },
    text: String,
    createdAt: { type: Date, default: Date.now }
  }]
});
```

### One-to-Many (Reference)
```js
// ✅ Reference when array is large or unbounded
const postSchema = new Schema({
  title: String,
  author: { type: Schema.Types.ObjectId, ref: "User", required: true },
});

// Query with populate
const posts = await Post.find().populate("author", "name email");
```

### Many-to-Many (Reference Array or Junction)
```js
// Approach 1: Array of refs on one side
const courseSchema = new Schema({
  students: [{ type: Schema.Types.ObjectId, ref: "User" }]
});

// Approach 2: Separate enrollment collection (when junction has data)
const enrollmentSchema = new Schema({
  student: { type: Schema.Types.ObjectId, ref: "User" },
  course: { type: Schema.Types.ObjectId, ref: "Course" },
  enrolledAt: Date,
  grade: Number,
});
```

### Index Declaration in Schema
```js
const userSchema = new Schema({
  email: { type: String, unique: true }, // index: true implied by unique
  city: String,
  age: Number,
});

// Compound index
userSchema.index({ city: 1, age: -1 });

// Text index
userSchema.index({ name: "text", bio: "text" });
```

### 📖 Resources
- [MongoDB — Schema Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)
- [MongoDB — 6 Rules of Thumb for MongoDB Schema Design](https://www.mongodb.com/blog/post/6-rules-of-thumb-for-mongodb-schema-design)

---

## 20. Common Pitfalls 🪤

### Node.js Pitfalls

```js
// ❌ Not handling promise rejections — crashes Node 15+
somePromise(); // unhandled!

// ✅ Always handle or chain .catch()
somePromise().catch(err => { logger.error(err); });
process.on("unhandledRejection", (err) => { logger.error(err); process.exit(1); });

// ❌ Using sync fs in request handlers
app.get("/data", (req, res) => {
  const data = fs.readFileSync("./data.json"); // blocks all requests!
});

// ✅ Use async
app.get("/data", async (req, res) => {
  const data = await fs.promises.readFile("./data.json", "utf8");
  res.json(JSON.parse(data));
});

// ❌ Leaking memory with EventEmitters
setInterval(() => {
  emitter.on("data", () => { /* never removed! */ }); // listener leak
}, 1000);

// ✅ Remove listeners when done
emitter.once("data", handler); // auto-removes after firing
emitter.removeListener("data", handler);
```

### Express Pitfalls

```js
// ❌ Forgetting next() in middleware — request hangs
app.use((req, res, next) => {
  // doing something...
  // forgot next()! ← silent bug
});

// ❌ Not using 4-arg error handler
app.use((err, req, res) => { /* Express ignores this! needs 4 args */ });
app.use((err, req, res, next) => { /* correct */ });

// ❌ Sending response after headers sent — crashes
app.get("/", async (req, res) => {
  res.json({ ok: true });
  res.json({ oops: true }); // ← Error: Cannot set headers after they are sent
});

// ❌ Missing await — returns pending promise
app.get("/users", async (req, res, next) => {
  const users = User.find(); // ← missing await!
  res.json(users); // sends Mongoose Query object, not data
});
```

### MongoDB / Mongoose Pitfalls

```js
// ❌ Not using .lean() when you don't need Mongoose features
// Mongoose documents are heavy objects with change tracking, methods, etc.
const users = await User.find(); // heavy
const users = await User.find().lean(); // ✅ plain objects, 2-3x faster

// ❌ Missing { new: true } in findByIdAndUpdate
const updated = await User.findByIdAndUpdate(id, { $set: { name } });
// returns the OLD document by default!
const updated = await User.findByIdAndUpdate(id, { $set: { name } }, { new: true }); // ✅

// ❌ Forgetting runValidators on updates
await User.findByIdAndUpdate(id, { $set: { age: -5 } });
// Validators don't run on update by default!
await User.findByIdAndUpdate(id, { $set: { age: -5 } }, { runValidators: true }); // ✅

// ❌ Opening a new mongoose connection per request
app.get("/users", async (req, res) => {
  await mongoose.connect(uri); // ← Never do this!
});
// ✅ Connect once at app startup

// ❌ Not closing sessions on error in transactions
const session = await mongoose.startSession();
session.startTransaction();
await User.create([{}], { session });
// if error here, session never closed → connection leak!
// ✅ Always use try/catch/finally with sessions

// ❌ Querying with string ID when ObjectId is expected
await User.findOne({ _id: req.params.id }); // Mongoose auto-casts, but be aware
await User.findOne({ _id: new mongoose.Types.ObjectId(req.params.id) }); // explicit
```

---

## 21. Quick-Fire Q&A Cheatsheet

| Question | Answer |
|---|---|
| What is the Node event loop? | A loop that processes phases: timers → pending callbacks → idle → poll → check → close. It offloads I/O to the OS/thread pool |
| `process.nextTick` vs `setImmediate`? | `nextTick` fires before any I/O (after current op, before event loop phases); `setImmediate` fires in the check phase (after I/O) |
| What is the libuv thread pool? | 4 threads (default) that handle file I/O, DNS, crypto — frees the event loop from blocking |
| What blocks the event loop? | CPU-heavy JS, `*Sync` fs methods, large JSON parsing, catastrophic regex, synchronous crypto |
| ORM vs ODM? | ORM maps SQL tables to objects (Sequelize, Prisma). ODM maps NoSQL documents to objects (Mongoose) |
| Embedding vs referencing in MongoDB? | Embed for 1:1 or 1:few, always-together data. Reference for large/growing data or shared data |
| What is `$lookup`? | MongoDB aggregation join — like SQL JOIN but as a pipeline stage |
| Mongoose `pre('save')` vs `pre('findOne')`? | `pre('save')` triggers on `.save()`. `pre('findOne')` triggers on all find queries. Use `this` (document vs query) differently |
| What does `.lean()` do? | Returns plain JS objects instead of Mongoose documents — much faster when you don't need methods/virtuals |
| What is `populate()`? | Mongoose replaces a referenced `_id` with the actual document from the referenced collection |
| What is `{ new: true }` in update? | Makes `findByIdAndUpdate` return the updated document instead of the original |
| `Promise.all` in Node? | Runs promises concurrently. Fails fast on first rejection. Use `allSettled` to wait for all |
| What is backpressure? | When a writable stream can't consume data as fast as a readable produces it — `pipe()` handles automatically |
| MongoDB index types? | Single, compound, unique, sparse, TTL (auto-expire), text (full-text search), wildcard |
| What is the N+1 problem? | Fetching N related items with 1 query per item. Fix: use `populate()`, `$lookup`, or `include` |
| `updateOne` vs `findByIdAndUpdate`? | `updateOne` is faster (no return). `findByIdAndUpdate` returns a document (old or new) |
| What is a Mongoose virtual? | A computed field not stored in DB. Can also be a virtual populate (reference without storing) |
| Cluster vs Worker Threads? | Cluster = multiple processes, each with own memory, share server port. Workers = threads in same process, can share memory via SharedArrayBuffer |
| What is `express.json()`? | Built-in middleware that parses `Content-Type: application/json` request bodies into `req.body` |
| What is helmet.js? | Express middleware that sets security HTTP headers (CSP, XSS protection, HSTS, etc.) |

---

## 🎯 Senior SWE Interview Tips — Backend Edition

1. **Always mention async alternatives** — if you talk about reading a file, instantly mention `fs.promises.readFile`.
2. **Event loop questions are depth tests** — know the 6 phases, know the microtask priority difference between browser and Node.
3. **MongoDB schema design is opinion-based** — state the tradeoff (embed = fast reads, reference = flexibility). There's no single right answer.
4. **Don't forget indexes** — mentioning indexes unprompted shows production experience.
5. **Security is a bonus** — mention helmet, rate-limiting, input validation, JWT expiry without being asked.
6. **Mention `.lean()`** — it signals you've worked with Mongoose at scale.
7. **Error handling pattern** — always show the 4-arg Express error handler and `catchAsync` wrapper.
8. **On "don't block the event loop"** — name specific culprits: `readFileSync`, `pbkdf2Sync`, heavy regex, and give the fix for each.

---

*Go get that shadow interview, Ahmed. The backend knows your name. 🚀*

> **Legend:** 📖 = Read this | ⚠️ = Watch out | ✅ = Do this | ❌ = Avoid this
