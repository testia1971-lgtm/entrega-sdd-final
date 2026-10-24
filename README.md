# Entrega Final — Gestión de RH y Proyectos (SDD + Agente IA Coordinador)

> **Proyecto final de la capacitación AI4Devs · Spec Driven Development**  
> Entrega: rama `feature/entrega-final-LL` (sin git en este checkout — ver §8)  
> Repositorio de detalle: [`../repo_v1/`](../repo_v1/)

---

## 1. Ficha del proyecto

| Campo | Valor |
|---|---|
| **Nombre** | Gestión de RH y Proyectos — fábrica de software |
| **Tipo** | Aplicación web end-to-end (agente IA coordinador integrado) |
| **Metodología** | Spec Driven Development (nivel *spec-anchored*, gates en CI) |
| **Alcance** | MVP documentado: 36 historias de usuario (EARS), 9 épicas, 22 reglas de negocio, 13 invariantes, 43 tareas en 7 sprints |
| **Estado** | **Propuesta SDD completa y trazable** (specs → designs → tasks → matriz). El entregable del curso es documental; el software se implementa siguiendo la trazabilidad US→tarea→test |
| **Stack decidido** | React 19 + TS + Vite · FastAPI + Python (Pydantic v2) · MongoDB · OpenAI Agents SDK · SSE |
| **Semilla** | Dataset determinista 20260101 (120 personas, 19 proyectos, 23 escenarios) |
| **Equipo** | Individual |

### 5 funcionalidades must-have (historias de usuario)

| # | US | Funcionalidad | Valor |
|---|---|---|---|
| 1 | **US-02.6** | Semáforo de presión (`indice_presion = déficit ÷ capacidad_restante`, ventana 75 %) | Ve la presión a tiempo: un proyecto al 78 % con déficit se marca; al 80 % sin déficit, no (BR-04) |
| 2 | **US-04.1** | Propuesta de asignación al crear proyecto (agente coordinador) | El agente propone equipo con score determinista, justificación y descartes — **nunca aplica** |
| 3 | **US-04.6** | Propuestas y aprobaciones (muro M1–M3) | Aprobación humana versionada, revalidación de invariantes en el commit, idempotencia |
| 4 | **US-04.3** | Préstamo temporal con fecha de devolución y piso de reserva (40 %) | El préstamo cede capacidad sin dejar al origen bajo su piso; el agente respeta la ventana |
| 5 | **US-01.7 / US-03.7** | Autogestión de vacaciones + reconocimiento de aniversario (gamificación ligera) | La persona solicita vacaciones con cobertura sugerida; logros de antigüedad (1/3/5/10/15/20 años) y badge Planificador (≥30 días de antelación); **BR-21: la gamificación no toca el scoring del agente** |

Cada US tiene criterio EARS verificable + test previsto en [`../repo_v1/traceability/matrix.md`](../repo_v1/traceability/matrix.md).

---

## 2. Arquitectura (resumen)

**Tres capas — el motor calcula, el LLM redacta, la persona decide:**

```
Motor determinista (Python puro, sin LLM)
  carga · presión · cobertura · candidatos · I-1..I-13
        │ hechos JSON con as_of
        ▼
Agente coordinador (OpenAI Agents SDK, dentro del backend)
  redacta propuesta EN_REVISION · explica · pregunta (G7)
        │ propuesta
        ▼
Persona: aprueba / rechaza → commit_assignment (needs_approval=True)
```

- **El LLM nunca emite un número**: toda cifra sale de una tool y se revalida contra el motor.
- **Muro de aprobación**: M1 versión al hash del payload · M2 revalidación de invariantes al commit · M3 idempotencia `(proposal_id, version)`.
- **MongoDB no impone rangos** → 3 puntos de enforcement: E1 servicio de dominio · E2 guardarraíl de tool · E3 reconciliador nocturno.
- **Semántica temporal**: todo periodo `[desde, hasta)`. Histórico append-only, sin soft delete.
- Detalle: [`../repo_v1/designs/0001-arquitectura.md`](../repo_v1/designs/0001-arquitectura.md) y [`../repo_v1/designs/0004-agente-coordinador.md`](../repo_v1/designs/0004-agente-coordinador.md).

**Decisiones de stack con licencias verificadas**: timeline propio sobre `d3-scale`+`d3-zoom`+`d3-time` (ningún Gantt OSS da avatar + ausencias + sobre-asignación); organigrama con `@xyflow/react`+`dagre` (DAG); SSE (no WebSocket); rechazados: `fullcalendar/resource-timeline` (premium), `primereact` v11 (no MIT), `elkjs` (copyleft), `gantt-task-react` (archivado).

