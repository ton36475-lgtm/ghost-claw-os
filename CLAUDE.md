# CLAUDE.md

Guidance for AI coding assistants (Claude Code and others) working in this repository.

## What this repo actually is

This repo is a **single Expo/React Native app** scaffolded from a "Manus WebDev" template
(`package.json` name is still `app-template`), branded as **"Ghost Claw OS" / "AI Multi-Tool
Master"** — a mobile-first content-production tool (topic → short-form video stories, long-form
video autocut, asset library, etc.).

**Important discrepancy to be aware of:** the Markdown docs at the repo root and in
`ghost-claw-docs/` (e.g. `ghost-claw-docs/01-PRODUCT-SUMMARY.md`, `04-FILE-TREE.md`,
`COMPLETION-SUMMARY.md`, `MOBILE-APP-SETUP.md`, `MODULES-STRUCTURE.md`) describe a much larger,
aspirational system: a monorepo with a Next.js 15 web app, a FastAPI/NestJS backend, Postgres,
Redis, MinIO, a Python ML-worker fleet, Docker Compose, 11 fully-built modules, "26/26 tests
passing", etc. **None of that exists in this repo.** What actually exists on disk is:

- One Expo Router app (this directory), with a small Express + tRPC + Drizzle/MySQL backend
  living inside the same project (`server/`, `drizzle/`).
- Only **3 of the 11 "modules"** described in the docs have any screen code at all
  (`story-engine`, `autocut-studio`, `asset-library`), and **none of them are wired into
  navigation** — the only registered tab route is `app/(tabs)/index.tsx` (Home). The module
  screens are orphan files, not reachable by navigating the running app.
- There is no Next.js app, no FastAPI/NestJS backend, no Postgres/Redis/MinIO, no Docker Compose
  file, and no Python ML workers anywhere in this repo.
- Treat the `ghost-claw-docs/*.md` and root-level `*-GUIDE.md` / `*-SUMMARY.md` files as
  **product vision / planning documents**, not as an accurate description of current code.
  Cross-check any claim in them against the actual source before relying on it.
- `todo.md` (Phases 1–10, all unchecked) is a more honest reflection of current state: most of
  the planned UI/UX work has not been done yet.

## Directory structure (top levels, as it actually exists)

```
ghost-claw-os/
├── app/                        # Expo Router routes (file-based routing)
│   ├── _layout.tsx              # Root layout: providers (tRPC, React Query, SafeArea, Theme)
│   ├── (tabs)/                  # Tab navigator — only "Home" tab is registered
│   │   ├── _layout.tsx
│   │   └── index.tsx
│   ├── dev/theme-lab.tsx        # Internal theme/dev-only screen
│   ├── oauth/callback.tsx       # Manus OAuth deep-link callback
│   └── modules/                 # Orphan module screens, NOT linked from navigation
│       ├── story-engine/        # StoryEngineScreen.tsx + duplicate page.tsx ('use client' — a
│       │                        # Next.js directive that has no effect in Expo/React Native;
│       │                        # looks like copy-pasted Next.js code, needs cleanup)
│       ├── autocut-studio/
│       └── asset-library/
├── components/                  # Shared RN components (screen-container, themed-view, ui/*)
├── constants/                   # theme.ts, oauth.ts, const.ts
├── hooks/                       # use-auth, use-colors, use-color-scheme(.web)
├── lib/
│   ├── _core/                   # Framework/template internals — avoid editing (api, auth,
│   │                            # manus-runtime, theme, nativewind-pressable)
│   ├── trpc.ts                  # tRPC React client setup (superjson transformer)
│   ├── theme-provider.tsx
│   ├── utils.ts
│   ├── backend-integration.ts    # Higher-level API/job-polling client
│   └── gemma4-client.ts          # Axios client for an external "Gemma 4" backend (localhost:8000
│                                 # by default) — this backend is NOT part of this repo
├── server/                       # Express + tRPC backend, run in the same project
│   ├── _core/                    # Framework internals — avoid editing (env, trpc, oauth,
│   │                             # context, storageProxy, llm, imageGeneration,
│   │                             # voiceTranscription, notification, systemRouter, sdk)
│   ├── routers.ts                 # Add tRPC procedures here (appRouter)
│   ├── db.ts                       # Add query helpers here
│   ├── storage.ts                  # S3-backed storage helpers (via Forge presign API)
│   └── README.md                   # Full backend guide — READ THIS before touching server/
├── shared/                        # Code shared between client & server (types, const, errors)
├── drizzle/                       # Drizzle ORM: schema.ts, relations.ts, migrations/, meta/
├── android/                       # Native Android project files (gradle.properties only, no
│                                  # full native project checked in)
├── android-release.keystore       # Signing keystore (binary, checked into git — be careful)
├── assets/                        # Images/icons for Expo app
├── scripts/                       # build-apk.sh, generate_qr.mjs, load-env.js, reset-project.js
├── tests/                         # Vitest tests (auth.logout.test.ts, gemma4-client.test.ts)
├── ghost-claw-docs/                # Aspirational architecture/product docs (see caveat above)
├── app.config.ts                  # Expo config (bundle id, plugins, splash, etc.)
├── eas.json                        # EAS Build profiles (development/preview/production/apk)
├── drizzle.config.ts               # Requires DATABASE_URL to be set even for `drizzle-kit` CLI
├── tailwind.config.js / global.css # NativeWind (Tailwind for RN) v3.4.x
├── metro.config.js
├── tsconfig.json                   # extends expo/tsconfig.base, strict mode, path aliases
│                                    # "@/*" -> repo root, "@shared/*" -> shared/
├── eslint.config.js                 # eslint-config-expo flat config
└── package.json                     # name is still "app-template"; packageManager: pnpm@9.12.0
```

