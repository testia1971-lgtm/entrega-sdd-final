# TASKS 0001 — Plan de trabajo por sprints

Área responsable: requisitos y trazabilidad · Trazabilidad: [`../traceability/matrix.md`](../traceability/matrix.md)

> Regla: cada tarea referencia **un** `US-*`, cabe en un PR y tiene un criterio de
> aceptación verificable. El agente de código toma tareas de aquí.

## Sprint 0 — Cimientos

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-001 | Crear las colecciones de MongoDB con sus índices | US-01.1 | Las colecciones de `designs/0002` existen con los índices del design: índice único parcial de `(personaId, proyectoId, rolId)` sobre `estado: "ACTIVA"` (I-6), compuestos `{personaId, fechaDesde, fechaHasta}` y `{proyectoId, fechaDesde, fechaHasta}`, y clave única de `idempotencyKey` (G9) |
| T-002 | Modelos Pydantic y repositorio de `Persona`, `Rol`, `Habilidad` | US-01.1 | Alta con rol contractual base, skills y fecha de ingreso; test del modelo; el mismo validador Pydantic sirve para la API y para el `output_type` del agente |
| T-003 | API de lectura de personas con filtros | US-03.1 | Filtro por rol, seniority, skill, proyecto, disponibilidad y búsqueda libre |
| T-004 | Cargar el seed mínimo | US-05.7 | `python -m scripts.seed` deja la base poblada, es idempotente y con la misma semilla produce la misma salida |

## Sprint 1 — Personas y ausencias

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-101 | Alta y edición de personas | US-01.1 | El formulario exige rol base, fecha de ingreso y fecha de alta |
| T-102 | Motor de capacidad y validación BR-01 | US-02.3 | No se puede asignar por encima del 100 % sin justificación auditada |
| T-103 | Registro de ausencias con reglas de saldo | US-01.3 | Incapacidad no consume saldo y exige cobertura; vacaciones sí consumen |
| T-104 | Cobertura: activación y liberación | US-04.5 | Al iniciar la incapacidad las asignaciones quedan `SUSPENDIDA` y la cobertura `ACTIVA` |
| T-105 | Ficha 360: endpoint y layout | US-01.2 | Un endpoint devuelve antigüedad, skills, asignaciones, historial, vacaciones y cumpleaños |
| T-106 | Autogestión de vacaciones: endpoint + validación de saldo + cobertura sugerida | US-01.7 | La persona solicita rango, se valida saldo, se sugiere cobertura (BR-13/14), estado `SOLICITADA` |
| T-107 | Reconocimiento de aniversario y logros (gamificación) | US-01.7 | En día de ingreso muestra banner con años, badge `ANIVERSARIO_N`, mensaje; logro queda en `LogroPersona` |

## Sprint 2 — Proyectos, roles y timeline

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-201 | Alta de proyecto con plantilla de roles | US-02.1 | La plantilla exige esfuerzo semanal por rol |
| T-202 | Asignaciones con roles múltiples | US-02.3 | Se aceptan varios roles distintos por persona y proyecto; el duplicado se rechaza |
| T-203 | Timeline con zoom (semana/quincena/mes/año) | US-02.4 | Cambiar el zoom reescala hitos y asignaciones sin perder información |
| T-204 | Card de proyecto | US-02.2 | Muestra fechas, avance, salud, carga vs. capacidad, roles cubiertos y faltantes |
| T-205 | Organigrama derivado de roles de proyecto | US-02.5 | Muestra personas, vacantes, becarios y mentores; el rol contractual se distingue |
| T-206 | Registro de carga estimada y real | US-02.7 | La curva demanda vs. capacidad se alimenta de lo registrado |

## Sprint 3 — Tablero de personal

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-301 | Listado con filtros y ordenación | US-03.1 | Filtros combinables y ordenación sin perder filtros |
| T-302 | Timeline individual alineado | US-03.3 | Comparte escala con el tablero de proyectos |
| T-303 | Pool de desasignados | US-03.4 | Muestra capacidad libre y distingue "en propuesta" de "asignado" |
| T-304 | Comparación de personas | US-03.5 | Dos personas alineadas con solapamientos visibles |
| T-305 | Estado de zoom compartido | US-03.6 | Cambiar de tablero conserva zoom, rango y filtro |
| T-306 | Portal de vacaciones y reconocimiento de aniversario (UI) | US-03.7 | Portal muestra saldo + botón solicitar; aniversario muestra banner + logro; ≥30 días muestra Planificador |