---

## 3. Modelo de datos (resumen)

12 entidades + 3 de gamificación: `Persona`, `Rol` (base y proyecto), `Proyecto`, `Asignación`, `Ausencia`, `SaldoVacaciones`, `Competencia`/`Herramienta`, `ConvenioBecario`, `Propuesta`, `BitacoraEvento`, `Feriado`, **`Logro`**, **`LogroPersona`**, **`SolicitudVacaciones`**.

**13 invariantes (lo que el sistema no puede violar)** — extracto:

| # | Invariante |
|---|---|
| I-1 | Σ % de asignaciones de una persona ≤ 100 % |
| I-2 | Préstamo con `fechaDevolucionPrevista` obligatoria |
| I-3 | Préstamo activo cede la capacidad al destino |
| I-5 | Cobertura termina a más tardar al regreso de la ausencia |
| I-6 | Sin dos asignaciones activas iguales `(persona, proyecto, rol)` |
| I-7 | Becario solo dentro de su convenio; nunca candidato a préstamo sin RRHH |
| I-10 | Toda mutación deja `BitacoraEvento` |
| I-11 | Préstamo nunca deja al origen bajo su piso de reserva (40 %) |
| I-12 | El scoring **no** recibe atributos sensibles (salud, edad, género…) |

Detalle completo: [`../repo_v1/designs/0002-modelo-datos.md`](../repo_v1/designs/0002-modelo-datos.md) + [`plan.md`](../repo_v1/designs/0002-modelo-datos/plan.md) (mapeo a colecciones MongoDB con índices).

---

## 4. Estructura del repositorio

```
repo_v1/
├── constitution.md                  # 11 artículos inmutables
├── PROPUESTA.md                     # base consolidada (15 secciones)
├── Prompts.md                       # registro del flujo de trabajo con IA
├── docs/
│   ├── PROPUESTA_FINAL.md           # ← entregable del curso (18 secciones)
│   ├── DATASET_SEED.md              # dataset determinista + 23 escenarios
│   ├── RIESGOS_Y_VALIDACION.md      # 10 riesgos + guion de demo 10 min
│   └── AUDITORIA_ADVERSARIAL.md     # auditoría de trazabilidad
├── specs/0001..0006/                # QUÉ: 36 US con EARS + BR aplicables
├── designs/0001..0005               # CÓMO: arquitectura, datos, UX, agente, presión
├── tasks/0001-tareas.md             # QUIÉN: 43 tareas, 7 sprints
└── traceability/matrix.md           # US → BR → I → diseño → tarea → test
```

---

## 5. Cómo verificar la entrega (evidencia)

No hay código (el entregable es la propuesta documental). La verificación es **ejecutable sobre los Markdown**:

```bash
# 1. Los conteos de la matriz cuadran con los archivos
#    36 US · 36 EARS · 36 con tarea · 0 sin cobertura · 43 tareas · 22 BR · 23 S · 13 X
grep -c "US-0" repo_v1/specs/*/requirements.md
grep "Totales:" repo_v1/traceability/matrix.md

# 2. Sin números viejos (regresiones de conteo)
grep -rn "34 historias\|19 escenarios\|40 tareas\|S-01..S-19\|BR-01..BR-20" repo_v1/
# → debe devolver 0 coincidencias

# 3. Referencias rotas: ninguna US o BR mencionada sin definirse
grep -rhoE "US-[0-9]+\.[0-9]+" repo_v1/ | sort -u
grep -rhoE "BR-[0-9]+" repo_v1/ | sort -u
```

**Evidencia de calidad ya verificada en esta entrega:**
- Trazabilidad 100 %: 36/36 US con EARS, 36/36 con tarea, 0 huérfanas, 43/43 tareas con fila en la matriz.
- 15/15 deudas de alineación cerradas (A-1..A-15, incluidas 2 que tocaban contenido: muro del agente y plantilla de spec).
- Determinismo del seed: misma semilla ⇒ JSON byte-idéntico (PRNG propia, sin `random` global).
- Guion de demo completo en `docs/PROPUESTA_FINAL.md` §13.2 (seed → operación → resultado visible, sin mocks del agente).

---

## 6. Decisiones de diseño (por qué este stack)

