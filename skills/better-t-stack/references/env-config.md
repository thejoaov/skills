# BTS Environment Config: @t3-oss/env

## Princípio central: .env único na raiz do monorepo

O BTS padrão espalha arquivos `.env` dentro de cada app (`apps/web/.env`, `apps/native/.env`). **Este padrão não é seguido.** Todos os projetos usam um único `.env` na raiz do monorepo, acessível a todos os pacotes e apps.

```
/                     ← raiz do monorepo
  .env                ← desenvolvimento local (gitignored)
  .env.example        ← template commitado no repositório
  .env.preview        ← variáveis de ambiente do ambiente preview (gitignored)
  .env.production     ← variáveis de ambiente de produção (gitignored)
  apps/
    web/              ← sem .env próprio
    native/           ← sem .env próprio
  packages/
    env/              ← lê o .env da raiz e valida por runtime
```

---

## Como o .env da raiz é carregado em packages/env

O `packages/env/src/server.ts` resolve o caminho até a raiz do workspace via `import.meta.url` e carrega o arquivo correto com `dotenv`:

```typescript
import path from "node:path"
import { fileURLToPath } from "node:url"
import { createEnv } from "@t3-oss/env-core"
import dotenv from "dotenv"
import { z } from "zod"

// Resolve a raiz do monorepo a partir do local do arquivo compilado
const workspaceRoot = path.resolve(
  path.dirname(fileURLToPath(import.meta.url)),
  "../../..", // packages/env/src → packages/env → packages → raiz
)

// Permite trocar o arquivo .env via variável de ambiente
const envFile = process.env.APP_ENV_FILE ?? ".env"

dotenv.config({
  path: path.join(workspaceRoot, envFile),
  quiet: true,
})

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().min(1),
    BETTER_AUTH_SECRET: z.string().min(32),
    BETTER_AUTH_URL: z.url(),
    // ...
    NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  },
  runtimeEnv: process.env,
  emptyStringAsUndefined: true,
})
```

**Por que `runtimeEnv: process.env` e não explícito?**
No servidor (Node.js/Next.js Route Handlers), `process.env` já está populado após o `dotenv.config`. Para o native (Expo), onde `process.env` é substituído pelo Expo bundler, o `runtimeEnv` precisa ser explícito — veja `native.ts` abaixo.

**Correção de path relativo para SQLite:**
Se `DATABASE_URL` começa com `file:`, o path é reescrito para absoluto antes de passar para o Prisma — caso contrário o processo filho do Prisma resolveria de um diretório diferente:

```typescript
const databaseUrl = process.env.DATABASE_URL
if (databaseUrl?.startsWith("file:")) {
  const sqlitePath = databaseUrl.slice("file:".length)
  process.env.DATABASE_URL = `file:${
    path.isAbsolute(sqlitePath)
      ? sqlitePath
      : path.resolve(workspaceRoot, sqlitePath)
  }`
}
```

---

## Múltiplos ambientes: .env.preview e .env.production

Além do `.env` de desenvolvimento, o padrão mantém arquivos por ambiente na raiz:

| Arquivo | Ambiente | Usado por |
|---|---|---|
| `.env` | Desenvolvimento local | `dotenv.config` padrão |
| `.env.example` | — | Template commitado; base para novos devs |
| `.env.preview` | Preview / staging | CI/CD ou carregado manualmente |
| `.env.production` | Produção | CI/CD ou plataforma de deploy |

Para carregar um ambiente diferente, setar a variável antes de rodar:

```bash
# Rodar com variáveis de preview
APP_ENV_FILE=.env.preview bun run dev

# Rodar com variáveis de produção (localmente, para testar)
APP_ENV_FILE=.env.production bun run dev
```

Em CI/CD (Vercel, GitHub Actions), as variáveis são injetadas diretamente no ambiente — o arquivo não é lido. O mecanismo de `APP_ENV_FILE` é para conveniência local.

---

## packages/env/src/server.ts

Variáveis de servidor. **Nunca importar de client components ou do app native.**

```typescript
import path from "node:path"
import { fileURLToPath } from "node:url"
import { createEnv } from "@t3-oss/env-core"
import dotenv from "dotenv"
import { z } from "zod"

const workspaceRoot = path.resolve(
  path.dirname(fileURLToPath(import.meta.url)),
  "../../..",
)

dotenv.config({
  path: path.join(workspaceRoot, process.env.APP_ENV_FILE ?? ".env"),
  quiet: true,
})

export const env = createEnv({
  server: {
    NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
    DATABASE_URL: z.string().min(1),
    BETTER_AUTH_SECRET: z.string().min(32),
    BETTER_AUTH_URL: z.url(),
    CORS_ORIGIN: z.string().min(1),
    RESEND_API_KEY: z.string().optional(),
    STRIPE_SECRET_KEY: z.string().min(1).optional(),
    STRIPE_WEBHOOK_SECRET: z.string().min(1).optional(),
    USE_MAILPIT: z.string().optional(),
    SMTP_HOST: z.string().optional(),
    SMTP_PORT: z.coerce.number().optional(),
    SKIP_SIGNUP_VERIFICATION: z.string().optional(),
    APP_ENV: z.enum(["development", "preview", "production"]).optional(),
  },
  runtimeEnv: process.env,
  emptyStringAsUndefined: true,
})
```

