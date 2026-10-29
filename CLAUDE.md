# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Arcade Vault: an online games platform where players compete for the highest score. The README says development follows spec-driven design (`/spec` and `/spec-impl` skills, installed via `npx skills@latest add Klerith/fernando-skills`). The repo is currently a fresh Create Next App scaffold (single `app/page.tsx`, no `spec/` directory yet).

## Commands

```bash
npm run dev     # next dev (Turbopack)
npm run build   # next build
npm run start   # next start
npm run lint    # eslint (flat config, eslint.config.mjs)
```

No test runner is configured.

## Next.js version caveat

Next.js 16.4.3 / React 19.3. Per `AGENTS.md`, this version has breaking changes versus common knowledge: read the relevant guide in `node_modules/next/dist/docs/` (`01-app`, `02-pages`, `03-architecture`, `04-community`) before writing Next.js code, and heed deprecation notices.

## Architecture notes

- App Router only (`app/`). TypeScript strict; path alias `@/*` maps to the repo root.
- `next.config.ts` enables non-default options: `cacheComponents: true`, `partialPrefetching: true`, `experimental.agentFeedback`. `cacheComponents` changes caching/dynamic rendering semantics, so check the docs before using data fetching or dynamic APIs.
- Tailwind v4 is wired through Turbopack via `@tailwindcss/turbopack` as a CSS loader rule in `next.config.ts` (not PostCSS). Global styles are in `app/globals.css`.
- `app/layout.tsx` uses the globally typed `LayoutProps<"/">` helper (no import) and loads Geist fonts via `next/font/google`.
