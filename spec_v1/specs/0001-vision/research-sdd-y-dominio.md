# Research — Evidencia para la propuesta

> Área responsable: dominio y agente · Estado: borrador para revisión (pendiente de confirmación del cliente)
> Regla: toda afirmación no marcada como **[supuesto]** va con fuente. Los supuestos se validan con el humano.

---

## 1. Qué es Spec Driven Development (y cuál nivel nos sirve)

### 1.1 Definiciones y niveles

Fowler/Böckeler (Thoughtworks, oct-2025) identifican **tres niveles** de SDD y advierte que el término está
semánticamente difuminado ("semantic diffusion"):

1. **Spec-first** — se escribe la spec antes del código; después puede archivarse.
2. **Spec-anchored** — la spec se conserva y se mantiene durante la evolución de la feature.
3. **Spec-as-source** — la spec es el archivo principal; el humano edita specs, no código.

> Fuente: https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html

**Decisión propuesta: nivel 2 (spec-anchored).** El nivel 3 (Tessl, aún en beta privada) obliga a un mapeo 1:1
spec↔archivo y elimina el código editable, lo que no es razonable para un proyecto de training que además
necesita explicar el razonamiento del agente.

Además, Fowler separa explícitamente **specs** (artefactos por feature) de **contexto/memory bank**
(`AGENTS.md`, `product.md`, `architecture.md`): el contexto es transversal a todas las sesiones; la spec,
específica de un cambio. Esto define la estructura de este repo: `.specify/memory/constitution.md` = contexto;
`specs/001-*/` = spec.

### 1.2 Herramientas SDD y flujo de artefactos

| Herramienta | Flujo | Contexto/memory | Artefactos por feature | Nota |
|---|---|---|---|---|
| **GitHub Spec Kit** | Constitution → Specify → Plan → Tasks → Implement → Converge | `constitution.md` | `spec.md`, `plan.md`, `tasks.md`, `research.md`, `data-model.md`, `api.md`, `component.md` | Plantillas + checklists en cada paso; el más personalizable. |
| **AWS Kiro** | Requirements → Design → Tasks | *steering* (`product.md`, `structure.md`, `tech.md`) | `requirements.md`, `design.md`, `tasks.md` | Requisitos estilo user story + GIVEN/WHEN/THEN. Más liviano (3 archivos). |
| **Tessl** | Spec → código generado | `KNOWLEDGE.md`, `AGENTS.md` | `.tessl/**.spec.md` | Spec-as-source; beta privada. |

> Fuentes: https://github.github.com/spec-kit/ · https://kiro.dev/docs/specs/feature-specs/ · https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html

**Decisión propuesta: Spec Kit como motor, con Kiro como referencia de redacción de requisitos.**
Motivo: Spec Kit expone artefactos separados (`research.md`, `data-model.md`) que calzan 1:1 con la separación de
trabajo de este equipo, y el paso `constitution` formaliza los invariantes del dominio — exactamente lo que
necesita un sistema con reglas duras como "respetar el préstamo temporal".
Riesgo asumido: Spec Kit es más verboso; se acota usando **una spec por épica**, no una spec gigante.

### 1.3 Los 5 riesgos de SDD (de Fowler) y cómo los mitiga esta propuesta

Fowler documenta fallas recurrentes. Cada una tiene contraprestación concreta aquí:

| Riesgo documentado | Mitigación en esta propuesta |
|---|---|
| **Sobrecarga de revisión** (spec-kit genera muchos MD repetitivos) | 1 spec por épica; `spec.md` ≤ 120 líneas; el resto del detalle vive en `designs/0002-modelo-datos.md` / `designs/0004-agente-coordinador.md` enlazados, no duplicado. |
| **Falsa sensación de control** (el agente ignora checklists e incluso la constitution) | Los invariantes se implementan en **código de dominio + tests**, no en prompts. La constitution es una capa de defensa, no la única. |
| **Separación funcional/técnica confusa** | Regla explícita: `spec.md` = QUÉ (comportamiento observable, sin nombres de tabla ni de librería); `plan.md`/`data-model.md` = CÓMO. |
| **Un unico flujo no sirve para todos los tamaños** (un bug chico genera 4 historias y 16 criterios) | Cada épica se dimensiona con criterio de corte: si no pasa el filtro "valor visible en la demo", no entra a este ciclo. |
| **Agente que ignora notas de research** (el caso de duplicar clases existentes de Fowler) | Cada artefacto declara sus "entradas" (`inputs`) y el agente de implementación debe citarlas; `/analyze` verifica referencias cruzadas. |

