---
title: "Propuesta final — Sistema de gestión de RH y asignaciones"
tags: [sdd, recursos-humanos, agentes-ia, propuesta-final]
status: active
created: 2026-10-21
deliverable_by: requisitos-y-trazabilidad
integrates: [dominio-y-agente, ux-y-frontend, arquitectura-y-plataforma]
---

# Propuesta de proyecto final — Sistema de Gestión de RH y Proyectos

**Sistema de gestión de recursos humanos y asignaciones de una fábrica de software, con agente coordinador IA.**
Metodología: **Spec Driven Development** (nivel *spec-anchored*, verificado en CI).

| | |
|---|---|
| **Entrega** | Requisitos y trazabilidad (requisitos, glosario, trazabilidad, ensamblado final) |
| **Aportan** | Dominio y agente (dominio, agente, carga/presión, dataset, UX, research con fuentes) · Arquitectura y plataforma (pendiente: D-07, D-08) |
| **Base consolidada** | [`../PROPUESTA.md`](../PROPUESTA.md) · [`../README.md`](../README.md) |
| **Detalle del proceso** | [`../specs/`](../specs) · [`../designs/`](../designs) · [`../tasks/`](../tasks) · [`../traceability/matrix.md`](../traceability/matrix.md) |

> Este documento es la **entrega final** que el cliente pidió: un solo MD con la opinión de todas las áreas de trabajo.
> Todas opinan; una sola entrega. Las decisiones del cliente mandan sobre las opiniones de las áreas de trabajo.

---

## 0. TL;DR para el evaluador

1. **Qué es**: una app web con dos tableros (proyectos y personas) y un agente IA que vigila la carga de los
   proyectos y **propone** refuerzos, préstamos temporales y coberturas. **Nunca aplica**: una persona aprueba.
2. **El corazón no es el dashboard**: es un **motor determinista** que sabe quién está libre, de vacaciones,
   prestado y a quién no se puede tocar. El LLM redacta y explica; el código decide.
3. **El 75 % no es una constante**: es un parámetro (`avanceVentanaPresion: 0.75`). La presión real se mide con
   `índice_presión = déficit ÷ capacidad_restante`. El 75 % es *cuándo mirar*, no *si hay problema*.
4. **Stack (decidido por el cliente)**: React + TS + Vite · FastAPI + Python · MongoDB · OpenAI Agents SDK.
5. **SDD de verdad**: 9 épicas · **36 historias** con criterio EARS · **22 reglas** de negocio · **13 invariantes**
   · **43 tareas** trazadas (T-001..T-606) · una constitution de 11 artículos · gates de CI que **pueden rechazar**.
6. **Riesgo #1**: el alcance, no la técnica. Hay un orden de sacrificio escrito (E8 → E7 → E3). El agente no se sacrifica.
7. **Pendiente real**: D-07 (autenticación) y D-08 (despliegue) siguen abiertas, pendientes de confirmación del
   cliente; se usan los defaults propuestos. La **deuda de alineación: 15 de 15 filas** del inventario (§16) están
   corregidas entre las distintas áreas y una auditoría independiente de trazabilidad. Diez filas eran vocabulario,
   referencias e IDs; dos tocaban contenido: el muro del agente quedó como *aprobación registrada* y ahora tiene
   **tres condiciones verificables** (aprobación versionada, revalidación en el commit, idempotencia), y la
   **plantilla** de spec usa el vocabulario canónico.

---

## 1. Qué vamos a construir

### 1.1 El problema

Una fábrica de software reparte personas entre decenas de proyectos con planillas dispersas. Cuatro fallos
concretos (no hipotéticos):

1. **No se sabe con quién cuenta cada proyecto** ni con qué rol — una persona puede tener varios roles por proyecto
   y un rol contractual base distinto de todos ellos.
2. **La presión se descubre tarde**: cuando el avance ya está al 80 % y el equipo va ahogado.
3. **Las ausencias se cubren a ojo**: o nadie cubre la incapacidad, o la cobertura se alarga sin fecha.
4. **El talento ocioso es invisible**: hay personas sin proyecto que nadie ubica.

El costo: sobrecarga de personas, proyectos que se atrasan porque nadie vio venir la presión, y decisiones de
asignación que **no se pueden explicar**.

### 1.2 La propuesta en una frase

> Una **fuente única de verdad** sobre quién está en qué proyecto, con qué rol, con qué % y hasta cuándo, y un
> **agente coordinador** que detecta la presión a tiempo y propone respuestas explicadas — siempre con una persona
> decidiendo.

### 1.3 Superficies del producto

| Superficie | Contenido | Historia |
|---|---|---|
| **Tablero de Proyectos** | Timeline con zoom (semana / quincena / mes / año), semáforo de presión, card de proyecto, organigrama del equipo | US-02.1…US-02.7 |
| **Tablero de Personal** | Listado con filtros, **pool de desasignados**, ficha 360 (antigüedad, skills, historial, vacaciones, cumpleaños), timeline individual | US-03.1…US-03.6 |
| **Coordinador IA** | Propuesta al crear un proyecto, monitoreo de presión, préstamo temporal, cobertura de ausencias, bandeja de aprobaciones, auditoría | US-04.1…US-04.8 |
| **Personas (RRHH)** | Alta, roles, ausencias, becarios, historial inmutable | US-01.1…US-01.6 |
| **Plataforma** | Trazabilidad, criterios ejecutables, control de acceso, auditoría inmutable, modo demo | US-05.1…US-05.7 |

### 1.4 Dentro y fuera de alcance

**Dentro**: los dos tableros, gestión de asignaciones con roles múltiples, ausencias con cobertura, préstamos
temporales con fecha de devolución, becarios con convenio, organigrama por proyecto, pool de desasignados,
agente coordinador con propuesta + aprobación humana, y el proceso SDD con trazabilidad a prueba.

**Fuera (explícito, y escrito para que no crezca)**: nómina y liquidaciones · ATS / contratación · evaluación de
desempeño · multi-empresa / multi-tenant · facturación al cliente · app móvil nativa · integración real con
Jira o Azure DevOps · ML predictivo de carga (el motor es heurístico y determinista **a propósito**).

---

## 2. Decisiones ya cerradas por el cliente

> Fuente: mensaje del cliente (2026-10-21): *"web unicamente · los datos reales los
> cargaré después, inicia con información dummy tuya · react+TS+Vite, fastapi, python + mongo; los agentes
> dentro de la solución, será con el estandar Open IA"*.

| ID | Decisión del cliente | Consecuencia de diseño |
|---|---|---|
| **D-01** | Solo web (responsive cubre móvil) | No hay app nativa; la regla "escritorio primero, alta densidad" manda |
| **D-02** | Datos dummy primero, reales después | Seed determinista + importador al mismo esquema con anonimización |
| **D-03** | React + TS + Vite | Un solo componente de timeline parametrizado da los dos tableros |
| **D-04** | FastAPI + Python + MongoDB | **Mongo no tiene `daterange` ni `EXCLUDE USING GIST`**: la segunda capa de los invariantes es el *reconciliador nocturno*, no la base |
| **D-05** | Agentes dentro de la solución, estándar OpenAI | **OpenAI Agents SDK (Python)** dentro del backend; el *tracing* del SDK es entregable |

---

## 3. La opinión de las áreas de trabajo

Todas opinan; una sola entrega. Este es el registro de **qué dijo cada área**, dónde hubo desacuerdo y cómo
quedó resuelto. No es un relato: son las posiciones, con su fundamento.

### 3.1 Dominio y diseño del agente — «el LLM explica, el código decide»

- **La métrica del 75 %**: *"el enunciado del cliente mezcla dos cosas distintas y el agente no debe confundirlas"*
  → `avance` (la ventana temporal) e `índice_presión = déficit ÷ capacidad_restante` (el hecho). *"Si el agente
  solo usara 'pasó 75 % → buscar gente', inventaría demanda en proyectos que van bien — el fallo clásico de
  alucinación de demanda."*
- **El gate va en la herramienta, no en el prompt**: con OpenAI Agents SDK, la tool con side-effect lleva
  `needs_approval=True`; la corrida se pausa con `interruptions`, la app guarda `result.to_state()`, el humano
  aprueba y **se reanuda la misma corrida**.
