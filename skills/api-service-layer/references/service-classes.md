# Service Classes: Conventions and Patterns

## Role

Service classes own **all business logic**. They are the only layer that:
- Runs complex queries against the database
- Orchestrates multiple steps (validate → write → send notification)
- Enforces domain rules (e.g., user can only have one active subscription)
- Calls provider abstractions for external integrations

Service classes are **transport-agnostic**: they know nothing about HTTP, tRPC, ORPC, or any request/response format.

---

## Constructor Signature

Services receive their dependencies via constructor, never via module-level imports:

```typescript
// packages/api/src/services/user.service.ts
import type { PrismaClient } from "@myapp/db"
import type { EmailService } from "./email.service"

type UserServiceDeps = {
  db: PrismaClient
  emailService: EmailService
}

export class UserService {
  private db: PrismaClient
  private email: EmailService

  constructor(deps: UserServiceDeps) {
    this.db = deps.db
    this.email = deps.email
  }

  async getProfile(userId: string): Promise<UserProfile> { ... }
  async updateProfile(userId: string, data: UpdateProfileInput): Promise<UserProfile> { ... }
  async deleteAccount(userId: string): Promise<void> { ... }
}
```

**Rule:** The `db` client is always passed in — never import the Prisma singleton directly inside a service. This keeps services testable.

---

## Method Contracts

Each method has a clear contract:

```typescript
// Return type is explicit — never `any`
async getProfile(userId: string): Promise<UserProfile>

// Input is the validated domain type, not the raw procedure input
async updateProfile(userId: string, data: UpdateProfileInput): Promise<UserProfile>

// Void for mutations that don't return data
async deleteAccount(userId: string): Promise<void>
```

**Rule:** Service methods throw on error — they do not return `null | undefined` for "not found" cases. Callers catch and handle errors.

---

## Error Handling

Services throw plain JavaScript errors or typed domain errors. They do NOT throw `TRPCError` or `ORPCError` — those are transport concerns.

```typescript
// services/user.service.ts
export class UserNotFoundError extends Error {
  constructor(userId: string) {
    super(`User not found: ${userId}`)
    this.name = "UserNotFoundError"
  }
}

export class UserService {
  async getProfile(userId: string): Promise<UserProfile> {
    const user = await this.db.user.findUnique({ where: { id: userId } })
    if (!user) throw new UserNotFoundError(userId)
    return formatUserProfile(user)
  }
}
```

The procedure maps service errors to transport errors:

```typescript
// router/user.ts (tRPC)
getProfile: protectedProcedure.query(async ({ ctx }) => {
  try {
    return await ctx.services.user.getProfile(ctx.session.user.id)
  } catch (err) {
    if (err instanceof UserNotFoundError) {
      throw new TRPCError({ code: "NOT_FOUND", message: err.message })
    }
    throw err  // re-throw unknown errors — let tRPC handle them
  }
})
```

---

## Service Composition

Services can depend on other services. Pass them via constructor:

```typescript
type SubscriptionServiceDeps = {
  db: PrismaClient
  stripe: StripeProvider
  emailService: EmailService
}

export class SubscriptionService {
  constructor(private deps: SubscriptionServiceDeps) {}

  async createSubscription(userId: string, priceId: string) {
    const user = await this.deps.db.user.findUniqueOrThrow({ where: { id: userId } })
    const subscription = await this.deps.stripe.createSubscription(user.stripeCustomerId, priceId)
    await this.deps.db.subscription.create({ data: { userId, stripeId: subscription.id } })
    await this.deps.emailService.sendSubscriptionConfirmation(user.email)
    return subscription
  }
}
```

---

## Test Requirements

**Every service class must have unit tests.** Tests are added or updated whenever a service is created or changed.

```typescript
// services/__tests__/user.service.test.ts
import { describe, test, expect, vi } from "vitest"
import { UserService, UserNotFoundError } from "../user.service"

const mockDb = {
  user: {
    findUnique: vi.fn(),
    update: vi.fn(),
  },
}
const mockEmailService = { sendWelcome: vi.fn() }

const service = new UserService({ db: mockDb as any, emailService: mockEmailService as any })

describe("UserService", () => {
  describe("getProfile", () => {
    test("returns formatted profile when user exists", async () => {
      mockDb.user.findUnique.mockResolvedValue({ id: "1", name: "Alice", email: "alice@example.com" })
      const profile = await service.getProfile("1")
      expect(profile.name).toBe("Alice")
    })

    test("throws UserNotFoundError when user does not exist", async () => {
      mockDb.user.findUnique.mockResolvedValue(null)
      await expect(service.getProfile("missing")).rejects.toThrow(UserNotFoundError)
    })
  })
})
```
