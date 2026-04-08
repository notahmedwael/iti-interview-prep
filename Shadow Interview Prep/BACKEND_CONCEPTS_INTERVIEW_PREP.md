# ⚙️ Backend Engineering — Complete Interview Guide

> **Part 5 of Ahmed's shadow interview prep.**
> Covers every backend concept that matters in interviews and real systems.
> Every concept has both **Node.js + Express** and **Python + Django** code examples.
> Databases are covered where they intersect with backend concerns — a full DB guide comes later.

---

## 📑 Table of Contents

1.  [What is Backend Engineering?](#1-what-is-backend-engineering)
2.  [API Paradigms: REST, GraphQL, gRPC, SOAP](#2-api-paradigms-rest-graphql-grpc-soap)
3.  [REST API Design — Deep Dive](#3-rest-api-design--deep-dive)
4.  [GraphQL — Deep Dive](#4-graphql--deep-dive)
5.  [gRPC — Deep Dive](#5-grpc--deep-dive)
6.  [SOAP — What It Is and Why It's Still Around](#6-soap--what-it-is-and-why-its-still-around)
7.  [Authentication vs Authorization](#7-authentication-vs-authorization)
8.  [JWT, Sessions & OAuth2](#8-jwt-sessions--oauth2)
9.  [Idempotency](#9-idempotency)
10. [Caching Strategies](#10-caching-strategies)
11. [Rate Limiting & Throttling](#11-rate-limiting--throttling)
12. [Concurrency & Race Conditions](#12-concurrency--race-conditions)
13. [Database Transactions & ACID](#13-database-transactions--acid)
14. [Optimistic vs Pessimistic Locking](#14-optimistic-vs-pessimistic-locking)
15. [N+1 Problem & Query Optimization](#15-n1-problem--query-optimization)
16. [Indexes & When to Use Them](#16-indexes--when-to-use-them)
17. [Database Migrations](#17-database-migrations)
18. [Pagination Patterns](#18-pagination-patterns)
19. [File Uploads & Storage](#19-file-uploads--storage)
20. [Background Jobs & Message Queues](#20-background-jobs--message-queues)
21. [WebSockets & Real-Time](#21-websockets--real-time)
22. [Security: OWASP Top 10 for Backend Devs](#22-security-owasp-top-10-for-backend-devs)
23. [Logging, Monitoring & Observability](#23-logging-monitoring--observability)
24. [Environment Config & Secrets Management](#24-environment-config--secrets-management)
25. [API Versioning](#25-api-versioning)
26. [CORS — Cross-Origin Resource Sharing](#26-cors--cross-origin-resource-sharing)
27. [Horizontal vs Vertical Scaling](#27-horizontal-vs-vertical-scaling)
28. [Common Backend Interview Questions & Answers](#28-common-backend-interview-questions--answers)

---

## 1. What is Backend Engineering?

Backend engineering is everything that happens on the **server side** — the code users never see but always depend on. It encompasses:

```
Client (browser / mobile / another service)
         │
         │ HTTP / WebSocket / gRPC
         ▼
    API Layer (Express / Django / FastAPI)
         │
    ┌────┼────────────────────┐
    │    │                    │
  Auth  Business Logic    Validation
    │    │                    │
    └────┼────────────────────┘
         │
   Data Layer
   ├── Relational DB (PostgreSQL, MySQL)
   ├── Document DB (MongoDB)
   ├── Cache (Redis)
   ├── Object Storage (S3)
   └── Message Queue (RabbitMQ, Kafka)
```

A backend developer's job is to design this system so it is **correct, fast, secure, and resilient**.

---

## 2. API Paradigms: REST, GraphQL, gRPC, SOAP

Before diving into each: an API is a contract between a client and a server. The paradigm defines the rules of that contract.

### Quick Comparison

| | REST | GraphQL | gRPC | SOAP |
|---|---|---|---|---|
| Protocol | HTTP | HTTP | HTTP/2 | HTTP/SMTP |
| Data format | JSON/XML | JSON | Protobuf (binary) | XML |
| Schema | OpenAPI (optional) | SDL (required) | .proto (required) | WSDL (required) |
| Type safety | Weak | Strong | Strong | Strong |
| Versioning | URL (`/v1/`) | Schema evolution | Proto versioning | WSDL versioning |
| Over/under-fetching | Common | Solved | Solved | Common |
| Streaming | Limited | Subscriptions | Native (bidirectional) | No |
| Browser support | Native | Native | Needs grpc-web | Limited |
| Best for | Public APIs, simplicity | Flexible clients, many consumers | Internal microservices, performance | Legacy enterprise |
| Learning curve | Low | Medium | High | High |

---

## 3. REST API Design — Deep Dive

REST (Representational State Transfer) is a set of architectural constraints for designing networked APIs. It's not a spec — it's a style.

### The 6 REST Constraints
1. **Client-Server** — UI and data storage are separated
2. **Stateless** — every request contains all info needed; no session on server
3. **Cacheable** — responses must declare whether they can be cached
4. **Uniform Interface** — consistent URL structure and HTTP verbs
5. **Layered System** — client doesn't know if it's talking to the server directly or a proxy
6. **Code on Demand** (optional) — server can send executable code to client

### HTTP Methods & Their Semantics

```
GET    /users          → list all users (safe, idempotent)
POST   /users          → create a user (neither safe nor idempotent)
GET    /users/:id      → get one user (safe, idempotent)
PUT    /users/:id      → replace entire user (idempotent)
PATCH  /users/:id      → partial update (not necessarily idempotent)
DELETE /users/:id      → delete user (idempotent)
```

**Safe** = no side effects (GET, HEAD, OPTIONS)
**Idempotent** = same result no matter how many times called (GET, PUT, DELETE)

### Standard HTTP Status Codes

```
2xx Success:
  200 OK              → successful GET, PUT, PATCH
  201 Created         → successful POST (include Location header)
  204 No Content      → successful DELETE or PUT with no body

4xx Client Errors:
  400 Bad Request     → validation error, malformed request
  401 Unauthorized    → not authenticated (missing/invalid token)
  403 Forbidden       → authenticated but no permission
  404 Not Found       → resource doesn't exist
  409 Conflict        → duplicate, version mismatch
  422 Unprocessable   → semantically invalid (e.g., end date before start date)
  429 Too Many Requests → rate limit hit

5xx Server Errors:
  500 Internal Server Error → unexpected crash
  502 Bad Gateway           → upstream service failed
  503 Service Unavailable   → server overloaded / maintenance
```

### REST API in Node + Express

```javascript
// routes/users.js
const express = require("express");
const router  = express.Router();
const { body, param, query, validationResult } = require("express-validator");
const UserService = require("../services/UserService");

// Standard response envelope
const respond = (res, statusCode, data, meta = {}) =>
  res.status(statusCode).json({ data, ...meta });

const respondError = (res, statusCode, code, message, details = null) =>
  res.status(statusCode).json({ error: { code, message, details } });

// GET /users?page=1&limit=20&sort=-createdAt&status=active
router.get("/",
  query("page").optional().isInt({ min: 1 }).toInt(),
  query("limit").optional().isInt({ min: 1, max: 100 }).toInt(),
  async (req, res, next) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return respondError(res, 400, "VALIDATION_ERROR", "Invalid query params", errors.array());
      }
      const { page = 1, limit = 20, sort = "-createdAt", ...filters } = req.query;
      const { users, total } = await UserService.list({ page, limit, sort, filters });
      respond(res, 200, users, {
        meta: { page, limit, total, pages: Math.ceil(total / limit) },
      });
    } catch (err) { next(err); }
  }
);

// POST /users
router.post("/",
  body("name").trim().notEmpty().isLength({ min: 2, max: 100 }),
  body("email").isEmail().normalizeEmail(),
  body("password").isLength({ min: 8 }).matches(/[A-Z]/).matches(/[0-9]/),
  async (req, res, next) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return respondError(res, 400, "VALIDATION_ERROR", "Invalid input", errors.array());
      }
      const user = await UserService.create(req.body);
      res.setHeader("Location", `/api/v1/users/${user.id}`);
      respond(res, 201, user);
    } catch (err) {
      if (err.code === "DUPLICATE_EMAIL") return respondError(res, 409, "CONFLICT", err.message);
      next(err);
    }
  }
);

// PATCH /users/:id
router.patch("/:id",
  param("id").isUUID(),
  body("name").optional().trim().isLength({ min: 2 }),
  body("email").optional().isEmail().normalizeEmail(),
  async (req, res, next) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) return respondError(res, 400, "VALIDATION_ERROR", "Invalid input", errors.array());
      const user = await UserService.update(req.params.id, req.body);
      if (!user) return respondError(res, 404, "NOT_FOUND", "User not found");
      respond(res, 200, user);
    } catch (err) { next(err); }
  }
);

// DELETE /users/:id
router.delete("/:id", param("id").isUUID(), async (req, res, next) => {
  try {
    const deleted = await UserService.delete(req.params.id);
    if (!deleted) return respondError(res, 404, "NOT_FOUND", "User not found");
    res.status(204).send();
  } catch (err) { next(err); }
});

module.exports = router;
```

### REST API in Python Django (DRF)

```python
# views.py
from rest_framework import viewsets, status, filters
from rest_framework.response import Response
from rest_framework.decorators import action
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend
from .models import User
from .serializers import UserSerializer, UserCreateSerializer, UserUpdateSerializer
from .pagination import StandardPagination

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all().order_by("-created_at")
    permission_classes = [IsAuthenticated]
    pagination_class = StandardPagination
    filter_backends = [DjangoFilterBackend, filters.OrderingFilter]
    filterset_fields = ["status", "role"]
    ordering_fields = ["name", "created_at"]

    def get_serializer_class(self):
        if self.action == "create":
            return UserCreateSerializer
        if self.action in ["update", "partial_update"]:
            return UserUpdateSerializer
        return UserSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)  # auto-returns 400 with errors
        user = serializer.save()
        headers = self.get_success_headers(serializer.data)
        return Response(
            {"data": UserSerializer(user).data},
            status=status.HTTP_201_CREATED,
            headers=headers,
        )

    def destroy(self, request, *args, **kwargs):
        instance = self.get_object()
        instance.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)

    @action(detail=True, methods=["post"], url_path="deactivate")
    def deactivate(self, request, pk=None):
        user = self.get_object()
        user.status = "inactive"
        user.save(update_fields=["status"])
        return Response({"data": UserSerializer(user).data})

# urls.py
from rest_framework.routers import DefaultRouter
router = DefaultRouter()
router.register(r"users", UserViewSet)

# pagination.py
from rest_framework.pagination import PageNumberPagination

class StandardPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = "limit"
    max_page_size = 100

    def get_paginated_response(self, data):
        return Response({
            "data": data,
            "meta": {
                "page": self.page.number,
                "total": self.page.paginator.count,
                "pages": self.page.paginator.num_pages,
                "limit": self.get_page_size(self.request),
            }
        })
```

### 📖 Resources
- [REST API Design Best Practices — Stack Overflow Blog](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)
- [HTTP Status Codes Reference](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---

## 4. GraphQL — Deep Dive

GraphQL is a query language for APIs. The client specifies **exactly what data it needs** — no more over-fetching (getting too much) or under-fetching (needing multiple requests).

### Core Concepts

```
SDL (Schema Definition Language) — defines the shape of your API
Query     — read data (like GET)
Mutation  — write data (like POST/PUT/DELETE)
Subscription — real-time updates over WebSocket
Resolver  — the function that fetches data for a field
```

### The Over/Under-Fetching Problem REST Has

```
REST Problem: You need a user's name + their 3 most recent post titles.

Option 1 — over-fetch:
  GET /users/1  → returns { id, name, email, address, phone, dob, ... } (too much!)
  GET /users/1/posts → returns all 47 posts with full content (way too much!)
  = 2 requests, too much data

Option 2 — under-fetch:
  GET /users/1/summary → only has name (not posts!)
  GET /posts?userId=1&limit=3 → need separate request
  = 2 requests, multiple endpoints to maintain

GraphQL Solution: 1 request, exactly what you asked for:
  query {
    user(id: "1") {
      name
      recentPosts: posts(limit: 3) {
        title
      }
    }
  }
```

### GraphQL in Node.js (Apollo Server)

```javascript
// schema.js
const { gql } = require("apollo-server-express");

const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    role: UserRole!
    posts(limit: Int = 10, offset: Int = 0): [Post!]!
    createdAt: String!
  }

  type Post {
    id: ID!
    title: String!
    body: String!
    author: User!
    tags: [String!]!
    createdAt: String!
  }

  enum UserRole {
    ADMIN
    USER
    MODERATOR
  }

  type Query {
    users(limit: Int, offset: Int): [User!]!
    user(id: ID!): User
    posts(userId: ID, limit: Int, offset: Int): [Post!]!
  }

  type Mutation {
    createUser(input: CreateUserInput!): User!
    updateUser(id: ID!, input: UpdateUserInput!): User!
    deleteUser(id: ID!): Boolean!
    createPost(input: CreatePostInput!): Post!
  }

  input CreateUserInput {
    name: String!
    email: String!
    password: String!
  }

  input UpdateUserInput {
    name: String
    email: String
  }

  input CreatePostInput {
    title: String!
    body: String!
    tags: [String!]
  }
`;

// resolvers.js
const resolvers = {
  Query: {
    users: async (_, { limit = 20, offset = 0 }, { dataSources }) =>
      dataSources.userAPI.getUsers({ limit, offset }),

    user: async (_, { id }, { dataSources }) =>
      dataSources.userAPI.getUserById(id),

    posts: async (_, { userId, limit, offset }, { dataSources }) =>
      dataSources.postAPI.getPosts({ userId, limit, offset }),
  },

  Mutation: {
    createUser: async (_, { input }, { dataSources, user }) => {
      if (!user) throw new AuthenticationError("Must be logged in");
      return dataSources.userAPI.createUser(input);
    },

    deleteUser: async (_, { id }, { dataSources, user }) => {
      if (user?.role !== "ADMIN") throw new ForbiddenError("Admins only");
      return dataSources.userAPI.deleteUser(id);
    },
  },

  // Field resolvers — called per User instance
  User: {
    posts: async (parent, { limit, offset }, { dataSources }) =>
      // parent.id is the user's id — the DataLoader batches these to prevent N+1
      dataSources.postAPI.getPostsByUserId(parent.id, { limit, offset }),
  },

  Post: {
    author: async (parent, _, { dataSources }) =>
      dataSources.userAPI.getUserById(parent.authorId),
  },
};

// server.js
const { ApolloServer } = require("apollo-server-express");
const DataLoader = require("dataloader");

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => ({
    user: req.user, // from auth middleware
    dataSources: {
      userAPI: new UserDataSource(),
      postAPI: new PostDataSource(),
    },
    // DataLoader per request — batches & caches DB calls
    userLoader: new DataLoader(async (ids) => {
      const users = await User.findMany({ id: { in: ids } });
      return ids.map(id => users.find(u => u.id === id));
    }),
  }),
  formatError: (err) => ({
    message: err.message,
    code: err.extensions?.code,
    path: err.path,
  }),
});
```

### GraphQL in Django (Strawberry)

```python
# schema.py
import strawberry
from strawberry.types import Info
from typing import List, Optional
from .models import User, Post
from .types import UserType, PostType

@strawberry.type
class Query:
    @strawberry.field
    def user(self, id: strawberry.ID, info: Info) -> Optional[UserType]:
        try:
            return User.objects.get(pk=id)
        except User.DoesNotExist:
            return None

    @strawberry.field
    def users(self, limit: int = 20, offset: int = 0) -> List[UserType]:
        return User.objects.all()[offset:offset + limit]

@strawberry.type
class Mutation:
    @strawberry.mutation
    def create_user(self, name: str, email: str, password: str, info: Info) -> UserType:
        if not info.context.request.user.is_authenticated:
            raise Exception("Authentication required")
        user = User.objects.create_user(username=email, email=email,
                                         first_name=name, password=password)
        return user

    @strawberry.mutation
    def delete_user(self, id: strawberry.ID, info: Info) -> bool:
        if not info.context.request.user.is_staff:
            raise Exception("Admin only")
        User.objects.filter(pk=id).delete()
        return True

schema = strawberry.Schema(query=Query, mutation=Mutation)

# urls.py
from strawberry.django.views import GraphQLView
urlpatterns = [
    path("graphql/", GraphQLView.as_view(schema=schema)),
]
```

### The N+1 Problem in GraphQL (and DataLoader)

```javascript
// The N+1 problem in GraphQL resolvers
// If you query 20 posts and each has an author field:
Post: {
  // This fires ONCE PER POST = 20 separate DB queries for 20 posts!
  author: (post) => User.findById(post.authorId)
}

// Solution: DataLoader — batches all author loads into ONE query
const userLoader = new DataLoader(async (userIds) => {
  // Called once with [id1, id2, ..., id20]
  const users = await User.find({ _id: { $in: userIds } });
  return userIds.map(id => users.find(u => u.id.toString() === id.toString()));
});

Post: {
  // DataLoader accumulates all calls, then batches into 1 DB query
  author: (post, _, { userLoader }) => userLoader.load(post.authorId)
}
```

### 📖 Resources
- [GraphQL official docs](https://graphql.org/learn/)
- [Apollo Server docs](https://www.apollographql.com/docs/apollo-server/)
- [Strawberry GraphQL (Django)](https://strawberry.rocks/docs/django/guide)
- [DataLoader — solving N+1](https://github.com/graphql/dataloader)

---

## 5. gRPC — Deep Dive

gRPC is a high-performance RPC (Remote Procedure Call) framework by Google. It uses **Protocol Buffers** (binary serialization) over HTTP/2, making it 5–10x faster than REST/JSON for internal service-to-service communication.

### Why gRPC for Microservices

```
REST/JSON:
  {"id": "123", "name": "Ahmed", "email": "ahmed@dev.com"}
  = ~55 bytes as text, must be parsed

Protobuf binary:
  = ~20 bytes, zero parsing overhead
  + generated client code in any language
  + streaming (unary, server-side, client-side, bidirectional)
  + built-in deadlines, cancellation, load balancing
```

### Proto File Definition

```protobuf
// user.proto
syntax = "proto3";
package user;

service UserService {
  rpc GetUser (GetUserRequest) returns (UserResponse);
  rpc CreateUser (CreateUserRequest) returns (UserResponse);
  rpc ListUsers (ListUsersRequest) returns (stream UserResponse); // server streaming
  rpc WatchUser (WatchUserRequest) returns (stream UserEvent);    // server streaming events
}

message GetUserRequest { string id = 1; }

message CreateUserRequest {
  string name     = 1;
  string email    = 2;
  string password = 3;
}

message UserResponse {
  string id         = 1;
  string name       = 2;
  string email      = 3;
  string role       = 4;
  int64  created_at = 5;
}

message ListUsersRequest {
  int32 limit  = 1;
  int32 offset = 2;
}

message WatchUserRequest { string user_id = 1; }
message UserEvent {
  string type    = 1; // "UPDATED" | "DELETED"
  UserResponse user = 2;
}
```

### gRPC Server in Node.js

```javascript
// server.js
const grpc = require("@grpc/grpc-js");
const protoLoader = require("@grpc/proto-loader");
const path = require("path");

const packageDef = protoLoader.loadSync(path.join(__dirname, "user.proto"), {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const grpcObj = grpc.loadPackageDefinition(packageDef);
const UserService = grpcObj.user.UserService;

// Implement service methods
const userServiceImpl = {
  async GetUser(call, callback) {
    try {
      const user = await UserRepository.findById(call.request.id);
      if (!user) {
        return callback({ code: grpc.status.NOT_FOUND, message: "User not found" });
      }
      callback(null, { id: user.id, name: user.name, email: user.email });
    } catch (err) {
      callback({ code: grpc.status.INTERNAL, message: "Internal error" });
    }
  },

  async CreateUser(call, callback) {
    try {
      const user = await UserRepository.create(call.request);
      callback(null, { id: user.id, name: user.name, email: user.email });
    } catch (err) {
      if (err.code === "DUPLICATE") {
        return callback({ code: grpc.status.ALREADY_EXISTS, message: "Email taken" });
      }
      callback({ code: grpc.status.INTERNAL, message: err.message });
    }
  },

  async ListUsers(call) {
    // Server-side streaming — send users one by one
    const { limit = 20, offset = 0 } = call.request;
    const users = await UserRepository.findAll({ limit, offset });
    for (const user of users) {
      call.write({ id: user.id, name: user.name, email: user.email });
    }
    call.end();
  },
};

const server = new grpc.Server();
server.addService(UserService.service, userServiceImpl);
server.bindAsync("0.0.0.0:50051", grpc.ServerCredentials.createInsecure(), () => {
  server.start();
  console.log("gRPC server running on :50051");
});

// client.js — calling the service from another service
const client = new UserService("localhost:50051", grpc.credentials.createInsecure());

// Unary call
client.GetUser({ id: "user-123" }, (err, response) => {
  if (err) throw err;
  console.log(response.name);
});

// Server streaming
const stream = client.ListUsers({ limit: 100, offset: 0 });
stream.on("data", (user) => console.log(user.name));
stream.on("end", () => console.log("All users received"));
stream.on("error", (err) => console.error(err));
```

### gRPC in Django (grpcio)

```python
# server.py
import grpc
from concurrent import futures
import user_pb2
import user_pb2_grpc
from django.contrib.auth.models import User

class UserServicer(user_pb2_grpc.UserServiceServicer):
    def GetUser(self, request, context):
        try:
            user = User.objects.get(pk=request.id)
            return user_pb2.UserResponse(
                id=str(user.id), name=user.get_full_name(), email=user.email
            )
        except User.DoesNotExist:
            context.abort(grpc.StatusCode.NOT_FOUND, "User not found")

    def CreateUser(self, request, context):
        if User.objects.filter(email=request.email).exists():
            context.abort(grpc.StatusCode.ALREADY_EXISTS, "Email already taken")
        user = User.objects.create_user(
            username=request.email,
            email=request.email,
            first_name=request.name,
            password=request.password,
        )
        return user_pb2.UserResponse(id=str(user.id), name=user.first_name, email=user.email)

    def ListUsers(self, request, context):
        # Server streaming
        users = User.objects.all()[request.offset:request.offset + request.limit]
        for user in users:
            yield user_pb2.UserResponse(
                id=str(user.id), name=user.get_full_name(), email=user.email
            )

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServiceServicer_to_server(UserServicer(), server)
    server.add_insecure_port("[::]:50051")
    server.start()
    server.wait_for_termination()
```

### 📖 Resources
- [gRPC official docs](https://grpc.io/docs/)
- [Protocol Buffers docs](https://protobuf.dev/overview/)
- [gRPC vs REST vs GraphQL comparison](https://www.baeldung.com/rest-vs-graphql-vs-grpc)

---

## 6. SOAP — What It Is and Why It's Still Around

SOAP (Simple Object Access Protocol) is a protocol for exchanging structured XML messages over HTTP (or other protocols). It predates REST and GraphQL. You'll encounter it in banking, healthcare, government, and enterprise systems.

### Why SOAP Still Exists

```
- Built-in WS-Security standard (enterprise-grade encryption, digital signatures)
- Formal WSDL contract — auto-generate client code in any language
- WS-ReliableMessaging — guaranteed delivery
- WS-AtomicTransaction — distributed transactions
- Industry mandates: SWIFT (banking), HL7 (healthcare), EDI (logistics)
```

### SOAP Request Structure

```xml
POST /UserService HTTP/1.1
Host: api.enterprise.com
Content-Type: text/xml; charset=utf-8
SOAPAction: "GetUser"

<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope
  xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:usr="http://enterprise.com/UserService">
  <soap:Header>
    <wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/...">
      <wsse:UsernameToken>
        <wsse:Username>apiUser</wsse:Username>
        <wsse:Password>apiPassword</wsse:Password>
      </wsse:UsernameToken>
    </wsse:Security>
  </soap:Header>
  <soap:Body>
    <usr:GetUserRequest>
      <usr:UserId>12345</usr:UserId>
    </usr:GetUserRequest>
  </soap:Body>
</soap:Envelope>
```

### Consuming SOAP in Node.js

```javascript
const soap = require("strong-soap").soap;

const wsdlUrl = "https://enterprise.com/UserService?wsdl";
const options  = {};

soap.createClient(wsdlUrl, options, (err, client) => {
  if (err) throw err;

  client.setSecurity(new soap.BasicAuthSecurity("apiUser", "apiPassword"));

  const args = { UserId: "12345" };
  client.GetUser(args, (err, result) => {
    if (err) throw err;
    console.log(result.User.Name); // XML is parsed into a JS object
  });
});
```

### Consuming SOAP in Django

```python
from zeep import Client, Settings
from zeep.transports import Transport
import requests

session = requests.Session()
session.auth = ("apiUser", "apiPassword")

settings = Settings(strict=False, xml_huge_tree=True)
client = Client(
    wsdl="https://enterprise.com/UserService?wsdl",
    transport=Transport(session=session),
    settings=settings,
)

result = client.service.GetUser(UserId="12345")
print(result.User.Name)
```

---

## 7. Authentication vs Authorization

Two concepts that are always confused but are fundamentally different.

```
Authentication (AuthN) — WHO are you?
  "Prove your identity" — username/password, OAuth token, API key, certificate

Authorization (AuthZ) — WHAT can you do?
  "You're allowed to do this" — roles, permissions, ownership checks

Flow:
  Request → Authentication middleware → Authorization middleware → Route handler
```

### The Key Distinction in Code

```javascript
// authentication.js — verifies identity
async function authenticate(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "No token — who are you?" });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}

// authorization.js — checks permission
function authorize(...requiredRoles) {
  return (req, res, next) => {
    if (!req.user) return res.status(401).json({ error: "Not authenticated" });
    if (!requiredRoles.includes(req.user.role)) {
      return res.status(403).json({ error: "You don't have permission for this" });
    }
    next();
  };
}

// checkOwnership.js — resource-level authorization
async function checkOwnership(req, res, next) {
  const post = await Post.findById(req.params.id);
  if (!post) return res.status(404).json({ error: "Not found" });
  if (post.authorId !== req.user.id && req.user.role !== "admin") {
    return res.status(403).json({ error: "This isn't your post" });
  }
  req.post = post;
  next();
}

// Usage
router.delete("/posts/:id", authenticate, checkOwnership, deletePost);
router.get("/admin/users", authenticate, authorize("admin", "super_admin"), listUsers);
```

```python
# Django — permission classes
from rest_framework.permissions import BasePermission

class IsOwnerOrAdmin(BasePermission):
    """Object-level permission — only owner or admin can modify."""
    def has_object_permission(self, request, view, obj):
        if request.user.is_staff:
            return True
        return obj.author == request.user

class IsAdmin(BasePermission):
    def has_permission(self, request, view):
        return request.user and request.user.is_staff

# Using in a ViewSet
class PostViewSet(viewsets.ModelViewSet):
    def get_permissions(self):
        if self.action in ["update", "partial_update", "destroy"]:
            return [IsAuthenticated(), IsOwnerOrAdmin()]
        if self.action == "admin_action":
            return [IsAuthenticated(), IsAdmin()]
        return [IsAuthenticated()]
```

---

## 8. JWT, Sessions & OAuth2

### Sessions (Stateful)

```
Client logs in → Server creates a session → Stores session in DB/Redis
Server sends session_id in a cookie → Client sends cookie on every request
Server looks up session_id to identify user

Pros:  Easy to invalidate (delete session), works everywhere
Cons:  State on server — hard to scale horizontally without sticky sessions or Redis
```

### JWT (Stateless)

```
Client logs in → Server creates and signs a JWT → Sends it to client
Client stores JWT → Sends in Authorization: Bearer <token> header
Server verifies signature — no DB lookup needed!

JWT structure:
  header.payload.signature
  eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiIxMjMifQ.abc123

Pros:  Stateless — any server can verify, great for microservices
Cons:  Cannot be invalidated before expiry (need a blocklist, which adds state)
       If stolen, attacker has access until expiry
```

### Secure JWT Implementation

```javascript
// auth.service.js
const jwt  = require("jsonwebtoken");
const crypto = require("crypto");

const ACCESS_TOKEN_TTL  = "15m";  // short-lived
const REFRESH_TOKEN_TTL = "7d";   // long-lived

async function login(email, password) {
  const user = await User.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.passwordHash)) {
    throw new UnauthorizedError("Invalid credentials");
  }

  // Access token — short-lived, lives in memory or Authorization header
  const accessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: ACCESS_TOKEN_TTL, issuer: "myapp", audience: "myapp-client" }
  );

  // Refresh token — long-lived, stored in httpOnly cookie + DB for revocation
  const refreshToken = crypto.randomBytes(40).toString("hex");
  const hashedRefreshToken = crypto.createHash("sha256").update(refreshToken).digest("hex");

  await RefreshToken.create({
    userId: user.id,
    tokenHash: hashedRefreshToken,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
  });

  return { accessToken, refreshToken };
}

async function refreshAccessToken(refreshToken) {
  const tokenHash = crypto.createHash("sha256").update(refreshToken).digest("hex");
  const stored = await RefreshToken.findOne({
    tokenHash,
    expiresAt: { $gt: new Date() },
    revokedAt: null,
  });
  if (!stored) throw new UnauthorizedError("Invalid or expired refresh token");

  const user = await User.findById(stored.userId);
  const newAccessToken = jwt.sign(
    { userId: user.id, role: user.role },
    process.env.JWT_SECRET,
    { expiresIn: ACCESS_TOKEN_TTL }
  );
  return newAccessToken;
}

async function logout(refreshToken) {
  const tokenHash = crypto.createHash("sha256").update(refreshToken).digest("hex");
  await RefreshToken.updateOne({ tokenHash }, { revokedAt: new Date() });
}
```

```python
# Django — JWT with SimpleJWT
# settings.py
from datetime import timedelta

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME":  timedelta(minutes=15),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
    "ROTATE_REFRESH_TOKENS":  True,   # new refresh token on each use
    "BLACKLIST_AFTER_ROTATION": True,  # blacklist old refresh token
    "ALGORITHM": "HS256",
    "AUTH_HEADER_TYPES": ("Bearer",),
}

INSTALLED_APPS = [
    "rest_framework_simplejwt",
    "rest_framework_simplejwt.token_blacklist",
]

# urls.py
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView, TokenBlacklistView
urlpatterns = [
    path("auth/login/",   TokenObtainPairView.as_view()),
    path("auth/refresh/", TokenRefreshView.as_view()),
    path("auth/logout/",  TokenBlacklistView.as_view()),
]
```

### OAuth2 Flow (Authorization Code)

```
1. User clicks "Login with Google"
2. Your server redirects to: https://accounts.google.com/oauth2/auth
   ?client_id=YOUR_CLIENT_ID
   &redirect_uri=https://yourapp.com/auth/callback
   &scope=email profile
   &response_type=code
   &state=RANDOM_CSRF_TOKEN

3. User logs in on Google → Google redirects to YOUR callback:
   https://yourapp.com/auth/callback?code=AUTH_CODE&state=RANDOM_CSRF_TOKEN

4. Your server exchanges the code for tokens:
   POST https://oauth2.googleapis.com/token
   { code, client_id, client_secret, redirect_uri, grant_type: "authorization_code" }

5. Google returns: { access_token, refresh_token, id_token }
6. You decode id_token (JWT) to get user info, create/find user in your DB
7. Issue your own session/JWT for your application
```

```javascript
// OAuth2 callback handler in Express
const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth20").Strategy;

passport.use(new GoogleStrategy({
  clientID:     process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL:  "https://yourapp.com/auth/google/callback",
}, async (accessToken, refreshToken, profile, done) => {
  try {
    let user = await User.findOne({ googleId: profile.id });
    if (!user) {
      user = await User.create({
        googleId: profile.id,
        email:    profile.emails[0].value,
        name:     profile.displayName,
        avatar:   profile.photos[0].value,
      });
    }
    return done(null, user);
  } catch (err) {
    return done(err);
  }
}));

router.get("/auth/google", passport.authenticate("google", { scope: ["email", "profile"] }));
router.get("/auth/google/callback",
  passport.authenticate("google", { session: false }),
  (req, res) => {
    const token = jwt.sign({ userId: req.user.id }, process.env.JWT_SECRET, { expiresIn: "15m" });
    res.redirect(`https://yourapp.com/app?token=${token}`);
  }
);
```

### 📖 Resources
- [JWT.io — JWT debugger and explanation](https://jwt.io/)
- [OAuth 2.0 Simplified](https://www.oauth.com/)
- [The OAuth 2.0 Authorization Framework (RFC 6749)](https://datatracker.ietf.org/doc/html/rfc6749)
- [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

---

## 9. Idempotency

**Idempotency** means calling an operation multiple times produces the same result as calling it once.

```
Safe:      GET /users — reading never changes state
Idempotent: PUT /users/1 — setting name to "Ahmed" twice = same result as once
            DELETE /users/1 — deleting twice = still deleted (404 on second is fine)

NOT idempotent: POST /orders — each call creates a new order!
                POST /payments — each call charges the user again!
```

### Why Idempotency Matters

```
Network is unreliable. When a POST /payments times out:
  - Did the payment go through? Or did it fail?
  - If you retry, you might double-charge the user.

Solution: Idempotency keys — the client sends a unique key.
  The server uses it to deduplicate: "I already processed this key, here's the stored result."
```

### Implementing Idempotency Keys

```javascript
// idempotency.middleware.js
const redis = require("../config/redis");

async function idempotencyMiddleware(req, res, next) {
  // Only apply to state-changing requests
  if (!["POST", "PUT", "PATCH"].includes(req.method)) return next();

  const idempotencyKey = req.headers["idempotency-key"];
  if (!idempotencyKey) return next(); // key is optional unless you enforce it

  const cacheKey = `idempotency:${req.user.id}:${idempotencyKey}`;

  // Check if we've seen this key before
  const cached = await redis.get(cacheKey);
  if (cached) {
    const stored = JSON.parse(cached);
    // Replay the original response
    return res.status(stored.status).json(stored.body);
  }

  // Intercept the response to store it
  const originalJson = res.json.bind(res);
  res.json = async (body) => {
    // Store the result for 24 hours
    await redis.setex(cacheKey, 86400, JSON.stringify({
      status: res.statusCode,
      body,
    }));
    return originalJson(body);
  };

  next();
}

// Usage
router.post("/payments", idempotencyMiddleware, processPayment);

// Client sends:
// POST /payments
// Idempotency-Key: a8098c1a-f86e-11da-bd1a-00112444be1e
// { amount: 5000, currency: "EGP" }
//
// If the request times out and the client retries with the SAME key,
// the server returns the original response — no double charge.
```

```python
# Django idempotency middleware
import json
from django.core.cache import cache

class IdempotencyMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        if request.method not in ("POST", "PUT", "PATCH"):
            return self.get_response(request)

        key = request.headers.get("Idempotency-Key")
        if not key:
            return self.get_response(request)

        user_id = getattr(request.user, "id", "anonymous")
        cache_key = f"idempotency:{user_id}:{key}"

        cached = cache.get(cache_key)
        if cached:
            from django.http import JsonResponse
            return JsonResponse(cached["body"], status=cached["status"], safe=False)

        response = self.get_response(request)

        # Cache the response (24 hours)
        try:
            body = json.loads(response.content)
            cache.set(cache_key, {"status": response.status_code, "body": body}, 86400)
        except (json.JSONDecodeError, AttributeError):
            pass

        return response
```

### 📖 Resources
- [Stripe — Idempotent Requests](https://stripe.com/docs/api/idempotent_requests)
- [Designing robust and predictable APIs with idempotency](https://brandur.org/idempotency-keys)

---

## 10. Caching Strategies

Caching stores the result of expensive computations or queries so future requests are faster.

### Where to Cache

```
1. In-memory (per-process) — fastest, not shared across instances
2. Redis / Memcached — fast, shared across all server instances
3. CDN — cache at the network edge, closest to the user
4. Database query cache — DB-level result caching
5. HTTP Cache headers — cache in the browser / proxy
```

### Caching Patterns

```
Cache-Aside (Lazy Loading) — most common:
  1. Check cache
  2. If hit: return cached value
  3. If miss: fetch from DB, store in cache, return

Write-Through:
  Write to cache AND DB simultaneously
  Pros: cache always has latest data
  Cons: write latency, cache filled with data that might not be read

Write-Behind (Write-Back):
  Write to cache, asynchronously write to DB
  Pros: very fast writes
  Cons: risk of data loss if cache crashes before flush

Read-Through:
  Cache sits in front — if miss, cache fetches from DB itself
  (How Redis + read-through libraries work)
```

### Cache-Aside Implementation

```javascript
// cache.service.js
const redis = require("./redis");

const DEFAULT_TTL = 3600; // 1 hour

async function getOrSet(key, fetchFn, ttl = DEFAULT_TTL) {
  // 1. Try cache
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // 2. Cache miss — fetch from DB
  const data = await fetchFn();

  // 3. Store in cache
  if (data !== null && data !== undefined) {
    await redis.setex(key, ttl, JSON.stringify(data));
  }

  return data;
}

async function invalidate(key) {
  await redis.del(key);
}

async function invalidatePattern(pattern) {
  const keys = await redis.keys(pattern); // e.g., "user:*"
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}

// In your service layer:
const UserService = {
  async getUser(id) {
    return getOrSet(
      `user:${id}`,
      () => db.query("SELECT * FROM users WHERE id = $1", [id]).then(r => r.rows[0]),
      3600
    );
  },

  async updateUser(id, data) {
    const user = await db.query(
      "UPDATE users SET name=$1, email=$2 WHERE id=$3 RETURNING *",
      [data.name, data.email, id]
    ).then(r => r.rows[0]);

    // Invalidate the cache entry so next read gets fresh data
    await invalidate(`user:${id}`);
    await invalidate(`users:list:*`); // invalidate list caches too
    return user;
  },
};
```

```python
# Django cache-aside with Redis
from django.core.cache import cache
from django.conf import settings
import json

CACHE_TTL = getattr(settings, "CACHE_TTL", 3600)

def get_or_set_cache(key, fetch_fn, ttl=CACHE_TTL):
    """Generic cache-aside helper."""
    cached = cache.get(key)
    if cached is not None:
        return cached

    data = fetch_fn()
    if data is not None:
        cache.set(key, data, ttl)
    return data

# In a Django view/service:
def get_user(user_id):
    return get_or_set_cache(
        f"user:{user_id}",
        lambda: User.objects.filter(pk=user_id).values().first(),
    )

def update_user(user_id, data):
    User.objects.filter(pk=user_id).update(**data)
    # Invalidate
    cache.delete(f"user:{user_id}")
    cache.delete_many([k for k in cache.keys("users:list:*")])
    return User.objects.get(pk=user_id)

# settings.py — Redis cache backend
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": { "CLIENT_CLASS": "django_redis.client.DefaultClient" }
    }
}
```

### Cache Invalidation Strategies

```
Time-based (TTL): Set expiry — simplest, but data can be stale up to TTL
Event-based:      Invalidate on write — fresh data, more complex
Tag-based:        Group cache entries by tag, invalidate all with a tag
Versioned keys:   user:123:v5 — bump version on change, old keys expire naturally
```

### HTTP Cache Headers

```javascript
// Express — set cache headers
router.get("/users/:id", async (req, res) => {
  const user = await UserService.getUser(req.params.id);

  // Public: cacheable by browsers AND CDNs
  // Private: cacheable only by the user's browser (for personal data)
  // max-age: cache for 5 minutes
  // stale-while-revalidate: serve stale for 60s while fetching fresh
  res.set("Cache-Control", "private, max-age=300, stale-while-revalidate=60");
  res.set("ETag", `"${user.updatedAt.getTime()}"`);
  res.json(user);
});
```

### 📖 Resources
- [Redis Caching Patterns](https://redis.io/docs/manual/patterns/)
- [Cloudflare — Caching strategies](https://developers.cloudflare.com/cache/concepts/cache-control/)

---

## 11. Rate Limiting & Throttling

Rate limiting protects your API from abuse, DoS attacks, and ensures fair use.

### Algorithms

```
Fixed Window:
  Allow 100 requests per minute per user.
  Resets every minute. Flaw: burst at boundary (200 requests in 2 seconds).

Sliding Window:
  Track requests in the last 60 seconds continuously.
  Smoother than fixed window, eliminates the boundary burst.

Token Bucket:
  Each user has a bucket of tokens. Each request consumes a token.
  Tokens refill at a fixed rate. Allows bursting up to bucket capacity.

Leaky Bucket:
  Requests go into a queue (bucket). Queue drains at a fixed rate.
  Smooths out bursts — no spikes, consistent rate.
```

### Rate Limiting in Express

```javascript
const rateLimit = require("express-rate-limit");
const RedisStore = require("rate-limit-redis");
const redis = require("./config/redis");

// Basic limiter — all routes
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,
  standardHeaders: true,    // Return RateLimit-* headers
  legacyHeaders: false,
  store: new RedisStore({ sendCommand: (...args) => redis.call(...args) }),
  message: { error: { code: "RATE_LIMIT", message: "Too many requests, slow down." } },
  keyGenerator: (req) => req.user?.id || req.ip, // per-user or per-IP
});

// Strict limiter — auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10, // only 10 login attempts per 15 minutes
  skipSuccessfulRequests: true, // don't count successful logins
  message: { error: { code: "RATE_LIMIT", message: "Too many login attempts." } },
});

// API tier limiter — different limits per plan
const apiLimiter = (req, res, next) => {
  const limits = { free: 100, pro: 1000, enterprise: 10000 };
  const limit = limits[req.user?.plan] || limits.free;

  return rateLimit({
    windowMs: 60 * 60 * 1000, // 1 hour
    max: limit,
    keyGenerator: (req) => `plan:${req.user?.id}`,
  })(req, res, next);
};

app.use("/api/", globalLimiter);
app.use("/api/auth/", authLimiter);
app.use("/api/v1/", apiLimiter);
```

```python
# Django — rate limiting with django-ratelimit
from django_ratelimit.decorators import ratelimit
from django_ratelimit.exceptions import Ratelimited
from django.http import JsonResponse

# Decorator approach
@ratelimit(key="user", rate="100/h", method=["POST"], block=True)
def create_order(request):
    pass

# Auth endpoint — by IP
@ratelimit(key="ip", rate="10/15m", method=["POST"], block=True)
def login(request):
    pass

# Global exception handler
def ratelimited_error(request, exception):
    return JsonResponse(
        {"error": {"code": "RATE_LIMIT", "message": "Too many requests"}},
        status=429
    )

# settings.py
RATELIMIT_ENABLE     = True
RATELIMIT_USE_CACHE  = "default"  # uses Redis cache defined above
RATELIMIT_FAIL_OPEN  = False      # deny when Redis is down (safer)

# middleware approach for DRF
from rest_framework.throttling import UserRateThrottle, AnonRateThrottle

class BurstRateThrottle(UserRateThrottle):
    scope = "burst"

class SustainedRateThrottle(UserRateThrottle):
    scope = "sustained"

# settings.py
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": [
        "myapp.throttles.BurstRateThrottle",
        "myapp.throttles.SustainedRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {
        "burst":     "60/min",
        "sustained": "1000/day",
        "anon":      "20/hour",
    }
}
```

### 📖 Resources
- [Cloudflare — Rate Limiting Guide](https://www.cloudflare.com/learning/bots/what-is-rate-limiting/)
- [Stripe — Rate Limiting](https://stripe.com/blog/rate-limiters)

---

## 12. Concurrency & Race Conditions

A race condition happens when two or more operations read and modify shared data simultaneously, and the final result depends on which operation completes first.

### The Classic Race Condition: Double Spend

```javascript
// BROKEN — race condition when two requests happen simultaneously
async function withdrawMoney(userId, amount) {
  const account = await Account.findById(userId);
  // Thread A reads: balance = 1000
  // Thread B reads: balance = 1000 (at the same time!)

  if (account.balance < amount) throw new Error("Insufficient funds");

  // Thread A withdraws 800: balance = 200 — saves 200
  // Thread B withdraws 800: balance = 200 — saves 200 (wrong! should have failed!)
  await Account.updateOne({ _id: userId }, { $set: { balance: account.balance - amount } });
}
// Result: account goes to -600. Both transactions "succeeded".
```

### Fix 1: Atomic Operations

```javascript
// MongoDB — atomic update with condition check
async function withdrawMoney(userId, amount) {
  const result = await Account.findOneAndUpdate(
    {
      _id: userId,
      balance: { $gte: amount }, // check and update atomically!
    },
    { $inc: { balance: -amount } },
    { new: true, runValidators: true }
  );

  if (!result) throw new InsufficientFundsError("Insufficient balance");
  return result;
}

// PostgreSQL — atomic update with RETURNING
async function withdrawMoney(userId, amount) {
  const { rows } = await db.query(`
    UPDATE accounts
    SET balance = balance - $2
    WHERE id = $1 AND balance >= $2
    RETURNING *
  `, [userId, amount]);

  if (rows.length === 0) throw new InsufficientFundsError("Insufficient balance");
  return rows[0];
}
```

### Fix 2: Database Transactions with Locks

```javascript
// PostgreSQL — SELECT FOR UPDATE (pessimistic locking)
async function transferMoney(fromId, toId, amount) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");

    // Lock both rows — no other transaction can touch them until we commit
    const { rows: [from] } = await client.query(
      "SELECT * FROM accounts WHERE id = $1 FOR UPDATE",
      [fromId]
    );
    const { rows: [to] } = await client.query(
      "SELECT * FROM accounts WHERE id = $1 FOR UPDATE",
      [toId]
    );

    if (from.balance < amount) throw new Error("Insufficient funds");

    await client.query(
      "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
      [amount, fromId]
    );
    await client.query(
      "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
      [amount, toId]
    );

    await client.query("COMMIT");
    return { from: from.balance - amount, to: to.balance + amount };
  } catch (err) {
    await client.query("ROLLBACK");
    throw err;
  } finally {
    client.release();
  }
}
```

```python
# Django ORM — atomic transaction with select_for_update
from django.db import transaction

def transfer_money(from_id, to_id, amount):
    with transaction.atomic():
        # Lock both rows — no other transaction can read/write them
        from_account = Account.objects.select_for_update().get(pk=from_id)
        to_account   = Account.objects.select_for_update().get(pk=to_id)

        if from_account.balance < amount:
            raise ValueError("Insufficient funds")

        from_account.balance -= amount
        to_account.balance   += amount

        from_account.save(update_fields=["balance"])
        to_account.save(update_fields=["balance"])

        return from_account, to_account
```

### Fix 3: Distributed Lock (Redis)

```javascript
// When the race condition spans multiple services or processes
const redlock = require("redlock");
const lock = new redlock([redis], { retryCount: 3, retryDelay: 100 });

async function processUniqueJob(jobId) {
  const resource = `lock:job:${jobId}`;
  const ttl      = 5000; // 5 seconds max lock time

  let acquired;
  try {
    acquired = await lock.acquire(resource, ttl);

    // Only ONE process can be here at a time
    const job = await Job.findById(jobId);
    if (job.status !== "pending") return; // already processed
    await processJob(job);
    await Job.updateOne({ _id: jobId }, { status: "done" });
  } catch (err) {
    if (err instanceof redlock.LockError) {
      console.log("Another process is handling this job");
      return;
    }
    throw err;
  } finally {
    if (acquired) await lock.release(acquired);
  }
}
```

### 📖 Resources
- [PostgreSQL — Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)
- [Redlock — Distributed Locking with Redis](https://redis.io/docs/manual/patterns/distributed-locks/)
- [Martin Kleppmann — Is Redlock safe?](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)

---

## 13. Database Transactions & ACID

A **transaction** is a group of DB operations that either all succeed or all fail. ACID defines what makes a transaction reliable.

### ACID Properties

```
Atomicity   — All or nothing. If step 3 of 5 fails, steps 1 and 2 are rolled back.
              "Transfer money" = debit + credit. If credit fails, debit is undone.

Consistency — Transaction brings DB from one valid state to another.
              A user can't have a negative balance if the schema prevents it.

Isolation   — Concurrent transactions don't interfere with each other.
              Four levels: READ UNCOMMITTED → READ COMMITTED → REPEATABLE READ → SERIALIZABLE

Durability  — Committed transactions survive crashes.
              Written to disk (WAL — Write-Ahead Log) before acknowledging success.
```

### Isolation Levels & Their Anomalies

```
Problem definitions:
  Dirty Read:        Read data another transaction hasn't committed yet
  Non-Repeatable Read: Same query returns different results within same transaction
  Phantom Read:      Query returns different ROWS on second run (insert/delete by another tx)

Level                Dirty Read    Non-Repeatable    Phantom
READ UNCOMMITTED     Possible      Possible          Possible      (never use)
READ COMMITTED       Prevented     Possible          Possible      (PostgreSQL default)
REPEATABLE READ      Prevented     Prevented         Possible      (MySQL default)
SERIALIZABLE         Prevented     Prevented         Prevented     (safest, slowest)
```

### Transactions in Node.js (PostgreSQL)

```javascript
// PostgreSQL transactions with pg
const { Pool } = require("pg");
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Generic transaction wrapper
async function withTransaction(callback) {
  const client = await pool.connect();
  try {
    await client.query("BEGIN");
    const result = await callback(client);
    await client.query("COMMIT");
    return result;
  } catch (err) {
    await client.query("ROLLBACK");
    throw err;
  } finally {
    client.release();
  }
}

// Usage
async function createOrder(userId, items) {
  return withTransaction(async (client) => {
    // 1. Lock user's cart
    const { rows: [user] } = await client.query(
      "SELECT * FROM users WHERE id = $1 FOR UPDATE",
      [userId]
    );

    // 2. Create order
    const { rows: [order] } = await client.query(
      "INSERT INTO orders (user_id, status, total) VALUES ($1, $2, $3) RETURNING *",
      [userId, "pending", calculateTotal(items)]
    );

    // 3. Create order items (batch insert)
    const values = items.flatMap((item, i) => [
      order.id, item.productId, item.quantity, item.price
    ]);
    const placeholders = items.map((_, i) =>
      `($${i * 4 + 1}, $${i * 4 + 2}, $${i * 4 + 3}, $${i * 4 + 4})`
    ).join(", ");

    await client.query(
      `INSERT INTO order_items (order_id, product_id, quantity, price) VALUES ${placeholders}`,
      values
    );

    // 4. Decrement stock for each item
    for (const item of items) {
      const { rows } = await client.query(`
        UPDATE products
        SET stock = stock - $1
        WHERE id = $2 AND stock >= $1
        RETURNING id
      `, [item.quantity, item.productId]);

      if (rows.length === 0) {
        throw new Error(`Insufficient stock for product ${item.productId}`);
        // This triggers ROLLBACK — everything above is undone
      }
    }

    return order;
  });
}
```

```python
# Django ORM transactions
from django.db import transaction
from django.db.models import F

def create_order(user_id, items):
    with transaction.atomic():
        # savepoint() — nested transactions
        user = User.objects.select_for_update().get(pk=user_id)
        total = sum(item["price"] * item["quantity"] for item in items)

        order = Order.objects.create(user=user, status="pending", total=total)

        for item in items:
            OrderItem.objects.create(
                order=order,
                product_id=item["product_id"],
                quantity=item["quantity"],
                price=item["price"],
            )
            # Atomic decrement with check
            updated = Product.objects.filter(
                pk=item["product_id"],
                stock__gte=item["quantity"]
            ).update(stock=F("stock") - item["quantity"])

            if updated == 0:
                raise ValueError(f"Insufficient stock for product {item['product_id']}")
                # transaction.atomic() auto-rollbacks on exception

        return order

# Savepoints — for partial rollback within a transaction
def bulk_process(records):
    with transaction.atomic():
        success_count = 0
        failure_count = 0
        for record in records:
            try:
                with transaction.atomic():  # savepoint
                    process_record(record)
                    success_count += 1
            except Exception as e:
                # Only this record's changes are rolled back
                failure_count += 1
                log_failure(record, e)
        return success_count, failure_count
```

### MySQL Transactions

```javascript
// MySQL2 transactions
const mysql = require("mysql2/promise");
const pool = mysql.createPool({ uri: process.env.MYSQL_URL });

async function transferCredits(fromId, toId, amount) {
  const conn = await pool.getConnection();
  try {
    await conn.beginTransaction();

    // Lock rows in consistent order (always lock lower ID first to prevent deadlocks)
    const [first, second] = fromId < toId ? [fromId, toId] : [toId, fromId];
    await conn.query("SELECT * FROM wallets WHERE id = ? FOR UPDATE", [first]);
    await conn.query("SELECT * FROM wallets WHERE id = ? FOR UPDATE", [second]);

    const [[from]] = await conn.query("SELECT balance FROM wallets WHERE id = ?", [fromId]);
    if (from.balance < amount) throw new Error("Insufficient funds");

    await conn.query("UPDATE wallets SET balance = balance - ? WHERE id = ?", [amount, fromId]);
    await conn.query("UPDATE wallets SET balance = balance + ? WHERE id = ?", [amount, toId]);

    await conn.commit();
  } catch (err) {
    await conn.rollback();
    throw err;
  } finally {
    conn.release();
  }
}
```

---

## 14. Optimistic vs Pessimistic Locking

Two strategies for handling concurrent modifications to the same data.

### Pessimistic Locking — "Assume conflict, block others"

```sql
-- Lock the row immediately — other transactions must wait
SELECT * FROM orders WHERE id = 1 FOR UPDATE;
-- Now update safely...
UPDATE orders SET status = 'processing' WHERE id = 1;
COMMIT;
```

Good for: high-contention data (bank accounts, inventory), short transactions.
Bad for: long-running operations — blocks other reads/writes.

### Optimistic Locking — "Assume no conflict, check at commit"

```javascript
// Add a 'version' column to your table
// Each update increments the version

async function updateOrder(orderId, updates, expectedVersion) {
  const result = await db.query(`
    UPDATE orders
    SET status = $1, updated_at = NOW(), version = version + 1
    WHERE id = $2
      AND version = $3  -- only update if version matches!
    RETURNING *
  `, [updates.status, orderId, expectedVersion]);

  if (result.rows.length === 0) {
    throw new ConflictError(
      "Order was modified by someone else. Please reload and try again."
    );
  }
  return result.rows[0];
}

// Client flow:
// 1. GET /orders/1 → { id: 1, status: "pending", version: 3 }
// 2. User edits...
// 3. PATCH /orders/1 { status: "cancelled", version: 3 }
//    → if version still 3: success (version becomes 4)
//    → if version is now 4 (someone else changed it): 409 Conflict
```

```python
# Django ORM optimistic locking with django-concurrency
from concurrency.fields import IntegerVersionField

class Order(models.Model):
    status  = models.CharField(max_length=50)
    version = IntegerVersionField()

    class Meta:
        app_label = "orders"

# The save() call will raise RecordModifiedError if version doesn't match
from concurrency.exceptions import RecordModifiedError

def update_order_status(order_id, new_status, expected_version):
    try:
        order = Order.objects.get(pk=order_id)
        order.version = expected_version  # set expected version
        order.status  = new_status
        order.save()    # raises RecordModifiedError if version changed
    except RecordModifiedError:
        raise ConflictError("Order was updated by another request")
```

| | Pessimistic | Optimistic |
|---|---|---|
| When | High contention, short tx | Low contention, long user operations |
| Approach | Lock first, then update | Update, fail if version mismatch |
| Throughput | Lower (others wait) | Higher (no blocking) |
| Complexity | Simple | Requires retry logic on client |
| Use case | Bank transfers | UI form submissions |

---

## 15. N+1 Problem & Query Optimization

The N+1 problem is when you make 1 query to get a list, then N more queries to get related data — one per item.

### Reproducing the N+1 Problem

```javascript
// N+1 in Express + Sequelize
app.get("/posts", async (req, res) => {
  const posts = await Post.findAll({ limit: 20 }); // 1 query

  // For each post, load the author — 20 more queries!
  for (const post of posts) {
    post.dataValues.author = await User.findByPk(post.authorId); // N queries
  }
  // Total: 21 queries for 20 posts
  res.json(posts);
});

// Fix: eager loading with include
app.get("/posts", async (req, res) => {
  const posts = await Post.findAll({
    limit: 20,
    include: [{ model: User, as: "author", attributes: ["id", "name", "avatar"] }]
  });
  // Total: 1 query (JOIN)
  res.json(posts);
});
```

```python
# N+1 in Django ORM
def get_posts(request):
    posts = Post.objects.all()[:20]  # 1 query
    # Django's lazy loading: accessing post.author for each post = N queries
    data = [{"title": p.title, "author": p.author.name} for p in posts]  # N+1!

    # Fix: select_related (for ForeignKey — SQL JOIN)
    posts = Post.objects.select_related("author").all()[:20]  # 1 query with JOIN

    # Fix: prefetch_related (for ManyToMany or reverse FK — separate query, Python join)
    posts = Post.objects.prefetch_related("tags", "comments__author").all()[:20]
    # 3 queries total: posts + tags + comments with authors
```

### Query Analysis Tools

```javascript
// PostgreSQL EXPLAIN ANALYZE
const { rows } = await db.query(`
  EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
  SELECT p.*, u.name as author_name
  FROM posts p
  JOIN users u ON u.id = p.author_id
  WHERE p.status = 'published'
  ORDER BY p.created_at DESC
  LIMIT 20
`);

// Look for:
// - Seq Scan (bad on large tables → needs index)
// - Index Scan (good — using index)
// - Nested Loop on large result sets (might need Hash Join)
// - High actual rows vs estimated rows (stale statistics → run ANALYZE)
```

```python
# Django query debugging
from django.db import connection, reset_queries

# In development — log all queries
reset_queries()
posts = list(Post.objects.select_related("author").all()[:20])
print(f"Queries executed: {len(connection.queries)}")
for q in connection.queries:
    print(q["sql"])
    print(f"Time: {q['time']}s")

# Or use Django Debug Toolbar in browser
```

---

## 16. Indexes & When to Use Them

```sql
-- When to create an index:
-- 1. Columns in WHERE clauses you query frequently
CREATE INDEX idx_users_email ON users(email);

-- 2. Columns in ORDER BY or GROUP BY
CREATE INDEX idx_posts_created_at ON posts(created_at DESC);

-- 3. Foreign key columns (JOIN performance)
CREATE INDEX idx_posts_author_id ON posts(author_id);

-- 4. Composite index — covers multi-column queries (order matters: equality first)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
-- Covers: WHERE user_id = X AND status = Y
-- Also covers: WHERE user_id = X (left prefix rule)
-- Does NOT cover: WHERE status = Y (alone)

-- 5. Partial index — index a subset of rows
CREATE INDEX idx_active_users ON users(email) WHERE deleted_at IS NULL;

-- 6. Covering index (PostgreSQL INCLUDE) — index includes extra columns
-- Allows index-only scan without hitting the table
CREATE INDEX idx_posts_cover ON posts(status, created_at)
INCLUDE (id, title, author_id);

-- When NOT to use an index:
-- - Columns with very low cardinality (e.g., boolean, status with 2-3 values)
-- - Tables with very few rows (< 1000 rows — full scan is faster)
-- - Columns that change very frequently (index maintenance cost > read benefit)
-- - After bulk imports — drop indexes, bulk insert, recreate
```

```javascript
// MongoDB indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.posts.createIndex({ authorId: 1, createdAt: -1 }); // compound
db.posts.createIndex({ status: 1 }, { partialFilterExpression: { status: { $ne: "deleted" } } });
db.posts.createIndex({ title: "text", body: "text" }); // full-text search

// Mongoose
const PostSchema = new Schema({
  title:    { type: String, index: true },
  status:   String,
  author:   { type: ObjectId, ref: "User", index: true },
  tags:     [String],
  createdAt:{ type: Date, default: Date.now },
});
PostSchema.index({ status: 1, createdAt: -1 }); // compound
PostSchema.index({ title: "text", body: "text" }); // text search
```

---

## 17. Database Migrations

Migrations are version-controlled changes to your database schema.

### Node.js (Knex.js Migrations)

```javascript
// migrations/20240401_create_users.js
exports.up = async (knex) => {
  await knex.schema.createTable("users", (table) => {
    table.uuid("id").primary().defaultTo(knex.raw("gen_random_uuid()"));
    table.string("name", 100).notNullable();
    table.string("email", 255).notNullable().unique();
    table.string("password_hash").notNullable();
    table.enum("role", ["admin", "user", "moderator"]).defaultTo("user");
    table.enum("status", ["active", "inactive", "banned"]).defaultTo("active");
    table.timestamp("created_at").defaultTo(knex.fn.now());
    table.timestamp("updated_at").defaultTo(knex.fn.now());
    table.timestamp("deleted_at").nullable();

    table.index(["email"]);
    table.index(["status", "created_at"]);
  });
};

exports.down = async (knex) => {
  await knex.schema.dropTableIfExists("users");
};

// migrations/20240402_add_avatar_to_users.js
exports.up = async (knex) => {
  await knex.schema.table("users", (table) => {
    table.string("avatar_url").nullable();
    table.string("phone", 20).nullable();
  });
};

exports.down = async (knex) => {
  await knex.schema.table("users", (table) => {
    table.dropColumn("avatar_url");
    table.dropColumn("phone");
  });
};
```

### Python Django Migrations

```python
# Django auto-generates migrations from model changes
# Run: python manage.py makemigrations

# models.py
from django.db import models
import uuid

class User(models.Model):
    id         = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name       = models.CharField(max_length=100)
    email      = models.EmailField(unique=True, db_index=True)
    role       = models.CharField(max_length=20, choices=[("admin","Admin"),("user","User")], default="user")
    status     = models.CharField(max_length=20, default="active", db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    updated_at = models.DateTimeField(auto_now=True)
    deleted_at = models.DateTimeField(null=True, blank=True)

    class Meta:
        indexes = [
            models.Index(fields=["status", "-created_at"]),
        ]

# Generated migration:
# python manage.py makemigrations
# python manage.py migrate

# Custom data migration (run Python code during migration)
from django.db import migrations

def populate_slugs(apps, schema_editor):
    Post = apps.get_model("blog", "Post")
    for post in Post.objects.all():
        post.slug = slugify(post.title)
        post.save(update_fields=["slug"])

class Migration(migrations.Migration):
    dependencies = [("blog", "0004_post_slug")]
    operations  = [migrations.RunPython(populate_slugs, migrations.RunPython.noop)]
```

---

## 18. Pagination Patterns

### Offset Pagination (simple but flawed)

```javascript
// Simple but has a drift problem: if items are inserted/deleted
// between pages, users see duplicates or miss items
router.get("/posts", async (req, res) => {
  const page  = parseInt(req.query.page)  || 1;
  const limit = parseInt(req.query.limit) || 20;
  const offset = (page - 1) * limit;

  const [posts, total] = await Promise.all([
    db.query("SELECT * FROM posts ORDER BY created_at DESC LIMIT $1 OFFSET $2", [limit, offset]),
    db.query("SELECT COUNT(*) FROM posts"),
  ]);

  res.json({
    data: posts.rows,
    meta: {
      page,
      limit,
      total: parseInt(total.rows[0].count),
      pages: Math.ceil(total.rows[0].count / limit),
    }
  });
});
```

### Cursor Pagination (stable, production-grade)

```javascript
// Uses a cursor (the last seen ID or timestamp) instead of offset
// No drift: each page picks up exactly where the last left off
router.get("/posts", async (req, res) => {
  const limit  = parseInt(req.query.limit) || 20;
  const cursor = req.query.cursor; // base64-encoded { id, createdAt }

  let query = "SELECT * FROM posts";
  let params = [];

  if (cursor) {
    const { id, createdAt } = JSON.parse(Buffer.from(cursor, "base64").toString());
    // Rows after the cursor (older than the cursor's timestamp)
    query += " WHERE (created_at, id) < ($1, $2)";
    params = [createdAt, id];
  }

  query += " ORDER BY created_at DESC, id DESC LIMIT $" + (params.length + 1);
  params.push(limit + 1); // fetch one extra to know if there's a next page

  const { rows } = await db.query(query, params);
  const hasMore  = rows.length > limit;
  const items    = hasMore ? rows.slice(0, limit) : rows;

  const nextCursor = hasMore
    ? Buffer.from(JSON.stringify({
        id:        items[items.length - 1].id,
        createdAt: items[items.length - 1].created_at,
      })).toString("base64")
    : null;

  res.json({
    data: items,
    meta: { nextCursor, hasMore },
  });
});
```

```python
# Django cursor pagination
from rest_framework.pagination import CursorPagination

class PostCursorPagination(CursorPagination):
    page_size           = 20
    ordering            = "-created_at"
    cursor_query_param  = "cursor"
    max_page_size       = 100

class PostListView(generics.ListAPIView):
    queryset            = Post.objects.filter(status="published")
    serializer_class    = PostSerializer
    pagination_class    = PostCursorPagination
```

| | Offset | Cursor |
|---|---|---|
| Implementation | Simple | More complex |
| Page drift | Yes (items shift between pages) | No |
| Jump to page | Yes (`page=5`) | No (sequential only) |
| Count | Easy (`COUNT(*)`) | Expensive |
| Performance on large data | Degrades (`OFFSET 10000` is slow) | Constant |
| Best for | Admin lists, small data | Feeds, infinite scroll |

---

## 19. File Uploads & Storage

### Direct Upload in Express

```javascript
const multer = require("multer");
const { S3Client, PutObjectCommand } = require("@aws-sdk/client-s3");
const sharp  = require("sharp");

const s3 = new S3Client({ region: process.env.AWS_REGION });

// In-memory storage for processing before upload
const upload = multer({
  storage: multer.memoryStorage(),
  limits: { fileSize: 10 * 1024 * 1024 }, // 10MB
  fileFilter: (req, file, cb) => {
    const allowed = ["image/jpeg", "image/png", "image/webp"];
    if (allowed.includes(file.mimetype)) cb(null, true);
    else cb(new Error("Only JPEG, PNG, and WebP allowed"));
  },
});

router.post("/users/:id/avatar",
  authenticate,
  upload.single("avatar"),
  async (req, res, next) => {
    try {
      if (!req.file) return res.status(400).json({ error: "No file uploaded" });

      // Resize and convert to WebP for efficiency
      const processed = await sharp(req.file.buffer)
        .resize(400, 400, { fit: "cover" })
        .webp({ quality: 85 })
        .toBuffer();

      const key = `avatars/${req.user.id}-${Date.now()}.webp`;

      await s3.send(new PutObjectCommand({
        Bucket:      process.env.S3_BUCKET,
        Key:         key,
        Body:        processed,
        ContentType: "image/webp",
        CacheControl:"max-age=31536000", // cache for 1 year (immutable filename)
      }));

      const avatarUrl = `https://${process.env.CDN_DOMAIN}/${key}`;
      await User.findByIdAndUpdate(req.user.id, { avatarUrl });

      res.json({ data: { avatarUrl } });
    } catch (err) { next(err); }
  }
);
```

### Presigned URLs (Client uploads directly to S3)

```javascript
// Better pattern for large files: don't route through your server
const { getSignedUrl }       = require("@aws-sdk/s3-request-presigner");
const { PutObjectCommand }    = require("@aws-sdk/client-s3");
const { v4: uuidv4 }          = require("uuid");

router.post("/upload/presign", authenticate, async (req, res) => {
  const { contentType, fileSize } = req.body;

  if (fileSize > 50 * 1024 * 1024) return res.status(400).json({ error: "Max 50MB" });

  const allowed = ["image/jpeg", "image/png", "video/mp4"];
  if (!allowed.includes(contentType)) return res.status(400).json({ error: "Type not allowed" });

  const key = `uploads/${req.user.id}/${uuidv4()}`;

  const url = await getSignedUrl(s3, new PutObjectCommand({
    Bucket:      process.env.S3_BUCKET,
    Key:         key,
    ContentType: contentType,
  }), { expiresIn: 300 }); // URL valid for 5 minutes

  res.json({ data: { uploadUrl: url, key } });
});

// Client flow:
// 1. POST /upload/presign → gets { uploadUrl, key }
// 2. PUT uploadUrl (directly to S3, bypasses your server entirely)
// 3. POST /posts { s3Key: key } → server confirms file exists and creates record
```

---

## 20. Background Jobs & Message Queues

### Why Background Jobs?

```
Some operations are too slow for a synchronous HTTP response:
  - Sending emails (500ms - 2s)
  - Generating PDF reports
  - Processing video/image
  - Sending webhooks to third parties
  - Bulk imports
  - Push notifications (millions of devices)

Pattern: Enqueue the job, return 202 Accepted immediately, process async.
```

### Bull Queue (Node.js + Redis)

```javascript
const Queue  = require("bull");
const emailQ = new Queue("email", { redis: process.env.REDIS_URL });
const reportQ= new Queue("reports", { redis: process.env.REDIS_URL });

// Producer — add jobs from your route handler
router.post("/users/:id/send-welcome", async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ error: "User not found" });

  await emailQ.add(
    "welcome-email",
    { userId: user.id, email: user.email, name: user.name },
    {
      attempts: 3,         // retry up to 3 times on failure
      backoff: { type: "exponential", delay: 2000 }, // 2s, 4s, 8s
      removeOnComplete: 100, // keep last 100 completed jobs for debugging
      removeOnFail: 200,
    }
  );

  res.status(202).json({ data: { message: "Email queued" } });
});

// Consumer — in a separate worker process
emailQ.process("welcome-email", async (job) => {
  const { userId, email, name } = job.data;
  console.log(`Processing welcome email for ${email}, attempt ${job.attemptsMade + 1}`);

  await sendEmail({
    to: email,
    subject: "Welcome to Kickoff!",
    template: "welcome",
    context: { name },
  });

  // Mark user as onboarded
  await User.findByIdAndUpdate(userId, { onboardedAt: new Date() });
});

// Event hooks for monitoring
emailQ.on("completed", (job) => console.log(`Job ${job.id} completed`));
emailQ.on("failed",    (job, err) => console.error(`Job ${job.id} failed:`, err.message));
emailQ.on("stalled",   (job) => console.warn(`Job ${job.id} stalled — worker crashed?`));

// Scheduled jobs (cron-like)
reportQ.add("daily-report", {}, {
  repeat: { cron: "0 8 * * *" }, // every day at 8am
  removeOnComplete: 10,
});

reportQ.process("daily-report", async (job) => {
  const report = await generateDailyReport();
  await sendReportEmail(report);
});
```

```python
# Django + Celery (most common combo)
# celery.py
import os
from celery import Celery
from celery.schedules import crontab

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")

app = Celery("myproject")
app.config_from_object("django.conf:settings", namespace="CELERY")
app.autodiscover_tasks()

# settings.py
CELERY_BROKER_URL    = "redis://localhost:6379/0"
CELERY_RESULT_BACKEND= "redis://localhost:6379/0"
CELERY_TASK_SERIALIZER = "json"
CELERY_ACCEPT_CONTENT  = ["json"]

# tasks.py
from celery import shared_task
from celery.exceptions import MaxRetriesExceededError

@shared_task(
    bind=True,
    max_retries=3,
    default_retry_delay=60,
    autoretry_for=(Exception,),
    retry_backoff=True,
)
def send_welcome_email(self, user_id: int):
    from .models import User
    from .emails import send_email

    try:
        user = User.objects.get(pk=user_id)
        send_email(
            to=user.email,
            subject="Welcome!",
            template="emails/welcome.html",
            context={"name": user.name},
        )
        User.objects.filter(pk=user_id).update(onboarded_at=timezone.now())
    except Exception as exc:
        raise self.retry(exc=exc, countdown=2 ** self.request.retries)

# Calling from a view
from .tasks import send_welcome_email

def create_user(request):
    user = User.objects.create(...)
    send_welcome_email.delay(user.id)       # fire and forget
    send_welcome_email.apply_async(
        args=[user.id],
        countdown=60,                        # delay 60 seconds
        queue="high_priority",
    )
    return JsonResponse({"data": UserSerializer(user).data}, status=201)

# Scheduled tasks (Celery Beat)
app.conf.beat_schedule = {
    "daily-report": {
        "task": "myapp.tasks.generate_daily_report",
        "schedule": crontab(hour=8, minute=0),
    },
}
```

### 📖 Resources
- [Bull — Node.js queue library](https://github.com/OptimalBits/bull)
- [Celery documentation](https://docs.celeryq.dev/)
- [RabbitMQ — Message broker](https://www.rabbitmq.com/tutorials)

---

## 21. WebSockets & Real-Time

```javascript
// Express + Socket.io — real-time bidirectional communication
const { createServer } = require("http");
const { Server }       = require("socket.io");
const jwt              = require("jsonwebtoken");

const httpServer = createServer(app);
const io         = new Server(httpServer, {
  cors: { origin: process.env.CLIENT_URL, credentials: true },
});

// Authentication middleware for WebSocket connections
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  try {
    socket.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    next(new Error("Unauthorized"));
  }
});

io.on("connection", (socket) => {
  const userId = socket.user.userId;
  console.log(`User ${userId} connected`);

  // Put user in their own room (for targeted messages)
  socket.join(`user:${userId}`);

  // Join a pitch/match room
  socket.on("join:match", (matchId) => {
    socket.join(`match:${matchId}`);
    socket.to(`match:${matchId}`).emit("user:joined", { userId, matchId });
  });

  socket.on("send:message", async ({ matchId, content }) => {
    const message = await Message.create({ authorId: userId, matchId, content });

    // Broadcast to everyone in the match room (including sender)
    io.to(`match:${matchId}`).emit("new:message", {
      id:        message.id,
      content:   message.content,
      author:    { id: userId, name: socket.user.name },
      createdAt: message.createdAt,
    });
  });

  socket.on("disconnect", () => {
    console.log(`User ${userId} disconnected`);
    socket.to(`match:${socket.currentMatch}`).emit("user:left", { userId });
  });
});

// Emit to a specific user from anywhere in your app (e.g., from a background job)
function notifyUser(userId, event, data) {
  io.to(`user:${userId}`).emit(event, data);
}

// Emit to all users in a room
function notifyMatch(matchId, event, data) {
  io.to(`match:${matchId}`).emit(event, data);
}

httpServer.listen(3000);
```

```python
# Django Channels — WebSocket support for Django
# consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from .models import Message

class MatchConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.match_id = self.scope["url_route"]["kwargs"]["match_id"]
        self.room_group_name = f"match_{self.match_id}"
        self.user = self.scope["user"]

        if not self.user.is_authenticated:
            await self.close()
            return

        # Join match group
        await self.channel_layer.group_add(self.room_group_name, self.channel_name)
        await self.accept()
        await self.channel_layer.group_send(self.room_group_name, {
            "type": "user.joined",
            "user_id": str(self.user.id),
        })

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(self.room_group_name, self.channel_name)

    async def receive(self, text_data):
        data = json.loads(text_data)
        if data.get("type") == "message":
            message = await self.save_message(data["content"])
            await self.channel_layer.group_send(self.room_group_name, {
                "type": "chat.message",
                "message": {
                    "id": str(message.id),
                    "content": message.content,
                    "author": self.user.get_full_name(),
                },
            })

    async def chat_message(self, event):
        await self.send(text_data=json.dumps(event["message"]))

    async def user_joined(self, event):
        await self.send(text_data=json.dumps({"type": "user_joined", "userId": event["user_id"]}))

    @database_sync_to_async
    def save_message(self, content):
        return Message.objects.create(
            match_id=self.match_id, author=self.user, content=content
        )
```

---

## 22. Security: OWASP Top 10 for Backend Devs

### 1. Injection (SQL, NoSQL, Command)

```javascript
// SQL Injection
// BAD — concatenating user input directly
const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;
// Input: ' OR '1'='1 → returns ALL users

// GOOD — parameterized queries (never string concat)
const { rows } = await db.query(
  "SELECT * FROM users WHERE email = $1", [req.body.email]
);

// NoSQL Injection (MongoDB)
// BAD
const user = await User.findOne({ email: req.body.email, password: req.body.password });
// Input: { "email": "admin@test.com", "password": { "$ne": "" } } → bypasses password check!

// GOOD — sanitize or use type checking
const { email, password } = req.body;
if (typeof email !== "string" || typeof password !== "string") {
  return res.status(400).json({ error: "Invalid input" });
}
const user = await User.findOne({ email, password: await bcrypt.hash(password, salt) });
```

### 2. Broken Authentication

```javascript
// Always hash passwords with bcrypt (or argon2)
const SALT_ROUNDS = 12;
const hash = await bcrypt.hash(plainPassword, SALT_ROUNDS);
const isValid = await bcrypt.compare(plainPassword, hash);

// Never log or expose passwords — even in errors
// Never store passwords in plain text, even temporarily
// Use constant-time comparison to prevent timing attacks (bcrypt.compare does this)
```

### 3. Sensitive Data Exposure

```javascript
// Never return sensitive fields in API responses
const user = await User.findById(id);
const { password, passwordResetToken, twoFactorSecret, ...safeUser } = user.toObject();
res.json({ data: safeUser });

// Or define what to expose explicitly (better)
const publicUser = {
  id: user.id,
  name: user.name,
  email: user.email,
  role: user.role,
  createdAt: user.createdAt,
};
```

### 4. IDOR — Insecure Direct Object Reference

```javascript
// BAD — user can access ANY order by changing the ID
router.get("/orders/:id", async (req, res) => {
  const order = await Order.findById(req.params.id); // returns any user's order!
  res.json(order);
});

// GOOD — scope queries to the authenticated user
router.get("/orders/:id", authenticate, async (req, res) => {
  const order = await Order.findOne({
    _id: req.params.id,
    userId: req.user.id, // must belong to this user
  });
  if (!order) return res.status(404).json({ error: "Not found" });
  res.json({ data: order });
});
```

### 5. Security Headers (Express)

```javascript
const helmet = require("helmet");

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc:   ["'self'", "'unsafe-inline'"],
      imgSrc:     ["'self'", "data:", "https:"],
      scriptSrc:  ["'self'"],
    },
  },
  hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
}));

// Additional headers
app.use((req, res, next) => {
  res.set("X-Content-Type-Options", "nosniff");
  res.set("X-Frame-Options", "DENY");
  res.set("Referrer-Policy", "strict-origin-when-cross-origin");
  next();
});
```

### 📖 Resources
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)

---

## 23. Logging, Monitoring & Observability

### The Three Pillars of Observability

```
Logs    — what happened (timestamped events, errors, info)
Metrics — how the system is performing (latency, error rate, CPU)
Traces  — how a request flowed through the system (distributed tracing)
```

### Structured Logging in Node.js

```javascript
const { createLogger, format, transports } = require("winston");

const logger = createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }), // include stack traces
    format.json()                    // structured JSON output
  ),
  defaultMeta: { service: "kickoff-api", version: process.env.APP_VERSION },
  transports: [
    new transports.Console(),
    new transports.File({ filename: "logs/error.log", level: "error" }),
  ],
});

