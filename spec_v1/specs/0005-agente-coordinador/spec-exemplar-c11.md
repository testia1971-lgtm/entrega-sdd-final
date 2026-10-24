> **Área responsable de este artefacto: requisitos y trazabilidad.** Es una *plantilla de formato* para
> `specs/000X-*/requirements.md`:
> forma del bloque `Req ID -> requisito EARS -> 1 escenario Given/When/Then -> contrato de datos -> trazabilidad`.
> Adaptar el vocabulario al glosario canonico (rol contractual base, tipos de asignacion, EN_RIESGO, piso de reserva)
> y al motor de persistencia que se confirme.
>
> **Vocabulario (regla del repo):** los *estados y entidades* usan el glosario canónico — semáforo
> `SALUDABLE | VIGILADO | EN_RIESGO | CRITICO`, propuesta `EN_REVISION`, invariantes `I-1..I-13` — y los
> *argumentos de las tools* van en `snake_case`. Un solo glosario, sin sinónimos (constitution Art. XI).
---
title: "Gestión de RH y Proyectos — Exemplar spec (C11 Agente Coordinador) in EARS + Gherkin"
tags: [sdd, ears, gherkin, spec-template, agent]
status: draft
created: 2026-10-20
---

**This file is a template, not just a spec.** The other 13 capabilities follow the same shape:
`Req ID → EARS requirement → ≥1 Given/When/Then scenario → data/contract → traceability`.
It is also the worked example of the format that `RESEARCH/SDD_TOOLING_2026.md` recommends.

---

# C11 — Agente Coordinador de Asignaciones

**Capability goal.** Detectar presión de capacidad, proponer refuerzos temporales y propuestas de equipo para
proyectos nuevos, y garantizar que toda asignación respeta las invariantes del dominio — sin que el modelo de
lenguaje decida nunca por sí solo una asignación.

**Owners.** Backend + IA. **Stakeholders.** Manager de fábrica, Scrum Masters, coordinadores de RH.

## C11.1 Alcance

| Dentro | Fuera |
|---|---|
| Detectar presión y déficit de capacidad | Decidir qué proyecto se sacrifica |
| Proponer refuerzos temporales y equipos para proyectos nuevos | Asignar sin aprobación humana (modo por defecto) |
| Detectar falta de cobertura por incapacidad/vacaciones | Autorizar incapacidades ni vacaciones |
| Rastrear el préstamo temporal y respetarlo | Modificar el calendario laboral oficial |
| Registrar alertas, propuestas y auditoría | Reasignar personas entre proyectos automáticamente |

## C11.2 Requisitos (EARS)

> Convención: `C11-R<n>` identificador estable; `C11-R<n>.<m>` escenario. Requisitos ubiquitousos:
> "THE SYSTEM SHALL …". Evento: "WHEN … THE SYSTEM SHALL …". Estado: "WHILE … THE SYSTEM SHALL …".
> No deseado: "IF … THEN THE SYSTEM SHALL …".

**C11-R1 (ubiquitous)** — THE SYSTEM SHALL compute the pressure index of a project over a given window using the
deterministic metrics engine, without involving a language model.

**C11-R2 (event-driven)** — THE SYSTEM SHALL classify every project with the unified metric of
`designs/0005-modelo-carga-y-presion.md`: `SALUDABLE` when `avance < 0.75` ∧ `indice_presion ≤ 1.00`, `VIGILADO`
when `avance ≥ 0.75` ∧ `indice_presion ≤ 1.00` (informa, **no** propone refuerzo), `EN_RIESGO` when
`indice_presion ∈ (1.00, 1.25]` (propone refuerzo) and `CRITICO` when `indice_presion > 1.25` (propone refuerzo y
escala a un humano). The 75 % window orders urgency; the deficit is what creates the proposal.

**C11-R3 (event-driven)** — WHEN a project is created, THE SYSTEM SHALL generate a staffing proposal containing a
candidate per required role, each with role, allocation percentage, period and rationale.

**C11-R4 (event-driven)** — WHEN an approved absence overlaps the assignments of a person, THE SYSTEM SHALL compute
the coverage of that absence and SHALL raise a `COVERAGE_GAP` alert when coverage < 1.0.

**C11-R5 (event-driven)** — WHEN the coordinator proposes an assignment, THE SYSTEM SHALL create a `Propuesta` with
status `EN_REVISION` and SHALL NOT create an `Assignment`. The only write path to `assignments` is
`commit_assignment`, which requires a recorded human approval (M1..M3 in `designs/0004` §4).

**C11-R6 (unwanted behaviour)** — IF the coordinator's output fails validation against invariants I-1..I-13, THEN THE
SYSTEM SHALL reject the output, SHALL retry at most twice with the validation errors as feedback, and SHALL
escalate to a human after the second failure.

