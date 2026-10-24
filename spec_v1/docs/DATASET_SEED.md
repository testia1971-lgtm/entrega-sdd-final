# Dataset semilla — Especificación

> Área responsable: dominio y agente · Estado: borrador
> Objetivo: que la demo sea reproducible, que los tests sean estables y que el agente tenga dónde demonstrate
> sus Skills. **Por defecto: datos sintéticos y deterministas** (Constitution VII).

---

## 1. Parámetros

| Parámetro | Valor | Nota |
|---|---|---|
| Personas | **120** | 108 empleados + 12 becarios |
| Proyectos | **19** | 12 `ACTIVO`, 3 `EN_PAUSA`, 3 `CERRADO` (con histórico) |
| Histórico | **36 meses** | Asignaciones, ausencias y saldos de vacaciones |
| Semilla | `20260101` | Fija en el código; el generador es determinista |
| Proyectos nuevos | 2 | Se crean **en vivo durante la demo** para disparar `proyecto.creado` |
| Feriados | 24 | 12 feriados anuales × 2 años, por país |

**Regla de oro: nada de aleatoriedad global sin semilla.** El generador usa una instancia propia
`rng = random.Random(semilla)` (Mersenne Twister, estable entre versiones de CPython) y **nunca** el módulo
`random` global, ni `os.urandom`, ni `time`: con la misma semilla, el JSON es byte-idéntico en cualquier máquina.

---

## 2. Distribución de roles (diseño, no dato duro)

| Rol | Familia | Personas | % capacidad efectiva (factor) |
|---|---|---|---|
| Desarrollador Backend | ENGINEERING | 26 | 1.00 |
| Desarrollador Frontend | ENGINEERING | 22 | 1.00 |
| QA Analyst | QA | 18 | 1.00 |
| QA Automatizador | QA | 6 | 1.00 |
| Tech Lead | LEADERSHIP | 12 | 0.60 |
| Scrum Master | DELIVERY | 9 | 0.80 |
| Project Manager | DELIVERY | 8 | 0.50 |
| Product Owner | DELIVERY | 5 | 0.50 |
| DevOps / SRE | ENGINEERING | 6 | 1.00 |
| Data Engineer | DATA | 4 | 1.00 |
| Becario (trainee) | ENGINEERING/QA | 12 | 0.60 |

> El `factorCapacidad` modela la realidad de que un Manager o PO no dedication 100 % al proyecto.
> Es un parámetro del modelo (`Rol.factorCapacidad`), no una constante del generador.

Niveles: `JUNIOR` 30 %, `MID` 40 %, `SENIOR` 22 %, `LEAD` 6 %, `PRINCIPAL` 2 %.
Antigüedad: distribución sesgada — 22 % menos de 1 año, 48 % entre 1 y 4 años, 30 % más de 4 años.

---

## 3. Escenarios sembrados a propósito (el corazón del dataset)

El seed no es "personas aleatorias": **cada escenario existe para que una épica o un caso del golden set tenga
material real**. Si un escenario no está sembrado, la feature no se puede demostrar.

| ID | Escenario | Dónde | Para qué sirve |
|---|---|---|---|
| **S-01** | Proyecto al **92 %** de carga (ROJO) | `P-ROJO-01` | Detección de presión crítica |
| **S-02** | Proyecto al **78 %** (ÁMBAR) — cruza el umbral de 75 % | `P-AMBAR-01` | Umbral L1 y disparador del agente |
| **S-03** | **11 proyectos en presión simultáneamente** (≈75 % del portafolio activo) | portafolio | Umbral L2 (presión de portafolio) |
| **S-04** | 3 devs al **100 %** de asignación | `P-ROJO-01` | Invariante I-1 y "no proponer" |
| **S-05** | Dev senior con **skills exactas** (React, Node, PostgreSQL) libre al 40 % | pool | Candidato obvio para préstamo |
| **S-06** | Persona con **préstamo activo** que vence dentro de 30 días | `P-AMBAR-01` | Respeto al préstamo (I-2, I-8) · golden set E-05 |
| **S-07** | Persona **prestado** a un proyecto que entra en ROJO | `P-ROJO-01` | No "robar" gente de un proyecto crítico (I-11) · golden set E-11 |
| **S-08** | **Incapacidad aprobada** de 20 días sin cobertura | `P-AMBAR-01` | Propuesta de cobertura (I-5) · golden set E-03 |
| **S-09** | 9 personas de **vacaciones** en las próximas 2 semanas | varios | Capacity planning y panel de vacaciones |
| **S-10** | **Desasignadas** (sin asignación vigente) con roles y skills varias | pool | Espacio "disponibles" (§4.3 del data model) |
| **S-11** | **Becario** en la última semana de su convenio de 6 meses | `P-AMBAR-01` | I-7 (no asignar fuera del convenio) |
| **S-12** | **Cumpleaños** de 5 personas en el mes en curso | varias | Panel de cumpleaños |
| **S-13** | Saldos de vacaciones consumidos (0 días, 5 días, 22 días) | varias | Conteo de vacaciones en la ficha de persona |
| **S-14** | Proyecto con **Tech Lead ausente** (en Planner) | `P-ROJO-01` | Regla I-9 (alerta de rol crítico) |
| **S-15** | Dos candidatas con **skill set idéntico** | pool | Desempate determinista · golden set E-09 |

### 3.1 `CargaEstimada` es obligatoria en el seed