// Request logging middleware
app.use((req, res, next) => {
  const start = Date.now();
  res.on("finish", () => {
    logger.info("HTTP request", {
      method:      req.method,
      url:         req.originalUrl,
      statusCode:  res.statusCode,
      duration_ms: Date.now() - start,
      userId:      req.user?.id,
      ip:          req.ip,
      userAgent:   req.get("user-agent"),
    });
  });
  next();
});

// Use structured logging everywhere
logger.error("Payment failed", {
  userId:  req.user.id,
  orderId: order.id,
  amount:  order.total,
  error:   err.message,
  stack:   err.stack,
});
```

```python
# Django structured logging
import logging
import structlog

structlog.configure(
    processors=[
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer(),
    ],
)

logger = structlog.get_logger()

# In a view
def create_order(request):
    log = logger.bind(user_id=request.user.id)
    try:
        order = OrderService.create(request.user, request.data)
        log.info("order.created", order_id=str(order.id), total=float(order.total))
        return Response(OrderSerializer(order).data, status=201)
    except InsufficientFundsError as e:
        log.warning("order.insufficient_funds", amount=float(request.data.get("total")))
        return Response({"error": str(e)}, status=400)
    except Exception as e:
        log.error("order.unexpected_error", error=str(e), exc_info=True)
        raise