- **Cero tools que apliquen**: el diseño original establecía que ninguna tool activa, cancela o modifica una asignación. La reconciliación (A-11) deja el muro en un sitio más preciso y más fuerte: la única tool con efecto secundario es `commit_assignment`, con `needs_approval=True`, así que **ninguna tool escribe una asignación sin una aprobación humana registrada**.
  11 guardarraíles (G1…G11) y golden set ejecutable en CI.
- **Spec-anchored, no spec-first**: *"las specs quedan estáticas y se pudren"* → estructura propia
  `specs/<capacidad>/{spec,plan,tasks}.md` + EARS + `[NEEDS CLARIFICATION]` + deltas ADDED/MODIFIED/REMOVED +
  gate de CI. *"Un gate solo es control si puede rechazar."*
- **El timeline no se compra, se construye**: ninguna librería Gantt OSS da las tres cosas necesarias (filas con
  avatar, bandas de ausencia, resaltado de sobre-asignación); lo que las tiene es DHTMLX **en la versión de pago**
  → capa propia sobre `d3-scale` + `d3-zoom` (~1.500–2.500 LOC presupuestadas), parametrizada por origen de fila
  (`proyecto | persona`) → **un componente, dos tableros**.
- **Licencias verificadas una por una**: `fullcalendar/resource-timeline` premium (CC BY-NC-ND), `primereact` v11 ya
  no es MIT, `gantt-schedule-timeline-calendar` no es OSS, `elkjs` copyleft EPL/GPL, `gantt-task-react` archivado
  con 31k descargas/semana. *"Las descargas no son señal de salud."*
- **Riesgo #1 de un parcial**: el alcance, no la técnica → recorte explícito a 10 capacidades MVP + 5 en fase 2.

### 3.2 Requisitos y trazabilidad — «si la spec no es verificable, no existe»

- **Los requisitos salen de la narrativa, no de la intuición**: 9 épicas → **36 historias** con criterio EARS
  (ubiquitous / event-driven / unwanted-behaviour) → **22 reglas BR** → **13 invariantes I** → **43 tareas**
  → **matriz requisito → diseño → tarea → prueba** con 0 reglas huérfanas.
- **Un glosario, y manda**: *rol contractual base* ≠ *rol de proyecto*; *asignación* con tipo
  `PERMANENTE/TEMPORARIA/SUSTITUCION/BECARIO`; *piso de reserva*; *pool de desasignados* como **vista derivada**,
  no estado persistido (para que "disponible" y "asignada" no se desincronicen).
- **La demo es parte del entregable** (constitution Art. VIII): *seed → operación → resultado visible*. Sin mocks
  del agente.
- **Los datos sensibles no entran al scoring** (BR-20 / I-12): el scorer solo recibe skills, disponibilidad, rol,
  carga, historial y ausencias; y hay un test que verifica **las entradas del scorer**, no la intención.
- **El "75 %" también es un requisito, no un número mágico**: BR-04 lo define como *`avance ≥ umbral` **y**
  déficit ⇒ presión*, para que un proyecto al 80 % sin déficit **no** dispare una búsqueda de gente.
- **Un error de redacción, corregido por el cliente**: el stack por defecto era Fastify + PostgreSQL + Prisma. El
  cliente fijó FastAPI + MongoDB. No se pelea: el modelo de dominio y el diseño del agente son **agnósticos del
  motor**, y lo único que cambia es *dónde* vive la segunda capa de defensa de los invariantes.

### 3.3 Arquitectura y plataforma — autenticación y despliegue (decisión tomada con defaults)

D-07 y D-08 siguen abiertas, pendientes de confirmación del cliente. Se usan los defaults propuestos:
**D-07 (autenticación)** con JWT y roles (`COORDINACION`/`RRHH`/`CONSULTA`), y **D-08 (despliegue)** como
**aplicación web con backend locales, sin Docker**. Su opinión se incorpora como delta si llega.

### 3.4 Dónde hubo desacuerdo y cómo se resolvió

| # | Dominio y agente | Requisitos y trazabilidad | Resolución |
|---|---|---|---|
| 1 | 75 % = avance (ventana) + carga (banda) + cartera (alerta), **una métrica, tres niveles** | 75 % = avance **y** déficit ⇒ presión | **Las dos.** `designs/0005` + BR-04: la ventana ordena la urgencia, el déficit crea la propuesta. Sin contradicción: `VIGILADO` no propone |
| 2 | Stack FastAPI+Mongo (cerrado por el cliente) | Default Fastify+Postgres (preguntado antes de la respuesta del cliente) | **El cliente manda:** FastAPI+Mongo. El default previo queda como nota histórica, no como alternativa |
| 3 | Gate de aprobación por **interrupción del Agents SDK** (recomendado) | Gate en la UI + `EN_REVISION` (BR-18) | **A con B como respaldo**: la interrupción es lo que permite que el agente *reaccione* al rechazo dentro de la misma corrida; el endpoint de UI queda para aprobar desde fuera |
| 4 | Motor en TypeScript (redactado antes del cierre del stack) | Motor agnóstico | **Motor en Python** (mismo stack del backend). Es la única redacción pendiente que *cambia código*, no diseño (§16) |

---

## 4. Modelo de dominio y glosario canónico

12 entidades, 13 invariantes, semántica temporal estricta. Detalle: [`../designs/0002-modelo-datos.md`](../designs/0002-modelo-datos.md).

### 4.1 Glosario (extracto normativo — manda sobre cualquier otro documento)

| Término | Significado operativo | **No** es |
|---|---|---|
| **Persona** | Ser humano con relación laboral `ACTIVA` (empleado, becario o contratista) | Un recurso abstracto; no "tiene" proyectos |
| **Rol contractual base** | Rol con el que fue contratada. Uno solo, de por vida. Define banda y seniority | Un rol por proyecto |
| **Rol de proyecto** | Rol que ejerce dentro de un proyecto. Varios por persona, incluso en el mismo proyecto | El rol contractual base |
| **Asignación** | Vínculo temporal `[desde, hasta)` persona ↔ proyecto ↔ rol, con un % de su capacidad | "Estar en" un proyecto sin fechas |
| **Tipo de asignación** | `PERMANENTE` · `TEMPORARIA` (préstamo) · `SUSTITUCION` (cobertura) · `BECARIO` | — |
| **Capacidad nominal** | Tiempo disponible de la persona (horas/día × días laborables × factor del rol) | Un porcentaje abstracto |
| **Utilización** | `Σ % de asignaciones activas ÷ 100` | La carga del proyecto |
| **Carga estimada** | Demanda registrada por rol y periodo (`CargaEstimada`) | Lo mismo que "avance" |
| **Índice de presión** | `déficit ÷ capacidad_restante` | Un porcentaje de avance |
| **Ausencia** | `VACACIONES` · `INCAPACIDAD` · `PERMISO` · `LICENCIA` · `BAJA_MEDICA` | Una asignación |
| **Cobertura** | Asignación `SUSTITUCION` que compensa una ausencia y **termina a más tardar al regreso** | Un préstamo indefinido |
| **Préstamo temporal** | Asignación `TEMPORARIA` con `fechaDevolucionPrevista` obligatoria y proyecto origen | Una asignación indefinida |
| **Piso de reserva** | % mínimo de su asignación original que el proyecto origen conserva durante un préstamo (40 % por defecto) | Un peso dentro de un score |
| **Pool de desasignados** | **Vista** de personas `ACTIVA` sin asignación activa | Un estado persistido |
| **Propuesta del agente** | Documento con sugerencia + justificación + alternativas descartadas + impacto, en `EN_REVISION` | Una asignación activa |
| **Convenio becario** | Contrato de aprendizaje con mentor y periodo acotado | Una asignación más |

### 4.2 Semántica temporal (la regla que más bugs genera)

- Todo periodo es **`[desde, hasta)`**: una asignación que termina el 30/06 y otra que empieza el 30/06 **no se
  solapan**.
- Un cambio de % en una asignación viva **no reescribe la historia**: se cierra una y se abre otra; el histórico
  es inmutable y auditable (`PersonaHistorial`, append-only).
- Sin *soft delete* en `Asignacion`, `Ausencia` ni `Propuesta`: `estado` + `BitacoraEvento` bastan.

### 4.3 Los 13 invariantes (lo que el sistema no puede violar)