### 1.4 Lo que un evaluador de SDD mirará

Porque el proyecto es **final de una capacitación**, la nota no está solo en la app: está en que el proceso sea
demostrable. Tres pruebas que un evaluador hará:
1. ¿Existe `spec.md` con requisitos EARS y trazabilidad a tests? → `traceability.md`.
2. ¿Se ve el antes/después de refinar una spec (no una spec escrita de una)?
3. ¿El agente respeta las reglas cuando "se le pide" saltárselas? → **guardarraíles en código** (Constitución, Art. IV y V).

---

## 2. Dominio: asignación de recursos y carga

### 2.1 Modelo de capacidad

Oracle Primavera usa la fórmula base de la industria:

> `horas de proyecto asignadas = porcentaje de asignación × horas de capacidad`

> Fuente: https://docs.oracle.com/en/cloud/saas/project-management/25d/oapem/how-project-resource-allocation-is-calculated.html

De ahí sale la definición de **utilización** de una persona en un periodo:

```
utilización(persona, periodo) = Σ (% asignación vigente) / 100
```

**[supuesto]** Capacidad de referencia por persona y mes: `días_laborables × horas_día − holidays_no_laborables`.
No hay fuente pública fiable para "cuántos días productivos" (las métricas de productividad están en disputa:
https://metr.org/blog/2026-02-24-uplift-update/ y https://www.mckinsey.com/industries/technology-media-and-telecommunications/our-insights/yes-you-can-measure-developer-productivity),
así que **no** se codifica una "capacidad real": se parametriza y el dataset semilla la define explícitamente.

### 2.2 El umbral de 75% — dos lecturas posibles, y por qué importa

El enunciado original ("casi siempre al 75% de los proyectos empieza la presión") admite dos lecturas distintas.
El sistema **debe** soportar ambas, con una como valor por defecto:

- **L1 — Presión por proyecto**: un proyecto entra en presión cuando su capacidad comprometida supera el 75%
  de la capacidad nominal de su equipo. Bandas: `verde <70% ≤ ámbar <85% ≤ rojo`.
- **L2 — Presión de portafolio**: la presión organizacional aparece cuando ≥75% de los proyectos activos están en ámbar/rojo.
  Es una vista de cartera, no de un proyecto.

**Pregunta #1 para el humano (no bloqueante): ¿cuál de las dos es la regla principal?** El modelo implementa las dos
con `Umbral.presionProyectoPct` y `Umbral.presionPortafolioPct`, ambos a 75 por defecto, para que la decisión
sea un parámetro y no una refactorización.

### 2.3 Buscar "recursos disponibles" es un problema de optimización

La búsqueda del agente ("buscar recursos disponibles que se puedan asignar temporalmente de otro proyecto")
es, formalmente, un **problema de la mochila (knapsack)**: elegir el conjunto de personas que maximiza la cobertura
de un déficit de capacidad sujeto a restricciones (capacidad por persona, ausencia, rol, skills, compromiso de
préstamo). Está bien identificado en la literatura de asignación de recursos
(https://en.wikipedia.org/wiki/Knapsack_Problem; p.ej. *Resource Allocation Using Weighted Greedy Knapsack Based
Algorithm*, https://www.researchgate.net/publication/363747028).

**Consecuencia de diseño (importante):** el agente **no debe decidir con LLM libre**. El LLM explica y redacta;
el cálculo de factibilidad y el ranking se hacen en **código determinista** (heurística voraz + knapsack 0/1 sobre
fracciones de asignación). Esto encaja con la Constitución, Artículos IV, V y VI, y es lo que hace la demo
reproducible. Ver `designs/0004-agente-coordinador.md`.

### 2.4 Ausencias, cobertura y devoluciones

- **Vacaciones**: saldo anual por persona (`días asignados / gozados / pendientes`) — el tablero de persona
  debe mostrar el conteo, no solo los rangos.
- **Incapacidad**: es una ausencia con retorno esperado; el sistema debe saber **quién cubre** y **hasta cuándo**.
  El dato sensible (diagnóstico) no se modela (Constitución, Art. VII).
- **Préstamo temporal**: es una asignación con fecha de devolución. La regla "el agente cuide que se respete la
  asignación temporal" se implementa como **invariante con fecha de expiración** (ver `data-model.md` §5),
  no como recordatorio en un prompt.

### 2.5 Becarios

El becario tiene: convenio (periodo, tutor, horas), asignación a proyecto y rol **de proyecto** (no base).
Dato de referencia: no se encontró una fuente pública robusta sobre "periodo típico de prak" → **[supuesto]**
parametrizado en el dataset (6 y 12 meses), nunca codificado como constante de negocio.

### 2.6 Organigrama

Se modela como **proyección** de `Asignación.rol` dentro de un proyecto (rol → persona), no como una jerarquía
de personas separada: así el organigrama nunca contradice las asignaciones vigentes. Layout: grafo dirigido
con nodos persona y aristas rol; se renderiza con `@xyflow/react` (https://reactflow.dev/) + layout por jerarquía.

---

## 3. Stack — cerrado por el cliente

> **ACTUALIZADO.** El cliente respondió: *"web únicamente · los datos reales los cargaré después,
> inicia con información dummy · **react+TS+Vite, fastapi, python + mongo; los agentes dentro de la solución, con el
> estándar OpenAI**"*. El default Fastify + PostgreSQL + Prisma queda **descartado**. Esta sección se conserva como
> registro del contraste original, con las correcciones que implica el stack definitivo.

Stack definitivo: **React + TS + Vite** · **FastAPI + Python** · **MongoDB** · agentes con el **OpenAI Agents SDK**.

| Propuesta | Motivo / fuente |
|---|---|
| **Spec Kit (`specify init --ai generic`)** como motor SDD | Ya elegido; el repo de este proyecto replica su topología. |
| **Pydantic v2** para validar entrada/salida de API **y de los argumentos de tool del agente** | Convierte "el modelo pidió algo raro" en error de validación antes de tocar la base. Sustituye a Zod, que queda para el front. |
| **Vitest + Playwright** (front) + **pytest** (back) | Vitest comparte config con Vite; Playwright para los flujos E2E que son la evidencia del Art. VIII. |
| **MongoDB + motor determinista + reconciliador nocturno** en lugar de constraints de Postgres | MongoDB no tiene tipos de rango ni `EXCLUDE`: el solapamiento de asignaciones pasa a ser una **restricción de aplicación** verificada en tres puntos (motor, guardarraíl de la tool, reconciliador). Ver `PROPUESTA.md` §9. |
| **SSE** para "carga en vivo" | El tablero necesita reflejar al agente sin recargar. Un solo stream multiplexado por pestaña (límite de 6 conexiones SSE por origen en HTTP/1.1). |
| **`d3-scale`/`d3-time` para el eje multi-zoom (semana/quincena/mes/año)** | Los zoom levels son escalas temporales, no "páginas". `d3` da ticks correctos por unidad; `vis-timeline` es la alternativa si se quiere pan/zoom out-of-the-box. |
| **`@xyflow/react` para organigrama** | Es la librería de grafos/pan-zoom más usada en React. |
| **Scheduler (APScheduler o similar) para el worker programado** | El sweep del agente es idempotente y reintentable; necesita un proceso propio, no cron dentro del request. |
| **OpenAI Agents SDK (Python) para el coordinador** | Es el estándar que fijó el cliente. `@function_tool`, `output_type` estructurado, `needs_approval=True` para el gate humano, guardarraíles y tracing. Ver `designs/0004-agente-coordinador.md` §4.1. |

**[supuesto]** Versiones concretas se fijan al momento de `plan.md` (el stack está en movimiento y no se quiere
clavar en el proposal una versión que envejezca).

---

## 4. Fuentes consultadas

1. Fowler/Böckeler, *Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl*, martinfowler.com, 15-oct-2025 — https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
2. GitHub Spec Kit — docs oficiales, https://github.github.com/spec-kit/ (actualizado 28-sep-2026)
3. Kiro, *Feature Specs* — https://kiro.dev/docs/specs/feature-specs/
4. Oracle, *How Project Resource Allocation Is Calculated* — https://docs.oracle.com/en/cloud/saas/project-management/25d/oapem/how-project-resource-allocation-is-calculated.html
5. Knapsack problem — https://en.wikipedia.org/wiki/Knapsack_Problem
6. *Resource Allocation Using Weighted Greedy Knapsack Based Algorithm* — https://www.researchgate.net/publication/363747028
7. React Flow — https://reactflow.dev/ · xyflow labs — https://xyflow.com/
8. METR, *Measuring AI impact on developer productivity: an update* (2026) — https://metr.org/blog/2026-02-24-uplift-update/
9. McKinsey, *Yes, you can measure developer productivity* — https://www.mckinsey.com/industries/technology-media-and-telecommunications/our-insights/yes-you-can-measure-developer-productivity
