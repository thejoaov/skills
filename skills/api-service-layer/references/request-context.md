# Request Context: Per-Request Dependency Injection

## What is the context?

The request context is a plain object constructed once per request. It carries everything a procedure or service needs: the authenticated session, the database client, and instantiated service classes. It is the single injection point — services and procedures never import these things directly.

---

## Context shape

```typescript
// packages/api/src/context.ts

import type { Session } from "@myapp/auth"
import type { PrismaClient } from "@myapp/db"
import type { UserService } from "./services/user.service"
import type { EmailService } from "./services/email.service"
import type { NotificationService } from "./services/notification.service"

export type RequestContext = {
  db: PrismaClient
  session: Session | null  // null for unauthenticated requests
  services: {
    user: UserService
    email: EmailService
    notification: NotificationService
  }
}

export type AuthenticatedContext = RequestContext & {
  session: Session  // guaranteed non-null after auth middleware
}
```

---

## Context factory

```typescript
// packages/api/src/context.ts
import { prisma } from "@myapp/db"
import { getSession } from "@myapp/auth"
import { UserService } from "./services/user.service"
import { EmailService } from "./services/email.service"
import { resendAdapter } from "./providers/email/resend.adapter"
import { mailpitAdapter } from "./providers/email/mailpit.adapter"
import { serverEnv } from "@myapp/env/server"

export async function createContext(opts: {
  req: Request
}): Promise<RequestContext> {
  const session = await getSession(opts.req)

  const emailProvider =
    serverEnv.NODE_ENV === "production" ? resendAdapter : mailpitAdapter

  // Services receive the db and provider dependencies — not the full context
  // This keeps service constructors testable without a full context object
  const emailService = new EmailService({ provider: emailProvider })
  const userService = new UserService({ db: prisma, emailService })

  return {
    db: prisma,
    session,
    services: {
      user: userService,
      email: emailService,
    },
  }
}
```

---

## tRPC context wiring

```typescript
// packages/api/src/trpc.ts
import { initTRPC, TRPCError } from "@trpc/server"
import type { RequestContext } from "./context"

const t = initTRPC.context<RequestContext>().create()

export const router = t.router
export const publicProcedure = t.procedure

// Auth middleware: throws if session is null
const isAuthed = t.middleware(({ ctx, next }) => {
  if (!ctx.session) {
    throw new TRPCError({ code: "UNAUTHORIZED" })
  }
  return next({ ctx: { ...ctx, session: ctx.session } })
})

export const protectedProcedure = t.procedure.use(isAuthed)
```

```typescript
// apps/web/src/app/api/trpc/[trpc]/route.ts (Next.js App Router)
import { fetchRequestHandler } from "@trpc/server/adapters/fetch"
import { appRouter } from "@myapp/api"
import { createContext } from "@myapp/api/context"

const handler = (req: Request) =>
  fetchRequestHandler({
    endpoint: "/api/trpc",
    req,
    router: appRouter,
    createContext: () => createContext({ req }),
  })

export { handler as GET, handler as POST }
```

---

## ORPC context wiring

```typescript
// packages/api/src/orpc.ts
import { createRouterClient, ORPCError, os } from "@orpc/server"
import type { RequestContext } from "./context"

export const base = os.context<RequestContext>()

const isAuthed = base.middleware(({ context, next }) => {
  if (!context.session) {
    throw new ORPCError("UNAUTHORIZED")
  }
  return next({ context: { ...context, session: context.session } })
})

export const protectedProcedure = base.use(isAuthed)
```

---

## What belongs in context vs. what doesn't

| Belongs in context | Does NOT belong in context |
|---|---|
| `db` (Prisma/Drizzle client) | Config/env values — import from `@myapp/env` directly |
| `session` (Better Auth) | Utility functions (use imports) |
| Service class instances | Per-route state (pass as procedure args) |
| Provider instances (if shared) | HTTP request/response objects (except for auth parsing) |

---

## Testing

Because context is constructed explicitly and services are passed via constructor, testing is straightforward:

```typescript
// Unit test for UserService
import { UserService } from "../services/user.service"
import { mockDeep } from "vitest-mock-extended"
import type { PrismaClient } from "@myapp/db"

const mockDb = mockDeep<PrismaClient>()
const mockEmailService = { sendWelcome: vi.fn() }

const service = new UserService({ db: mockDb, emailService: mockEmailService })

test("getProfile throws when user not found", async () => {
  mockDb.user.findUniqueOrThrow.mockRejectedValue(new Error("Not found"))
  await expect(service.getProfile("non-existent-id")).rejects.toThrow()
})
```