## Sprint 4 — Motor de carga y presión

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-401 | Cálculo de capacidad y utilización | US-02.7 | Ventanas 4/8/12 semanas correctas |
| T-402 | Índice de presión y disparador del 75 % | US-02.6 | Un proyecto al 78 % con déficit se marca; uno al 80 % sin déficit no |
| T-403 | Proyecciones con préstamos y ausencias | US-04.4 | La proyección del origen respeta la fecha de fin del préstamo |
| T-404 | Job de expiración de préstamos | US-04.4 | Al vencer, se reembolsa la capacidad y se notifica |

## Sprint 5 — Agente coordinador

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-501 | Catálogo de tools de lectura | US-04.7 | Cada tool devuelve datos con su origen auditable |
| T-502 | Capa de scoring determinista | US-04.1 | El score se calcula en código, con desglose por componente; test que verifica que no usa atributos sensibles (BR-20) |
| T-503 | `create_proposal` + estados | US-04.6 | La propuesta nace `EN_REVISION`; el agente no escribe asignaciones (G2) |
| T-504 | Propuesta al crear proyecto | US-04.1 | Genera candidatos por rol con score, justificación, alternativas e impacto |
| T-505 | Propuesta de alivio por presión | US-04.2 | La alerta trae diagnóstico, causa probable, opciones y confianza |
| T-506 | Propuesta de préstamo temporal | US-04.3 | Filtra por piso de reserva, ausencia, becario y conflicto de clientes |
| T-507 | Bloqueo y devolución de préstamo | US-04.4 | La capacidad reservada no se puede consumir; al vencer se reembolsa |
| T-508 | Cobertura de ausencias | US-04.5 | Incapacidad ⇒ propuesta de sustitutos calificados |
| T-509 | Validación post-ejecución (G6) | US-04.7 | Toda cifra de la propuesta aparece en la salida de las tools |
| T-510 | Bandeja de aprobaciones | US-05.4 | Aprobar aplica en transacción; rechazar guarda motivo; editar guarda el diff |

## Sprint 6 — Gobernanza, auditoría y demo

| ID | Tarea | Requisito | Criterio de aceptación |
|---|---|---|---|
| T-601 | Auditoría inmutable | US-05.6 | No se puede modificar ni borrar un registro |
| T-602 | Roles y control de acceso | US-05.5 | `CONSULTA` no ve incapacidades; `RRHH` sí |
| T-603 | Explicabilidad en la UI | US-04.7 | Se ve el desglose del score y la evidencia citada |
| T-604 | Avisos de fin de ausencia/préstamo y cumpleaños | US-04.8 | Se emiten con la antelación configurada |
| T-605 | Dataset semilla determinista | US-05.7 | Los **23 escenarios S-01..S-23** de `docs/DATASET_SEED.md` §3 reproducen el comportamiento esperado; `python -m scripts.verify_seed` falla si falta alguno o si el seed viola I-1..I-8 |
| T-606 | Validación de trazabilidad en CI | US-05.2 | `python -m scripts.traceability_check` falla si hay requisito sin EARS, criterio sin test o tarea sin requisito |

## Dependencias

- Sprint 0 bloquea a todos.
- T-401/T-402 bloquean a T-505/T-506 (el agente razona sobre presión calculada).
- T-502 bloquea a T-503 (la propuesta se construye sobre el score).
- T-507 depende de T-402 y T-403.

## Preguntas abiertas (afectan al desglose)

1. **D-07 / D-08** (auth y despliegue): siguen abiertas, pendientes de confirmación del cliente; el default propuesto
   es **JWT con 3 roles** y **aplicación web con backend locales, sin Docker** (ver `docs/PROPUESTA_FINAL.md` §15).
   Mientras siga abierto, el desglose asume ese default.
2. ¿El job de presión corre dentro del backend (APScheduler) o como worker con cola? *Default tomado:*
   `fastapi/worker/` con `reconciler.py` programable, idempotente por `jobRunId`.
3. ¿Cuántos módulos entran en la demo del curso? Propuesta: sprints 0–5, con el 6 como "endurecimiento".

> **Comandos de referencia** (backend Python, frontend TS): `pytest`, `ruff`, `mypy`, `playwright`,
> `python -m scripts.seed`, `python -m scripts.verify_seed`, `python -m scripts.traceability_check`.
> Ver `AGENTS.md` §5.
