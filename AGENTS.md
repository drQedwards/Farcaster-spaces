# AGENTS.md

## Cursor Cloud specific instructions

### Overview

This is a **Next.js 16** Farcaster Mini App called "Casting Spaces" — live audio rooms powered by LiveKit, with Farcaster identity via Neynar SDK. The source code lives in `/workspace/src/` (not the workspace root).

### Working directory

All commands (`pnpm dev`, `pnpm lint`, `pnpm type-check`, etc.) must be run from `/workspace/src/`.

### Running the dev server

```bash
cd /workspace/src
export DATABASE_URL=postgresql://devuser:devpass@localhost:5432/farcaster_spaces
pnpm dev
```

The dev server runs on `http://localhost:3000` using Turbopack.

**Important:** Before running `pnpm dev`, PostgreSQL must be running:
```bash
sudo pg_ctlcluster 16 main start
```

The `pnpm dev` script automatically runs `db:push` (schema sync) before starting Next.js. If `DATABASE_URL` is unset, schema sync is skipped gracefully.

### Environment variables

A `.env.local` file in `/workspace/src/` provides:
- `DATABASE_URL` — PostgreSQL connection string (required for persistence features)
- `NEYNAR_API_KEY` — Farcaster API (placeholder is fine; only crashes if `/api/neynar/` routes are hit)
- `NEXT_PUBLIC_USER_FID` — Farcaster ID (any integer works for dev)
- `NEXT_PUBLIC_LOCAL_URL` — Used for building canonical URLs

**LiveKit keys** (`LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`, `LIVEKIT_URL`, `NEXT_PUBLIC_LIVEKIT_URL`) are optional for basic dev; the app gracefully shows a "setup needed" UI without them. Only required to test actual audio room functionality.

### Type checking

`pnpm type-check` requires `next-env.d.ts` to exist. This file is auto-generated on the first `pnpm dev` or `pnpm build` run. If type-check fails with `Cannot find name 'PageProps'`, start the dev server first.

### Lint & formatting

- `pnpm lint` — ESLint (some pre-existing errors in the repo)
- `pnpm format:check` — Prettier check
- `pnpm validate` — Runs type-check + lint + format check

### Key architecture notes

- The pnpm store is configured to `../.pnpm-store` (relative to `/workspace/src/`, i.e. `/workspace/.pnpm-store`)
- The project uses pnpm workspaces with `packages: []` (no sub-packages)
- Database module (`src/neynar-db-sdk/src/db.ts`) creates a no-op stub when `DATABASE_URL` is unset — routes that access DB will throw at runtime but the app still starts
- `private-config.ts` uses `zod.parse()` which throws if `NEYNAR_API_KEY` is empty, but it's only loaded when `/api/neynar/` or `/api/coingecko/` routes are accessed (server-only, lazy compilation)
