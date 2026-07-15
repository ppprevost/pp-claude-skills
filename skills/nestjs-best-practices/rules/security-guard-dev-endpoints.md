---
title: Guard or Remove Development-Only Endpoints
impact: HIGH
impactDescription: Seed/debug/reset endpoints left reachable in production allow data destruction or privilege escalation
tags: security, environment, production, seed
---

## Guard or Remove Development-Only Endpoints

Seed routes, debug dumps, and reset/test-login endpoints are convenient in development and dangerous in production. Every such endpoint must explicitly refuse to run outside development, checked at the top of the handler — not left to routing configuration or "we'll remove it before merge."

**Incorrect (dev-only endpoint mergeable to production):**

```typescript
@Controller('debug')
export class DebugController {
  @Post('seed')
  async seed() {
    await this.seedService.run(); // wipes and reseeds tables — no guard
    return { ok: true };
  }

  @Get('dump-users')
  dumpUsers() {
    return this.userService.findAll(); // full user dump, no auth
  }
}
```

**Correct (explicit environment guard, fails closed):**

```typescript
@Controller('debug')
export class DebugController {
  @Post('seed')
  async seed() {
    if (process.env.NODE_ENV === 'production') {
      throw new ForbiddenException('Disabled in production');
    }
    await this.seedService.run();
    return { ok: true };
  }
}
```

Better: don't register the controller/module at all in production.

```typescript
@Module({
  imports: [
    ...(process.env.NODE_ENV !== 'production' ? [DebugModule] : []),
  ],
})
export class AppModule {}
```

Before deploying, grep for common dev-endpoint patterns and confirm each one is either removed or guarded:

```bash
grep -rn "seed\(\)\|dump-users\|/debug\|dev-login" src/
```

`dev-login` and similar auth-bypass routes need the same treatment — never rely on "nobody will guess the URL."
