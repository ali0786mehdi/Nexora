# ⚡ Nexora

A production-grade real-time chat platform built with Node.js, TypeScript, Socket.io, and MongoDB.

Nexora handles bidirectional communication at scale — channels, direct messages, file sharing, typing indicators, read receipts, and background job processing. Built as a portfolio project to demonstrate event-driven architecture, WebSocket scaling with Redis Pub/Sub, and async job queues.

![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=flat-square&logo=node.js&logoColor=white)
![Socket.io](https://img.shields.io/badge/Socket.io-4.x-010101?style=flat-square&logo=socket.io&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-7.x-47A248?style=flat-square&logo=mongodb&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7+-DC382D?style=flat-square&logo=redis&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

---

## Overview

Nexora is a backend-first real-time chat system. It starts with a clean MVP — register, login, send messages in real time — and progressively adds production-grade features: Redis-powered WebSocket scaling, S3 file uploads, BullMQ background jobs, and push notifications.

The architecture is intentionally built to handle what breaks most chat apps at scale: multiple server instances, offline message queuing, and connection state recovery.

---

## MVP — What ships first

The MVP is a fully working real-time chat backend with these capabilities:

- User registration and login with JWT authentication
- Create and join channels (public workspaces)
- Send and receive messages in real time via WebSocket
- Direct messages between two users
- Online / offline presence indicators
- Paginated message history from MongoDB

Everything above Phase 2 is a production enhancement layered on top of the MVP.

---

## Full Feature Set

| Feature | Phase | Status |
|---------|-------|--------|
| User auth (register, login, JWT) | 1 | ⬜ Upcoming |
| Channel creation and membership | 1 | ⬜ Upcoming |
| Real-time messaging via WebSocket | 2 | ⬜ Upcoming |
| Direct messages (DMs) | 2 | ⬜ Upcoming |
| Online/offline presence | 2 | ⬜ Upcoming |
| Paginated message history | 2 | ⬜ Upcoming |
| Typing indicators | 2 | ⬜ Upcoming |
| Read receipts | 2 | ⬜ Upcoming |
| Redis Pub/Sub (multi-server scaling) | 3 | ⬜ Upcoming |
| BullMQ background job queues | 4 | ⬜ Upcoming |
| Email notifications (missed messages) | 4 | ⬜ Upcoming |
| Cron jobs (cleanup, digest emails) | 4 | ⬜ Upcoming |
| File and image uploads (S3) | 5 | ⬜ Upcoming |
| Push notifications | 5 | ⬜ Upcoming |
| Server-Sent Events (SSE) | 5 | ⬜ Upcoming |
| Docker + docker-compose | 6 | ⬜ Upcoming |
| Rate limiting and security hardening | 6 | ⬜ Upcoming |

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime | Node.js 18+ |
| Language | TypeScript 5.x |
| Framework | Express.js |
| Real-time | Socket.io 4.x |
| Database | MongoDB 7 + Mongoose |
| Cache / Scaling | Redis 7 (Pub/Sub + sessions) |
| Job Queue | BullMQ |
| File Storage | AWS S3 / Cloudinary |
| Email | Nodemailer |
| Auth | JWT (access + refresh tokens) |
| Validation | Zod |
| Containerization | Docker + docker-compose |

### Why MongoDB here and not PostgreSQL?

Chat messages are an append-only, schema-flexible workload. MongoDB's document model fits naturally — a message is a self-contained document with its sender, timestamps, reactions, and thread metadata all in one place. Relational joins across millions of messages per day are expensive; MongoDB aggregation pipelines handle analytics on this shape of data much more efficiently.

PostgreSQL handles users, channels, and memberships (structured, relational data). MongoDB handles messages (high-volume, document-shaped data). This project intentionally uses both so you learn when to pick which database.

---

## Architecture

```
Client (any frontend / Postman / wscat)
        │
        ▼
  ┌─────────────────────────────┐
  │    Express HTTP Server      │  ← REST: auth, channels, history
  │    Socket.io WS Server      │  ← WebSocket: messages, presence
  └────────────┬────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
  PostgreSQL          MongoDB
  (users,            (messages,
   channels,          reactions,
   memberships)       threads)
       │
       ▼
     Redis
  ┌────────────────────────────┐
  │  Pub/Sub  │  Sessions      │  ← scales WebSockets across
  │           │  BullMQ queues │    multiple server instances
  └────────────────────────────┘
               │
               ▼
          BullMQ Workers
  (email notifications, cleanup crons,
   push notifications, offline queuing)
```

---

## Project Structure

```
nexora/
├── prisma/
│   ├── migrations/
│   └── schema.prisma          # Users, channels, memberships
├── src/
│   ├── config/
│   │   └── env.ts             # Zod-validated environment config
│   ├── lib/
│   │   ├── prisma.ts          # PostgreSQL client singleton
│   │   ├── mongoose.ts        # MongoDB connection
│   │   └── redis.ts           # Redis client singleton (Phase 3)
│   ├── models/
│   │   └── message.model.ts   # Mongoose message schema
│   ├── routes/
│   │   ├── health.routes.ts
│   │   ├── auth.routes.ts     # register, login, logout
│   │   └── channel.routes.ts  # CRUD for channels
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   └── channel.controller.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts  # JWT verify
│   │   └── error.middleware.ts
│   ├── socket/
│   │   ├── index.ts            # Socket.io server setup
│   │   ├── handlers/
│   │   │   ├── message.handler.ts
│   │   │   ├── presence.handler.ts
│   │   │   └── typing.handler.ts
│   │   └── events.ts           # Event name constants
│   ├── queues/                  # Phase 4 — BullMQ
│   │   ├── email.queue.ts
│   │   └── notification.queue.ts
│   ├── workers/                 # Phase 4 — job processors
│   │   └── email.worker.ts
│   ├── app.ts
│   └── server.ts
├── .env.example
├── .gitignore
├── docker-compose.yml           # Phase 6
├── Dockerfile                   # Phase 6
├── tsconfig.json
├── package.json
└── README.md
```

---

## Getting Started

### Prerequisites

- Node.js v18+
- MongoDB 7 (or Docker)
- PostgreSQL 16 (or Docker)
- Redis 7 (or Docker)

Quickest setup — spin everything with Docker:

```bash
# PostgreSQL (users, channels)
docker run --name nexora-postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=nexora \
  -p 5432:5432 -d postgres:16

# MongoDB (messages)
docker run --name nexora-mongo \
  -p 27017:27017 -d mongo:7

# Redis (pub/sub, queues)
docker run --name nexora-redis \
  -p 6379:6379 -d redis:7-alpine
```

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/ali0786mehdi/nexora.git
cd nexora

# 2. Install dependencies
npm install

# 3. Copy and fill environment variables
cp .env.example .env

# 4. Run database migrations
npx prisma migrate dev

# 5. Start the development server
npm run dev
```

### Environment Variables

**Phase 1 (foundation)**

```env
NODE_ENV=development
PORT=5000
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/nexora?schema=public"
MONGODB_URI="mongodb://localhost:27017/nexora"
```

**Added in later phases**

```env
# Phase 2 — JWT auth
JWT_ACCESS_SECRET=
JWT_REFRESH_SECRET=
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# Phase 3 — Redis
REDIS_URL=redis://localhost:6379

# Phase 4 — Email (BullMQ jobs)
SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASS=
EMAIL_FROM=

# Phase 5 — File uploads
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_REGION=
S3_BUCKET_NAME=

# Phase 6 — CORS
CLIENT_URL=http://localhost:3000
```

### Available Scripts

```bash
npm run dev            # Dev server with hot reload (tsx watch)
npm run build          # Compile TypeScript → dist/
npm run start          # Run compiled output (production)
npm run prisma:studio  # Visual database browser
```

---

## API Reference

### Health

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/api/health` | None | Server + database status |

### Auth — Phase 1

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/v1/auth/register` | None | Register new user |
| `POST` | `/api/v1/auth/login` | None | Login, returns token pair |
| `POST` | `/api/v1/auth/logout` | Bearer | Revoke refresh token |
| `POST` | `/api/v1/auth/refresh` | Cookie | Rotate access token |

### Channels — Phase 1

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/v1/channels` | Bearer | Create a channel |
| `GET` | `/api/v1/channels` | Bearer | List all public channels |
| `POST` | `/api/v1/channels/:id/join` | Bearer | Join a channel |
| `GET` | `/api/v1/channels/:id/messages` | Bearer | Paginated message history |

### WebSocket Events — Phase 2

**Client → Server (emit)**

| Event | Payload | Description |
|-------|---------|-------------|
| `message:send` | `{ channelId, content }` | Send a message to a channel |
| `dm:send` | `{ toUserId, content }` | Send a direct message |
| `typing:start` | `{ channelId }` | Broadcast typing started |
| `typing:stop` | `{ channelId }` | Broadcast typing stopped |
| `channel:join` | `{ channelId }` | Subscribe to a channel room |
| `channel:leave` | `{ channelId }` | Unsubscribe from a channel room |

**Server → Client (on)**

| Event | Payload | Description |
|-------|---------|-------------|
| `message:new` | `{ message }` | New message in a channel |
| `dm:new` | `{ message }` | Incoming direct message |
| `typing:update` | `{ userId, channelId, isTyping }` | Someone is typing |
| `presence:update` | `{ userId, status }` | User online/offline change |
| `error` | `{ message }` | Server-side socket error |

---

## Build Phases

- [ ] **Phase 1 — Foundation:** Express, TypeScript, PostgreSQL + Prisma, MongoDB + Mongoose, env validation, health check
- [ ] **Phase 2 — Core real-time (MVP):** Socket.io, channel messaging, DMs, typing indicators, read receipts, presence, message history pagination
- [ ] **Phase 3 — Scaling:** Redis Pub/Sub adapter for Socket.io, Redis session store, multi-server support
- [ ] **Phase 4 — Background jobs:** BullMQ queues, email notifications for missed messages, cron cleanup jobs
- [ ] **Phase 5 — Files & notifications:** S3 file/image uploads, push notifications, Server-Sent Events
- [ ] **Phase 6 — Production:** Dockerfile, docker-compose, rate limiting, security hardening

---

## License

MIT © [Ali Mehdi Mirza](https://github.com/ali0786mehdi)