**C11-R7 (ubiquitous)** — THE SYSTEM SHALL cite, in every proposal, the identifiers of the people, projects,
assignments and absences used as evidence, and SHALL cite every numeric figure from a metrics-engine result.

**C11-R8 (unwanted behaviour)** — IF a person is covered by an active temporary loan, THEN THE SYSTEM SHALL NOT
propose any assignment for that person outside the loan window, and SHALL NOT exceed the person's remaining
allocation at the source project.

**C11-R9 (unwanted behaviour)** — IF a person is on approved medical leave or vacation, THEN THE SYSTEM SHALL NOT
propose an assignment for that person during the absence.

**C11-R10 (unwanted behaviour)** — IF an absence has no recorded return date, THEN THE SYSTEM SHALL ask a
clarifying question and SHALL NOT assume a return date.

**C11-R11 (state-driven)** — WHILE the agent is disabled, THE SYSTEM SHALL NOT execute any coordinator run and
SHALL display a paused banner.

**C11-R12 (ubiquitous)** — THE SYSTEM SHALL record an `AuditLog` entry for every proposal, every validation
failure, every escalation and every human decision, including the actor, the timestamp, the evidence and the
outcome.

**C11-R13 (ubiquitous)** — THE SYSTEM SHALL expose the coordinator's autonomy level as configuration with values
`suggest`, `propose` and `apply_with_revert`, and SHALL default to `propose`.

**C11-R14 (event-driven)** — WHEN a person is created or their skills, base role or employment type change, THE
SYSTEM SHALL mark every cached availability snapshot for that person as stale.

## C11.3 Escenarios (Given / When / Then)

**C11-R1.1**
- **Given** a project "API Banking" with `start_date` 2026-01-05, `end_date` 2026-07-31, `backlog_hours` 2400 and
  `hours_consumed` 1900
- **And** three active assignments totalling 2.0 FTE over the remaining period
- **When** `compute_pressure(project_id, today)` is called
- **Then** the result contains `avance`, `indice_presion`, `deficit`, `severity` and `as_of`
- **And** every numeric value equals the value recomputed independently by the metrics engine

**C11-R2.1**
- **Given** a project whose elapsed progress is 0.82 and whose pressure index is 0.9
- **When** the nightly recompute runs
- **Then** the project state becomes `VIGILADO`
- **And** no staffing proposal is created

**C11-R2.2**
- **Given** a project whose elapsed progress is 0.55 and whose pressure index is 1.4
- **When** the nightly recompute runs
- **Then** the project state becomes `CRITICO`
- **And** a staffing proposal is created with at least one candidate
- **And** the proposal cites the deficit and the capacity that is missing
- **And** if the demand data of the project has low confidence, the coordinator asks for an updated estimate
  instead of proposing (no proposal from a guess)

**C11-R3.1**
- **Given** a newly created project requiring 1 Scrum Master, 2 Developers and 1 QA
- **And** the available pool contains 3 matching people with free capacity
- **When** the coordinator runs for that project
- **Then** exactly one proposal per required role is created
- **And** each proposed person has capacity for the proposed allocation percentage
- **And** each proposal states the skills that make the person a match

**C11-R4.1**
- **Given** the only QA of a project is on approved medical leave from 2026-06-01 to 2026-06-15
- **And** another QA with the required skill is free for that period
- **When** the absence is approved
- **Then** a `COVERAGE_GAP` alert is raised
- **And** the coordinator proposes the available QA as a temporary replacement
- **And** the proposal cites the absence and the covered assignments as evidence

**C11-R4.2**
- **Given** the only QA of a project is on approved medical leave
- **And** no other QA is available
- **When** the coordinator runs
- **Then** the coordinator escalates to a human instead of proposing an invalid substitute

**C11-R5.1**
- **Given** an autonomy level of `propose`
- **When** the coordinator decides a reinforcement is needed
- **Then** a `Propuesta` with status `EN_REVISION` exists
- **And** the collection `assignments` contains no new document

**C11-R6.1**
- **Given** a draft proposal that assigns a person who is already at 120% allocation
- **When** the proposal is validated
- **Then** validation returns the violated invariant `I-1` with the conflicting assignment
- **And** the proposal is not submitted
- **And** a second attempt is made with the violation as feedback

**C11-R7.1**
- **Given** any submitted proposal
- **When** the proposal is rendered
- **Then** it displays at least one project identifier, one person identifier, the pressure index and the evidence
  list
- **And** each displayed numeric figure matches the metrics-engine value for the same `as_of` timestamp

