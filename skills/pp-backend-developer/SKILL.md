---
name: pp-backend-developer
description: Backend stack opinions for TypeScript projects — REST/oRPC API design, caching strategies, ORM (Prisma/Drizzle), security checklist (env, JWT, RBAC, rate limiting, sensitive columns, crypto), Result-type error handling, BullMQ queues, Outbox pattern, Pino observability, Jest+Supertest testing. Use when building, reviewing, or refactoring backend TypeScript code in projects using NestJS, Next.js server-side, Hono, Express, Fastify, Astro SSR. Triggers on API design, REST endpoints, oRPC, tRPC, caching, Redis, Prisma, Drizzle, JWT auth, refresh token, RBAC, rate limit, BullMQ, queues, outbox, event-driven, Pino logger, health check, observability.
license: MIT
metadata:
  author: pp-backend-developer agent
  version: "1.0.0"
  triggers:
    - rest api design
    - openapi
    - oRPC
    - tRPC
    - caching strategy
    - cache-aside
    - Redis cache
    - Prisma
    - Drizzle
    - JWT auth
    - refresh token
    - RBAC
    - rate limit
    - sensitive columns
    - BullMQ
    - queue
    - outbox pattern
    - event-driven
    - Pino logger
    - health check
    - observability
    - Result type
    - error handling
---

# pp-backend-developer Skill

Backend stack opinions + reusable patterns for TypeScript projects. Companion to `pp-backend-developer` agent (workflow + persona).

## TL;DR

```
review/build task → match a row in [Task → Section]
                  → scan [Code-Pattern Triggers]
                  → apply pattern
                  → cite construct (Result type, Outbox, JWT refresh, ...)
```

Skip when: not backend (use `pp-frontend-developer`), pure NestJS DI question (use `nestjs-best-practices`), pure architecture (use `clean-ddd-hexagonal`), pure TS types (use `mastering-typescript`).

## Workflow

1. **Classify** task via Task → Section table.
2. **Read** relevant section(s).
3. **Detect** triggers; map to fix.
4. **Apply** pattern. Default to opinionated stack unless project overrides.
5. **Cite** construct in diff/comment.

## Task → Section Table

| User intent | Section |
|---|---|
| "layering / dependency direction / feature structure / where does X go" | DDD Layering |
| "use-case imports infra / domain imports prisma / leaky types" | DDD Layering |
| "design REST API / endpoints / versioning" | REST API Design |
| "monorepo fullstack typed contract" | oRPC / tRPC Decision |
| "cache strategy / Redis / TTL / invalidation" | Caching |
| "ORM choice / Prisma vs Drizzle" | ORM |
| "schema DB / migration / index / soft delete" | ORM |
| "JWT auth / refresh / sessions" | Security: Auth |
| "RBAC / roles / permissions" | Security: Authorization |
| "rate limiting / throttler" | Security: Rate Limiting |
| "sensitive cols (password, secrets) / select default" | Security: Data Exposure |
| "env validation / secrets handling" | Security: Environment |
| "crypto / OTP / token generation" | Security: Crypto |
| "seed endpoint / dev-only routes" | Security: Dev Endpoints |
| "headers / CORS / helmet" | Security: Surface |
| "SQL injection / raw queries / external data" | Security: Injection |
| "error response / Result type / try-catch tree" | Error Handling |
| "background job / queue / async processing" | Queues |
| "event-driven / pub-sub / @OnEvent" | Inter-services |
| "outbox / reliable events post-DB-write" | Inter-services |
| "logging / observability / Pino" | Observability |
| "health check / liveness / readiness" | Observability |
| "testing / Jest / supertest / mock" | Testing |

If no row matches, scan Code-Pattern Triggers below.

## Code-Pattern Triggers

