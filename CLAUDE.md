# CLAUDE.md — Reglas globales del proyecto Soga

Este archivo es autoridad. Si algo en un prompt contradice CLAUDE.md, gana CLAUDE.md.

## 1. Stack fijo (no cambiar sin permiso explícito del usuario)
- **Runtime:** Node.js 20 LTS
- **Lenguaje:** TypeScript 5.4+ estricto (`strict: true`, `noUncheckedIndexedAccess: true`)
- **Backend:** NestJS 10 + Fastify adapter
- **Realtime:** Socket.io 4 (gateway NestJS nativo)
- **Queue:** BullMQ sobre Redis 7
- **ORM:** Prisma 5 sobre Postgres 16
- **Vector:** pgvector (extensión de la misma Postgres — NO Chroma, NO Pinecone)
- **Workflows multi-agente:** LangGraph.js (solo en bounded context `workflows`)
- **Auth:** Better-Auth
- **Frontend:** Next.js 14 App Router + React 18 + Tailwind + shadcn/ui + TipTap
- **Monorepo:** pnpm workspaces + Turborepo
- **Contenedores:** Docker Compose para dev (Postgres+pgvector, Redis, MinIO)
- **Tests:** Vitest (unit), Supertest (integración), Playwright (E2E)

## 2. Estructura del monorepo
soga/
├── apps/
│   ├── api/          # NestJS backend
│   └── web/          # Next.js frontend
├── packages/
│   ├── shared/       # Tipos, DTOs, contratos de eventos, schemas Zod
│   ├── db/           # Schema Prisma + migraciones + seeds
│   └── config/       # ESLint, TS, Tailwind compartidos
├── docker/           # Dockerfile dev, compose, scripts
├── docs/             # product-brief, ADRs, runbooks
├── CLAUDE.md
├── project.md
├── turbo.json
└── pnpm-workspace.yaml

## 3. Arquitectura hexagonal por bounded contexts
Dentro de `apps/api/src/` cada bounded context vive en su propia carpeta con esta
estructura EXACTA:
contexts/<context-name>/
├── domain/          # Entidades, value objects, eventos de dominio, repos (interfaces)
├── application/     # Use cases, ports, DTOs internos
├── infrastructure/  # Adapters: Prisma repos, HTTP clients, event publishers
└── presentation/    # Controllers, gateways Socket.io, DTOs HTTP (Zod)

**Contextos (los 9):** `identity`, `org`, `vault`, `knowledge`, `sync`, `agents`,
`workflows`, `governance`, `realtime`. No crear nuevos sin discutirlo.

## 4. Reglas duras (violarlas = rechazar el código)
1. **Nunca** lógica de negocio en controllers/gateways. Solo traducen HTTP/WS ↔ use case.
2. **Nunca** acceso a Prisma desde `domain/` o `application/`. Solo desde `infrastructure/`.
3. **Nunca** un bounded context importa de `domain/` o `application/` de otro. Se
   comunican solo por eventos del event bus o por ports públicos expuestos.
4. **Todo** cambio de estado relevante publica un evento de dominio al event bus
   (patrón outbox — tabla `outbox` en la misma transacción).
5. **Todo** endpoint y gateway valida input con Zod desde `@soga/shared`.
6. **Todo** use case tiene test unitario en `application/`. Todo aggregate con
   invariantes tiene test unitario en `domain/`.
7. **Multi-tenant:** toda query SQL filtra por `project_id` vía Row-Level Security
   en Postgres. El middleware de contexto inyecta `SET LOCAL app.current_project_id`.
8. **Nada** de `any` en TS salvo en adapters de librerías externas con comentario `// @soga-any: <razón>`.
9. **Nada** de lógica de negocio en migraciones Prisma. Migraciones solo estructura.
10. **Commits:** Conventional Commits. `feat(context): ...`, `fix(context): ...`,
    `chore: ...`, `test(context): ...`, `docs: ...`.

## 5. Cómo añadir un bounded context nuevo
1. Crear carpeta `contexts/<name>/` con las 4 subcarpetas.
2. Crear `<name>.module.ts` en la raíz del contexto, registrarlo en `AppModule`.
3. Definir entidades y eventos de dominio ANTES que los use cases.
4. Definir ports (interfaces) en `application/` ANTES que adapters.
5. Migración Prisma con prefijo de tablas del contexto (ej. `org_subgroup`, `vault_note`).
6. Tests en paralelo a cada capa.
7. Documentar el contexto en `docs/contexts/<name>.md` con diagrama de entidades.

## 6. Event bus y patrón outbox
- Implementación en `contexts/realtime/infrastructure/event-bus/`.
- Tabla `outbox` en la misma transacción que el cambio de estado.
- Worker BullMQ lee outbox y publica. Nadie publica eventos directamente.
- Todo evento tiene tipo en `@soga/shared/events/` con Zod schema versionado (`v1`).
- Auto-indexación del `knowledge` context se suscribe a eventos, no se llama directo.

## 7. Reglas de comunicación con el usuario (Iván)
- Iván ha usado terminales, API keys, webhooks — no es novato total, pero tampoco
  experto en arquitectura limpia. Explica decisiones arquitectónicas cuando tomes una
  no obvia.
- **Comandos destructivos** (`rm -rf`, `DROP`, `prisma migrate reset`, `docker volume rm`):
  explicar ANTES de ejecutar y pedir confirmación.
- **Comandos rutinarios** (`pnpm install`, `pnpm dev`, migraciones `up`, tests):
  ejecutar directamente sin pedir permiso.
- Si un error es recuperable, arréglalo tú y sigue. Si no lo es, para y explica.
- Al final de cada fase/prompt, imprimir un checklist con `✅` o `❌` por criterio
  de aceptación. Si hay algún `❌`, NO dar la fase por terminada.
- Responder en español (Iván escribe en español).

## 8. Lo que NUNCA hace Claude Code en este proyecto
- Instalar dependencias no listadas aquí sin justificar.
- Cambiar de ORM, DB, o framework.
- Usar `localStorage` sin discutirlo (hay Better-Auth + cookies httpOnly).
- Tocar `CLAUDE.md` o `project.md` sin que Iván lo pida explícitamente.
- Crear archivos fuera de la estructura del monorepo definida.
- Meter lógica cross-context por imports directos en vez de eventos.
