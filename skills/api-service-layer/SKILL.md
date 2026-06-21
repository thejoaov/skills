---
name: api-service-layer
description: Use when designing or implementing the API layer of a fullstack TypeScript application. Covers the thin-transport pattern (tRPC or ORPC as HTTP transport only), request-scoped context injection, service classes as the business logic layer, and the provider/adapter pattern for external integrations. Applies regardless of whether the HTTP transport is tRPC, ORPC, Express, Fastify, or Hono — the service layer and provider pattern are transport-agnostic. Trigger when the user asks about where to put business logic, how to structure API routes, how to inject context (auth session, db, services), or how to abstract external dependencies.
---

# API Service Layer

This skill covers the layered architecture used across all personal projects. The core principle is **thin transport, fat services**: HTTP/RPC procedures are entry points only — they validate input, authorize, call a service, and return the result. Business logic lives in service classes. External dependencies are accessed through provider interfaces with swappable adapters.

## References

```text
references/
  thin-transport.md     tRPC and ORPC as pure HTTP transport — what procedures should and shouldn't do
  request-context.md    Request-scoped context: session, db, services, how to build and inject it
  service-classes.md    Service class conventions: constructor injection, method contracts, error handling
  providers-adapters.md Provider interface + adapter pattern for external integrations (email, SMS, storage, etc.)
  error-handling.md     TRPCError / ORPCError conventions, service-layer error types, client error mapping
```

## When to use

- **Adding a new API route/procedure** — understand where business logic goes (hint: not in the router)
- **Injecting a new dependency** (db, a new service, an external API) — understand context and constructor injection
- **Abstracting an external integration** — email, push notifications, storage, payments, SMS
- **Debugging a procedure** — tracing the request from transport → context → service → provider
- **Migrating between HTTP transports** — confirms that service layer is unchanged when swapping tRPC → ORPC or adding Express

## Architecture Overview

```
Request (HTTP)
  │
  ▼
Transport Layer  ←── tRPC / ORPC / Express / Fastify / Hono
  │                  Thin: parse input, authorize, call service, return result
  │                  Zero business logic here
  ▼
Request Context  ←── Built per-request: { session, db, services }
  │                  Injected into every procedure
  ▼
Service Layer    ←── Service classes receive context via constructor
  │                  Own all business logic and data access
  ▼
Provider Layer   ←── Abstract interfaces for external dependencies
  │                  Implementations are adapters (Resend, Twilio, S3, etc.)
  ▼
External Services / Database
```

## Quick Reference

### The three-layer rule

| Layer | Contains | Does NOT contain |
|---|---|---|
| Transport (router/procedure) | Input parsing, auth check, service call, response shaping | Business logic, DB queries, external API calls |
| Service class | Business logic, DB queries, orchestration | HTTP concerns, request/response types, transport errors |
| Provider/adapter | External API calls | Business logic, service state |

### Adding a new feature

1. Define the procedure in the router (thin — input + output types only)
2. Create or extend a service class to own the logic
3. If the feature needs a new external dependency, create a provider interface + adapter
4. Inject the adapter into the context or directly into the service constructor
5. Call the service from the procedure

See individual reference files for detailed patterns and code examples.
