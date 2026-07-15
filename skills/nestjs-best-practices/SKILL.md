---
name: nestjs-best-practices
description: NestJS best practices and architecture patterns for production-ready apps. Use when writing, reviewing, or refactoring NestJS code. Triggers on NestJS modules, controllers, services, providers, guards, interceptors, pipes, DTOs, dependency injection, JWT auth, TypeORM/Prisma, microservices, BullMQ queues, or any "make this NestJS code better/safer/faster".
license: MIT
metadata:
  author: Kadajett
  version: "1.6.0"
---
> **Fork notice**: originally installed from [kadajett/agent-nestjs-skills](https://github.com/kadajett/agent-nestjs-skills). This version has been locally tuned; check the upstream repo for its license before redistributing.


# NestJS Best Practices

43 rules across 10 categories. Reference catalog with task-driven workflow.

## TL;DR

```
task → Task → Rules row → read 1-3 rules/{id}.md
     → scan code for [Code-Pattern Triggers]
     → apply by tier: CRITICAL → HIGH → MEDIUM → LOW
     → cite rule id
```

Skip when: not NestJS (use `nodejs-backend-patterns`), pure TS type Q (use `mastering-typescript`), framework-agnostic DDD (use `clean-ddd-hexagonal`).

## Workflow

1. **Classify** task via Task → Rules table.
2. **Read** 1-3 rule files at `rules/{rule-id}.md`.
3. **Detect** code-pattern triggers below.
4. **Apply** by priority tier.
5. **Cite** rule id: `// security-validate-all-input: ValidationPipe`.

## Task → Rules Table

| User intent | Read first | Then |
|---|---|---|
| build module / scaffold | `arch-feature-modules`, `arch-single-responsibility`, `di-prefer-constructor-injection` | `arch-module-sharing` |
| circular dep error | `arch-avoid-circular-deps`, `arch-module-sharing` | — |
| JWT / auth / authz | `security-auth-jwt`, `security-use-guards` | `security-rate-limiting` |
| validate request body / DTO | `security-validate-all-input`, `api-use-pipes`, `api-use-dto-serialization` | `security-sanitize-output` |
| exception / error handling | `error-use-exception-filters`, `error-throw-http-exceptions`, `error-handle-async-errors` | — |
| seed/debug/reset route, dev-only endpoint | `security-guard-dev-endpoints` | — |
| bootstrap hardening (`main.ts`, CORS, headers) | `security-harden-attack-surface` | `security-rate-limiting` |
| REST vs oRPC vs tRPC / frontend-backend contract | `api-choose-rpc-layer` | — |
| slow API / N+1 / DB perf | `db-avoid-n-plus-one`, `perf-optimize-database`, `perf-use-caching` | `arch-use-repository-pattern` |
| lazy load / startup | `perf-lazy-loading`, `perf-async-hooks` | — |
| testing / mock services | `test-use-testing-module`, `test-mock-external-services`, `test-e2e-supertest` | — |
| DB transaction / migration | `db-use-transactions`, `db-use-migrations` | — |
| interceptor / cross-cut | `api-use-interceptors`, `api-use-pipes` | `api-use-dto-serialization` |
| API versioning | `api-versioning` | — |
| microservice / queues / BullMQ | `micro-use-patterns`, `micro-use-queues` | `micro-use-health-checks` |
| config / env | `devops-use-config-module` | `devops-use-logging` |
| logging / observability | `devops-use-logging` | `devops-graceful-shutdown` |
| graceful shutdown / SIGTERM | `devops-graceful-shutdown` | `perf-async-hooks` |
| rate limit / throttle | `security-rate-limiting` | `security-use-guards` |
| DI scope / request-scoped | `di-scope-awareness`, `di-prefer-constructor-injection` | `di-use-interfaces-tokens` |
| interface / SOLID | `di-use-interfaces-tokens`, `di-interface-segregation`, `di-liskov-substitution` | `arch-use-repository-pattern` |
| event-driven | `arch-use-events` | `micro-use-queues` |

If no row matches, scan Code-Pattern Triggers.

## Code-Pattern Triggers

| Pattern | Rule |
|---|---|
| `forwardRef(() => OtherModule)` | `arch-avoid-circular-deps` |
| Service with many unrelated methods | `arch-single-responsibility` |
| `new` instantiating own deps inside `@Injectable()` | `di-prefer-constructor-injection` |
| `@Inject('STRING_TOKEN')` | `di-use-interfaces-tokens` |
| `throw new Error(...)` in controller/service | `error-throw-http-exceptions` |
| No global `useGlobalFilters` | `error-use-exception-filters` |
| `@Body() dto: any` or no DTO class | `security-validate-all-input`, `api-use-dto-serialization` |
| Controller without `@UseGuards()` | `security-use-guards` |
| `process.env.X` in service | `devops-use-config-module` |
| `console.log` in production code | `devops-use-logging` |
| Repository called from controller | `arch-use-repository-pattern` |
| Loop with `await repo.findOne(...)` | `db-avoid-n-plus-one` |
| Multiple writes without transaction | `db-use-transactions` |
| `Test.createTestingModule` without mocks | `test-mock-external-services` |
| Bull/BullMQ direct in controller | `micro-use-queues` |
| Per-request scope injected into singleton | `di-scope-awareness` |
| No `/health` in microservice | `micro-use-health-checks` |
| `seed()`/`dump-*`/`debug` controller route, no `NODE_ENV` check | `security-guard-dev-endpoints` |
| `app.enableCors()` with no `origin` allowlist, or missing `helmet()` in `main.ts` | `security-harden-attack-surface` |
| Hand-duplicated frontend types matching a REST response shape | `api-choose-rpc-layer` |

## Priority Tiers

| Tier | Categories | Prefix |
|---|---|---|
| CRITICAL | Architecture, DI | `arch-`, `di-` |
| HIGH | Error, Security, Performance | `error-`, `security-`, `perf-` |
| MEDIUM-HIGH | Testing, Database | `test-`, `db-` |
| MEDIUM | API, Microservices | `api-`, `micro-` |
| LOW-MEDIUM | DevOps | `devops-` |

## Checkpoints

Pause and confirm before:
- Bulk circular-dep refactor (>3 files)
- Adding global guards/filters/interceptors
- Switching DI scope `Default → REQUEST`
- Repository-pattern refactor across many services
- Replacing `class-validator` with Zod (or vice versa)
- Adding rate-limit/auth globally
- DB migration that drops columns or changes constraints

Single-file fixes: apply directly, cite rule id.

## Output Format

Diff: `// {rule-id}: {one-line why}`
Review: `path:line — {rule-id}: {problem}. {fix}.`

**Recall vs precision:** flag speculative matches with trailing `[speculative]`. Don't drop ambiguous cases.

## Fallbacks & Edge Cases

- **Rule file missing** → fall back to `AGENTS.md`.
- **NestJS v9 vs v10/v11** — some APIs renamed; check rule file's doc link.
- **Fastify adapter** — interceptor/hooks differ; flag in comment.
- **Prisma vs TypeORM** — `db-*` rules apply to both, syntax differs.
- **Microservices project** — `arch-*`/`di-*` apply; `api-*` may not.
- **Test files (*.spec.ts)** — relax runtime rules (e.g. rate limit).
- **Full audit requested** — walk every Task → Rules row touching changed files.

## Resources

- `rules/{rule-id}.md` — single rule (~1-3KB): why + bad + good. List with `ls rules/`.
- `AGENTS.md` — full catalog (~160KB), 40 rules. Use only when rule file missing or full audit.