```

### Health Check Endpoint

```javascript
// GET /health — used by load balancers, Kubernetes, monitoring tools
router.get("/health", async (req, res) => {
  const checks = {
    database: "ok",
    redis:    "ok",
    uptime:   process.uptime(),
    memory:   process.memoryUsage(),
  };

  try {
    await db.query("SELECT 1"); // quick DB ping
  } catch {
    checks.database = "error";
  }

  try {
    await redis.ping();
  } catch {
    checks.redis = "error";
  }

  const isHealthy = Object.values(checks).every(v => v === "ok" || typeof v !== "string");
  res.status(isHealthy ? 200 : 503).json({ status: isHealthy ? "healthy" : "unhealthy", checks });
});
```

### 📖 Resources
- [Winston — Node.js logging](https://github.com/winstonjs/winston)
- [structlog — Python logging](https://www.structlog.org/)
- [The three pillars of observability](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/ch04.html)
- [OpenTelemetry — Distributed tracing](https://opentelemetry.io/)

---

## 24. Environment Config & Secrets Management

```javascript
// .env — never commit this to git!
DATABASE_URL=postgresql://user:pass@localhost/mydb
JWT_SECRET=super-long-random-string-here
REDIS_URL=redis://localhost:6379
STRIPE_SECRET_KEY=sk_live_xxxx
AWS_ACCESS_KEY_ID=AKIA...

