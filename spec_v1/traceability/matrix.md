# TRACEABILITY MATRIX — requisito → regla → invariante → diseño → tarea → prueba

> Área responsable: requisitos y trazabilidad · Última actualización: 2026-10-21
> Fuentes: `specs/000{2..6}-*/requirements.md` · `designs/0002-modelo-datos.md` (invariantes I-1..I-13) ·
> `designs/0004-agente-coordinador.md` (guardarraíles G1..G11) ·
> `designs/0005-modelo-carga-y-presion.md` · `tasks/0001-tareas.md`

**Estados:** `ESPECIFICADO` = requisito con criterio EARS y tarea asignada, sin implementar.
Nada pasa a `VERIFICADO` sin un test que falle si el criterio se incumple.

---

## 1. Matriz principal

| US | Título corto | BR-* | I-* | Diseño | Tarea | Prueba prevista | Estado |
|---|---|---|---|---|---|---|---|
| **US-01.1** | Registrar persona | BR-01, BR-16 | I-1 | 0002 | T-001, T-002, T-101 | `test_alta_persona.py` | ESPECIFICADO |
| **US-01.2** | Ficha 360 | BR-01, BR-19 | I-10 | 0002, 0003 | T-105 | `test_ficha_360.py` | ESPECIFICADO |
| **US-01.3** | Registrar y cubrir ausencias | BR-03, BR-13, BR-15 | I-5, I-13 | 0002 | T-103, T-104 | `test_ausencias.py` | ESPECIFICADO |
| **US-01.4** | Pool de desasignados | BR-01 | I-1 | 0002, 0003 | T-303 | `test_pool_desasignados.py` | ESPECIFICADO |
| **US-01.5** | Gestión de becarios | BR-11, BR-16, BR-17 | I-7 | 0002 | T-205, T-604 | `test_becarios.py` | ESPECIFICADO |
| **US-01.6** | Historial append-only | BR-19 | I-10 | 0002 | T-601 | `test_historial.py` | ESPECIFICADO |
| **US-01.7** | Solicitar vacaciones con reconocimiento de aniversario | BR-13, BR-14, BR-19, BR-20 | I-5, I-10 | 0002, 0003 | T-106, T-107, T-306 | `test_vacaciones_autogestion.py`, `test_aniversario_reconocimiento.py`, `test_gamificacion_logros.py` | ESPECIFICADO |
| **US-02.1** | Crear proyecto | BR-06, BR-18 | — | 0003 | T-201, T-504 | `test_crear_proyecto.py` | ESPECIFICADO |
| **US-02.2** | Card de proyecto | BR-05 | — | 0003 | T-204 | `e2e/card_proyecto.spec.ts` | ESPECIFICADO |
| **US-02.3** | Asignar personas y roles | BR-01, BR-02 | I-1, I-6 | 0002 | T-102, T-202 | `test_asignaciones.py` | ESPECIFICADO |
| **US-02.4** | Timeline con zoom | — | — | 0001, 0003 | T-203 | `e2e/zoom_timeline.spec.ts` | ESPECIFICADO |
| **US-02.5** | Organigrama del proyecto | BR-17 | I-7 | 0003 | T-205 | `e2e/organigrama.spec.ts` | ESPECIFICADO |
| **US-02.6** | Semáforo de presión | BR-04, BR-05, BR-06 | — | 0005 | T-402, T-505 | `test_indice_presion.py` | ESPECIFICADO |
| **US-02.7** | Carga de trabajo registrada | BR-04, BR-05 | — | 0005 | T-206, T-401 | `test_carga.py` | ESPECIFICADO |
| **US-03.1** | Listado con filtros | BR-20 | I-12 | 0003 | T-003, T-301 | `e2e/listado_personas.spec.ts` | ESPECIFICADO |
| **US-03.2** | Ficha 360 (panel) | BR-20 | I-12 | 0003 | T-105, T-603 | `e2e/ficha_360.spec.ts` | ESPECIFICADO |
| **US-03.3** | Timeline individual | — | — | 0003 | T-302 | `e2e/timeline_persona.spec.ts` | ESPECIFICADO |
| **US-03.4** | Pool de desasignados | BR-18 | — | 0003, 0004 | T-303 | `e2e/pool.spec.ts` | ESPECIFICADO |
| **US-03.5** | Comparación de personas | BR-20 | I-12 | 0003 | T-304 | `e2e/comparacion.spec.ts` | ESPECIFICADO |
| **US-03.6** | Timeline compartido | — | — | 0001, 0003 | T-305 | `e2e/zoom_compartido.spec.ts` | ESPECIFICADO |
| **US-03.7** | Portal de vacaciones y reconocimiento de aniversario | BR-13, BR-14, BR-19, BR-20 | I-5, I-10 | 0002, 0003 | T-106, T-107, T-306 | `test_vacaciones_autogestion.py`, `test_aniversario_reconocimiento.py`, `test_gamificacion_logros.py` | ESPECIFICADO |
| **US-04.1** | Propuesta al crear proyecto | BR-06, BR-18, BR-20 | I-12 | 0004 | T-502, T-504 | `test_propuesta_inicial.py` | ESPECIFICADO |
| **US-04.2** | Monitoreo de presión | BR-04, BR-06 | — | 0004, 0005 | T-505 | `test_alerta_presion.py` | ESPECIFICADO |
| **US-04.3** | Propuesta de préstamo temporal | BR-07, BR-08, BR-11, BR-12 | I-2, I-7, I-11 | 0004 | T-506 | `test_prestamo_temporal.py` | ESPECIFICADO |
| **US-04.4** | Respeto de la asignación temporal | BR-09, BR-10 | I-3, I-4 | 0004 | T-403, T-404, T-507 | `test_prestamo_respetado.py` | ESPECIFICADO |
| **US-04.5** | Cobertura de ausencias | BR-13, BR-15 | I-5, I-13 | 0004 | T-104, T-508 | `test_cobertura.py` | ESPECIFICADO |
| **US-04.6** | Propuestas y aprobaciones | BR-18 | — | 0004 | T-503, T-510 | `test_aprobacion.py` + `test_commit_rejects_stale_version` + `test_commit_revalidates_invariants` + `test_commit_idempotent` | ESPECIFICADO |
| **US-04.7** | Explicabilidad y auditoría | BR-19, BR-20 | I-10, I-12 | 0004 | T-501, T-509, T-601, T-603 | `test_auditoria.py` | ESPECIFICADO |
| **US-04.8** | Alertas y avisos | BR-19 | I-10 | 0004 | T-604 | `test_avisos.py` | ESPECIFICADO |
| **US-05.1** | Artefactos versionados | — | — | — | T-606 | `test_traces.py` | ESPECIFICADO |
| **US-05.2** | Trazabilidad requisito→prueba | — | — | — | T-606 | `test_traces.py` | ESPECIFICADO |
| **US-05.3** | Criterios ejecutables | — | — | — | T-606 | `test_criterios.py` | ESPECIFICADO |
| **US-05.4** | Bandeja de aprobaciones | BR-18 | — | 0003 | T-510 | `e2e/bandeja.spec.ts` | ESPECIFICADO |
| **US-05.5** | Roles y control de acceso | BR-20 | I-12 | 0002 | T-602 | `test_acceso.py` | ESPECIFICADO |
| **US-05.6** | Auditoría inmutable | BR-19 | I-10 | 0002 | T-601 | `test_auditoria_inmutable.py` | ESPECIFICADO |
| **US-05.7** | Modo demostración | BR-01..BR-22 | I-1..I-13 | 0001 | T-004, T-605 | `test_seed_determinista.py` | ESPECIFICADO |

