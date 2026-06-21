# Error Handling Conventions

## Layers and Their Error Types

| Layer | Error type | Example |
|---|---|---|
| Service | Domain errors (plain `Error` subclasses) | `UserNotFoundError`, `SubscriptionExpiredError` |
| Transport (tRPC) | `TRPCError` | `{ code: "NOT_FOUND" }` |
| Transport (ORPC) | `ORPCError` | `new ORPCError("NOT_FOUND")` |
| External adapters | Wraps vendor errors with context | `new EmailSendError(cause)` |

**Rule:** Service classes never throw transport errors. Procedures map service errors to transport errors.

---

## Domain Error Classes

Define domain errors alongside the service file:

```typescript
// services/user.service.ts

export class UserNotFoundError extends Error {
  constructor(id: string) {
    super(`User not found: ${id}`)
    this.name = "UserNotFoundError"
  }
}

export class UserAlreadyExistsError extends Error {
  constructor(email: string) {
    super(`User already exists with email: ${email}`)
    this.name = "UserAlreadyExistsError"
  }
}
```

Use a `cause` for wrapping underlying errors:

```typescript
export class DatabaseError extends Error {
  constructor(message: string, cause?: unknown) {
    super(message, { cause })
    this.name = "DatabaseError"
  }
}
```

---

## tRPC Error Mapping

```typescript
// router/user.ts
import { TRPCError } from "@trpc/server"
import { UserNotFoundError, UserAlreadyExistsError } from "../services/user.service"

function mapServiceError(err: unknown): TRPCError {
  if (err instanceof UserNotFoundError) {
    return new TRPCError({ code: "NOT_FOUND", message: err.message })
  }
  if (err instanceof UserAlreadyExistsError) {
    return new TRPCError({ code: "CONFLICT", message: err.message })
  }
  // Unknown errors become INTERNAL_SERVER_ERROR
  return new TRPCError({ code: "INTERNAL_SERVER_ERROR", cause: err })
}

export const userRouter = router({
  createUser: publicProcedure
    .input(createUserSchema)
    .mutation(async ({ input, ctx }) => {
      try {
        return await ctx.services.user.create(input)
      } catch (err) {
        throw mapServiceError(err)
      }
    }),
})
```

---

## ORPC Error Mapping

```typescript
// router/user.ts (ORPC)
import { ORPCError } from "@orpc/server"

function mapServiceError(err: unknown): ORPCError {
  if (err instanceof UserNotFoundError) {
    return new ORPCError("NOT_FOUND", { message: err.message })
  }
  if (err instanceof UserAlreadyExistsError) {
    return new ORPCError("CONFLICT", { message: err.message })
  }
  return new ORPCError("INTERNAL_SERVER_ERROR", { cause: err })
}
```

---

## Provider Error Wrapping

Adapters catch vendor errors and wrap them with context:

```typescript
// providers/email/resend.adapter.ts
export class EmailSendError extends Error {
  constructor(message: string, cause?: unknown) {
    super(message, { cause })
    this.name = "EmailSendError"
  }
}

export const resendAdapter: EmailProvider = {
  async send(options) {
    try {
      const result = await resend.emails.send({ ... })
      if (!result.data?.id) throw new Error(JSON.stringify(result.error))
      return { id: result.data.id }
    } catch (err) {
      throw new EmailSendError(`Failed to send email to ${options.to}`, err)
    }
  },
}
```

Services catch provider errors and decide whether to propagate or handle them:

```typescript
// services/email.service.ts
async sendWelcomeEmail(userId: string): Promise<void> {
  const user = await this.db.user.findUniqueOrThrow({ where: { id: userId } })
  try {
    await this.provider.send({
      to: user.email,
      subject: "Welcome!",
      html: renderWelcomeEmail(user),
    })
  } catch (err) {
    // Log but don't fail the parent operation for non-critical emails
    console.error("Failed to send welcome email", err)
  }
}
```

---

## Client Error Handling (tRPC React Query)

```typescript
// apps/web/src/components/profile-form.tsx
const updateProfile = api.user.updateProfile.useMutation({
  onError(err) {
    if (err.data?.code === "CONFLICT") {
      toast.error("That username is already taken")
      return
    }
    toast.error("Something went wrong. Please try again.")
  },
})
```
