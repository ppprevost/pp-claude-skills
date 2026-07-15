---
name: mastering-typescript
description: |
  Master enterprise-grade TypeScript development with type-safe patterns, modern tooling, and framework integration. Provides task-driven guidance for TypeScript 5.9+, covering type system fundamentals (generics, mapped/conditional types, satisfies), enterprise patterns (error handling, Zod validation), React/NestJS/LangChain integration, and modern toolchains (Vite 7, pnpm, ESLint flat config, Vitest). Use when building type-safe applications, migrating JS, configuring TS toolchains, implementing advanced type patterns, or comparing TS with Java/Python.
version: 1.1.0
category: programming-languages
triggers:
  - typescript
  - ts
  - type-safe
  - generics
  - tsconfig
  - type guards
  - mapped types
  - conditional types
  - satisfies operator
  - zod validation
  - migrate to typescript
  - typescript strict mode
  - any to unknown
author: Richard Hightower
license: MIT
tags:
  - typescript
  - type-safety
  - enterprise
  - react
  - nestjs
  - langchain
  - vite
---
> **Fork notice**: originally installed from [spillwavesolutions/mastering-typescript-skill](https://github.com/spillwavesolutions/mastering-typescript-skill). This version has been locally tuned; check the upstream repo for its license before redistributing.


# Mastering Modern TypeScript

Build enterprise-grade, type-safe applications with TypeScript 5.9+.

> **Compat:** TypeScript 5.9+, Node.js 22 LTS, Vite 7, NestJS 11, React 19

## TL;DR

```
user task → match a row in [Task → References]
         → read 1-2 reference files in references/{name}.md
         → scan code for [Code-Pattern Triggers]
         → apply fix and cite the TS construct used (`satisfies`, `infer`, `z.object`...)
```

Skip when: task is non-TS (pure JS, Python, etc.), pure runtime logic with no type concern, framework-specific question already covered by `nestjs-best-practices` / `vercel-react-best-practices`.

## Workflow

1. **Classify** task using the Task → References table.
2. **Read** the candidate reference file(s) listed (don't re-read the catalog).
3. **Detect** code patterns from the Triggers table; each maps to a fix.
4. **Apply** the fix. Prefer `unknown` over `any`, `satisfies` over `as`, Zod at boundaries.
5. **Cite** the construct: `// satisfies: preserves literal types`, `// z.infer: type from schema`.

## Task → References Table

| User intent | Read first | Then consider |
|---|---|---|
| "convert JS to TS / migrate" | `references/enterprise-patterns.md` (Migration), Common Mistakes below | `references/type-system.md` |
| "set up tsconfig / strict mode" | `assets/tsconfig-template.json`, `references/toolchain.md` | Quick Start below |
| "configure ESLint / Vitest / Vite" | `references/toolchain.md`, `assets/eslint-template.js` | — |
| "type a React component / hook" | `references/react-integration.md` | `references/generics.md` |
| "build NestJS DTO / controller / module" | `references/nestjs-integration.md` | Validation with Zod below |
| "validate API input / runtime types" | `references/enterprise-patterns.md` (Validation), Validation with Zod below | `references/nestjs-integration.md` |
| "generic function / utility type" | `references/generics.md`, Utility Types below | `references/type-system.md` |
| "mapped types / conditional types / template literals" | `references/type-system.md`, `references/generics.md` | — |
| "type guards / discriminated unions" | `references/type-system.md` | Common Mistakes below |
| "error handling / Result type" | `references/enterprise-patterns.md` | — |
| "compare TS vs Java/Python" | Cross-Language Comparison below | — |

If task doesn't match a row, scan Code-Pattern Triggers below.

## Code-Pattern Triggers

| Pattern in code | Action |
|---|---|
| `: any` annotation | replace with `unknown`, narrow via type guard or schema |
| `as SomeType` cast | replace with `satisfies SomeType` (preserves literal types) |
| `enum Foo { ... }` | prefer literal union `type Foo = 'a' \| 'b'` (no runtime cost) |
| `Object.keys(x)` typed as `string[]` | use `(Object.keys(x) as Array<keyof typeof x>)` or `satisfies` |
| `JSON.parse(...)` returning `any` | parse via Zod schema: `Schema.parse(JSON.parse(...))` |
| `req.body` / `req.query` untyped | validate with Zod and `z.infer` for the type |
| `// @ts-ignore` | replace with `// @ts-expect-error <reason>` (errors when fixed) |
| `function foo(x): T` (param without type) | annotate or let inference flow from caller |
| `interface Foo { ... }` for FP code | use `type Foo = { ... }` (per project preference) |
| `readonly` missing on shared array/object props | add `readonly`, use `ReadonlyArray<T>` |
| `useState([])` (untyped) in React | use `useState<Item[]>([])` |
| `useRef(null)` for DOM | use `useRef<HTMLInputElement>(null)` |
| `class-validator` decorators in non-NestJS code | switch to Zod (smaller, FP-friendly) |
| `tsconfig.json` without `strict: true` | enable strict + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` |
| `.js` file with JSDoc `@param`/`@returns` | rename to `.ts`, convert JSDoc to native types |
| `() => asyncFn()` passed as callback | mark `async`: `async () => asyncFn()` — rule `@typescript-eslint/promise-function-async` |
| inline time arithmetic `30 * 24 * 60 * 60 * 1000` | use `ONE_DAY_MS` from `time.constants`, then `const X_MS = N_DAYS * ONE_DAY_MS` — rule `@typescript-eslint/no-magic-numbers` |
| domain import after infrastructure import | reorder: `@/.../domain/` before `@/.../infrastructure/` — rule `import-x/order` (caught by pre-commit lint) |

## Quick Start

```bash
pnpm create vite@latest my-app --template vanilla-ts
cd my-app && pnpm install
```

`tsconfig.json` (strict baseline):

```json
{
  "compilerOptions": {
    "target": "ES2024",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

Full template: `assets/tsconfig-template.json`.

## Validation with Zod (skeleton)

```ts
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  role: z.enum(['user', 'admin']),
});
type User = z.infer<typeof UserSchema>;

// At boundary (HTTP, DB, localStorage):
const result = UserSchema.safeParse(input);
if (!result.success) return { error: result.error.issues };
const user = result.data; // typed as User
```

Full patterns + error handling: `references/enterprise-patterns.md`.

## Utility Types Reference

| Type | Purpose | Example |
|------|---------|---------|
| `Partial<T>` | All properties optional | `Partial<User>` |
| `Required<T>` | All properties required | `Required<Config>` |
| `Pick<T, K>` | Select properties | `Pick<User, 'id' \| 'name'>` |
| `Omit<T, K>` | Exclude properties | `Omit<User, 'password'>` |
| `Record<K, V>` | Map of K to V | `Record<string, number>` |
| `ReturnType<F>` | Function return type | `ReturnType<typeof fn>` |
| `Parameters<F>` | Function param tuple | `Parameters<typeof fn>` |
| `Awaited<T>` | Unwrap Promise | `Awaited<Promise<User>>` |
| `NonNullable<T>` | Strip null \| undefined | `NonNullable<T \| null>` |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Using `any` | Defeats type safety | Use `unknown` and narrow |
| Skipping strict mode | Misses null/undefined bugs | Enable all strict flags |
| Type assertions (`as`) | Hides type errors | Use `satisfies` or guards |
| Enum for simple unions | Generates runtime code | Use literal unions |
| Not validating API data | Runtime mismatches | Zod at boundaries |
| `@ts-ignore` | Suppresses silently forever | `@ts-expect-error <reason>` |
| `JSON.parse` returning `any` | No runtime guarantee | Wrap in Zod parse |
| Inferring from inline literal then mutating | Loses literal narrowing | Use `as const` or `satisfies` |
| Non-`async` callback returning Promise | Pre-commit hook error (`promise-function-async`) | Always `async () =>` when the body `await`s or returns a Promise |
| Magic number time arithmetic inline | `no-magic-numbers` lint error | Named constant from `time.constants` + semantic wrapper (`FREE_RESET_INTERVAL_DAYS * ONE_DAY_MS`) |
| Domain import ordered after infrastructure import | `import-x/order` lint error | Group order: `domain/` → `use-cases/` → `infrastructure/` → `presentation/` |

## Cross-Language Comparison

| Feature | TypeScript | Java | Python |
|---------|------------|------|--------|
| Type System | Structural | Nominal | Gradual (duck typing) |
| Nullability | Explicit (`T \| null`) | `@Nullable` | `Optional` (typing) |
| Generics | Type-level, erased | Type-level, erased | Runtime via typing |
| Interfaces | Structural matching | Must implement | Protocol (3.8+) |
| Enums | Avoid (use unions) | First-class | `Enum` class |

## Checkpoints

Pause and confirm with user before:

- **Enabling `strict: true`** on existing codebase — will surface many errors. Show error count first.
- **Bulk `any → unknown`** across more than 3 files. List affected files first.
- **Bulk `interface → type`** (or vice versa) — project convention call. Don't auto-rewrite.
- **Removing all `@ts-ignore`** — may reveal hidden bugs or runtime errors. Convert to `@ts-expect-error <reason>` first, fix incrementally.
- **Switching `class-validator → Zod`** across NestJS DTOs — touches controllers, pipes, possibly OpenAPI. Migrate one module at a time.
- **Changing `tsconfig` target/module** — affects build, bundler, runtime semantics. Verify build passes after each change.
- **Adopting `noUncheckedIndexedAccess` / `exactOptionalPropertyTypes`** — high-friction strict flags. Enable one at a time.

For single-file, single-fix edits: apply directly, cite construct.

## Worked Example

**Input:** "Convert this Express handler from JS to TS with Zod validation:
`app.post('/users', (req, res) => { const { name, email } = req.body; res.json({ id: '1', name, email }); })`"

1. *Classify* → matches "validate API input / runtime types" + "convert JS to TS / migrate".
2. *Read* → `enterprise-patterns.md` (Validation), `nestjs-integration.md` skipped (Express, not Nest).
3. *Detect triggers* → `req.body` untyped → Zod schema. JSON.parse-style implicit untyped data → boundary parse. JS file → migrate to `.ts`.
4. *Apply*:
   ```ts
   import { z } from 'zod';
   import type { Request, Response } from 'express';

   const CreateUserBody = z.object({
     name: z.string().min(1),
     email: z.string().email(),
   });
   type CreateUserBody = z.infer<typeof CreateUserBody>;

   app.post('/users', (req: Request, res: Response) => {
     // z.infer + safeParse: typed body without `any`
     const parsed = CreateUserBody.safeParse(req.body);
     if (!parsed.success) return res.status(400).json({ error: parsed.error.issues });
     const { name, email } = parsed.data; // typed CreateUserBody
     return res.json({ id: '1', name, email } satisfies { id: string } & CreateUserBody);
   });
   ```
5. *Cite* `// z.infer`, `// safeParse: boundary validation`, `// satisfies: literal types preserved`.

## Fallbacks & Edge Cases

- **TS version < 5.0** → `satisfies` not available; use `as const` + manual validation. Document constraint.
- **No `tsconfig.json`** → ask user before creating one (might affect existing build). If user agrees, copy `assets/tsconfig-template.json` and adapt.
- **Mixed JS/TS codebase** → enable `allowJs: true`, `checkJs: false`; migrate file-by-file (see `enterprise-patterns.md`).
- **NestJS project** → defer framework-specific patterns to `nestjs-best-practices` skill; use this skill only for generic TS constructs.
- **React project** → defer to `vercel-react-best-practices` for performance; use this skill for type-level correctness.
- **CommonJS legacy code** → flag explicitly; ESM-first guidance assumes `"type": "module"`.
- **Code uses `interface` per legacy preference** → don't bulk-rewrite; only flag in new code per project convention.

## Resources

- `references/type-system.md` — primitives, unions, guards, satisfies, narrowing
- `references/generics.md` — generic functions, constraints, infer, mapped/conditional types
- `references/enterprise-patterns.md` — error handling, Result types, validation, migration
- `references/react-integration.md` — props, hooks, events, refs, generic components
- `references/nestjs-integration.md` — DTOs, modules, decorators, pipes, guards
- `references/toolchain.md` — Vite, pnpm, ESLint, Vitest, Prettier configuration
- `assets/tsconfig-template.json` — strict enterprise tsconfig
- `assets/eslint-template.js` — ESLint 9 flat config
- `scripts/validate-setup.sh` — verify environment

## Output Format

When applying a fix or reviewing code, cite the TS construct used:

```ts
// satisfies: preserves literal types from inline object
const cfg = { theme: 'dark' } satisfies Config;

// z.infer: TS type derived from runtime schema
type User = z.infer<typeof UserSchema>;
```

For review: `path:line — {construct}: {problem}. {fix}.`

**Recall vs precision:** also flag speculative matches (possible `any` leak via inference, possible literal-narrowing loss, suspicious `as` cast) with trailing `[speculative]`. Don't drop ambiguous cases.
