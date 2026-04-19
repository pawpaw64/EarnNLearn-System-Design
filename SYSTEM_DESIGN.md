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


---


---


---


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


---


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