There is no `.env.example` in the repo (despite several docs telling you to `cp .env.example
.env.local` — that file does not exist; you'll need to create `.env`/`.env.local` from the
variable list below).

## Setup / dev / build / test / lint commands

Package manager is **pnpm** (`packageManager: pnpm@9.12.0` in `package.json`). No lockfile for
npm/yarn is present — use pnpm.

```bash
pnpm install                # install dependencies

pnpm dev                    # runs server + metro concurrently:
                            #   pnpm dev:server -> tsx watch server/_core/index.ts (Express+tRPC)
                            #   pnpm dev:metro  -> expo start --web (port $EXPO_PORT or 8081)
pnpm dev:server             # backend only
pnpm dev:metro              # Expo/Metro only (web)

pnpm android                # expo start --android
pnpm ios                    # expo start --ios

pnpm check                  # tsc --noEmit  (typecheck)
pnpm lint                   # expo lint (uses eslint.config.js / eslint-config-expo)
pnpm format                 # prettier --write .
pnpm test                   # vitest run  (no vitest.config.* file — uses Vitest defaults)

pnpm build                  # esbuild bundles server/_core/index.ts -> dist/ (Node/ESM, server only;
                            # this does NOT build the Expo app itself)
pnpm start                  # NODE_ENV=production node dist/index.js (run the built server)

pnpm db:push                # drizzle-kit generate && drizzle-kit migrate (requires DATABASE_URL)

pnpm qr                     # scripts/generate_qr.mjs — QR code for Expo Go testing
```

There are no GitHub Actions workflows in `.github/` (only issue templates), so there is no CI
pipeline to consult for the "real" commands — the `package.json` scripts above are the source of
truth.

APK build via `scripts/build-apk.sh` and EAS profiles in `eas.json` (development / preview /
production / playstore / apk) — see `APK-BUILD-GUIDE.md` / `QUICK-BUILD.md` for the intended
flow, but verify commands still match `package.json`/`eas.json` before trusting them, per the
discrepancy warning above.

## Architecture & conventions actually used in the code

- **Expo Router** (file-based routing) under `app/`. Root `_layout.tsx` wires up: NativeWind
  global CSS, React Query, tRPC provider, gesture handler root, safe-area context (with manual
  `initManusRuntime()` / `subscribeSafeAreaInsets` glue for the web-embedded case), and a custom
  `ThemeProvider`. Native headers are hidden by default (`headerShown: false`); enable explicitly
  per-screen if needed.
- **tRPC** end-to-end typing: `server/routers.ts` exports `appRouter`/`AppRouter`; the client in
  `lib/trpc.ts` uses `createTRPCReact<AppRouter>()` with `httpBatchLink` + `superjson`
  transformer. **The transformer must be set inside `httpBatchLink`, not at the client root**
  (tRPC v11 requirement — this is called out explicitly in `server/README.md`).
- **Auth**: "Manus OAuth". Web uses HTTP-only cookies; native uses a bearer token in
  `expo-secure-store`. `hooks/use-auth.ts` branches on `Platform.OS === "web"` vs native. Use
  `protectedProcedure` (vs `publicProcedure`) in `server/routers.ts` for routes that require
  `ctx.user`; frontend callers must catch `UNAUTHORIZED` tRPC errors explicitly.
