# Soga — Visión técnica y roadmap

## Bounded contexts (los 9)

| Contexto      | Responsabilidad                                             | Fase |
|---------------|-------------------------------------------------------------|------|
| `identity`    | Humans, sesiones, invitaciones, perfiles compartibles       | F1   |
| `org`         | Projects, subgroups, departments, dual org chart            | F1   |
| `vault`       | Notas .md, backlinks, uploads, wikilinks, grafo             | F1   |
| `sync`        | Chat realtime, presencia, pulses, focus, retros, briefs     | F1→F3|
| `realtime`    | Gateway Socket.io, event bus, outbox, pub/sub interno       | F1   |
| `agents`      | Tipos de agente, ownership humano→agente, heartbeats, budgets básicos | F2   |
| `knowledge`   | Embeddings pgvector, auto-indexer, RAG API, mensajes inter-agente | F2   |
| `workflows`   | LangGraph, cadenas multi-agente, cross-analysis del Evaluator | F3 |
| `governance`  | Budgets multinivel, approvals con escalado, audit log       | F3   |

## Modelo de dominio — entidades clave

**identity:** `Human`, `Session`, `Invitation`, `SharedProfile`
**org:** `Project`, `Subgroup`, `Department`, `Membership`, `DualOrgChart`
**vault:** `VaultNote`, `Backlink`, `Upload`, `WikilinkRef`
**sync:** `ChatChannel`, `ChatMessage`, `Presence`, `Pulse`, `FocusSession`, `Retro`, `Brief`
**realtime:** `DomainEvent`, `OutboxRecord`, `SocketNamespace`
**agents:** `Agent`, `AgentKind` (worker|coordinator|evaluator|facilitator),
            `AgentOwnership`, `Heartbeat`, `Ticket`, `AgentMessage`, `Budget`
**knowledge:** `KnowledgeChunk`, `Embedding`, `IndexJob`, `RagQuery`
**workflows:** `Workflow`, `WorkflowRun`, `GraphNode`, `GraphEdge`, `CrossAnalysis`
**governance:** `BudgetTier`, `Approval`, `Escalation`, `AuditLogEntry`

### Relaciones clave
- `Human` *1—N* `AgentOwnership` *N—1* `Agent` → los agentes se van con su dueño.
- `Project` *1—N* `Department` *1—N* `Subgroup` *1—N* `Membership` *N—1* `Human`.
- `Agent` *N—1* `Subgroup` | `Department` | `Project` (nivel donde opera).
- `Ticket.workflowRunId?` → si viene de un workflow, está ligado.
- `KnowledgeChunk.sourceType` ∈ {note, upload, chat, run, brief, check-in}.

## Dependencias entre fases
F1 (núcleo humano) → F2 (agentes + cerebro) → F3 (workflows + gobernanza) → F4 (pulido)

Dentro de F1 el orden es: `realtime` (event bus y Socket.io base) antes que
`sync`, porque sync depende del gateway. `identity` y `org` pueden ir en paralelo
pero `org` consume eventos de `identity`.

## Roadmap

### Fase 1 — Núcleo colaborativo humano (semanas 1–4)
Multi-tenant, dual org chart, chat realtime, vault .md con backlinks, uploads.
**Sin agentes todavía.** Entregable: un grupo de 5 personas puede usar Soga como
Obsidian colaborativo con chat.

### Fase 2 — Agentes con cerebro compartido (semanas 5–9)
Los 4 tipos de agente, heartbeats BullMQ, adapters Claude Code/HTTP/bash,
pgvector + auto-indexer, RAG pre-heartbeat, mensajes inter-agente, budgets
básicos. Entregable: demoable, diferencial real frente a Paperclip.

### Fase 3 — Orquestación avanzada (semanas 10–14)
LangGraph workflows, Evaluator cross-departamento, governance multinivel con
escalado, Facilitator agent, briefs personalizados, pulses/focus/retros.

### Fase 4 — Pulido (semanas 15–18)
Grafo visual del vault tipo Obsidian, UI de condiciones graduales, dashboards
de cross-analysis, mobile, templates exportables de proyectos.

## Criterios globales de aceptación del MVP (F1+F2)
- [ ] 5 personas levantan un proyecto en <10 min (medido con cronómetro real).
- [ ] Chat realtime <200ms entre clientes del mismo subgrupo.
- [ ] Upload PDF → buscable en RAG en <30s.
- [ ] Un agente Worker completa un ticket usando RAG sin contexto manual extra.
- [ ] Los agentes de un humano desaparecen cuando el humano deja el proyecto.
- [ ] Tests: >80% cobertura en `domain/` y `application/` de todos los contextos F1+F2.