| # | Invariante | Verificación | Regla |
|---|---|---|---|
| I-1 | `Σ %` de una persona en un periodo ≤ 100 % | motor + test | BR-01 |
| I-2 | Un préstamo tiene `fechaDevolucionPrevista` obligatoria y ≤ fin del proyecto origen | esquema + regla | BR-07 |
| I-3 | Mientras el préstamo está activo, la asignación origen queda cedida en el % prestado | motor | BR-09/10 |
| I-4 | `origen + Σ préstamos activos ≤ 100 %` | derivado de I-1 + I-3 | BR-01 |
| I-5 | Una cobertura termina **a más tardar** el regreso de la ausencia | regla cruzada | BR-13 |
| I-6 | No hay dos asignaciones activas con el mismo `(persona, proyecto, rol)` | índice único | BR-02 |
| I-7 | Un becario no se asigna fuera de su convenio y nunca es candidato a préstamo sin RRHH | rango + filtro del motor | BR-11/16 |
| I-8 | Una asignación activa no puede referenciar un proyecto `CERRADO` | motor | — |
| I-9 | Un proyecto `ACTIVO` no puede quedar sin `Tech Lead` más de N días | alerta (no bloquea) | — |
| I-10 | Toda mutación relevante deja registro en `BitacoraEvento` (actor, antes/después, `spec_ref`) | integración + test | BR-19 |
| I-11 | Un préstamo no deja al origen por debajo del piso de reserva | **filtro duro** del motor | BR-08/12 |
| I-12 | El motor de scoring **no** recibe atributos sensibles | test sobre las entradas del scorer | BR-20 |
| I-13 | Al iniciar una incapacidad, sus asignaciones pasan a `SUSPENDIDA` y la cobertura a `ACTIVA` | transición | BR-15 |

> **Consecuencia de D-04 (MongoDB)**: no hay `CHECK`, ni tipos de rango, ni `EXCLUDE USING GIST`. La regla dura
> vive en **tres puntos de enforcement**, y los tres deben poder *rechazar*:
> **E1** servicio de dominio (valida I-1..I-13 antes del `insert`) · **E2** guardarraíl de tool (ninguna propuesta
> del LLM llega a E1 sin revalidarse) · **E3** reconciliador nocturno (escanea las colecciones y abre alerta).
> Lo que Mongo **sí** impone por su cuenta: índice único parcial (`unique` + `partialFilterExpression` sobre
> `estado: "ACTIVA"` → I-6), índices compuestos `{personaId, fechaDesde, fechaHasta}` y `{proyectoId, …}`, clave
> única de `idempotencyKey` (G9) y `$jsonSchema`. Detalle: `designs/0002-modelo-datos.md` §5.

---

## 5. El agente coordinador

Detalle: [`../designs/0004-agente-coordinador.md`](../designs/0004-agente-coordinador.md) · plantilla de spec:
[`../specs/0005-agente-coordinador/spec-exemplar-c11.md`](../specs/0005-agente-coordinador/spec-exemplar-c11.md).

### 5.1 Tres capas — la decisión load-bearing

```
┌──────────────────────────────────────────────────────────────────┐
│ CAPA 1 — Motor determinista (Python puro, SIN LLM)              │
│  utilización · presión · cobertura · conflictos · disponibilidad │
│  valida I-1..I-13 · emite "hechos" con marca de tiempo `as_of`   │
└───────────────────────▲──────────────────────────────────────────┘
                        │ hechos (JSON) + ids de evidencia
┌───────────────────────┴──────────────────────────────────────────┐
│ CAPA 2 — Agente (LLM + tool calling, OpenAI Agents SDK)          │
│  formula propuestas · redacta · explica · pregunta                │
└───────────────────────▲──────────────────────────────────────────┘
                        │ propuesta EN_REVISION
┌───────────────────────┴──────────────────────────────────────────┐
│ CAPA 3 — Persona: aprueba / rechaza / edita → aplica + audita    │
└──────────────────────────────────────────────────────────────────┘
```

**Corolario: el LLM nunca emite un número.** Toda cifra de una propuesta sale de una tool y se re-valida contra
el motor antes de mostrarse. Eso mata en la raíz la "disponibilidad alucinada" — el modo de fallo más caro de un
agente de RRHH.

**Evidencia que la sostiene**: *"Any mistakes in the solution come from errors in the formulation process"* (MIT,
2025) — el LLM formula, no calcula; asignar turnos es un espacio de ~10^63 soluciones (Timefold), así que la
búsqueda es **código**, no prompt.

### 5.2 Qué hace y cuándo

| Disparador | Acción del agente |
|---|---|
| `proyecto.creado` | **Propuesta de asignación inicial** (US-04.1) — es el pedido literal del cliente |
| `avance.actualizado` | Recalcula presión; si entra en `EN_RIESGO`, propone refuerzo |
| `ausencia.aprobada` | Propone **cobertura** con plazo acotado a la ausencia |
| `prestamo.por_vencer` | Recuerda la devolución o propone extender (**nunca** auto-extiende) |
| `sweep.diario` / `sweep.semanal` | Cartera en presión, cumpleaños, devoluciones, ausencias sin cubrir |
| `consulta.usuario` | Preguntas de estado en la app |

Cada disparador es **idempotente** (`idempotencyKey = trigger + entidad + versión`).

### 5.3 Herramientas y el muro

- **Lectura (11)**: `read_spec`, `get_project`, `get_person`, `compute_pressure`, `compute_utilization`,
  `find_candidates`, `list_absences`, `check_coverage`, `list_available_pool`, `historial_persona`, `list_loans`.
- **Propuesta (6)**: `validate_proposal`, `create_proposal`, `raise_alert`, `ask_human`, `escalate`,
  `notify_in_app`.
- **Efecto secundario (1)**: `commit_assignment(proposal_id, version)` — la **única** tool que escribe en el dominio, y
  solo con `needs_approval=True`: la corrida se pausa y el humano decide.
- **El muro es la aprobación, no la ausencia de tools.** Ninguna tool escribe una asignación, un préstamo o una
  cobertura **sin una aprobación humana registrada**; la decisión queda en el estado de la corrida y en la
  bitácora (constitution Art. V, BR-18). Aprobar también puede ocurrir en la UI
  (`POST /propuestas/:id/aprobar`). Catálogo canónico: **18 tools** (`designs/0004` §4 = `spec-exemplar-c11` C11.5).
- **Y el muro tiene tres condiciones, o es nominal** (`designs/0004` §4): **M1** la aprobación va **versionada** al
  payload (se aprueba *un* contenido, no "la propuesta en general"); **M2** al aplicar, el motor **revalida** los
  invariantes en ese momento (entre la propuesta y el clic cambió el mundo); **M3** el commit es **idempotente** por
  `(proposal_id, version)`. Cada una tiene su test: `test_commit_rejects_stale_version`,
  `test_commit_revalidates_invariants`, `test_commit_idempotent`.

### 5.4 Scoring determinista (pesos en configuración, nunca en el prompt)

```
score = 0.35 × cobertura_disponible        # días que puede aportar
      + 0.25 × afinidad_de_rol              # rol base == rol pedido ? 1.0 : 0.4
      + 0.15 × coincidencia_de_herramientas # Jaccard(persona, requeridas)
      + 0.10 × estabilidad                  # penaliza origen EN_RIESGO/CRITICO
      + 0.10 × no_duplicidad_de_prestamo     # penaliza repetir en el mismo destino
      + 0.05 × seniority_en_destino         # desempate suave, nunca sensible (I-12)
```

Selección tipo **knapsack 0/1 sobre fracciones**, con top-K (K = 10) candidatos por llamada para que la propuesta
sea legible. El **piso de reserva del 40 %** es un **filtro duro**, no un peso: si el préstamo dejaría al origen
por debajo, el candidato **no existe**.

### 5.5 Autonomía: `propose` (fijada)

| Nivel | El agente puede | Riesgo | Estado |
|---|---|---|---|
| `suggest` | reportar | bajo | superado |
| **`propose`** | crear `Propuesta` + `Alerta`; **nunca** escribe una asignación | bajo | **FIJADO (default)** |
| `apply_with_revert` | crear asignaciones `ACTIVA` con expiración y deshacer | medio-alto | documentado y **desactivado** |

Reflejado en constitution Art. V y X (interruptor global de pausa, visible en la UI y en el canal) y en BR-18.
**En la demo nunca se usa `apply_with_revert`** — la constitution lo prohíbe también en modo demo.