La presión es `demanda vs capacidad`. Si el seed no trae `CargaEstimada` (rol × semana × horas, con `origen`
estimado/real), el tablero no tiene nada que pintar y el agente no tiene nada que proponer.

| ID | Escenario | Para qué sirve |
|---|---|---|
| **S-16** | Proyectos con curva de demanda semanal realista: rame al inicio, meseta, y **cola** al final | `deficit` real en la cola → justifica refuerzos |
| **S-17** | Un proyecto cuyo `avance` va **adelantado** respecto a su demanda (va bien de carga, mal de calendario) | Distinguir "ventana de presión" de "déficit real": aquí **no** hay que proponer gente, hay que reordenar |
| **S-18** | Proyectos con `origen=real` menor que `origen=estimado` (el equipo va más rápido de lo previsto) | El índice de presión baja solo; el agente debe decirlo |
| **S-19** | **Aniversario de 1 año** — 3 personas cumplen su primer año esta semana | varias | Reconocimiento `ANIVERSARIO_1` + badge en portal |
| **S-20** | **Aniversario de 5 años** — 1 persona cumple 5 años (Quinquenio) | varias | Reconocimiento `ANIVERSARIO_5` + logro desbloqueado |
| **S-21** | **Solicitud de vacaciones con antelación ≥ 30 días** — 2 personas | varias | Badge `PLANIFICADOR` + punto de gamificación |
| **S-22** | **Solicitud de vacaciones sin cobertura completa** — 1 persona | `P-AMBAR-01` | Validación de cobertura sugerida antes de aprobar |
| **S-23** | **Préstamo que violaría `pisoReserva`** | `P-ROJO-02` | Préstamo debe ser rechazado por el motor (I-11, BR-08, BR-12) |

> Sin S-16/S-17 el agente no puede demostrar la diferencia entre "presión" y "ocupación", que es justamente la
> parte interesante del modelo (`designs/0005-modelo-carga-y-presion.md`).

### 3.2 Métricas de Ficha 360

El seed garantiza la presencia de datos para las 5 métricas clave de la ficha 360 de persona:

| Métrica | Cobertura en el Seed | Escenario/Origen |
|---|---|---|
| **Antigüedad** | Generada para todas las personas | `Persona.fechaIngreso`, distribución sesgada |
| **Skills** | Generadas para todas las personas | `personas_competencias.json`, `personas_herramientas.json` |
| **Historial** | 36 meses de datos | `asignaciones.json`, `personas_historial.json` (append-only) |
| **Vacaciones** | Saldos y registro de ausencias | `S-13`, `ausencias.json` (tipo `VACACIONES`) |
| **Cumpleaños** | Generados para todas las personas | `S-12`, `Persona.fechaNacimiento` |

---

## 4. Estructura de archivos

```
seed/
  seed.config.json          # parámetros del generador (semilla, conteos, fechas)
  personas.json             # salida generada
  proyectos.json
  asignaciones.json
  ausencias.json
  saldos_vacaciones.json
  skills.json
  herramientas.json
  feriados.json
scripts/
  seed.py                   # genera + carga en MongoDB (idempotente)
  verify_seed.py             # valida invariantes + cobertura de escenarios
```

**El generador escribe JSON; el seed se carga en la base con el script de seed del backend Python** (MongoDB,
`motor`/`pymongo`, con *upsert* idempotente y salida byte-idéntica entre ejecuciones).
Regla: el script de seed debe ser **idempotente** (borrado + inserción) y su salida debe ser **byte-idéntica**
entre ejecuciones. Test: `pytest test_seed_determinism` compara dos ejecuciones.

> `seed.config.json` sigue siendo JSON porque es **configuración declarativa** (el mismo archivo puede consumir el
> importador de datos reales). El código va en Python (`seed.py`, `verify_seed.py`), no en TypeScript: el motor
> determinista y las tools del agente son Python, y un seed en otro lenguaje obligaría a duplicar los modelos.

---

## 5. Reglas del generador

1. Determinismo total con la misma semilla → mismo JSON.
2. **Nunca genera datos que violen un invariante.** El generador valida su propia salida contra **I-1..I-8** (el
   subconjunto que es verificable sobre datos estáticos: capacidad, préstamo, cobertura, unicidad, convenio de
   becario, proyecto cerrado) antes de escribir; si violaría, ajusta (reduce %, acorta periodos, cambia rol).
   I-9..I-13 no aplican a un dataset estático (son reglas de transición y de auditoría: las verifica el motor en
   runtime y el reconciliador nocturno).
3. Cobertura mínima garantizada: al menos 1 instancia de S-01..S-23.
4. Los rangos de fechas son relativos a una fecha ancla (`seed.config.json.fechaAncla`) para que el dataset
   no envejezca: el proyecto se puede demonstrar dentro de un mes sin regenerar.
5. Sin datos reales ni PII real: nombres y documentos sintéticos con prefijos (`PER-0001`), correos `@ejemplo.local`.

---

## 6. Verificación

`scripts/verify_seed.py` falla si:

- el dataset no es reproducible (dos generaciones difieren),
- alguna asignación viola I-1..I-8,
- falta algun escenario S-01..S-23,
- hay personas `ACTIVA` sin `rolBaseId`,
- hay asignaciones en proyectos `CERRADO` (I-8),
- el pool de desasignados (S-10) tiene menos de 8 personas.

Este script es también un **artefacto de evaluación**: demuestra que la spec se puede verificar automáticamente.