| Pattern | Action |
|---|---|
| `process.env.X` direct in service/controller | inject `ConfigService` (NestJS) or import validated `env` from Zod schema |
| `process.env.X \|\| 'fallback-secret'` | remove fallback, fail fast — Zod env schema must reject missing |
| `Math.random()` for tokens/OTPs/IDs | `crypto.randomInt` / `crypto.randomBytes(N).toString('hex')` |
| `bcrypt.hash` rounds < 10 | use 10+ rounds; document why if higher |
| `password: string` selected by default in repo query | `select: false` (TypeORM) or `omit: { password: true }` (Prisma) |
| Refresh token in `localStorage` (frontend hint) | flag — use httpOnly cookie |
| `app.enableCors({ origin: '*' })` in prod | strict allowlist via `env.CORS_ORIGINS` |
| `helmet()` missing in `main.ts` | add `app.use(helmet())` |
| `ValidationPipe` without `whitelist: true` | enable `whitelist + forbidNonWhitelisted` |
| `$queryRaw` with template interpolation | use `Prisma.sql\`\`` tagged template |
| Multi-table writes without `$transaction` | wrap in `prisma.$transaction([...])` |
| `forEach` with `await` | use `for...of` (sequential) or `Promise.all` (parallel) |
| `try/catch` returning generic 500 | use exception filter mapping `DomainError` → HTTP code |
| Endpoint `@Post('seed')` without env guard | add `if (env.NODE_ENV === 'production') throw ForbiddenException` |
| `console.log` in service code | `Logger` (NestJS) or Pino |
| Logger logging full `req.body` or password | `redact: ['req.body.password', 'req.headers.authorization']` |
| API route handler missing ownership check on user-owned resource | add `where: { id, createdBy: session.userId }` filter |
| Redirect via `res.redirect()` on endpoint called by axios | return `{ redirectUrl }` JSON instead |
| Static route declared after `@Get(':id')` | move static route above paramaterized |
| Pagination via `skip + take` on large dataset | switch to cursor pagination |
| `attempts` missing on BullMQ `queue.add` calling external API | add `attempts: 3, backoff: { type: 'exponential', delay: 2000 }` |
| `removeOnComplete` / `removeOnFail` missing | add to prevent Redis bloat |
| Event emit + DB write not in transaction | use Outbox pattern (event row in same `$transaction`) |
| External webhook payload not validated | parse via Zod before processing |
| Hardcoded magic numbers in business rules (min/max) | extract to `domain/{x}.rules.ts` constants |
| `use-case` / `domain` file imports from `infrastructure/` (esp. `import type` of an infra type) | inward-only deps: move the type to `domain/` (or a use-case port), have infra conform to it. See DDD Layering |
| Feature-specific logic living in `shared/infrastructure/` (e.g. `sendXBatch` for one feature in shared email) | relocate to `features/{ctx}/infrastructure/`; keep only generic primitives in shared |
| Port dep (`sendBatch`, repo fn) typed with an infra-owned type | define the port type in the use-case/domain; the adapter imports it, not the reverse |
| `while (true)` / `for (;;)` with `if (...) break` as real exit | rewrite with an explicit loop condition. Keyset/cursor pagination: drive on a `hasMore` flag (`hasMore = page.length === PAGE_SIZE`). Stream readers: read-ahead `let chunk = await reader.read(); while (!chunk.done) { ...; chunk = await reader.read() }`. Reserve infinite-loop syntax for genuinely unbounded pumps (queue consumers, daemons) and comment why |

## DDD Layering

Feature-first vertical slice. Dependencies point INWARD only. Outer layer imports inner, never the reverse.

```
domain/          -> nothing (zero external import). Entities, value objects, rules, port TYPES.
use-cases/       -> domain/ + own infrastructure/repository. Pure functions, deps injected.
infrastructure/  -> domain/ + shared/infrastructure (prisma, generic helpers). Adapters conform to ports.
presentation/    -> use-cases/ + infrastructure/ (cross-feature OK here only).
```

**Iron rules:**
- A `use-case` or `domain` file importing from `infrastructure/` (even `import type`) is a layering violation. The dependency arrow is backwards. Move the type inward (to `domain/` or a use-case port); the adapter conforms to it.
- **Port types belong to the caller, not the adapter.** A use-case port (`sendBatch`, `loadPage`, a repo fn signature) is defined in the use-case/domain. Infrastructure imports those types and implements them. Never type a port with an infra-owned type.
- **`shared/infrastructure` holds only generic primitives** (prisma client, resend client, `wrapEmailHtml`, `cn`, env). Feature-specific logic (e.g. `sendNewsletterBatch`) lives in `features/{ctx}/infrastructure/`, importing the generic bits from shared. `shared` never imports a feature.
- A use-case never crosses into another feature's internals — go through an API, an event, or the outbox.
- `domain/` never imports prisma, Zod-from-schema, or any I/O. Invariants only.

