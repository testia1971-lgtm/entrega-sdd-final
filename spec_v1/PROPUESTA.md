# Propuesta de proyecto final — Sistema de Gestión de RH y Asignaciones

**Training:** Codificación de agentes de IA · **Metodología:** Spec Driven Development
**Repositorio:** `sdd-rrhh-proyectos` · **Estado:** base consolidada — la **entrega final** es [`docs/PROPUESTA_FINAL.md`](docs/PROPUESTA_FINAL.md)

> 📌 Este documento es la **base consolidada**, escrita a partir de las opiniones de las distintas áreas de trabajo.
> La **entrega final** coordina requisitos y ensambla la versión oficial:
> ➡️ **[`docs/PROPUESTA_FINAL.md`](docs/PROPUESTA_FINAL.md)**. Este archivo queda como base; ante cualquier
> discrepancia entre ambos, **manda el documento final**.
> Material de detalle en `specs/`, `designs/`, `tasks/` y `docs/`.

---

## 1. Resumen ejecutivo

Se propone construir **una aplicación web de gestión de recursos humanos para una fábrica de software**, con un
**agente IA coordinador** que vigila la carga de los proyectos, detecta dónde la presión se vuelve crítica y
**propone** soluciones (asignaciones, refuerzos, préstamos temporales, coberturas) que una persona aprueba.

El proyecto existe para **demostrar Spec Driven Development de punta a punta**: cada funcionalidad nace de una
spec con criterios de aceptación verificables, pasa por un diseño y un plan, se descompone en tareas trazables y
se cierra con evidencia de ejecución.

**El diferenciador no es el dashboard: es el agente con reglas duras.** La parte visual (tableros, Gantt multi-zoom,
ficha de persona, organigrama) es el escenario donde se ve el resultado; la parte sustantiva es un motor determinista
que sabe quién está libre, quién está de vacaciones, quién está prestado y a quién no se puede tocar.

---

## 2. Problema

Una fábrica de software con cientos de personas y decenas de proyectos simultáneos pierde visibilidad sobre:

- **Quién está en qué proyecto, con qué rol y con qué % de su tiempo** (una persona puede tener varios roles y proyectos).
- **Dónde empieza la presión**: normalmente cuando el proyecto ya lleva ~75 % de su plazo transcurrido y la capacidad
  restante del equipo no alcanza para el trabajo pendiente (ver §14.1 para la formalización en tres niveles).
- **Qué hacer cuando alguien falta** (vacaciones, incapacidad) y **quién cubre** y hasta cuándo.
- **Qué pasa con los préstamos temporales**: la persona se va a otro proyecto y nunca vuelve, o vuelve tarde.
- **Quién está disponible** y **quiénes son becarios** con convenio y periodo vigente.

Hoy esto vive en planillas y en la cabeza de los managers. El costo es claro: sobrecarga de personas,
proyectos que se atrasan porque nadie vio venir la presión, y decisiones de asignación que no se pueden explicar.

---

## 3. Objetivo y alcance

### Objetivo
Un sistema que (a) muestra la asignación de personas de forma gráfica y navegable, y (b) mantiene un agente
coordinador que **detecta la presión y propone respuestas explicadas**, respetando siempre las reglas del negocio.

### Dentro de alcance
1. Tablero de proyectos con vista Gantt y **zoom semana / quincena / mes / año**, coloreado por presión.
2. Fichas de proyecto (click) y **ficha de persona** (click) con historial, skills, herramientas, antigüedad, vacaciones, cumpleaños.
3. Gestión de **asignaciones** (persona ↔ proyecto ↔ rol ↔ % ↔ fechas), con roles base y roles por proyecto.
4. **Ausencias**: vacaciones (con saldo anual) e incapacidad, con **cobertura** asociada.
5. **Préstamos temporales** con fecha de devolución obligatoria y respetada por el agente.
6. **Becarios**: convenio, tutor, periodo de asignación, rol en el proyecto.
7. **Organigrama del proyecto** derivado de las asignaciones vigentes.
8. **Panel de desasignados/disponibles** para asignación inmediata.
9. **Agente coordinador**: vigilancia de carga, propuestas de asignación (incluido al crear un proyecto),
   propuestas de cobertura, control de préstamos y avisos.
