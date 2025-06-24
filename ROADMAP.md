# ROADMAP – vocablist

> This file is the single source of truth for Codex-driven development. After each merged PR, Codex must mark completed tasks with ✅, then open the next unchecked task as a new issue using the **Feature / Task** template. When all tasks in a milestone are done, tick the milestone header.
> Bear in mind that this file too was AI generated. You have are pretty free to come up with your own next steps if you think you know better. Or if you think that you know better (free) tools for the job (and can back up your decisions with recent sources), then you are very welcome to do things in a different way.

## M0 – Repo Foundation & Governance

- [x] T0‑01 Initialize Git repository `vocablist`, add MIT License, README scaffold
- [x] T0‑02 Commit `AGENTS.md` and `ROADMAP.md` seeds
- [x] T0‑03 Add `.editorconfig`, `.nvmrc`, and root `pnpm-workspace.yaml`
- [x] T0‑04 Set up branch protection rules; commit `CODEOWNERS` (maintainer) and default labels
- [x] T0‑05 Add GitHub Issue templates (`💡 Feature / Task`, `🛠 Manual / Ops`, PR template)
- [ ] T0‑06 Enable Vercel Remote Cache tokens & add `TURBO_TOKEN`, `TURBO_TEAM` as **TODO secrets**
- [x] T0‑07 Install Husky + lint‑staged pre‑commit hooks
- [ ] T0‑08 Create GitHub Project board `Backlog` columns: Todo / In‑Progress / Review / Done

## M1 – Monorepo Skeleton & CI

- [ ] T1‑01 Run `pnpm dlx create-turbo@latest` to scaffold monorepo (`apps/`, `packages/`)
- [ ] T1‑02 Add base TypeScript `tsconfig` and enable `strict`
- [ ] T1‑03 Install ESLint with `@typescript-eslint`, `eslint-config-turbo`; configure Prettier + Tailwind plugin
- [ ] T1‑04 Add Vitest & coverage threshold 95%
- [ ] T1‑05 GitHub Actions workflow `ci.yml`: lint → test → build using Turborepo cache
- [ ] T1‑06 Workflow `deploy-preview.yml`: on PR push preview to Vercel (`--prebuilt`)
- [ ] T1‑07 Workflow `deploy-prod.yml`: on merge into `main` promote to production

## M2 – Auth & Database

- [ ] T2‑01 Provision Neon Free Postgres project (manual) and set `DATABASE_URL` secret
- [ ] T2‑02 Package `packages/db`: Drizzle schema & migrations for `user`
- [ ] T2‑03 Integrate BetterAuth in `apps/api` (Express or Next route handlers)
- [ ] T2‑04 Add sign‑up / sign‑in pages in Next.js 15 App Router
- [ ] T2‑05 Write integration tests for auth flow with Supertest & embedded Postgres

## M3 – Media Core Schema

- [ ] T3‑01 Extend Drizzle schema: `media`, `episode`, `subtitle_line`, `token`, `known_token`
- [ ] T3‑02 Seed script generating sample media and user for dev
- [ ] T3‑03 Add GraphQL or tRPC endpoint `/media/:id` returning basic media meta

## M4 – Upload & Storage

- [ ] T4‑01 Create Supabase project (manual) and add `SUPABASE_URL`, `SUPABASE_KEY` secrets
- [ ] T4‑02 Next.js upload page with drag‑and‑drop `.srt` (Zod validation)
- [ ] T4‑03 Store raw `.srt` in Supabase Storage bucket `subtitles/`
- [ ] T4‑04 Enqueue BullMQ `ingest` job with file URL, mediaId

## M5 – Queue & Worker

- [ ] T5‑01 Provision Upstash Redis free DB (manual) and add `REDIS_URL` secret
- [ ] T5‑02 Set up `apps/worker` container; subscribe to `ingest` queue
- [ ] T5‑03 Install spaCy with English model in worker Docker image
- [ ] T5‑04 Worker parses SRT, tokenizes, lemmatizes; bulk inserts tokens via Drizzle
- [ ] T5‑05 Unit tests for tokenizer logic (spaCy mocks) & integration test against Neon using pg‑embed

## M6 – Frequency & Chronology API