**C11-R8.1**
- **Given** "Persona 12" is on a temporary loan to project B until 2026-08-31 and remains 40% allocated to project A
- **When** the coordinator considers a reinforcement for project C
- **Then** "Persona 12" is either not proposed or proposed with an allocation that leaves project A at its
  remaining 40%
- **And** the proposal never extends past 2026-08-31

**C11-R9.1**
- **Given** "Persona 31" has approved vacation from 2026-07-01 to 2026-07-15
- **When** the coordinator searches candidates for any project
- **Then** "Persona 31" is never returned for a period overlapping the vacation

**C11-R10.1**
- **Given** an approved medical leave without a `return_date`
- **When** the coordinator evaluates the coverage
- **Then** exactly one clarifying question is created asking for the return date
- **And** no proposal is created until the question is answered
- **And** no return date is assumed by default

**C11-R11.1**
- **Given** the agent is disabled by configuration
- **When** the nightly scheduler fires
- **Then** no coordinator run is started
- **And** the UI shows the paused state

**C11-R12.1**
- **Given** a human approves a proposal
- **When** the decision is recorded
- **Then** an `AuditLog` entry exists with actor, timestamp, proposal id and outcome
- **And** the assignment it created links back to the proposal id

**C11-R13.1**
- **Given** the configuration is unset
- **When** the autonomy level is read
- **Then** it is `propose`

## C11.4 Contrato de datos (colecciones MongoDB)

```
agent_proposals            # entidad canonica: Propuesta (designs/0002 §2.9)
  _id, kind (NEW_PROJECT_STAFFING | REINFORCEMENT | REBALANCE | COVERAGE_GAP),
  project_id, payload[] { person_id, project_id, role_id, allocation_pct,
                          start_date, end_date, source_assignment_id, rationale },
  rationale, confidence, evidence[] { kind, id, as_of, value },
  version, hash_payload, version_aprobada,   # M1: la aprobacion ata un payload concreto
  status (EN_REVISION | ACEPTADA | RECHAZADA | EXPIRADA),
  validation { valid, violations[] { invariant, message, refs[] } },
  agent_run_id, decided_by, decided_at, created_at

alerts                     # entidad canonica: Alerta
  _id, project_id?, person_id?, kind (OVERALLOCATION | COVERAGE_GAP | LEAVE_UNCOVERED
        | SKILL_GAP | DEADLINE_RISK), severity (SALUDABLE | VIGILADO | EN_RIESGO | CRITICO),
  window_start, window_end, evidence[], status (OPEN | ACK | RESOLVED), created_at

audit_log                  # entidad canonica: BitacoraEvento (designs/0002 §2.10)
  _id, actor { type: HUMAN|AGENT, id }, action, entity, entity_id,
  before, after, spec_ref (C11-R5), agent_run_id, created_at
```

> La **severidad** de una alerta reutiliza el semáforo canónico (`SALUDABLE | VIGILADO | EN_RIESGO | CRITICO`): un
> solo vocabulario para el mismo hecho. Un enum nuevo (`WATCH | CRITICAL | SEVERE`) sería un cuarto término para lo
> que ya se llama de otra forma (constitution Art. XI: un único glosario, sin sinónimos).

## C11.5 Contrato de herramientas (OpenAI Agents SDK)

Todas devuelven JSON con `as_of`. Ninguna escribe en `assignments` salvo `commit_assignment`, que solo escribe tras
una aprobación humana registrada, **versionada al payload** y **revalidada contra el motor en el momento de
aplicar** (M1..M3).

| Tool | In | Out | Notas |
|---|---|---|---|
| `read_spec(capability)` | `capability` | texto de spec + versión | el agente lee su propio spec antes de actuar |
| `get_project` | `project_id` | proyecto + roles requeridos + fechas | |
| `get_person` | `person_id` | perfil + rol base + skills | **no** expone notas médicas |
| `compute_pressure` | `project_id, at` | `avance, indice_presion, deficit, severity, as_of` | motor determinista |
| `compute_utilization` | `person_id, from, to` | `util, overloaded, at_capacity, as_of` | |
| `find_candidates` | `role, tools[], from, to, pct` | candidatos con carga, skills, conflictos, marca de préstamo | ya filtrado por el motor |
| `list_absences` | `from, to` | lista de ausencias | tipo + fechas, sin notas médicas |
| `check_coverage` | `absence_id` | `demanda, oferta, cobertura, gap_roles` | |
| `list_available_pool` | `from, to, tools[]` | lista de disponibles | |
| `historial_persona` | `person_id` | timeline de la persona: asignaciones, ausencias, ascensos | sin datos clínicos |
| `list_loans` | `project_id?, person_id?` | préstamos vigentes con fecha de devolución | % cedido al proyecto destino |
| `validate_proposal` | `payload` | `valid, violations[]` | corre I-1..I-13 (motor determinista) |
| `create_proposal` | `kind, payload, rationale, evidence` | `proposal_id` | escribe `agent_proposals`, nunca `assignments` |
| `raise_alert` | `kind, severity, evidence` | `alert_id` | |
| `ask_human` | `question` | `question_id` | una pregunta a la vez |
| `escalate` | `reason` | `ticket_id` | |
| `notify_in_app` | `message, link` | confirma la publicación | publica en la bandeja de notificaciones de la app |
| `commit_assignment` | `proposal_id, version` | `assignment_id` | **`needs_approval=True`** → interrumpe la corrida y espera decisión humana. Aplica **solo la versión aprobada**, revalida I-1..I-13 en el momento del commit (M2) y es idempotente por `(proposal_id, version)` (M3) |