// config.js — centralized, validated config
const { z } = require("zod");
require("dotenv").config();

const schema = z.object({
  NODE_ENV:       z.enum(["development", "staging", "production"]),
  PORT:           z.string().transform(Number).default("3000"),
  DATABASE_URL:   z.string().url(),
  JWT_SECRET:     z.string().min(32, "JWT_SECRET must be at least 32 chars"),
  REDIS_URL:      z.string().url(),
  AWS_REGION:     z.string().default("us-east-1"),
  STRIPE_SECRET:  z.string().optional(),
});

const parsed = schema.safeParse(process.env);
if (!parsed.success) {
  console.error("Invalid environment config:", parsed.error.format());
  process.exit(1); // fail fast on bad config — don't silently use undefined values
}

module.exports = parsed.data;
// Now: const { DATABASE_URL, JWT_SECRET } = require("./config");
```

```python
# Django — environment config with django-environ
import environ
env = environ.Env(
    DEBUG=(bool, False),
    ALLOWED_HOSTS=(list, ["localhost"]),
)
environ.Env.read_env(".env")

SECRET_KEY      = env("SECRET_KEY")
DEBUG           = env("DEBUG")
DATABASE_URL    = env.db("DATABASE_URL")  # auto-parses postgres:// URL
REDIS_URL       = env("REDIS_URL")
STRIPE_SECRET   = env("STRIPE_SECRET_KEY", default=None)

