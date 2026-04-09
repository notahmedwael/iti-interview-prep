# 🗄️ The Complete Database Guide — From ERD to Advanced SQL

> **Part 7 of Ahmed's shadow interview prep.**
> Written as a story — we build one real system (a football pitch booking platform) from a blank page to a fully indexed, secured, production database. Every concept builds on the last.
> Covers **PostgreSQL**, **MySQL**, and **MongoDB** — with honest tradeoffs between them.

---

## 📑 Table of Contents

1. [SQL vs NoSQL — The Fundamental Choice](#1-sql-vs-nosql--the-fundamental-choice)
2. [Entity-Relationship Diagrams (ERD)](#2-entity-relationship-diagrams-erd)
3. [Mapping ERD to a Schema](#3-mapping-erd-to-a-schema)
4. [Data Types Reference](#4-data-types-reference)
5. [Basic SQL — CRUD](#5-basic-sql--crud)
6. [Filtering, Sorting & Aggregation](#6-filtering-sorting--aggregation)
7. [JOINs — Connecting Tables](#7-joins--connecting-tables)
8. [Subqueries](#8-subqueries)
9. [Common Table Expressions (CTEs)](#9-common-table-expressions-ctes)
10. [Window Functions](#10-window-functions)
11. [Indexes & Their Types](#11-indexes--their-types)
12. [Views](#12-views)
13. [Stored Procedures & Functions](#13-stored-procedures--functions)
14. [Triggers](#14-triggers)
15. [Row-Level Security & Policies](#15-row-level-security--policies)
16. [Transactions & Isolation Levels](#16-transactions--isolation-levels)
17. [MongoDB — Document Model](#17-mongodb--document-model)
18. [Database Design Patterns & Anti-Patterns](#18-database-design-patterns--anti-patterns)
19. [Interview Questions & Answers](#19-interview-questions--answers)

---

## 1. SQL vs NoSQL — The Fundamental Choice

Before writing a single table, you make the most important architectural decision: **which type of database fits this problem?**

### The Mental Model

```
SQL (Relational):
  Data fits into neat rows and columns.
  Relationships between entities are well-defined and stable.
  You care deeply about data integrity — no orphaned records, no duplicates.
  Your queries are complex and varied — you don't know what you'll need to ask.

NoSQL (Non-Relational):
  Data is document-like, hierarchical, or graph-like.
  Schema changes frequently as product evolves.
  You're optimizing for one specific access pattern (read by userId, write at extreme scale).
  Horizontal scaling is a first-class requirement.
```

### Side-by-Side Comparison

| Dimension | SQL (PostgreSQL/MySQL) | NoSQL (MongoDB) |
|---|---|---|
| Data model | Tables with rows & columns | Documents (JSON/BSON), Key-Value, Column, Graph |
| Schema | Fixed — must define before inserting | Flexible — documents in same collection can differ |
| Relationships | JOINs, foreign keys, enforced by DB | Manual references, `$lookup`, or embedding |
| ACID | Full — by default | Partial — depends on configuration |
| Scaling | Vertical (scale up) + read replicas | Horizontal (shard) natively |
| Queries | SQL — expressive, ad-hoc | Query language (varies) — optimized for known patterns |
| Consistency | Strong | Tunable (eventual → strong) |
| Transactions | Multi-table, multi-row — native | Multi-document — supported since MongoDB 4.0 |
| Best for | Financial systems, ERP, anything with complex relations | Real-time feeds, catalogs, user profiles, time-series |

### The Honest Tradeoffs

```
SQL strengths:
  ✅ ACID guarantees out of the box — perfect for money, orders, medical records
  ✅ JOINs — answer any question without denormalizing in advance
  ✅ Constraints — DB enforces business rules (NOT NULL, UNIQUE, FK, CHECK)
  ✅ Mature tooling — decades of optimization, monitoring, backup tools
  ✅ Standard language — SQL transfers between systems

SQL weaknesses:
  ❌ Schema migrations on large tables are slow and painful
  ❌ Vertical scaling has a ceiling — one very expensive machine
  ❌ Object-relational impedance mismatch — code thinks in objects, DB thinks in rows
  ❌ Poor fit for hierarchical / graph data

NoSQL (MongoDB) strengths:
  ✅ Schema flexibility — ship fast, evolve as you learn
  ✅ Document = one round trip — no joins for common access patterns
  ✅ Horizontal sharding — scale to petabytes
  ✅ Great for high-write workloads (time-series, events, logs)
  ✅ Developer-friendly — documents look like your code objects

NoSQL weaknesses:
  ❌ No joins — denormalize everything, which causes update anomalies
  ❌ No schema enforcement by default — data quality is your problem
  ❌ Eventual consistency traps — stale reads without careful configuration
  ❌ "Flexible schema" becomes "chaos schema" without discipline
  ❌ Multi-document transactions are supported but add overhead
```

### When to Use What — Real Examples

```
Use PostgreSQL/MySQL when:
  - Banking, payments, e-commerce orders (ACID critical)
  - Reporting and analytics (complex ad-hoc SQL)
  - Data with stable, well-defined relationships
  - Regulatory compliance (financial, healthcare)
  → Our Kickoff booking platform: YES — bookings, payments, user accounts

Use MongoDB when:
  - Product catalogs (different products have wildly different attributes)
  - User activity feeds (write-heavy, read by userId)
  - Real-time IoT / sensor data
  - Content management (articles, comments — hierarchical)
  - Prototyping when schema is unknown
  → Our Kickoff platform: player stats logs, pitch metadata with dynamic attributes
```

### PostgreSQL vs MySQL — Key Differences

```
PostgreSQL:
  ✅ Full SQL standard compliance — window functions, CTEs, lateral joins
  ✅ Rich type system — arrays, JSONB, ranges, custom types, enums
  ✅ Row-level security — policy-based access control
  ✅ Better for complex queries and analytics
  ✅ MVCC — reads never block writes
  ✅ Extensions — PostGIS (geo), pg_vector (AI), TimescaleDB, etc.
  ⚠️ Slightly more complex to configure for very high write throughput

MySQL:
  ✅ Simpler to set up and manage — widely hosted
  ✅ Excellent read performance for simple queries
  ✅ InnoDB — ACID with good write performance
  ✅ Ubiquitous — every hosting platform supports it
  ⚠️ ENUM type is a column-level change — painful in production
  ⚠️ JSON support exists but less powerful than PostgreSQL's JSONB
  ⚠️ Some SQL standard features missing or quirky
  ⚠️ GROUP BY is more lenient (non-standard behavior without ONLY_FULL_GROUP_BY)

Choose PostgreSQL for new projects. Use MySQL when the team knows it or hosting requires it.
```

### 📖 Resources
- [PostgreSQL vs MySQL — Detailed comparison](https://www.postgresql.org/about/advantages/)
- [MongoDB — When to use NoSQL](https://www.mongodb.com/nosql-explained/nosql-vs-sql)
- [Designing Data-Intensive Applications — Martin Kleppmann](https://dataintensive.net/)

---

## 2. Entity-Relationship Diagrams (ERD)

An ERD is a **blueprint** of your database — drawn before writing a single line of SQL. It shows what data you store (entities), what properties each has (attributes), and how they relate (relationships).

### The Story Begins

We're building **Kickoff** — a platform where players can find football pitches, book time slots, form teams, and track stats.

Let's think about what real things exist in this world:

```
Things that exist in Kickoff:
  → Users   (players, pitch owners, admins)
  → Pitches (football grounds with location, capacity, surface type)
  → Bookings (a user books a pitch for a time slot)
  → Teams   (groups of users who play together)
  → Matches (a match happens at a booking)
  → Reviews (a user reviews a pitch after a match)
  → Payments (financial transaction for a booking)
```

### ERD Notation

```
Entity      = Rectangle  [User]
Attribute   = Oval       (name, email, created_at)
Relationship= Diamond    <makes>
Cardinality = lines on the relationship ends

Cardinality symbols:
  |    = exactly one (mandatory)
  O    = zero (optional)
  <    = many
  
  ||—|<  = one-to-many (mandatory on both ends)
  |O—O<  = one-to-many (optional)
  >|—|<  = many-to-many
```

### The Kickoff ERD (text representation)

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│     USER     │         │   BOOKING    │         │    PITCH     │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ PK user_id   │──────── │ PK booking_id│ ──────── │ PK pitch_id  │
│    name      │  1    * │ FK user_id   │ *     1  │    name      │
│    email     │         │ FK pitch_id  │         │    owner_id FK│
│    phone     │         │    start_time│         │    city      │
│    role      │         │    end_time  │         │    surface   │
│    created_at│         │    status    │         │    capacity  │
└──────────────┘         │    total_amt │         │    price_hr  │
        │                └──────────────┘         └──────────────┘
        │  1                    │ 1                      │
        │                       │                        │
        │                  ┌────┴──────┐                 │
        │  *               │  PAYMENT  │                 │
        │            *     ├───────────┤                 │
        │                  │ PK pay_id │                 │
┌───────┴──────┐           │ FK book_id│         ┌───────┴──────┐
│  TEAM_MEMBER │           │    amount │         │    REVIEW    │
├──────────────┤           │    method │         ├──────────────┤
│ FK user_id   │           │    status │         │ FK user_id   │
│ FK team_id   │           └───────────┘         │ FK pitch_id  │
│    role      │                                 │    rating    │
│    joined_at │                                 │    comment   │
└──────────────┘                                 │    created_at│
        │                                        └──────────────┘
        │
┌───────┴──────┐
│     TEAM     │
├──────────────┤
│ PK team_id   │
│    name      │
│    created_by│
│    created_at│
└──────────────┘

Relationships:
  User    ──<  Booking     (one user makes many bookings)
  Pitch   ──<  Booking     (one pitch has many bookings)
  Booking ──1  Payment     (one booking has one payment)
  User    ──<  Review      (one user writes many reviews)
  Pitch   ──<  Review      (one pitch has many reviews)
  User    >──< Team        (many users belong to many teams — junction: team_members)
  User    ──<  Pitch       (owner — one user owns many pitches)
```

### Types of Relationships

```
One-to-One (1:1):
  User ──── UserProfile
  Each user has exactly one profile. Each profile belongs to exactly one user.
  Implementation: FK on either table, or same PK.

One-to-Many (1:N):
  User ──< Booking
  One user makes many bookings. Each booking belongs to one user.
  Implementation: FK on the "many" side (booking.user_id).

Many-to-Many (N:M):
  User >──< Team
  A user can be in many teams. A team can have many users.
  Implementation: Junction/bridge table (team_members) with two FKs.
```

### Normalization — Eliminating Redundancy

```
1NF (First Normal Form):
  Every column holds atomic (indivisible) values.
  No repeating groups.
  ❌ BAD: booking_members = "user1,user2,user3" (comma-separated in one field)
  ✅ GOOD: separate team_members table with one row per member

2NF (Second Normal Form):
  Must be in 1NF.
  Every non-key attribute is fully dependent on the entire primary key.
  (Only matters for composite PKs — an attribute can't depend on just PART of the PK)
  ❌ BAD: team_member(team_id, user_id, team_name) — team_name depends only on team_id
  ✅ GOOD: Move team_name to teams table

3NF (Third Normal Form):
  Must be in 2NF.
  No transitive dependencies (non-key → non-key → non-key).
  ❌ BAD: booking(booking_id, pitch_id, city) — city depends on pitch_id, not booking_id
  ✅ GOOD: city belongs in pitches table, not bookings

BCNF (Boyce-Codd):
  Stricter version of 3NF — rarely needed in practice.

Denormalization (intentional):
  Sometimes you duplicate data FOR PERFORMANCE.
  booking.pitch_name — copying pitch name into booking so we don't need a JOIN
  for every booking list. Acceptable trade-off when reads dominate.
  Document this and make sure writes keep copies in sync.
```

---

## 3. Mapping ERD to a Schema

Now we translate the ERD into real SQL. This is where the design becomes concrete.

### The Full Kickoff Schema (PostgreSQL)

```sql
-- ============================================================
-- KICKOFF DATABASE SCHEMA
-- PostgreSQL — Production Grade
-- ============================================================

-- Enable UUID generation
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- ============================================================
-- USERS
-- ============================================================
CREATE TYPE user_role AS ENUM ('player', 'owner', 'admin');

CREATE TABLE users (
    user_id     UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    phone       VARCHAR(20),
    password_hash TEXT       NOT NULL,
    role        user_role   NOT NULL DEFAULT 'player',
    avatar_url  TEXT,
    is_active   BOOLEAN     NOT NULL DEFAULT true,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Constraints
    CONSTRAINT email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT phone_format CHECK (phone IS NULL OR phone ~ '^\+?[0-9]{7,15}$')
);

-- ============================================================
-- PITCHES
-- ============================================================
CREATE TYPE surface_type AS ENUM ('natural_grass', 'artificial_turf', 'futsal', 'beach');

CREATE TABLE pitches (
    pitch_id    UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id    UUID        NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
    name        VARCHAR(150) NOT NULL,
    description TEXT,
    city        VARCHAR(100) NOT NULL,
    address     TEXT        NOT NULL,
    latitude    DECIMAL(9,6),
    longitude   DECIMAL(9,6),
    surface     surface_type NOT NULL,
    capacity    SMALLINT    NOT NULL CHECK (capacity BETWEEN 5 AND 22),
    price_per_hour DECIMAL(10,2) NOT NULL CHECK (price_per_hour > 0),
    is_active   BOOLEAN     NOT NULL DEFAULT true,
    images      TEXT[],     -- PostgreSQL array of image URLs
    amenities   JSONB,      -- {"showers": true, "parking": true, "floodlights": true}
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- BOOKINGS
-- ============================================================
CREATE TYPE booking_status AS ENUM ('pending', 'confirmed', 'cancelled', 'completed', 'no_show');

CREATE TABLE bookings (
    booking_id  UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID        NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
    pitch_id    UUID        NOT NULL REFERENCES pitches(pitch_id) ON DELETE RESTRICT,
    start_time  TIMESTAMPTZ NOT NULL,
    end_time    TIMESTAMPTZ NOT NULL,
    status      booking_status NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10,2) NOT NULL CHECK (total_amount >= 0),
    notes       TEXT,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Business rule: end must be after start
    CONSTRAINT booking_time_valid CHECK (end_time > start_time),
    -- Business rule: bookings must be at least 1 hour
    CONSTRAINT booking_min_duration CHECK (end_time - start_time >= INTERVAL '1 hour'),
    -- Business rule: can't book in the past
    CONSTRAINT booking_not_in_past CHECK (start_time > NOW() - INTERVAL '1 minute')
);

-- Prevent double-booking on same pitch overlapping time — EXCLUDE constraint
-- This is PostgreSQL-specific and extremely powerful
ALTER TABLE bookings ADD CONSTRAINT no_overlap
    EXCLUDE USING gist (
        pitch_id WITH =,
        tstzrange(start_time, end_time) WITH &&
    )
    WHERE (status NOT IN ('cancelled'));
-- Requires: CREATE EXTENSION btree_gist;

-- ============================================================
-- PAYMENTS
-- ============================================================
CREATE TYPE payment_method  AS ENUM ('card', 'cash', 'wallet', 'bank_transfer');
CREATE TYPE payment_status  AS ENUM ('pending', 'completed', 'failed', 'refunded');

CREATE TABLE payments (
    payment_id      UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    booking_id      UUID        NOT NULL UNIQUE REFERENCES bookings(booking_id) ON DELETE RESTRICT,
    amount          DECIMAL(10,2) NOT NULL CHECK (amount > 0),
    method          payment_method NOT NULL,
    status          payment_status NOT NULL DEFAULT 'pending',
    transaction_ref VARCHAR(100) UNIQUE,  -- from payment gateway
    paid_at         TIMESTAMPTZ,
    refunded_at     TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ============================================================
-- TEAMS
-- ============================================================
CREATE TABLE teams (
    team_id     UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    name        VARCHAR(100) NOT NULL,
    description TEXT,
    created_by  UUID        NOT NULL REFERENCES users(user_id) ON DELETE RESTRICT,
    logo_url    TEXT,
    is_active   BOOLEAN     NOT NULL DEFAULT true,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TYPE team_role AS ENUM ('captain', 'member');

CREATE TABLE team_members (
    team_id     UUID        NOT NULL REFERENCES teams(team_id) ON DELETE CASCADE,
    user_id     UUID        NOT NULL REFERENCES users(user_id) ON DELETE CASCADE,
    role        team_role   NOT NULL DEFAULT 'member',
    joined_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    PRIMARY KEY (team_id, user_id)  -- composite primary key
);

-- ============================================================
-- REVIEWS
-- ============================================================
CREATE TABLE reviews (
    review_id   UUID        PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID        NOT NULL REFERENCES users(user_id) ON DELETE SET NULL,
    pitch_id    UUID        NOT NULL REFERENCES pitches(pitch_id) ON DELETE CASCADE,
    booking_id  UUID        UNIQUE REFERENCES bookings(booking_id),  -- one review per booking
    rating      SMALLINT    NOT NULL CHECK (rating BETWEEN 1 AND 5),
    comment     TEXT,
    is_visible  BOOLEAN     NOT NULL DEFAULT true,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- One review per user per pitch
    UNIQUE (user_id, pitch_id)
);
```

### The Same Schema in MySQL

```sql
-- MySQL differences:
-- No UUID type — use CHAR(36) or BINARY(16)
-- No TIMESTAMPTZ — use DATETIME + application timezone handling
-- No ENUM as separate type — defined inline on column
-- No EXCLUDE constraint — must handle with triggers or application logic
-- No arrays — use JSON or separate table
-- No JSONB — use JSON (not indexed, binary stored)

CREATE DATABASE kickoff CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE kickoff;

CREATE TABLE users (
    user_id     CHAR(36)    PRIMARY KEY DEFAULT (UUID()),
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    phone       VARCHAR(20),
    password_hash TEXT       NOT NULL,
    role        ENUM('player','owner','admin') NOT NULL DEFAULT 'player',
    avatar_url  TEXT,
    is_active   TINYINT(1)  NOT NULL DEFAULT 1,
    created_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_users_email (email),
    INDEX idx_users_role (role)
) ENGINE=InnoDB;

CREATE TABLE pitches (
    pitch_id    CHAR(36)    PRIMARY KEY DEFAULT (UUID()),
    owner_id    CHAR(36)    NOT NULL,
    name        VARCHAR(150) NOT NULL,
    city        VARCHAR(100) NOT NULL,
    address     TEXT        NOT NULL,
    surface     ENUM('natural_grass','artificial_turf','futsal','beach') NOT NULL,
    capacity    TINYINT     NOT NULL,
    price_per_hour DECIMAL(10,2) NOT NULL,
    is_active   TINYINT(1)  NOT NULL DEFAULT 1,
    amenities   JSON,       -- MySQL 5.7+ has JSON type (no indexing like JSONB)
    created_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (owner_id) REFERENCES users(user_id) ON DELETE RESTRICT,
    CHECK (capacity BETWEEN 5 AND 22),
    CHECK (price_per_hour > 0)
) ENGINE=InnoDB;

CREATE TABLE bookings (
    booking_id  CHAR(36)    PRIMARY KEY DEFAULT (UUID()),
    user_id     CHAR(36)    NOT NULL,
    pitch_id    CHAR(36)    NOT NULL,
    start_time  DATETIME    NOT NULL,
    end_time    DATETIME    NOT NULL,
    status      ENUM('pending','confirmed','cancelled','completed','no_show') NOT NULL DEFAULT 'pending',
    total_amount DECIMAL(10,2) NOT NULL,
    created_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id)  REFERENCES users(user_id)  ON DELETE RESTRICT,
    FOREIGN KEY (pitch_id) REFERENCES pitches(pitch_id) ON DELETE RESTRICT,
    CHECK (end_time > start_time),
    INDEX idx_bookings_pitch_time (pitch_id, start_time, end_time),
    INDEX idx_bookings_user (user_id),
    INDEX idx_bookings_status (status)
) ENGINE=InnoDB;
```

### Foreign Key Behaviors — ON DELETE / ON UPDATE

```sql
-- These determine what happens to child rows when the parent is deleted:

ON DELETE CASCADE     -- delete child rows automatically
  -- booking deleted → payment deleted
  -- team deleted    → team_members deleted

ON DELETE RESTRICT    -- prevent deletion if children exist (default)
  -- can't delete a user who has bookings

ON DELETE SET NULL    -- set FK column to NULL (column must allow NULL)
  -- delete user → review.user_id = NULL (review remains, anonymous)

ON DELETE NO ACTION   -- same as RESTRICT but deferred (checked at end of transaction)

-- Real decision for each FK in Kickoff:
-- users.user_id → bookings.user_id:   RESTRICT (don't lose booking history)
-- pitches.pitch_id → bookings:        RESTRICT (don't lose booking history)
-- bookings.booking_id → payments:     RESTRICT (financial record, never delete)
-- teams.team_id → team_members:       CASCADE  (team deleted → members gone too)
-- pitches.pitch_id → reviews:         CASCADE  (pitch deleted → reviews gone)
-- users.user_id → reviews:            SET NULL (user deleted → review stays, anonymous)
```

---

## 4. Data Types Reference

Choosing the right data type is a performance and correctness decision.

```sql
-- ============================================================
-- NUMBERS
-- ============================================================
SMALLINT        -- 2 bytes, -32768 to 32767       (capacity, rating, age)
INTEGER / INT   -- 4 bytes, ~2.1 billion           (counts, IDs in small tables)
BIGINT          -- 8 bytes, ~9.2 quintillion        (auto-increment IDs at scale)
DECIMAL(p,s)    -- exact fixed-point               (money — NEVER use FLOAT for money!)
NUMERIC(p,s)    -- same as DECIMAL
REAL            -- 4-byte float (approximate)      (coordinates, stats where precision ok)
DOUBLE PRECISION-- 8-byte float (approximate)

-- ❌ NEVER store money as FLOAT — floating point is approximate!
-- 0.1 + 0.2 = 0.30000000000000004 in floating point
-- ✅ Use DECIMAL(10,2) or store as INTEGER cents: 5000 = $50.00

-- ============================================================
-- TEXT
-- ============================================================
CHAR(n)         -- fixed length, padded with spaces (rare use: country codes, status codes)
VARCHAR(n)      -- variable length, max n chars     (names, emails)
TEXT            -- unlimited length                  (descriptions, comments, SQL)
-- PostgreSQL: TEXT is as efficient as VARCHAR — no performance difference
-- MySQL: TEXT is stored off-page above a threshold — slightly slower

-- ============================================================
-- DATES & TIMES
-- ============================================================
DATE            -- date only: 2024-01-15
TIME            -- time only: 14:30:00
TIMESTAMP       -- date + time, NO timezone (dangerous! stores local time)
TIMESTAMPTZ     -- date + time WITH timezone (PostgreSQL) — ALWAYS use this
                -- stored as UTC internally, displayed in session timezone
DATETIME        -- MySQL equivalent of TIMESTAMP (no timezone)
INTERVAL        -- duration: '2 hours', '1 day 3 hours', '3 months'

-- ✅ Always use TIMESTAMPTZ (PostgreSQL) or store UTC explicitly
-- ❌ TIMESTAMP without timezone causes "2am doesn't exist" bugs during DST

-- ============================================================
-- BOOLEANS
-- ============================================================
BOOLEAN         -- PostgreSQL: true/false
TINYINT(1)      -- MySQL: 0/1 (MySQL has no true BOOLEAN type)

-- ============================================================
-- IDENTIFIERS — UUIDs vs BIGINT SERIAL
-- ============================================================
-- Option 1: Auto-increment integer (fast, small, guessable)
BIGINT GENERATED ALWAYS AS IDENTITY  -- PostgreSQL modern syntax
BIGSERIAL                            -- PostgreSQL shorthand
AUTO_INCREMENT                       -- MySQL

-- Option 2: UUID (random, not guessable, globally unique, larger)
UUID            -- PostgreSQL native type
CHAR(36)        -- MySQL (stores as string "550e8400-e29b-41d4-a716-446655440000")
BINARY(16)      -- MySQL (stores as 16 bytes — faster but less readable)

-- UUID v4 = random (security) — use when IDs are exposed in URLs
-- UUID v7 = time-ordered random — indexes better than v4, still unpredictable
-- BIGSERIAL = fast, great for internal tables never exposed to users

-- PostgreSQL JSONB vs JSON:
JSONB           -- Binary JSON — indexed, parsed, faster queries, larger storage
JSON            -- Text JSON — stored as-is, reparsed each query, preserves formatting

-- ✅ Always use JSONB in PostgreSQL when you need JSON
-- amenities JSONB -- you can do: amenities->>'showers' = 'true' and INDEX it

-- PostgreSQL arrays:
images TEXT[]   -- array of strings
-- INSERT: ARRAY['url1.jpg', 'url2.jpg']  or  '{"url1.jpg","url2.jpg"}'
-- Query: WHERE 'url1.jpg' = ANY(images)
```

---

## 5. Basic SQL — CRUD

Now the story continues. Our schema is ready. Let's populate it and query it.

### INSERT — Adding Data

```sql
-- Insert a single user
INSERT INTO users (name, email, phone, role, password_hash)
VALUES ('Ahmed Ali', 'ahmed@kickoff.dev', '+201012345678', 'player',
        '$2b$12$hashed_password_here');

-- Insert with RETURNING (PostgreSQL) — get the generated ID back immediately
INSERT INTO users (name, email, role, password_hash)
VALUES ('Sara Hassan', 'sara@kickoff.dev', 'owner', '$2b$12$hash')
RETURNING user_id, name, created_at;

-- Insert multiple rows at once (more efficient than N separate inserts)
INSERT INTO pitches (owner_id, name, city, address, surface, capacity, price_per_hour)
VALUES
    ('uuid-of-sara', 'Cairo Sports Club Field A', 'Cairo', '5 Tahrir St', 'artificial_turf', 14, 200.00),
    ('uuid-of-sara', 'Cairo Sports Club Field B', 'Cairo', '5 Tahrir St', 'natural_grass',   22, 300.00),
    ('uuid-of-sara', 'Zamalek Mini Pitch',         'Cairo', '12 Nile Corniche', 'futsal',    10, 150.00);

-- Insert or update (UPSERT) — if duplicate, update instead
-- PostgreSQL:
INSERT INTO pitches (pitch_id, name, price_per_hour)
VALUES ('existing-uuid', 'Updated Name', 250.00)
ON CONFLICT (pitch_id) DO UPDATE
    SET name = EXCLUDED.name,
        price_per_hour = EXCLUDED.price_per_hour,
        updated_at = NOW();

-- MySQL:
INSERT INTO pitches (pitch_id, name, price_per_hour)
VALUES ('existing-uuid', 'Updated Name', 250.00)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    price_per_hour = VALUES(price_per_hour),
    updated_at = NOW();
```

### SELECT — Reading Data

```sql
-- Get all active pitches in Cairo
SELECT * FROM pitches
WHERE city = 'Cairo' AND is_active = true;

-- Select specific columns (NEVER use SELECT * in production)
SELECT
    pitch_id,
    name,
    surface,
    capacity,
    price_per_hour
FROM pitches
WHERE city = 'Cairo' AND is_active = true
ORDER BY price_per_hour ASC
LIMIT 10 OFFSET 0;  -- page 1

-- OFFSET-based pagination
-- Page 1: LIMIT 10 OFFSET 0
-- Page 2: LIMIT 10 OFFSET 10
-- Page n: LIMIT 10 OFFSET (n-1)*10

-- Aliases — rename for readability
SELECT
    p.name          AS pitch_name,
    p.price_per_hour AS hourly_rate,
    p.capacity      AS max_players
FROM pitches p      -- table alias 'p'
WHERE p.surface = 'artificial_turf';

-- DISTINCT — deduplicate results
SELECT DISTINCT city FROM pitches WHERE is_active = true ORDER BY city;

-- Conditional expressions
SELECT
    name,
    price_per_hour,
    CASE
        WHEN price_per_hour < 150 THEN 'budget'
        WHEN price_per_hour < 300 THEN 'mid-range'
        ELSE 'premium'
    END AS price_tier,
    CASE surface
        WHEN 'natural_grass'   THEN '🌿 Natural Grass'
        WHEN 'artificial_turf' THEN '🟩 Artificial Turf'
        WHEN 'futsal'          THEN '🏟️ Futsal'
        WHEN 'beach'           THEN '🏖️ Beach'
    END AS surface_display
FROM pitches
WHERE is_active = true;

-- COALESCE — return first non-null value
SELECT
    name,
    COALESCE(phone, 'No phone provided') AS contact_phone,
    COALESCE(avatar_url, '/default-avatar.png') AS avatar
FROM users;

-- NULLIF — return null if values are equal (avoids division by zero)
SELECT
    pitch_id,
    total_revenue,
    booking_count,
    total_revenue / NULLIF(booking_count, 0) AS avg_revenue_per_booking
FROM pitch_stats;
```

### UPDATE — Modifying Data

```sql
-- Update a single field
UPDATE pitches
SET price_per_hour = 250.00, updated_at = NOW()
WHERE pitch_id = 'abc-123-uuid';

-- Update with calculation
UPDATE bookings
SET total_amount = (
    EXTRACT(EPOCH FROM (end_time - start_time)) / 3600  -- hours
    * p.price_per_hour
)
FROM pitches p
WHERE bookings.pitch_id = p.pitch_id
  AND bookings.booking_id = 'booking-uuid';

-- Conditional update (CASE in SET clause)
UPDATE users
SET role = CASE
    WHEN role = 'player' AND conditions THEN 'owner'
    ELSE role
END
WHERE user_id = 'some-uuid';

-- Update multiple columns atomically
UPDATE bookings
SET
    status     = 'confirmed',
    updated_at = NOW()
WHERE booking_id = 'booking-uuid'
  AND status = 'pending'  -- optimistic locking!
RETURNING *;              -- see what changed (PostgreSQL)
```

### DELETE — Removing Data

```sql
-- Soft delete — MUCH safer than actual DELETE
-- Never truly delete bookings, payments, or user-generated content
UPDATE users
SET is_active = false, updated_at = NOW()
WHERE user_id = 'some-uuid';

-- Hard delete — use for truly temporary data
DELETE FROM sessions
WHERE expires_at < NOW();

-- Delete with JOIN (MySQL)
DELETE b FROM bookings b
JOIN pitches p ON b.pitch_id = p.pitch_id
WHERE p.owner_id = 'some-uuid' AND b.status = 'pending';

-- Delete with subquery (PostgreSQL / MySQL)
DELETE FROM bookings
WHERE pitch_id IN (
    SELECT pitch_id FROM pitches WHERE owner_id = 'some-uuid'
)
AND status = 'pending';

-- TRUNCATE — delete ALL rows extremely fast (no logging per row)
TRUNCATE TABLE sessions;          -- PostgreSQL: also resets sequences
TRUNCATE TABLE sessions;          -- MySQL: same
-- ⚠️ TRUNCATE cannot be rolled back in MySQL (DDL statement)
-- ✅ PostgreSQL TRUNCATE can be rolled back (inside a transaction)
```

---

## 6. Filtering, Sorting & Aggregation

### WHERE Clause — The Power is in the Conditions

```sql
-- Comparison operators
WHERE price_per_hour = 200           -- equal
WHERE price_per_hour != 200          -- not equal
WHERE price_per_hour <> 200          -- also not equal
WHERE price_per_hour > 100
WHERE price_per_hour BETWEEN 100 AND 300  -- inclusive
WHERE capacity IN (5, 7, 10, 11)    -- any of these values
WHERE capacity NOT IN (22)

-- String matching — LIKE
WHERE name LIKE 'Cairo%'     -- starts with 'Cairo'
WHERE name LIKE '%Club%'     -- contains 'Club'
WHERE name LIKE '%Field_'    -- _ matches exactly one character
-- ⚠️ LIKE is case-sensitive in PostgreSQL, case-insensitive in MySQL
-- PostgreSQL case-insensitive:
WHERE name ILIKE '%cairo%'   -- PostgreSQL: case-insensitive LIKE
WHERE LOWER(name) LIKE '%cairo%'  -- works in both

-- NULL checks — NEVER use = NULL or != NULL
WHERE phone IS NULL
WHERE phone IS NOT NULL
-- ❌ WHERE phone = NULL  -- always returns 0 rows (NULL = anything is NULL, not true)

-- Logical operators
WHERE city = 'Cairo' AND is_active = true
WHERE city = 'Cairo' OR city = 'Alexandria'
WHERE NOT (status = 'cancelled')

-- Complex conditions — use parentheses!
WHERE (city = 'Cairo' OR city = 'Giza')
  AND surface = 'artificial_turf'
  AND price_per_hour BETWEEN 150 AND 350

-- Pattern matching with regex (PostgreSQL)
WHERE email ~ '^[a-zA-Z0-9._%+-]+@gmail\.com$'  -- email is a gmail address
WHERE phone ~ '^\+20'                             -- Egyptian phone number

-- Check JSONB field (PostgreSQL)
WHERE amenities->>'showers' = 'true'
WHERE amenities @> '{"parking": true}'  -- amenities contains this JSON

-- Array operators (PostgreSQL)
WHERE 'artificial_turf' = ANY(allowed_surfaces)  -- value in array column
WHERE ARRAY['pool','gym'] && amenity_list         -- any overlap (&&)
WHERE ARRAY['pool','gym'] <@ amenity_list         -- left is subset of right
```

### Aggregation — Summarizing Data

```sql
-- Core aggregate functions
SELECT COUNT(*) FROM bookings;                    -- count all rows (includes NULLs)
SELECT COUNT(phone) FROM users;                   -- count non-NULL phone values
SELECT COUNT(DISTINCT city) FROM pitches;         -- count unique cities

SELECT SUM(total_amount) FROM bookings
WHERE status = 'completed';                        -- total revenue

SELECT AVG(rating) FROM reviews WHERE pitch_id = 'uuid';  -- average rating

SELECT
    MIN(price_per_hour) AS cheapest,
    MAX(price_per_hour) AS most_expensive,
    AVG(price_per_hour) AS average_price,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price_per_hour) AS median_price
FROM pitches WHERE is_active = true;

-- GROUP BY — aggregate per group
SELECT
    city,
    surface,
    COUNT(*)                      AS pitch_count,
    AVG(price_per_hour)           AS avg_price,
    SUM(CASE WHEN is_active THEN 1 ELSE 0 END) AS active_count
FROM pitches
GROUP BY city, surface    -- group by BOTH columns
ORDER BY city, avg_price DESC;

-- HAVING — filter AFTER grouping (WHERE filters BEFORE grouping)
SELECT
    pitch_id,
    COUNT(*) AS booking_count,
    SUM(total_amount) AS total_revenue
FROM bookings
WHERE status = 'completed'        -- WHERE: filter rows before grouping
GROUP BY pitch_id
HAVING COUNT(*) >= 10             -- HAVING: filter groups (after aggregation)
   AND SUM(total_amount) > 5000   -- only pitches with 10+ completed bookings & >5000 revenue
ORDER BY total_revenue DESC;

-- Combining GROUP BY with a range
SELECT
    DATE_TRUNC('month', created_at) AS month,  -- truncate to month
    COUNT(*)                         AS new_users
FROM users
WHERE created_at >= NOW() - INTERVAL '12 months'
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;

-- GROUP BY with ROLLUP — subtotals and grand total (PostgreSQL/MySQL)
SELECT
    city,
    surface,
    COUNT(*) AS count
FROM pitches
GROUP BY ROLLUP(city, surface);
-- Gives: per city+surface, per city (subtotal), grand total
```

### Date & Time Operations

```sql
-- Current timestamps
NOW()                           -- current date + time with timezone (PostgreSQL)
CURRENT_TIMESTAMP               -- same as NOW()
CURRENT_DATE                    -- current date only

-- Extracting parts
EXTRACT(YEAR  FROM created_at)  -- 2024
EXTRACT(MONTH FROM created_at)  -- 1-12
EXTRACT(DOW   FROM created_at)  -- 0=Sunday, 6=Saturday
EXTRACT(HOUR  FROM start_time)  -- 0-23
EXTRACT(EPOCH FROM (end_time - start_time))  -- total seconds

-- Date arithmetic
NOW() + INTERVAL '7 days'
NOW() - INTERVAL '1 month'
start_time + INTERVAL '2 hours'
AGE(created_at)                 -- PostgreSQL: '3 years 2 months 15 days'
DATEDIFF('2024-12-31', '2024-01-01')  -- MySQL: days between

-- Truncation
DATE_TRUNC('day',   created_at)   -- PostgreSQL: round down to start of day
DATE_TRUNC('week',  created_at)   -- round down to start of week
DATE_TRUNC('month', created_at)   -- round down to start of month
DATE(created_at)                  -- both: extract date part

-- Format
TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI')  -- PostgreSQL
DATE_FORMAT(created_at, '%Y-%m-%d %H:%i')  -- MySQL

-- Find bookings in the next 24 hours
SELECT * FROM bookings
WHERE start_time BETWEEN NOW() AND NOW() + INTERVAL '24 hours'
  AND status = 'confirmed';

-- Find pitches available at a specific time (NOT booked)
SELECT p.*
FROM pitches p
WHERE p.is_active = true
  AND p.pitch_id NOT IN (
    SELECT b.pitch_id FROM bookings b
    WHERE b.status NOT IN ('cancelled')
      AND b.start_time < '2024-06-15 20:00'   -- requested end
      AND b.end_time   > '2024-06-15 18:00'   -- requested start
  );
```

---

## 7. JOINs — Connecting Tables

JOINs are the heart of relational databases. They combine data from multiple tables based on a related column.

```
INNER JOIN: only rows where the join condition matches in BOTH tables
LEFT JOIN:  all rows from left table + matching rows from right (NULLs where no match)
RIGHT JOIN: all rows from right table + matching from left (use LEFT JOIN and swap tables instead)
FULL OUTER JOIN: all rows from both tables (NULLs where no match on either side)
CROSS JOIN: cartesian product — every row in left × every row in right
SELF JOIN:  join a table to itself
```

### Visual Guide to JOINs

```
Users Table:      Bookings Table:
user_id | name    booking_id | user_id
1       | Ahmed   B1         | 1
2       | Sara    B2         | 1
3       | Omar    B3         | 2
                  B4         | 99 (orphan — user doesn't exist)

INNER JOIN (users.user_id = bookings.user_id):
  Ahmed → B1, B2    (2 rows)
  Sara  → B3        (1 row)
  Omar  → (none)    ← not included — Omar has no bookings

LEFT JOIN:
  Ahmed → B1, B2    (2 rows)
  Sara  → B3        (1 row)
  Omar  → NULL      ← included with NULL booking columns

FULL OUTER JOIN:
  Ahmed → B1, B2
  Sara  → B3
  Omar  → NULL
  NULL  → B4        ← orphan booking included with NULL user columns
```

### JOIN Examples — The Kickoff Story

```sql
-- INNER JOIN: Get all bookings with user and pitch info
SELECT
    b.booking_id,
    u.name          AS player_name,
    u.email         AS player_email,
    p.name          AS pitch_name,
    p.city,
    b.start_time,
    b.end_time,
    b.status,
    b.total_amount
FROM bookings b
INNER JOIN users   u ON b.user_id  = u.user_id
INNER JOIN pitches p ON b.pitch_id = p.pitch_id
WHERE b.status = 'confirmed'
ORDER BY b.start_time;

-- LEFT JOIN: All pitches, including ones with no bookings yet
SELECT
    p.pitch_id,
    p.name,
    p.city,
    COUNT(b.booking_id)     AS total_bookings,
    COALESCE(SUM(b.total_amount), 0) AS total_revenue
FROM pitches p
LEFT JOIN bookings b
    ON p.pitch_id  = b.pitch_id
    AND b.status   = 'completed'     -- join condition — filter on RIGHT side
WHERE p.is_active = true
GROUP BY p.pitch_id, p.name, p.city
ORDER BY total_revenue DESC;

-- ⚠️ IMPORTANT: Filtering the RIGHT table's columns in WHERE vs ON
-- WHERE b.status = 'completed'  -- filters AFTER join — turns LEFT JOIN into INNER JOIN!
-- ON ... AND b.status = 'completed' -- filters DURING join — keeps all left rows

-- Multiple LEFT JOINs: Full pitch report
SELECT
    p.name                          AS pitch_name,
    p.city,
    p.price_per_hour,
    COUNT(DISTINCT b.booking_id)    AS booking_count,
    ROUND(AVG(r.rating), 1)         AS avg_rating,
    COUNT(DISTINCT r.review_id)     AS review_count,
    COALESCE(SUM(b.total_amount), 0) AS lifetime_revenue
FROM pitches p
LEFT JOIN bookings b ON p.pitch_id = b.pitch_id AND b.status = 'completed'
LEFT JOIN reviews  r ON p.pitch_id = r.pitch_id AND r.is_visible = true
WHERE p.is_active = true
GROUP BY p.pitch_id, p.name, p.city, p.price_per_hour
ORDER BY avg_rating DESC NULLS LAST, booking_count DESC;

-- SELF JOIN: Find users who booked the same pitch on the same day
-- (potential teammates who don't know each other)
SELECT DISTINCT
    u1.name AS player_1,
    u2.name AS player_2,
    p.name  AS pitch,
    DATE(b1.start_time) AS match_date
FROM bookings b1
JOIN bookings b2
    ON  b1.pitch_id = b2.pitch_id
    AND DATE(b1.start_time) = DATE(b2.start_time)
    AND b1.user_id < b2.user_id  -- avoid duplicates (A,B and B,A)
JOIN users u1 ON b1.user_id = u1.user_id
JOIN users u2 ON b2.user_id = u2.user_id
JOIN pitches p ON b1.pitch_id = p.pitch_id
WHERE b1.status = 'completed'
  AND b2.status = 'completed';

-- FULL OUTER JOIN: Find orphaned records (data integrity check)
SELECT
    u.user_id,
    u.name,
    b.booking_id,
    b.status
FROM users u
FULL OUTER JOIN bookings b ON u.user_id = b.user_id
WHERE u.user_id IS NULL   -- bookings with no user (orphaned)
   OR b.booking_id IS NULL; -- users with no bookings (fine, but shows the join)

-- CROSS JOIN: Generate a price matrix (all pitches × all time slots)
WITH time_slots AS (
    SELECT generate_series(8, 22) AS hour  -- PostgreSQL: generates 8,9,...,22
)
SELECT
    p.name AS pitch,
    ts.hour,
    p.price_per_hour AS rate,
    CASE WHEN ts.hour BETWEEN 18 AND 22 THEN p.price_per_hour * 1.2
         ELSE p.price_per_hour
    END AS peak_adjusted_rate
FROM pitches p
CROSS JOIN time_slots ts
WHERE p.is_active = true
ORDER BY p.name, ts.hour;

-- LATERAL JOIN (PostgreSQL) — like a correlated subquery but can return multiple rows
-- "For each pitch, get its 3 most recent reviews"
SELECT
    p.name,
    r.rating,
    r.comment,
    r.created_at
FROM pitches p
JOIN LATERAL (
    SELECT * FROM reviews
    WHERE pitch_id = p.pitch_id
    ORDER BY created_at DESC
    LIMIT 3
) r ON true;

-- MySQL equivalent using subquery (MySQL 8+ supports LATERAL)
```

---

## 8. Subqueries

A subquery is a query nested inside another query. They let you use the result of one query as input to another.

### Types of Subqueries

```sql
-- 1. Scalar subquery — returns exactly one value
SELECT
    b.*,
    (SELECT name FROM users WHERE user_id = b.user_id) AS player_name,
    (SELECT name FROM pitches WHERE pitch_id = b.pitch_id) AS pitch_name
FROM bookings b
WHERE b.booking_id = 'some-uuid';
-- ⚠️ Scalar subquery in SELECT runs once per row — avoid for large results
-- ✅ Better: use JOIN for this case

-- 2. Row subquery — returns one row with multiple columns
SELECT * FROM bookings
WHERE (user_id, pitch_id) = (
    SELECT user_id, pitch_id
    FROM bookings
    WHERE booking_id = 'reference-uuid'
);

-- 3. Table subquery (derived table) — used in FROM clause
SELECT
    city_stats.city,
    city_stats.pitch_count,
    city_stats.avg_price
FROM (
    SELECT
        city,
        COUNT(*)          AS pitch_count,
        AVG(price_per_hour) AS avg_price
    FROM pitches
    WHERE is_active = true
    GROUP BY city
) AS city_stats              -- MUST give alias to derived table
WHERE city_stats.pitch_count >= 3
ORDER BY city_stats.avg_price;

-- 4. EXISTS subquery — check if any matching row exists (returns true/false)
-- Find users who have made at least one booking
SELECT u.name, u.email
FROM users u
WHERE EXISTS (
    SELECT 1 FROM bookings b   -- SELECT 1 — we don't care about value, just existence
    WHERE b.user_id = u.user_id
      AND b.status  = 'completed'
);

-- NOT EXISTS — users who have never booked
SELECT u.name, u.email
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM bookings b
    WHERE b.user_id = u.user_id
);
-- ✅ EXISTS is often faster than IN for large result sets because it short-circuits

-- 5. IN / NOT IN subquery
-- Pitches booked in the last 7 days
SELECT * FROM pitches
WHERE pitch_id IN (
    SELECT DISTINCT pitch_id FROM bookings
    WHERE start_time >= NOW() - INTERVAL '7 days'
      AND status != 'cancelled'
);

-- ⚠️ NOT IN with NULLs is dangerous!
-- If subquery returns any NULL, NOT IN returns NOTHING (not what you expect!)
-- Use NOT EXISTS instead for safety:
SELECT * FROM pitches p
WHERE NOT EXISTS (
    SELECT 1 FROM bookings b
    WHERE b.pitch_id = p.pitch_id
      AND b.start_time >= NOW() - INTERVAL '7 days'
);

-- 6. Correlated subquery — references the outer query (runs once per outer row)
-- Find each user's most expensive booking
SELECT
    u.name,
    b.booking_id,
    b.total_amount,
    b.start_time
FROM users u
JOIN bookings b ON u.user_id = b.user_id
WHERE b.total_amount = (
    SELECT MAX(b2.total_amount)   -- references outer b via u.user_id
    FROM bookings b2
    WHERE b2.user_id = u.user_id  -- ← correlated: uses outer query's user_id
)
ORDER BY b.total_amount DESC;

-- 7. ANY / ALL
-- Pitches cheaper than any pitch in Cairo
SELECT name, price_per_hour FROM pitches
WHERE price_per_hour < ANY (
    SELECT price_per_hour FROM pitches WHERE city = 'Cairo'
);
-- < ANY = < at least one value = less than the maximum

-- Pitches cheaper than ALL pitches in Cairo
SELECT name, price_per_hour FROM pitches
WHERE price_per_hour < ALL (
    SELECT price_per_hour FROM pitches WHERE city = 'Cairo'
);
-- < ALL = less than every value = less than the minimum

-- Real business query: Find pitches with above-average rating in their city
SELECT
    p.name,
    p.city,
    ROUND(AVG(r.rating), 2) AS avg_rating
FROM pitches p
JOIN reviews r ON p.pitch_id = r.pitch_id
GROUP BY p.pitch_id, p.name, p.city
HAVING AVG(r.rating) > (
    SELECT AVG(r2.rating)
    FROM reviews r2
    JOIN pitches p2 ON r2.pitch_id = p2.pitch_id
    WHERE p2.city = p.city  -- same city — correlated
);
```

---

## 9. Common Table Expressions (CTEs)

CTEs are **named subqueries** defined at the top of a query with `WITH`. They make complex queries readable and can be referenced multiple times.

```sql
-- Basic CTE — same as a subquery but readable
WITH completed_bookings AS (
    SELECT
        b.*,
        u.name  AS player_name,
        p.name  AS pitch_name,
        p.city
    FROM bookings b
    JOIN users   u ON b.user_id  = u.user_id
    JOIN pitches p ON b.pitch_id = p.pitch_id
    WHERE b.status = 'completed'
),
revenue_by_pitch AS (
    SELECT
        pitch_id,
        pitch_name,
        city,
        COUNT(*)          AS booking_count,
        SUM(total_amount) AS total_revenue,
        AVG(total_amount) AS avg_booking_value
    FROM completed_bookings
    GROUP BY pitch_id, pitch_name, city
)
SELECT
    *,
    ROUND(total_revenue / NULLIF(booking_count, 0), 2) AS revenue_per_booking
FROM revenue_by_pitch
WHERE total_revenue > 10000
ORDER BY total_revenue DESC;

-- Multiple CTEs — reference earlier ones in later ones
WITH
monthly_revenue AS (
    SELECT
        DATE_TRUNC('month', b.created_at) AS month,
        SUM(b.total_amount)               AS revenue
    FROM bookings b
    WHERE b.status = 'completed'
    GROUP BY 1
),
revenue_with_growth AS (
    SELECT
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue
    FROM monthly_revenue
)
SELECT
    TO_CHAR(month, 'Month YYYY') AS period,
    revenue,
    prev_month_revenue,
    ROUND(
        (revenue - prev_month_revenue) / NULLIF(prev_month_revenue, 0) * 100,
        1
    ) AS growth_pct
FROM revenue_with_growth
ORDER BY month;

-- Recursive CTE — queries that reference themselves
-- Use case: organizational hierarchies, bill of materials, graph traversal

-- Example: Find all team members and their "level" in a hierarchy
-- Imagine teams can have sub-teams (a tree structure)
WITH RECURSIVE team_hierarchy AS (
    -- Anchor: start from root teams (no parent)
    SELECT
        team_id,
        name,
        parent_team_id,
        0 AS depth,
        name::TEXT AS path
    FROM teams
    WHERE parent_team_id IS NULL

    UNION ALL

    -- Recursive: join children to their parents
    SELECT
        t.team_id,
        t.name,
        t.parent_team_id,
        h.depth + 1,
        (h.path || ' > ' || t.name)::TEXT
    FROM teams t
    JOIN team_hierarchy h ON t.parent_team_id = h.team_id
    WHERE h.depth < 10  -- safety: prevent infinite recursion
)
SELECT
    REPEAT('  ', depth) || name AS indented_name,  -- indent by depth
    depth,
    path
FROM team_hierarchy
ORDER BY path;

-- Recursive CTE: Find all reachable pitches from a starting city
-- (imagine a transport network)
WITH RECURSIVE reachable AS (
    SELECT city_from, city_to, 1 AS hops, ARRAY[city_from, city_to] AS visited
    FROM city_connections
    WHERE city_from = 'Cairo'

    UNION ALL

    SELECT c.city_from, c.city_to, r.hops + 1, r.visited || c.city_to
    FROM city_connections c
    JOIN reachable r ON c.city_from = r.city_to
    WHERE c.city_to != ALL(r.visited)  -- no cycles
      AND r.hops < 5
)
SELECT DISTINCT city_to AS reachable_city, MIN(hops) AS min_hops
FROM reachable
GROUP BY city_to;
```

---

## 10. Window Functions

Window functions perform calculations **across a set of related rows without collapsing them into a group**. They're one of the most powerful SQL features.

```
Normal aggregate: GROUP BY → collapses N rows into 1
Window function:  OVER()   → adds a computed column, keeps all rows

Think of it like: "calculate something across a window of rows, but keep every row"
```

### The OVER() Clause

```sql
function_name() OVER (
    PARTITION BY column  -- divide into groups (like GROUP BY but doesn't collapse)
    ORDER BY   column    -- order within each partition
    ROWS/RANGE BETWEEN start AND end  -- define the window frame
)
```

### Ranking Functions

```sql
-- ROW_NUMBER: unique sequential number per partition
-- RANK: same value = same rank, leaves gaps
-- DENSE_RANK: same value = same rank, NO gaps
-- NTILE(n): divide into n equal buckets

SELECT
    name,
    city,
    price_per_hour,
    ROW_NUMBER()   OVER (PARTITION BY city ORDER BY price_per_hour)       AS rn,
    RANK()         OVER (PARTITION BY city ORDER BY price_per_hour)       AS rnk,
    DENSE_RANK()   OVER (PARTITION BY city ORDER BY price_per_hour)       AS dense_rnk,
    NTILE(4)       OVER (PARTITION BY city ORDER BY price_per_hour DESC)  AS quartile
FROM pitches
WHERE is_active = true
ORDER BY city, price_per_hour;

-- Practical use: Get the top-rated pitch in each city
WITH ranked_pitches AS (
    SELECT
        p.pitch_id,
        p.name,
        p.city,
        AVG(r.rating) AS avg_rating,
        COUNT(r.review_id) AS review_count,
        ROW_NUMBER() OVER (
            PARTITION BY p.city
            ORDER BY AVG(r.rating) DESC, COUNT(r.review_id) DESC
        ) AS rank_in_city
    FROM pitches p
    JOIN reviews r ON p.pitch_id = r.pitch_id
    WHERE p.is_active = true AND r.is_visible = true
    GROUP BY p.pitch_id, p.name, p.city
)
SELECT * FROM ranked_pitches WHERE rank_in_city = 1;
-- ✅ This is the canonical "top N per group" pattern
```

### Offset Functions — Look Back or Forward

```sql
-- LAG: access value from a previous row
-- LEAD: access value from a following row

SELECT
    DATE_TRUNC('month', b.created_at)            AS month,
    COUNT(*)                                      AS bookings,
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY DATE_TRUNC('month', b.created_at))
                                                  AS bookings_vs_prev_month,
    LEAD(COUNT(*)) OVER (ORDER BY DATE_TRUNC('month', b.created_at))
                                                  AS next_month_forecast
FROM bookings b
WHERE b.status = 'completed'
GROUP BY DATE_TRUNC('month', b.created_at)
ORDER BY month;

-- LAG with offset and default
LAG(revenue, 1, 0) OVER (ORDER BY month)  -- look back 1 row, default 0 if none

-- First and last values in a window
SELECT
    user_id,
    booking_id,
    start_time,
    total_amount,
    FIRST_VALUE(total_amount) OVER (PARTITION BY user_id ORDER BY start_time)
        AS first_booking_amount,
    LAST_VALUE(total_amount) OVER (
        PARTITION BY user_id
        ORDER BY start_time
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_booking_amount    -- ⚠️ LAST_VALUE needs explicit frame!
FROM bookings
WHERE status = 'completed';
```

### Aggregate Window Functions — Running Totals & Moving Averages

```sql
-- Running total of revenue over time
SELECT
    b.booking_id,
    u.name AS player,
    b.start_time,
    b.total_amount,
    SUM(b.total_amount) OVER (
        ORDER BY b.start_time
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- running sum
    ) AS running_revenue,
    AVG(b.total_amount) OVER (
        ORDER BY b.start_time
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW  -- 7-row moving average
    ) AS moving_avg_7
FROM bookings b
JOIN users u ON b.user_id = u.user_id
WHERE b.status = 'completed'
ORDER BY b.start_time;

-- Running total per user — using PARTITION BY
SELECT
    user_id,
    booking_id,
    start_time,
    total_amount,
    SUM(total_amount) OVER (
        PARTITION BY user_id     -- reset for each user
        ORDER BY start_time
    ) AS user_running_total,
    COUNT(*) OVER (
        PARTITION BY user_id
    ) AS user_total_bookings    -- total count (no ORDER BY = entire partition)
FROM bookings
WHERE status = 'completed';

-- Percentage of total — elegant with window functions
SELECT
    p.name  AS pitch,
    p.city,
    SUM(b.total_amount) AS pitch_revenue,
    SUM(SUM(b.total_amount)) OVER ()                AS total_revenue,
    SUM(SUM(b.total_amount)) OVER (PARTITION BY p.city) AS city_revenue,
    ROUND(
        SUM(b.total_amount) /
        SUM(SUM(b.total_amount)) OVER () * 100,
        1
    ) AS pct_of_total,
    ROUND(
        SUM(b.total_amount) /
        SUM(SUM(b.total_amount)) OVER (PARTITION BY p.city) * 100,
        1
    ) AS pct_of_city
FROM bookings b
JOIN pitches p ON b.pitch_id = p.pitch_id
WHERE b.status = 'completed'
GROUP BY p.pitch_id, p.name, p.city;

-- Cumulative distribution and percentile rank
SELECT
    name,
    price_per_hour,
    PERCENT_RANK() OVER (ORDER BY price_per_hour) AS percentile_rank,
    CUME_DIST()    OVER (ORDER BY price_per_hour) AS cumulative_distribution
FROM pitches WHERE is_active = true;
-- PERCENT_RANK: what % of rows have a lower value (0 to 1)
-- CUME_DIST: what % of rows have a value <= this one (0 to 1)
```

### Window Frame Syntax — The Full Picture

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- from start to current row
ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING          -- 5-row centered window
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING   -- from current to end
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING -- entire partition

-- ROWS vs RANGE:
-- ROWS: physical rows
-- RANGE: logical range of values (handles ties together)
-- When in doubt, use ROWS — RANGE has subtleties with duplicates
```

---

## 11. Indexes & Their Types

An index is a **separate data structure** the database maintains so it can find rows without scanning the entire table. Think of it like a book's index — instead of reading every page, you jump to the right page immediately.

### The Cost of Indexes

```
Indexes speed up reads but slow down writes.
Every INSERT, UPDATE, DELETE must also update all indexes on that table.

Rule of thumb:
  ✅ Index columns you filter on frequently (WHERE, JOIN ON)
  ✅ Index columns you sort on frequently (ORDER BY)
  ✅ Index foreign key columns
  ❌ Don't index columns with very few distinct values (boolean, status with 2 values)
  ❌ Don't over-index write-heavy tables (bulk imports, event logs)
  ❌ Remove unused indexes — they slow writes for no benefit
```

### B-Tree Index — The Default

```sql
-- Default index type — works for =, <, >, BETWEEN, LIKE 'prefix%', IN
-- Stored as a balanced tree — O(log n) lookup

-- Explicit creation
CREATE INDEX idx_bookings_start_time
    ON bookings (start_time);

-- Unique index — enforces uniqueness AND creates an index
CREATE UNIQUE INDEX idx_users_email
    ON users (email);

-- Composite (multi-column) index
-- Order matters! This index serves: (user_id), (user_id, status), (user_id, status, start_time)
-- It does NOT serve: (status) alone or (start_time) alone
CREATE INDEX idx_bookings_user_status_time
    ON bookings (user_id, status, start_time DESC);

-- The LEFT PREFIX RULE:
-- Composite index on (A, B, C) serves queries on:
--   A alone          ✅
--   A, B             ✅
--   A, B, C          ✅
--   B alone          ❌ (can't skip first column)
--   B, C             ❌
--   A, C             ✅ partial (uses A, then scans for C)

-- Descending index (PostgreSQL)
CREATE INDEX idx_bookings_created_desc
    ON bookings (created_at DESC);  -- optimal for ORDER BY created_at DESC LIMIT n

-- Conditional (Partial) Index — index only rows matching a condition
-- Smaller, faster, only covers the rows you care about
CREATE INDEX idx_active_bookings
    ON bookings (user_id, start_time)
    WHERE status IN ('pending', 'confirmed');  -- only index non-terminal bookings

CREATE INDEX idx_active_pitches_city
    ON pitches (city, price_per_hour)
    WHERE is_active = true;  -- searching inactive pitches is rare

-- Covering Index (PostgreSQL: INCLUDE)
-- Index contains all columns the query needs — no table heap access needed
CREATE INDEX idx_bookings_cover
    ON bookings (user_id, status)
    INCLUDE (booking_id, start_time, end_time, total_amount);
-- SELECT booking_id, start_time, total_amount FROM bookings
-- WHERE user_id = X AND status = 'completed'
-- → Can be answered entirely from the index (index-only scan)
```

### Hash Index — O(1) Equality Lookups

```sql
-- Hash index: only supports = (equality), not ranges
-- Faster than B-tree for pure equality lookups

-- PostgreSQL:
CREATE INDEX idx_users_email_hash
    ON users USING HASH (email);
-- Best for: WHERE email = 'exact@value.com'
-- Cannot use for: LIKE, <, >, ORDER BY

-- MySQL: Hash indexes only on MEMORY tables (not InnoDB)
-- InnoDB has an adaptive hash index built-in (automatic)
```

### GIN Index — Arrays, JSONB, Full-Text Search

```sql
-- GIN (Generalized Inverted Index): indexes the elements within a value
-- Used for: JSONB, arrays, full-text search, trigrams

-- Index JSONB for fast key/value lookups
CREATE INDEX idx_pitches_amenities_gin
    ON pitches USING GIN (amenities);
-- Enables fast: WHERE amenities @> '{"parking": true}'

-- Index array column
CREATE INDEX idx_pitches_surfaces_gin
    ON pitches USING GIN (allowed_surfaces);
-- Enables fast: WHERE 'artificial_turf' = ANY(allowed_surfaces)

-- Full-text search index
-- Step 1: Add a tsvector column (or use it inline)
ALTER TABLE pitches ADD COLUMN search_vector tsvector;

UPDATE pitches SET search_vector =
    to_tsvector('english', name || ' ' || COALESCE(description, '') || ' ' || city);

CREATE INDEX idx_pitches_fts
    ON pitches USING GIN (search_vector);

-- Step 2: Query with full-text search
SELECT name, city, ts_rank(search_vector, query) AS relevance
FROM pitches, to_tsquery('english', 'artificial & turf') AS query
WHERE search_vector @@ query
ORDER BY relevance DESC;

-- Trigram index (pg_trgm extension) — supports LIKE '%middle%'
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_pitches_name_trgm
    ON pitches USING GIN (name gin_trgm_ops);
-- Now this is fast: WHERE name ILIKE '%sport%'
-- Regular B-tree can't use LIKE with leading wildcard
```

### GiST Index — Geometric & Range Data

```sql
-- GiST (Generalized Search Tree): for geometric, range, and exclusion constraints
-- Used for: PostGIS geography, tsrange, daterange, exclusion constraints

-- Geographic proximity search (PostGIS)
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE INDEX idx_pitches_location
    ON pitches USING GIST (ST_MakePoint(longitude, latitude)::geography);

-- Find pitches within 5km of a point
SELECT name, city,
    ST_Distance(
        ST_MakePoint(longitude, latitude)::geography,
        ST_MakePoint(31.2357, 30.0444)::geography  -- Cairo center
    ) / 1000 AS distance_km
FROM pitches
WHERE ST_DWithin(
    ST_MakePoint(longitude, latitude)::geography,
    ST_MakePoint(31.2357, 30.0444)::geography,
    5000  -- 5km in meters
)
ORDER BY distance_km;

-- Range index for no-overlap constraint (our booking system)
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE INDEX idx_bookings_pitch_range
    ON bookings USING GIST (pitch_id, tstzrange(start_time, end_time));
-- Supports the EXCLUDE constraint for double-booking prevention
```

### BRIN Index — Very Large Tables with Natural Order

```sql
-- BRIN (Block Range INdex): tiny index, extremely fast for naturally-ordered data
-- Best for: timestamps in append-only tables (logs, events, time-series)
-- Not good for: randomly distributed values

CREATE INDEX idx_bookings_created_brin
    ON bookings USING BRIN (created_at);
-- Tiny index (kilobytes not megabytes) — perfect for multi-billion row event tables

-- MySQL uses B-Tree for all user-created indexes
-- (InnoDB has clustered index, secondary indexes, and adaptive hash index)
```

### EXPLAIN — Analyzing Query Plans

```sql
-- The most important command for performance tuning
EXPLAIN SELECT * FROM bookings WHERE user_id = 'some-uuid';

-- EXPLAIN ANALYZE actually runs the query and shows real stats
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT b.*, u.name
FROM bookings b
JOIN users u ON b.user_id = u.user_id
WHERE b.status = 'confirmed'
  AND b.start_time > NOW();

-- What to look for:
-- Seq Scan      → full table scan — missing index (bad for large tables)
-- Index Scan    → using index to find rows, then fetching from heap (good)
-- Index Only Scan → covering index, no heap access needed (best)
-- Bitmap Index Scan → fetches many rows from index, then batches heap reads
-- Hash Join     → hashing smaller table, probing with larger (good for large joins)
-- Nested Loop   → for each row in outer, scan inner (good when outer is small)
-- Sort          → explicit sort happening (consider index with ORDER BY)
-- cost=0.00..45.23 → estimated cost (startup..total)
-- rows=100      → estimated rows
-- actual time=0.123..5.456 → real time (ANALYZE only)
-- Buffers: hit=X shared=Y → cache hits vs disk reads

-- MySQL equivalent
EXPLAIN FORMAT=JSON SELECT * FROM bookings WHERE user_id = 'some-uuid';
-- Look for: type (ALL=bad, index, ref, eq_ref, const=best)
-- key: which index was used
-- rows: estimated rows scanned
-- Extra: "Using index" (covering), "Using filesort" (needs ORDER BY fix)

-- Force index usage (testing purposes — don't use in production code)
-- PostgreSQL: SET enable_seqscan = off;
-- MySQL:      SELECT * FROM bookings FORCE INDEX (idx_bookings_user) WHERE ...
```

### Index Maintenance

```sql
-- Find unused indexes (PostgreSQL)
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan    AS times_used,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;

-- Find table/index sizes (PostgreSQL)
SELECT
    relname                                    AS table_name,
    pg_size_pretty(pg_total_relation_size(oid)) AS total_size,
    pg_size_pretty(pg_relation_size(oid))       AS table_size,
    pg_size_pretty(pg_indexes_size(oid))        AS indexes_size
FROM pg_class
WHERE relkind = 'r'
ORDER BY pg_total_relation_size(oid) DESC;

-- Rebuild bloated indexes (after heavy updates/deletes)
REINDEX INDEX idx_bookings_user_status_time;  -- PostgreSQL
-- MySQL: ALTER TABLE bookings ENGINE=InnoDB;  -- rebuilds all indexes

-- Update table statistics (helps query planner)
ANALYZE bookings;         -- PostgreSQL
ANALYZE TABLE bookings;   -- MySQL
```

---

## 12. Views

A view is a **saved SELECT query** that you can query like a table. It doesn't store data — it runs the underlying query each time.

### Creating Views

```sql
-- Basic view — simplify a complex join
CREATE OR REPLACE VIEW booking_details AS
SELECT
    b.booking_id,
    b.start_time,
    b.end_time,
    b.status,
    b.total_amount,
    u.name          AS player_name,
    u.email         AS player_email,
    u.phone         AS player_phone,
    p.name          AS pitch_name,
    p.city,
    p.surface,
    p.price_per_hour,
    o.name          AS owner_name,
    pay.status      AS payment_status,
    pay.method      AS payment_method,
    pay.paid_at
FROM bookings b
JOIN users   u ON b.user_id   = u.user_id
JOIN pitches p ON b.pitch_id  = p.pitch_id
JOIN users   o ON p.owner_id  = o.user_id
LEFT JOIN payments pay ON b.booking_id = pay.booking_id;

-- Now query it like a table
SELECT * FROM booking_details WHERE city = 'Cairo' AND status = 'confirmed';
SELECT player_name, COUNT(*) FROM booking_details GROUP BY player_name;

-- Revenue dashboard view
CREATE OR REPLACE VIEW pitch_revenue_dashboard AS
SELECT
    p.pitch_id,
    p.name,
    p.city,
    p.surface,
    p.price_per_hour,
    ROUND(AVG(r.rating), 2)          AS avg_rating,
    COUNT(DISTINCT r.review_id)      AS review_count,
    COUNT(DISTINCT CASE WHEN b.status = 'completed' THEN b.booking_id END) AS completed_bookings,
    COUNT(DISTINCT CASE WHEN b.status = 'cancelled' THEN b.booking_id END) AS cancelled_bookings,
    COALESCE(SUM(CASE WHEN b.status = 'completed' THEN b.total_amount END), 0) AS total_revenue,
    COALESCE(AVG(CASE WHEN b.status = 'completed' THEN b.total_amount END), 0) AS avg_booking_value
FROM pitches p
LEFT JOIN bookings b ON p.pitch_id = b.pitch_id
LEFT JOIN reviews  r ON p.pitch_id = r.pitch_id AND r.is_visible = true
WHERE p.is_active = true
GROUP BY p.pitch_id, p.name, p.city, p.surface, p.price_per_hour;

-- Drop a view
DROP VIEW IF EXISTS booking_details;
```

### Materialized Views (PostgreSQL) — Cached Results

```sql
-- A materialized view stores the actual query result on disk.
-- Query it instantly — no re-execution of the underlying query.
-- But it's STALE until you REFRESH it.

CREATE MATERIALIZED VIEW monthly_revenue_summary AS
SELECT
    DATE_TRUNC('month', b.created_at) AS month,
    p.city,
    COUNT(*)                           AS bookings,
    SUM(b.total_amount)                AS revenue
FROM bookings b
JOIN pitches p ON b.pitch_id = p.pitch_id
WHERE b.status = 'completed'
GROUP BY DATE_TRUNC('month', b.created_at), p.city
WITH DATA;  -- populate immediately (vs WITH NO DATA)

-- Create index on materialized view (can't do this on regular views)
CREATE INDEX idx_mv_revenue_month
    ON monthly_revenue_summary (month, city);

-- Query is instant — no underlying join runs
SELECT * FROM monthly_revenue_summary WHERE city = 'Cairo' ORDER BY month;

-- Refresh when underlying data changes
REFRESH MATERIALIZED VIEW monthly_revenue_summary;  -- locks view during refresh

-- Refresh without blocking reads (PostgreSQL 9.4+)
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_revenue_summary;
-- Requires: UNIQUE index on the materialized view

-- When to use materialized views:
-- Dashboard queries that run every second but data changes every hour
-- Pre-computed aggregations over millions of rows
-- Reporting queries too expensive to run live
-- ⚠️ Not suitable when data must be up-to-the-second fresh

-- MySQL has no materialized views — simulate with:
-- 1. Summary tables populated by scheduled jobs
-- 2. Events (MySQL scheduler)
-- 3. Application-layer caching (Redis)
```

### Updatable Views

```sql
-- Simple views over one table without aggregates can be updated through the view
CREATE VIEW active_pitches AS
SELECT * FROM pitches WHERE is_active = true;

-- This works if: no GROUP BY, no DISTINCT, no aggregate functions, no subqueries
UPDATE active_pitches SET price_per_hour = 250 WHERE pitch_id = 'uuid';
-- Becomes: UPDATE pitches SET price_per_hour = 250 WHERE pitch_id = 'uuid'

-- WITH CHECK OPTION — prevent inserts/updates that violate the view's WHERE
CREATE VIEW active_pitches AS
SELECT * FROM pitches WHERE is_active = true
WITH CHECK OPTION;  -- can't insert/update a pitch with is_active = false through this view
```

---

## 13. Stored Procedures & Functions

### Functions (Return a Value)

```sql
-- PostgreSQL: PL/pgSQL functions
-- Function: Calculate booking price for a pitch duration
CREATE OR REPLACE FUNCTION calculate_booking_price(
    p_pitch_id  UUID,
    p_start     TIMESTAMPTZ,
    p_end       TIMESTAMPTZ
) RETURNS DECIMAL(10,2)
LANGUAGE plpgsql
AS $$
DECLARE
    v_price_per_hour DECIMAL(10,2);
    v_hours          DECIMAL(10,2);
    v_total          DECIMAL(10,2);
BEGIN
    -- Get pitch price
    SELECT price_per_hour INTO v_price_per_hour
    FROM pitches
    WHERE pitch_id = p_pitch_id AND is_active = true;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'Pitch % not found or inactive', p_pitch_id;
    END IF;

    -- Calculate hours (minimum 1 hour, round up to nearest 30 min)
    v_hours := GREATEST(
        1.0,
        CEIL(EXTRACT(EPOCH FROM (p_end - p_start)) / 1800) / 2.0  -- 30-min increments
    );

    -- Apply peak-hour pricing (18:00-22:00 weekdays)
    IF EXTRACT(DOW FROM p_start) BETWEEN 1 AND 5   -- Monday to Friday
    AND EXTRACT(HOUR FROM p_start) BETWEEN 18 AND 21 THEN
        v_total := v_price_per_hour * v_hours * 1.25;  -- 25% peak surcharge
    ELSE
        v_total := v_price_per_hour * v_hours;
    END IF;

    RETURN ROUND(v_total, 2);
END;
$$;

-- Usage:
SELECT calculate_booking_price(
    'pitch-uuid',
    '2024-06-15 18:00:00+03',
    '2024-06-15 20:00:00+03'
);

-- Function that returns a table (set-returning function)
CREATE OR REPLACE FUNCTION get_available_pitches(
    p_city      VARCHAR,
    p_start     TIMESTAMPTZ,
    p_end       TIMESTAMPTZ,
    p_capacity  SMALLINT DEFAULT 0
) RETURNS TABLE (
    pitch_id        UUID,
    name            VARCHAR,
    city            VARCHAR,
    surface         surface_type,
    capacity        SMALLINT,
    price_per_hour  DECIMAL,
    estimated_price DECIMAL
)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY
    SELECT
        p.pitch_id,
        p.name,
        p.city,
        p.surface,
        p.capacity,
        p.price_per_hour,
        calculate_booking_price(p.pitch_id, p_start, p_end) AS estimated_price
    FROM pitches p
    WHERE p.city      = p_city
      AND p.is_active = true
      AND p.capacity  >= p_capacity
      AND NOT EXISTS (
          SELECT 1 FROM bookings b
          WHERE b.pitch_id = p.pitch_id
            AND b.status NOT IN ('cancelled')
            AND b.start_time < p_end
            AND b.end_time   > p_start
      )
    ORDER BY p.price_per_hour;
END;
$$;

-- Usage like a table:
SELECT * FROM get_available_pitches('Cairo', NOW(), NOW() + INTERVAL '2 hours', 10);

-- MySQL stored function
DELIMITER $$
CREATE FUNCTION calculate_booking_price_mysql(
    p_pitch_id    CHAR(36),
    p_hours       DECIMAL(5,2)
) RETURNS DECIMAL(10,2)
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE v_price DECIMAL(10,2);
    SELECT price_per_hour INTO v_price FROM pitches WHERE pitch_id = p_pitch_id;
    IF v_price IS NULL THEN SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Pitch not found'; END IF;
    RETURN ROUND(v_price * p_hours, 2);
END$$
DELIMITER ;
```

### Stored Procedures (Execute Logic, No Direct Return)

```sql
-- PostgreSQL: Procedure for creating a complete booking
CREATE OR REPLACE PROCEDURE create_booking(
    p_user_id    UUID,
    p_pitch_id   UUID,
    p_start      TIMESTAMPTZ,
    p_end        TIMESTAMPTZ,
    p_method     payment_method,
    OUT o_booking_id UUID,
    OUT o_total      DECIMAL
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_conflict_count INT;
BEGIN
    -- 1. Check for conflicts
    SELECT COUNT(*) INTO v_conflict_count
    FROM bookings
    WHERE pitch_id = p_pitch_id
      AND status NOT IN ('cancelled')
      AND start_time < p_end
      AND end_time   > p_start;

    IF v_conflict_count > 0 THEN
        RAISE EXCEPTION 'Pitch is already booked for this time slot'
            USING ERRCODE = 'P0001';
    END IF;

    -- 2. Calculate price
    o_total := calculate_booking_price(p_pitch_id, p_start, p_end);

    -- 3. Create booking
    INSERT INTO bookings (user_id, pitch_id, start_time, end_time, total_amount)
    VALUES (p_user_id, p_pitch_id, p_start, p_end, o_total)
    RETURNING booking_id INTO o_booking_id;

    -- 4. Create pending payment record
    INSERT INTO payments (booking_id, amount, method, status)
    VALUES (o_booking_id, o_total, p_method, 'pending');

    -- Transaction commits automatically when procedure ends successfully
    -- Rollback happens automatically if any exception is raised
    RAISE NOTICE 'Booking % created for %', o_booking_id, o_total;
END;
$$;

-- Call a procedure
CALL create_booking(
    'user-uuid',
    'pitch-uuid',
    '2024-06-15 18:00:00+03',
    '2024-06-15 20:00:00+03',
    'card',
    NULL,  -- OUT parameter placeholder
    NULL   -- OUT parameter placeholder
);

-- MySQL stored procedure
DELIMITER $$
CREATE PROCEDURE cancel_booking(
    IN  p_booking_id CHAR(36),
    IN  p_reason     TEXT,
    OUT p_refund_amt DECIMAL(10,2)
)
BEGIN
    DECLARE v_status VARCHAR(20);
    DECLARE v_amount DECIMAL(10,2);

    -- Declare handler for errors
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    START TRANSACTION;

    SELECT status, total_amount INTO v_status, v_amount
    FROM bookings WHERE booking_id = p_booking_id FOR UPDATE;

    IF v_status != 'confirmed' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Only confirmed bookings can be cancelled';
    END IF;

    UPDATE bookings
    SET status = 'cancelled', updated_at = NOW()
    WHERE booking_id = p_booking_id;

    -- Full refund if cancelled > 24 hours before
    IF EXISTS (
        SELECT 1 FROM bookings
        WHERE booking_id = p_booking_id
          AND start_time > NOW() + INTERVAL 24 HOUR
    ) THEN
        SET p_refund_amt = v_amount;
        UPDATE payments SET status = 'refunded', refunded_at = NOW()
        WHERE booking_id = p_booking_id;
    ELSE
        SET p_refund_amt = v_amount * 0.5;  -- 50% refund
    END IF;

    COMMIT;
END$$
DELIMITER ;
```

---

## 14. Triggers

A trigger is a function that **automatically executes** when a specified event (INSERT, UPDATE, DELETE) occurs on a table.

```
Types of triggers:
  BEFORE trigger: runs before the operation — can modify data or prevent it
  AFTER trigger:  runs after the operation — good for audit logs, cascades
  INSTEAD OF:     replaces the operation — used on views

Row-level trigger: runs once per affected row (FOR EACH ROW)
Statement-level:   runs once per statement (FOR EACH STATEMENT)
```

### PostgreSQL Triggers

```sql
-- Trigger 1: Auto-update the updated_at timestamp
-- First, create the trigger function
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    NEW.updated_at = NOW();  -- NEW = the row being inserted/updated
    RETURN NEW;              -- must RETURN NEW in BEFORE triggers
END;
$$;

-- Attach it to multiple tables
CREATE TRIGGER trg_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER trg_pitches_updated_at
    BEFORE UPDATE ON pitches
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER trg_bookings_updated_at
    BEFORE UPDATE ON bookings
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();
-- Now you never need to manually set updated_at in any UPDATE query

-- Trigger 2: Audit log — record all booking status changes
CREATE TABLE booking_audit_log (
    log_id          BIGSERIAL PRIMARY KEY,
    booking_id      UUID NOT NULL,
    changed_by      UUID REFERENCES users(user_id),
    old_status      booking_status,
    new_status      booking_status,
    old_amount      DECIMAL(10,2),
    new_amount      DECIMAL(10,2),
    changed_at      TIMESTAMPTZ DEFAULT NOW(),
    change_source   TEXT  -- 'api', 'admin', 'scheduler'
);

CREATE OR REPLACE FUNCTION log_booking_changes()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Only log if status or amount actually changed
    IF OLD.status IS DISTINCT FROM NEW.status
    OR OLD.total_amount IS DISTINCT FROM NEW.total_amount
    THEN
        INSERT INTO booking_audit_log
            (booking_id, old_status, new_status, old_amount, new_amount)
        VALUES
            (NEW.booking_id, OLD.status, NEW.status, OLD.total_amount, NEW.total_amount);
    END IF;
    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_booking_audit
    AFTER UPDATE ON bookings
    FOR EACH ROW
    EXECUTE FUNCTION log_booking_changes();

-- Trigger 3: Validate business rules BEFORE insert
CREATE OR REPLACE FUNCTION validate_booking()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_capacity   SMALLINT;
    v_open_hour  SMALLINT;
    v_close_hour SMALLINT;
BEGIN
    -- Get pitch info
    SELECT capacity INTO v_capacity
    FROM pitches WHERE pitch_id = NEW.pitch_id;

    -- Business rule: booking must be between 6am and midnight
    IF EXTRACT(HOUR FROM NEW.start_time) < 6
    OR EXTRACT(HOUR FROM NEW.end_time)   > 24 THEN
        RAISE EXCEPTION 'Bookings only allowed between 06:00 and 24:00'
            USING ERRCODE = 'P0002';
    END IF;

    -- Business rule: can't book more than 30 days in advance
    IF NEW.start_time > NOW() + INTERVAL '30 days' THEN
        RAISE EXCEPTION 'Cannot book more than 30 days in advance'
            USING ERRCODE = 'P0003';
    END IF;

    -- Calculate and set the total amount automatically
    NEW.total_amount := calculate_booking_price(NEW.pitch_id, NEW.start_time, NEW.end_time);

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_validate_booking
    BEFORE INSERT OR UPDATE ON bookings
    FOR EACH ROW
    EXECUTE FUNCTION validate_booking();

-- Trigger 4: INSTEAD OF on a view — make a view updatable
CREATE OR REPLACE FUNCTION insert_booking_via_view()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
DECLARE
    v_user_id UUID;
    v_pitch_id UUID;
BEGIN
    -- Look up IDs from names
    SELECT user_id INTO v_user_id FROM users WHERE email = NEW.player_email;
    SELECT pitch_id INTO v_pitch_id FROM pitches WHERE name = NEW.pitch_name AND city = NEW.city;

    INSERT INTO bookings (user_id, pitch_id, start_time, end_time)
    VALUES (v_user_id, v_pitch_id, NEW.start_time, NEW.end_time);

    RETURN NEW;
END;
$$;

CREATE TRIGGER trg_insert_booking_detail
    INSTEAD OF INSERT ON booking_details
    FOR EACH ROW
    EXECUTE FUNCTION insert_booking_via_view();
```

### MySQL Triggers

```sql
-- MySQL trigger syntax
DELIMITER $$

-- Auto-update timestamp
CREATE TRIGGER trg_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
END$$

-- Audit log
CREATE TRIGGER trg_booking_status_log
    AFTER UPDATE ON bookings
    FOR EACH ROW
BEGIN
    IF OLD.status != NEW.status THEN
        INSERT INTO booking_audit_log
            (booking_id, old_status, new_status, changed_at)
        VALUES
            (NEW.booking_id, OLD.status, NEW.status, NOW());
    END IF;
END$$

-- Validate before insert
CREATE TRIGGER trg_validate_booking
    BEFORE INSERT ON bookings
    FOR EACH ROW
BEGIN
    IF NEW.end_time <= NEW.start_time THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'end_time must be after start_time';
    END IF;
    IF NEW.start_time < NOW() THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Cannot book in the past';
    END IF;
END$$

DELIMITER ;

-- MySQL limitations vs PostgreSQL triggers:
-- ❌ No statement-level triggers (always row-level)
-- ❌ No INSTEAD OF triggers
-- ❌ Can't call stored procedures from triggers
-- ❌ No access to pg_settings or complex types
-- ✅ Simpler syntax for simple use cases
```

---

## 15. Row-Level Security & Policies

Row-Level Security (RLS) is a **PostgreSQL feature** that lets you define who can see or modify which rows — enforced at the database level, not the application level.

```
Without RLS: application code does: WHERE owner_id = current_user_id
  → If you forget that WHERE clause, user sees everyone's data
  → Every query must manually add the filter

With RLS: database automatically adds the filter to EVERY query
  → Even if application code forgets, the DB enforces it
  → Zero-trust at the data layer
```

### Setting Up RLS

```sql
-- Step 1: Enable RLS on a table
ALTER TABLE pitches ENABLE ROW LEVEL SECURITY;
ALTER TABLE bookings ENABLE ROW LEVEL SECURITY;

-- Step 2: By default, all rows are HIDDEN for non-superusers
-- You must create policies to grant access

-- Step 3: Create policies (similar to WHERE clauses applied automatically)

-- Policy: pitch owners can see their own pitches, admins see all
CREATE POLICY pitches_owner_policy ON pitches
    FOR ALL                                    -- SELECT, INSERT, UPDATE, DELETE
    USING (
        owner_id = current_setting('app.current_user_id')::UUID  -- set by app before query
        OR current_setting('app.current_user_role') = 'admin'
    );

-- Policy: players see only active pitches
CREATE POLICY pitches_player_select ON pitches
    FOR SELECT
    USING (
        is_active = true
        OR owner_id = current_setting('app.current_user_id')::UUID
    );

-- Policy: bookings — users see only their own, owners see bookings for their pitches
CREATE POLICY bookings_user_policy ON bookings
    FOR SELECT
    USING (
        user_id = current_setting('app.current_user_id')::UUID  -- their booking
        OR EXISTS (                                               -- their pitch
            SELECT 1 FROM pitches p
            WHERE p.pitch_id = bookings.pitch_id
              AND p.owner_id = current_setting('app.current_user_id')::UUID
        )
        OR current_setting('app.current_user_role') = 'admin'
    );

-- Policy: users can only create bookings for themselves
CREATE POLICY bookings_insert_policy ON bookings
    FOR INSERT
    WITH CHECK (
        user_id = current_setting('app.current_user_id')::UUID
    );

-- Policy: users can only update/delete their own pending bookings
CREATE POLICY bookings_update_policy ON bookings
    FOR UPDATE
    USING (
        user_id = current_setting('app.current_user_id')::UUID
        AND status IN ('pending', 'confirmed')
    )
    WITH CHECK (
        user_id = current_setting('app.current_user_id')::UUID
    );

-- Bypass RLS for admin role (app service account)
ALTER TABLE pitches  FORCE ROW LEVEL SECURITY;
ALTER TABLE bookings FORCE ROW LEVEL SECURITY;

-- A special role that bypasses RLS
CREATE ROLE kickoff_admin BYPASSRLS;
-- Normal app role — respects RLS
CREATE ROLE kickoff_app;

-- In application code — set the user context before each query:
-- await db.query("SET LOCAL app.current_user_id = $1", [userId]);
-- await db.query("SET LOCAL app.current_user_role = $1", [userRole]);
-- (LOCAL means it's scoped to the current transaction)

-- List existing policies
SELECT * FROM pg_policies WHERE tablename = 'bookings';

-- Drop a policy
DROP POLICY IF EXISTS bookings_user_policy ON bookings;

-- Disable RLS temporarily (for admin queries)
ALTER TABLE bookings DISABLE ROW LEVEL SECURITY;
```

### Supabase-Style RLS (Real-World Pattern)

```sql
-- In Supabase (and many production PostgreSQL setups), JWT claims are used:

-- The authenticated user's ID comes from JWT claim
-- current_user is the DB role, auth.uid() is the app user
CREATE POLICY "Users can view own bookings" ON bookings
    FOR SELECT
    USING (user_id = auth.uid());

CREATE POLICY "Users can create own bookings" ON bookings
    FOR INSERT
    WITH CHECK (user_id = auth.uid());

CREATE POLICY "Owners can view their pitch bookings" ON bookings
    FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM pitches
            WHERE pitch_id = bookings.pitch_id
              AND owner_id = auth.uid()
        )
    );
```

---

## 16. Transactions & Isolation Levels

A transaction is a **unit of work** that either completes fully or not at all. We covered this in the backend guide — here's the database-level detail.

```sql
-- Basic transaction
BEGIN;  -- or START TRANSACTION

UPDATE accounts SET balance = balance - 500 WHERE user_id = 'sender';
UPDATE accounts SET balance = balance + 500 WHERE user_id = 'receiver';

-- If everything looks good:
COMMIT;
-- If something went wrong:
ROLLBACK;

-- Savepoints — nested rollback points
BEGIN;

INSERT INTO bookings (...) VALUES (...);
SAVEPOINT after_booking;

INSERT INTO payments (...) VALUES (...);
-- Payment failed!
ROLLBACK TO SAVEPOINT after_booking;  -- undo payment, keep booking

INSERT INTO payments (...) VALUES (...);  -- retry with different method
COMMIT;

-- Set isolation level for a transaction
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- or:
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### Isolation Levels in Practice

```sql
-- PostgreSQL isolation levels:

-- READ COMMITTED (default in PostgreSQL):
-- Each statement sees data committed before IT started
-- Prevents dirty reads. Allows non-repeatable reads and phantom reads.
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;  -- sees 1000
-- Another transaction commits: balance → 900
SELECT balance FROM accounts WHERE id = 1;  -- sees 900 ← different result!
COMMIT;

-- REPEATABLE READ (default in MySQL InnoDB):
-- Each transaction sees a snapshot from when IT started
-- Prevents dirty reads and non-repeatable reads.
-- PostgreSQL also prevents phantom reads (stronger than the standard)
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- sees 1000
-- Another transaction commits: balance → 900
SELECT balance FROM accounts WHERE id = 1;  -- still sees 1000 (snapshot)
COMMIT;

-- SERIALIZABLE:
-- Transactions appear to execute one after another, even if concurrent
-- Prevents all anomalies. Slowest. Use for financial operations.
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- PostgreSQL uses SSI (Serializable Snapshot Isolation) — less blocking than locking
-- Transactions may fail with serialization error — application must retry

-- Setting default isolation level (PostgreSQL)
ALTER DATABASE kickoff SET default_transaction_isolation = 'read committed';

-- Detecting and handling deadlocks
-- PostgreSQL automatically detects and resolves deadlocks by aborting one transaction
-- Always lock rows in a consistent order to prevent deadlocks:
-- ✅ Always lock user_id < other_user_id first
-- ✅ Always lock pitch_id before booking_id

-- Checking active transactions and locks (PostgreSQL)
SELECT
    pid,
    usename,
    application_name,
    state,
    query,
    wait_event_type,
    wait_event,
    now() - xact_start AS duration
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;

-- Find blocking queries
SELECT
    blocked_locks.pid         AS blocked_pid,
    blocked_activity.usename  AS blocked_user,
    blocking_locks.pid        AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query    AS blocked_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_locks.pid = blocked_activity.pid
JOIN pg_catalog.pg_locks blocking_locks ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.relation = blocked_locks.relation
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_locks.pid = blocking_activity.pid
WHERE NOT blocked_locks.granted;

-- Kill a blocking query
SELECT pg_terminate_backend(pid_here);
```

---

## 17. MongoDB — Document Model

MongoDB stores data as flexible JSON-like documents. Let's build the same Kickoff system in MongoDB to see the contrast.

### MongoDB vs PostgreSQL — Schema Approach

```javascript
// MongoDB — no schema definition needed — just insert
// But use Mongoose for schema validation in Node.js apps

// Embedded document approach — denormalization
// "A booking document contains everything you need"
{
  "_id": ObjectId("64f1..."),
  "userId": ObjectId("64f2..."),
  "pitchId": ObjectId("64f3..."),
  "pitch": {                          // ← EMBEDDED pitch snapshot
    "name": "Cairo Sports Club A",
    "city": "Cairo",
    "surface": "artificial_turf",
    "pricePerHour": 200
  },
  "player": {                         // ← EMBEDDED user snapshot
    "name": "Ahmed Ali",
    "email": "ahmed@kickoff.dev",
    "phone": "+201012345678"
  },
  "startTime": ISODate("2024-06-15T15:00:00Z"),
  "endTime":   ISODate("2024-06-15T17:00:00Z"),
  "status": "confirmed",
  "totalAmount": 400.00,
  "payment": {                        // ← EMBEDDED payment
    "method": "card",
    "status": "completed",
    "transactionRef": "txn_abc123",
    "paidAt": ISODate("2024-06-14T10:30:00Z")
  },
  "createdAt": ISODate("2024-06-14T10:00:00Z")
}
```

### Mongoose Schema Definition

```javascript
// models/Booking.js
const mongoose = require("mongoose");

const bookingSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "User",
    required: true,
    index: true,
  },
  pitchId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "Pitch",
    required: true,
    index: true,
  },
  // Embed a snapshot of the pitch for fast reads (denormalization)
  pitchSnapshot: {
    name:          { type: String, required: true },
    city:          { type: String, required: true },
    surface:       { type: String, enum: ["natural_grass","artificial_turf","futsal","beach"] },
    pricePerHour:  { type: Number, required: true },
  },
  startTime:  { type: Date, required: true, index: true },
  endTime:    { type: Date, required: true },
  status: {
    type: String,
    enum: ["pending","confirmed","cancelled","completed","no_show"],
    default: "pending",
    index: true,
  },
  totalAmount: { type: Number, required: true, min: 0 },
  payment: {
    method:  { type: String, enum: ["card","cash","wallet","bank_transfer"] },
    status:  { type: String, enum: ["pending","completed","failed","refunded"], default: "pending" },
    transactionRef: { type: String, unique: true, sparse: true },
    paidAt:  Date,
  },
  notes: String,
}, {
  timestamps: true,  // auto adds createdAt, updatedAt
});

// Custom validation
bookingSchema.path("endTime").validate(function(endTime) {
  return endTime > this.startTime;
}, "End time must be after start time");

// Compound index to prevent double booking
bookingSchema.index(
  { pitchId: 1, startTime: 1, endTime: 1 },
  { partialFilterExpression: { status: { $nin: ["cancelled"] } } }
);

// Virtual field
bookingSchema.virtual("durationHours").get(function() {
  return (this.endTime - this.startTime) / (1000 * 60 * 60);
});

module.exports = mongoose.model("Booking", bookingSchema);
```

### MongoDB CRUD

```javascript
const Booking = require("./models/Booking");
const Pitch   = require("./models/Pitch");

// ── CREATE ──────────────────────────────────────────────────────────────
const booking = await Booking.create({
  userId:    userId,
  pitchId:   pitchId,
  pitchSnapshot: {
    name: pitch.name, city: pitch.city,
    surface: pitch.surface, pricePerHour: pitch.pricePerHour,
  },
  startTime:   new Date("2024-06-15T15:00:00Z"),
  endTime:     new Date("2024-06-15T17:00:00Z"),
  totalAmount: 400,
  payment: { method: "card", status: "pending" },
});

// ── READ ─────────────────────────────────────────────────────────────────
// Find one
const booking = await Booking.findById(bookingId);
const booking = await Booking.findOne({ userId, status: "confirmed" });

// Find many with filtering, sorting, pagination
const bookings = await Booking.find({
    userId: userId,
    status: { $in: ["confirmed", "completed"] },
    startTime: { $gte: new Date("2024-01-01") },
  })
  .sort({ startTime: -1 })        // descending
  .skip((page - 1) * limit)
  .limit(limit)
  .populate("userId", "name email phone")  // JOIN equivalent — fetches user doc
  .lean();                                  // return plain objects (faster)

// Find available pitches — complex query
const unavailablePitchIds = await Booking.distinct("pitchId", {
  status: { $nin: ["cancelled"] },
  startTime: { $lt: requestedEnd },
  endTime:   { $gt: requestedStart },
});

const availablePitches = await Pitch.find({
  _id:      { $nin: unavailablePitchIds },
  city:     requestedCity,
  isActive: true,
  capacity: { $gte: requestedCapacity },
}).sort({ pricePerHour: 1 });

// ── UPDATE ───────────────────────────────────────────────────────────────
// findOneAndUpdate — returns updated doc
const updated = await Booking.findOneAndUpdate(
  { _id: bookingId, status: "pending" },   // find condition
  {
    $set: { status: "confirmed", updatedAt: new Date() },
    $push: { statusHistory: { status: "confirmed", at: new Date() } },
  },
  { new: true, runValidators: true }        // return updated doc, run schema validators
);

// Update many
await Booking.updateMany(
  { endTime: { $lt: new Date() }, status: "confirmed" },
  { $set: { status: "completed" } }
);

// ── DELETE ───────────────────────────────────────────────────────────────
// Soft delete (preferred)
await Booking.findByIdAndUpdate(bookingId, { $set: { isDeleted: true } });

// Hard delete
await Booking.deleteOne({ _id: bookingId });
await Booking.deleteMany({ status: "cancelled", createdAt: { $lt: cutoff } });
```

### MongoDB Aggregation Pipeline

```javascript
// Aggregation pipeline — MongoDB's equivalent of complex SQL
// Each stage transforms the documents

const revenueByCity = await Booking.aggregate([
  // Stage 1: Filter — like WHERE
  { $match: {
      status: "completed",
      createdAt: { $gte: new Date("2024-01-01") }
  }},

  // Stage 2: Join — like JOIN
  { $lookup: {
      from:         "pitches",        // collection to join
      localField:   "pitchId",        // field in bookings
      foreignField: "_id",            // field in pitches
      as:           "pitch"           // output array field
  }},

  // Stage 3: Unwind the array (lookup creates array, unwind flattens it)
  { $unwind: "$pitch" },

  // Stage 4: Group — like GROUP BY
  { $group: {
      _id:            "$pitch.city",
      totalRevenue:   { $sum: "$totalAmount" },
      bookingCount:   { $count: {} },
      avgRevenue:     { $avg: "$totalAmount" },
      uniquePlayers:  { $addToSet: "$userId" },  // collect unique values
  }},

  // Stage 5: Add computed field
  { $addFields: {
      uniquePlayerCount: { $size: "$uniquePlayers" },
  }},

  // Stage 6: Filter groups — like HAVING
  { $match: {
      totalRevenue: { $gte: 10000 }
  }},

  // Stage 7: Sort — like ORDER BY
  { $sort: { totalRevenue: -1 } },

  // Stage 8: Shape output — like SELECT
  { $project: {
      _id:               0,            // exclude _id
      city:              "$_id",       // rename
      totalRevenue:      1,
      bookingCount:      1,
      avgRevenue:        { $round: ["$avgRevenue", 2] },
      uniquePlayerCount: 1,
  }},
]);

// Window functions equivalent — $setWindowFields (MongoDB 5+)
const bookingsWithRunningTotal = await Booking.aggregate([
  { $match: { status: "completed" }},
  { $setWindowFields: {
      partitionBy: "$userId",
      sortBy:      { startTime: 1 },
      output: {
        runningTotal: {
          $sum: "$totalAmount",
          window: { documents: ["unbounded", "current"] }  // running sum
        },
        bookingNumber: {
          $documentNumber: {}  // ROW_NUMBER equivalent
        },
      }
  }},
]);

// Facet — multiple aggregations in parallel
const pitchStats = await Pitch.aggregate([
  { $match: { isActive: true }},
  { $facet: {
      bySurface: [
        { $group: { _id: "$surface", count: { $count: {} } } }
      ],
      byCity: [
        { $group: { _id: "$city", count: { $count: {} }, avgPrice: { $avg: "$pricePerHour" } } },
        { $sort: { count: -1 } },
        { $limit: 10 }
      ],
      priceRanges: [
        { $bucket: {
            groupBy: "$pricePerHour",
            boundaries: [0, 100, 200, 300, 500],
            default: "500+",
            output: { count: { $count: {} } }
        }}
      ]
  }}
]);
```

### MongoDB Indexes

```javascript
// Mongoose index syntax
const schema = new Schema({ ... });

// Single field index
schema.index({ userId: 1 });
schema.index({ email: 1 }, { unique: true });
schema.index({ createdAt: -1 });  // descending

// Compound index
schema.index({ pitchId: 1, startTime: 1, status: 1 });

// Text index (full-text search)
schema.index({ name: "text", description: "text", city: "text" });
// Query: Pitch.find({ $text: { $search: "artificial turf cairo" } })

// TTL index — auto-delete documents after expiry
schema.index({ expiresAt: 1 }, { expireAfterSeconds: 0 });
// Document is deleted when expiresAt <= now
// Perfect for: sessions, OTP codes, temporary tokens

// Partial index (sparse equivalent)
schema.index(
  { transactionRef: 1 },
  {
    unique: true,
    partialFilterExpression: { "payment.transactionRef": { $exists: true } }
  }
);

// MongoDB shell syntax
db.bookings.createIndex(
  { pitchId: 1, startTime: 1 },
  { partialFilterExpression: { status: { $ne: "cancelled" } } }
);

// Explain a query
db.bookings.find({ userId: ObjectId("...") }).explain("executionStats");
// Look for: IXSCAN (index scan) vs COLLSCAN (collection scan)
```

---

## 18. Database Design Patterns & Anti-Patterns

### Patterns — Things to Do

```sql
-- 1. SOFT DELETE — mark as deleted instead of removing
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMPTZ;
ALTER TABLE users ADD COLUMN deleted_by UUID REFERENCES users(user_id);

-- Partial index so queries on active data are fast
CREATE INDEX idx_users_active ON users (email) WHERE deleted_at IS NULL;

-- All queries automatically exclude deleted records
CREATE VIEW active_users AS SELECT * FROM users WHERE deleted_at IS NULL;

-- 2. AUDIT TRAIL — every table gets history tracking
CREATE TABLE users_history (
    history_id  BIGSERIAL   PRIMARY KEY,
    operation   CHAR(1)     NOT NULL,  -- 'I'nsert, 'U'pdate, 'D'elete
    changed_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    changed_by  UUID,
    -- Copy all user columns:
    user_id     UUID,
    name        VARCHAR(100),
    email       VARCHAR(255),
    role        user_role,
    -- ... etc
);

-- 3. POLYMORPHIC ASSOCIATION — one table relates to multiple entities
-- e.g., comments can be on pitches, matches, or teams
CREATE TABLE comments (
    comment_id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content         TEXT NOT NULL,
    user_id         UUID NOT NULL REFERENCES users(user_id),
    -- Polymorphic reference:
    entity_type     VARCHAR(50) NOT NULL,  -- 'pitch', 'match', 'team'
    entity_id       UUID NOT NULL,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);
-- Can't use FK (entity_id references different tables depending on entity_type)
-- Enforce at application level or with triggers

-- 4. ENUM vs LOOKUP TABLE
-- Enums: status values that almost never change (booking status, payment method)
-- Lookup table: values that users or admins might add (cities, sport types, amenity tags)

CREATE TABLE sport_types (
    sport_id    SMALLINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    name        VARCHAR(50) NOT NULL UNIQUE,
    icon_url    TEXT
);
-- Now pitches.sport_id references this table — sports can be added without schema change

-- 5. PARTITIONING — for very large tables (PostgreSQL)
-- Partition bookings by month — each month is a separate physical table
CREATE TABLE bookings (
    booking_id  UUID NOT NULL,
    start_time  TIMESTAMPTZ NOT NULL,
    -- ... other columns
    PRIMARY KEY (booking_id, start_time)  -- partition key must be in PK
) PARTITION BY RANGE (start_time);

CREATE TABLE bookings_2024_01 PARTITION OF bookings
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE bookings_2024_02 PARTITION OF bookings
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
-- Queries automatically routed to the right partition
-- DELETE old partitions instead of slow DELETE queries (DROP TABLE is instant!)
```

### Anti-Patterns — Things to Avoid

```sql
-- ❌ 1. Storing comma-separated values in a column
-- BAD:
CREATE TABLE bookings (
    booking_id UUID,
    player_ids TEXT  -- "uuid1,uuid2,uuid3"
);
-- Can't JOIN, can't index, can't enforce FK, can't query efficiently
-- ✅ Use a junction table instead

-- ❌ 2. Using string for IDs / booleans
-- BAD:
status TEXT  -- stores "1", "0", "true", "false", "yes", "no" — inconsistent
-- ✅ Use BOOLEAN or ENUM

-- ❌ 3. Putting too many tables in one database (bounded contexts violation)
-- ✅ Separate schema per domain: public, analytics, audit, auth

-- ❌ 4. SELECT * in views and stored code
-- BAD:
CREATE VIEW pitch_view AS SELECT * FROM pitches;
-- When you add a column to pitches, view exposes it immediately (might be sensitive)
-- ✅ Always explicitly list columns

-- ❌ 5. Missing indexes on foreign keys
-- Every FK column should have an index for JOIN performance
-- PostgreSQL does NOT automatically index FK columns (MySQL InnoDB does)
-- ✅ Add: CREATE INDEX idx_bookings_pitch_id ON bookings(pitch_id);

-- ❌ 6. Using DATETIME / TIMESTAMP without timezone (MySQL trap)
-- BAD: storing 2024-06-15 18:00:00 without knowing the timezone
-- During DST changes, 2am becomes 3am or vice versa — duplicates or gaps
-- ✅ PostgreSQL: always use TIMESTAMPTZ
-- ✅ MySQL: store as UTC, convert in application layer

-- ❌ 7. Wide tables (too many columns)
-- BAD: users table with 80 columns (address, preferences, stats, billing, ...)
-- ✅ Vertical partitioning: users (core), user_profiles (details), user_preferences (settings)

-- ❌ 8. N+1 in raw SQL (same as ORM N+1)
-- BAD:
-- SELECT * FROM pitches;           -- 1 query
-- SELECT * FROM reviews WHERE pitch_id = ?;  -- N queries (one per pitch)
-- ✅ Use JOIN or multiple targeted queries in one go

-- ❌ 9. Storing calculated fields without derivation tracking
-- BAD: bookings.duration_hours = 2 (stored)
-- When start_time or end_time changes, duration_hours can go stale
-- ✅ Calculate in query, or use a trigger to auto-update, or use a generated column:
ALTER TABLE bookings ADD COLUMN duration_hours
    DECIMAL(5,2) GENERATED ALWAYS AS
    (EXTRACT(EPOCH FROM (end_time - start_time)) / 3600) STORED;

-- ❌ 10. Transaction-less multi-statement operations
-- BAD:
INSERT INTO bookings VALUES (...);
INSERT INTO payments VALUES (...);  -- if this fails, booking exists with no payment!
-- ✅ Always wrap related operations in a transaction
```

---

## 19. Interview Questions & Answers

**Q: What is the difference between WHERE and HAVING?**

WHERE filters rows before they are grouped — it works on individual row values. HAVING filters groups after GROUP BY has already been applied — it works on aggregated values. You cannot use aggregate functions like SUM() or COUNT() in a WHERE clause. A query can have both: `WHERE status = 'completed'` filters rows first, then `HAVING SUM(total_amount) > 1000` filters the resulting groups.

---

**Q: What is the difference between INNER JOIN and LEFT JOIN?**

INNER JOIN returns only rows where there is a match in both tables — unmatched rows from either table are excluded. LEFT JOIN returns all rows from the left table plus matching rows from the right; where there is no match, the right side columns contain NULL. Use LEFT JOIN when you need to include records from the left table even if they have no related records in the right table — for example, listing all pitches even if some have no bookings yet.

---

**Q: Explain the difference between a clustered and non-clustered index.**

A clustered index determines the physical order of data in the table — the rows are stored on disk in the order of the clustered index key. A table can have only one clustered index. In MySQL InnoDB, the primary key is always the clustered index. PostgreSQL has no traditional clustered index (rows are stored in heap), but you can use `CLUSTER table USING index` to reorder them. A non-clustered index is a separate structure that points back to the actual rows — like a book index pointing to page numbers. A table can have many non-clustered indexes.

---

**Q: What is a covering index?**

A covering index contains all the columns that a query needs — the query can be satisfied entirely from the index without accessing the actual table rows. This is called an "index-only scan" in PostgreSQL and is significantly faster because it eliminates the heap access. In PostgreSQL, you create covering indexes with the INCLUDE clause: `CREATE INDEX idx ON bookings (user_id, status) INCLUDE (booking_id, total_amount)`. The query planner will use an index-only scan when all SELECTed and WHEREd columns are in the index.

---

**Q: When would you use a materialized view vs a regular view?**

A regular view is a saved query that executes every time you query it — always current but has the full query cost every time. A materialized view caches the query result on disk — queries are instant but data is stale until you REFRESH it. Use materialized views for expensive aggregations or joins over millions of rows that are queried frequently (dashboards, reports) but where the underlying data changes slowly. Use regular views for simplifying complex queries, enforcing access patterns, or when data must be current to the second.

---

**Q: What is the N+1 query problem in SQL?**

The N+1 problem occurs when you fetch N rows then make one additional query per row to get related data, resulting in N+1 total queries. Example: fetch 20 pitches (1 query), then for each pitch fetch its owner (20 more queries) = 21 total. Fix: use JOIN to get all related data in one query, or use `IN` to batch-fetch all owners at once. In ORM contexts: use `eager loading` (Sequelize `include`, Django `select_related`, Mongoose `populate`).

---

**Q: Explain ACID and give a real example of each property.**

Atomicity: a money transfer either debits the sender AND credits the receiver, or neither happens — never just one. Consistency: a booking for a pitch that doesn't exist is rejected because the FK constraint is violated — the database never enters an inconsistent state. Isolation: two users booking the same pitch at the same time — with SERIALIZABLE isolation, one will succeed and the other will fail cleanly rather than creating a double booking. Durability: once a payment is committed, it survives a server crash because it's written to the Write-Ahead Log (WAL) on disk before the transaction is acknowledged.

---

**Q: What is the difference between TRUNCATE, DELETE, and DROP?**

DELETE removes rows matching a WHERE clause (or all rows if no WHERE), is logged per-row, can be rolled back, and fires DELETE triggers. TRUNCATE removes all rows instantly by deallocating data pages, is not logged per-row (much faster), cannot be rolled back in MySQL (DDL), can be rolled back in PostgreSQL, and does not fire row-level triggers. DROP removes the entire table structure plus all its data, indexes, triggers, and constraints — it cannot be rolled back.

---

**Q: How do you handle many-to-many relationships?**

Use a junction (bridge/associative) table. For users and teams, create a team_members table with (team_id, user_id) as a composite primary key, both columns being foreign keys to their respective tables. This junction table can also carry additional attributes specific to the relationship — like role (captain vs member) or joined_at. Never store comma-separated IDs in a single column; that violates 1NF and makes querying impossible.

---

**Q: When should you use NoSQL over SQL?**

NoSQL is the better choice when: your schema is genuinely unpredictable and will evolve rapidly (product catalog where different products have entirely different attributes); your data is naturally hierarchical and always accessed together (a blog post with its nested comments, accessed as one document); you need to scale writes horizontally across many servers; or you're dealing with time-series or event data at very high ingestion rates. SQL remains the better choice when data has clear relationships, you need complex ad-hoc queries, ACID transactions are critical, or you're reporting across multiple entities.

---

**Q: What is a window function and how does it differ from GROUP BY?**

GROUP BY collapses multiple rows into a single result row per group — you lose access to individual row values. A window function computes a value for each row based on a window (set of related rows) but keeps every original row in the output. `SUM(amount) GROUP BY user_id` gives one row per user. `SUM(amount) OVER (PARTITION BY user_id ORDER BY created_at)` gives one row per booking with a running total for that user. Window functions enable running totals, moving averages, rankings, and lead/lag comparisons that are impossible with GROUP BY alone.

---

## 📖 Master Resource List

- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/) — the gold standard
- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [Use The Index, Luke](https://use-the-index-luke.com/) — index and query optimization
- [The Art of PostgreSQL](https://theartofpostgresql.com/)
- [Designing Data-Intensive Applications — Kleppmann](https://dataintensive.net/) ← must read
- [SQL Antipatterns — Bill Karwin](https://pragprog.com/titles/bksqla/sql-antipatterns/)
- [PostgreSQL Exercises](https://pgexercises.com/) — free interactive practice
- [SQLZoo](https://sqlzoo.net/) — beginner interactive SQL
- [Mode Analytics SQL Tutorial](https://mode.com/sql-tutorial/)

---

*That's the full journey from blank page to production-grade database. Build it, break it, index it, secure it. You've got the map now, Ahmed. 🗄️*
