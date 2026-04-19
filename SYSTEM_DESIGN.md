# Earn-N-Learn: Campus Marketplace Platform — System Design Document

**Project:** Earn-N-Learn  
**Document Type:** System Design Document (Academic / Production-Oriented)  
**Institution:** United International University — Department of CSE  
**Course:** CSE 3412 — System Analysis and Design Laboratory  
**Group:** Game of Threads (Group 5, Section D)  
**Authors:** Tabassum Sumaiya · Jarin Tabassum · Md. Hasibul Hossain · Abu Hurayra Mahbe · Ashraful Islam Tanzil  
**Date:** April 2026  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Requirements](#2-requirements)
3. [High-Level Architecture](#3-high-level-architecture)
4. [Detailed Component Design](#4-detailed-component-design)
5. [Database Design](#5-database-design)
6. [Scalability and Performance](#6-scalability-and-performance)
7. [Reliability and Fault Tolerance](#7-reliability-and-fault-tolerance)
8. [Security Design](#8-security-design)
9. [Monitoring and Observability](#9-monitoring-and-observability)
10. [Deployment Architecture](#10-deployment-architecture)
11. [Future Improvements and Known Limitations](#11-future-improvements-and-known-limitations)

---

## 1. Executive Summary

### 1.1 What Is Earn-N-Learn?

**Earn-N-Learn** is a campus-focused digital marketplace and collaboration platform designed exclusively for university students. It is a web and mobile application that consolidates five fragmented student workflows — job hunting, peer tutoring, academic material exchange, project collaboration, and financial management — into a single, university-verified environment.

Students currently rely on disconnected platforms: Facebook groups for gig posts, OLX for textbook sales, LinkedIn for job applications, and WhatsApp for project coordination. This fragmentation wastes time, creates security risks, and excludes students who lack access to the right networks. Earn-N-Learn collapses all of these into one trusted, student-only ecosystem.

### 1.2 Why It Exists

A survey of 209 UIU students revealed that over 60% strongly desired an all-in-one campus platform, and a majority cited fragmented information channels as their primary frustration. The platform directly addresses this validated pain point by providing:

- A verified **Job & Gig Marketplace** for part-time and freelance work
- A **Skill-Sharing Network** for peer tutoring and micro-services
- A **Material Exchange** for buying, selling, and renting academic resources
- A **Secure Escrow Wallet** for safe campus-level financial transactions
- A **Campus Social Hub** for discussions, events, and community
- A **Project Collaboration Workspace** with task boards and milestones
- A **Financial Savings Dashboard** for goal tracking and literacy

### 1.3 System Classification

Earn-N-Learn is a **modular monolith** with real-time capabilities. The backend is a single deployable Node.js/Express application organized into well-separated domain modules. Real-time communication is handled via Socket.IO. The architecture is designed to evolve toward microservices as user load demands it, without requiring a full rewrite.

---

## 2. Requirements

### 2.1 Functional Requirements

#### FR-01: User Registration and Profile Management
- New users register using a university email and student ID for institutional verification.
- Profiles include skills, certifications, portfolio links, ratings, and endorsements.
- Role-based accounts: Student, Recruiter/Employer, Admin.
- Profile serves as the identity hub linked across all modules.

#### FR-02: Job and Gig Posting & Application System
- Verified users can post part-time jobs, gigs, and freelance opportunities with title, budget, skills required, deadline, and project type.
- Applicants can browse, filter by skill/budget/type, and apply directly.
- Posters manage a dashboard to review, shortlist, interview, hire, or reject applicants.
- Status transitions trigger automated notifications.

#### FR-03: Material Exchange System
- Users list academic materials (textbooks, lab equipment) for sale, purchase, or rental.
- Listings include item condition, price, category, photos, and delivery/pickup details.
- Escrow-backed transactions protect both buyer and seller.

#### FR-04: Skill Sharing System
- Students offer or request tutoring, coding help, design services, or other peer services.
- Listings include rate, availability, location preference, and ratings.
- Booking system with confirmation, calendar slots, and transaction history.

#### FR-05: Escrow Payment System
- A built-in digital wallet holds funds in escrow until service/job completion.
- Funds release only upon client approval or auto-release after a defined window.
- Dispute resolution managed by Admin.
- Integration with SSLCommerz for top-ups and withdrawals.

#### FR-06: Real-Time Messaging and Notifications
- One-to-one and group chats with typing indicators, read receipts, and file sharing.
- System-generated push notifications for application updates, payment events, and deadlines.
- Optional voice/video call support for tutoring sessions.

#### FR-07: Campus Social Network
- Interest-based groups (academic departments, clubs, study circles).
- Personalized campus feed with posts, polls, comments, reactions.
- Event announcements with RSVP tracking.

#### FR-08: Project and Task Management
- Project workspaces with Kanban-style task boards, milestones, and deadlines.
- Shared resource storage (documents, links) and integrated project chat.
- Progress tracking visible to all collaborators; linked to user profiles for portfolio credibility.

#### FR-09: Financial Savings and Analytics Dashboard
- Users set savings goals (e.g., laptop, tuition) with target amounts and dates.
- Earnings can be allocated toward specific goals from the wallet.
- Real-time visual progress tracker and analytics on income and spending.

---

### 2.2 Non-Functional Requirements

| Category | Requirement |
|---|---|
| **Availability** | ≥ 99.5% uptime (≤ 43.8 hours downtime per year) |
| **Latency** | Page load < 2 s; API response < 500 ms; WebSocket message delivery < 100 ms |
| **Scalability** | Support 1,000+ concurrent users; horizontal scaling via load balancer + DB replicas |
| **Consistency** | Eventual consistency acceptable for feeds and notifications; strong consistency required for payment and escrow operations |
| **Durability** | Daily automated database backups with point-in-time recovery |
| **Security** | JWT session management, bcrypt password hashing, HTTPS/TLS everywhere, RBAC, rate limiting |
| **Compliance** | GDPR data handling; PCI-DSS compliance for payment flows; clear data retention and deletion policy |
| **Maintainability** | TypeScript codebase, ESLint + Prettier, modular architecture, inline documentation |
| **Portability** | Responsive across mobile (≥ 320 px), tablet (≥ 768 px), desktop (≥ 1024 px) |

---

### 2.3 Constraints and Assumptions

**Constraints:**
- Authentication is restricted to university email/student ID verification; general public cannot register.
- Payment processing depends on SSLCommerz availability (Bangladesh-local) and Stripe for international.
- Development timeline: 16 weeks with a team of 5 students.
- Hosting budget: $50–$200/month on a VPS or cloud provider (AWS, Render, or Heroku).
- No offline-first capability in V1; a stable internet connection (minimum 2 Mbps) is required.

**Assumptions:**
- At least 50% of the campus student body will adopt the platform within one academic year.
- The university permits a student-driven platform to operate on campus.
- Payment gateway APIs remain stable and backward-compatible.
- Server resources will be provisioned for peak-usage periods (semester start, exam season).
- Students have access to smartphones and/or desktop browsers.

---

## 3. High-Level Architecture

### 3.1 Architecture Style: Modular Monolith with Real-Time Layer

Earn-N-Learn adopts a **modular monolith** architecture for V1. All domain logic (users, jobs, marketplace, payments, messaging, social, projects) lives in one deployable Node.js/Express application, organized into well-isolated feature modules. This choice reduces operational complexity (no distributed tracing, no inter-service network calls, no duplicate config management) while preserving the ability to extract hot modules into independent services as load grows.

The **real-time layer** is a semi-independent concern handled by Socket.IO, which sits alongside the HTTP API server within the same Node.js process but can be horizontally scaled independently using a Redis Pub/Sub adapter.

### 3.2 System Architecture Diagram (ASCII)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             CLIENT TIER                                      │
│                                                                              │
│  ┌──────────────────────┐         ┌───────────────────────┐                 │
│  │  React 18 / TypeScript│         │  Mobile App           │                 │
│  │  (Web Browser)        │         │  (Android / iOS PWA)  │                 │
│  └──────────┬───────────┘         └──────────┬────────────┘                 │
└─────────────┼───────────────────────────────┼─────────────────────────────┘
              │  HTTPS / WSS                   │  HTTPS / WSS
              ▼                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             EDGE / DELIVERY TIER                             │
│                                                                              │
│   ┌────────────────────┐        ┌──────────────────────────┐                │
│   │  CDN (CloudFront / │        │  SSL Termination          │                │
│   │  Cloudflare)       │        │  Nginx Reverse Proxy      │                │
│   │  (Static assets,   │        │  (Load balancer → API     │                │
│   │   media uploads)   │        │   server pool)            │                │
│   └────────────────────┘        └──────────┬───────────────┘                │
└────────────────────────────────────────────┼────────────────────────────────┘
                                             │
                                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           APPLICATION TIER                                   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │            Node.js / Express API Server (Modular Monolith)          │    │
│  │                                                                     │    │
│  │  ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌──────────────────────┐ │    │
│  │  │  Auth    │ │  Jobs &  │ │ Marketplace│ │  Escrow / Wallet     │ │    │
│  │  │  Module  │ │  Gigs    │ │  Module    │ │  Module              │ │    │
│  │  └──────────┘ └──────────┘ └────────────┘ └──────────────────────┘ │    │
│  │  ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌──────────────────────┐ │    │
│  │  │  Social  │ │ Projects │ │  Profile   │ │  Notification        │ │    │
│  │  │  Module  │ │  Module  │ │  Module    │ │  Module              │ │    │
│  │  └──────────┘ └──────────┘ └────────────┘ └──────────────────────┘ │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌──────────────────────────────────────┐                                   │
│  │   Socket.IO Real-Time Server         │                                   │
│  │   (Chat, Notifications, Presence)    │                                   │
│  └──────────────────────────────────────┘                                   │
└────────────────────────────┬──────────────────────────────┬─────────────────┘
                             │                              │
              ┌──────────────┘                              └───────────────┐
              ▼                                                             ▼
┌─────────────────────────┐                             ┌───────────────────────┐
│      DATA TIER          │                             │  EXTERNAL SERVICES    │
│                         │                             │                       │
│  ┌─────────────────┐    │                             │  ┌─────────────────┐  │
│  │  MySQL (Primary)│    │                             │  │  SSLCommerz /   │  │
│  │  + Read Replica │    │                             │  │  Stripe         │  │
│  └─────────────────┘    │                             │  │  (Payments)     │  │
│  ┌─────────────────┐    │                             │  └─────────────────┘  │
│  │  Redis          │    │                             │  ┌─────────────────┐  │
│  │  (Cache + WS    │    │                             │  │  AWS S3 /       │  │
│  │   Pub/Sub)      │    │                             │  │  Cloudinary     │  │
│  └─────────────────┘    │                             │  │  (File Storage) │  │
│  ┌─────────────────┐    │                             │  └─────────────────┘  │
│  │  AWS S3 /       │    │                             │  ┌─────────────────┐  │
│  │  Cloudinary     │    │                             │  │  SendGrid /     │  │
│  │  (Media Store)  │    │                             │  │  Nodemailer     │  │
│  └─────────────────┘    │                             │  │  (Email)        │  │
└─────────────────────────┘                             └───────────────────────┘
```

---

### 3.3 Component Breakdown with Responsibilities

| Component | Technology | Responsibility |
|---|---|---|
| **React Frontend** | React 18, TypeScript, Tailwind CSS, Vite | Renders the UI; communicates with the API over REST and with Socket.IO over WebSocket |
| **Nginx Reverse Proxy** | Nginx | SSL termination, static file serving, upstream load balancing to API nodes |
| **API Server** | Node.js 20+, Express.js | Handles all HTTP REST endpoints; enforces authentication/authorization middleware; orchestrates domain logic |
| **Socket.IO Server** | Socket.IO 4 | Manages persistent WebSocket connections for chat, presence, and real-time notifications |
| **Auth Module** | JWT, bcrypt | Registration, login, token issuance/refresh/revocation, university email verification |
| **Job/Gig Module** | Express Router | CRUD for job listings; application submission and status management; filtering and search |
| **Marketplace Module** | Express Router | Material listings with escrow-tied transaction initiation; buy/sell/rent workflows |
| **Skill Sharing Module** | Express Router | Service listings; booking workflow; calendar integration |
| **Escrow/Wallet Module** | Express Router + DB transactions | Wallet top-up/withdrawal; fund locking into escrow; milestone-based release; dispute flagging |
| **Social Module** | Express Router | Campus feed CRUD; group management; event posts; reactions and comments |
| **Project Module** | Express Router | Project workspace CRUD; Kanban task state machine; milestone tracking; file attachment |
| **Notification Module** | Node.js event emitter + Socket.IO | Internal event bus for cross-module notifications; batching for email/push delivery |
| **MySQL Database** | MySQL 8 | Persistent relational storage for all structured data; ACID transactions for payments |
| **Redis** | Redis 7 | Session caching; rate-limit counters; Socket.IO adapter for multi-node pub/sub |
| **AWS S3 / Cloudinary** | S3 or Cloudinary API | Durable object storage for profile photos, portfolio files, material listing images, chat attachments |
| **SSLCommerz / Stripe** | Payment SDK | Fiat payment processing; webhook events for deposit/withdrawal confirmation |
| **SendGrid / Nodemailer** | SMTP / API | Transactional emails: registration verification, password reset, payment receipts |
| **CDN** | CloudFront or Cloudflare | Global delivery of static assets (JS bundles, CSS, images); reduces API server load |

---

### 3.4 Data Flow Between Components

**User Login Flow:**
```
Browser → Nginx → API Server (Auth Module)
  → MySQL (validate credentials, fetch user record)
  → bcrypt.compare(password, hash)
  → Issue JWT (access token 15 min + refresh token 7 days)
  ← Return tokens to browser (httpOnly cookie + response body)
```

**Job Application Flow:**
```
Browser (POST /jobs/:id/apply) → Nginx → API Server (Jobs Module)
  → JWT Middleware (verify token)
  → MySQL (insert application record)
  → Notification Module (emit "new_application" event)
    → Socket.IO (push to job poster's room)
    → SendGrid (email to job poster)
  ← 201 Created response to applicant
```

**Escrow Payment Flow:**
```
Client (initiates payment) → API Server (Wallet Module)
  → SSLCommerz/Stripe (charge client's payment method)
  → Webhook confirmation received by API Server
  → MySQL transaction:
      BEGIN
        DEBIT client wallet (or record direct gateway charge)
        INSERT escrow_ledger (locked funds, job_id, parties)
      COMMIT
  → Provider marks job "Completed"
  → Client approves (or timeout after N days)
  → MySQL transaction:
      BEGIN
        DELETE from escrow_ledger
        CREDIT provider wallet
        INSERT transaction_history
      COMMIT
  → Notification Module → Socket.IO + Email to both parties
```

**Real-Time Chat Flow:**
```
Sender (WS emit "chat:message") → Socket.IO Server
  → Persist message to MySQL (messages table, async)
  → Redis Pub/Sub (broadcast to target room on all nodes)
  → Socket.IO deliver to recipient(s)
  ← Deliver read receipt back to sender
```

---

## 4. Detailed Component Design

### 4.1 Auth Module

**Responsibilities:** Registration, email verification, login, token lifecycle, password reset, role management.

**Key Endpoints:**

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/auth/register` | Register with university email + student ID |
| `POST` | `/api/auth/verify-email` | Confirm email via token link |
| `POST` | `/api/auth/login` | Return access + refresh JWT pair |
| `POST` | `/api/auth/refresh` | Issue new access token using refresh token |
| `POST` | `/api/auth/logout` | Invalidate refresh token in DB/Redis |
| `POST` | `/api/auth/forgot-password` | Send password reset email |
| `POST` | `/api/auth/reset-password` | Validate reset token and update password hash |

**Design Decisions:**

- **JWT over session cookies:** JWTs allow stateless horizontal scaling of API nodes. The trade-off is that JWTs cannot be revoked before expiry; this is mitigated by keeping access tokens short-lived (15 minutes) and storing refresh tokens in a Redis blocklist on logout.
- **bcrypt (cost factor 12):** Deliberately slow to resist brute-force attacks. The cost factor of 12 adds ~300 ms per hash, acceptable for a login endpoint.
- **University email domain whitelist:** Only `@uiu.ac.bd` (or configured institution domains) are accepted at registration, ensuring the platform remains campus-verified. Admins can extend the whitelist.
- **Refresh token rotation:** Each refresh issues a new refresh token and invalidates the previous one, limiting the blast radius of a stolen token.

---

### 4.2 Job & Gig Module

**Responsibilities:** CRUD for job/gig listings; application submission; application lifecycle state machine; dashboard for posters.

**Application State Machine:**
```
SUBMITTED → REVIEWING → INTERVIEW_SCHEDULED → [HIRED | REJECTED]
                      → REJECTED
```

**Key Endpoints:**

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/jobs` | List/search/filter jobs (pagination, skill, budget, type) |
| `POST` | `/api/jobs` | Create a job posting (requires Recruiter or Student role) |
| `GET` | `/api/jobs/:id` | Get job details |
| `PUT` | `/api/jobs/:id` | Update listing (poster only) |
| `DELETE` | `/api/jobs/:id` | Soft-delete listing |
| `POST` | `/api/jobs/:id/apply` | Submit application with cover letter and profile link |
| `GET` | `/api/jobs/:id/applications` | List applicants (poster only) |
| `PATCH` | `/api/jobs/:id/applications/:appId` | Update application status |

**Design Decisions:**

- **Soft delete** (is_deleted flag) preserves referential integrity with applications and allows audit trails.
- **Full-text search** is implemented with MySQL `FULLTEXT` indexes on title and description. If scale demands it, this can be replaced with Elasticsearch without API changes.
- Status changes emit internal events consumed by the Notification Module — the Job module does not directly send notifications, keeping it decoupled.

---

### 4.3 Marketplace Module (Material Exchange & Skill Sharing)

**Responsibilities:** Listing lifecycle for academic materials and peer services; search and filtering; initiating escrow-tied purchase transactions.

**Key Endpoints:**

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/marketplace/materials` | Browse material listings |
| `POST` | `/api/marketplace/materials` | Create a material listing |
| `POST` | `/api/marketplace/materials/:id/purchase` | Initiate an escrow-backed buy/rent transaction |
| `GET` | `/api/marketplace/skills` | Browse skill service listings |
| `POST` | `/api/marketplace/skills` | Offer a skill/tutoring service |
| `POST` | `/api/marketplace/skills/:id/book` | Book a session (creates escrow record) |

**Design Decisions:**

- Material and skill listings share the same listing schema but have a `type` discriminator (MATERIAL | SKILL), simplifying search aggregation across both.
- Images are uploaded to S3/Cloudinary via a pre-signed URL flow: the client calls `POST /api/uploads/presign`, receives a pre-signed S3 URL, uploads directly from the browser (bypassing the API server), then saves the returned URL in the listing record. This keeps large file payloads off the API server.

---

### 4.4 Escrow / Wallet Module

**Responsibilities:** Wallet CRUD; escrow fund locking, release, and dispute; integration with payment gateways; savings goal tracking.

**Escrow Ledger States:**
```
LOCKED → RELEASED (normal completion)
       → DISPUTED → RESOLVED_FOR_CLIENT | RESOLVED_FOR_PROVIDER
       → REFUNDED (if provider cancels or admin rules for client)
```

**Key Endpoints:**

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/wallet` | Wallet balance and transaction history |
| `POST` | `/api/wallet/topup` | Initiate SSLCommerz/Stripe deposit |
| `POST` | `/api/wallet/withdraw` | Request withdrawal to bank/mobile wallet |
| `GET` | `/api/wallet/escrow` | List active escrow records for current user |
| `POST` | `/api/wallet/escrow/:id/release` | Client approves fund release to provider |
| `POST` | `/api/wallet/escrow/:id/dispute` | Raise a dispute for admin review |
| `GET` | `/api/wallet/goals` | List savings goals |
| `POST` | `/api/wallet/goals` | Create a savings goal |
| `PATCH` | `/api/wallet/goals/:id/allocate` | Allocate earnings toward a goal |

**Design Decisions:**

- All wallet mutations use **MySQL transactions with FOR UPDATE row locks** to prevent double-spend race conditions. This is the primary reason MySQL (ACID-compliant) is the authoritative store for financial data rather than a NoSQL alternative.
- **Escrow, not direct transfer:** Funds never move directly from client to provider on service initiation. The escrow record acts as a first-class entity guaranteeing that the provider can only receive funds after client approval, and the client cannot claw back funds arbitrarily once work is in progress.
- Payment gateway webhooks are idempotent: duplicate webhook deliveries (common with SSLCommerz) are deduplicated using a `gateway_txn_id` unique index.

---

### 4.5 Messaging Module

**Responsibilities:** Persistent chat history; one-to-one and group rooms; file attachment delivery; real-time delivery via Socket.IO.

**Socket.IO Event Contract:**

| Event | Direction | Payload |
|---|---|---|
| `chat:join_room` | Client → Server | `{ roomId }` |
| `chat:message` | Client → Server | `{ roomId, content, attachments[] }` |
| `chat:message` | Server → Client | `{ messageId, senderId, content, timestamp, attachments[] }` |
| `chat:typing` | Client → Server | `{ roomId, isTyping }` |
| `chat:read` | Client → Server | `{ roomId, lastReadMessageId }` |
| `notification:push` | Server → Client | `{ type, payload, timestamp }` |

**Design Decisions:**

- Messages are **persisted asynchronously** after Socket.IO delivery to avoid blocking the real-time path. The client optimistically shows the message immediately; a background worker persists it to MySQL.
- For **multi-node Socket.IO scaling**, a Redis Pub/Sub adapter is added. Each Socket.IO node subscribes to Redis channels; when node A receives a message destined for a client connected to node B, it publishes to Redis, and node B delivers it. This is the only change required to scale from one to N Socket.IO processes.
- Chat history is paginated (cursor-based, 50 messages per page) to avoid sending full conversation history on room join.

---

### 4.6 Social Module

**Responsibilities:** Campus feed, group management, post/comment/reaction CRUD, event announcements.

**Key Endpoints (abbreviated):**

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/social/feed` | Personalized feed (followed groups + trending) |
| `POST` | `/api/social/posts` | Create a post (text, poll, event, media) |
| `POST` | `/api/social/posts/:id/reactions` | Like/react to a post |
| `GET/POST` | `/api/social/groups` | List or create interest groups |
| `POST` | `/api/social/groups/:id/join` | Join a group |
| `GET` | `/api/social/events` | Browse campus events |

**Design Decisions:**

- The feed is **pull-based** (fan-out on read) at this scale. When a user loads their feed, the server fetches recent posts from all groups they belong to and merges them chronologically. Fan-out on write (pre-building feeds in Redis) is a future optimization for when follower counts grow large.
- **Soft deletion** for posts: content is cleared but the record is retained for 30 days to support moderation review.

---

### 4.7 Project Module

**Responsibilities:** Project workspace CRUD; task Kanban with state transitions; milestone tracking; member management; file attachments.

**Task State Machine:**
```
TODO → IN_PROGRESS → IN_REVIEW → DONE
     → BLOCKED
```

**Design Decisions:**

- Task state changes are **event-sourced** in a `task_events` table (actor, from_state, to_state, timestamp) for audit trail and portfolio credibility without a full event-sourcing architecture.
- Project files are stored in S3 under a `projects/{projectId}/` prefix, with pre-signed URL upload/download to keep file payloads off the API server.

---

### 4.8 Notification Module

**Responsibilities:** Internal event bus; routing to delivery channels (WebSocket, email, in-app).

**Architecture:**

```
Domain Module
    │  emits typed event (e.g., "application.status_changed")
    ▼
EventEmitter (in-process)
    ├── Socket.IO Handler  → push to connected client
    ├── Email Handler      → SendGrid/Nodemailer queue
    └── DB Handler         → persist to notifications table
```

**Design Decisions:**

- Using Node's built-in `EventEmitter` for in-process event dispatch keeps the V1 implementation simple with zero network hops. The interface is defined as a typed contract so it can be replaced with a message queue (RabbitMQ, Kafka) when the system scales beyond a single process.
- Email delivery is **queued** through an in-memory queue with retry logic (bull/bee-queue backed by Redis in production) to avoid blocking API responses on SMTP latency.

---

## 5. Database Design

### 5.1 Database Choice: MySQL 8

**Justification — SQL over NoSQL:**

| Criterion | MySQL 8 | MongoDB (NoSQL alternative) |
|---|---|---|
| **ACID transactions** | ✅ Native, multi-row | ⚠️ Multi-document transactions added in v4.0, heavier overhead |
| **Escrow/financial consistency** | ✅ Critical requirement | ❌ Not recommended for financial ledgers |
| **Relational integrity** | ✅ Foreign keys, JOINs | ❌ No native referential integrity |
| **Schema flexibility** | ⚠️ Migrations required | ✅ Schema-less |
| **Full-text search** | ✅ FULLTEXT indexes (sufficient for V1) | ✅ Built-in |
| **Familiarity / operational maturity** | ✅ Widely understood | ⚠️ Steeper learning curve |

**CAP Theorem Positioning:** MySQL with synchronous replication favors **Consistency + Partition Tolerance (CP)**. For the financial escrow core, this is the correct choice. The social feed and notification feeds tolerate eventual consistency and could be moved to a more available store (Redis, DynamoDB) in the future without affecting financial correctness.

**Redis complements MySQL** for:
- Session/token caching (sub-millisecond reads)
- Rate-limit counters (atomic INCR/EXPIRE)
- Socket.IO pub/sub adapter
- Leaderboard/ranking for future gamification features

---

### 5.2 Schema Overview

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  USERS & AUTH                                                                 │
│                                                                               │
│  users(id, university_id, email, password_hash, role, full_name,             │
│         profile_picture_url, bio, skills_json, created_at, is_deleted)       │
│                                                                               │
│  refresh_tokens(id, user_id FK, token_hash, expires_at, revoked_at)          │
│                                                                               │
│  email_verifications(id, user_id FK, token, expires_at, verified_at)         │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  JOB MARKETPLACE                                                              │
│                                                                               │
│  job_listings(id, poster_id FK, title, description, budget, type,            │
│               required_skills_json, deadline, status, is_deleted,            │
│               created_at, updated_at)                                        │
│                                                                               │
│  job_applications(id, job_id FK, applicant_id FK, cover_letter,              │
│                   status ENUM, created_at, updated_at)                       │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  MARKETPLACE (MATERIAL & SKILL)                                               │
│                                                                               │
│  listings(id, seller_id FK, type ENUM(MATERIAL,SKILL), title,                │
│           description, price, condition, category, images_json,              │
│           availability_json, status, is_deleted, created_at)                 │
│                                                                               │
│  bookings(id, listing_id FK, buyer_id FK, seller_id FK,                      │
│           scheduled_at, status ENUM, escrow_id FK, created_at)              │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  WALLET & ESCROW                                                              │
│                                                                               │
│  wallets(id, user_id FK UNIQUE, balance DECIMAL(15,2), created_at)           │
│                                                                               │
│  transactions(id, wallet_id FK, type ENUM, amount, reference_id,             │
│               gateway_txn_id UNIQUE, created_at)                             │
│                                                                               │
│  escrow_records(id, client_id FK, provider_id FK, amount,                    │
│                 reference_type ENUM, reference_id, status ENUM,              │
│                 locked_at, released_at, disputed_at, resolved_at)            │
│                                                                               │
│  savings_goals(id, user_id FK, name, target_amount, current_amount,          │
│                target_date, created_at)                                      │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  MESSAGING                                                                    │
│                                                                               │
│  chat_rooms(id, type ENUM(DIRECT,GROUP), name, created_at)                   │
│                                                                               │
│  chat_members(room_id FK, user_id FK, joined_at, last_read_message_id FK)    │
│                                                                               │
│  messages(id, room_id FK, sender_id FK, content TEXT, attachments_json,      │
│           created_at, is_deleted)                                            │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  SOCIAL                                                                       │
│                                                                               │
│  groups(id, name, description, category, privacy ENUM, creator_id FK,        │
│         created_at)                                                          │
│                                                                               │
│  group_members(group_id FK, user_id FK, role ENUM, joined_at)                │
│                                                                               │
│  posts(id, author_id FK, group_id FK NULL, content TEXT, type ENUM,          │
│         media_json, created_at, is_deleted)                                  │
│                                                                               │
│  reactions(post_id FK, user_id FK, type ENUM, created_at)                    │
│                                                                               │
│  comments(id, post_id FK, author_id FK, content TEXT, created_at)            │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  PROJECT MANAGEMENT                                                           │
│                                                                               │
│  projects(id, name, description, owner_id FK, escrow_id FK NULL,             │
│           created_at, status)                                                │
│                                                                               │
│  project_members(project_id FK, user_id FK, role ENUM, joined_at)            │
│                                                                               │
│  tasks(id, project_id FK, title, description, assignee_id FK,                │
│        status ENUM, due_date, created_at, updated_at)                        │
│                                                                               │
│  task_events(id, task_id FK, actor_id FK, from_status, to_status,            │
│              created_at)                                                     │
│                                                                               │
│  milestones(id, project_id FK, title, due_date, is_completed)                │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  NOTIFICATIONS                                                                │
│                                                                               │
│  notifications(id, user_id FK, type, title, body, reference_type,            │
│                reference_id, is_read, created_at)                            │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 5.3 Indexing and Query Optimization

| Table | Index | Justification |
|---|---|---|
| `users` | UNIQUE(email), UNIQUE(university_id) | Login lookups; prevents duplicate registration |
| `job_listings` | FULLTEXT(title, description), INDEX(status, deadline), INDEX(poster_id) | Search queries; filtering active jobs; poster dashboard |
| `job_applications` | INDEX(job_id, status), INDEX(applicant_id) | Poster application dashboard; applicant "my applications" view |
| `listings` | FULLTEXT(title, description), INDEX(type, status), INDEX(seller_id) | Marketplace search; type filtering; seller dashboard |
| `messages` | INDEX(room_id, created_at DESC) | Paginated chat history per room |
| `escrow_records` | INDEX(client_id), INDEX(provider_id), INDEX(status) | Wallet module queries; admin dispute dashboard |
| `transactions` | UNIQUE(gateway_txn_id), INDEX(wallet_id, created_at) | Idempotent webhook dedup; transaction history pagination |
| `notifications` | INDEX(user_id, is_read, created_at) | Notification badge count; sorted notification feed |
| `tasks` | INDEX(project_id, status), INDEX(assignee_id) | Kanban board queries; "my tasks" view |

**Query Optimization Strategies:**

- **Pagination:** All list endpoints use cursor-based pagination (`WHERE id > :cursor ORDER BY id LIMIT :size`) rather than OFFSET pagination, avoiding full table scans as data grows.
- **N+1 Prevention:** Related entities (e.g., applicant profiles alongside applications) are fetched using JOINs or batched WHERE IN queries at the repository layer, never in a loop.
- **Read Replica:** Long-running read queries (feed generation, report queries) are routed to the MySQL read replica to reduce load on the primary.
- **Connection Pooling:** `mysql2` connection pool (min: 5, max: 20 connections per Node.js process) avoids connection overhead.

---

## 6. Scalability and Performance

### 6.1 Horizontal vs. Vertical Scaling Strategy

**Phase 1 (Current — up to ~1,000 concurrent users):**
- Single API server node (vertically scaled: 2–4 vCPUs, 4–8 GB RAM).
- Single MySQL instance.
- Single Redis instance.
- Nginx in front for SSL termination and static file serving.

**Phase 2 (~1,000–10,000 concurrent users):**
- **Horizontal API scaling:** Add 2–3 API server nodes behind Nginx upstream pool (round-robin load balancing). Node.js is stateless (JWT-based auth, no in-process sessions), so nodes scale trivially.
- **MySQL read replica:** Add one read replica; route all SELECT queries from feed, marketplace browse, and notification history to the replica. Write queries (INSERT, UPDATE, DELETE) remain on the primary.
- **Redis Socket.IO adapter:** Enable `@socket.io/redis-adapter` so WebSocket events route correctly across all Socket.IO nodes.
- **Database connection pool scaling:** Increase pool size per node and consider ProxySQL for connection multiplexing.

**Phase 3 (10,000+ users — post-MVP):**
- Extract the highest-load modules (Messaging, Social Feed) into independent microservices.
- Introduce a message queue (e.g., RabbitMQ or Kafka) to decouple Notification delivery from the request path.
- Consider sharding the `messages` table by `room_id` range or moving it to Cassandra (optimized for append-heavy time-series chat data).
- Evaluate managed databases (AWS RDS with Multi-AZ) for automatic failover.

**Why horizontal over vertical scaling?**  
Vertical scaling (adding CPU/RAM) has a ceiling and requires downtime for hardware changes. Horizontal scaling of stateless Node.js processes can be done live (deploy new nodes, Nginx picks them up). The additional complexity (session stickiness for WebSockets, distributed caching) is a worthwhile trade-off for resilience and cost-elasticity.

---

### 6.2 Caching Strategy

| Layer | What is Cached | TTL | Invalidation |
|---|---|---|---|
| **CDN (CloudFront/Cloudflare)** | Static JS/CSS bundles, images | 365 days (content-hashed filenames) | Deploy new bundle with new hash |
| **Redis (Server-side)** | JWT blocklist (revoked tokens); rate-limit counters; frequently accessed user profiles; job listing browse results | Tokens: until expiry; Rate limits: 1 min window; Profiles: 5 min; Listings: 2 min | Cache-aside: expire on write/update |
| **React Query (Client-side)** | API responses for marketplace listings, profile data, notification count | Stale-while-revalidate: 30 s | Invalidated on mutation (post, apply, book) |
| **Browser Cache** | Fonts, vendor JS bundles | Max-Age: 30 days | URL fingerprinting via Vite build hash |

**Cache-aside pattern** (the standard for Redis): The API server checks Redis first; on miss, queries MySQL and populates Redis. On mutations, the corresponding Redis key is deleted (not updated), forcing the next read to repopulate from the authoritative source.

**What is NOT cached:** Escrow balances, wallet balances, and active transaction records — these are always read from the MySQL primary to guarantee correctness.

---

### 6.3 Load Balancing Approach

Nginx is configured as an **upstream load balancer** using the `upstream` block with the `least_conn` algorithm (routes new requests to the node with the fewest active connections), which is fairer than round-robin when requests have variable processing times.

```nginx
upstream earnlearn_api {
    least_conn;
    server api_node_1:3000;
    server api_node_2:3000;
    server api_node_3:3000;
    keepalive 32;
}
```

**WebSocket Sticky Sessions:** Socket.IO requires that after a connection is established, subsequent polling/upgrade requests go to the same server node. Nginx is configured with `ip_hash` for the `/socket.io/` location block, or the Socket.IO Redis adapter is used to eliminate the stickiness requirement entirely.

**Health Checks:** Nginx `health_check` module (or a sidecar process) polls `GET /health` on each API node every 5 seconds; unhealthy nodes are automatically removed from the upstream pool.

---

### 6.4 CDN Usage

- **Static Assets:** Vite produces content-hashed bundles (`main.abc123.js`). These are pushed to a CDN origin bucket (S3) and served from CloudFront/Cloudflare edge nodes globally.
- **User-Uploaded Media:** Profile photos, listing images, and portfolio files are stored in AWS S3 and served via a CloudFront distribution with a signed-URL policy, preventing hotlinking and unauthorized access.
- **API Responses:** API endpoints are NOT cached at the CDN layer (dynamic, user-specific data). The CDN only handles static and media assets.

---

## 7. Reliability and Fault Tolerance

### 7.1 Failure Scenarios and Handling

| Failure Scenario | Detection | Response |
|---|---|---|
| **API node crash** | Nginx health check fails on /health | Nginx removes node from upstream; remaining nodes absorb traffic; auto-restart via PM2 or Kubernetes pod restart policy |
| **MySQL primary failure** | Application DB connection error; monitoring alert | Promote read replica to primary (manual or automated with Orchestrator/MHA); update application DSN via environment variable; target RTO: < 10 min |
| **Redis failure** | Connection refused errors in logs | API falls back to in-memory rate-limit counters (degraded but functional); Socket.IO loses multi-node pub/sub (chat works on single node); target: Redis Sentinel for automatic failover |
| **SSLCommerz/Stripe gateway timeout** | HTTP 5xx or timeout on payment API call | Return error to user with "retry later" message; no funds deducted (no partial commit); webhook-based eventual consistency means successful charges will be confirmed asynchronously |
| **S3/Cloudinary unavailable** | File upload/download 5xx | Image uploads fail gracefully (non-critical path); existing media served from CDN cache; no impact on core transactional features |
| **SendGrid/Email failure** | SMTP error or 4xx API response | Retry with exponential backoff (max 3 attempts, 1/5/15 min intervals); if all retries fail, log and alert; registration/verification emails are critical and must succeed before user can proceed |
| **Memory leak / high CPU in Node.js process** | Monitoring alert on CPU > 80% for > 5 min | PM2 auto-restart after memory threshold exceeded; Nginx routes traffic to other nodes during restart; alerts notify ops team |

---

### 7.2 Replication and Redundancy Strategy

- **MySQL Replication:** One primary + one read replica using standard MySQL asynchronous replication (`binlog`). For Phase 2, switch to semi-synchronous replication to limit replication lag.
- **Redis Sentinel:** Three Redis nodes (one primary, two replicas) with Sentinel for automatic failover without manual intervention.
- **API Server Redundancy:** Minimum 2 API nodes in production; deployment rolling updates ensure at least one node is always serving traffic.
- **File Storage Redundancy:** AWS S3 Standard storage class provides 99.999999999% (11 nines) durability across multiple Availability Zones natively.

---

### 7.3 Disaster Recovery and Backup Plan

| Asset | Backup Method | RPO | RTO |
|---|---|---|---|
| **MySQL database** | Daily `mysqldump` to S3; binary log streaming for point-in-time recovery | 24 hours (daily) → 5 min (binlog) | < 1 hour |
| **Redis data** | RDB snapshot every 15 minutes; AOF persistence | 15 minutes | < 5 minutes |
| **User-uploaded files** | S3 Cross-Region Replication to a backup region | Near-real-time | < 30 minutes |
| **Application code** | Git repository on GitHub (already version-controlled) | 0 (every commit) | < 15 min (redeploy) |

**Backup Verification:** A monthly automated restore drill reads the latest S3 backup, restores to a staging database, and runs smoke tests to confirm integrity.

---

## 8. Security Design

### 8.1 Authentication and Authorization Model

**Authentication:** JWT-based stateless authentication.
- Access tokens (15 min TTL) carry `userId`, `role`, and `iat`/`exp` claims, signed with HS256 using a 256-bit secret stored in environment variables.
- Refresh tokens (7 day TTL) are opaque random strings hashed with SHA-256 and stored in MySQL (`refresh_tokens` table). The raw token is sent to the client in an `httpOnly`, `Secure`, `SameSite=Strict` cookie.
- On every protected API request, JWT middleware verifies the signature and expiry, then checks the Redis blocklist for revoked tokens.

**Authorization:** Role-Based Access Control (RBAC) with three roles:

| Role | Capabilities |
|---|---|
| `STUDENT` | Register, browse, post listings/jobs, apply, chat, manage own wallet |
| `RECRUITER` | Everything STUDENT can do, plus post and manage job listings with Recruiter-level visibility |
| `ADMIN` | Full platform access; moderation; dispute resolution; user management; content removal |

Authorization middleware is applied at the route level:
```
router.patch('/escrow/:id/release', authenticate, authorize(['STUDENT','RECRUITER']), handler)
router.delete('/users/:id', authenticate, authorize(['ADMIN']), handler)
```

Resource-level authorization (e.g., "only the job poster can update this listing") is enforced inside the handler by comparing `req.user.id` against the record's `poster_id`.

---

### 8.2 Data Encryption

| Data State | Mechanism |
|---|---|
| **In transit** | TLS 1.2+ enforced at Nginx; HSTS header (`max-age=31536000; includeSubDomains`); certificates via Let's Encrypt (auto-renewed) |
| **Passwords at rest** | bcrypt with cost factor 12; raw passwords never logged or stored |
| **Refresh tokens at rest** | SHA-256 hash of the token stored in DB; raw token only in client cookie |
| **Database at rest** | MySQL data directory on an AES-256 encrypted EBS volume (AWS) or equivalent VPS disk encryption |
| **File storage at rest** | S3 server-side encryption (SSE-S3 or SSE-KMS) enabled on the upload bucket |
| **Sensitive env config** | Stored in environment variables (never in source code); managed via `.env` files in development and cloud secret manager (AWS Secrets Manager, Render Secrets) in production |

---

### 8.3 Input Validation and Injection Prevention

- All user inputs are validated using **Zod** (TypeScript-first schema validation) at the API layer before reaching business logic. Invalid input returns 400 with a structured error response.
- SQL queries use **`mysql2` prepared statements** exclusively; raw string interpolation into SQL is prohibited by code review policy.
- Uploaded file types are validated server-side by MIME type (not just extension) using `file-type` library. Allowed types: `image/jpeg`, `image/png`, `image/webp`, `application/pdf`. Maximum file size: 5 MB.
- HTML content in posts and messages is sanitized with **`DOMPurify`** on the client and **`sanitize-html`** on the server to prevent stored XSS.

---

### 8.4 Rate Limiting and DDoS Protection

| Endpoint Group | Rate Limit | Window |
|---|---|---|
| `POST /api/auth/login` | 10 attempts | 15 min per IP |
| `POST /api/auth/register` | 5 attempts | 1 hour per IP |
| `POST /api/auth/forgot-password` | 3 attempts | 1 hour per email |
| All other authenticated endpoints | 300 requests | 1 min per user |
| All unauthenticated endpoints | 60 requests | 1 min per IP |

Rate limits are enforced using **`express-rate-limit`** backed by Redis (`rate-limit-redis` store) for consistency across multiple API nodes. Exceeded limits return `429 Too Many Requests` with a `Retry-After` header.

**DDoS mitigation at the edge:** Cloudflare (or AWS WAF) sits in front of Nginx, providing volumetric DDoS protection, bot detection, and IP reputation filtering. The Nginx layer adds a second defense with `limit_req_zone` directives for burst control.

---

## 9. Monitoring and Observability

### 9.1 Logging Strategy

**Log Levels:** `error` → `warn` → `info` → `debug` (production uses `info` and above).

**Logging Library:** `winston` with structured JSON output.

Every log entry includes:
```json
{
  "timestamp": "2026-04-19T05:00:00.000Z",
  "level": "info",
  "service": "earnlearn-api",
  "requestId": "uuid-v4",
  "userId": "123",
  "method": "POST",
  "path": "/api/jobs/42/apply",
  "statusCode": 201,
  "durationMs": 87,
  "message": "Job application submitted"
}
```

**Request ID propagation:** Nginx generates a unique `X-Request-ID` header for each incoming request. The API server reads this header and attaches it to every log line and response, enabling end-to-end trace correlation across Nginx and API logs without a full distributed tracing system.

**Log Storage:** Logs are streamed to a centralized service (CloudWatch Logs on AWS, or a self-hosted ELK stack: Elasticsearch + Logstash + Kibana) for storage, search, and alerting. Raw logs are retained for 30 days; compressed archives for 1 year.

**What is always logged:**
- All authentication events (login, logout, failed attempts, token refresh)
- All financial transactions (wallet top-ups, escrow lock/release, withdrawals)
- All admin actions
- All unhandled exceptions (with stack trace)
- Rate limit violations

**What is never logged:**
- Passwords or password hashes
- Raw JWT tokens or session tokens
- Full payment card details
- Personally Identifiable Information (PII) beyond user ID in non-auth logs

---

### 9.2 Metrics and Alerting

**Metrics collection:** `prom-client` exposes a `/metrics` Prometheus endpoint from each API node. A Prometheus server scrapes all nodes; Grafana provides dashboards.

**Key Metrics:**

| Metric | Type | Alert Threshold |
|---|---|---|
| `http_request_duration_ms` (p50, p95, p99) | Histogram | p99 > 1 s for 5 min |
| `http_requests_total` by status code | Counter | 5xx error rate > 1% for 5 min |
| `active_websocket_connections` | Gauge | N/A (capacity planning) |
| `mysql_query_duration_ms` (p95) | Histogram | p95 > 300 ms for 5 min |
| `redis_commands_duration_ms` (p95) | Histogram | p95 > 50 ms for 5 min |
| `escrow_funds_locked_total` | Gauge | N/A (business metric) |
| `node_heap_used_bytes` | Gauge | > 80% of max heap |
| `process_cpu_seconds_total` | Counter | CPU > 80% sustained for 5 min |

**Alerting:** Alertmanager sends alerts to a Slack channel (`#earnlearn-alerts`) and email on-call. Critical alerts (database down, payment gateway unreachable) page the on-call engineer.

---

### 9.3 Distributed Tracing

In the V1 modular monolith, **correlation via Request ID** (described in §9.1) provides sufficient tracing — every log line for a single user request shares the same `requestId`, making it trivial to reconstruct a full request lifecycle in Kibana/CloudWatch Insights.

For Phase 2 (when services are split), **OpenTelemetry** will be introduced:
- The Node.js `@opentelemetry/sdk-node` auto-instruments Express, MySQL, and Redis clients.
- Traces are exported to **Jaeger** or **Tempo** (Grafana stack).
- Each span captures service name, operation, duration, and error status.
- Trace IDs are propagated in HTTP headers (`traceparent` per W3C Trace Context standard) between services.

**Reason for deferring full tracing to Phase 2:** In a monolith, a stack trace already gives full call context. Adding a tracing agent to every function call in a monolith adds overhead (CPU, memory, egress to the tracing backend) without meaningful benefit over structured logging with Request IDs.

---

## 10. Deployment Architecture

### 10.1 Cloud Infrastructure Overview

**Target Platform:** AWS (primary) or Render/Railway (budget-friendly alternative for academic phase).

**AWS Architecture:**

```
                    ┌───────────────────────────────────┐
                    │         Route 53 (DNS)             │
                    └──────────────┬────────────────────┘
                                   │
                    ┌──────────────▼────────────────────┐
                    │   CloudFront CDN                   │
                    │   (Static assets + Media files)    │
                    └──────────────┬────────────────────┘
                                   │
                    ┌──────────────▼────────────────────┐
                    │   Application Load Balancer (ALB)  │
                    └──────────┬────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
        ┌──────────┐    ┌──────────┐    ┌──────────┐
        │  EC2 /   │    │  EC2 /   │    │  EC2 /   │
        │  ECS     │    │  ECS     │    │  ECS     │
        │  Node 1  │    │  Node 2  │    │  Node 3  │
        └──────────┘    └──────────┘    └──────────┘
               │
               ├──────────────────────────────────────┐
               ▼                                      ▼
  ┌────────────────────────┐          ┌──────────────────────────┐
  │  RDS MySQL (Multi-AZ)  │          │  ElastiCache Redis        │
  │  Primary + Read Replica│          │  (Cluster mode)          │
  └────────────────────────┘          └──────────────────────────┘
               │
               ▼
  ┌────────────────────────┐
  │  S3 Bucket             │
  │  (User media, backups) │
  └────────────────────────┘
```

**Key Infrastructure Decisions:**

- **RDS MySQL Multi-AZ** over self-managed EC2 MySQL: Automatic failover (< 60 s), managed backups, OS-level patching handled by AWS. The higher cost (~2× single-AZ) is justified for financial data availability.
- **ECS (Fargate) over raw EC2 for API nodes:** No OS management; define a task definition (CPU/memory), and ECS handles scheduling. Scaling is a single `update-service --desired-count N` command.
- **ALB over Nginx for Phase 2:** ALB natively understands HTTP/2, WebSocket upgrades, and health checks, removing the need to maintain a custom Nginx config for load balancing.

---

### 10.2 CI/CD Pipeline

```
Developer pushes feature branch
         │
         ▼
  GitHub Pull Request
         │
         ▼
  ┌──────────────────────────────┐
  │  GitHub Actions CI Workflow   │
  │                              │
  │  1. npm ci                   │
  │  2. ESLint + TypeScript check │
  │  3. Unit tests (Jest)        │
  │  4. Integration tests        │
  │  5. Docker image build       │
  │  6. Container security scan  │
  └──────────┬───────────────────┘
             │ All checks pass
             ▼
     Code Review Approval
             │
             ▼
     Merge to main branch
             │
             ▼
  ┌──────────────────────────────┐
  │  GitHub Actions CD Workflow   │
  │                              │
  │  1. Build Docker image       │
  │  2. Push to ECR              │
  │  3. Run DB migrations        │
  │     (migrate:deploy)         │
  │  4. Rolling deploy to ECS    │
  │     (--deployment-config     │
  │      minimum 50% healthy)    │
  │  5. Smoke test prod endpoint │
  │  6. Notify Slack on success  │
  └──────────────────────────────┘
```

**Branching Strategy:** `main` (production) ← `develop` (staging) ← `feature/*` branches. Hotfixes branch from `main` directly.

**Database Migrations:** Managed with `db-migrate` or Prisma Migrate. Migrations run as a pre-deploy step in the CD pipeline. All migrations must be backward-compatible (no column drops or renames in the same release as code that depends on the new schema) to allow safe rollback.

**Rollback Plan:** ECS task definitions are versioned. If a deployment introduces a critical error (monitored by the 5xx error rate alert), redeploying the previous task definition revision takes < 2 minutes.

---

### 10.3 Containerization and Orchestration

**Docker:**

Every component is containerized. The `Dockerfile` for the API server follows a multi-stage build:

```
Stage 1 (builder): node:20-alpine → npm ci → tsc → prune dev deps
Stage 2 (runner):  node:20-alpine → copy dist + node_modules → USER node (non-root) → EXPOSE 3000
```

Multi-stage builds keep the production image lean (~150 MB vs ~800 MB for a naive build).

**Docker Compose (local development):**
```yaml
services:
  api:       # Node.js API server
  mysql:     # MySQL 8 with init scripts
  redis:     # Redis 7
  nginx:     # Reverse proxy to api
```

**Container Orchestration: ECS Fargate (production) / Docker Compose (development)**

- **ECS Fargate** is chosen over a self-managed Kubernetes cluster because:
  - No EC2 nodes to manage, patch, or scale.
  - Cheaper at small scale (< 10 containers) than EKS control plane fees.
  - Native integration with ALB, ECR, IAM, and CloudWatch.
  - Kubernetes is the recommended path if the platform expands to true microservices with service mesh requirements (Istio, Envoy).

**Resource Limits per API container:**
- CPU: 0.5 vCPU (burstable to 1 vCPU)
- Memory: 512 MB (alert threshold: 80%)

Auto-scaling policy: Scale out at average CPU > 60% for 3 minutes; scale in at average CPU < 30% for 10 minutes; min 2 tasks, max 6 tasks.

---

## 11. Future Improvements and Known Limitations

### 11.1 Known Limitations

| Limitation | Impact | Mitigation in Roadmap |
|---|---|---|
| **No offline mode** | Users in low-connectivity areas cannot use the platform | Service Worker + IndexedDB for critical read-only views (V2) |
| **Full-text search is MySQL FULLTEXT** | Limited relevance ranking; no fuzzy matching; will become a bottleneck at high data volume | Migrate to Elasticsearch or OpenSearch with a write-through sync when listings exceed ~100,000 rows |
| **Single-region deployment** | Latency for users outside the primary region; no geographic redundancy | Multi-region AWS with Route 53 latency-based routing (Phase 3) |
| **No mobile-native app in V1** | PWA has limitations (push notifications on iOS, camera access) | React Native app using the same API (Phase 2) |
| **No AI-powered matching** | Job/skill recommendations are filter-based only | Collaborative filtering model trained on booking/application history (Phase 3) |
| **Monolith deployment unit** | A bug in the Social module can affect the Payment module; deploys are all-or-nothing | Module extraction to microservices as load demands (Phase 2+) |
| **Payment: SSLCommerz only** | International students cannot use the wallet | Stripe integration for international card payments (Phase 2) |
| **No video call infrastructure** | Voice/video tutoring depends on third-party embeds | WebRTC peer-to-peer (native) or Twilio Video integration (Phase 2) |

---

### 11.2 Future Enhancements

1. **AI-Driven Job and Skill Matching**  
   A recommendation engine using collaborative filtering and NLP on job/skill descriptions will surface the most relevant opportunities to each student based on their profile, activity history, and peer behavior.

2. **Mobile App (React Native)**  
   A dedicated Android/iOS app sharing the same REST/Socket.IO API, enabling native push notifications (via Firebase Cloud Messaging), camera-based material listing uploads, and biometric authentication.

3. **Gamification and Reputation System**  
   Badges, experience points, and leaderboards to incentivize quality listings, timely deliveries, and community contributions. Reputation scores feed directly into job application ranking.

4. **University Partnership API**  
   An OAuth2-based SSO integration with university identity providers (SAML, Google Workspace) to eliminate manual student ID verification and streamline onboarding.

5. **Multi-Institution Expansion**  
   Extend the university email whitelist model to support a federation of campuses. Students from different institutions can collaborate on inter-campus projects while each institution's marketplace remains isolated.

6. **Advanced Financial Tools**  
   Automated savings contributions (round-up on every transaction), spending analytics with category breakdowns, and integration with national mobile banking services (bKash, Nagad) for seamless withdrawals.

7. **Microservices Migration**  
   When any single module's load justifies independent scaling (likely Messaging first), extract it as a standalone service behind an API Gateway, using the existing internal event contract as the service interface.

8. **GDPR Self-Service Portal**  
   A user-facing privacy dashboard for viewing all stored data, requesting data export (JSON), and initiating account deletion with a 30-day grace period.

---

*End of System Design Document*

---

> **Document Version:** 1.0  
> **Last Updated:** April 2026  
> **Status:** Final — Academic Submission (CSE 3412, UIU)