**Smell → fix:**
- `import type { Foo } from '@/features/shared/infrastructure/email'` inside a use-case → define `Foo` in `features/{ctx}/domain/{ctx}.types.ts`, both use-case and adapter import from there.
- `sendXBatch` / `renderXHtml` for one feature sitting in `shared/email.ts` → relocate to `features/x/infrastructure/`; export the generic render/send primitives from shared and have the feature file compose them.

## REST API Design

**Versioning** dès premier endpoint public :
```ts
app.enableVersioning({ type: VersioningType.URI })
@Controller({ path: 'orders', version: '1' })
```
- Maintenir N-1 en production
- Breaking change → nouvelle version, jamais modifier existant
- `Sunset` header pour dépréciations

**Réponses cohérentes** :
```ts
type ApiResponse<T> = { data: T; meta?: { page: number; total: number; limit: number } }
type ApiError = { error: { code: string; message: string } }
```

**Codes HTTP** : 200 GET, 201 POST créé, 204 DELETE, 400 validation, 401 non auth, 403 non autorisé, 404 not found (préférer à 403 pour cacher existence), 409 conflit, 422 règle métier, 429 rate limit.

**Pagination + filtres** :
```ts
export class PaginationDto {
  @IsOptional() @IsInt() @Min(1) @Type(() => Number) page = 1
  @IsOptional() @IsInt() @Min(1) @Max(100) @Type(() => Number) limit = 20
}
```

**Contract-first OpenAPI** : `@nestjs/swagger`, DTOs `@ApiProperty()` source de vérité, générer client TS via `openapi-typescript`.

## oRPC / tRPC Decision

**oRPC = défaut nouveau projet.** Fait tout ce que tRPC fait + génération OpenAPI native depuis même router.

| Cas | Choix |
|---|---|
| Nouveau monorepo fullstack TS | **oRPC** |
| Projet existant tRPC | Garder tRPC, migration optionnelle |
| API publique ou clients non-TS | oRPC + `@orpc/openapi` |
| NestJS + Swagger déjà en place | REST + `@nestjs/swagger` + openapi-typescript |
| Microservices inter-services | REST ou gRPC |

**oRPC patterns** :
```ts
import { os } from '@orpc/server'
const base = os.context<Context>()
const protectedProcedure = base.middleware(({ context, next }) => {
  if (!context.user) throw new ORPCError('UNAUTHORIZED')
  return next({ context: { ...context, user: context.user } })
})
export const appRouter = {
  user: {
    getById: protectedProcedure.input(z.object({ id: z.string().uuid() }))
      .handler(({ input, context }) => userRepo.findById(input.id, context.user)),
  },
}
export type AppRouter = typeof appRouter
```

**Sécurité oRPC** : context = Guards NestJS. `protectedProcedure` pour auth, `adminProcedure` pour RBAC. Zod input validation = `ValidationPipe({ whitelist: true })`. Rate limiting en HTTP middleware en amont (Hono/Express), pas dans procedures.

## Frontend/Backend Contract

**Backend définit. Frontend consomme. Aucun ne duplique.**

| Contexte | Solution |
|---|---|
| Monorepo fullstack TS (nouveau) | oRPC — type-safe + OpenAPI natif |
| Monorepo fullstack TS (tRPC) | Garder tRPC |
| REST + NestJS | `@nestjs/swagger` + openapi-typescript |
| GraphQL en place | graphql-codegen |

**Règles non-négociables** :
- Tout changement contrat = coordonné backend + frontend
- Frontend ne hardcode JAMAIS shapes API — types générés/partagés
- Backend ne expose JAMAIS types domaine internes — DTOs ou AppRouter
- Breaking change backend = nouvelle version API ou nouveau procedure — jamais modifier silencieusement

