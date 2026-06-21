---
name: better-t-stack
description: Use when scaffolding, working in, or reasoning about a Better-T-Stack (BTS) project. Covers the standard BTS monorepo layout, technology choices and personal defaults (ORM, database, auth, package manager, linting), and how the stack options compose. Trigger when the user asks about BTS setup, technology choices within a BTS project, or how to extend a BTS scaffold with additional packages or patterns.
---

# Better-T-Stack

Personal conventions for projects created with [Better-T-Stack](https://better-t-stack.amanv.dev/) (BTS). BTS is an opinionated fullstack monorepo CLI that scaffolds Next.js + Expo Native apps with tRPC, auth, a database/ORM, and common tooling.

## References

```text
references/
  stack-choices.md      ORM options (Prisma vs Drizzle), database options, package manager choices
  project-structure.md  Standard BTS package layout and what each package owns
  tooling-setup.md      Biome, Lefthook, Ruler, Turborepo configuration patterns
  env-config.md         Centralized root .env, multi-environment files, packages/env split by runtime
  bts-jsonc.md          bts.jsonc format and what it tracks
```

## When to use

- **Starting a new BTS project** — choosing stack options, understanding the scaffold output
- **Adding features to a BTS project** — knowing which package to touch, how to wire new services
- **Debugging BTS-specific wiring** — auth setup, tRPC client/server wiring, Expo integration
- **Evaluating technology choices** — comparing Prisma vs Drizzle, SQLite vs Postgres, Bun vs pnpm
- **Setting up environment variables** — centralized root `.env`, multi-environment files, `packages/env` split

## Quick Reference

### Standard BTS scaffold command

```bash
bunx create-better-t-stack@latest
```

Interactive prompts select:
- **Package manager**: Bun (personal default) or pnpm
- **Frontend**: Next.js, Next.js + Native, or Native only
- **Native styling**: NativeWind (Expo) or Uniwind (HeroUI Native)
- **Backend**: Self-hosted (Next.js Route Handlers) or Hono
- **Database**: SQLite (local/Turso), PostgreSQL (Supabase/Neon), MySQL
- **ORM**: Prisma or Drizzle
- **Auth**: Better Auth (default) or none
- **Addons**: Biome, Lefthook, Turborepo, Ruler, Skills, MCP

### Personal defaults

| Option | Default | When to deviate |
|---|---|---|
| Package manager | Bun | pnpm when strict isolation needed |
| ORM | Prisma | Drizzle for edge/serverless (Turso, Cloudflare D1) |
| Database | Postgres (Supabase) | SQLite/Turso for small projects or pure edge |
| Native styling | Uniwind (HeroUI Native) | NativeWind for custom design systems |
| Linting | Biome | — |
| Git hooks | Lefthook | — |
| Orchestration | Turborepo | — |
| Agent sync | Ruler | — |

See `references/stack-choices.md` for tradeoffs.

### Environment variables — centralized at the root

**The BTS default scatters `.env` files inside each app. This pattern does not follow that.** All env files live at the monorepo root and are loaded from there by `packages/env`.

```
/                     ← monorepo root
  .env                ← local development (gitignored)
  .env.example        ← committed template for new devs
  .env.preview        ← preview/staging values (gitignored)
  .env.production     ← production values (gitignored)
  packages/env/       ← reads the root .env, validates per-runtime
  apps/web/           ← no own .env
  apps/native/        ← no own .env
```

`packages/env/src/server.ts` resolves the workspace root from `import.meta.url` and loads the correct file with `dotenv`:

```typescript
const workspaceRoot = path.resolve(
  path.dirname(fileURLToPath(import.meta.url)),
  "../../..", // packages/env/src → packages/env → packages → root
)
dotenv.config({ path: path.join(workspaceRoot, process.env.APP_ENV_FILE ?? ".env") })
```

To use a different env file locally:

```bash
APP_ENV_FILE=.env.preview bun run dev
APP_ENV_FILE=.env.production bun run dev
```

In CI/CD (Vercel, GitHub Actions), variables are injected directly — no file is read.

See `references/env-config.md` for the full `server.ts`, `web.ts`, `native.ts` implementation and import rules.

### Adding features after scaffold

```bash
bunx create-better-t-stack add
```

Available addons: Starlight docs, Fumadocs, Biome, Oxlint, Ruler, Turborepo, PWA, Tauri, Husky.

### Database commands (from repo root)

```bash
bun run db:generate    # Generate Prisma/Drizzle client artifacts
bun run db:push        # Push schema changes (dev — no migration file)
bun run db:migrate     # Run migrations (prod)
bun run db:studio      # Open database UI (port 3001)
```

### Dev ports (personal standard)

| Service | Port |
|---|---|
| `apps/web` | 3000 |
| DB Studio (Prisma/Drizzle) | 3001 |
| React Email dev | 3002 |
| Expo Atlas | 3003 |

### Workspace helper: bun f

Run any `bun` command inside a specific workspace without `cd`. Accepts package name, folder name, short name, or partial match:

```bash
bun f db run dev
bun f web add zod
bun f native run atlas
```

Wire it in the root `package.json`:

```json
{ "scripts": { "f": "bash ./scripts/f" } }
```

Copy `scripts/f` from this skill into the project's `scripts/` directory. The script is included alongside this skill at `scripts/f`.
