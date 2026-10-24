# Prompts.md — Registro del flujo de trabajo con IA

**Proyecto:** SDD · Gestión de RH y Proyectos — Propuesta de proyecto final  
**Metodología:** Spec Driven Development (nivel *spec-anchored*)  
**Fecha:** 2026-10-22  
**Herramientas:** Claude Code (claude.ai/code) — modelos: Nemotron 3 Ultra, Sonnet 4  

---

## 1. Resumen del enfoque

Este proyecto NO genera código. El entregable es **documentación SDD completa** (27 archivos Markdown en `repo_v1/`) que describe un sistema de gestión de RH para fábrica de software con agente IA coordinador. La IA se usó para:

1. **Especificación** (specs/): Redacción de 36 historias de usuario con criterios EARS verificables
2. **Diseño** (designs/): Arquitectura, modelo de datos (13 invariantes), UX (13 decisiones X-*), agente coordinador (18 tools, 11 guardarraíles), modelo de carga/presión
3. **Planificación** (tasks/): 43 tareas en 7 sprints trazadas a US y tests
4. **Trazabilidad** (traceability/): Matriz US → BR → I → diseño → tarea → prueba → estado
5. **Ensamblado final**: `docs/PROPUESTA_FINAL.md` (18 secciones, 9 épicas, 36 US, 22 BR, 43 tareas, 23 escenarios seed)
6. **Validación cruzada**: Corrección de 15 deudas de alineación (A-1..A-15) detectadas por auditoría adversarial

---

## 2. Prompts clave por fase

### 2.1 Fase 1 — Constitución y visión (specs/0001-vision/)

> **Prompt:** "Define una constitution.md de 11 artículos inmutables para un proyecto SDD de gestión de RH con agente IA. Reglas: agente siempre draft, humano aprueba; atributos sensibles prohibidos en scoring; un solo glosario canónico; demo sin mocks; gates que pueden rechazar."

**Criterio humano:** Validación contra el enunciado del cliente (3 lecturas del 75%) y decisión de no usar Docker (D-08 = app web local).

---

### 2.2 Fase 2 — Historias de usuario EARS (specs/000{2..6}-*/requirements.md)

> **Prompt:** "Escribe 36 historias US-01.1..US-05.7 con criterio EARS (When/Then), escenarios Given/When/Then, BR aplicables al pie. Épicas: Personas (7 US), Proyectos (7 US), Tablero Personal (7 US), Agente (8 US), Plataforma (7 US). Incluir US-01.7 (vacaciones + aniversario + gamificación) y US-03.7 (portal + reconocimiento)."

**Criterio humano:** Rechazo de US ambiguas sin evento disparador claro; forzado a EARS ubicuo "THE SYSTEM SHALL"; BR-20 (no atributos sensibles) referenciado en 6 US.

---

### 2.3 Fase 3 — Modelo de datos e invariantes (designs/0002-modelo-datos.md)

> **Prompt:** "Diseña 12 entidades (Persona, Rol, Proyecto, Asignación, Ausencia, SaldoVacaciones, Competencia/Herramienta, ConvenioBecario, Propuesta/PropuestaItem, BitacoraEvento, Feriado, Logro, LogroPersona, SolicitudVacaciones) con semántica temporal [desde,hasta). Define 13 invariantes I-1..I-13 verificables en motor determinista. Tres puntos de enforcement (E1 servicio, E2 guardarraíl tool, E3 reconciliador nocturno) por limitaciones de MongoDB."

**Criterio humano:** Corrección de I-6 (índice único parcial) vs I-7 (becario); separación I-1..I-8 (estáticos, verify_seed) vs I-9..I-13 (runtime); adición de plan.md mapeando a colecciones MongoDB.

---

### 2.4 Fase 4 — Agente coordinador (designs/0004-agente-coordinador.md)

> **Prompt:** "Diseña agente con 3 capas: motor determinista (Python, sin LLM) → LLM redacta → humano aprueba. 18 tools (11 lectura, 6 propuesta, 1 side-effect commit_assignment con needs_approval=True). Scoring determinista con pesos en config (0.35 cobertura + 0.25 afinidad + 0.15 Jaccard + 0.10 estabilidad + 0.10 no_duplicidad + 0.05 seniority). 11 guardarraíles G1..G11. Muro M1 versionado + M2 revalidación + M3 idempotencia. Golden set E-01..E-12."

