# Design 0002 — Modelo de datos

> Área responsable: dominio y agente · Estado: borrador · **Alineado con el glosario de la spec de requisitos**
> (`specs/0001-vision/spec.md` §3)
> Regla: este documento define **qué** existe y **qué no puede romperse**. El mapeo a colecciones (MongoDB) es de `plan.md`.

---

## 0. Alineación de vocabulario con la spec de requisitos

El glosario de la spec de requisitos es el que manda. Este documento usa **sus** términos; la equivalencia con el
borrador inicial queda registrada aquí para que nadie tenga que adivinar:

| Término canónico (spec de requisitos) | Equivalente previo | Nota |
|---|---|---|
| **Rol contractual base** | "rol contractual base" | Un solo rol vitalicio por persona |
| **Rol de proyecto** | "Rol de proyecto" | Varios por persona, incluso en el mismo proyecto |
| **Tipo de asignación** `PERMANENTE` / `TEMPORARIA` / `SUSTITUCION` / `BECARIO` | `BASE` / `PRESTAMO` / `COBERTURA` / `BECA` | Se adoptan los canónicos |
| **Capacidad nominal** | "Capacidad" | Horas/días disponibles sin descontar asignaciones |
| **Utilización** | "Utilización" | `carga_asignada / capacidad_nominal` |
| **Carga / esfuerzo estimado** | `DemandaProyecto` | Se materializa como `CargaEstimada` (§2.4b) |
| **Presión de proyecto** | "Presión" | Ver `designs/0005-modelo-carga-y-presion.md` |
| **Estado de proyecto** `EN_RIESGO` | `ROJO` | `EN_RIESGO` lo **propone** el agente y lo **confirma** un humano (BR-04/BR-06) |
| **Cobertura** | "Cobertura" | Asignación `SUSTITUCION` que termina con la ausencia |
| **Pool de desasignados** | "Personas desasignadas" | Vista, no estado persistido |
| **Bitácora / auditoría** | `BitacoraEvento` | `AuditLog` con `before`/`after` y `spec_ref` |

---

## 1. Glosario canónico (extracto normativo)

| Término | Definición operativa | No es |
|---|---|---|
| **Persona** | Ser humano con relación laboral ACTIVA (empleado, becario o contratista) con la fábrica de software. | Un recurso abstracto; no tiene proyectos propios. |
| **rol contractual base** | Rol con el que la persona fue contratada. Se define una sola vez y es de por vida. | Un rol por proyecto. |
| **Rol de proyecto** | Rol que la persona ejerce dentro de un proyecto concreto. Una persona puede tener varios, incluso simultáneos. | El rol contractual base. |
| **Asignación** | Vínculo temporal `[desde, hasta)` entre una Persona, un Proyecto y un Rol, con un `%` de su capacidad. | Una persona "en" un proyecto sin fechas. |
| **Préstamo temporal** | Asignación de tipo `PRESTAMO` con fecha de devolución obligatoria, sobre una persona ya asignada a su proyecto base. | Una asignación indefinida. |
| **Ausencia** | Periodo en que la persona no está disponible (vacaciones, incapacidad, permiso, licencia). Reduce capacidad; no es asignación. | Un proyecto sin personal. |
| **Capacidad** | Días disponibles de una persona en un periodo, después de descontar ausencias approved y días no laborables. | Un porcentaje abstracto. |
| **Utilización** | `Σ % de asignación ACTIVA / 100` para una persona en un periodo. | La carga del proyecto. |
| **Carga de proyecto** | `Σ (% asignación × capacidad de la persona)` en el periodo, normalizada contra la capacidad total del equipo. | Un conteo de personas. |
| **Propuesta** | Conjunto de asignaciones **sugeridas** por el agente, en estado `EN_REVISION`. | Una asignación activa. |
| **Cobertura** | Asignación temporal que compensa una ausencia, con un periodo que termina a más tardar al regreso. | Un préstamo indefinido. |

---

## 2. Entidades

