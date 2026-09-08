# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project

Arcade Vault — plataforma para jugar online y competir por puntos. Early stage: currently a stock `create-next-app` scaffold; no game/feature code yet. Primary language of specs/docs: Spanish.

## Commands

- `npm run dev` — dev server (Turbopack, default in Next 16)
- `npm run build` — production build (Turbopack)
- `npm run start` — serve production build
- `npm run lint` — ESLint (flat config `eslint.config.mjs`; `next lint` removed in Next 16, call `eslint` directly)

No test runner configured yet.

## Stack

Next.js 16.3.4 (App Router) · React 19.2.8 · Tailwind CSS v4 · TypeScript strict. Path alias `@/*` → repo root.

## Next.js 16 — read before writing code

`AGENTS.md` (managed block written by `next dev`) requires reading the version-matched docs in `node_modules/next/dist/docs/` before writing any Next code. This Next.js differs from older training data. Key differences:

- Async Request APIs: `params`, `searchParams`, `cookies()`, `headers()`, `draftMode()` are Promise-only — must `await`. Sync access removed.
- Typed route helpers `PageProps<'/route'>`, `LayoutProps<'/'>`, `RouteContext<'/route'>` are global (see `app/layout.tsx`). Regenerate with `npx next typegen` after adding/changing routes.
- Turbopack is default for dev and build. `turbopack` config is a top-level key in `next.config.ts` (no longer `experimental.turbopack`). A stray `webpack` config makes `next build` fail.
- Upgrade guide: `node_modules/next/dist/docs/01-app/02-guides/upgrading/version-16.md`.

Commit the `<!-- BEGIN:nextjs-agent-rules -->` block in `AGENTS.md` alongside your changes — `next dev` recreates it otherwise.

## Tailwind v4

CSS-first config, no `tailwind.config.js`. Theme tokens live in `app/globals.css` under `@theme inline` with `@import "tailwindcss"`; PostCSS plugin `@tailwindcss/postcss` (`postcss.config.mjs`).

## Intended workflow

Spec Driven Design via the `/spec` and `/spec-impl` skills from `Klerith/fernando-skills` (see README). Install with `npx skills@latest add Klerith/fernando-skills` — skills are not yet present in the repo.