### 5.6 Los 11 guardarraíles

G1 el LLM no calcula · G2 el agente no escribe asignaciones · G3 las reglas duras viven en el motor ·
G4 toda propuesta cita evidencia · G5 todo hecho lleva `as_of` · G6 auto-verificación con máximo 2 reintentos y
escalada · G7 **preguntar, no suponer** (dato faltante → pregunta, nunca inventa) · G8 respeta los préstamos
vigentes · G9 idempotencia · G10 apagado global · G11 el agente no lee notas médicas (solo tipo y fechas).

---

## 6. El 75 %: una métrica, tres niveles

El enunciado del cliente — *"casi siempre al 75 % de los proyectos empieza la presión"* — admite tres lecturas.
Implementadas como reglas sueltas, el agente dispararía tres alertas contradictorias sobre el mismo proyecto.

```
avance(p, t)            = (t − inicio) / (fin_prevista − inicio)          # 0..1
capacidad_restante(p,t) = Σ_a a.fte × días_laborables(a ∩ [t, fin_prevista])
carga_restante(p, t)    = max(0, demanda_horas − horas_consumidas)
deficit(p, t)           = max(0, carga_restante − capacidad_restante)
indice_presion(p, t)    = deficit ÷ max(capacidad_restante, 1)
ventana_75(p, t)        = (avance ≥ 0.75)
```

| Condición | Estado | Qué hace el agente |
|---|---|---|
| `avance < 0.75` ∧ `índice ≤ 1.00` | `SALUDABLE` | Nada |
| `avance ≥ 0.75` ∧ `índice ≤ 1.00` | `VIGILADO` | **Reporta, no propone**: el problema es de mezcla de skills, no de falta de gente |
| `índice ∈ (1.00, 1.25]` | `EN_RIESGO` | **Propone refuerzo** (asignación, préstamo o cobertura) con impacto cuantificado |
| `índice > 1.25` | `CRITICO` | Propone refuerzo **y escala a humano** |

- **L1 (avance)** → la **ventana de presión**: define *cuándo* se mira.
- **L2 (carga)** → las **bandas** del tablero (`verde < 70 % ≤ ámbar < 85 % ≤ rojo`): define *cuánta* falta.
- **L3 (cartera)** → **alerta de dirección**: `% de proyectos activos en EN_RIESGO/CRITICO ≥ 75 %`. No genera
  propuestas por proyecto: genera un informe de cartera.

> **El 75 % es un parámetro de configuración, no una constante del código.** Si el cliente confirma una sola
> lectura, se cambia un parámetro; el diseño no se toca.
> `umbrales: { avanceVentanaPresion: 0.75, cargaBanda: {verde: 0.70, ambar: 0.85}, indicePresion: {riesgo: 1.00,
> critico: 1.25}, presionCartera: 0.75, pisoReservaPrestamo: 0.40 }`

**Verificación (T-402)**: *un proyecto al 78 % con déficit se marca; uno al 80 % sin déficit, no.*

---

## 7. Los dos tableros (12 decisiones cerradas)

Detalle y mockups: [`../designs/0003-tableros-ux.md`](../designs/0003-tableros-ux.md) (UX y frontend). Cada decisión
tiene criterio de verificación y está mapeada a US → test.

| | Decisión |
|---|---|
| X-01 | 4 anclas de zoom (semana/quincena/mes/año) + arrastre libre con imantado |
| X-02 | Fila **por persona**, agrupable por rol; detalle diario en hover |
| X-03 | **Un canal visual por magnitud**: color = salud · opacidad = FTE · trama = préstamo · rayado = sobre-asignación |
| X-04 | Semáforo de **4 estados**, con el umbral visible al lado del número |
| X-05 | Card de proyecto en **drawer lateral** (no modal): no se pierde el contexto |
| X-06 | Ficha 360 = **una vista larga con anclas**, no pestañas dispersas ("toda la historia en la empresa") |
| X-07 | Skills: **radar** (persona vs rol) + **heatmap** (persona × herramienta) |
| X-08 | Organigrama = **DAG**, no árbol (multi-rol = multi-arista); vacantes, becarios y mentores con estilo propio |
| X-09 | Vacaciones: barra de saldo + heatmap de año + **autogestión (solicitar, cobertura sugerida, Planificador)**; cumpleaños: panel del **equipo**; **aniversario: banner + badge + logro** |
| X-10 | Préstamo visible: trama azul + borde doble + **sombra en el proyecto origen** + contador de días restantes |
| X-11 | El semáforo **nunca es solo color**: icono + patrón + texto; navegable sin ratón |
| X-12 | Escritorio primero, alta densidad (40–60 personas por pantalla), claro/oscuro |
| **X-13** | **Gamificación ligera**: logros de antigüedad (1/3/5/10/15/20 años), Planificador (≥30 días), Primeras vacaciones, Año sin ausencias; **no** compite con scoring del agente (BR-21) |
| X-10 | Préstamo visible: trama azul + borde doble + **sombra en el proyecto origen** + contador de días restantes |
| X-11 | El semáforo **nunca es solo color**: icono + patrón + texto; navegable sin ratón |
| X-12 | Escritorio primero, alta densidad (40–60 personas por pantalla), claro/oscuro |

**La decisión que más ahorra trabajo**: un solo motor de timeline parametrizado por `rowSource`
(`proyecto | persona`) → el tablero de proyectos, el de personal y la ficha 360 son **el mismo componente**, y el
zoom compartido (US-03.6) sale gratis.

**Rendimiento** (regla dura): la API del tablero acepta `from`/`to`/`zoom` y devuelve **solo lo visible**; el front
nunca descarga 3 años de asignaciones para pintar un trimestre; en zoom año, agregación server-side.

---

## 8. Requisitos: de la narrativa a la prueba

| Artefacto | Cantidad | Dónde |
|---|---|---|
| Épicas | **9** (`EP-01`…`EP-09`) | [`../specs/`](../specs) |
| Historias de usuario con criterio EARS | **36** (36/36 con criterio, 36/36 con tarea, **0 sin cobertura**) | `specs/000{2..6}-*/requirements.md` |
| Reglas de negocio | **22** (`BR-01`…`BR-22`), ninguna huérfana | pie de cada spec + matriz |
| Invariantes de dominio | **13** (`I-1`…`I-13`) | `designs/0002-modelo-datos.md` §5 |
| Tareas | **43** (`T-001`…`T-606` + T-106, T-107, T-306) en 7 sprints, cada una con US, criterio y test esperado | `../tasks/0001-tareas.md` |
| Decisiones de UX | **13** (`X-01`…`X-13`) | `designs/0003-tableros-ux.md` |
| Escenarios de seed | **23** (`S-01`…`S-23`) | `docs/DATASET_SEED.md` |
| Casos del golden set | **12** (`E-01`…`E-12`) | `designs/0004-agente-coordinador.md` §9 |
| Guardarraíles del agente | 11 (`G1`…`G11`) | `designs/0004-agente-coordinador.md` §6 |
| Artículos de la constitution | 11 | [`../constitution.md`](../constitution.md) |

### 8.1 Forma de un requisito (EARS)

```markdown
## US-04.3 — Propuesta de préstamo temporal
*When* un proyecto está en riesgo y su capacidad interna libre no cubre la necesidad,
*the system shall* buscar personas disponibles en **otros** proyectos que cumplan todas las
condiciones: skill suficiente, disponibilidad real en la ventana, ausencia del piso de reserva
del proyecto origen, no ser becario (o tener aprobación de RRHH), y no tener una asignación
fija que entra en conflicto.
```

Tres cosas que esta forma garantiza y que la narrativa no daba: el actor, el evento y el resultado **verificable**
por un test. Un criterio sin test es un criterio no implementado.

### 8.2 Trazabilidad

[`../traceability/matrix.md`](../traceability/matrix.md) mapea `US → BR → I → diseño → tarea → prueba → estado`, y
declara sus propios gaps (I-8/I-9 sin US propio; BR-14 sin EARS; préstamos entre clientes distintos a la espera de
decisión). El gate `scripts.traceability_check` falla si: un US no tiene EARS, un US no tiene tarea, una tarea no referencia
un US, una BR no aparece en el pie de ninguna spec, un I no aparece en la matriz, o algo marcado `VERIFICADO` no
tiene test.

