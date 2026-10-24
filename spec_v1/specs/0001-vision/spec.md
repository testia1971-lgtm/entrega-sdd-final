# SPEC 0001 — Visión del producto

Estado: **aprobado por el cliente** (2026-10-20) · Área responsable: requisitos y trazabilidad · Épicas: `EP-01`…`EP-05`

## 1. Problema

Una fábrica de software reparte personas entre proyectos con planillas dispersas. En la
práctica eso produce cuatro fallos:

1. **No se sabe con quién cuenta cada proyecto** ni con qué rol.
2. **La presión se descubre tarde**: cuando el avance ya está al 80 % y el equipo va ahogado.
3. **Las ausencias (vacaciones, incapacidad) se cubren a ojo**, o no se cubren.
4. **El talento ocioso no es visible**: hay personas sin proyecto que nadie ubica.

## 2. Objetivo del producto

> Dar una **fuente única de verdad** sobre la asignación de personas a proyectos, y un
> **agente coordinador** que detecte la presión a tiempo (por defecto al 75 % del avance) y
> proponga reasignaciones **temporales y respetuosas** con los compromisos existentes.

### 2.1 Objetivos medibles

| ID | Objetivo | Métrica de éxito |
|---|---|---|
| O-01 | Detectar la presión antes del incidente | Acción correctiva < 48 h desde la señal |
| O-02 | No romper compromisos de asignación | 100 % de préstamos temporales con fecha de fin y piso de reserva respetado |
| O-03 | Eliminar la planilla | 100 % de asignaciones vigentes en el sistema |
| O-04 | Cobertura automática de ausencias | 100 % de incapacidades con cobertura propuesta < 24 h |
| O-05 | Decisión humana siempre | 0 acciones del agente aplicadas sin aprobación humana |

### 2.2 Fuera de alcance (v1)

- Reemplazar un ATS/HRIS corporativo (se integra por API más adelante).
- Nómina y liquidaciones.
- Evaluación de desempeño individual y planes de carrera (riesgo legal: ver §9).
- Multi-tenancy y control de acceso fino por cliente.
- Facturación al cliente.

## 3. Usuarios y Needs

| Usuario | Necesidad principal |
|---|---|
| **PMO / Coordinación** | Ver la carga de todos los proyectos y actuar antes de que se quiebren. |
| **Manager de proyecto** | Saber con qué equipo cuento, qué roles faltan y quién está de vacaciones. |
| **RRHH** | Fichas de personas, antigüedad, vacaciones, incapacidades, becarios y su cobertura. |
| **Persona (empleada)** | Saber en qué proyectos estoy, con qué rol y cuándo me reincorporo. |
| **Agente coordinador (IA)** | Operar sobre datos verificados y proponer con evidencia. |

## 4. Glosario

Ver el **glosario canónico** en [`../../designs/0002-modelo-datos.md`](../../designs/0002-modelo-datos.md) §1
(copia extendida en [`../../designs/0002-modelo-datos/plan.md`](../../designs/0002-modelo-datos/plan.md)).
Resumen operativo (el resto de las specs deben usar **exactamente** estos
términos):

| Término | Significado en el sistema |
|---|---|
| **Persona** | Humano con contrato vigente. Entidad raíz. |
| **Rol contractual base** | Rol con el que fue contratado. Define banda y seniority. No cambia por proyecto. |
| **Rol de proyecto** | Rol que desempeña dentro de un proyecto. Varios por persona y proyecto. Par *(persona, proyecto, rol)* único. |
| **Asignación** | Vínculo persona ↔ proyecto con rol, FTE, fechas y tipo. |
| **Tipo de asignación** | `PERMANENTE` · `TEMPORARIA` (préstamo) · `SUSTITUCION` (cobertura) · `BECARIO`. |
| **Capacidad nominal** | Tiempo disponible de la persona (p. ej. 8 h/día × días laborables). |
| **Utilización** | `carga asignada / capacidad nominal`. |
| **Presión de proyecto** | Indicador 0–100 de esfuerzo vs. capacidad vs. avance. |
| **Ausencia** | `VACACIONES` · `INCAPACIDAD` · `PERMISO` · `BAJA`. |
| **Cobertura** | Asignación `SUSTITUCION` que cubre una ausencia. |
| **Préstamo temporal** | Asignación `TEMPORARIA` acotada en el tiempo, con `proyecto_origen`. |
| **Piso de reserva** | % mínimo de la asignación original que el proyecto origen conserva durante un préstamo. |
| **Pool de desasignados** | Personas sin asignación activa, listas para asignar. |
| **Becario** | Contrato de aprendizaje con periodo acotado y mentor. |
| **Propuesta del agente** | Documento con sugerencia + justificación + alternativas + impacto. Requiere aprobación. |

