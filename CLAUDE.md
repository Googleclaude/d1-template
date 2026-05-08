# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Minimal Cloudflare Worker template demonstrating a D1 (serverless SQL) binding. The Worker runs `SELECT * FROM comments LIMIT 3` against the `DB` binding and returns the rows rendered as HTML.

## Commands

| Command | Action |
| --- | --- |
| `npm install` | Install deps |
| `npm run dev` | Apply migrations to the **local** D1 (`seedLocalD1`), then `wrangler dev` |
| `npm run check` | Typecheck (`tsc`) + `wrangler deploy --dry-run` — use as the CI gate |
| `npm run cf-typegen` | Regenerate `worker-configuration.d.ts` from `wrangler.json` after binding changes |
| `npm run deploy` | `predeploy` applies remote D1 migrations, then `wrangler deploy` |
| `npm run seedLocalD1` | `wrangler d1 migrations apply DB --local` |

`npm run dev` invokes `pnpm seedLocalD1` per `package.json` — install pnpm or replace it with `npm run seedLocalD1` if you don't have pnpm available.

## First-time setup (per README)

The committed `wrangler.json` contains a placeholder `database_id`. To use this template you must:

1. `npx wrangler d1 create d1-template-database` and paste the returned `database_id` into `wrangler.json`.
2. `npx wrangler d1 migrations apply --remote d1-template-database`.
3. `npx wrangler deploy`.

## Architecture

- **`src/index.ts`** — single fetch handler that queries the `DB` binding and returns HTML.
- **`src/renderHtml.ts`** — HTML template helper.
- **`wrangler.json`** — binds D1 database `d1-template-database` to `DB`. `compatibility_date` is `2025-10-08`. `upload_source_maps` and `observability` are enabled.
- **`migrations/`** — numbered SQL migrations. Filename format: `NNNN_description.sql`. Always increment the number; `wrangler d1 migrations apply` runs them in order.
- **`worker-configuration.d.ts`** — generated; provides the `Env` type used by `src/index.ts` (`ExportedHandler<Env>`). Do not hand-edit; run `npm run cf-typegen` after wrangler changes.

## Conventions

- Indentation is **tabs** (see `package.json`, `src/index.ts`).
- New SQL migrations go in `migrations/` with the next sequential number; both the local seed (`npm run dev`) and the deploy hook (`predeploy`) will pick them up.