DATABASES = { "default": DATABASE_URL }
```

### Secrets in Production

```
Never put secrets in:
- Code (even in a "test" branch)
- Docker images
- Container environment variables printed in logs

Use:
- AWS Secrets Manager / Parameter Store
- HashiCorp Vault
- GCP Secret Manager
- Azure Key Vault
- Kubernetes Secrets (with external-secrets-operator for rotation)
```

---

## 25. API Versioning

```javascript
// Strategy 1: URL versioning (most common, most explicit)
app.use("/api/v1", require("./routes/v1"));
app.use("/api/v2", require("./routes/v2"));

// Strategy 2: Header versioning
app.use((req, res, next) => {
  req.apiVersion = req.headers["api-version"] || "v1";
  next();
});
app.get("/users", (req, res) => {
  if (req.apiVersion === "v2") return userControllerV2.list(req, res);
  return userControllerV1.list(req, res);
});

// Strategy 3: Query parameter
// GET /users?version=2

// Best practices:
// 1. Never break existing clients — add, don't remove
// 2. Deprecate before removing — add Deprecation and Sunset headers
// 3. Maintain at least 1 old version during transition
res.set("Deprecation", "true");
res.set("Sunset", "Sat, 01 Jan 2026 00:00:00 GMT");
res.set("Link", '</api/v2/users>; rel="successor-version"');
```

---

## 26. CORS — Cross-Origin Resource Sharing

```javascript
const cors = require("cors");