---

## 9. Spec Driven Development aplicado

### 9.1 Topología del repo

```
constitution.md                      # 11 artículos inmutables
PROPUESTA.md                         # base consolidada del dominio
docs/PROPUESTA_FINAL.md              # ← este documento (entrega final)
specs/0001-vision/                   # visión, glosario, research con fuentes
specs/000{2..6}-*/requirements.md    # QUÉ: US-* con EARS + BR aplicables
designs/000{1..5}-*.md               # CÓMO: arquitectura, datos, UX, agente, presión
tasks/0001-tareas.md                 # QUIÉN: tareas con US y criterio
traceability/matrix.md               # requisito → diseño → tarea → prueba
docs/DATASET_SEED.md                 # dataset determinista y sus escenarios
docs/RIESGOS_Y_VALIDACION.md         # riesgos, supuestos, plan de validación
```

### 9.2 Ciclo

`constitution → specify → plan → tasks → implement → verify → (refactor: corrige la spec)`

Regla de oro del repo (`AGENTS.md` §1): **no se escribe código sin un requisito `US-*` y un criterio EARS que lo
verifique**; **un PR = una tarea**; y si el código revela que una spec es ambigua o falsa, **se corrige la spec en
el mismo PR**.

### 9.3 Herramientas y evidencia

| Decisión | Elección | Por qué |
|---|---|---|
| Sintaxis de requisitos | **EARS** (Kiro) | La formalización disponible más probada; ya es la que usan nuestras specs |
| Marcador de ambigüedad | **`[NEEDS CLARIFICATION: …]`** (spec-kit) | Obliga al agente a preguntar en vez de suponer (guardarraíl G7) |
| Plantillas + constitution | spec-kit, sobre la topología propia `specs/ designs/ tasks/` | Nomenclatura conocida + freedom de estructura |
| Nivel de SDD | **spec-anchored** | *Spec-as-source* (Tessl) sigue en beta y no permite edición humana del código |
| Puertas de CI | `scripts.spec_validate`, `scripts.traceability_check`, `scripts.verify_seed` (invariantes y cobertura S-01..S-23) y el golden set en `pytest` | *"Un gate solo es control si puede rechazar"* |
| Alternativa sin dependencias | La misma estructura a mano + validador propio | Si falla la instalación, el proceso no se cae |

Comparativa completa (Spec Kit, Kiro, OpenSpec, BMAD, GSD, cc-sdd) con fuentes:
[`../specs/0001-vision/research-tooling-2026.md`](../specs/0001-vision/research-tooling-2026.md).

### 9.4 La constitution (11 artículos)

Spec como fuente de verdad · verificabilidad (todo requisito con ID estable y prueba) · **el dominio manda** (el
glosario es la ley) · invariantes en el dominio, no en la UI · **el agente propone y la persona decide** ·
determinismo y auditabilidad (snapshot de entradas en cada propuesta) · privacidad por diseño · **la demo es parte
del entregable** · prohibición de atributos sensibles en el scoring · autonomía graduada con apagado de emergencia ·
un único glosario, sin sinónimos.

### 9.5 Las cinco fallas documentadas de SDD (y qué hacemos con cada una)

| Falla documentada | Mitigación en esta propuesta |
|---|---|
| Sobrecarga de revisión (el costo real es revisar) | 1 spec por épica; los anexos se enlazan, no se duplican; el entregable es el diff, no el MD |
| Falsa sensación de control | Un criterio sin test no está implementado; la matriz exige test para `VERIFICADO` |
| Confusión spec funcional ↔ técnica | Separación explícita `specs/` (QUÉ) / `designs/` (CÓMO) / `tasks/` (QUIÉN) |
| Workflows que no escalan | Reglas de glosario + IDs estables + validador de trazabilidad |
| Agentes que ignoran el research | El research es parte del repo y las decisiones citan la fuente |

---

## 10. Stack y arquitectura (definitiva)

| Capa | Elección | Licencia / nota |
|---|---|---|
| Frontend | React 19 + TypeScript + Vite · TanStack Query · Tailwind + shadcn/ui | MIT |
| **Timeline (zoom)** | **Capa propia sobre `d3-scale` + `d3-zoom` + `d3-time`** | ISC · ~1.500–2.500 LOC presupuestadas · un componente → dos tableros |
| Organigrama | `@xyflow/react` + `@dagrejs/dagre` | MIT / MIT · grafo, no árbol |
| Gráficos | Apache ECharts (radar, gauge, heatmap, stacked bar) | Apache-2.0 |
| Realtime | **SSE** — un solo stream multiplexado por pestaña | ojo: 6 conexiones SSE por origen en HTTP/1.1; `Last-Event-ID` para reanudar |
| Backend | **FastAPI + Python** · Pydantic v2 | valida también los argumentos de las tools |
| Datos | **MongoDB** | índices compuestos `{personaId, desde, hasta}` / `{proyectoId, estado}` |
| Motor | Paquete Python puro, **sin LLM**, funciones puras (`carga`, `capacidad`, `indice_presion`, `candidatos`) | testeable y determinista por construcción |
| Agente | **OpenAI Agents SDK** dentro del backend · `output_type` pydantic · `needs_approval=True` en la tool de side-effect | el *tracing* del SDK es entregable |
| Jobs | Worker programado + **reconciliador nocturno** de I-1..I-13 | idempotencia por `jobRunId` |
| Tests | pytest + Vitest + Playwright | los E2E son la evidencia de la demo |
| Despliegue | **Aplicación web con backend locales, sin Docker** | la demo se sirve como aplicación web; el backend (FastAPI + MongoDB) corre en local |

### 10.1 No adoptar (verificado)

`fullcalendar/resource-timeline` (premium, CC BY-NC-ND — blocker para una app de empresa) · `primereact` v11 (ya no
es MIT) · `gantt-schedule-timeline-calendar` (no es OSS) · `elkjs` (copyleft EPL/GPL, 423 KB) · `gantt-task-react`
(repo archivado, 31k descargas/semana) · `dagre@0.8.5` (usar `@dagrejs/dagre`).

> **Riesgo de licencia declarado**: el proyecto solo usa MIT/Apache/ISC. Si en una demo apareciera una pieza con
> licencia distinta, hay que decirlo en el documento de entrega.

### 10.2 El flujo de una propuesta (camino "estándar OpenAI")

```
agente → tool con side-effect (needs_approval=True)
       → la corrida se PAUSA: interruptions + state
       → la app serializa result.to_state() y muestra la propuesta editable en la bandeja
       → el humano aprueba  →  state.approve(interruption) + Runner.run(agent, state)
       → el agente continúa en la MISMA corrida (si fue rechazado, propone alternativa)
```

Respaldo: si el humano aprueba desde otra pantalla o sesión de la app, el endpoint aplica la propuesta y marca la
corrida como cancelada. En ambos casos: **el modelo nunca escribe una asignación por su cuenta** y toda acción queda
auditada.

---

## 11. Datos: dataset semilla determinista

Detalle: [`../docs/DATASET_SEED.md`](../docs/DATASET_SEED.md).

| Parámetro | Valor |
|---|---|
| Personas | **120** (108 empleados + 12 becarios) |
| Proyectos | **19** (12 `ACTIVO`, 3 `EN_PAUSA`, 3 `CERRADO` con histórico) |
| Histórico | **36 meses** de asignaciones, ausencias y saldos |
| Semilla | `20260101`, PRNG de la stdlib (`random.Random(semilla)`): sin aleatoriedad global ni `Math.random()` |
| Feriados | 24 |
| Proyectos creados en vivo | 2 (para disparar `proyecto.creado` en la demo) |

**23 escenarios sembrados a propósito**, cada uno para que una épica o un caso del golden set tenga material real:
proyecto al 92 % de carga · proyecto al 78 % (ventana de presión) · 11 proyectos en presión (≈75 % de la cartera) ·
3 devs al 100 % · dev senior con skills exactas libre al 40 % · préstamo que vence en 30 días · persona prestada en
proyecto crítico · incapacidad de 20 días sin cubrir · 9 personas de vacaciones · pool de desasignados · becario en
la última semana de convenio · cumpleaños del mes · saldos de vacaciones consumidos · proyecto sin Tech Lead · dos
candidatas con skill set idéntico · curvas de demanda con cola real · proyecto adelantado en calendario pero no en
carga · consumo real menor que el estimado · **aniversario con badge ANIVERSARIO_1** · **aniversario con badge ANIVERSARIO_5** · **solicitud de vacaciones con cobertura sugerida** · **solicitud de vacaciones sin cobertura (Planificador badge)**.

