# Soga — Product Brief

## One-liner
Soga es la torre de control de un aeropuerto entero: donde los humanos están en
sync, los agentes comparten cerebro, y el conocimiento crece solo.

## Posicionamiento vs Paperclip
Paperclip es un cockpit para 1 piloto. Soga es infraestructura para grupos
humanos que trabajan *con* enjambres de agentes. Paperclip orquesta agentes de
una empresa; Soga orquesta **personas + sus agentes** como una organización viva.

## Los 5 pilares (donde encajan los 17 diferenciales)

### 1. Colaboración humana síncrona (#1, #9, #13, #14, #15)
Subgrupos 3–5 → departamentos → proyecto. Chat realtime, presencia, Facilitator
agent, briefs personalizados, pulses/focus/retros opcionales. Condiciones
graduales: las herramientas de sync están diseñadas para desactivarse cuando
el grupo ya no las necesita.

### 2. Cerebro compartido (#4, #5, #6, #16)
pgvector como memoria colectiva. Event bus auto-indexa todo lo que pasa (runs,
uploads, chat, notas, briefs). Cada agente hace RAG sobre el proyecto entero
antes de actuar. Mensajes directos entre agentes leídos al inicio de cada
heartbeat.

### 3. Orquestación multi-agente (#2, #3, #7, #12)
Los agentes pertenecen a personas, no al proyecto — cuando te vas, te los
llevas. 4 tipos: **Worker** (ejecuta), **Coordinator** (revisa departamento),
**Evaluator** (cruza departamentos), **Facilitator** (cuida cohesión humana).
Workflows LangGraph con condicionales y bucles de corrección.

### 4. Vault de conocimiento (#8, #10)
Vault de archivos `.md` compatible con Obsidian, con backlinks, grafo visual
y editor con `[[wikilinks]]`. Humanos suben fotos, PDFs, CSVs, resultados de
tests — los agentes los procesan. Sirve igual a científicos que a builders.

### 5. Gobernanza multinivel (#11, #17)
Dual org chart (humanos arriba, agentes abajo, líneas de ownership). Budgets
por agente → subgrupo → departamento → proyecto con drill-down. Approvals con
escalado automático entre niveles.

## Personas objetivo (MVP)
1. **Grupos de investigación** (4–12 personas) con datos de campo + análisis.
2. **Startups early-stage** donde cada founder trae sus propios agentes.
3. **Colectivos creativos** (medios, estudios) coordinando proyectos paralelos.

## Métricas de éxito del MVP (F1+F2)
- Un grupo de 5 personas levanta un proyecto en <10 min.
- Un agente Worker completa su primera tarea con contexto RAG sin prompt manual extra.
- Un mensaje enviado en chat aparece en <200ms en todos los clientes del subgrupo.
- El event bus indexa un upload PDF en el vault y lo hace buscable en <30s.

## Qué Soga NO es (para no perderse)
- No es un chatbot. No es un builder de workflows no-code.
- No es un framework de agentes — usa los tuyos (Claude Code, Codex, HTTP, bash).
- No es single-player. Si eres 1 persona, usa Paperclip.