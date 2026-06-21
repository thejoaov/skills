# Skills

[![skills.sh](https://skills.sh/b/thejoaov/skills)](https://skills.sh/thejoaov/skills)

Personal agent skills for building fullstack TypeScript products — from architecture decisions to feature planning.

These are small, composable, and opinionated. They reflect years of personal conventions, not generic best practices. Fork them, adapt them, make them yours.

## Install

```bash
npx skills add thejoaov/skills
```

Pick the skills you want and which agents to install them on. Works with OpenCode, Claude Code, Cursor, Copilot, and [70+ more](https://github.com/vercel-labs/skills#supported-agents).

## Skills

### Architecture

Conventions for structuring TypeScript monorepos and the API layer.

- **[monorepo-architecture](./skills/monorepo-architecture/SKILL.md)** — Workspace setup (Bun/pnpm), Turborepo pipelines and caching, package boundary conventions (`apps/packages/tooling` tiers), and dependency management with catalogs.

- **[better-t-stack](./skills/better-t-stack/SKILL.md)** — Personal defaults for [Better-T-Stack](https://better-t-stack.amanv.dev/) projects: ORM choice (Prisma vs Drizzle), database choice (SQLite/Turso vs Postgres), tooling (Biome, Lefthook, Ruler), and `@t3-oss/env` runtime config split.

- **[api-service-layer](./skills/api-service-layer/SKILL.md)** — Thin-transport pattern: tRPC or ORPC as a pure HTTP dispatcher, request-scoped context injection, service classes owning all business logic, and provider/adapter abstraction for external integrations (email, SMS, storage, etc.).

### Planning

Skills for turning ideas into structured, executable plans.

- **[to-prd](./skills/to-prd/SKILL.md)** — Synthesize the current conversation into a PRD saved as a Markdown file in `docs/`. No interview — just capture what's already been discussed, surface open questions, and produce a document ready for implementation.

- **[feature-management](./skills/feature-management/SKILL.md)** — Full lifecycle for a new feature: optional spike for technical unknowns, PRD as the source of truth, and a detailed task checklist to track implementation. Produces versioned, non-destructive documents under `docs/[feature-name]/`.
