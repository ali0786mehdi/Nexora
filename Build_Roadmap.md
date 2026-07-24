# 🛠️ Nexora — Build Roadmap

A step-by-step execution plan for building Nexora from an empty folder to an enterprise-hardened, production-ready real-time chat platform.

> **Rule of thumb:** finish and test one phase before starting the next. Commit after every numbered step, not just every phase.

---

## 📅 Timeline Overview

| Phase | Focus | Duration | Status |
|-------|-------|----------|--------|
| [Phase 0](#-phase-0--environment--project-setup-day-0) | Environment & Project Setup | Day 0 | ⬜ Not Started |
| [Phase 1](#-phase-1--foundation-week-1) | Foundation | Week 1 | ⬜ Not Started |
| [Phase 2](#-phase-2--core-real-time-mvp-weeks-23) | Core Real-Time MVP | Weeks 2–3 | ⬜ Not Started |
| [Phase 3](#-phase-3--scaling-with-redis-week-4) | Scaling with Redis | Week 4 | ⬜ Not Started |
| [Phase 4](#-phase-4--background-jobs-week-5) | Background Jobs | Week 5 | ⬜ Not Started |
| [Phase 5](#-phase-5--files--notifications-week-6) | Files & Notifications | Week 6 | ⬜ Not Started |
| [Phase 6](#-phase-6--production-hardening-week-7) | Production Hardening | Week 7 | ⬜ Not Started |

Update the **Status** column as you go: `⬜ Not Started` → `🟨 In Progress` → `✅ Done`

---

## 🔹 Phase 0 — Environment & Project Setup (Day 0)

**Goal of this phase:** a folder that runs `npm run dev` and does nothing yet, but is properly structured.

- [ ] Install Node.js v18+, verify with `node -v`
- [ ] Install Docker Desktop (used for Postgres, MongoDB, Redis instead of installing each locally)
- [ ] Create the project folder and run `npm init -y`
- [ ] Install TypeScript and initialize config: `typescript`, `ts-node`, `tsx`, `@types/node`, then generate `tsconfig.json`
- [ ] Set up folder structure exactly as in the README (`src/config`, `src/lib`, `src/routes`, etc.) — create empty placeholder files so the structure exists early
- [ ] Initialize git, add `.gitignore` (`node_modules`, `.env`, `dist`)
- [ ] Set up ESLint + Prettier for consistent code style
- [ ] Create `.env.example` with just the Phase 1 variables
- [ ] Push empty skeleton to GitHub as commit #1

---

## 🔹 Phase 1 — Foundation (Week 1)

**Checkpoint:** `npm run dev` boots, `/api/health` returns 200 with both DBs connected, migrations run cleanly.

### Step 1: Spin up databases via Docker
- [ ] Run Postgres, MongoDB, Redis containers using the docker commands in the README
- [ ] Verify each connects using a GUI tool (TablePlus/Compass) or CLI ping

### Step 2: Express server bootstrap
- [ ] Build `src/app.ts` (Express app config) and `src/server.ts` (HTTP listener) separately — this separation matters later for testing
- [ ] Add `/api/health` endpoint that pings both Postgres and MongoDB and returns status

### Step 3: Environment validation
- [ ] Build `src/config/env.ts` using Zod to validate all env vars at boot — fail fast if something's missing

### Step 4: Database clients
- [ ] Set up Prisma, define initial schema (User, Channel, Membership models) in `schema.prisma`
- [ ] Run first migration
- [ ] Set up Mongoose connection singleton in `src/lib/mongoose.ts` (Message model comes in Phase 2)

### Step 5: Error handling middleware
- [ ] Centralized error handler (`error.middleware.ts`) so every route returns consistent JSON error shapes

---

## 🔹 Phase 2 — Core Real-Time MVP (Weeks 2–3)

> This is the heart of the project — build it in this **exact sub-order**.

**Checkpoint:** Two authenticated users can register, join a channel, exchange real-time messages, see typing indicators, and pull paginated history. This is your working MVP — a legitimate portfolio piece even if you stopped here.

### 1. Auth (REST first, no sockets yet)
- [ ] Password hashing (bcrypt), register/login/logout/refresh endpoints
- [ ] JWT access + refresh token pair, refresh stored as httpOnly cookie
- [ ] Auth middleware to protect routes

### 2. Channels (REST)
- [ ] Create channel, list channels, join channel endpoints
- [ ] Membership table enforced via Prisma relations

### 3. Message model (MongoDB)
- [ ] Define `message.model.ts` schema: sender, channelId, content, timestamps, reactions placeholder
- [ ] Build paginated history endpoint (`GET /channels/:id/messages`) — use cursor-based pagination, not offset, since you'll need this pattern to scale later

### 4. Socket.io server
- [ ] Attach Socket.io to the same HTTP server
- [ ] Build socket auth middleware — verify JWT on connection handshake, reject unauthenticated sockets
- [ ] Define event constants file (`events.ts`) before writing handlers, so names are consistent

### 5. Real-time messaging handlers
- [ ] `channel:join` / `channel:leave` — join/leave Socket.io rooms
- [ ] `message:send` → validate, persist to MongoDB, broadcast `message:new` to room
- [ ] `dm:send` → persist, emit `dm:new` directly to target user's socket(s)

### 6. Presence
- [ ] Track online/offline in-memory first (Map of userId → socket count)
- [ ] Emit `presence:update` on connect/disconnect

### 7. Typing indicators & read receipts
- [ ] `typing:start` / `typing:stop` → broadcast `typing:update`, no persistence needed
- [ ] Read receipts: store `lastReadAt` per user per channel, simplest as a Mongo/Postgres field updated on a `message:read` event

### ⚠️ Before moving on
- [ ] Test this phase manually with two clients — use Postman's WebSocket support or `wscat` in two terminals to prove real messages flow both ways **before** writing any frontend

---

## 🔹 Phase 3 — Scaling with Redis (Week 4)

**Checkpoint:** you can horizontally scale your socket servers and messages still flow correctly across instances.

- [ ] Replace in-memory presence tracking with Redis (so it survives restarts and works across instances)
- [ ] Install `@socket.io/redis-adapter`, wire it into Socket.io setup — this is what lets multiple Node instances share broadcast events via Pub/Sub
- [ ] Move session/refresh-token storage into Redis instead of only Postgres, if you want faster revocation checks
- [ ] **Prove it works:** run two instances of your server on different ports behind a simple round-robin (or just two terminal tabs), connect one client to each, send a message from one — confirm the other receives it via Redis Pub/Sub, not direct memory

---

## 🔹 Phase 4 — Background Jobs (Week 5)

**Checkpoint:** killing your main server doesn't lose queued jobs; worker processes them independently and you can watch jobs move through queued → active → completed (use Bull Board for a visual dashboard).

- [ ] Set up BullMQ with Redis as the queue backend
- [ ] Build `email.queue.ts` and `email.worker.ts` — worker runs as a separate process (`npm run worker`)
- [ ] Trigger email queue job when a DM is sent to an offline user ("missed message" notification)
- [ ] Add a cron job (BullMQ repeatable jobs) for cleanup — e.g., purging expired refresh tokens, or a daily digest email job
- [ ] Add retry/backoff strategy and a dead-letter approach for failed jobs (log + alert)

---

## 🔹 Phase 5 — Files & Notifications (Week 6)

**Checkpoint:** users can share images/files in a channel, and get notified even when their socket isn't connected.

- [ ] Set up AWS S3 (or Cloudinary if you want zero AWS setup friction) bucket + credentials
- [ ] Build a signed-upload-URL endpoint so files upload directly client → S3, not through your server
- [ ] Extend the Message model to support attachments (URL, type, size)
- [ ] Add Server-Sent Events endpoint for one-way notification streams (useful for browser tab notifications without full socket overhead)
- [ ] Add push notification support (web push via VAPID keys, or a service like Firebase Cloud Messaging)

---

## 🔹 Phase 6 — Production Hardening (Week 7)

**Checkpoint:** `docker-compose up` alone brings up the entire stack from a clean machine, tests pass in CI, and the app has the baseline hardening expected of a real production service.

- [ ] Write `Dockerfile` (multi-stage build: install → build → slim runtime image)
- [ ] Write `docker-compose.yml` wiring app + Postgres + MongoDB + Redis + worker together
- [ ] Add rate limiting (`express-rate-limit` for REST, custom middleware for socket events) to prevent abuse
- [ ] Add Helmet for security headers, strict CORS using `CLIENT_URL`
- [ ] Add structured logging (pino or winston) and request IDs for traceability
- [ ] Add input validation everywhere via Zod schemas (if not already done per-route)
- [ ] Write integration tests for auth, channels, and socket events (Jest/Vitest + socket.io-client for test sockets)
- [ ] Set up CI (GitHub Actions): lint → test → build on every push
- [ ] Add basic monitoring: health check endpoint feeding into an uptime tool, plus Redis/DB connection alerts

---

## ✅ How to Actually Execute This

- **Build and fully test one phase before starting the next** — resist adding Phase 3 code while still in Phase 2
- **Commit after each numbered step**, not just each phase — small commits make it easy to see your own progress and roll back
- **Update the status column** in this file (⬜ → 🟨 → ✅) as you complete each row — turns this roadmap into a live progress tracker
- **Manually test sockets with `wscat`/Postman** before ever building a frontend — don't let frontend debugging mask backend bugs
