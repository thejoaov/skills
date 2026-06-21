# Provider Interface + Adapter Pattern

## Purpose

External services (email, SMS, push notifications, storage, payments, AI) are accessed through a **provider interface**. Each interface has one or more **adapters** that implement it. The active adapter is selected in the context factory (usually based on `NODE_ENV` or config).

This means:
- In development, use a local/fake adapter (Mailpit instead of Resend, mock instead of Stripe)
- In production, use the real adapter
- In tests, use an in-memory mock
- Swapping providers never changes service classes

---

## Structure

```
packages/api/src/providers/
  email/
    email.provider.ts       Interface + shared types
    resend.adapter.ts       Production implementation
    mailpit.adapter.ts      Local dev implementation
  sms/
    sms.provider.ts
    twilio.adapter.ts
    mock-sms.adapter.ts
  push/
    push.provider.ts
    expo-push.adapter.ts
  storage/
    storage.provider.ts
    supabase-storage.adapter.ts
    local-storage.adapter.ts
```

---

## Email Example

### Interface

```typescript
// providers/email/email.provider.ts

export type SendEmailOptions = {
  to: string | string[]
  subject: string
  html: string
  from?: string
  replyTo?: string
}

export type SendEmailResult = {
  id: string
}

export interface EmailProvider {
  send(options: SendEmailOptions): Promise<SendEmailResult>
}
```

### Resend Adapter (production)

```typescript
// providers/email/resend.adapter.ts
import { Resend } from "resend"
import { serverEnv } from "@myapp/env/server"
import type { EmailProvider } from "./email.provider"

const resend = new Resend(serverEnv.RESEND_API_KEY)

export const resendAdapter: EmailProvider = {
  async send({ to, subject, html, from, replyTo }) {
    const result = await resend.emails.send({
      from: from ?? "noreply@myapp.com",
      to,
      subject,
      html,
      replyTo,
    })
    if (!result.data?.id) throw new Error(`Resend failed: ${JSON.stringify(result.error)}`)
    return { id: result.data.id }
  },
}
```

### Mailpit Adapter (local dev)

```typescript
// providers/email/mailpit.adapter.ts
import type { EmailProvider } from "./email.provider"

// Mailpit runs locally at port 1025 (SMTP) or 8025 (web UI)
export const mailpitAdapter: EmailProvider = {
  async send({ to, subject, html, from }) {
    const response = await fetch("http://localhost:8025/api/v1/send", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        From: { Email: from ?? "dev@localhost" },
        To: Array.isArray(to) ? to.map((e) => ({ Email: e })) : [{ Email: to }],
        Subject: subject,
        HTML: html,
      }),
    })
    const data = await response.json()
    return { id: data.id ?? "mailpit-dev" }
  },
}
```

### Context wiring

```typescript
// context.ts
const emailProvider =
  serverEnv.NODE_ENV === "production" ? resendAdapter : mailpitAdapter

const emailService = new EmailService({ provider: emailProvider })
```

---

## Naming Conventions

| Type | Naming | Example |
|---|---|---|
| Interface file | `<domain>.provider.ts` | `email.provider.ts` |
| Interface export | `<Domain>Provider` | `EmailProvider` |
| Adapter file | `<name>.adapter.ts` | `resend.adapter.ts` |
| Adapter export | `<name>Adapter` (const) | `resendAdapter` |

---

## When to create a provider

Create a provider interface whenever:
1. You need to call an external API that might change (different vendor in prod vs dev)
2. You want to test service logic without making real external calls
3. The integration might have multiple implementations in the same project (e.g., multiple SMS vendors by region)

Do **not** create a provider for:
- Internal database calls (use the db client directly in services)
- Simple utility functions (crypto, date formatting, etc.)
- Auth (Better Auth has its own adapter model)
