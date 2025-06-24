# AGENTS.md – Vocab-Insight

> **One file to rule how AI (Codex) and humans collaborate on this codebase.**
>
> Keep it in the repo root. Update it whenever architecture, tooling, or conventions evolve. Codex must always adhere to the latest committed version.

---

## 1 · Purpose & Vision

* **Mission** – Build the world’s most comprehensive, multi-language *vocabulary-in-media* platform so language learners can instantly gauge how much of a film, episode, book, or game they will understand and receive personalized study assets.
* **Core Value Proposition** – By aligning a user’s *known word set* with frequency-ordered and chronologically ordered vocab extracted from any media, we surface *comprehension-ready* content and *recommended next words*.
* **MVP Scope ("walking skeleton")**

  1. Upload an **SRT** subtitle file.
  2. Tokenize & lemmatize → store tokens in **PostgreSQL**.
  3. Show two sortable tables: **frequency** & **chronology**.
  4. Authenticated user can paste a list of known words → coverage % badge.
  5. CI-backed deployment on each push to **main**.
* **Long-term Horizons**
  ▸ Automatic crawling (OpenSubtitles, RSS, scrapers).
  ▸ Trusted-source submissions & moderation queue.
  ▸ Books (e-Pub/Kindle), visual novels, game scripts.
  ▸ Multi-language NLP pipelines.
  ▸ Word-knowledge import via *textbooks/courses finished* & **MCP server** sync.
  ▸ Export: **CSV**, **Anki .apkg** (via genanki-js).
  ▸ Recommended words to learn for target media.
  ▸ Series/Season/Episode hierarchies & franchise roll-ups.
  ▸ Graphs – *unique vs total known words over time*.
  ▸ Cost dashboards & eventual paid tier when free quotas burn out.

---

## 2 · Non-Functional Principles

1. **Best practices first** – Lint, test, review, automate. No TODO debt.
2. **Vertical-slice delivery** – Value in prod every sprint; avoid “big-bang” micro-service decomposition.
3. **AI-friendly code** – Deterministic formatting, exhaustive typing, descriptive names, rich inline docs so Codex can reason.
4. **Fail fast, recover faster** – Typed runtime validation (Zod) at every boundary, retryable queues, idempotent jobs.
5. **Observability from day 1** – Basic logging, tracing, metrics.

---

## 3 · Architecture Overview (MVP)

```
+--------------+     SRT Upload      +---------------------+
|  Next.js 15  |  ─────────────────▶ |  Express API Layer  |
|  App Router  |                     |  (tRPC optional)    |
+--------------+                     +-----------+---------+
        ▲                                        │  enqueue BullMQ "ingest"
        │ GraphQL/REST                            ▼
+-------+-------+                         +------+------+
|  BetterAuth   |    JWT / Session        |  Worker    |
+---------------+    ───────────▶         |  (spaCy)   |
                                          +------+------+
                                                 │ Drizzle ORM
                                                 ▼
                                          +------+------+
                                          | PostgreSQL |
                                          +-------------+
```

* **Frontend** – Next.js 15 (App Router) + React 19, Tailwind 4, TanStack Query.
* **API layer** – Express **or** Next.js route handlers; future tRPC possible.
* **Auth** – BetterAuth (email/password + Google OAuth; 2FA ready).
* **Queue** – BullMQ on Redis for CPU-bound ETL jobs (tokenization, Anki export).
* **NLP** – spaCy pipelines per language; pluggable custom rules.
* **ORM** – Drizzle (typed SQL, migrations in repo).
* **Containerisation** – Docker & Compose; each service its own image.
* **Hosting (free-tier)**
  ▸ Web → Vercel Hobby.
  ▸ Worker & API → Cloudflare Workers.
  ▸ Postgres → Neon Free.
  ▸ Redis → Upstash Free.
  ▸ Auth email sender (when needed) → Vercel Resend dev plan.

---

## 4 · Technology Stack & Versions