## Caching

**Cache-aside par défaut** :
```ts
const getCachedOrder = async (id: OrderId): Promise<Order> => {
  const cached = await cacheManager.get<Order>(`order:${id}`)
  if (cached) return cached
  const order = await orderRepo.findById(id)
  await cacheManager.set(`order:${id}`, order, 3600)
  return order
}

// Toujours invalider agrégats (listes) quand item change
const updateOrder = async (order: Order) => {
  await orderRepo.save(order)
  await cacheManager.del(`order:${order.id}`)
  await cacheManager.del(`orders:user:${order.userId}`)
}
```

**Next.js App Router** :
```ts
// fetch + cache tags
fetch(url, { next: { tags: [`order:${id}`], revalidate: 60 } })

// unstable_cache pour DB
const getCachedUser = unstable_cache(
  (id: string) => db.query.users.findFirst({ where: eq(users.id, id) }),
  ['user'],
  { tags: ['users'], revalidate: 3600 },
)

// Invalidation depuis Server Action
revalidateTag(`order:${orderId}`)
```

**Règles** :
- TTL explicite sur toutes clés — jamais cache sans expiration
- Cache-aside par défaut, write-through pour données critiques
- Distinguer cache HTTP (CDN), applicatif (Redis), DB (query cache)

## ORM

**Prisma par défaut.** Demander si projet a déjà autre chose.

```prisma
model Order {
  id        String      @id @default(uuid())
  userId    String      @map("user_id")
  status    OrderStatus @default(PENDING)
  total     Decimal     @db.Decimal(10, 2)
  createdAt DateTime    @default(now()) @map("created_at")
  user      User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@index([userId])
  @@index([status, createdAt])
  @@map("orders")
}
```

**Règles non-négociables** :
- Index sur toutes FK (Prisma ne crée pas auto)
- Index composites pour `WHERE x AND ORDER BY y` fréquents
- `$transaction` pour ops multi-tables
- `explain analyze` (via `$queryRaw`) avant prod requêtes complexes
- Pagination cursor pour grands datasets — jamais `skip` naïf
- Soft delete via `deletedAt` sur entités sensibles
- Migrations versionnées, jamais modif directe schéma prod
- Types Prisma restent infra — jamais exportés vers domaine

**Mapper DB → Domain** :
```ts
import { Order as PrismaOrder } from '@prisma/client'
const toOrder = (row: PrismaOrder): Order => ({
  id: row.id as OrderId,
  userId: row.userId as UserId,
  status: row.status.toLowerCase() as Order['status'],
  total: row.total.toNumber(),
})
```

**Drizzle alt** si : équipe préfère SQL explicite, edge runtime (Cloudflare/Bun où Prisma limité), perf query critique.

## Security: Environment

**Fail fast au démarrage** :
```ts
import { z } from 'zod'
const EnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  REDIS_URL: z.string().url(),
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(['development', 'test', 'production']),
  CORS_ORIGINS: z.string().transform(s => s.split(',')),
})
export const env = EnvSchema.parse(process.env)
```

Variable manquante → crash démarrage. Jamais runtime.

## Security: Auth

**JWT + refresh token** :
- Access token court (15min), Refresh long (7j) en httpOnly cookie
- **Jamais** refresh token en localStorage
- Hash refresh token en DB pour invalidation

```ts
@Injectable()
export class AuthService {
  async login(user: AuthenticatedUser) {
    const accessToken = this.jwtService.sign(
      { sub: user.id, role: user.role },
      { secret: env.JWT_SECRET, expiresIn: '15m' }
    )
    const refreshToken = this.jwtService.sign(
      { sub: user.id },
      { secret: env.JWT_REFRESH_SECRET, expiresIn: '7d' }
    )
    await this.tokenRepo.saveRefreshToken(user.id, await bcrypt.hash(refreshToken, 10))
    return { accessToken, refreshToken }
  }
}
```

**Jamais** implémenter son propre crypto. Utiliser `@nestjs/jwt`, `bcrypt`.

## Security: Authorization (RBAC)