// Permissive CORS (development only — never in production like this)
app.use(cors());

// Production CORS
const allowedOrigins = [
  "https://yourapp.com",
  "https://admin.yourapp.com",
  process.env.NODE_ENV === "development" && "http://localhost:3000",
].filter(Boolean);

app.use(cors({
  origin: (origin, callback) => {
    // Allow requests with no origin (mobile apps, curl, Postman)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error(`Origin ${origin} not allowed`));
    }
  },
  credentials: true,      // allow cookies to be sent
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
  allowedHeaders: ["Content-Type", "Authorization", "Idempotency-Key"],
  exposedHeaders: ["X-Total-Count", "X-Request-Id"],
  maxAge: 600,            // preflight cache: 10 minutes
}));
```

```python
# Django CORS with django-cors-headers
# settings.py
INSTALLED_APPS = ["corsheaders", ...]
MIDDLEWARE = ["corsheaders.middleware.CorsMiddleware", ...]

CORS_ALLOWED_ORIGINS = [
    "https://yourapp.com",
    "https://admin.yourapp.com",
]
CORS_ALLOW_CREDENTIALS  = True
CORS_ALLOW_METHODS      = ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
CORS_ALLOW_HEADERS      = ["Content-Type", "Authorization", "Idempotency-Key"]
CORS_EXPOSE_HEADERS     = ["X-Total-Count", "X-Request-Id"]
CORS_PREFLIGHT_MAX_AGE  = 600
```

---

## 27. Horizontal vs Vertical Scaling

```
Vertical Scaling (Scale Up):
  Add more CPU/RAM to the existing server.
  Simple, no code changes needed.
  Limit: You can only scale one machine so far. Single point of failure.

