# Design 0004 — Agente coordinador (arquitectura, tools, guardarraíles)

> Área responsable: dominio y agente · Estado: borrador · Alineado con la spec de requisitos (US-04.1..US-04.7,
> BR-05..BR-20)
> Principio rector: **el LLM explica, el código decide**. (constitution Artículos IV, V, VI)

---

## 1. Por qué NO dejamos que el LLM decida la asignación

El pedido es "que el agente identifique dónde se vuelve crítico, busque recursos disponibles y marque los préstamos
temporales". Resuelto con LLM libre, aparecen tres fallos:

1. **Alucinación de disponibilidad** — el modelo no sabe quién está de vacaciones; lo inventa.
2. **No determinismo** — la misma situación da respuestas distintas; la demo no es reproducible y no se puede testear.
3. **Violación silenciosa de invariantes** — el modelo puede proponer 120 % de capacidad de una persona.

**Evidencia que sostiene la decisión de diseño:**

- MIT: *"As long as the inputs to the solver are correct, it will give the right answer. Any mistakes in the solution
  come from errors in the formulation process."* — el LLM **formula**, no calcula.
  (https://news.mit.edu/2025/researchers-teach-llms-to-solve-complex-planning-challenges-0402)
- Timefold: asignar turnos a 10 empleados para una semana son ~10^63 combinaciones; los LLMs chocan con límites de
  espacio de problema y de contexto. La recomendación es híbrida, con taxonomía de restricciones
  **duras / medias / suaves**. (https://timefold.ai/blog/llms-cant-optimize-schedules-but-ai-can)
- Tool-grounded availability es la mitigación contra la disponibilidad alucinada (https://arxiv.org/html/2504.14773v1).

### 1.1 Las tres capas (la decisión load-bearing)

```
┌───────────────────────────────────────────────────────────────────┐
│ CAPA 1 — Motor determinista (Python puro, SIN LLM)               │
│  utilización · presión · cobertura · conflictos · disponibilidad   │
│  valida I-1..I-13 · produce "hechos" con marca de tiempo `as_of` │
└───────────────▲───────────────────────────────────────────────────┘
                │ hechos (JSON) + ids de evidencia
┌───────────────┴───────────────────────────────────────────────────┐
│ CAPA 2 — Agente (LLM + tool calling)                              │
│  lee la capability spec · pregunta · formula propuestas · redacta│
└───────────────▲───────────────────────────────────────────────────┘
                │ propuesta
┌───────────────┴───────────────────────────────────────────────────┐
│ CAPA 3 — Revisión humana + aplicación + AuditLog                 │
│  aprobar / rechazar / editar → aplica asignación + registra      │
└───────────────────────────────────────────────────────────────────┘
```

**Corolario: el LLM nunca emite un número.** Toda cifra de una propuesta viene de una tool, y el validador la
vuelve a contrastar contra el motor antes de mostrar la propuesta. Esto mata en la raíz la "disponibilidad
alucinada".

---

## 2. Política de autonomía (decisión pendiente del humano)

| Nivel | El agente puede | El humano debe | Riesgo |
|---|---|---|---|
| `suggest` | reportar alertas y sugerencias en la UI | hacerlo todo | bajo |
| **`propose`** *(recomendado)* | crear `Propuesta` + `Alerta` | aprobar / rechazar / editar | bajo |
| `apply_with_revert` | crear asignaciones `ACTIVA` con expiración automática y un clic de deshacer | solo escalar | medio-alto |

**Recomendación: `propose`.** Demuestra el ciclo completo, mantiene la demo segura y coincide con la
recomendación de que la autonomía del agente debe bajar a medida que sube el riesgo
(AWS, https://aws.amazon.com/blogs/security/balancing-speed-and-safety-a-control-framework-for-ai-coding-agents).
En la demo **nunca** se usa `apply_with_revert`.

---

## 3. Disparo (triggers)

| Trigger | Evento | Acción del agente |
|---|---|---|
| `proyecto.creado` | Se crea un proyecto con fechas, roles plantilla y `CargaEstimada` | **Propuesta de asignación inicial** (US-04.1) |
| `avance.actualizado` | Cambia el avance de un hito o la carga real | Recalcula `indice_presion`; si entra en `EN_RIESGO`, propone refuerzo |
| `ausencia.aprobada` | RH aprueba vacaciones o incapacidad | Propone **cobertura** con plazo acotado a la ausencia (US-04.5) |
| `asignacion.creada` | Alta manual de asignación | Revalida invariantes y recalcula la presión del proyecto |
| `prestamo.por_vencer` | Faltan 7 días para `fechaDevolucionPrevista` | Recuerda la devolución o propone extender (nunca auto-extiende) |
| `sweep.diario` | Worker programado (08:00 local) | Cartera en presión, próximos cumpleaños/vacaciones, devoluciones, cobertura de incapacidades |
| `sweep.semanal` | Worker programado (lunes) | Salud de los proyectos activos, `OVERLOADED`, cumplimiento de devoluciones |
| `consulta.usuario` | Chat en la UI de la aplicación | Preguntas de estado, "quién puede cubrir X", "quién está libre" |

Cada trigger es **idempotente**: la clave `idempotencyKey = trigger + entidad + versión` evita duplicar propuestas.

---

## 4. Catálogo de tools

Cada tool tiene: propósito, esquema JSON (validado con **Pydantic v2** — el mismo validador de la API y del
`output_type` del agente), efecto (`READ` o `READ+PROPOSE`) y **contract test**. El motor de la capa 1 y las tools
viven en **Python** (mismo lenguaje que el backend FastAPI): los modelos `pydantic.BaseModel` son el contrato
único, y un argumento mal tipado falla antes de tocar Mongo.

### Lectura (no mutan nada)

| Tool | Entradas | Salida |
|---|---|---|
| `read_spec(capability)` | id de capability | Texto y versión de la spec de la capability sobre la que actúa |
| `get_project(project_id)` | id | Proyecto, roles requeridos, fechas, carga, salud |
| `get_person(person_id)` | id | Perfil, rol contractual base, skills, ausencias, préstamos |
| `compute_pressure(project_id, at)` | id, fecha | `avance, indice_presion, deficit, severity, as_of` |
| `compute_utilization(person_id, window)` | id, ventana | `util, overloaded, at_capacity, as_of` |
| `find_candidates(role, tools[], window, pct)` | filtros | Candidatos con utilización, skills, conflictos y banderas de préstamo |
| `list_absences(window)` | ventana | Ausencias aprobadas/en curso |
| `check_coverage(absence_id)` | id | `demanda, oferta, cobertura, gap_roles` |
| `list_available_pool(window, tools[])` | ventana, skills | Pool de desasignados ordenados |
| `historial_persona(person_id)` | id | Timeline: asignaciones, ausencias, ascensos, certificaciones |
| `list_loans(project_id?, person_id?)` | filtros | Préstamos vigentes con fecha de devolución y % cedido |

### Propuesta (escriben **solo** en `Propuesta` y `Alerta`)

| Tool | Entradas | Efecto |
|---|---|---|
| `validate_proposal(payload)` | borrador | Corre **I-1..I-13** sobre la propuesta → `valid, violations[]` |
| `create_proposal(kind, payload, rationale, evidence)` | tipo (`NUEVO_PROYECTO`/`REFUERZO`/`REBALANCEO`/`COBERTURA`), ítems, razón, ids de evidencia | Crea `Propuesta` en `EN_REVISION` + `AuditLog` |
| `raise_alert(kind, severity, evidence)` | tipo (`OVERALLOCATION`/`COVERAGE_GAP`/`LEAVE_UNCOVERED`/`SKILL_GAP`/`DEADLINE_RISK`), severidad, evidencia | `Alerta` visible en tableros (no muta el dominio) |
| `ask_human(question)` | una pregunta | Pregunta al humano; **una** pregunta a la vez |
| `escalate(reason)` | motivo | Entrega a un humano |
| `notify_in_app(message, link)` | texto, enlace | Publica una notificación en la bandeja de la app y deja el estado del agente visible en la UI |

### Efecto secundario (única tool que escribe en el dominio, y solo con aprobación)

| Tool | Entradas | Efecto |
|---|---|---|
| `commit_assignment(proposal_id, version)` | id de propuesta + versión/hash del payload aprobado | **Única tool con efecto secundario.** Lleva `needs_approval=True` (§4.1): la corrida **se pausa**, el humano aprueba y la tool se ejecuta. Sin esa aprobación registrada, el modelo no puede escribir una asignación |

**El muro es la aprobación, no la ausencia de tools.** Ninguna tool activa, cancela o modifica una asignación, un
préstamo o una cobertura **sin una aprobación humana registrada**. La única con efecto secundario es
`commit_assignment`, y lleva `needs_approval=True` (§4.1): el modelo no puede escribir una asignación por su
cuenta; la decisión queda en el estado de la corrida y en la bitácora. Ese es el muro de la constitution (Art. V) y
de BR-18. Aprobar también puede ocurrir en la UI (`POST /propuestas/:id/aprobar`), que escribe en el dominio y
deja traza.

#### Las tres condiciones que hacen que el muro sea un muro

Una aprobación sin más es un **muro nominal**: se puede esquivar sin querer y sin querer. La regla operativa es
*"un gate solo es control si puede rechazar"* (Consort, arXiv 2609.09671). Por eso la aprobación tiene que estar
**atada a un payload concreto** y **revalidada en el momento de aplicar**:

| # | Condición | El hueco que tapa | Requisito que la respalda | Test que lo prueba |
|---|---|---|---|---|
| **M1** | **La aprobación va versionada.** Se aprueba `proposal_id` + `version` (hash del payload). `commit_assignment` **rechaza** si el payload a aplicar no es exactamente el aprobado | Un humano aprueba "Ana, 40 %, 12/05→30/05" y el agente aplica "Ana, 80 %, 12/05→30/05": la aprobación deja de ser del algo que se revisó | US-04.6 (*"registrar las diferencias entre la versión del agente y la aprobada"*) | `test_commit_rejects_stale_version` |
| **M2** | **Revalidación en el momento del commit.** Al aplicar, el motor (E1) vuelve a correr sobre el payload *en ese momento*; si un invariante ya no se cumple (alguien entró de vacaciones entre la propuesta y el clic), aborta y la propuesta vuelve a la bandeja como "requiere corrección manual" | La propuesta se generó con un `as_of`; entre la propuesta y la aprobación el mundo cambia | US-04.6 (*"bloquear la aprobación automática y exigir corrección manual"*) + G5 | `test_commit_revalidates_invariants` |
| **M3** | **Idempotencia del commit.** La misma `(proposal_id, version)` no se aplica dos veces | Doble clic en "aprobar", o una corrida reanudada dos veces, duplican la asignación | G9 | `test_commit_idempotent` |

> Con M1..M3, el enunciado del muro es **verificable** y no una promesa: *toda escritura en `assignments` tiene una
> aprobación humana registrada, esa aprobación apunta a un payload concreto, y ese payload se revalidó contra el
> motor en el instante de escribirse*. Eso es lo que comprueba **C11-NFR3**, con un test que mira la colección
> completa — no con confianza.

---

### 4.1 Estándar OpenAI — Agents SDK (Python), dentro del backend

El cliente fijó el stack y el estándar de agentes: **los agentes viven dentro de la solución** y usan el estándar
OpenAI. Con backend FastAPI/Python, eso es el **Agents SDK** (`pip install openai-agents`), no un orquestador propio.

**Composición del backend**

```
fastapi/
├── routers/        people · projects · assignments · absences · board · proposals · alerts
├── services/
│   ├── motor.py           # determinista, sin LLM: carga, presión, cobertura, factibilidad
│   ├── assignment_service.py
│   └── audit.py
├── agent/
│   ├── coordinator.py     # Agent(...) del Agents SDK
│   ├── tools.py           # @function_tool sobre motor.py
│   ├── guardrails.py      # @input_guardrail / @output_guardrail / tool guardrail
│   ├── approvals.py       # interrupciones -> bandeja de la UI
│   └── schemas.py         # pydantic output_type
└── worker/         scheduler: recomputo nocturno + re-check por evento
```

**El coordinador**

```python
from agents import Agent, Runner, function_tool

coordinator = Agent(
    name="Coordinador de Asignaciones",
    instructions=COORDINATOR_INSTRUCTIONS,   # generado desde la spec C11 + la constitution
    tools=[...],                              # wrappers finos sobre motor.py
    output_type=AgentProposalSchema,          # salida estructurada, nunca prosa libre
)
```

- Las tools son **wrappers finos de `motor.py`**. Ninguna escribe en el dominio salvo `commit_assignment`, que solo
  escribe tras una aprobación humana registrada, **versionada al payload** y **revalidada contra el motor en el
  momento de aplicar** (M1..M3, §4).
- `output_type` (pydantic) obliga a que la salida sea una **propuesta estructurada**, no texto.
- El path server-side del SDK incluye **tracing**: el panel de Traces pasa a ser un entregable del proyecto
  (calls al modelo, tool calls, guardrails, handoffs).

**Guardarraíles en los puntos que importan**

| Guardarraíl | Tipo | Qué bloquea |
|---|---|---|
| Alcance | `@input_guardrail` | peticiones fuera del ámbito del coordinador |
| Invariantes | tool guardrail sobre la tool que aplica | violaciones de I-1..I-13 |
| Evidencia | `@output_guardrail` | propuestas sin evidencia o que se atribuyen haber aplicado algo |
| Cifras | `validate_proposal` + relectura del motor | números que no coinciden con el motor |

> **Límite del SDK que hay que respetar:** los guardarraíles a nivel de agente no corren en todas partes —
> `input_guardrails` solo aplican al primer agente de la cadena, `output_guardrails` solo al que produce la salida
> final. Por eso la validación de invariantes debe quedar **junto a la tool que produce el efecto**, no en el prompt
> ni en un guardarraíl de agente. Con un solo agente hoy no hay problema; el día que se agregue un segundo, esta
> regla sigue valiendo.

**El gate de aprobación: dos mecanismos, una recomendación**

| Mecanismo | Cómo funciona | Cuándo elegirlo |
|---|---|---|
| **A. Interrupción del SDK (recomendado)** | La tool que aplica lleva `needs_approval=True`. La corrida **se pausa**, devuelve `interruptions` + `state`; la app serializa `result.to_state()`, el humano aprueba y se **reanuda la misma corrida** con `state.approve(interruption)` + `Runner.run(agent, state)` | Cuando querés que el **agente reaccione** al rechazo (repregunta, propone alternativa). Es el camino "estándar OpenAI" |
| **B. Endpoint de aprobación en la UI** | El agente crea la propuesta en `EN_REVISION`; un humano la aprueba en la UI vía `POST /propuestas/:id/aprobar`, que escribe en el dominio | Cuando querés **desacoplar** la UI del ciclo de vida de la corrida (más simple, y sobrevive a reinicios) |

**Recomendación: A, con B como respaldo.** El mecanismo A es el que preserva el estado de la corrida y el que hace
visible la decisión humana *dentro* del ciclo del agente — y de paso cumple el requisito del cliente de que el
agente cuide que se respete la asignación temporal: si el humano rechaza un refuerzo, el agente lo sabe y propone
otra cosa en la misma corrida. El mecanismo B se conserva para el caso de "el humano aprueba desde otra pantalla
de la app, o desde otra sesión": el endpoint aplica la propuesta y marca la corrida como cancelada.

En ambos casos se cumple lo mismo: **el modelo nunca escribe una asignación por su cuenta**, y toda acción queda en
`AuditLog` con actor, evidencia y resultado.

---

## 5. Motor determinista: cómo decide

### 5.1 Presión

Se usa la métrica única de `designs/0005-modelo-carga-y-presion.md`:
`indice_presion = deficit / capacidad_restante`, con la ventana del 75 % de avance como **disparador de urgencia**,
no como generador de demanda.

### 5.2 Búsqueda de candidatos (cobertura de un déficit)

1. **Filtro de factibilidad** (invariantes I-1..I-13 + ausencias + rol + skills):
   - `capacidadLibre(persona, t) = capacidadNominal − ausencias − Σ asignaciones − Σ préstamos cedidos`
   - la persona cumple el rol requerido (contractual base o rol de proyecto) y tiene las skills pedidas;
   - **no** tiene un préstamo activo que venza después del horizonte (I-2);
   - **no** es becario sin aprobación de RRHH (I-7);
   - su proyecto de origen conserva el **piso de reserva** (I-11 / BR-08): si el préstamo lo dejaría por debajo del
     40 % de su asignación original, **no es candidato**.
2. **Ranking** (determinista, sin LLM):
   ```
   score = 0.35 × cobertura_disponible        # días que puede aportar
         + 0.25 × afinidad_de_rol              # rol contractual base == rol requerido ? 1.0 : 0.4
         + 0.15 × coincidencia_de_herramientas # Jaccard(herramientas persona, requeridas)
         + 0.10 × estabilidad                  # penaliza proyectos de origen en EN_RIESGO/CRITICO
         + 0.10 × no_duplicidad_de_prestamo     # penaliza repetir en el mismo destino
         + 0.05 × seniority_en_destino         # desempate suave, nunca por atributo sensible (I-12)
   ```
   Los pesos viven en configuración, **no** en el prompt.
3. **Selección tipo knapsack 0/1 sobre fracciones**: maximiza cobertura acumulada sin superar la capacidad de
   cada persona, con top-K (K = 10) candidatos por llamada para que la propuesta sea legible.
   Base: https://en.wikipedia.org/wiki/Knapsack_Problem

### 5.3 Reglas que el motor aplica siempre

- No propone a alguien con préstamo vigente en el horizonte; no propone a alguien de vacaciones o incapacidad.
- No recorta el % que hace cruzar a la persona de 100 %: o recorta o descarta el candidato (I-1).
- No propone sobre ausencias: la capacidad ya está descontada.
- No "roba" personal de un proyecto en `EN_RIESGO`/`CRITICO` sin autorización explícita.
- No usa atributos sensibles para puntuar (I-12): el scorer solo recibe skills, disponibilidad, rol, carga,
  historial y ausencias.
- Si los datos de demanda del proyecto son de baja confianza, **no propone**: pide actualizar la `CargaEstimada`.

---

## 6. Guardarraíles

| # | Guardarraíl | Por qué |
|---|---|---|
| **G1** | El LLM nunca calcula fechas, porcentajes ni duraciones: solo los lee de las tools | Disponibilidad alucinada |
| **G2** | El agente nunca escribe una `Asignación` por su cuenta: escribe una `Propuesta`, y la única escritura ocurre tras una aprobación humana registrada, versionada al payload y revalidada contra el motor (M1..M3). Autonomía por defecto `propose` | El agente no compromete personal sin aprobación humana |
| **G3** | Las restricciones duras viven en el motor (I-1..I-13), nunca en el prompt | Sobre-asignación y conflictos de ownership |
| **G4** | Cada propuesta cita **ids de evidencia** (persona, proyecto, asignación, ausencia) y los hechos numéricos usados | Nobody puede revisar lo que no puede trazar |
| **G5** | Todo hecho lleva `as_of`; los snapshots rancios se refrescan o la propuesta se marca como tal | Alucinación por contexto obsoleto |
| **G6** | Auto-verificación: antes de enviar, llama `validate_proposal`; si falla, corrige **como máximo dos veces** y luego escala | No preguntar por aclaración = 11,65 % de las fallas de MAST |
| **G7** | Preguntar, no suponer: dato faltante (p. ej. fecha de regreso desconocida) → pregunta, nunca inventa | Convención `[NEEDS CLARIFICATION]` de spec-kit; "one question at a time" de Tessl |
| **G8** | El agente respeta los préstamos temporales vigentes y nunca reasigna a alguien prestado sin consentimiento del proyecto origen | Requisito explícito del cliente (US-04.4) |
| **G9** | Idempotencia: clave de propuesta `(proyecto, rol, ventana, candidato)` evita duplicados al reintentar | Repetición de pasos = 17,14 % de las fallas de MAST |
| **G10** | Interruptor global de apagado + banner "agente en pausa" | Seguridad operativa |
| **G11** | El agente no lee ni expone notas médicas; solo tipo de ausencia y fechas | Minimización de datos de salud |

---

## 7. Contrato de prompts

Tres prompts versionados en el repo (`agent/prompts/`), con hash registrado en la bitácora.

### 7.1 `system.coordinator.md`

```
Eres el Coordinador de Asignaciones de una fábrica de software.
OBSERVAS, CALCULAS (con las tools), EXPLICAS y PROPONES.

Reglas inviolables:
1. No afirmes un dato que no venga de una tool. Si no lo sabes, llama a la tool.
2. Nunca actives, modifiques o canceles una asignación, un préstamo o una cobertura.
3. Toda propuesta incluye: persona, rol, %, fechas, razón, y las alternativas descartadas con su motivo.
4. Si la información es ambigua, pregunta (una pregunta a la vez) y registra la pregunta.
5. Explica en español claro, para un manager de RRHH. Sin jerga.
6. Cada propuesta cita sus ids de evidencia y los hechos numéricos usados.
7. Antes de proponer, llama validate_proposal. Si hay violaciones, corrige (máx. 2 intentos) o escala.
```

### 7.2 Salida de las tools
JSON con `datos`, `as_of`, `umbralesAplicados` y `evidencias[]`. **El motor decide qué incluye**, no el modelo.

### 7.3 `output.propuesta.md`

```
Proyecto: <nombre> — índice de presión 1.18 (EN_RIESGO) · avance 78 % (ventana de presión)
Déficit: faltan 6 días-persona/semana de <Rol> entre <t0> y <t1>

Propuesta (pendiente de aprobación):
 • Persona A — <Rol> — 40 % — del 12/05 al 30/05 — razón: …
   Evidencia: persona #p12, asignación #a44, ausencia #x01
   Alternativas descartadas: Persona B (sin PostgreSQL), Persona C (proyecto origen EN_RIESGO, piso de reserva)
 • Persona D — <Rol> — 20 % — … — razón: …
Impacto: el índice de presión pasa de 1.18 a 0.72. Persona B (origen) baja de 85 % a 70 % (piso 40 % OK).
⚠ Requiere aprobación de: Project Manager + RRHH
```

---

## 8. Superficie de interacción

1. **En la app**: panel del Coordinador con chat, bandeja de propuestas (aprobar/rechazar con motivo), alertas
   activas y el "por qué" de cada decisión.
2. **Notificaciones dentro de la app**: el agente publica en la bandeja de notificaciones el enlace a la propuesta y un
   resumen semanal de la presión de cartera, y deja el **estado del agente** visible en el panel (hechos `as_of`,
   alertas, próximas devoluciones).
3. **Corte global**: un interruptor "agente en pausa" visible en la UI (G10).

---

## 9. Evaluación (cómo se demuestra que funciona)

**Golden set** ejecutable en CI. Cada caso tiene entrada, expectativa y un invariante que no debe romperse.

| # | Escenario | Comportamiento esperado | Invariante |
|---|---|---|---|
| E-01 | Proyecto al 80 % de avance con déficit de 60 h/mes | Propone refuerzo temporal con skill match, sin exceder la capacidad del candidato | I-1 |
| E-02 | Dev al 120 % entre dos proyectos | Alerta `OVERLOADED` y propone **rebalanceo**, no más carga | I-1 |
| E-03 | El único QA del proyecto está de incapacidad 15 días | Alerta `COVERAGE_GAP` y propone sustituto QA o escala | I-5 |
| E-04 | Se crea un proyecto nuevo | Propone equipo desde perfiles + disponibilidad, respetando rol base y skills | I-1, I-6 |
| E-05 | Persona con préstamo temporal vigente | Nunca propone moverla fuera de la ventana ni duplica | I-2, I-8 |
| E-06 | Becario en el proyecto A | No propone el proyecto B por encima del 30 % | I-7 |
| E-07 | Incapacidad sin `fecha_regreso` conocida | **Pregunta**; no inventa la fecha | G7 |
| E-08 | Dos proyectos críticos dependen de la misma persona clave | Ofrece opciones (mover / dividir / escalar) con trade-offs explícitos | G4 |
| E-09 | Dos candidatas con skill set idéntico | Desempate estable por `id` (determinismo) | determinismo |
| E-10 | Misma entrada repetida 3 veces | Misma propuesta, sin duplicados | G9 |
| E-11 | Proyecto origen en `EN_RIESGO` | No "roba" personal sin autorización | I-11, R-02 |
| E-12 | Mismo caso 3 veces con distinta redacción | Identical carga y candidatos | determinismo |

**Métricas**:
- **Tasa de validez de propuestas** (pasa `validate_proposal`) — objetivo **100 %**.
- **Recordatorio de alertas** sobre el golden set — objetivo 100 % en los casos sembrados.
- **Tasa de falsos positivos** — objetivo 0 en el golden set.
- **Corrección de aclaración**: pregunta cuando debe y no cuando no.
- **Determinismo**: 100 % de propuestas idénticas con la misma entrada.
- Latencia y costo por propuesta (observable, no bloqueante).

---

## 10. Riesgos del diseño del agente

| Riesgo | Mitigación |
|---|---|
| El LLM "se inventa" datos de una persona | Toda afirmación viene de una tool; G1; golden set E-02/E-03/E-07 |
| Un "robo" de personal a un proyecto crítico | Piso de reserva (I-11) + estabilidad en el score + autorización explícita |
| Propuestas que nadie revisa | Alertas con SLA visible, bandeja priorizada y expiración de la propuesta |
| Proposal spam | Idempotencia (G9) + cooldown por entidad + expiración |
| Deriva de la spec (el agente empieza a hacer cosas no especificadas) | Las tools son la **única** superficie de acción; añadir una tool es un cambio de spec |
| Privacidad | Solo tipo de ausencia y rango de fechas; G11; el motor no recibe atributos sensibles (I-12) |
| Dependencia de un modelo concreto | El motor es agnóstico: cambiar de LLM no cambia las cifras (solo la redacción) |