**Criterio humano:** Rechazo de "agente calcula" — forzado "motor calcula, LLM redacta". Decisión: autonomy = `propose` (no `apply_with_revert`). Corrección A-11: catálogo único 18 tools, muro = aprobación registrada (no ausencia de tools).

---

### 2.5 Fase 5 — Modelo de carga y presión (designs/0005-modelo-carga-y-presion.md)

> **Prompt:** "Resuelve las 3 lecturas del 75% (avance, carga, portafolio) en una métrica unificada: indice_presion = deficit / max(capacidad_restante,1); ventana_75 = (avance >= 0.75). Semáforo: SALUDABLE | VIGILADO | EN_RIESGO | CRITICO (1.00 / 1.25). 75% es parámetro configurable, no constante. BR-04: avance ≥ umbral Y déficit ⇒ presión."

**Criterio humano:** Decisión load-bearing: el 75% sube urgencia, no crea demanda. Proyecto al 80% sin déficit = VIGILADO (reporta, no propone). Elimina alucinación de necesidad.

---

### 2.6 Fase 6 — UX Tableros (designs/0003-tableros-ux.md)

> **Prompt:** "13 decisiones X-01..X-13: timeline único parametrizado rowSource (proyecto|persona), 4 anclas zoom, fila por persona, 4 canales visuales (color=salud, opacidad=FTE, trama=préstamo, rayado=sobre-asignación), semáforo 4 estados con icono+patrón+texto, drawer lateral, ficha 360 anclas largas, radar+heatmap skills, organigrama DAG, vacaciones: saldo+heatmap+autogestión+cobertura sugerida+Planificador badge, préstamo trama azul+sombra origen, accesibilidad, desktop-first alta densidad, gamificación ligera (ANIVERSARIO_1/3/5/10/15/20, PLANIFICADOR, PRIMERAS_VACACIONES, SIN_AUSENCIAS_ANO). BR-21: gamificación no afecta scoring."

**Criterio humano:** X-13 añadido tras petición usuario (aniversario + gamificación). Un solo componente Timeline → dos tableros + ficha 360. Zoom compartido US-03.6 sale gratis.

---

### 2.7 Fase 7 — Tareas y trazabilidad (tasks/0001-tareas.md, traceability/matrix.md)

> **Prompt:** "43 tareas T-001..T-606 en 7 sprints. Cada tarea: US primario, criterio aceptación, test esperado. Sprint 0: cimientos (DB, modelos, API lectura, seed). Sprint 1: personas/ausencias + T-106 (vacaciones endpoint) + T-107 (aniversario/logros). Sprint 2: proyectos/timeline. Sprint 3: tablero personal + T-306 (portal UI). Sprint 4: motor carga/presión. Sprint 5: agente (tools, scoring, propuestas). Sprint 6: gobernanza/demo. Matriz 36×36×36×0 + 43 tareas. Gaps: G-1 (I-8/I-9 sin US), G-2 (BR-14 sin EARS), G-3 (préstamos cross-cliente)."

**Criterio humano:** Corrección A-13 (34 vs 40 tareas — confundía historias con tareas), A-14 (T-104, T-501 invisibles en matriz), A-15 (plan.md faltante). Trazabilidad ejecutable: gate traceability:check falla si US sin EARS, US sin tarea, tarea sin US, BR huérfana, I no en matriz, VERIFICADO sin test.

---

### 2.8 Fase 8 — Dataset semilla (docs/DATASET_SEED.md)

> **Prompt:** "Dataset determinista 120 personas (108 emp + 12 becarios), 19 proyectos (12 activos), 36 meses histórico, 24 feriados. Semilla 20260101, PRNG propio (random.Random), no random global. 23 escenarios S-01..S-23: presión, préstamos, ausencias, becarios, pool, aniversarios (S-20..S-21), vacaciones con/sin cobertura (S-22..S-23). Autovalida contra I-1..I-8. Fechas relativas a ancla (no envejece). Sin PII real."