### 2.1 `Persona`

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `nombre`, `email` | text | PII |
| `tipoContrato` | enum | `EMPLEADO` \| `BECARIO` \| `CONTRATISTA` |
| `rolContratualBaseId` | FK → `Rol` | Obligatorio. Verificado: no puede quedar nulo mientras la persona esté `ACTIVA`. |
| `nivelExperiencia` | enum | `JUNIOR` \| `MID` \| `SENIOR` \| `LEAD` \| `PRINCIPAL` |
| `fechaIngreso` | date | Alimenta antigüedad. |
| `fechaSalida` | date? | Null = ACTIVA. |
| `fechaNacimiento` | date | Solo día/mes se exponen en UI. |
| `diasLaborablesSemana` | decimal | Parte de la capacidad. |
| `horasDia` | decimal | |
| `estado` | enum | `ACTIVA` \| `INACTIVA`. `DESASIGNADA` **no es un estado persistido**: se deriva (§4.3). |
| `avatarUrl` | text? | |

Relaciones: `Competencia` (n:m, con nivel), `Herramienta` (n:m, con nivel/certificado), `ConvenioBecario` (0..1),
`SaldoVacaciones` (1..n anual), `Asignacion` (n), `Ausencia` (n), `LogroPersona` (n:m), `SolicitudVacaciones` (1..n).

### 2.1b `Logro` — catálogo de logros (gamificación)

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `codigo` | text | `ANIVERSARIO_1`, `ANIVERSARIO_3`, `ANIVERSARIO_5`, `ANIVERSARIO_10`, `ANIVERSARIO_15`, `ANIVERSARIO_20`, `PLANIFICADOR`, `PRIMERAS_VACACIONES`, `SIN_AUSENCIAS_ANO` |
| `nombre` | text | "Primer año", "3 años", "Quinquenio", "Década", "Quince años", "Veinte años", "Planificador", "Primeras vacaciones", "Año sin ausencias" |
| `descripcion` | text | Texto para mostrar en la ficha |
| `icono` | text | Nombre del icono (lucide-react) |
| `puntos` | int | Puntos que otorga (para ranking opcional) |
| `categoria` | enum | `ANTIGUEDAD` \| `PLANIFICACION` \| `BIENESTAR` \| `ESPECIAL` |

### 2.1c `LogroPersona` — logros desbloqueados por persona

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `personaId` | FK → `Persona` | |
| `logroId` | FK → `Logro` | |
| `fechaDesbloqueo` | date | |
| `contexto` | json? | Metadatos: años cumplidos, rango de vacaciones, etc. |

### 2.1d `SolicitudVacaciones` — autogestión de vacaciones

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `personaId` | FK → `Persona` | |
| `fechaDesde`, `fechaHasta` | date | `[desde, hasta)` |
| `diasSolicitados` | int | Derivado al aprobar (días laborables) |
| `estado` | enum | `SOLICITADA` \| `APROBADA` \| `RECHAZADA` \| `CANCELADA` \| `EN_CURSO` \| `FINALIZADA` |
| `coberturaSugeridaId` | FK? → `Asignacion` | Asignación de cobertura propuesta |
| `solicitadoEn` | datetime | |
| `aprobadoPor` | FK? → `Persona` | Quién aprobó (RRHH) |
| `aprobadoEn` | datetime? | |
| `puntosPlanificacion` | int | `1` si antelación ≥ 30 días, `0` en otro caso (BR-14) |

### 2.2 `Rol`

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `nombre` | text | "Desarrollador Backend", "QA Analyst", "Scrum Master", "Tech Lead", "Project Manager", "Product Owner"… |
| `familia` | enum | `ENGINEERING` \| `QA` \| `DELIVERY` \| `LEADERSHIP` \| `DATA` |
| `alcance` | enum | `GLOBAL` (catálogo organizacional) \| `PROYECTO` (creado ad-hoc en un proyecto) |
| `proyectoId` | FK? | Obligatorio si `alcance = PROYECTO`. |
| `factorCapacidad` | decimal | Multiplicador de capacidad efectiva (p. ej. un Manager dedica 50% a entrega) → afecta la carga. |
| `rolCritico` | bool | Rol que, si falta, bloquea el proyecto (p.ej. `Tech Lead`). |