- [ ] T6‑01 SQL materialized view `media_token_frequency` (token, count, pct)
- [ ] T6‑02 Endpoint `/media/:id/tokens?order=frequency|chrono` with pagination
- [ ] T6‑03 Add TanStack Query hooks in frontend to fetch token lists

## M7 – Frontend Tables & UI

- [ ] T7‑01 Tailwind 4 powered table components with sortable headers & virtualised rows
- [ ] T7‑02 Media detail page shows frequency and chronological tables
- [ ] T7‑03 Add global layout, navigation bar, dark mode toggle (shadcn/ui)
- [ ] T7‑04 Lighthouse CI run to keep perf > 90

## M8 – Known Words & Coverage

- [ ] T8‑01 User settings UI: textarea paste list of known words
- [ ] T8‑02 Backend endpoint to upsert `known_token` list (Zod validated)
- [ ] T8‑03 Coverage algorithm: percentage of token occurrences covered
- [ ] T8‑04 Badge component on Media page displaying coverage
- [ ] T8‑05 E2E tests with Playwright: upload SRT, mark known words, expect ≥ badge

## M9 – Deployment Secrets & Envs

- [ ] T9‑01 Document all required env vars in `docs/ENVIRONMENT.md`
- [ ] T9‑02 GitHub Actions step to `echo $LATEST_ENV` into Vercel for previews
- [ ] T9‑03 Manual ops issue: configure Cloudflare Workers project (API edge) and add tokens

## M10 – OpenSubtitles Crawler

- [ ] T10‑01 Create service `apps/crawler` with cron schedule (GitHub Actions or Workers Cron)
- [ ] T10‑02 Implement licensed download flow, respect rate limits
- [ ] T10‑03 Parse metadata, create media records, enqueue ingest jobs
- [ ] T10‑04 Admin UI page listing crawl backlog & failures

## M11 – Multi‑language Support

- [ ] T11‑01 Auto language‑detect SRT via Compact Language Detector
- [ ] T11‑02 Add spaCy pipelines for JP, ES, DE using appropriate models
- [ ] T11‑03 Token table gains `language` column; DB index adjustments
- [ ] T11‑04 UI language switch and flag display

## M12 – Export & Study Assets

- [ ] T12‑01 Implement CSV export endpoint with proper mime headers
- [ ] T12‑02 Integrate `genanki-js` to create deck (.apkg) per media
- [ ] T12‑03 Queue job `export_anki`; upload result to Supabase Storage and provide signed URL
- [ ] T12‑04 Add "Download CSV" and "Download Anki" buttons in UI

## M13 – MCP Server Integration

- [ ] T13‑01 Implement Model Context Protocol client; schedule sync of user vocab
- [ ] T13‑02 Allow users to link MCP account from settings page
- [ ] T13‑03 Backfill `known_token` via MCP tokens

## M14 – Series / Season Hierarchy

- [ ] T14‑01 DB migration: `series`, `season` tables; link episodes
- [ ] T14‑02 API returns aggregated coverage across season / series
- [ ] T14‑03 UI breadcrumb & selector season/episode

## M15 – Word Recommendation Engine

- [ ] T15‑01 Simple TF‑IDF based ranking of top unknown tokens per user & media
- [ ] T15‑02 Endpoint `/media/:id/recommendations` returns list with definitions (lookup dictionary API)
- [ ] T15‑03 Side panel in UI showing top 20 words to learn next

## M16 – Observability & Cost Dashboards

- [ ] T16‑01 Add Pino logging + stdout JSON; pipe to Logtail free tier
- [ ] T16‑02 Add Prometheus compatible metrics endpoint; scrape with Grafana Cloud free
- [ ] T16‑03 Weekly GitHub Action to summarise Upstash/Neon usage and open `🛠 Manual` issue if near limits

## M17 – Paid Tier Prep

- [ ] T17‑01 Terms of Service & Privacy Policy pages

## M18 – Polish & Accessibility

- [ ] T18‑01 a11y audit, fix contrast and keyboard nav issues
- [ ] T18‑02 Internationalized UI strings via `next-intl`
- [ ] T18‑03 End‑to‑end smoke tests for critical flows on CI
- [ ] T18‑04 Documentation site using Next.js static export (`/docs`)