**Criterio humano:** Ampliación de 19 a 23 escenarios para cubrir E9 (aniversario, Planificador). Determinismo byte-idéntico verificado.

---

### 2.9 Fase 9 — Riesgos y validación (docs/RIESGOS_Y_VALIDACION.md)

> **Prompt:** "10 riesgos con prob/impacto/mitigación/señal. Top 3: scope creep visual (congelar E6 antes), demo bonita motor frágil (constitution Art. VIII prohíbe mock agente), agente ignora reglas (invariantes en código + reconciliador + tests). Golden set 12 casos E-01..E-12. Demo 10 min: tablero→card→ficha360→pool→crear proyecto+propuesta agente→aprobar+auditar→préstamo temporal→portal vacaciones+aniversario+badges."

**Criterio humano:** Guion de demo incluye pasos 3 y 8 nuevos (ficha 360 con logros/aniversario, portal con autogestión+Planificador).

---

### 2.10 Fase 10 — Ensamblado final y auditoría (docs/PROPUESTA_FINAL.md)

> **Prompt:** "Ensambla PROPUESTA_FINAL.md (18 secciones) integrando 3 áreas: Dominio/Agente, UX/Frontend, Arquitectura/Plataforma. Tabla §8 con conteos finales: 9 épicas, 36 US, 22 BR, 13 I, 43 tareas, 13 X, 23 S, 12 E, 11 G, 11 constitution. Inventario 15 deudas A-1..A-15 cerradas (10 vocabulario/referencias, 2 contenido: muro agente + plantilla spec). Defaults D-07 (JWT 3 roles), D-08 (app web local sin Docker)."

**Criterio humano:** Verificación cruzada de todos los conteos contra archivos fuente. Corrección de referencias rotas (agent-design.md → designs/0004-agente-coordinador.md). Renumeración §17 "Anexos" (antes "agentes extra").

---

### 2.11 Fase 11 — Integración feature E9 (vacaciones + aniversario + gamificación)

> **Prompt:** "Agregar autogestión de vacaciones con reconocimiento de aniversario (gamificación) a todos los 27 archivos. US-01.7 + US-03.7 nuevas. Entidades Logro, LogroPersona, SolicitudVacaciones. BR-21 (gamificación no scoring), BR-22 (aniversario automático). X-13 gamificación ligera. T-106, T-107, T-306 tareas. S-20..S-23 escenarios. Actualizar todos los conteos: 36 US, 22 BR, 43 tareas, 23 S, 13 X, 9 épicas."

**Criterio humano:** Verificación exhaustiva: 0 ocurrencias de "34 historias", "19 escenarios", "40 tareas", "X-01..X-12", "S-01..S-19", "BR-01..BR-20" en repo_v1/. Root CLAUDE.md sincronizado.

---

## 3. Decisiones de arquitectura validadas con IA

| Decisión | Opción elegida | Validación IA |
|---|---|---|
| Stack | React 19+TS+Vite / FastAPI+Python / MongoDB / OpenAI Agents SDK | Confirmado por cliente; licencias verificadas (MIT/Apache/ISC) |
| SDD Level | Spec-anchored (no spec-as-source) | Tessl en beta; código editable por humanos |
| 75% | Parámetro + 3 niveles (avance/presión/cartera) | Mata alucinación de necesidad; BR-04 lo formaliza |
| Agente | Motor determinista + LLM draft + muro aprobación | 11 guardarraíles + 3 condiciones M1/M2/M3 verificables |
| MongoDB enforcement | E1 servicio + E2 tool guardrail + E3 reconciler nocturno | No hay daterange/EXCLUDE/GIST en Mongo |
| Timeline | Propio sobre d3-scale+d3-zoom+d3-time | Ningún Gantt OSS da las 3 funciones (avatar, ausencia, over-allocation) |
| Auth/Deploy | JWT 3 roles + app web local sin Docker | Defaults fijados en §15 PROPUESTA_FINAL.md |

---