### 2.3 `Proyecto`

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `codigo`, `nombre` | text | |
| `descripcion` | text | |
| `cliente` | text? | |
| `fechaInicio`, `fechaFinPrevista` | date | Definen la barra del Gantt. |
| `fechaFinReal` | date? | |
| `estado` | enum | `PROPUESTA` \| `PLANIFICACION` \| `ACTIVO` \| `EN_PAUSA` \| `CERRADO` |
| `capacidadDiariaEquipo` | decimal | Suma de capacidades del equipo planificado. |
| `demandaHoras` | decimal | Curva de demanda por fase (tabla `DemandaProyecto`). |

Derivado y **cacheado** (para no recalcular en cada render): `salud` (`VERDE`/`AMARILLO`/`ROJO`),
`cargaComprometidaPct`, `fechaUltimaEvaluacion`.

### 2.4 `Asignacion` — la entidad central

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `personaId` | FK | |
| `proyectoId` | FK | |
| `rolId` | FK | Rol **de proyecto** (puede ser `GLOBAL` reutilizado). |
| `tipo` | enum | `PERMANENTE` \| `TEMPORARIA` (préstamo) \| `SUSTITUCION` (cobertura) \| `BECARIO` — nomenclatura canónica |
| `porcentaje` | decimal(5,2) | FTE 0 < p ≤ 1 (se almacena como fracción de la capacidad nominal). |
| `fechaDesde`, `fechaHasta` | date | Semántica `[desde, hasta)`; `fechaHasta` obligatoria en `TEMPORARIA`, `SUSTITUCION` y `BECARIO`. |
| `estado` | enum | `PROPUESTA` \| `APROBADA` \| `ACTIVA` \| `SUSPENDIDA` (por ausencia, BR-15) \| `CERRADA` \| `RECHAZADA` |
| `origen` | enum | `HUMANO` \| `AGENTE` |
| `asignacionOrigenId` | FK? | Para `TEMPORARIA`: apunta a la asignación base que cede capacidad. |
| `proyectoOrigenId` | FK? | Desnormalizado para "volver a casa" sin join. |
| `fechaDevolucionPrevista` | date? | **Obligatoria** si `tipo = TEMPORARIA`; además ≤ fin del proyecto origen (BR-07). |
| `pisoReservaOrigen` | decimal? | % mínimo que el proyecto origen conserva mientras dura el préstamo (BR-08, 40 % por defecto). |
| `nota` | text? | |

Índices: `(personaId, fechaDesde, fechaHasta)`, `(proyectoId, estado)`, `(tipo, estado)`, único parcial
`(personaId, proyectoId, rolId)` entre asignaciones `ACTIVA` (BR-02 / I-6).

### 2.4b `CargaEstimada` — el insumo que hace posible medir la presión

Sin demanda registrada no hay presión que medir: la comparación es `demanda vs capacidad`.

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `proyectoId` | FK | |
| `rolId` | FK | Rol para el que se estima el esfuerzo |
| `periodoSemanaInicio` | date | Granularidad semanal |
| `esfuerzoHoras` | decimal | Horas de ese rol en ese periodo |
| `origen` | enum | `estimado` \| `real` |
| `registradoPor` | FK → `Persona` | Quién lo cargó |

Derivado: `cargaRestante(p, t) = max(0, Σ esfuerzo(estimado) − Σ esfuerzo(real))` → alimenta el `deficit`
de `designs/0005-modelo-carga-y-presion.md`.

### 2.4c `Hito` y `PersonaHistorial`

- `Hito(id, proyectoId, fecha, titulo, estado)` — hitos del proyecto: son los que dan forma al Gantt multi-zoom
  y los "hitos" que aparecen en la línea de tiempo individual (US-03.2).
- `PersonaHistorial(id, personaId, fecha, tipo, entidad, entidadId, detalle)` — **append-only**. Es la fuente de
  la "toda la historia en la empresa" que pide el cliente (altas, cambios de rol, ascensos, ausencias, préstamos).
  No se borra nunca; las asignaciones lo alimentan por evento.

### 2.5 `Ausencia`