10. Spec Driven Development como proceso: repo con `specs/`, constitution, trazabilidad requisito→prueba.

### Fuera de alcance (explícito)
Nómina, ATS/contratación, evaluación de desempeño, multi-empresa, facturación al cliente, app móvil nativa,
integración real con Jira/Azure DevOps, y ML predictivo de carga (el motor es heurístico y determinista a propósito).

---

## 4. Usuarios

| Usuario | Necesidad principal |
|---|---|
| **HR Manager** | Ficha de persona, antigüedad, vacaciones, cumpleaños, historial, panel de disponibles |
| **Project Manager** | Presión del proyecto, quién falta, propuesta de refuerzo/cobertura, organigrama |
| **Scrum Master / Tech Lead** | Quién está asignado y con qué rol, vista de carga semanal |
| **Coordinador (agente IA)** | Observa todo, propone, explica, no decide |

---

## 5. Épicas

Cada épica se entrega con: spec con criterios EARS, diseño, tareas, pruebas y evidencia de ejecución.

| # | Épica | Resultado observable | Riesgo principal |
|---|---|---|---|
| **E1** | Modelo de dominio y datos | Personas, roles, proyectos, asignaciones, ausencias, préstamos, **logros, solicitudes de vacaciones**, invariantes verificados por test | Reglas ambiguas → datos sin sentido |
| **E2** | Tablero de proyectos | Gantt con zoom semana/quincena/mes/año, color por presión, click → card del proyecto | Que el scroll no escale bien |
| **E3** | Ficha de persona | Antigüedad, skills, herramientas, historial completo, vacaciones, cumpleaños, **logros y reconocimiento de aniversario** | "Bonito" vs "informativo" |
| **E4** | Ausencias y cobertura | Solicitar/aprobar vacaciones e incapacidad; **autogestión de vacaciones con cobertura sugerida**; cobertura con un plazo acotado | Reglas de saldo y cobertura |
| **E5** | Préstamos temporales | Crear préstamo, baja el % del origen, fecha de devolución, el agente lo respeta | Que el "%" se desincronice |
| **E6** | Agente coordinador | Detecta presión, propone asignaciones al crear proyecto, propone cobertura, explica descartes | Que el agente "haga theater" |
| **E7** | Becarios y organigrama | Convenio vigente, asignación válida, organigrama por rol | Salir del convenio sin avisar |
| **E8** | Notificaciones y estado del agente | El agente notifica dentro de la app y deja su estado visible en la UI | Que se quede en promesa |
| **E9** | **Gamificación y portal de persona** | **Aniversarios celebrados, logros desbloqueados, autogestión de vacaciones con badges** | Mantenerlo ligero sin competir con el scoring del agente |

> Detalle de historias y criterios de aceptación: `specs/000X-*/requirements.md` y `traceability/matrix.md`.

---

## 6. Modelo de dominio (resumen)

Entidades: `Persona`, `Rol` (base y de proyecto), `Proyecto`, `Asignación`, `Ausencia`, `SaldoVacaciones`,
`Competencia`/`Herramienta`, `ConvenioBecario`, `Propuesta`/`PropuestaItem`, `BitacoraEvento`, `Feriado`,
**`Logro`**, **`LogroPersona`**, **`SolicitudVacaciones`**.

**Semántica temporal**: todo periodo es `[desde, hasta)`. El histórico es inmutable: un cambio de % cierra una
asignación y abre otra, con traza en la bitácora.

**Cálculos**:
```
utilización(persona, periodo)      = Σ % de asignaciones vigentes / 100
carga(proyecto, periodo)           = Σ (%/100) × capacidad(persona) × factorCapacidad(rol)
presión(proyecto, periodo)         = carga / capacidad del equipo
persona disponible                 = ACTIVA ∧ sin asignaciones vigentes ∧ sin ausencia bloqueante
```

