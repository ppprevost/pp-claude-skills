---
title: Choose the Right API/RPC Layer for the Stack
impact: MEDIUM
impactDescription: Wrong contract layer causes duplicated types, drift between frontend and backend, or unnecessary REST boilerplate
tags: api, architecture, orpc, trpc, contract
---

## Choose the Right API/RPC Layer for the Stack

Don't default to hand-written REST + manually synced frontend types. Pick the contract layer based on project shape, and keep the decision consistent across the codebase.

| Situation | Choice |
|---|---|
| New fullstack TS monorepo | oRPC (superset of tRPC, generates OpenAPI from the same router) |
| Existing tRPC project | Keep tRPC; migrate to oRPC only if OpenAPI/public-client support is needed |
| Public API or non-TS clients | oRPC + `@orpc/openapi`, or REST + `@nestjs/swagger` |
| NestJS with Swagger already in place | Keep REST + `@nestjs/swagger` + `openapi-typescript` for client generation |
| Service-to-service (microservices) | REST or gRPC, not RPC-over-HTTP frameworks meant for browser clients |

**Incorrect (hand-rolled REST + manually duplicated frontend types):**

```typescript
// backend/user.controller.ts
@Controller('users')
export class UserController {
  @Get(':id')
  getUser(@Param('id') id: string) {
    return this.userService.findById(id);
  }
}

// frontend/types.ts — hand-copied, drifts silently when backend changes
type User = { id: string; name: string; email: string };
```

**Correct (oRPC — type-safe end-to-end, OpenAPI generated from the same router):**

```typescript
import { os, ORPCError } from '@orpc/server';
import { z } from 'zod';

const base = os.context<Context>();

const protectedProcedure = base.middleware(({ context, next }) => {
  if (!context.user) throw new ORPCError('UNAUTHORIZED');
  return next({ context: { ...context, user: context.user } });
});

export const appRouter = {
  user: {
    getById: protectedProcedure
      .input(z.object({ id: z.string().uuid() }))
      .handler(({ input, context }) => userRepo.findById(input.id, context.user)),
  },
};

export type AppRouter = typeof appRouter; // frontend imports this type, never redefines it
```

Non-negotiable rules once a layer is chosen:
- Backend defines the contract, frontend only consumes it. Never hand-duplicate response shapes.
- Backend never exposes internal domain entities directly — DTOs (REST) or router output types (RPC) only.
- Breaking a contract requires a new version/procedure, never a silent shape change.
- If using oRPC/tRPC alongside NestJS guards, wire the RPC context to reuse existing auth (`protectedProcedure` mirrors `@UseGuards()`), don't build a second auth system.

Reference: [oRPC docs](https://orpc.unnoq.com), [tRPC docs](https://trpc.io)