| Layer       | Choice             | Version  | Notes                                       |
| ----------- | ------------------ | -------- | ------------------------------------------- |
| Runtime     | Node.js            | 20 LTS   | Align with Next 15 requirement              |
| Frontend    | Next.js            | **15.x** | React 19-ready, Turbopack builds            |
| Styling     | TailwindCSS        | **4.x**  | New config & faster compiler                |
| Package Mgr | pnpm               | ^9       | Workspace & strict-dep guarantees           |
| Monorepo    | Turborepo          | latest   | Remote cache via Vercel token               |
| Auth        | BetterAuth         | latest   | TS-first, 2FA                               |
| DB          | PostgreSQL         | 15       | Cloud hosted (Railway)                      |
| ORM         | Drizzle ORM        | latest   | SQL-first migrations                        |
| Queue       | BullMQ             | ^4       | Redis-backed                                |
| Validation  | Zod                | ^3       | Compile-time types & runtime guards         |
| Tests       | Vitest + Supertest | latest   | 95 %+ coverage gate                         |
| Lint        | eslint             | latest   | `@typescript-eslint`, `eslint-config-turbo` |
| Format      | Prettier           | 3        | `tailwindcss` plugin                        |

---

## 5 · Repository Layout

```
/ (root)
├─ apps/
│  ├─ web/            # Next.js frontend
│  ├─ api/            # Express/Next handlers
│  └─ worker/         # BullMQ consumers
├─ packages/
│  ├─ db/             # Drizzle schema & migrations
│  ├─ config/         # Shared tsconfig, eslint, jest, vitest
│  ├─ ui/             # Re-export shadcn/ui components
│  └─ types/          # Shared Zod schemas & TS types
├─ .github/workflows/ # CI definitions
├─ docker/            # Dockerfiles & compose files
├─ AGENTS.md          # ← you are here
└─ README.md
```

---

## 6 · Coding Standards

1. **TypeScript strict** (`strict: true`, `exactOptionalPropertyTypes`).
2. **Folder-by-feature** inside `/apps/web`; co-locate tests (`*.test.tsx`).
3. **Prettier enforced** – run as *pre-commit* via Husky + lint-staged.
4. **eslint rules** – `unused-imports`, `no-explicit-any`, `import/order`, `tailwindcss/no-custom-classname`.
5. **Zod at boundaries** – every request body, env var (`zod-env`).
6. **Error handling** – never swallow errors; use typed `Result<T, E>` or throw `AppError`.

---

## 7 · Testing Strategy

| Level       | Tooling                    | What we cover                     |
| ----------- | -------------------------- | --------------------------------- |
| Unit        | Vitest                     | Pure logic, utils, Drizzle models |
| Integration | Supertest + Vitest         | API ↔ DB, Worker ↔ Queue          |
| E2E         | Playwright (later)         | Critical user journeys            |
| Contract    | Dredd/openAPI-test (later) | External consumers                |

* **Coverage gate** – 95 % lines + branches enforced in CI.

---

## 8 · CI / CD

* **GitHub Actions**
  ▸ `lint` → `test` → `build` → *Turborepo remote cache*.
  ▸ Fails the run if coverage <95 %.
  ▸ Preview deploy on every PR via Vercel; production deploy when **main** passes.
* **Secrets needed** – `DATABASE_URL`, `REDIS_URL`, `BETTERAUTH_SECRET`, `TURBO_TEAM`, `TURBO_TOKEN`, `VERCEL_TOKEN`.

---

## 9 · Contribution Workflow for Codex Agents

1. **Read latest `AGENTS.md`** and align output.
2. Generate code in feature branch `feat/<ticket-id>-<slug>`.
3. Ensure `pnpm turbo lint test build` passes locally.
4. Open PR with description: *Problem → Solution → Doc updates*.
5. Human reviewer merges; CI auto-deploys.

> **Codex MUST never push directly to main.**

---

## 10 · Definition of Done

* New feature fully implemented & covered by tests.
* Documentation & schema migrations updated.
* No ESLint/Prettier errors.
* CI green; preview URL posted.
* If touching data model, backfill script & migration included.

---

## 11 · Glossary

* **Coverage %** – Percentage of tokens in media that appear in user’s `KnownToken` set.
* **Token** – Normalised lemma representation of a word after spaCy pipeline.
* **KnownToken** – Join table linking user ↔ token.
* **MCP** – *Model Context Protocol* server for getting a user's known words or the vocab in from a particular piece of media

---

## 12 · References / Further Reading

* Next.js 15 Release Notes
* Tailwind CSS 4.0 GA
* Vercel Remote Cache for Turborepo
* pnpm Workspaces Docs
* Drizzle ORM Docs
* BullMQ Docs
* BetterAuth Docs
* spaCy Multi-language Models
* OpenSubtitles API & License
* genanki-js project
* Agents.md best-practice site

(See repository `/docs/REFERENCES.md` for full citation URLs.)

---

> *Update history* – Keep a changelog appendix when major architectural decisions change.
