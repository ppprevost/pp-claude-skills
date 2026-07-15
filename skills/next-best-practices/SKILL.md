---
name: next-best-practices
description: Next.js best practices — file conventions, RSC boundaries, data patterns, async APIs, metadata, error handling, route handlers, image/font optimization, bundling. Use when writing or reviewing Next.js (App Router) code. Triggers on Next.js, App Router, RSC, server components, server actions, route handlers, params/searchParams, generateMetadata, next/image, next/font, hydration error, CSR bailout, parallel routes, intercepting routes, middleware, edge runtime.
license: MIT
metadata:
  version: "1.5.0"
user-invocable: false
---
> **Fork notice**: originally installed from [vercel-labs/next-skills](https://github.com/vercel-labs/next-skills). This version has been locally tuned; check the upstream repo for its license before redistributing.


# Next.js Best Practices

19 topic modules covering App Router patterns. Task-driven workflow.

## TL;DR

```
task → Task→References row → read 1-2 refs
     → scan [Code-Pattern Triggers]
     → cite construct (`async params`, `Suspense`, `next/image`...)
```

Skip: non-Next.js (→ `nodejs-backend-patterns`), pure React perf (→ `vercel-react-best-practices`), pure TS (→ `mastering-typescript`).

## Workflow

0. **Skip check.** If task is non-Next.js, pure React perf, or pure TS → bail to Skip targets above. Don't proceed.
1. **Classify** intent via Task→References table (20 rows). Multiple matches → take the most specific row; broad task ("build a page") → take all matching rows in sequence.
2. **Read** 1-2 ref files (~1-6KB each). Each module is self-contained.
3. **Detect** code-pattern triggers below; merge findings with table-routed refs.
4. **Apply** fix per reference. Cite the exact construct, not a paraphrase.
5. **Cite** construct as inline comment or review line: `// async params`, `// next/image: LCP`, `// Suspense: CSR bailout`.

**Fallback when stuck:**
- No table row matches → scan Code-Pattern Triggers by symbol.
- No pattern row either → check Fallbacks & Edge Cases section.
- Still unclear → mark output `[speculative]` (see Output Format) and proceed with closest module.

## Task → References Table

| User intent | Read first | Then |
|---|---|---|
| "build page / route / segment" | `file-conventions.md`, `data-patterns.md` | `rsc-boundaries.md` |
| "params / searchParams sync error (Next 15)" | `async-patterns.md` | `functions.md` |
| "metadata / SEO / OG image" | `metadata.md` | `file-conventions.md` |
| "RSC vs client / 'use client' / serializable props" | `rsc-boundaries.md`, `directives.md` | `data-patterns.md` |
| "server action / mutation" | `directives.md`, `data-patterns.md` | `error-handling.md` |
| "route handler / API route" | `route-handlers.md` | `runtime-selection.md` |
| "edge runtime / Node runtime" | `runtime-selection.md` | `bundling.md` |
| "error boundary / not-found / forbidden" | `error-handling.md` | `file-conventions.md` |
| "data fetching / waterfalls / Promise.all" | `data-patterns.md` | `rsc-boundaries.md` |
| "next/image / LCP" | `image.md` | — |
| "next/font / Tailwind" | `font.md` | — |
| "hydration error" | `hydration-error.md` | `rsc-boundaries.md` |
| "useSearchParams / CSR bailout" | `suspense-boundaries.md` | `functions.md` |
| "parallel routes / modal" | `parallel-routes.md` | — |
| "bundle / ESM CJS" | `bundling.md` | — |
| "next/script / GA" | `scripts.md` | — |
| "self-host / Docker / ISR" | `self-hosting.md` | — |
| "navigation hooks" | `functions.md` | `suspense-boundaries.md` |
| "cookies/headers/draftMode" | `functions.md`, `async-patterns.md` | — |
| "debug build / MCP" | `debug-tricks.md` | — |

If no row matches, scan Code-Pattern Triggers.

## Code-Pattern Triggers

| Pattern | Reference |
|---|---|
| `params.id` (sync access in Next 15) | `async-patterns.md` |
| `searchParams.q` (sync) | `async-patterns.md` |
| `cookies()`/`headers()` not awaited (Next 15) | `async-patterns.md` |
| `'use client'` + `async function Component` | `rsc-boundaries.md` |
| Function passed as prop from RSC → Client | `rsc-boundaries.md` |
| `<img>` instead of `<Image>` | `image.md` |
| Custom `<link rel="font">` | `font.md` |
| `useSearchParams()` without `<Suspense>` parent | `suspense-boundaries.md` |
| Sequential `await fetch()` in RSC | `data-patterns.md` |
| `throw new Error()` in server code without `unstable_rethrow` | `error-handling.md` |
| `redirect()` inside try/catch | `error-handling.md` |
| `route.ts` GET + sibling `page.tsx` | `route-handlers.md` |
| Inline `<script>` without `id` | `scripts.md` |
| `process.env` in client component (NEXT_PUBLIC_ missing) | `bundling.md` |
| Modal via `useState` instead of parallel routes | `parallel-routes.md` |
| Browser API at module top of RSC | `hydration-error.md` |
| `metadata` exported from client component | `metadata.md` |
| `runtime: 'edge'` with Node-only deps | `runtime-selection.md`, `bundling.md` |

## Checkpoints

Pause and confirm before:
- Switching `runtime: 'edge'` ↔ `nodejs` on existing route — affects bundle, deps.
- Bulk converting client components to RSC (or vice versa) — touches data flow.
- Adding global `error.tsx` / `not-found.tsx` at root — catches all routes.
- Migrating `params/searchParams` sync → async across pages (Next 15) — run codemod first.
- Adding `output: 'standalone'` to `next.config.ts` — affects deploy.
- Changing image domains in `next.config.ts` — runtime impact.

Single-file fixes: apply directly, cite construct.

## Output Format

Diff: `// {construct}: {one-line why}`
Review: `path:line — {construct}: {problem}. {fix}.`

**When uncertain** (rule may not apply, version unclear, edge case not in modules): prefix the cited construct with `[speculative]` so the reader sees it as a guess to verify, not a hard rule. Example: `// [speculative] async params required here too`.

## Worked Example

**Input:** "Build /products/[id] with async params, metadata, OG image, RSC fetch + client island."

1. *Classify* → "build page" + "metadata / OG" + "RSC vs client".
2. *Read* → `file-conventions.md`, `metadata.md`, `rsc-boundaries.md`.
3. *Detect* → Next 15 params async; OG → `next/og`; client interaction → `'use client'` island.
4. *Apply* RSC `Page` with `await params`, `generateMetadata` for dynamic title + OG, `app/products/[id]/og/route.tsx` returning `ImageResponse`.
5. *Cite* `// async params`, `// generateMetadata`, `// next/og`, `// 'use client'` on island.

## Fallbacks & Edge Cases

- **Pages Router** (legacy `pages/`) → most rules N/A; recommend App Router.
- **Next < 15** → params/searchParams sync; skip async-patterns codemod.
- **`use cache`** → experimental, behind `dynamicIO` flag; verify config first.
- **Vercel vs self-host** → some patterns differ (ISR cache, image opt); note in fix.
- **Monorepo** → check `transpilePackages` if package not bundling.
- **Full audit** → walk every Task → References row touching changed files.

## Resources

19 topic modules (1-8KB each):

| Module | Topic |
|---|---|
| `file-conventions.md` | Project structure, route segments, middleware→proxy rename |
| `rsc-boundaries.md` | RSC/client split, async client invalid, serializable props |
| `directives.md` | `'use client'`, `'use server'`, `'use cache'` |
| `async-patterns.md` | Next 15 async `params`, `cookies()`, `headers()` |
| `functions.md` | Navigation hooks, server fns, generate fns |
| `data-patterns.md` | RSC fetch, Server Actions, waterfalls, preload, cache |
| `route-handlers.md` | `route.ts`, GET conflicts, when vs Server Actions |
| `runtime-selection.md` | Node vs Edge runtime trade-offs |
| `error-handling.md` | `error.tsx`, `notFound`, `redirect` in try/catch, `unstable_rethrow` |
| `metadata.md` | Static/dynamic metadata, `generateMetadata`, OG images |
| `image.md` | `next/image`, `sizes`, blur placeholder, LCP priority |
| `font.md` | `next/font`, Google/local fonts, Tailwind integration |
| `scripts.md` | `next/script`, inline `id`, GA via `@next/third-parties` |
| `bundling.md` | Server-incompatible deps, CSS imports, ESM/CJS |
| `hydration-error.md` | Browser APIs, dates, invalid HTML, fixes |
| `suspense-boundaries.md` | `useSearchParams`/`usePathname` CSR bailout |
| `parallel-routes.md` | `@slot`, `(.)` interceptors, modal pattern, `default.tsx` |
| `self-hosting.md` | `output: 'standalone'`, Docker, multi-instance ISR |
| `debug-tricks.md` | MCP endpoint, `--debug-build-paths` |