```ts
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles)

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(ctx: ExecutionContext): boolean {
    const required = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      ctx.getHandler(), ctx.getClass(),
    ])
    if (!required) return true
    const { user } = ctx.switchToHttp().getRequest()
    return required.includes(user.role)
  }
}

@Roles('admin')
@Delete(':id')
remove(@Param('id') id: string) {}
```

## Security: Rate Limiting

`@nestjs/throttler` granulaire par sensibilité :
```ts
ThrottlerModule.forRoot([{ ttl: 60000, limit: 100 }]) // défaut global
@Throttle({ default: { limit: 5, ttl: 60000 } })    // login - très restrictif
@Throttle({ default: { limit: 10, ttl: 60000 } })   // mutations sensibles
@Throttle({ default: { limit: 200, ttl: 60000 } })  // lecture publique
```

## Security: Data Exposure

**Colonnes sensibles** (`password`, `refreshTokenHash`, `totpSecret`) jamais dans query standard.

```ts
// TypeORM
@Column({ select: false })
password: string

// Prisma — omit dans queries courantes
const user = await prisma.user.findUnique({ where: { id }, omit: { password: true } })

// Quand explicitement nécessaire (auth)
const user = await repo.findOne({ where: { email }, select: ['id', 'email', 'password'] })
```

## Security: Crypto

**`crypto` du node, jamais `Math.random()`** :
```ts
import * as crypto from 'crypto'
const otp = crypto.randomInt(100000, 1000000)
const token = crypto.randomBytes(32).toString('hex')
const suffix = crypto.randomBytes(4).toString('hex')
```

Règle : `grep -r "Math.random" src/` → 0 résultats dans code auth/tokens/IDs.

## Security: Dev Endpoints

```ts
@Post('seed')
async seed() {
  if (env.NODE_ENV === 'production') throw new ForbiddenException('Disabled in production')
  // seed logic
}
```

Jamais merger seed sans guard. Vérifier `grep -r "seed\(\)" src/` avant déploiement.

## Security: Surface

```ts
app.use(helmet())
app.enableCors({ origin: env.CORS_ORIGINS, credentials: true }) // jamais '*' en prod
app.use(compression())
app.setGlobalPrefix('api')
app.useGlobalFilters(new GlobalExceptionFilter(env.NODE_ENV)) // jamais stack traces en prod
```

## Security: Validation

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,                                      // élimine champs inconnus
  forbidNonWhitelisted: true,                           // erreur si champ inconnu
  transform: true,                                      // string → number pour query params
  transformOptions: { enableImplicitConversion: false },
}))
```

## Security: Injection

- Prisma paramétrise par défaut. `$queryRaw` uniquement avec `Prisma.sql\`\`` tagged template, jamais string interpolation
- Toute donnée externe (webhooks, APIs tierces, queue messages) validée Zod avant traitement
- Pas de `eval()`, `Function()`, ou template dynamique non sanitisé

## Error Handling

**Result type pour erreurs métier dans domaine** :
```ts
type Ok<T> = { success: true; data: T }
type Err<E extends string> = { success: false; error: E }
type Result<T, E extends string> = Ok<T> | Err<E>

const ok = <T>(data: T): Ok<T> => ({ success: true, data })
const err = <E extends string>(error: E): Err<E> => ({ success: false, error })
```

**Stratégie par couche** :
- **Domain** : retourne `Result<T, E>` — JAMAIS throw dans fonction pure
- **Infrastructure** : throw erreurs typées (`DatabaseError`, `CacheError`)
- **Application** (services NestJS) : unwrap Result, throw `DomainError`
- **Presentation** : un seul `ExceptionFilter` global → erreurs HTTP

```ts
@Catch(DomainError)
export class DomainExceptionFilter implements ExceptionFilter {
  catch(error: DomainError, host: ArgumentsHost) {
    const statusMap = {
      not_found: 404,
      cannot_cancel: 422,
      unauthorized: 401,
      conflict: 409,
    } satisfies Record<string, number>
    host.switchToHttp().getResponse()
      .status(statusMap[error.code] ?? 500)
      .json({ error: { code: error.code, message: error.message } })
  }
}
```