## 5. Principios de producto

1. **El agente propone, la persona decide.** (BR-18)
2. **Nada de datos sensibles en el scoring.** (BR-20)
3. **La explicabilidad es parte del producto**, no un extra. (US-04.7)
4. **Un dato sin fuente no se muestra.** Cada cifra en pantalla es trazable a un cálculo.
5. **El vacío es información**: "sin proyecto" es un estado a primera clase, no un filtro.

## 6. Superficie funcional

| Superficie | Contenido |
|---|---|
| **Tablero de Proyectos** | Timeline con zoom (semana/quincena/mes/año), hitos, swimlanes de asignación, semáforo de presión, card de proyecto, card de persona, organigrama. |
| **Tablero de Personal** | Listado con filtros, pool de desasignados, ficha 360 (antigüedad, historial, skills, vacaciones, cumpleaños, ausencias), timeline individual alineado con el de proyectos. |
| **Coordinador IA** | Propuesta al crear proyecto, monitoreo de presión, propuestas de préstamo temporal, cobertura de ausencias, bandeja de aprobaciones, auditoría. |

## 7. Métricas del producto

| Métrica | Definición |
|---|---|
| Utilización de persona | `carga / capacidad nominal` (ventana 4 semanas) |
| Utilización de proyecto | `Σ demanda por rol / Σ capacidad del equipo` |
| Índice de presión | `f(avance, carga restante, capacidad disponible, ausencias impacto)` |
| Gap de roles | Roles de plantilla con cobertura < 100 % |
| Índice de disponibilidad | % del pool con capacidad libre |
| Préstamos vigentes | count + FTE prestado + % devueltos en fecha |
| Lead time de propuesta | `aprobación − generación` |
| Ausencias sin cobertura | count |

## 8. Restricciones asumidas

| ID | Restricción | Se confirma en |
|---|---|---|
| A-01 | El esfuerzo por rol se registra a mano al inicio del proyecto (sin Jira en v1). | Fase 1 |
| A-02 | Capacidad nominal uniforme: 8 h/día × días laborables, ajustable por jornada parcial. | Fase 1 |
| A-03 | 22 días laborables de vacaciones al año, con saldo arrastrable. | Fase 1 |
| A-04 | El umbral de 75 % es el valor por defecto y es configurable por proyecto. | Fase 1 |
| A-05 | Un solo tenant, sin aislamiento por cliente. | Fase 1 |

## 9. Gobernanza y riesgo (no negociable)

| Riesgo | Control |
|---|---|
| Sesgo algorítmico en la recomendación | Prohibir atributos sensibles en el scoring; publicar el pool completo de candidatos; auditar la distribución de propuestas. |
| Automation bias (la persona obedece la máquina) | Human-in-the-loop obligatorio + explicabilidad + auditoría + nada se aplica solo. |
| Datos de salud (incapacidades) | Acceso restringido y auditado; no visible en tableros generales. |
| Alucinaciones del agente | El agente razona solo sobre datos expuestos por tools; toda cifra en su salida viene de una tool verificable. |
| Legitimidad de la decisión | La UI debe mostrar siempre la evidencia y quién aprobó. |

## 10. Definición de éxito del proyecto (curso)

- [ ] `specs/`, `designs/`, `tasks/` y `traceability/matrix.md` completos y coherentes.
- [ ] Cada requisito con criterio EARS y al menos un test.
- [ ] Los dos tablereros navegables con los zoom y las cards descritos.
- [ ] El agente coordinador genera propuestas trazables, auditables y aprobables.
- [ ] La propuesta final (`docs/PROPUESTA_FINAL.md`) con la opinión de todas las áreas de trabajo.