**Reglas del generador**: determinismo total con la misma semilla · **nunca genera datos que violen un invariante**
(el seed se autovalida: un seed que viola una regla es un bug del seed) · cobertura garantizada de S-01..S-23 ·
rangos de fechas relativos a una fecha ancla (el dataset no envejece) · sin PII real (`PER-0001`, `@ejemplo.local`).

**Cuando lleguen los datos reales**: importador (CSV/JSON) al **mismo esquema** + anonimización. El modelo no cambia.

---

## 12. Plan de trabajo — 8 semanas

| Sem | Hito | Salida verificable |
|---|---|---|
| 1 | Constitution, glosario, `spec.md` v1, repo, seed mínimo | `scripts.traceability_check` en verde con 36 historias |
| 2 | E1 dominio + invariantes + verificación del seed | tests de I-1..I-13 verdes |
| 3–4 | E2 tablero de proyectos (zoom) + E3 ficha de persona | 4 capturas de zoom + E2E |
| 5 | E4 ausencias/cobertura + E5 préstamos | E2E del flujo vacaciones→cobertura |
| 6–7 | **E6 agente coordinador** (motor, tools, golden set) + E7 becarios/organigrama | golden set 12/12, 0 invariantes violados |
| 8 | E8 notificaciones y estado del agente, pulido visual, guion de demo, entrega final + vídeo | demo ≤ 10 min reproducible |

**Dependencias duras**: Sprint 0 bloquea a todos · T-401/T-402 (motor) bloquean a T-505/T-506 (agente) · T-502
(scoring) bloquea a T-503 (propuesta) · T-507 depende de T-402 y T-403.

**Orden de sacrificio si aprieta el tiempo**: E8 (notificaciones) → E7 (organigrama) → E3 (ficha de persona enriquecida).
**E6 (el agente) no se sacrifica**: es el corazón del proyecto y del curso.

---

## 13. Validación y demo

### 13.1 Golden set (12 casos, ejecutable en CI)

| # | Escenario | Comportamiento esperado | Invariante |
|---|---|---|---|
| E-01 | Proyecto al 80 % con déficit de 60 h/mes | Propone refuerzo temporal con skill match, sin exceder capacidad | I-1 |
| E-02 | Dev al 120 % entre dos proyectos | Alerta `OVERLOADED` y propone **rebalanceo**, no más carga | I-1 |
| E-03 | El único QA está de incapacidad 15 días | Alerta `COVERAGE_GAP` y propone sustituto o escala | I-5 |
| E-04 | Se crea un proyecto nuevo | Propone equipo desde perfiles + disponibilidad, respetando rol base y skills | I-1, I-6 |
| E-05 | Persona con préstamo vigente | Nunca la propone fuera de la ventana ni la duplica | I-2, I-8 |
| E-06 | Becario en el proyecto A | No lo propone en el proyecto B por encima de su límite | I-7 |
| E-07 | Incapacidad **sin fecha de regreso** | **Pregunta**; no inventa la fecha | G7 |
| E-08 | Dos proyectos críticos dependen de la misma persona | Ofrece opciones (mover / dividir / escalar) con trade-offs | G4 |
| E-09 | Dos candidatas con skill set idéntico | Desempate estable por `id` | determinismo |
| E-10 | Misma entrada repetida 3 veces | Misma propuesta, sin duplicados | G9 |
| E-11 | Proyecto origen en `EN_RIESGO` | No "roba" personal sin autorización | I-11 |
| E-12 | Mismo caso con distinta redacción | Idéntica carga y candidatos | determinismo |

**Métricas de aceptación**: 100 % de propuestas válidas · 100 % de detección en los casos sembrados · 0 falsos
positivos en el golden set · **100 % de determinismo** (misma entrada → misma propuesta) · **0 invariantes violados**.

### 13.2 Guion de demo (10 min)

1. Tablero general: 19 proyectos, zoom semana→mes→año. Señalar S-01 (rojo) y S-02 (ámbar, 78 %).
2. Click en el proyecto en ámbar → card con equipo, carga por semana, roles y quién está en préstamo.
3. Click en un analista → ficha 360: antigüedad, skills, herramientas, historial, vacaciones, cumpleaños, **logros y banner de aniversario si hoy es su día**.
4. Panel "Disponibles": el pool de desasignados con sus skills.
5. **Crear un proyecto nuevo → el agente propone asignaciones** → explicar por qué eligió a cada persona y por qué
   descartó a las otras. *(corazón de la propuesta)*
6. Aprobar → el tablero se actualiza; auditar en la bitácora.
7. Préstamo temporal: crear, ver cómo baja el origen y la fecha de devolución; mostrar que **el agente no vuelve a
   proponer a esa persona** mientras dure el préstamo.
8. **Portal de persona**: solicitar vacaciones con cobertura sugerida; si antelación ≥ 30 días → badge **Planificador**;
   si hoy es aniversario → banner + badge **ANIVERSARIO_N**.

---

## 14. Riesgos y anti-scope

Los 10 riesgos con probabiliad, impacto, mitigación y señal de alarma temprana están en
[`../docs/RIESGOS_Y_VALIDACION.md`](../docs/RIESGOS_Y_VALIDACION.md). Los tres que mandan:

1. **Scope creep visual** — el tablero "visualmente rico" se come el presupuesto y no queda la parte difícil.
   *Mitigación*: una épica entra solo si se puede demostrar en la demo; E6 se congela antes que las demás.
2. **Demo bonita, motor frágil** — el tablero funciona con datos mocks y el agente no lo usa.
   *Mitigación*: constitution Art. VIII; prohibido el mock del agente; `seed → operación → resultado visible`.
3. **El agente ignora las reglas** (falsa sensación de control).
   *Mitigación*: los invariantes viven en el **código** y en el reconciliador nocturno, no en el prompt; y hay un
   test que mira la colección completa para comprobar que no hubo escrituras sin aprobación (NFR-07).

**Lo que NO se va a hacer** (escrito para poder decir que no): multi-tenant · payroll/nómina · ATS · evaluación de
desempeño · facturación al cliente · app móvil nativa · integración real con Jira/Azure DevOps · ML para predecir
carga.

---

## 15. Defaults fijados y preguntas que quedan

Estas decisiones se fijan aquí, con el motivo explícito, para que el proyecto **no se detenga**. Cada una es un
parámetro o un cambio pequeño, no una reformulación.

### 15.1 Decisiones fijadas

| # | Decisión | Default fijado | Motivo | Cómo se revierte |
|---|---|---|---|---|
| 1 | **El 75 %** | Las tres lecturas, en tres niveles (`avanceVentanaPresion 0.75` + bandas 70/85 + alerta de cartera 75 %) | Es la única que no deja fuera ningún caso del enunciado | un parámetro |
| 2 | **Autonomía del agente** | `propose` | Constitution Art. V + BR-18 + el riesgo de un parcial | subir a `apply_with_revert` exige decisión registrada |
| 3 | **Capacidad base** | 21 días laborables/mes × 8 h, menos feriados y ausencias | Hypothesis estándar; hace comparables las cifras | `Persona.diasLaborablesSemana`, `horasDia` |
| 4 | **Factor por rol** | Manager/PO 0.50 · Scrum Master 0.80 · Tech Lead 0.60 · entrega 1.00 | La realidad de una fábrica: nadie está al 100 % en un proyecto | `Rol.factorCapacidad` |
| 5 | **Préstamos entre clientes distintos** | **No** por defecto: la propuesta se marca `REQUIERE_REVISIÓN_DE_CONFIDENCIALIDAD` y no se auto-aprueba | Es confidencialidad, no diseño; el bloqueo por defecto es la opción segura | una política por cliente |
| 6 | **PII** | Fecha de cumpleaños: solo día/mes en la UI. Motivo de incapacidad: texto libre **sin** datos clínicos; el modelo solo ve tipo y fechas | Minimización de datos de salud (G11, C11-NFR4) | — |
| 7 | **Duración** | 8 semanas | Es el marco habitual del proyecto final del curso | cronograma |
| 8 | **Nombre del producto** | «Gestión de RH y Proyectos»: describe lo que hace sin comprometer el naming | Nombre de trabajo; no bloquea nada | — |
| 9 | **Despliegue** | **Aplicación web con backend locales, sin Docker** | Es lo que pidió el cliente; la demo arranca con un solo comando sobre el seed | el guion de arranque |