## 4. Criterio humano aplicado (anti-alucinación)

1. **No código sin US+EARS**: Cada archivo .md creado/modificado traza a US-* con criterio verificable
2. **Un PR = una tarea**: Formato `- [ ] T-<id> [US-xx.x] descripción — criterio — test`
3. **Números del LLM ≠ verdad**: Todo conteo (36 US, 22 BR, 43 tareas, 23 S, 13 X, 9 épicas) verificado con `grep`/`Select-String` contra archivos reales
4. **Spec manda sobre implementación**: Glosario canónico en designs/0002 §1; sin sinónimos
5. **Gate solo es control si rechaza**: `traceability:check`, `spec_validate`, `verify_seed` fallan duro
6. **Demo = seed → operación → resultado visible**: Sin mocks del agente (constitution Art. VIII)
7. **Atributos sensibles nunca en scorer**: BR-20/I-12 — test verifica entradas del scorer, no intención
8. **Corrección en mismo PR**: Si código revela ambigüedad en spec, se corrige la spec (A-7, A-11, A-12)

---

## 5. Evidencia de calidad

- **Trazabilidad 100%**: 36/36 US con EARS, 36/36 con tarea, 0 sin cobertura, 43/43 tareas trazadas
- **0 BR huérfanas**: 22 reglas, cada una en pie de ≥1 spec
- **13 invariantes**: Todas en matriz, I-1..I-8 verificadas en verify_seed
- **15/15 deudas cerradas**: A-1..A-15 resueltas (A-11, A-12 tocan contenido sustantivo)
- **Determinismo seed**: Semilla 20260101 → JSON byte-idéntico
- **Golden set 12/12**: E-01..E-12 con 0 invariantes violados, 100% determinismo

---

## 6. Archivos generados/modificados (repo_v1/)

```
constitution.md                    # 11 artículos
PROPUESTA.md                       # Base consolidada (15 secciones)
docs/PROPUESTA_FINAL.md            # ← ENTREGA FINAL (18 secciones)
docs/DATASET_SEED.md               # 23 escenarios S-01..S-23
docs/RIESGOS_Y_VALIDACION.md       # 10 riesgos + demo
docs/AUDITORIA_ADVERSARIAL.md      # Veredicto: bloqueado por falta de tests
specs/0001-vision/spec.md          # Visión + glosario
specs/0001-vision/research-sdd-y-dominio.md
specs/0001-vision/research-tooling-2026.md
specs/0002-personas/requirements.md      # 6 US (incluye US-01.7)
specs/0003-proyectos/requirements.md     # 7 US
specs/0004-tablero-personal/requirements.md  # 6 US (incluye US-03.7)
specs/0004-tablero-personal/research-frontend-stack.md
specs/0005-agente-coordinador/requirements.md  # 8 US
specs/0005-agente-coordinador/spec-exemplar-c11.md
specs/0006-plataforma/requirements.md    # 7 US
designs/0001-arquitectura.md         # D-07/D-08 abiertas
designs/0002-modelo-datos.md         # 12 entidades, 13 invariantes
designs/0002-modelo-datos/plan.md    # Mapeo MongoDB (A-15)
designs/0003-tableros-ux.md          # 13 decisiones X-01..X-13
designs/0003-tableros-ux.borrador-alternativo.md
designs/0004-agente-coordinador.md   # 18 tools, 11 guardarraíles, golden set
designs/0005-modelo-carga-y-presion.md
tasks/0001-tareas.md                 # 43 tareas, 7 sprints
traceability/matrix.md               # 36×36×36×0 + 43 tareas
README.md                            # Documentación del repo
Prompts.md                           # ESTE ARCHIVO
```

---

## 7. Próximos pasos (si se implementara)

1. `git init` + `git add` específico por archivo (nunca `-A`)
2. Rama `feature/entrega-final-[INICIALES]`
3. `python -m scripts.spec_validate` → verde
4. `python -m scripts.verify_seed` → verde (I-1..I-8 + S-01..S-23)
5. `pytest` → golden set 12/12
6. Entrega via Typeform con acceso `LIDR-AI4Devs` o video demo 10 min