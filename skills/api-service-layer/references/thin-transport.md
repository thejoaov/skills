# Thin Transport: tRPC and ORPC as HTTP Layers

## The Rule

**The transport layer is a dispatcher, not a business logic container.** A procedure does exactly four things:

1. Declare input/output types
2. Check authorization
3. Call a service method
4. Return the result

If a procedure is doing more than that, the logic belongs in a service class.

---

## tRPC

```typescript
// packages/api/src/router/user.ts
import { z } from "zod"
import { router, protectedProcedure } from "../trpc"
import { TRPCError } from "@trpc/server"

export const userRouter = router({
  // Good: thin procedure
  getProfile: protectedProcedure.query(async ({ ctx }) => {
    return ctx.services.user.getProfile(ctx.session.user.id)
  }),

  updateProfile: protectedProcedure
    .input(z.object({
      name: z.string().min(1).max(100),
      bio: z.string().max(500).optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      return ctx.services.user.updateProfile(ctx.session.user.id, input)
    }),
})
```

```typescript
// Anti-pattern: fat procedure (move this to UserService)
getProfile: protectedProcedure.query(async ({ ctx }) => {
  const user = await ctx.db.user.findUnique({  // ❌ DB query in procedure
    where: { id: ctx.session.user.id },
    include: { preferences: true },
  })
  if (!user) throw new TRPCError({ code: "NOT_FOUND" })
  const subscription = await stripe.subscriptions.retrieve(user.stripeId)  // ❌ External API in procedure
  return { ...user, isPremium: subscription.status === "active" }
})
```

---

## ORPC

ORPC is the successor/alternative to tRPC. The thin-transport rule applies identically. Treat ORPC as the HTTP transport and nothing more.

```typescript
// packages/api/src/router/user.ts (ORPC)
import { z } from "zod"
import { os } from "@orpc/server"
import { base } from "../orpc"

export const userRouter = base.router({
  getProfile: base.handler(async ({ context }) => {
    return context.services.user.getProfile(context.session.user.id)
  }),

  updateProfile: base
    .input(z.object({
      name: z.string().min(1).max(100),
      bio: z.string().max(500).optional(),
    }))
    .handler(async ({ input, context }) => {
      return context.services.user.updateProfile(context.session.user.id, input)
    }),
})
```

---

## Migrating from tRPC to ORPC

Because business logic lives in service classes and not in procedures, the migration is mechanical:

1. Replace `router(...)` → `base.router(...)`
2. Replace `protectedProcedure.query/mutation` → `base.handler()` with auth middleware
3. Replace `TRPCError` → ORPC error types
4. Service classes and providers are **unchanged**

---

## Transport-Agnostic Services

The same service works with any HTTP transport:

```typescript
// Works with tRPC, ORPC, Express, Fastify, or Hono
// The transport adapts to call the service — not the other way around

class UserService {
  constructor(private ctx: RequestContext) {}

  async getProfile(userId: string): Promise<UserProfile> {
    const user = await this.ctx.db.user.findUniqueOrThrow({
      where: { id: userId },
      include: { preferences: true },
    })
    return formatUserProfile(user)
  }
}
```

If you add an Express route alongside tRPC:

```typescript
// Express adapter — same service, different transport
app.get("/api/user/profile", requireAuth, async (req, res) => {
  const ctx = await createRequestContext(req)
  const profile = await ctx.services.user.getProfile(ctx.session.user.id)
  res.json(profile)
})
```