| Campo | Tipo | Notas |
|---|---|---|
| `id` | uuid | |
| `personaId` | FK | |
| `tipo` | enum | `VACACIONES` \| `INCAPACIDAD` \| `PERMISO` \| `LICENCIA` \| `BAJA_MEDICA` |
| `fechaDesde`, `fechaHasta` | date | |
| `estado` | enum | `SOLICITADA` \| `APROBADA` \| `EN_CURSO` \| `FINALIZADA` \| `RECHAZADA` \| `CANCELADA` |
| `diasComputados` | int | Derivado al aprobar. |
| `motivo` | text? | Solo texto libre, **sin** datos clínicos. |
| `cubiertoPorAsignacionId` | FK? | Cobertura asociada. |
| `aprobadoPor` | FK → `Persona`? | Quién aprobó. |

### 2.6 `SaldoVacaciones`

`personaId`, `anio`, `diasAsignados`, `diasGozados`, `diasPendientes` (calculado), `cerrado` (bool), `cerradoPor`.

### 2.7 `Competencia` / `Habilidad` / `Herramienta`

- `Competencia(id, nombre, familia)` — p.ej. "Backend", "Testing", "Gestión de equipos", "Inglés técnico".
- `PersonaCompetencia(personaId, competenciaId, nivel 1..5, aniosExperiencia)`.
- `Herramienta(id, nombre, categoria)` — p.ej. "React", "PostgreSQL", "Jira", "Azure DevOps", "Playwright".
- `PersonaHerramienta(personaId, herramientaId, nivel 1..5, certificado bool)`.
  Requiere nivel: el ranking del agente necesita distinguir "usa React" de "es experto en React".

### 2.8 `ConvenioBecario`

`id`, `personaId` (1..1), `fechaInicio`, `fechaFin`, `duracionMeses`, `tutorId` (FK `Persona`), `horasSemanales`,
`universidad`, `modalidad` (`REMOTO`/`PRESENCIAL`/`MIXTO`), `estado` (`ACTIVA`/`FINALIZADO`/`RENOVADO`).

### 2.9 `Propuesta` y `PropuestaItem`

- `Propuesta(id, proyectoId, estado, creadaPorAgente, versionAgente, hashPayload, versionAprobada,
  resumenCargaAntes, resumenCargaDespues, fechaDisparador, createdAt, decidedBy, decidedAt, motivoDecision)`
  `hashPayload` fija el contenido exacto de la propuesta: se aprueba y se aplica **ese** payload, no otro (M1 en
  `designs/0004` §4). `versionAprobada` es la versión que el humano aprobó cuando editó la propuesta antes de
  aprobar, y `BitacoraEvento` guarda la diferencia con la versión del agente (US-04.6).
- `PropuestaItem(id, propuestaId, personaId, rolId, porcentaje, fechaDesde, fechaHasta, tipo,
  puntaje 0..100, razon, alternativasDescartadas json,)`
  `alternativasDescartadas` guarda **por qué no** se eligió a otro candidato: es el requisito de explicabilidad del agente.

### 2.10 `BitacoraEvento` (auditoría)

`id`, `occurredAt`, `actorTipo` (`AGENTE`/`HUMANO`/`SISTEMA`), `actorId`, `tipoEvento`, `entidad`, `entidadId`,
`payloadAntes` json, `payloadDespues` json, `justificacion`, `correlationId`, `confianza` (0..1),
`snapshotEntradas` json (capacidades y reglas usadas por el agente en ese momento).

### 2.11 Calendario y capacidad

- `Feriado(fecha, nombre, ambito: GLOBAL|PAIS|CIUDAD, pais?, ciudad?)` — necesario para el zoom por semana/mes/año.
- `CapacidadPersonaPeriodo(personaId, periodo, diasLaborables, diasAusentes, diasDisponibles)` — materializado por el worker.

---

## 3. Semántica temporal (la regla que más bugs genera)

- Todos los periodos son **`[desde, hasta)`**: una asignación que termina el 30/06 y otra que empieza el 30/06 **no se solapan**.
- Fechas en UTC; el "día" de negocio se define en la zona del calendario laboral, no del servidor.
- Un cambio de porcentaje en una asignación viva **no reescribe la historia**: se cierra la asignación con
  `fechaHasta` y se abre otra. El histórico es inmutable y auditable.
- Historial: no se usa *soft delete* en `Asignacion`, `Ausencia` ni `Propuesta`. `estado` + `BITACORA` bastan.

---

## 4. Cálculos derivados

### 4.1 Utilización de una persona