**Totales: 36 historias · 36 con criterio EARS · 36 con tarea · 0 sin cobertura · 43 tareas trazadas
(T-001..T-606 + T-106, T-107, T-306) · 0 tareas sin fila.**

> **Convención de la columna "Tarea"**: la lista es el conjunto de tareas que **implementan** el US, no solo la
> tarea principal. `tasks/0001-tareas.md` declara el **US primario** de cada tarea; una tarea puede atender
> varias historias (p. ej. `T-601` —auditoría inmutable— tiene su US primario en `US-05.6` y también sostiene
> `US-01.6` y `US-04.7`). Lo que **no** se permite es que la tarea de un US no aparezca en su fila: el gate
> `traceability:check` (§4) falla si una tarea no tiene US, y esta matriz falla si una fila pierde la tarea que
> la implementa.

---

## 2. Cobertura de reglas de negocio (BR-*)

| BR | Regla | US que la exigen | I-* |
|---|---|---|---|
| BR-01 | Σ FTE ≤ capacidad nominal | US-01.1, US-02.3, US-03.1 | I-1, I-4 |
| BR-02 | No 2 roles idénticos en el mismo proyecto | US-02.3 | I-6 |
| BR-03 | No asignar en ausencia sin cobertura | US-01.3 | — |
| BR-04 | `avance ≥ umbral` **y** déficit ⇒ presión | US-02.6, US-02.7, US-04.2 | — |
| BR-05 | Capacidad disponible = equipo − carga − reservas | US-02.2, US-02.6, US-02.7 | — |
| BR-06 | El agente propone; el humano confirma | US-02.1, US-02.6, US-04.1, US-04.2 | — |
| BR-07 | Préstamo con `fecha_fin` obligatoria | US-04.3 | I-2 |
| BR-08 | Piso de reserva (40 %) del proyecto origen | US-04.3 | I-11 |
| BR-09 | Préstamo vigente bloquea consumir la capacidad reservada | US-04.4 | I-3 |
| BR-10 | Al vencer, la capacidad se reembolsa al origen | US-04.4 | I-3, I-4 |
| BR-11 | Becarios no son candidatos a préstamo sin RRHH | US-01.5, US-04.3 | I-7 |
| BR-12 | Un préstamo nunca deja al origen bajo su piso | US-04.3 | I-11 |
| BR-13 | Vacaciones consumen saldo; incapacidad no | US-01.3, US-04.5 | I-5 |
| BR-14 | Anticipación mínima de vacaciones (configurable) | US-01.3 | — |
| BR-15 | Incapacidad suspende asignaciones y activa cobertura | US-01.3, US-04.5 | I-13 |
| BR-16 | Periodo acotado del becario | US-01.1, US-01.5 | I-7 |
| BR-17 | Todo becario tiene mentor | US-01.5, US-02.5 | I-7 |
| BR-18 | Ninguna propuesta se aplica sin aprobación humana | US-02.1, US-03.4, US-04.1, US-04.6, US-05.4 | — |
| BR-19 | Auditoría de toda entrada del agente | US-01.6, US-01.7, US-03.7, US-04.7, US-04.8, US-05.6 | I-10 |
| BR-20 | Prohibido puntuar con atributos sensibles | US-03.1, US-03.2, US-03.5, US-04.1, US-04.7, US-05.5 | I-12 |
| **BR-21** | **Gamificación no afecta scoring del agente** | US-01.7, US-03.7 | — |
| **BR-22** | **Aniversario automático en día de ingreso** | US-01.7, US-03.7 | I-10 |