**Invariantes críticos** (implementados en el motor determinista + guardarraíl de tool + reconciliador nocturno, nunca
solo en el prompt del agente): `I-1` Σ% ≤ 100 por persona · `I-2` todo préstamo tiene fecha de devolución ·
`I-3` el préstamo cede capacidad del proyecto origen · `I-5` la cobertura termina a más tardar al regreso ·
`I-7` un becario no se asigna fuera de su convenio · `I-8` no hay asignaciones en proyectos cerrados.

Detalle completo: `designs/0002-modelo-datos.md`.

---

## 7. El agente coordinador

**Principio: el LLM explica, el código decide.** Un motor determinista (Python puro, sin LLM) calcula carga,
factibilidad y ranking; el LLM redacta la propuesta, explica los descartes y conversa. Esto garantiza que
**el agente nunca inventa disponibilidad** y que la demo sea **reproducible**.

**Disparadores**: creación de proyecto · actualización de progreso · ausencia aprobada · asignación creada ·
préstamo por vencer · sweep diario y semanal · consulta del usuario.

**Herramientas** (catálogo único de **18**, `designs/0004` §4): 11 de lectura (`read_spec`, `get_project`,
`get_person`, `compute_pressure`, `compute_utilization`, `find_candidates`, `list_absences`, `check_coverage`,
`list_available_pool`, `historial_persona`, `list_loans`), 6 de propuesta (`validate_proposal`, `create_proposal`,
`raise_alert`, `ask_human`, `escalate`, `notify_in_app`) y **1 con efecto secundario** (`commit_assignment`).

**Muro de seguridad**: el muro **es la aprobación**, no la ausencia de tools. Ninguna tool escribe una asignación, un
préstamo o una cobertura sin una **aprobación humana registrada**; la única con efecto secundario es
`commit_assignment` (`needs_approval=True`), y su aprobación va **versionada al payload** y se **revalida contra el
motor en el momento de aplicar** (M1..M3 en `designs/0004` §4). El agente escribe propuestas en `EN_REVISION`; una
persona aprueba (constitution Art. V, BR-18). Además, el motor nunca propone a alguien con préstamo vigente en el
horizonte, ni a quien está de vacaciones, ni a quien excede el 100 %, ni "roba" personal de un proyecto en riesgo,
ni usa atributos sensibles para puntuar. **11 guardarraíles** (G1..G11) en
`designs/0004-agente-coordinador.md` §6.

**Scoring de candidatos** (determinista, pesos en configuración): cobertura disponible 35 % + afinidad de rol 25 % +
coincidencia de herramientas 15 % + estabilidad del proyecto origen 10 % + penalización por préstamo repetido 10 % +
seniority en destino 5 %; selección tipo knapsack sobre fracciones de asignación. El piso de reserva del proyecto
origen (40 %) es un **filtro duro**, no un peso.

**Presión**: una sola métrica, `indice_presion = deficit ÷ capacidad_restante`, con la ventana del 75 % de avance
como disparador de urgencia y no como generador de demanda (`designs/0005-modelo-carga-y-presion.md`).

Detalle completo: `designs/0004-agente-coordinador.md`.

---

## 8. Spec Driven Development aplicado

### Estructura del repo (topología `specs/ → designs/ → tasks/`)
```
constitution.md                       # 11 artículos inmutables
PROPUESTA.md                          # este documento
specs/0001-vision/research-sdd-y-dominio.md      # evidencia SDD + modelo de asignación
specs/0001-vision/research-tooling-2026.md       # comparativa de herramientas SDD
specs/000{2..6}-*/requirements.md                # QUÉ
designs/0002-modelo-datos.md         # CÓMO: entidades, temporalidad, invariantes I-1..I-13
designs/0004-agente-coordinador.md   # CÓMO: motor, tools, guardarraíles G1..G11
designs/0005-modelo-carga-y-presion.md # CÓMO: qué significa el 75 %
tasks/0001-tareas.md                  # QUIÉN LO HACE: tareas con US-* y test
traceability/matrix.md               # US-* ↔ diseño ↔ tarea ↔ test ↔ estado
docs/DATASET_SEED.md · docs/RIESGOS_Y_VALIDACION.md
```

