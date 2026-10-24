# Riesgos, supuestos y plan de validación

> Área responsable: dominio y agente · Estado: borrador
> Uso: el evaluador de la capacitación mirará tanto el entregable como el proceso. Esta lista es la defensa
> del proyecto frente a "se veía bonito en pantalla pero no se sostiene".

---

## 1. Riesgos del proyecto

| # | Riesgo | Prob. | Impacto | Mitigación concreta | Señal de alarma temprana |
|---|---|---|---|---|---|
| R-01 | **Scope creep**: "tablero visualmente rico" se come el presupuesto y no queda la parte difícil (el agente). | Alta | Alto | Regla de corte por épica: una épica entra solo si se puede demostrar en la demo. La FIRST (épica del agente) se congela antes que el resto. | Al week 2, más de 3 épicas "a medio hacer" |
| R-02 | **Demo bonita, motor frágil**: el tablero funciona con datos mocks y el agente no lo usa. | Media | Alto | Constitution Art. VIII: toda épica se demuestra con el flujo real (seed → operación → resultado). Prohibido el mock del agente. | Un componente con datos hardcodeados |
| R-03 | **El agente "hace theater"**: responde en texto pero no calcula. | Media | Alto | Separación motor determinista / LLM (`designs/0004-agente-coordinador.md` §1) + golden set con invariantes = 0. | Una propuesta del agente sin `snapshotEntradas` |
| R-04 | **Datos irreales**: no hay set real y se inventa a posteriori un "histórico" inconsistente. | Media | Alto | El seed determinista con escenarios sembrados a propósito (S-01..S-23) es el set "real" del proyecto. Si llegan datos reales: importador CSV al mismo esquema + anonimización. | Cualquier cifra del tablero que no salga del seed |
| R-05 | **El umbral de 75 % no significa nada** si no se define "capacidad". | Alta | Alto | `factorCapacidad` por rol + `capacidadBase` por persona + config; los umbrales son parámetros, no constantes. | "El tablero dice 78 % de carga" y nadie sabe qué es el 100 % |
| R-06 | **Sobrecarga de SDD**: muchos MD, poca implementación (riesgo documentado por Fowler). | Media | Medio | 1 spec por épica; `requirements.md` ≤ 120 líneas; los anexos enlazados, no duplicados. | Más de 4 archivos MD por épica sin código asociado |
| R-07 | **El agente ignora las reglas** (falsa sensación de control, Fowler). | Media | Alto | Invariantes en el motor determinista + guardarraíl de tool + reconciliador nocturno + tests; la constitution es una capa extra, no la única. | Un test que falla solo "cuando el LLM se equivoca" |
| R-08 | **El panel de estado y las notificaciones se quedan en el aire** (si se prometen y no hay tiempo). | Media | Medio | Son *adicionales*, no críticos: el agente expone su estado en la UI, pero el flujo de propuesta y aprobación de la app funciona solo. | — |
| R-09 | **Privacidad**: el dataset o la demo exponen datos que parecen reales. | Baja | Alto | Nombres/correos sintéticos; en la UI solo se expone tipo de ausencia y rango de fechas. | Un "diagnóstico" o "motivo médico" en pantalla |
| R-10 | **El evaluador no entiende SDD** si el repo no lo deja ver el proceso. | Media | Medio | README con topología tipo spec-kit + `PROPUESTA.md` + PRs con la spec en la descripción. | Preguntas tipo "¿esto es SDD o un Jira?" |

---

## 2. Supuestos (si el humano no responde, se avanza con estos)