**Ninguna regla huérfana:** las 22 reglas BR-* están exigidas por al menos un US, y cada US
declara sus reglas aplicables en el pie de su spec.

---

## 3. Gap conocido (honesto)

| # | Gap | Por qué | Mitigación |
|---|---|---|---|
| G-1 | **I-8** (no asignar en proyecto `CERRADO`) y **I-9** (proyecto activo sin `Tech Lead` más de N días) no tienen US propio | son reglas transversales, no funcionalidades | se verifican en `test_invariantes.py` (I-1..I-13) sobre el seed y en cada cambio de estado |
| G-2 | **BR-14** (anticipación de vacaciones) queda como validación configurable sin criterio EARS propio | el cliente no la especificó | si el cliente la confirma, se agrega un criterio a US-01.3 en el mismo PR que la active |
| G-3 | US-04.3 exige `REQUIERE_REVISIÓN_DE_CONFIDENCIALIDAD` para préstamos entre clientes distintos | el cliente no respondió si se permiten | el estado existe y bloquea la aprobación automática hasta que alguien con rol `COORDINACION` lo levante |

---

## 4. Regla de validación en CI

El gate `traceability:check` falla si:

1. Un `US-*` no tiene al menos un criterio en EARS.
2. Un `US-*` no tiene tarea asignada.
3. Una tarea no referencia ningún `US-*`.
4. Una `BR-*` no aparece en el pie de ninguna spec.
5. Un `I-*` no aparece en la matriz.
6. Un requisito marcado `VERIFICADO` no tiene test asociado en el repositorio.

> Regla de la constitution: *un gate solo es control si puede rechazar*. Este gate puede
> rechazar, por lo tanto es un control.