## Queues

**BullMQ via `@nestjs/bullmq`** dès que : traitement >quelques secondes, peut échouer/rejouer, parallélisable.

```ts
type SendEmailJob = { to: string; subject: string; templateId: string; variables: Record<string, string> }

// Producer
@Injectable()
export class OrderService {
  constructor(@InjectQueue('emails') private emailQueue: Queue) {}
  async confirmOrder(orderId: OrderId) {
    const order = await this.orderRepo.findById(orderId)
    await this.emailQueue.add('order-confirmed', {
      to: order.userEmail, subject: 'Commande confirmée',
      templateId: 'order-confirmed', variables: { orderId },
    } satisfies SendEmailJob, {
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
      removeOnComplete: 100,
      removeOnFail: 500,
    })
  }
}

// Consumer
@Processor('emails')
export class EmailProcessor extends WorkerHost {
  async process(job: Job<SendEmailJob>) {
    const handlers = {
      'order-confirmed': this.sendOrderConfirmation.bind(this),
      'password-reset': this.sendPasswordReset.bind(this),
    } satisfies Record<string, (job: Job<SendEmailJob>) => Promise<void>>
    const handler = handlers[job.name]
    if (!handler) throw new Error(`Unknown job: ${job.name}`)
    return handler(job)
  }
}
```

**Règles** :
- Producer / consumer dans modules distincts (voire processus distincts)
- `attempts` + `backoff exponential` sur tous jobs avec API externe
- `removeOnComplete` + `removeOnFail` pour éviter accumulation Redis
- BullBoard en dev pour inspection

## Inter-services

**Choix pattern** :
| Besoin | Pattern |
|---|---|
| Réponse immédiate | HTTP REST ou tRPC/oRPC |
| Tolérance latence | Message queue (BullMQ, RabbitMQ, Kafka) |
| Broadcast vers N consommateurs | Events (`@nestjs/event-emitter` ou broker externe) |

**Events internes** (`@nestjs/event-emitter`) :
```ts
type OrderCancelledEvent = { orderId: OrderId; userId: UserId; cancelledAt: Date }

this.eventEmitter.emit('order.cancelled', { orderId, userId, cancelledAt: new Date() } satisfies OrderCancelledEvent)

@OnEvent('order.cancelled')
async handleOrderCancelled({ orderId, userId }: OrderCancelledEvent) {
  await this.notificationService.notifyUser(userId, `Order ${orderId} cancelled`)
}
```

**Outbox pattern** quand event doit absolument être publié post-DB :
```ts
await prisma.$transaction([
  prisma.order.update({ where: { id: orderId }, data: { status: 'CANCELLED' } }),
  prisma.outboxEvent.create({ data: { type: 'OrderCancelled', payload: { orderId } } }),
])
// Worker lit table outbox + publie events → garantit at-least-once delivery
```

**Appels HTTP inter-services** :
- Typer contrats (openapi-typescript ou tRPC partagé)
- Timeout explicite sur tous appels sortants
- Circuit breaker pour services externes critiques
- Jamais faire confiance données reçues — Zod validation

## Observability

**Logs structurés Pino** :
```ts
PinoModule.forRoot({
  pinoHttp: {
    level: env.NODE_ENV === 'production' ? 'info' : 'debug',
    transport: env.NODE_ENV !== 'production'
      ? { target: 'pino-pretty', options: { colorize: true } }
      : undefined,
    serializers: {
      req: (req) => ({ method: req.method, url: req.url, id: req.id }),
      res: (res) => ({ statusCode: res.statusCode }),
    },
    redact: ['req.headers.authorization', 'req.body.password'],
  },
})
```

**Logger règles** :
- Toujours logger : erreurs avec context, actions métier importantes, perf requêtes lentes
- Jamais logger : passwords, tokens, PII (sauf obligation légale + masquage), bodies/résultats DB volumineux

```ts
this.logger.error({ err, orderId, userId }, 'Failed to process order')
this.logger.info({ orderId, duration }, 'Order processed successfully')
```