Horizontal Scaling (Scale Out):
  Add more servers. Put a load balancer in front.
  Requires: Stateless servers (no in-memory sessions), shared state in Redis/DB.
  Pros: Theoretically unlimited scale, no single point of failure.
  Cons: More complex, session sharing, distributed state.

For Node.js (single-threaded):
  Cluster module or PM2 — one process per CPU core:
```

```javascript
// pm2.config.js
module.exports = {
  apps: [{
    name:     "kickoff-api",
    script:   "./src/server.js",
    instances:"max",           // one per CPU
    exec_mode:"cluster",       // share the port
    env: { NODE_ENV: "production" },
    max_memory_restart: "500M",
  }]
};
// pm2 start pm2.config.js
```

```
Load Balancer strategies:
  Round Robin:   Each request goes to the next server in rotation
  Least Connections: Route to the server with fewest active connections (best for varying request times)
  IP Hash:       Same IP always goes to same server (useful for non-Redis sessions)
  Weighted:      More powerful servers get more traffic

Stateless requirement:
  - JWT instead of sessions (or sessions in Redis)
  - File uploads go to S3, not local disk
  - Logs go to central logging service
  - No in-memory caches (use Redis)
  - Scheduled jobs use locking (don't run on every instance)
```

---

## 28. Common Backend Interview Questions & Answers

**Q: What is the difference between REST and GraphQL?**

REST uses multiple endpoints (one per resource), each returning a fixed structure. GraphQL uses a single endpoint where the client specifies exactly what fields it wants. REST can over-fetch (returning more data than needed) or under-fetch (requiring multiple requests). GraphQL solves both at the cost of complexity on the server side (resolvers, DataLoader for N+1). REST is better for simple public APIs; GraphQL shines when you have many different clients with different data needs.

---

**Q: What is idempotency and why does it matter in APIs?**

An operation is idempotent if performing it multiple times produces the same result as performing it once. GET, PUT, and DELETE are idempotent by design. POST is not — hitting POST /orders twice creates two orders. This matters because networks are unreliable: a client might retry a request after a timeout, not knowing if the first request succeeded. Without idempotency, retries cause duplicate charges, duplicate records, or duplicate emails. The solution is idempotency keys — the client sends a unique key per logical operation, and the server stores the result so it can replay it on retry instead of re-executing.

---

**Q: How would you handle a race condition in a payment system?**

Three approaches, in increasing robustness: (1) Atomic DB operations — use `UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?` and check that one row was affected. (2) Database-level locking — `SELECT FOR UPDATE` inside a transaction to lock the row while processing. (3) Distributed lock using Redis Redlock when the operation spans multiple services. For payments specifically, I'd use atomic updates + idempotency keys together so retries are safe.

---

**Q: What's the difference between optimistic and pessimistic locking?**

Pessimistic locking assumes conflicts are likely: lock the resource before reading it (`SELECT FOR UPDATE`), preventing others from touching it until you commit. It's safe but reduces concurrency. Optimistic locking assumes conflicts are rare: read without locking, add a `version` column, and at update time check that the version hasn't changed — if it has, return a 409 Conflict and let the client retry. Pessimistic locking suits high-contention data like bank balances; optimistic locking suits lower-contention scenarios like form submissions.

---

**Q: Explain the N+1 query problem and how to fix it.**

When fetching a list of N items and then making one additional DB query per item to get related data, you end up with N+1 total queries. Example: fetch 20 posts (1 query), then fetch each post's author (20 queries) = 21 queries. Fix: eager loading — JOIN or pre-fetch related data in the initial query (`include` in Sequelize, `select_related` in Django, DataLoader in GraphQL). DataLoader is especially important in GraphQL because it batches all author loads triggered by individual resolvers into a single query.

---

**Q: What is ACID and why does it matter?**

ACID is the set of properties that guarantee database transactions are reliable. Atomicity: all steps in a transaction succeed or all are rolled back — no partial writes. Consistency: every transaction moves the database from one valid state to another. Isolation: concurrent transactions don't see each other's intermediate states. Durability: committed data survives crashes (written to disk before acknowledging). These properties are critical in financial systems, order processing, and any place where partial or duplicate writes would be catastrophic.

---

**Q: How does JWT authentication work and what are its tradeoffs?**

The server signs a payload (userId, role, expiry) with a secret key and returns the token to the client. On subsequent requests, the client sends the token in the Authorization header. The server verifies the signature — no DB lookup required. This makes JWTs fast and stateless (any server can verify them). The main tradeoff is that JWTs can't be invalidated before expiry. If a token is stolen or you want to force logout, you can't revoke it without maintaining a server-side blocklist — which reintroduces state and eliminates much of the stateless benefit. Mitigation: use very short expiry (15 minutes) with a refresh token pattern.

---

**Q: When would you use cursor pagination over offset pagination?**

Offset pagination (`LIMIT 20 OFFSET 100`) is simple but has two problems: performance degrades on large offsets (the DB must skip 100 rows before returning 20), and page drift (if a new item is inserted while the user is browsing, pages shift and the user sees duplicates or skips items). Cursor pagination uses the last item's ID/timestamp as a pointer to start the next page, solving both problems — performance is constant and pages are stable. Use cursor for feeds, infinite scroll, and large datasets. Use offset when you need "jump to page X" or the dataset is small.

---

**Q: How do you prevent SQL injection?**

Always use parameterized queries (prepared statements). Never concatenate user input into a query string. In Node.js with `pg`: `db.query("SELECT * FROM users WHERE email = $1", [email])`. In Django ORM: the ORM handles parameterization automatically — `User.objects.filter(email=email)` is safe. Raw queries in Django: `User.objects.raw("SELECT * FROM users WHERE email = %s", [email])` — always use `%s` placeholders, never f-strings.

---

**Q: What is the difference between 401 and 403?**

401 Unauthorized means the request lacks valid authentication credentials — "who are you?" (confusingly named, should be called "Unauthenticated"). A 401 tells the client to authenticate. 403 Forbidden means authentication succeeded but the authenticated user doesn't have permission — "I know who you are, but you can't do this." A 403 tells the client they won't get access regardless of credentials. As a real example: an unauthenticated request to DELETE /admin/users gets 401. A logged-in regular user trying the same gets 403.

---

**Q: How would you design a rate limiter?**

For a single server: use an in-memory sliding window. For multiple servers: use Redis so all instances share state. The key is `ratelimit:{userId or IP}`, stored as a sorted set of timestamps. On each request: remove entries older than the window, count remaining entries, if over the limit return 429 with a `Retry-After` header, otherwise add the current timestamp. For different tiers (free/pro/enterprise), use different limits keyed by the user's plan. For auth endpoints, rate limit by IP rather than userId since an attacker won't have a valid userId.

---

## 📖 Master Resource List

- [REST API Design Best Practices](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
- [GraphQL official docs](https://graphql.org/learn/)
- [gRPC official docs](https://grpc.io/docs/)
- [OAuth 2.0 RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Stripe — Idempotent Requests](https://stripe.com/docs/api/idempotent_requests)
- [Brandur — Idempotency Keys](https://brandur.org/idempotency-keys)
- [Redlock — Distributed Locks](https://redis.io/docs/manual/patterns/distributed-locks/)
- [PostgreSQL — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [Martin Kleppmann — Designing Data-Intensive Applications](https://dataintensive.net/)
- [High Scalability Blog](http://highscalability.com/)
- [ByteByteGo Newsletter](https://blog.bytebytego.com/) ← system design + backend depth

---

*You've got the backend mapped. Go build real things — and go land that interview. ⚙️*

> **Legend:** ✅ = correct | ❌ = avoid | ⚠️ = careful here