## C11.6 Requisitos no funcionales

| ID | Requisito | Métrica |
|---|---|---|
| C11-NFR1 | Toda propuesta se valida en < 200 ms (motor, sin LLM) | p95 medido en CI |
| C11-NFR2 | La corrida del coordinador completa < 30 s con ≤ 12 tool calls | medido con el tracing del SDK |
| C11-NFR3 | Cero escrituras en `assignments` originadas por el LLM sin `approval` registrado **y vinculado a la versión del payload aplicado** | test de invariante sobre la colección completa |
| C11-NFR4 | Ningún dato de historia clínica (nota, diagnóstico) sale del backend hacia el modelo | test que inspecciona el payload de cada tool call |
| C11-NFR5 | El 100% de las acciones del agente deja entrada en `audit_log` | test de reconciliación |
| C11-NFR6 | La API del tablero devuelve solo la ventana visible (`from`, `to`, `zoom`) | test de contrato + assertion de tamaño de respuesta |

## C11.7 Trazabilidad (plantilla)

| Requisito | Tarea | Prueba | Tipo |
|---|---|---|---|
| C11-R1 | T11.1 motor `compute_pressure` | `test_pressure_index` | unit |
| C11-R2 | T11.2 evaluador nocturno | `test_state_transitions` | unit |
| C11-R3 | T11.3 propuesta de equipo | `test_new_project_staffing` | golden |
| C11-R4 | T11.4 cobertura de ausencias | `test_coverage_gap` | golden |
| C11-R5 | T11.5 propuesta sin escritura | `test_no_assignment_written` | integración |
| C11-R6 | T11.6 reintento y escalamiento | `test_validation_retry` | unit |
| C11-R7 | T11.7 evidencia y cifras | `test_proposal_evidence` | integración |
| C11-R8 | T11.8 respeto del préstamo | `test_temporary_loan_respected` | golden |
| C11-R9 | T11.9 exclusión por ausencia | `test_absent_not_proposed` | golden |
| C11-R10 | T11.10 pregunta de clarificación | `test_unknown_return_date_asks` | golden |
| C11-R11 | T11.11 kill switch | `test_agent_disabled` | unit |
| C11-R12 | T11.12 auditoría | `test_audit_entries` | integración |
| C11-R13 | T11.13 configuración de autonomía | `test_default_autonomy` | unit |
| C11-R14 | T11.14 invalidación de caché | `test_snapshot_stale_on_change` | unit |
| C11-NFR1..6 | T11.15 observabilidad y tests de contrato | `test_nfr_*` | integración |
| C11-M1 (la aprobación va versionada) | T11.5 commit con `version` | `test_commit_rejects_stale_version` | integración |
| C11-M2 (revalidación en el commit) | T11.5 commit revalida I-1..I-13 | `test_commit_revalidates_invariants` | integración |
| C11-M3 (idempotencia del commit) | T11.5 commit idempotente | `test_commit_idempotent` | integración |

## C11.8 Preguntas abiertas (bloqueantes para cerrar la spec)

1. ¿La carga de trabajo del proyecto entra como `backlog_hours` manual, o se deriva de un sistema externo (Jira,
   Clockify)? Si es externo, ¿cuál? (cambia C11-R1 y el contrato de `get_project`).
2. ¿El "holiday table" es por país, por sede, o por contrato? Define el cálculo de días hábiles (C11-R4).
3. ¿La cobertura de una incapacidad se define por *rol* (cubrir el rol) o por *persona equivalente* (mismas skills)?
   Hoy la spec asume **rol**, con desempate por skill.
4. ¿Quién aprueba: el Manager de fábrica, el Scrum Master del proyecto, o ambos? (C11-R5 → el actor de `decided_by`).
5. ¿El umbral 75% es configurable por proyecto o global? (C11-R2 dice 0.75 global; la spec permite override).
6. ¿Qué pasa con las asignaciones en el pasado (histórico)? ¿Se corrigen o se congelan? Afecta la reconciliación.