### 15.2 Preguntas que **solo el cliente** puede responder

| # | Pregunta | Recomendación | Impacto si cambia |
|---|---|---|---|
| C-1 | ¿El 75 % es avance, carga o cartera? | **las tres, en niveles distintos** | un parámetro |
| C-2 | ¿El agente debe poder **aplicar** con reversión automática? | **no**: `propose` | cambia C11-R5/C11-R13 y la UX de la bandeja |
| C-3 | ¿Un préstamo temporal puede cruzar **clientes distintos**? | **no** por defecto, con revisión explícita | cambia el filtro de candidatos |
| C-4 | ¿D-07 autenticación: OIDC corporativo o sesiones con roles? | **JWT con roles** (`COORDINACION`/`RRHH`/`CONSULTA`) + hook OIDC | define el login de la demo |
| C-5 | ¿D-08 despliegue: qué forma exacta toma el arranque local? | **aplicación web con backend locales, sin Docker**, un solo comando por NFR-06 | define el guion de arranque |
| C-6 | ¿Cuántos módulos entran en la demo? | sprints 0–5 completos + endurecimiento del 6 | define el alcance de la entrega |
| C-7 | ¿El dataset real llegará antes de la semana 6? | si sí, el importador entra en el sprint 5 | si no, el sprint 5 se refuerza con casos del seed |

### 15.3 Deuda abierta

| ID | Qué falta | Default propuesto (no bloquea) |
|---|---|---|
| **D-07** | Autenticación | JWT con 3 roles + hook OIDC-ready |
| **D-08** | Despliegue | Aplicación web con backend locales, sin Docker (web + api + mongo + seed, arranque local) |

---

## 16. Deuda que encontré al ensamblar (inventario honesto)

Esta sección existe porque **un plan que dice "todo listo" y no lo está es peor que uno que dice qué falta**. Al
integrar las tres partes aparecieron estas inconsistencias; ninguna cambia el diseño, todas son trabajo de
alineación.

