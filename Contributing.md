# Contributing to Nexora

Thank you for considering contributing to Nexora. This document explains how to get the project running locally, how to submit changes, and the conventions this project follows.

---

## Table of Contents

- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Branch Naming](#branch-naming)
- [Commit Messages](#commit-messages)
- [Pull Request Process](#pull-request-process)
- [Code Style](#code-style)
- [Reporting Bugs](#reporting-bugs)
- [Suggesting Features](#suggesting-features)

---

## Getting Started

### Prerequisites

- Node.js v18+
- Docker (recommended — runs Postgres, MongoDB, and Redis in one command)
- Git

### Local Setup

```bash
# 1. Fork the repo on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/nexora.git
cd nexora

# 2. Add the original repo as upstream
git remote add upstream https://github.com/ali0786mehdi/nexora.git

# 3. Install dependencies
npm install

# 4. Copy the example env file
cp .env.example .env
# Fill in your local values in .env

# 5. Start all databases with Docker
docker run --name nexora-postgres \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=nexora \
  -p 5432:5432 -d postgres:16

docker run --name nexora-mongo \
  -p 27017:27017 -d mongo:7

docker run --name nexora-redis \
  -p 6379:6379 -d redis:7-alpine

# 6. Run migrations
npx prisma migrate dev

# 7. Start the dev server
npm run dev
```

The server runs on `http://localhost:5000`. Hit `/api/health` to verify everything is connected.

---

## Development Workflow

```
main                  ← production-ready, protected
└── feat/your-feature ← your working branch
```

Never commit directly to `main`. Always create a feature branch, make your changes, then open a PR.

```bash
# Sync your fork with the latest upstream changes before starting work
git fetch upstream
git checkout main
git merge upstream/main

# Create your feature branch
git checkout -b feat/your-feature-name
```

---

## Branch Naming

Use one of these prefixes followed by a short, hyphen-separated description:

| Prefix | When to use |
|--------|------------|
| `feat/` | Adding a new feature |
| `fix/` | Fixing a bug |
| `refactor/` | Restructuring code without changing behavior |
| `docs/` | Documentation only |
| `chore/` | Build scripts, dependencies, config |
| `test/` | Adding or fixing tests |

**Examples:**
```
feat/typing-indicators
fix/redis-reconnect-loop
docs/websocket-events
chore/upgrade-socket-io
```

---

## Commit Messages

This project uses [Conventional Commits](https://www.conventionalcommits.org/).

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `perf`

**Examples:**
```bash
feat(socket): add typing indicator event handler
fix(auth): refresh token not cleared on logout
refactor(message): extract pagination logic into helper
docs(readme): update websocket events table
chore(deps): upgrade mongoose to 8.x
test(channel): add unit tests for join endpoint
```

Keep the subject line under 72 characters. Use the body to explain *why*, not *what*.

---

## Pull Request Process

1. Make sure your branch is up to date with `upstream/main`
2. Run the project locally and verify your changes work
3. Make sure TypeScript compiles without errors: `npm run build`
4. Open a PR against `main` on the original repo
5. Fill in the PR template completely — incomplete PRs will not be reviewed
6. Link the issue your PR closes using `Closes #ISSUE_NUMBER` in the description
7. Wait for review — address feedback before requesting re-review

PRs that skip the template, break TypeScript compilation, or don't have a linked issue will be closed without review.

---

## Code Style

- **TypeScript strict mode is on.** No `any` types unless explicitly justified in a comment.
- **No path aliases.** Use relative imports only — `../lib/prisma`, not `@/lib/prisma`.
- **CommonJS modules only.** No `import()` dynamic imports or ESM syntax.
- **Zod for all validation.** Never trust raw `req.body` directly in a controller.
- **Singleton patterns for clients.** Prisma, Redis, and Mongoose clients each have a single shared instance in `src/lib/`.
- **Error handling.** All async route handlers must have try/catch. Use the global error middleware for consistent error responses.

---

## Reporting Bugs

Use the **Bug Report** issue template on GitHub. Include:

- Steps to reproduce the bug
- Expected behavior
- Actual behavior
- Your Node.js version (`node -v`) and OS
- Relevant logs or error messages

Do not report security vulnerabilities as a public issue. See [SECURITY.md](./SECURITY.md) instead.

---

## Suggesting Features

Use the **Feature Request** issue template on GitHub. Describe:

- The problem you're trying to solve
- Your proposed solution
- Any alternatives you considered

Small, focused features are reviewed faster than large, sweeping proposals. If you're unsure whether a feature fits the project, open a discussion first.