| # | Supuesto | Impacto si cambia | Dónde se configura |
|---|---|---|---|
| A-01 | Presión por proyecto = carga comprometida / capacidad del equipo; bandas 70 / 85 % | Cambia los umbrales y la lógica del agente | `Umbral.presionProyectoPct` |
| A-02 | El 75 % se aplica **también** a nivel portafolio | Cambia la alerta de presión organizacional | `Umbral.presionPortafolioPct` |
| A-03 | Capacidad base = 21 días laborables/mes × 8 h, menos feriados y ausencias | Escala todas las cifras del tablero | `Persona.diasLaborablesSemana`, `horasDia` |
| A-04 | Un Manager/PO aporta 50 % de su capacidad a un proyecto | Cambia la presión de todos los proyectos | `Rol.factorCapacidad` |
| A-05 | El rol base es único y vitalicio; los roles de proyecto son instancias | Cambia el modelo y la UI de ficha | `Persona.rolBaseId`, `Rol.alcance` |
| A-06 | Los préstamos temporales se ceden desde el proyecto origen (el origen baja su %) | Cambia la I-3 | `Asignacion.asignacionOrigenId` |
| A-07 | El agente propone, la persona decide (nunca auto-aplica) | Cambia la UX y el risk model | Constitution V |
| A-08 | Dataset sintético determinista (no hay datos reales) | Cambia el plan de datos | `docs/DATASET_SEED.md` |
| A-09 | La demo vive en la aplicación web, que es la fuente de verdad; el estado del agente y las notificaciones se muestran dentro de la app | Cambia el alcance de la integración | `designs/0004-agente-coordinador.md` §8 |
| A-10 | Multi-idioma no es requisito; la UI es en español | Añade i18n | — |

---

## 3. Plan de validación por épica

| Épica | Se valida con | Criterio de "hecho" |
|---|---|---|
| E1 · Modelo de dominio y datos | Tests de invariantes (I-1..I-13) + `verify_seed.py` | Todos los invariantes cubiertos por test; el seed pasa verificación |
| E2 · Tablero de proyectos (Gantt multi-zoom) | E2E Playwright: cambiar zoom semana→año mantiene legibilidad y coherencia | Capturas en 4 zooms + E2E verde |
| E3 · Ficha de persona | E2E + golden set de 5 fichas | Las 5 fichas coinciden con el seed |
| E4 · Ausencias, vacaciones, cobertura | E2E del flujo vacaciones→cobertura; test de I-5 | El flujo completo funciona con el seed |
| E5 · Préstamos temporales | E2E: crear préstamo → baja el origen → se respeta la devolución | Invariantes I-2..I-4 en verde |
| E6 · Agente coordinador | Golden set (E-01..E-12) | 0 invariantes violados; 100 % determinismo; detección de presión 100 % en los casos sembrados |
| E7 · Becarios y organigrama | E2E + test I-7 + test I-9 | Sin asignación fuera de convenio; organigrama coherente con asignaciones |
| E8 · Notificaciones y estado del agente | Prueba manual en la app: notificación visible + enlace a la propuesta | Publicación visible con link a la propuesta |

---

## 4. Guion de demo (10 minutos)

1. **(1 min)** Tablero general: 19 proyectos, zoom semana→mes→año. Señalar S-01 (ROJO) y S-02 (ÁMBAR, 78 %).
2. **(1.5 min)** Click en el proyecto en ámbar → card con equipo, carga por semana, roles y quién está en préstamo.
3. **(1.5 min)** Click en un analista → ficha: antigüedad, skills, herramientas, historial, vacaciones, cumpleaños.
4. **(1 min)** Panel "Disponibles": el pool de desasignados con sus skills.
5. **(2.5 min)** Crear un proyecto nuevo → **el agente propone asignaciones** → explicar por qué eligió a cada persona
   y por qué descartó a las otras (es el corazón de la propuesta).
6. **(1.5 min)** Aprobar una asignación → el tablero se actualiza; auditar en la bitácora.
7. **(1 min)** Préstamo temporal: crearlo, ver cómo baja el proyecto origen y la fecha de devolución; mostrar
   que el agente no vuelve a proponer a esa persona mientras dure el préstamo.

---

## 5. Lo que NO se va a hacer (anti-scope, explícito)

- Multi-tenant / multi-empresa.
- Payroll, nómina, contratación (ATS) o evaluación de desempeño.
- Facturación al cliente y control de presupuesto financiero.
- App móvil nativa (el responsive web cubre el caso).
- Integracion con Jira/Azure DevOps (se documenta la interfaz, no se implementa).
- Machine learning para predecir carga (el motor es heurístico y determinista a propósito).