### Decisión de tooling (con evidencia)
- **Sintaxis de requisitos**: **EARS** (`WHEN <trigger> THE SYSTEM SHALL <respuesta>`), de Kiro — la mejor
  formalización disponible y la que ya usa la spec de requisitos.
- **Convención `[NEEDS CLARIFICATION: …]`** de spec-kit: obliga al agente a preguntar en vez de suponer.
- **Constitution + plantillas** de spec-kit, sobre la topología `specs/ designs/ tasks/`.
- **Puertas de CI que de verdad bloquean**: `spec:validate`, `test:invariantes`, `test:golden`, `test:seed`.
  Regla operativa: *un gate solo es control si puede rechazar* — falla duro en `main`, es informativo en las ramas.
- **Nivel de SDD elegido**: *spec-anchored* (la spec sobrevive a la implementación y se valida en CI), no
  *spec-as-source* (Tessl, aún en beta): el código sigue siendo editable por humanos.
- **Alternativa sin dependencias**: la misma forma de carpetas a mano + `python -m scripts.spec_validate` propio.
  Fuentes y comparativa completa (Spec Kit, Kiro, OpenSpec, BMAD, GSD, cc-sdd) en
  `specs/0001-vision/research-tooling-2026.md`.

### Ciclo
`constitution → /specify → /plan → /tasks → /implement → /analyze`

- **`specs/000X-*/requirements.md`**: épicas, historias de usuario y criterios de aceptación en EARS (`US-*`).
- **`designs/`** (el `/plan` de este repo): stack, arquitectura, modelo de datos, contratos de API, modelo de carga y presión.
- **`tasks/0001-tareas.md`**: tareas con referencia a los `US-*`/`BR-*` que cierran.
- **`traceability/matrix.md`**: matriz `requisito → historia → diseño → tarea → prueba → estado`.
- **`specs/0001-vision/research-*.md`**: decisiones con evidencia externa, no solo opinión.

### Constitución (resumen de los 11 artículos)
Spec como fuente de verdad · todo requisito es verificable y trazable · el glosario manda · los invariantes se
implementan en el dominio, no en la UI · **el agente propone y la persona decide** · determinismo y auditabilidad ·
privacidad por diseño · la demo es parte del entregable · **prohibido puntuar con atributos sensibles** ·
autonomía graduada con apagado de emergencia · un único glosario, sin sinónimos.
Detalle: `constitution.md`.

### Por qué esta variante (evidencia)
El término "SDD" tiene tres niveles (spec-first / spec-anchored / spec-as-source) y se está semanticamente
difuminando; además, la práctica documenta cinco fallas: sobrecarga de revisión, falsa sensación de control,
confusión entre spec funcional y técnica, workflows que no escalan de tamaño y agentes que ignoran las notas de
research. La propuesta elige **Spec Kit + nivel spec-anchored** y mitiga cada falla de forma explícita.
Fuentes en `specs/0001-vision/research-sdd-y-dominio.md` y `specs/0001-vision/research-tooling-2026.md`.

---

## 9. Stack y arquitectura

### El stack está decidido por el cliente (cierre de la pregunta pendiente)

En el canal, el cliente respondió explícitamente: *"web únicamente · los datos reales los cargaré después, inicia con
información dummy · **react+TS+Vite, fastapi, python + mongo; los agentes dentro de la solución, con el estándar
OpenAI**"*. La variante Fastify + PostgreSQL + Prisma queda **descartada**. El stack de esta propuesta es:

| Capa | Elección | Nota |
|---|---|---|
| Frontend | React + TypeScript + Vite, TanStack Query, Tailwind + shadcn/ui | Timeline propio sobre `d3-scale` + `d3-zoom`; organigrama con `@xyflow/react` + `@dagrejs/dagre`; paneles con Apache ECharts |
| Gráficos / organigrama | `@xyflow/react` + `@dagrejs/dagre` + Apache ECharts | Licencias verificadas una por una en `specs/0004-tablero-personal/research-frontend-stack.md` (ojo: el `resource-timeline` de FullCalendar es premium y PrimeReact v11 ya no es MIT) |
| Backend | **FastAPI + Python** | Pydantic v2 valida también los argumentos de las tools del agente |
| Datos | **MongoDB** | El motor determinista garantiza los invariantes (ver abajo) |
| Agente | **OpenAI Agents SDK (Python, `openai-agents`)** dentro del backend | Motor determinista separado del LLM. Ver `designs/0004-agente-coordinador.md` §4.1 |
| Real-time | **SSE** (un solo stream multiplexado por pestaña; ojo: el límite de 6 conexiones SSE por origen en HTTP/1.1) | `Last-Event-ID` para reanudar |
| Tests | pytest + Vitest + Playwright | Los E2E son la evidencia de la demo |
| SDD | Topología `specs/ designs/ tasks/` + EARS + `[NEEDS CLARIFICATION]` + constitution | Ver §8 |
| Despliegue | **Aplicación web con backend locales, sin Docker** | La demo se sirve como aplicación web; el backend (FastAPI + MongoDB) corre en local |

### Cómo se imponen los invariantes en MongoDB

El stack es agnóstico del modelo de dominio, pero **la forma de imponer las reglas duras cambia con la base de
datos**, y conviene dejarlo escrito para que nadie lo descubra tarde:

| Necesidad | Con PostgreSQL | Con **MongoDB (lo que elegimos)** |
|---|---|---|
| Solapamiento de asignaciones | `daterange` + `EXCLUDE USING GIST` -> la BD lo rechaza | Se calcula en el motor determinista + capa de servicio |
| Invariantes I-1..I-13 | constraints, checks y `WITHOUT OVERLAPS` (PG 18) | servicio + guardarraíl de tools + **reconciliador nocturno** que escanea violaciones y alerta |
| Días laborables | tabla `feriados` | colección `holidays` (misma información; `date-holidays` para el seed) |
| Índices | índices GiST sobre rangos | índices compuestos `{person_id, start_date, end_date}` y `{project_id, start_date, end_date}` |

> El principio **no cambia**: *las restricciones duras viven en el motor determinista, nunca en el prompt del LLM*.
> MongoDB no tiene tipos de rango ni constraints de exclusión, así que **la defensa extra es el reconciliador
> nocturno**: un job que escanea las colecciones buscando violaciones de I-1..I-13 y abre una alerta si encuentra
> alguna. Es la misma propiedad (dos puntos de enforcement) que daba la constraint de Postgres, pero con un motor
> propio.

---

## 10. Datos

Dataset **sintético y determinista** (semilla fija): 120 personas (108 empleados + 12 becarios), 19 proyectos
(12 activos), 36 meses de histórico, 24 feriados. Incluye **23 escenarios sembrados a propósito** (S-01..S-23:
proyecto al 92 %, proyecto al 78 %, personas al 100 %, préstamo vigente, incapacidad sin cubrir, becario al final de
convenio, pool de desasignados, dos candidatas con skills idénticas para probar el determinismo, **aniversarios con badges, solicitudes de vacaciones con y sin cobertura, Planner badge**…). El generador **se
autovalida** contra los invariantes antes de escribir: un seed que viola una regla es un bug del seed.
Detalle: `docs/DATASET_SEED.md`.

---

## 11. Riesgos, validación y demo

Los 10 riesgos con su mitigación están en `docs/RIESGOS_Y_VALIDACION.md`. Los tres que mandan:

1. **Scope creep visual**: se congela la épica del agente antes que las demás.
2. **Demo bonita y motor frágil**: prohibido el mock del agente; todo se demuestra con el flujo real.
3. **El agente ignora las reglas**: los invariantes viven en el código y en la base, no en el prompt.

**Validación**: golden set de 12 casos (E-01..E-12: proyecto nuevo, sobre-asignado, de vacaciones, prestado,
proyecto rojo, incapacidad, empate, idempotencia, becario, umbral, dos candidatos idénticos, proyecto origen en
riesgo). Métrica mínima: **0 invariantes violados** y **100 % de determinismo**.