---

## packages/env/src/web.ts

Variáveis do Next.js acessíveis em client components — apenas `NEXT_PUBLIC_*`.

```typescript
import { createEnv } from "@t3-oss/env-nextjs"
import { z } from "zod"

export const env = createEnv({
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url().optional(),
    NEXT_PUBLIC_BETTER_AUTH_URL: z.string().url().optional(),
  },
  runtimeEnv: {
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
    NEXT_PUBLIC_BETTER_AUTH_URL: process.env.NEXT_PUBLIC_BETTER_AUTH_URL,
  },
  emptyStringAsUndefined: true,
})
```

O Next.js injeta `NEXT_PUBLIC_*` no bundle de client automaticamente — não é necessário carregar `dotenv` aqui.

---

## packages/env/src/native.ts

Variáveis do Expo — apenas `EXPO_PUBLIC_*` para o client. O `runtimeEnv` deve ser **explícito** (não `process.env`) porque o bundler do Expo substitui `process.env.X` estaticamente em tempo de build.

```typescript
import { createEnv } from "@t3-oss/env-core"
import { z } from "zod"

export const env = createEnv({
  clientPrefix: "EXPO_PUBLIC_",
  client: {
    EXPO_PUBLIC_SERVER_URL: z.string().url(),
    EXPO_PUBLIC_APP_ENV: z
      .enum(["development", "preview", "production"])
      .default("development"),
    EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().optional(),
    EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID: z.string().optional(),
    EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID: z.string().optional(),
  },
  // No Expo, process.env é substituído estaticamente pelo bundler.
  // Por isso o runtimeEnv precisa ser explícito — não usar process.env diretamente.
  runtimeEnv: {
    EXPO_PUBLIC_SERVER_URL: process.env.EXPO_PUBLIC_SERVER_URL,
    EXPO_PUBLIC_APP_ENV: process.env.EXPO_PUBLIC_APP_ENV,
    EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY: process.env.EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY,
    EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID: process.env.EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID,
    EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID: process.env.EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID,
  },
  emptyStringAsUndefined: true,
})
```

---

## Regras de importação

| De onde | Importar |
|---|---|
| `packages/api` (server) | `@myapp/env/server` |
| `apps/web` Route Handler / Server Component | `@myapp/env/server` |
| `apps/web` Client Component | `@myapp/env/web` (`NEXT_PUBLIC_*` apenas) |
| `apps/native` | `@myapp/env/native` (`EXPO_PUBLIC_*` apenas) |

**Nunca importar `@myapp/env/server` de um client component ou do app native.**

---

## .env.example — o template commitado

O único arquivo env que vai para o repositório. Deve conter todas as variáveis com valores placeholder e comentários explicativos:

```dotenv
# Banco de dados
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp
DIRECT_DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp

# Auth
BETTER_AUTH_SECRET="gerar-com-openssl-rand-base64-32"
BETTER_AUTH_URL=http://localhost:3000
CORS_ORIGIN=http://localhost:3000

# Ambiente
APP_ENV=development
NODE_ENV=development

# Email (desenvolvimento local usa Mailpit na porta 1025)
USE_MAILPIT=true
SMTP_HOST=localhost
SMTP_PORT=1025
RESEND_API_KEY=""

# Stripe
STRIPE_SECRET_KEY="sk_test_placeholder"
STRIPE_WEBHOOK_SECRET="whsec_placeholder"

# Expo / Native
EXPO_PUBLIC_SERVER_URL=http://localhost:3000
EXPO_PUBLIC_APP_ENV=development
EXPO_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_placeholder"
```

---

## bts.jsonc

Gerado pelo BTS CLI. Rastreia as opções selecionadas no scaffold para que `bunx create-better-t-stack add` saiba o estado atual do projeto.

```jsonc
{
  // Created: 2026-03-06T10:00:00.000Z
  // Version: 3.22.1
  "frontend": ["next", "native-uniwind"],
  "backend": "self",
  "database": "sqlite",
  "orm": "prisma",
  "auth": true,
  "addons": ["biome", "lefthook", "mcp", "ruler", "skills", "turborepo"],
  "examples": true
}
```

Não editar manualmente. Seguro deletar se o comando `add` não for mais necessário.
