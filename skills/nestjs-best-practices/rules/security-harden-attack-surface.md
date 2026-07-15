---
title: Harden the App-Level Attack Surface
impact: HIGH
impactDescription: Missing headers/CORS/compression/prefix hardening exposes stack traces, allows any-origin requests, or leaks framework fingerprint
tags: security, cors, headers, bootstrap, surface
---

## Harden the App-Level Attack Surface

Attack-surface hardening happens once, in `main.ts`, at bootstrap — not per-controller. Missing any of these is a silent gap that only shows up during a pentest or an incident.

**Incorrect (default bootstrap, no hardening):**

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors(); // reflects any origin
  await app.listen(3000);
}
```

**Correct (explicit hardening at bootstrap):**

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.use(helmet());
  app.enableCors({
    origin: process.env.CORS_ORIGINS?.split(','), // never '*' in production
    credentials: true,
  });
  app.use(compression());
  app.setGlobalPrefix('api');
  app.useGlobalFilters(new GlobalExceptionFilter(process.env.NODE_ENV));

  await app.listen(3000);
}
```

Each piece closes a specific gap:
- `helmet()` — sets security headers (`X-Frame-Options`, `X-Content-Type-Options`, etc.), removes `X-Powered-By`.
- `enableCors({ origin: allowlist })` — never `origin: '*'` with `credentials: true`; that combination is rejected by browsers anyway but signals a misconfigured policy.
- `compression()` — response size, not a security control by itself, but expected at this layer.
- `setGlobalPrefix('api')` — namespaces routes, avoids accidental collision with static/health routes.
- `useGlobalFilters(new GlobalExceptionFilter(env))` — the filter must strip stack traces when `NODE_ENV === 'production'`; leaking stack traces reveals file paths and dependency versions.

Pair this with `security-guard-dev-endpoints` (route-level) and `security-validate-all-input` (input-level) — this rule is the outer perimeter, not a substitute for the other two.