**Guion de demo (10 min)**: tablero y zoom → ficha de proyecto → ficha de persona → panel de disponibles →
**crear proyecto y ver la propuesta del agente** → aprobar y auditar → préstamo temporal y su fecha de devolución.

---

## 12. Cronograma sugerido (8 semanas)

| Sem | Hito |
|---|---|
| 1 | Constitution, glosario, `requirements.md` v1, repo Spec Kit, seed mínimo |
| 2 | E1 dominio + invariantes + `verify_seed` |
| 3–4 | E2 tablero de proyectos (zoom) + E3 ficha de persona |
| 5 | E4 ausencias/cobertura + E5 préstamos |
| 6–7 | **E6 agente coordinador** (motor determinista, tools, golden set) + E7 becarios/organigrama |
| 8 | E8 notificaciones + **E9 gamificación y portal** + pulido visual, guion de demo, entrega final MD + vídeo |

> Si el tiempo aprieta, el orden de sacrificio es: E8 (notificaciones) → E7 (organigrama) → E3 (ficha de persona enriquecida).
> **E6 no se sacrifica**: es el corazón del proyecto.

---

## 13. Definición de éxito

- Un revisor puede abrir el repo y ver, para cada requisito, la spec, la tarea y la prueba que lo cierran.
- El tablero muestra los 19 proyectos con zoom funcional en 4 niveles y refleja la presión real.
- El agente, ante un proyecto nuevo, propone asignaciones **factibles**, **explicadas** (incluye los descartes)
  y **reproducibles**.
- Ningún escenario del golden set viola un invariante.
- Existe un vídeo de ≤ 10 minutos que recorre la demo.

---

## 14. Preguntas abiertas (no bloqueantes)

### 14.1 El umbral de 75 % — un parámetro, tres niveles (cerrado por diseño)

El enunciado admite **tres lecturas** y las tres son defendibles:
(a) **avance** del proyecto, (b) **carga** comprometida del equipo, (c) **cartera**: % de proyectos de la fábrica en
problemas. Implementadas como reglas sueltas, el agente dispararía tres alertas contradictorias sobre el mismo
proyecto. La propuesta las resuelve con **una sola métrica y tres niveles**
(`designs/0005-modelo-carga-y-presion.md`):

```
indice_presion = deficit / capacidad_restante     # el hecho
ventana_75     = avance >= 0.75                   # cuándo mirar
```

- `avance < 75 %` y sin déficit → `SALUDABLE`
- `avance >= 75 %` y sin déficit → `VIGILADO` (reporta, **no propone**; el problema es de skills, no de falta de gente)
- `indice_presion > 1.00` → `EN_RIESGO` (propone refuerzo) · `> 1.25` → `CRITICO` (propone y escala)
- `% de la cartera en EN_RIESGO/CRITICO >= 75 %` → alerta de dirección (nivel de cartera)

**Respuesta a la pregunta abierta:** el enunciado del cliente habla de *"al 75 % **de los proyectos** empieza
la presión"*, que es la lectura **(a) avance** como **ventana**. El disparo real es el **déficit**, no el porcentaje: si
el agente solo usara "pasó 75 % → buscar gente", inventaría demanda en proyectos que van bien, que es el fallo
clásico de alucinación de necesidad. Por eso el 75 % queda como **parámetro de configuración** (`presion.avance_umbral`,
default `0.75`), no como constante en el código: si el cliente confirma una sola lectura, se cambia un parámetro y el
diseño no se toca.

### 14.2 Preguntas que siguen abiertas

1. **Datos**: el cliente ya respondió — *"los datos reales los cargaré después, inicia con información dummy"*.
   → **dataset semilla determinista** como punto de partida, más un importador (CSV/JSON) al mismo esquema con
   anonimización para cuando lleguen los reales. Ver `docs/DATASET_SEED.md`.