**Health checks** (`@nestjs/terminus`) :
```ts
@Controller('health')
export class HealthController {
  @Get()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      () => this.redis.checkHealth('redis'),
      () => this.disk.checkStorage('storage', { thresholdPercent: 0.9, path: '/' }),
    ])
  }
}
```
- `/health/live` (liveness — process tourne)
- `/health/ready` (readiness — dépendances dispo)
- Pas exposer publiquement sans auth basique

## Testing

- **Jest + `@nestjs/testing`** (`Test.createTestingModule`)
- Tests unitaires domaine : fonctions pures, zéro mock, zéro NestJS
- Tests intégration services : mock repos avec `jest.fn()`
- Tests E2E controllers : Supertest sur app NestJS complète + DB test réelle
- Mocks API externes : Nock ou MSW (demander si déjà en place)

## Checkpoints

Pause et confirme avant :
- **Bulk migration** REST → oRPC (touche tous controllers + types frontend)
- **Switch Prisma ↔ Drizzle** ou autre ORM
- **Adding global guards/filters/interceptors** (`useGlobalGuards`) — affecte toutes routes
- **Schéma DB drop column / change constraint** — irréversible sans backup
- **Adding rate limit / auth globally** — peut casser endpoints publics existants
- **Outbox pattern** sur tables existantes (migration + worker setup)
- **Switch logger** (`Logger` natif → Pino) — affecte tous les emit
- **Disable health check auth** — surface attaque
- **Bulk replace `Math.random` → `crypto`** sur code auth (peut révéler bugs latents)

Single-file fixes : apply directly, cite construct.

## Output Format

Diff: `// {construct}: {one-line why}`
Review: `path:line — {construct}: {problem}. {fix}.`

**Recall vs precision** : flag speculative matches (possible BOLA gap, possible missing index, possible cache invalidation gap) avec `[speculative]`. Ne pas drop ambigus.

## Worked Example

**Input:** "Add endpoint POST /orders/:id/cancel with auth + ownership check + cache invalidation."

1. *Classify* → "REST endpoint" + "auth" + "ownership check" + "caching".
2. *Read* → REST API Design + Security: Auth + Caching.
3. *Detect triggers* → ownership pattern (BOLA), cache invalidation aggregates, exception filter mapping.
4. *Apply*:
   ```ts
   @Post(':id/cancel')
   @UseGuards(JwtAuthGuard)
   async cancel(@Param('id') id: OrderId, @Req() req: AuthenticatedRequest) {
     // BOLA: ownership check + 404 (not 403) to hide existence
     const order = await this.repo.findFirst({ where: { id, createdBy: req.user.id } })
     if (!order) throw new NotFoundException()
     // Domain Result<T, E>
     const result = cancelOrder(order)
     if (!result.success) throw new DomainError(result.error) // → ExceptionFilter maps to HTTP
     await this.repo.save(result.data)
     // Cache invalidation: item + aggregate
     await this.cache.del(`order:${id}`)
     await this.cache.del(`orders:user:${req.user.id}`)
     return { data: toDto(result.data) }
   }
   ```
5. *Cite* `// BOLA: ownership filter`, `// Result<T,E>: domain pure`, `// cache aggregate invalidation`.

## Fallbacks & Edge Cases

- **Project uses Express/Fastify (not NestJS)** → defer to `nodejs-backend-patterns`. Most patterns apply (Result, Outbox, Pino, BullMQ, Zod env).
- **Project uses Hono / edge runtime** → defer to `hono` skill. Some patterns differ (no NestJS guards, use Hono middleware).
- **Project uses tRPC** → adapt oRPC patterns; sécurité similar (context, middleware).
- **Project uses Drizzle (not Prisma)** → ORM rules apply, syntax differs (`db.transaction(...)` vs `prisma.$transaction`).
- **Microservice architecture** → API versioning per-service; events via broker (RabbitMQ/Kafka), not in-process emitter.
- **Serverless functions** → no persistent connection pool — use connection per invocation, BullMQ may not apply (use SQS/managed queue).
- **Test files** → relax some prod rules (mock allowed, env validation can use test fixtures).
- **Full audit requested** → walk every Task → Section row touching changed files.