| # | Dónde | Qué dice | Qué debería decir | Impacto | Estado |
|---|---|---|---|---|---|
| A-1 | `designs/0004` §1.1, §4 | "Motor determinista (**TypeScript** puro)", "esquema JSON validado con **Zod**" | Python + **Pydantic** | **Código**: el motor se escribe en Python | ✅ **Corregido** (dominio y agente) |
| A-2 | `designs/0002` §5 | "check en BD", "índice único parcial", "trigger de dominio" | motor determinista + reconciliador nocturno + índices de Mongo | **Documento**: reescribir la columna "Verificación" | ✅ **Corregido** (dominio y agente) |
| A-3 | `tasks/0001-tareas.md` T-001/T-004 | "esquema de base de datos y las migraciones", "`npm run seed`", "`npm run test:criterio`" | colecciones e índices de Mongo; `pytest`; `ruff`/`mypy`/`playwright` | **Tareas**: reescribir en el vocabulario del stack final | ✅ **Corregido** (requisitos y trazabilidad) |
| A-4 | `AGENTS.md` §5 | comandos `npm run …` | comandos Python (el frontend sí es npm: vitest/playwright) | **Handoff** de la parte de arquitectura | ✅ **Corregido** (requisitos y trazabilidad; D-07/D-08 abiertas, con default fijado) |
| A-5 | `docs/DATASET_SEED.md` §4, `docs/RIESGOS` §3 | `seed.ts`, `verify-seed.ts` | `seed.py`, `verify_seed.py` | **Scripts** | ✅ **Corregido** (dominio y agente) |
| A-6 | `PROPUESTA.md` §10 y §11 | "15 escenarios" y "golden set de 10 casos" | **23 escenarios (S-01..S-23)** y **12 casos (E-01..E-12)** | **Conteos**: el canónico es el design | ✅ **Corregido** (dominio y agente) |
| A-7 | `specs/0003-proyectos` US-02.6 | "alcanza el 75 % ⇒ `EN_RIESGO`" | "alcanza el 75 % **y hay déficit** ⇒ `EN_RIESGO`" (BR-04) | **Spec**: corregido en el mismo PR | ✅ Corregido (requisitos y trazabilidad) |
| A-8 | `specs/0001-vision` §2 y §10 | dos frases corruptas ("propongaacity", "coordinador.generate") | texto limpio | **Spec**: corregido en el mismo PR | ✅ Corregido (requisitos y trazabilidad) |
| A-9 | `docs/DATASET_SEED.md` §3, `docs/RIESGOS` §3, `PROPUESTA.md` §6 | IDs de escenario inexistentes (`C-04`..`C-07`), golden set `C-01..C-10`, invariantes `I-1..I-10`, invariante equivocado (`I-6` por el becario, que es `I-7`) | `E-01..E-12`, `I-1..I-13`, `I-7` | **Documento**: referencias rotas a reglas que no existían | ✅ **Corregido** (dominio y agente) |
| A-10 | `designs/0002` §4.2 y §7, `constitution.md` §1, `docs/RIESGOS` R-03/A-09, `specs/0001-vision` | rutas a archivos inexistentes (`research.md`, `agent-design.md`) | rutas reales (`specs/0001-vision/research-sdd-y-dominio.md`, `designs/0004-agente-coordinador.md`) | **Documento**: enlaces rotos en el entregable | ✅ **Corregido** (dominio y agente) |
| A-11 | `designs/0004` §4 y `specs/0005-agente-coordinador/spec-exemplar-c11.md` C11.5 | catálogos de tools divergentes (17 vs 15) y contradicción interna: *"no existe ninguna tool que aplique"* vs `commit_assignment` con `needs_approval=True` | catálogo único de **18 tools**; el muro pasa a ser *"ninguna tool escribe una asignación sin una aprobación humana registrada"*; `validate_proposal` valida I-1..I-13 (no I1–I9) | **Diseño + spec**: el muro del agente | ✅ **Corregido** (requisitos y trazabilidad) |
| A-12 | `specs/0005-agente-coordinador/spec-exemplar-c11.md` (C11-R2, R5, R6, C11.4, C11.5) | la **plantilla** usaba un vocabulario que no era el canónico: estados `WATCH`/`CRITICAL` en vez de `SALUDABLE`/`VIGILADO`/`EN_RIESGO`/`CRITICO`, propuesta `PROPOSED` en vez de `EN_REVISION`, invariantes `I1–I9` y `I2`, severidad `WATCH\|CRITICAL\|SEVERE`, y una regla de deadband ("no proponer antes del 75 % salvo índice > 1.25") que contradecía la métrica unificada | semáforo y estados canónicos, `EN_REVISION`, `I-1..I-13`, severidad = semáforo, y el deadband sustituido por la regla de confianza de la demanda | **Spec (plantilla)**: si la plantilla no se corrige, las 13 capacidades restantes copian la regla equivocada | ✅ **Corregido** (dominio y agente) |
| A-13 | `AGENTS.md` §6, `README.md`, `docs/PROPUESTA_FINAL.md` §0/§3/§17 | "34 tareas en 7 sprints" | "**43 tareas** en 7 sprints (Sprint 0..6)" | **Conteo**: el 34 era el número de *historias*; `tasks/0001-tareas.md` tiene 40 tareas (T-001..T-606). Quien planifica con el número viejo subestima el trabajo en 6 tareas | ✅ **Corregido** (requisitos y trazabilidad, auditoría independiente) |
| A-14 | `traceability/matrix.md` (filas `US-04.5` y `US-04.7`) | `T-104` (→ US-04.5) y `T-501` (→ US-04.7) no aparecían en la fila de su US en la matriz | cada tarea aparece en la fila de su US (y se declara la convención "una tarea puede atender varias US; el US primario vive en `tasks/`) | **Trazabilidad**: 2 tareas eran invisibles para quien planifica por la matriz | ✅ **Corregido** (requisitos y trazabilidad, auditoría independiente) |
| A-15 | `designs/0002-modelo-datos.md` §0 | "el mapeo a colecciones (MongoDB) es de `plan.md`" | un `plan.md` real: mapeo entidad → colección → índices → `$jsonSchema` → carga del seed | **Artefacto ausente**: el design promete un documento que no existe en el repo; el mapeo a Mongo vive solo en prosa | ✅ **Corregido** — el artefacto existe en `designs/0002-modelo-datos/plan.md` |

> **A-9, A-10 y A-12 no estaban en el inventario** y aparecieron al re-revisar los mismos archivos después de
> corregir A-1/A-2/A-5/A-6: IDs de reglas que nadie había actualizado, rutas a documentos que ya no existen, y una
> **plantilla** que usaba un segundo vocabulario. Un entregable que dice "trazabilidad 34/34" y enlaza a
> `agent-design.md` no es trazable; y una plantilla con el semáforo equivocado reparte el error a 13 capacidades.

> **A-13, A-14 y A-15 los encontró una auditoría independiente** (script de verificación de trazabilidad: US ↔ BR ↔
> invariante ↔ tarea ↔ prueba ↔ rutas citadas) hecha **sobre el repositorio ya entregado**, no sobre el borrador.
> Salieron tres cosas: un **conteo** que mezclaba historias con tareas, dos **tareas invisibles** en la matriz, y un
> **documento prometido que no existe**. La lección es la misma de siempre: *"trazabilidad 34/34" no es una verdad,
> es una afirmación*; hay que ejecutarla contra los archivos.

> **Estado del inventario: 15 de 15 filas cerradas** (A-7 y A-8 por requisitos y trazabilidad; A-1, A-2, A-5, A-6,
> A-9, A-10 y A-12 por dominio y agente; A-3, A-4, A-11, A-13 y A-14 por requisitos y trazabilidad). Diez filas eran
> vocabulario, referencias e IDs; **A-11 y A-12 sí tocaban
> contenido**: el muro del agente se define como *aprobación registrada* (no como *ausencia de tools*, frase que el
> propio diseño se contradecía) y ahora además tiene **tres condiciones verificables** (M1 versionada, M2
> revalidación en el commit, M3 idempotencia) en `designs/0004` §4. **A-15** era un artefacto que faltaba: ya existe
> en `designs/0002-modelo-datos/plan.md` y no bloquea el plan.
> Lo que sigue abierto **no es deuda de alineación**, son las dos decisiones **D-07 (autenticación)** y
> **D-08 (despliegue)**, con default propuesto (§15) para que el plan no se detenga, y las **5 respuestas del
> cliente** (§15).

**Regla que queda escrita para el repo**: `git add <archivo>` específico (nunca `-A`) y nunca checkout de rama
encima de trabajo ajeno sin commitear. Y: *si el código revela que una spec es ambigua o falsa, se corrige la spec
en el mismo PR* (`AGENTS.md` §1.6).

---

## 17. Anexos

### 17.1 Índice de artefactos

| Artefacto | Área responsable | Estado |
|---|---|---|
| [`constitution.md`](../constitution.md) | Dominio y agente | borrador, abierto a revisión |
| [`PROPUESTA.md`](../PROPUESTA.md) | Dominio y agente | base consolidada (15 secciones) |
| [`docs/PROPUESTA_FINAL.md`](./PROPUESTA_FINAL.md) | Requisitos y trazabilidad | **este documento — entrega final** |
| [`specs/0001-vision/spec.md`](../specs/0001-vision/spec.md) | Requisitos y trazabilidad | aprobado por el cliente |
| [`specs/0001-vision/research-sdd-y-dominio.md`](../specs/0001-vision/research-sdd-y-dominio.md) | Dominio y agente | con fuentes |
| [`specs/0001-vision/research-tooling-2026.md`](../specs/0001-vision/research-tooling-2026.md) | Dominio y agente | comparativa de herramientas |
| [`specs/0002-personas/requirements.md`](../specs/0002-personas/requirements.md) | Requisitos y trazabilidad | 7 US |
| [`specs/0003-proyectos/requirements.md`](../specs/0003-proyectos/requirements.md) | Requisitos y trazabilidad | 7 US |
| [`specs/0004-tablero-personal/requirements.md`](../specs/0004-tablero-personal/requirements.md) | Requisitos y trazabilidad | 7 US |
| [`specs/0004-tablero-personal/research-frontend-stack.md`](../specs/0004-tablero-personal/research-frontend-stack.md) | UX y frontend | licencias verificadas |
| [`specs/0005-agente-coordinador/requirements.md`](../specs/0005-agente-coordinador/requirements.md) | Requisitos y trazabilidad | 8 US |
| [`specs/0005-agente-coordinador/spec-exemplar-c11.md`](../specs/0005-agente-coordinador/spec-exemplar-c11.md) | Dominio y agente | plantilla de capacidad |
| [`specs/0006-plataforma/requirements.md`](../specs/0006-plataforma/requirements.md) | Requisitos y trazabilidad | 7 US |
| [`designs/0001-arquitectura.md`](../designs/0001-arquitectura.md) | Arquitectura y plataforma | D-07/D-08 abiertas |
| [`designs/0002-modelo-datos.md`](../designs/0002-modelo-datos.md) | Dominio y agente | 12 entidades, 13 invariantes |
| [`designs/0002-modelo-datos/plan.md`](../designs/0002-modelo-datos/plan.md) | Dominio y agente | mapeo a colecciones MongoDB |
| [`designs/0003-tableros-ux.md`](../designs/0003-tableros-ux.md) | UX y frontend | 13 decisiones + mockups |
| [`designs/0003-tableros-ux.borrador-alternativo.md`](../designs/0003-tableros-ux.borrador-alternativo.md) | — | anexo preservado, no pisado |
| [`designs/0004-agente-coordinador.md`](../designs/0004-agente-coordinador.md) | Dominio y agente | 11 guardarraíles, 18 tools |
| [`designs/0005-modelo-carga-y-presion.md`](../designs/0005-modelo-carga-y-presion.md) | Dominio y agente | la métrica del 75 % |
| [`tasks/0001-tareas.md`](../tasks/0001-tareas.md) | Requisitos y trazabilidad | 43 tareas / 7 sprints (Sprint 0..6) |
| [`traceability/matrix.md`](../traceability/matrix.md) | Requisitos y trazabilidad | 36/36/36/0 + 43/43 tareas |
| [`docs/DATASET_SEED.md`](../docs/DATASET_SEED.md) | Dominio y agente | 23 escenarios |
| [`docs/RIESGOS_Y_VALIDACION.md`](../docs/RIESGOS_Y_VALIDACION.md) | Dominio y agente | 10 riesgos + demo |
| [`docs/AUDITORIA_ADVERSARIAL.md`](../docs/AUDITORIA_ADVERSARIAL.md) | Auditoría independiente | informe de trazabilidad |

### 17.2 Fuentes externas citadas (las que sostienen decisiones)

| Decisión | Fuente |
|---|---|
| El LLM formula, no calcula | MIT News (2025) — *researchers teach LLMs to solve complex planning challenges* |
| Optimización de schedules: taxonomías de restricciones duras/media/suaves | Timefold — *LLMs can't optimize schedules, but AI can* |
| Disponibilidad alucinada en agentes | arXiv 2504.14773 (tool-grounded availability) |
| Spec-anchored > spec-first | Fowler/Böckeler · Piskala (arXiv 2602.00180) · Thoughtworks Radar |
| Gates que de verdad bloquean | Consort (arXiv 2609.09671) |
| Autonomía graduada con riesgo | AWS — *Balancing speed and safety: a control framework for AI coding agents* |
| Un gate solo es control si puede rechazar | práctica documentada de SDD |

### 17.3 Glosario corto (para el que lea solo el resumen)

`Persona` · `Rol contractual base` · `Rol de proyecto` · `Asignación` (`PERMANENTE`/`TEMPORARIA`/`SUSTITUCION`/`BECARIO`)
· `Carga estimada` · `Capacidad nominal` · `Utilización` · `Índice de presión` · `Ausencia` · `Cobertura` ·
`Préstamo temporal` · `Piso de reserva` · `Propuesta del agente` (`EN_REVISION`) · `Pool de desasignados` ·
`Convenio becario` · `Bitácora`.

---

**Entregado por requisitos y trazabilidad** · 2026-10-21 · integra la opinión de todas las áreas de trabajo;
D-07/D-08 pendientes de confirmación del cliente
(actualización por delta, no reescritura).