2. **Capacidad**: ¿21 días laborables × 8 h? ¿4×10 o 5×8? ¿Un Manager cuenta como 50 % o 100 % en el proyecto?
3. **Nivel de autonomía del agente**: `suggest` / `propose` / `apply_with_revert`. **Propuesta y default: `propose`** —
   el agente crea propuestas y alertas; nunca aplica. `apply_with_revert` queda documentado pero desactivado.
4. **PII**: ¿se guarda la fecha de nacimiento exacta o solo día/mes? ¿El motivo de incapacidad es texto libre?
   (Recomendación: el agente **solo** ve tipo y fecha de las ausencias; el motivo queda en el backend y nunca sale
   hacia el modelo — ver C11-NFR4 en `specs/0005-agente-coordinador/spec-exemplar-c11.md`.)
5. **Duración**: ¿8 semanas es el marco real del proyecto final?
6. **Intereses cruzados**: ¿los préstamos temporales pueden cruzar proyectos de **clientes distintos**? Es una pregunta
   de **confidencialidad**, no de diseño. **Default recomendado: no** — un préstamo no cruza la frontera de un cliente
   salvo autorización explícita, y el filtro de candidatos lo aplica como restricción dura del motor (no del prompt).

### 14.3 Preguntas ya resueltas (histórico)

- ~~**Stack**: dos lecturas en circulación~~ → **cerrado por el cliente**: React + TS + Vite · FastAPI + Python ·
  MongoDB · agentes con el estándar OpenAI. Ver §9.
- ~~**App propia o integrada con un canal externo**~~ → **la app web es la fuente de verdad**: la notificación, el
  estado del agente y el flujo de aprobación ocurren dentro de la propia aplicación web.
- ~~**App propia o integrada**~~ · **Web únicamente**, sin móvil.
- ~~**Despliegue en contenedores**~~ → **aplicación web con backend locales, sin Docker** (ver `docs/PROPUESTA_FINAL.md`
  §15, D-08).

---

## 15. Responsabilidades y estado

| Área | Aporta | Estado |
|---|---|---|
| **Requisitos y trazabilidad** | Requisitos (EP-01..EP-09, US-*, criterios EARS), glosario canónico, reglas BR-01..BR-22, topología SDD, `tasks.md`, trazabilidad, **entrega final del MD** (`docs/PROPUESTA_FINAL.md`) | **Entregado** (2026-10-21) |
| **Arquitectura y plataforma** | Parte de arquitectura/UX: **D-07 (autenticación)** y **D-08 (despliegue)**; el diseño de tableros lo entregó el área de UX y frontend | **Pendiente de confirmación del cliente**: entra como delta, no como reescritura (defaults en `docs/PROPUESTA_FINAL.md` §15) |
| **Dominio y agente** | Evidencia SDD + tooling, modelo de datos (I-1..I-13), diseño del agente (G1..G11, golden set E-01..E-12), modelo de carga y presión, dataset semilla (S-01..S-23), riesgos y validación, constitución, research de stack frontend con licencias verificadas, spec exemplar C11, tableros UX (X-01..X-13) | **Entregado** |

---

## Anexo · Glosario mínimo (canónico)

`Persona` · **Rol contractual base** (único, vitalicio) · **Rol de proyecto** (varios por persona) ·
`Asignación` (persona ↔ proyecto ↔ rol ↔ FTE ↔ fechas, tipo `PERMANENTE`/`TEMPORARIA`/`SUSTITUCION`/`BECARIO`) ·
`Carga estimada` (la demanda; sin ella no hay presión medible) · `Capacidad nominal` · `Utilización` ·
`Índice de presión` (deficit ÷ capacidad restante) · `Ausencia` (vacaciones / incapacidad) · `Cobertura` ·
`Préstamo temporal` (con fecha de devolución y piso de reserva) · `Propuesta del agente` (estado `EN_REVISION`) ·
`Pool de desasignados` (vista, no estado) · `Convenio becario` · `Bitácora`.

Detalle completo en `designs/0002-modelo-datos.md` §1 y en el glosario de la spec de requisitos.