```
utilización(persona, [d1, d2)) = Σ porcentaje(asignaciones ACTIVAS que intersectan [d1,d2)) / 100
```

### 4.2 Capacidad y carga de un proyecto

```
capacidadEquipo(proyecto, periodo) = Σ capacidad(p) de personas asignadas (su peso) × factorCapacidad(rol)
cargaProyecto(proyecto, periodo)    = Σ (porcentaje/100) × capacidad(p, periodo) × factorCapacidad(rol)
presiónProyecto = cargaProyecto / capacidadEquipo
```

Bandas por defecto (umbrales configurables, ver `specs/0001-vision/research-sdd-y-dominio.md` §2.2):
`VERDE < 70% ≤ AMARILLO < 85% ≤ ROJO`.

### 4.3 Persona "desasignada" (disponible)

Una persona `ACTIVA` sin asignaciones `ACTIVA` que intersecten el periodo **y** sin ausencias que la bloqueen.
No es un estado: es una consulta. Esto evita que "disponible" y "asignada" se desincronicen.

### 4.4 Antigüedad, cumpleaños, vacaciones

- `antiguedad = fechaActual - fechaIngreso` (años/meses/días), visible en la ficha de persona.
- `cumpleaños = fechaNacimiento` (día/mes) — se alimenta un panel de "cumpleaños del mes".
- `vacaciones`: saldo anual (`SaldoVacaciones`) + ausencias de tipo `VACACIONES` en curso/próximas.

---

## 5. Invariantes (lo que el sistema NO puede violar)

> Implementadas como **restricciones de dominio** (servicio + motor determinista en Python) + **tests** + **reconciliador
> nocturno**. El LLM nunca las evalúa (§ Constitution IV).

**Por qué no "check en BD":** MongoDB no tiene `CHECK`, ni tipos de rango, ni `EXCLUDE USING GIST`. La regla dura se
impone en **tres puntos de enforcement**, y los tres deben poder *rechazar*:

| # | Enforcement | Qué hace | Cuándo corre |
|---|---|---|---|
| **E1** | **Servicio de dominio** (`fastapi/services/assignment_service.py`) | toda escritura de `Asignacion`, `Ausencia` o préstamo pasa por el motor determinista, que valida I-1..I-13 **antes** del `insert`/`update` | sincrónico, en cada request |
| **E2** | **Guardarraíl de tool del agente** (`agent/guardrails.py`) | ninguna propuesta del LLM llega a `E1` sin volver a validarse | sincrónico, por tool call |
| **E3** | **Reconciliador nocturno** (`fastapi/worker/reconciler.py`) | escanea las colecciones buscando violaciones y abre `Alerta` + `BitacoraEvento` | programado (08:00 local) |

> `E3` es la red que compensa la ausencia de constraints: si algo se coló por un `upsert` manual, por una importación
> o por un bug, la violación aparece en menos de 24 h en lugar de vivir meses en el dataset.

Lo que MongoDB **sí** puede imponer por su cuenta:

| Mecanismo | MongoDB | Uso |
|---|---|---|
| Índice único parcial | `unique: true` + `partialFilterExpression: {estado: "ACTIVA"}` | I-6: no repetir `(personaId, proyectoId, rolId)` en una asignación `ACTIVA` |
| Índice compuesto | `{personaId: 1, fechaDesde: 1, fechaHasta: 1}` · `{proyectoId: 1, fechaDesde: 1, fechaHasta: 1}` | consultas de solapamiento del motor y del timeline |
| Clave única de idempotencia | `unique: true` sobre `idempotencyKey` | G9: una propuesta por `(proyecto, rol, ventana, candidato)` |
| Validador de esquema (JSON Schema) | `collMod` / `$jsonSchema` | forma de los documentos; **no** expresses rangos ni solapamiento |