| Decisión | Motivo |
|---|---|
| Motor determinista separado del LLM | El LLM formula, no calcula (MIT 2025, Timefold): asignar personas es un espacio de ~10^63 soluciones — la búsqueda es código |
| `propose` como autonomía fijada | Constitution Art. V + BR-18: el agente nunca aplica; `apply_with_revert` documentado y desactivado |
| El 75 % como parámetro, no constante | Tres lecturas del enunciado (avance/carga/cartera) → una métrica unificada; el 75 % sube urgencia, no crea demanda |
| MongoDB + reconciliador nocturno | Sin `daterange`/`EXCLUDE GIST`: segunda capa de defensa = job que escanea I-1..I-13 y alerta |
| Spec-anchored, no spec-as-source | La spec sobrevive a la implementación y se valida en CI; el código sigue editable por humanos |
| Timeline propio sobre d3 | Ninguna librería Gantt OSS daba las 3 funciones necesarias (verificado, licencias una por una) |

---

## 7. Riesgos top y mitigación

1. **Scope creep visual** → orden de sacrificio escrito (E8 → E7 → E3); E6 (agente) no se sacrifica.
2. **Demo bonita, motor frágil** → prohibido el mock del agente (constitution Art. VIII); golden set E-01..E-12 en CI.
3. **Agente ignora reglas** → invariantes en el motor + guardarraíl de tool + reconciliador + test que verifica las *entradas* del scorer (no la intención).

Detalle: [`../repo_v1/docs/RIESGOS_Y_VALIDACION.md`](../repo_v1/docs/RIESGOS_Y_VALIDACION.md).

---

## 8. Instrucciones de entrega (curso)

| Requisito (Instrucciones.txt) | Cumplimiento |
|---|---|
| `README.md` con ficha, arquitectura, modelo de datos, tickets | Este archivo + `repo_v1/tasks/0001-tareas.md` (43 tareas trazadas) |
| `Prompts.md` con flujo IA, prompts clave, criterio humano | [`../repo_v1/Prompts.md`](../repo_v1/Prompts.md) — 11 fases, decisiones, criterio anti-alucinación |
| 3–5 must-have como historias de usuario | §1 de este README (5 US con EARS) |
| Evidencia de funcionamiento | §5: verificación ejecutable + trazabilidad 100 %; el repo completo es la evidencia |
| Rama `feature/entrega-final-[INICIALES]` | Sugerida: `feature/entrega-final-LL`. **Nota**: este checkout no tiene `.git` (ver CLAUDE.md §8); al versionarlo: `git init && git checkout -b feature/entrega-final-LL && git add <archivo> específico` |
| Acceso `LIDR-AI4Devs` / Typeform | Acción del alumno: subir a repo privado con acceso de lectura, o fork público solo con docs; enviar en `lidr.typeform.com/proyectoai4devs` con el correo del calendario |

> El entregable del curso de esta propuesta es **documental**: `docs/PROPUESTA_FINAL.md` (18 secciones) integra la opinión de todas las áreas y es el documento a evaluar. La implementación sigue ticket por ticket desde `tasks/0001-tareas.md`, con cada criterio EARS cubierto por ≥ 1 test.

---

## 9. Implementación (MVP en `./MVP/`)

Este directorio contiene la implementación completa del software siguiendo la especificación SDD:

```
MVP/
├── contracts/api-contract.md   # contrato API (convergencia backend ∥ frontend)
├── backend/                    # FastAPI: models, services (motor), agent, routers, worker
├── tests/                      # pytest (criterios EARS) — 132 tests pasando
├── web/                        # React 19 + TS + Vite + Tailwind + shadcn/ui — 45 tests vitest
├── .env.example                # plantilla de variables
├── BITACORA.md                 # registro de la construcción multiagente
└── README.md                   # este archivo
```

### Cómo correr en local

```bash
# Backend
pip install -e .            # o: pip install -r requirements.txt
python -m scripts.seed      # dataset determinista → Ai-codingSep26
uvicorn backend.main:app --reload --port 8000

# Frontend
npm --prefix web install
npm --prefix web run dev    # http://localhost:5173 (proxy → :8000)

# Tests
pytest                      # suite completa (132 passed)
npm --prefix web test       # vitest (45 passed)
```

### Despliegue (Vercel CLI, dos proyectos)

```bash
# api (FastAPI serverless)          # web (SPA)
cd MVP && vercel --prod             # cd MVP/web && vercel --prod
```

Variables en Vercel: `MONGO_URL`, `MONGO_DB_NAME`, `OPENAI_API_KEY`, `AGENT_MODEL`,
`JWT_SECRET`, `CRON_SECRET`, `CORS_ORIGINS` (api) · `VITE_API_URL` (web).

> Detalle de la construcción multiagente: [`BITACORA.md`](BITACORA.md).
> Especificación canónica: `repo_v1/docs/PROPUESTA_FINAL.md`.