- **Database**: Drizzle ORM targeting **MySQL/TiDB** (`drizzle-orm/mysql-core`, `drizzle-kit`
  dialect `mysql`), not Postgres (the `ghost-claw-docs` architecture doc says Postgres — that is
  aspirational, not what's implemented). Only table defined today is `users` in
  `drizzle/schema.ts`. `server/db.ts`'s `getDb()` is written to fail soft (returns `null`/warns)
  when `DATABASE_URL` isn't set, so local tooling can run without a live DB — but
  `drizzle.config.ts` (the CLI config used by `pnpm db:push`) hard-throws if `DATABASE_URL` is
  missing.
- **`_core/` directories** (`server/_core/`, `lib/_core/`, `shared/_core/`) are template/framework
  plumbing — the project convention (documented in `server/README.md`) is: **don't edit these
  unless you're intentionally extending framework infrastructure**; add your own logic in the
  sibling non-`_core` files (`server/routers.ts`, `server/db.ts`, `lib/trpc.ts`, etc.).
- **Styling**: NativeWind (Tailwind for React Native) v3.4.x via `global.css` +
  `tailwind.config.js`; do not upgrade to Tailwind v4 patterns. Theme colors are centralized in
  `theme.config.js`/`constants/theme.ts` (light/dark pairs) and consumed via `hooks/use-colors.ts`
  and `lib/theme-provider.tsx`.
- **Path aliases** (`tsconfig.json`): `@/*` → repo root, `@shared/*` → `shared/`.
- **External AI backend**: `lib/gemma4-client.ts` and `lib/backend-integration.ts` are Axios
  clients that expect a separate "Gemma 4" HTTP service (default `http://localhost:8000`,
  configurable via `EXPO_PUBLIC_GEMMA4_API_URL` / similar). That service is **not part of this
  repo** — it's an external dependency the docs assume exists elsewhere.
- **LLM / storage / voice / image-gen helpers** already scaffolded server-side under
  `server/_core/` (`llm.ts`, `imageGeneration.ts`, `voiceTranscription.ts`, `storageProxy.ts`) plus
  `server/storage.ts` (S3 via a Forge presign API) — call these only from server-side tRPC
  procedures, never expose their credentials to the client. See `server/README.md` for full usage
  examples (LLM `invokeLLM`, structured JSON responses, `storagePut`/`storageGetSignedUrl`,
  `notifyOwner`).
- **Testing**: Vitest, tests live in `tests/*.test.ts`. `tests/auth.logout.test.ts` is currently
  `describe.skip(...)` with a `// TODO: Remove .skip once you implement user authentication`
  comment — i.e. auth isn't actually implemented/tested yet despite the docs' claims of
  "production-ready" auth. `tests/gemma4-client.test.ts` tests the Axios client above (likely via
  mocked HTTP).

## Gotchas specific to this repo

1. **Docs vs. code mismatch is severe.** Don't take `ghost-claw-docs/`, `COMPLETION-SUMMARY.md`,
   `INTEGRATION-GUIDE.md`, `MODULES-STRUCTURE.md`, or `MOBILE-APP-SETUP.md` at face value for
   anything architectural (stack, module completeness, test counts, endpoints, monorepo layout).
   Verify against `app/`, `server/`, `drizzle/`, and `package.json` first.
2. **Module screens are unreachable from the running app.** `app/modules/story-engine`,
   `autocut-studio`, and `asset-library` exist as files but are not registered in any Expo Router
   segment/tab — there is no way to navigate to them today. If asked to "test the Story Engine
   screen" in the running app, you'll need to add routing/navigation first.
3. **`story-engine` has duplicate/conflicting files**: both `StoryEngineScreen.tsx` and
   `page.tsx` exist for the same module; `page.tsx` starts with `'use client'`, a Next.js App
   Router directive that does nothing in Expo/React Native — evidence of copy-pasted Next.js code
   that needs reconciling, not a working pattern to copy elsewhere.
4. **No `.env.example`.** Several docs instruct `cp .env.example .env.local`; that file isn't in
   the repo. Environment variables actually read by the code (see `server/_core/env.ts`,
   `server/README.md`, and `lib/gemma4-client.ts`) include: `DATABASE_URL`, `JWT_SECRET`,
   `VITE_APP_ID`, `OAUTH_SERVER_URL`, `VITE_OAUTH_PORTAL_URL`, `OWNER_OPEN_ID`, `OWNER_NAME`,
   `BUILT_IN_FORGE_API_URL`, `BUILT_IN_FORGE_API_KEY`, `PORT`, `EXPO_PORT`, and
   `EXPO_PUBLIC_`-prefixed client vars (`EXPO_PUBLIC_APP_ID`, `EXPO_PUBLIC_API_BASE_URL`,
   `EXPO_PUBLIC_OAUTH_PORTAL_URL`, plus whatever URL `gemma4-client.ts` / `backend-integration.ts`
   read for the external AI backend). No secret values are stored anywhere in this repo; do not
   invent or print any.
5. **`android-release.keystore` is committed to git** at the repo root as a binary file. Treat it
   as sensitive; do not copy it elsewhere or expose its contents, and flag to a human if this
   looks unintentional.
6. **`drizzle.config.ts` throws immediately if `DATABASE_URL` is unset** — `pnpm db:push` (and any
   other `drizzle-kit` invocation) will fail fast without it, even though the app's own runtime
   `getDb()` degrades gracefully.
7. **No CI workflows** exist (`.github/` only has issue templates) and **no `vitest.config.*`**
   file exists — `pnpm test` runs on Vitest defaults; don't assume CI-verified commands beyond
   what's in `package.json`.
8. **Backend is MySQL/TiDB via Drizzle, not Postgres** — despite `ghost-claw-docs` describing
   Postgres/Redis/MinIO; don't introduce Postgres-specific SQL/migrations without confirming with
   a human first.