| # | Invariante | Verificación | Ref. spec |
|---|---|---|---|
| I-1 | `Σ porcentaje` de una persona en un periodo ≤ 100 % (capacidad nominal) | E1 (motor: `capacidadLibre` < 0 ⇒ rechaza) + E3 + test de dominio | BR-01 |
| I-2 | Un préstamo temporal tiene `fechaDevolucionPrevista` obligatoria, `asignacionOrigenId` válida y `fechaDevolucionPrevista ≤ fin del proyecto origen` | E1: validación de esquema (Pydantic) + regla de negocio del préstamo | BR-07 |
| I-3 | Mientras un préstamo está `ACTIVO`, la asignación origen queda **cedida** en el % prestado | E1: el alta del préstamo descuenta el % del origen en la misma transacción + E3 | BR-09, BR-10 |
| I-4 | `proyectoOrigen.porcentaje + Σ préstamos ACTIVAs ≤ 100 %` (la persona no puede estar más de su capacidad) | derivado de I-1 + I-3 (mismo motor) | BR-01 |
| I-5 | Una cobertura termina **a más tardar** el regreso de la ausencia | E1: validación cruzada `Ausencia` ↔ `Asignacion` + E3 | BR-13 |
| I-6 | No hay dos asignaciones `ACTIVA` con el mismo `(personaId, proyectoId, rolId)` | **índice único parcial** de Mongo (`partialFilterExpression`) + E3 | BR-02 |
| I-7 | Un becario no puede tener asignación fuera de su `ConvenioBecario.fechaInicio..fechaFin` y nunca es candidato a préstamo sin aprobación de RRHH | E1: validación de rango del convenio + filtro duro del motor de candidatos | BR-11, BR-16 |
| I-8 | Una asignación `ACTIVA` no puede referenciar un proyecto `CERRADO` | E1: el servicio valida el estado del proyecto + E3 | — |
| I-9 | Un proyecto en `ACTIVO` no puede quedar sin `Tech Lead` asignado más de N días | regla del agente + alerta (no bloquea) | — |
| I-10 | Toda mutación relevante deja registro en `BitacoraEvento` (`AuditLog`) con actor, antes/después, `spec_ref` y decisión humana | E1: la escritura y su traza van en la misma transacción + test de integración | BR-19 |
| I-11 | **Piso de reserva**: un préstamo no puede dejar al proyecto origen por debajo de `pisoReserva` (40 % por defecto) de la asignación original del prestatario | filtro duro del motor de candidatos | BR-08, BR-12 |
| I-12 | El agente **no** puede usar atributos sensibles (género, edad, origen, estado civil, motivo clínico) para puntuar | el motor solo recibe `skills, disponibilidad, rol, carga, historial, ausencias`; test que verifica las entradas del scorer | BR-20 |
| I-13 | Al iniciar una incapacidad, las asignaciones activas de esa persona pasan a `SUSPENDIDA` y las de cobertura a `ACTIVA` | E1: transición de `Ausencia` en el servicio + E3 | BR-15 |

---

## 6. Diagrama (resumen)

```
Persona ─┬─(1..n) Asignacion ─(n..1) Proyecto ─(1..n) Rol
         ├─(1..n) Ausencia ──────────────┐
         ├─(1..n) Competencia / Herramienta (n:m, con nivel)
         ├─(1..1) ConvenioBecario
         └─(1..n) SaldoVacaciones

Asignacion ─(tipo=PRESTAMO|COBERTURA)─► Asignacion (origen)
Propuesta ─(1..n) PropuestaItem ─► Persona / Rol / Proyecto
BitacoraEvento  (auditoría de todo actor: humano, agente, sistema)
```

`Rol` es catálogo global o de proyecto; `Proyecto` usa `Rol` y `Persona` usa `Rol` como rol contractual base.

---

## 7. Preguntas abiertas para el humano (no bloqueantes; se avanza con supuestos)

1. **Umbral 75 %**: ¿presión por proyecto, de portafolio, o ambas? (ver `specs/0001-vision/research-sdd-y-dominio.md` §2.2)
2. **Factor de capacidad por rol**: ¿un Manager/PM cuenta como 50 % de su tiempo en el proyecto o 100 %? Afecta toda la carga.
3. **Capacidad base**: ¿21 días laborables/mes × 8 h? ¿Turnos 4×10 o 5×8? (necesario para que las cifras del tablero signifiquen algo)
4. **Becarios**: ¿el doble proyecto está permitido? El modelo lo permite; ¿la política lo prohíbe?
5. **PII**: ¿fecha de nacimiento exacta se puede guardar, o solo día/mes?
